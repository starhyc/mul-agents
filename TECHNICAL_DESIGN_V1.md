# 技术方案 v1（可开发版）

> 面向：通用 Agent 框架 + 智能问题定位系统（MVP）
>
> 冻结原则：已确认项优先；未确认项采用默认值并标记为 `DEFAULT`。

---

## 1. 方案冻结总览

### 1.1 已确认（来自需求评审）
- 技术栈：后端 Python、前端 Vue。
- 部署：MVP 模块化单体。
- 存储：PostgreSQL + Redis。
- 消息队列：Redis Stream。
- 实时推送：WebSocket（按 `session` 多 channel）。
- Agent 执行引擎：基于 LangGraph 封装。
- MCP 与 Skill 沙盒：进程级隔离。
- 风险默认策略：平衡。
- R2/R3：不强制双人审批。
- 历史策略：审计日志与诊断详情永久保留，用户可删除。
- 相似问题推荐：方案 A（MVP 不引入向量数据库）。
- 用户体系：暂不接企业 SSO。
- 功能范围：问题类型同时上线；P0/P1 全部进入 V1；历史重放为 P0。
- 回放保真：全量原文。
- 回放导出：支持 Markdown 报告。

### 1.2 未确认项默认值（本次补齐）
- ReAct 无增量阈值：`N=3`（DEFAULT）。
- Hierarchical 子 Agent 失败率阈值：`40%`（DEFAULT）。
- `waiting_user` 超时策略：超时转人工待办（DEFAULT）。
- 重试策略：LLM 3 次指数退避；工具 2 次线性退避；节点支持 1 次人工重试（DEFAULT）。
- 删除策略：诊断软删；审计日志不可删（DEFAULT）。
- 性能口径：简单问题 `<30s` 按 P95；并发 100+ 指并发诊断任务（DEFAULT）。
- 事件补偿窗口：`24h`（DEFAULT）。
- API 幂等：创建诊断任务支持 `Idempotency-Key`（DEFAULT）。

---

## 2. 系统架构设计

### 2.1 架构风格
采用**模块化单体**，内部按领域模块拆分，通过事件总线（Redis Stream）解耦。

模块边界：
1. `gateway`：HTTP API + WS 接入、Session 鉴权、限流。
2. `diagnosis`：任务编排、状态机、LangGraph 执行桥接。
3. `agent_runtime`：Orchestrator/Specialist 调度、模式路由。
4. `tooling`：内置工具、MCP 适配器、Skill 运行器（进程级隔离）。
5. `approval`：风险分级、审批流、超时处理。
6. `history_replay`：事件存储、重放服务、导出。
7. `admin_config`：LLM/MCP/Skill 配置与版本管理。
8. `observability`：指标、日志、审计。

### 2.2 关键运行链路
1. 用户创建诊断任务（REST）。
2. 任务入队（Redis Stream），状态 `queued -> running`。
3. LangGraph 驱动节点执行；每个节点产出事件写入 `diagnosis_events`。
4. Event Dispatcher 按 session channel 推送 WS 增量。
5. 遇审批点切到 `waiting_user`，超时转人工待办。
6. 完成后落库诊断报告；回放服务可按时间线复盘。

### 2.3 实时通道规范（已冻结）
- Channel 模型：`/ws/sessions/{session_id}`。
- Envelope：`event_id / session_id / type / ts / seq / payload`。
- 传输语义：至少一次投递；前端按 `event_id` 去重。
- 顺序：按 `session_id` 保序。
- 断线恢复：客户端携带 `last_seq`，服务端补偿（24h 窗口）。
- 高频事件合并：`metrics.updated` 500ms 聚合。
- 鉴权：Session。

### 2.4 非功能保障
- 可用性：无状态 API + 可横向扩展 worker。
- 性能：历史查询关键索引优化，目标 `<1s`。
- 安全：敏感字段加密存储 + 回放权限控制。

---

## 3. 数据库设计

### 3.1 PostgreSQL 核心表

#### 3.1.1 用户与权限
- `users(id, username, password_hash, status, created_at, deleted_at)`
- `roles(id, role_key)`：`admin/user/readonly`
- `user_roles(user_id, role_id)`

#### 3.1.2 诊断主表
- `diagnosis_sessions`
  - `id, title, problem_type, description, status`
  - `mode_selected, mode_source(user/router/default)`
  - `created_by, created_at, started_at, ended_at`
  - `token_in_total, token_out_total, estimated_cost`
  - `deleted_at`（软删）

- `agent_runs`
  - `id, session_id, parent_run_id, agent_name, mode`
  - `status, started_at, ended_at`
  - `input_snapshot(jsonb), output_summary(jsonb)`

- `tool_calls`
  - `id, session_id, agent_run_id, tool_name`
  - `risk_level, idempotency_key`
  - `request_payload(jsonb), response_payload(jsonb)`
  - `duration_ms, status, created_at`

- `checkpoints`
  - `id, session_id, agent_run_id, checkpoint_no`
  - `state_blob(jsonb), created_at`

- `approvals`
  - `id, session_id, risk_level, action_name`
  - `status(pending/approved/rejected/timeout)`
  - `reason, approver_id, created_at, decided_at`

- `diagnosis_events`
  - `id(bigserial), event_id(uuid), session_id`
  - `seq(bigint), type, ts`
  - `payload(jsonb)`
  - `trace_id, producer`

- `diagnosis_reports`
  - `id, session_id, root_cause_md, fix_suggestion_md`
  - `confidence_score, related_services(jsonb), code_locations(jsonb)`
  - `export_md_path`

#### 3.1.3 配置域
- `llm_providers`, `llm_models`, `llm_routing_policies`
- `mcp_servers`, `mcp_tools`
- `skills`, `skill_versions`, `skill_files`

#### 3.1.4 知识与历史
- `knowledge_items(id, title, content_md, tags, created_by, created_at)`
- `similar_case_links(session_id, knowledge_item_id, score, strategy)`

#### 3.1.5 审计
- `audit_logs(id, session_id, actor_id, action, object_type, object_id, detail, created_at)`
- 约束：`audit_logs` 不允许业务删除。

### 3.2 索引与分区
- `diagnosis_sessions(status, created_at desc)`
- `diagnosis_sessions(problem_type, created_at desc)`
- `agent_runs(session_id, parent_run_id, status)`
- `tool_calls(session_id, created_at desc)`
- `approvals(session_id, status, created_at desc)`
- `diagnosis_events(session_id, seq)` 唯一索引
- `diagnosis_events(ts)`（可选按月分区）

### 3.3 Redis 使用边界
- `stream:diagnosis`：任务执行队列。
- `stream:events`：事件分发缓冲。
- `cache:session_snapshot:{id}`：首屏快照缓存。
- 不将 Redis 作为事实主存储。

---

## 4. API 接口设计

### 4.1 REST（/api/v1）

#### 4.1.1 诊断任务
- `POST /diagnosis-sessions`
  - Header: `Idempotency-Key`
  - Body: `title, problem_type, description, preferred_mode?`
  - Resp: `session_id, status`

- `GET /diagnosis-sessions/{id}`
  - 返回任务快照、当前状态、统计指标。

- `POST /diagnosis-sessions/{id}/cancel`
- `POST /diagnosis-sessions/{id}/resume`

#### 4.1.2 时间线与重放
- `GET /diagnosis-sessions/{id}/timeline?from_seq=&limit=`
- `GET /diagnosis-sessions/{id}/agent-tree`
- `GET /diagnosis-sessions/{id}/replay`
- `POST /diagnosis-sessions/{id}/replay/export-md`

#### 4.1.3 审批
- `GET /diagnosis-sessions/{id}/pending-approvals`
- `POST /approvals/{approval_id}/approve`
- `POST /approvals/{approval_id}/reject`

#### 4.1.4 管理与配置
- `POST /skills/upload`
- `GET /skills`, `GET /skills/{id}/versions`
- `GET /mcp/servers`, `POST /mcp/servers/test`
- `GET /llm/models`, `POST /llm/models`

#### 4.1.5 历史与知识
- `GET /history?problem_type=&service=&from=&to=&page=&size=`
- `GET /history/{session_id}`
- `POST /knowledge-items`
- `GET /knowledge-items`

### 4.2 WebSocket

#### 4.2.1 连接
- URL: `/ws/sessions/{session_id}?last_seq={n}`
- Auth: Session（cookie）

#### 4.2.2 事件类型（v1）
- `session.created|status_changed|completed|failed|canceled`
- `agent.node_started|node_updated|node_completed|node_failed`
- `tool.call_started|call_completed|call_failed`
- `approval.required|approved|rejected|timeout`
- `timeline.appended`
- `metrics.updated`

#### 4.2.3 示例
```json
{
  "event_id": "5d0f...",
  "session_id": "sess_123",
  "type": "tool.call_completed",
  "ts": "2026-02-10T12:00:00Z",
  "seq": 118,
  "payload": {
    "agent_run_id": "run_9",
    "tool_name": "log_search",
    "duration_ms": 132,
    "risk_level": "R0"
  }
}
```

---

## 5. Agent 工作流设计（LangGraph）

### 5.1 图模型
- 主图：`OrchestratorGraph`
  - 输入解析 -> 模式选择 -> 子任务规划 -> 并行 Specialist -> 证据聚合 -> 根因输出。
- 子图：`SpecialistGraph`
  - ReAct 循环：`reason -> act(tool) -> observe -> decide(next)`。

### 5.2 执行模式规则
- 优先级：用户指定 > 路由矩阵 > default_mode。
- ReAct 降级：连续 `N=3` 无增量切 Plan-Execute。
- Hierarchical 降级：子节点失败率超过 `40%` 切最小可行路径。

### 5.3 状态机
`queued -> running -> waiting_user -> retrying -> completed|failed|canceled`

- `waiting_user`：审批点进入。
- 超时：转人工待办（不自动放行）。
- `retrying`：按重试策略执行。

### 5.4 重试与恢复
- LLM：最多 3 次，指数退避（1s, 2s, 4s）。
- Tool：最多 2 次，线性退避（1s, 2s）。
- Agent 节点：人工触发 1 次重试。
- 恢复粒度：Agent 节点 + tool call 结果复用。
- 写操作工具必须带幂等键。

### 5.5 风险与审批
- R0：自动执行。
- R1：一次确认。
- R2：二次确认。
- R3：强制确认 + 审计。
- R2/R3 全量审计入库。

---

## 6. 历史重放与导出设计

### 6.1 回放能力
- 支持完整事件流重建：状态、调用链、工具调用、审批节点、Token 与成本。
- 支持按 `session_id + seq` 顺序回放。
- 支持过滤：按 Agent 子树、事件类型、时间区间。

### 6.2 导出 Markdown
导出内容模板：
1. 任务元信息
2. 根因结论
3. 关键证据链（调用链 + 工具结果）
4. 审批记录
5. Token 与成本汇总
6. 修复建议

---

## 7. 安全与权限
- 鉴权：Session + RBAC（admin/user/readonly）。
- 敏感数据：数据库加密字段（LLM key、MCP 凭据）。
- 回放权限：readonly 可看结果与时间线；高风险 payload 可按策略屏蔽（后续前端细化）。
- 审计：R2/R3 与回放导出操作均记录。

---

## 8. 可观测性与SLO

### 8.1 MVP 核心指标
- 事件投递成功率
- WebSocket 重连率
- 端到端延迟（事件产生到前端展示）

### 8.2 性能目标
- 简单问题定位：P95 < 30s
- 并发诊断任务：100+
- 历史查询：< 1s

---

## 9. 开发里程碑（建议）
1. M1：数据库与状态机骨架、REST 基础接口。
2. M2：LangGraph 主干 + 工具调用与审批流。
3. M3：WS 事件总线 + 回放服务 + Markdown 导出。
4. M4：稳定性优化、压测、审计与权限收口。

---

## 10. 待后续单独确认（前端专题）
- 画布布局、时间线交互、树视图折叠策略。
- 回放播放器（快进/暂停/跳转）细节。
- 大规模事件渲染与虚拟列表策略。
