# Agent 框架架构设计

## 1. 架构概览

```
用户请求
    ↓
┌─────────────────────────────────────────┐
│      Central Agent (中心代理)            │
│  - 接收用户输入                          │
│  - 协调整体流程                          │
│  - 管理用户交互                          │
└─────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────┐
│      Plan Agent (规划代理)               │
│  - 分析问题类型                          │
│  - 制定诊断计划                          │
│  - 确定执行步骤                          │
└─────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────┐
│   Execution Agents (执行代理层)          │
│  ┌─────────────┐  ┌─────────────┐      │
│  │ Log Agent   │  │ Code Agent  │      │
│  │ 日志分析    │  │ 代码分析    │      │
│  └─────────────┘  └─────────────┘      │
│  ┌─────────────┐  ┌─────────────┐      │
│  │ Trace Agent │  │ Root Cause  │      │
│  │ 链路追踪    │  │ 根因分析    │      │
│  └─────────────┘  └─────────────┘      │
└─────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────┐
│      Tool Layer (工具层)                 │
│  - 内置工具                              │
│  - MCP 扩展                              │
│  - Skill 扩展                            │
└─────────────────────────────────────────┘
```

## 2. Agent 类型定义

### 2.1 Central Agent (中心代理)

**职责**：
- 接收用户问题描述
- 创建诊断会话
- 调度 Plan Agent 和 Execution Agents
- 管理用户交互确认（bash命令、流程阶段）
- 汇总最终结果
- 处理失败和重试

**输入**：
- 问题描述
- 用户上下文（时间、用户ID、相关服务等）

**输出**：
- 诊断结果
- 根因分析
- 修复建议
- 完整的执行树

**关键能力**：
- 会话管理
- 状态持久化（支持失败恢复）
- 用户交互控制
- 超时和重试管理

---

### 2.2 Plan Agent (规划代理)

**职责**：
- 分析问题类型（业务逻辑异常/系统报错/性能问题/数据不一致）
- 制定诊断计划（需要哪些步骤、顺序、依赖关系）
- 确定需要调用的 Execution Agents
- 评估所需资源（日志、代码、监控数据）

**输入**：
- 问题描述
- 业务规范（从Skill获取）
- 历史相似问题

**输出**：
- 诊断计划（JSON格式）
  ```json
  {
    "problem_type": "business_logic_error",
    "steps": [
      {
        "agent": "LogAgent",
        "task": "收集订单服务日志",
        "params": {"service": "order", "time_range": "..."}
      },
      {
        "agent": "TraceAgent",
        "task": "分析调用链路",
        "depends_on": ["step_1"]
      }
    ]
  }
  ```

**关键能力**：
- 问题分类
- 计划生成
- 依赖关系管理

---

### 2.3 Execution Agents (执行代理)

#### 2.3.1 Log Agent (日志分析代理)

**职责**：
- 收集相关服务日志
- 解析日志格式
- 识别错误和异常
- 提取关键信息（错误码、堆栈、时间戳）

**工具**：
- 日志搜索工具
- 日志解析工具
- 时间范围过滤

**输出**：
- 错误日志列表
- 异常模式
- 时间线分析

---

#### 2.3.2 Trace Agent (链路追踪代理)

**职责**：
- 分析服务调用链路
- 识别调用失败点
- 计算各服务响应时间
- 定位性能瓶颈

**工具**：
- 日志关联分析（通过trace_id）
- 调用链重建
- 性能分析

**输出**：
- 调用链路图
- 失败节点
- 性能瓶颈

---

#### 2.3.3 Code Agent (代码分析代理)

**职责**：
- 搜索相关代码
- 分析代码逻辑
- 对比业务规范
- 识别潜在bug

**工具**：
- 代码搜索
- 代码读取
- 静态分析

**输出**：
- 相关代码位置
- 逻辑分析
- 潜在问题点

---

#### 2.3.4 Root Cause Agent (根因分析代理)

**职责**：
- 综合所有信息
- 推断问题根因
- 生成修复建议
- 计算置信度

**输入**：
- Log Agent 输出
- Trace Agent 输出
- Code Agent 输出
- 业务规范

**输出**：
- 根因分析
- 置信度评分
- 修复建议
- 相似历史问题

---

## 3. Agent 通信协议

### 3.1 消息格式

```python
class AgentMessage:
    agent_id: str          # Agent唯一标识
    parent_id: str         # 父Agent ID
    task_type: str         # 任务类型
    input_data: dict       # 输入数据
    output_data: dict      # 输出数据
    status: str            # pending/running/completed/failed
    start_time: datetime
    end_time: datetime
    tokens_used: dict      # {input: 100, output: 200}
    cost: float
```

### 3.2 状态管理

```python
class AgentState:
    PENDING = "pending"       # 等待执行
    RUNNING = "running"       # 执行中
    WAITING_USER = "waiting_user"  # 等待用户确认
    COMPLETED = "completed"   # 已完成
    FAILED = "failed"         # 失败
    RETRYING = "retrying"     # 重试中
```

---

## 4. 执行流程示例

### 场景：订单提交失败

```
1. Central Agent 接收问题
   输入: "用户A在2026-02-10 10:30提交订单失败"

2. Central Agent → Plan Agent
   Plan Agent 分析:
   - 问题类型: 业务逻辑异常
   - 计划: 日志分析 → 链路追踪 → 代码分析 → 根因定位

3. Central Agent → Log Agent
   Log Agent 执行:
   - 收集订单服务日志 (10:29-10:31)
   - 发现: "库存服务返回 stock_insufficient"

4. Central Agent → Trace Agent
   Trace Agent 执行:
   - 重建调用链: 订单服务 → 库存服务
   - 发现: 库存服务返回500错误

5. Central Agent → Code Agent
   Code Agent 执行:
   - 搜索库存扣减代码
   - 分析: 发现并发扣减逻辑缺少锁

6. Central Agent → Root Cause Agent
   Root Cause Agent 综合分析:
   - 根因: 库存扣减未加锁，并发场景下超卖
   - 置信度: 85%
   - 建议: 添加分布式锁或数据库乐观锁

7. Central Agent 返回结果给用户
```

---

## 5. 用户交互确认点

### 5.1 Bash 命令确认

```python
class BashCommandType:
    READ = "read"          # 读操作: ls, cat, grep
    WRITE = "write"        # 写操作: echo >, cp, mv
    SYSTEM = "system"      # 系统命令: ps, netstat, systemctl
    DANGEROUS = "dangerous" # 危险操作: rm, kill, shutdown
```

**确认策略**：
- READ: 自动执行（可配置）
- WRITE: 询问用户
- SYSTEM: 询问用户
- DANGEROUS: 必须确认

### 5.2 流程阶段确认

**确认点**：
- Plan Agent 完成计划后
- 每个 Execution Agent 完成后
- Root Cause Agent 给出结论前

**用户选项**：
- 继续执行
- 修改计划
- 跳过当前步骤
- 终止诊断

---

## 6. 故障处理机制

### 6.1 重试策略

```python
class RetryPolicy:
    max_retries: int = 3
    retry_delay: int = 5  # 秒
    exponential_backoff: bool = True

    # 可重试的错误类型
    retryable_errors = [
        "LLMTimeout",
        "LLMRateLimitError",
        "ToolExecutionTimeout",
        "NetworkError"
    ]
```

### 6.2 超时控制

```python
class TimeoutConfig:
    tool_call_timeout: int = 30      # 单个工具调用
    agent_execution_timeout: int = 300  # 单个Agent执行
    total_diagnosis_timeout: int = 1800  # 整体诊断
```

### 6.3 失败恢复

- 保存每个Agent的执行状态
- 失败时可从最后成功的Agent继续
- 用户可选择重试失败的Agent或跳过

---

## 7. 数据流

```
用户输入
  ↓
Central Agent (创建会话)
  ↓
Plan Agent (生成计划) → 保存到 diagnosis_records.agent_tree
  ↓
Execution Agents (并行/串行执行)
  ├→ Log Agent → 保存到 agent_executions
  ├→ Trace Agent → 保存到 agent_executions
  └→ Code Agent → 保存到 agent_executions
  ↓
Root Cause Agent (综合分析)
  ↓
Central Agent (汇总结果)
  ↓
返回用户 + 保存历史记录
```

---

## 8. 可视化支持

### 8.1 层级导航树

```json
{
  "root": {
    "agent_id": "central_001",
    "type": "CentralAgent",
    "status": "completed",
    "children": [
      {
        "agent_id": "plan_001",
        "type": "PlanAgent",
        "status": "completed"
      },
      {
        "agent_id": "log_001",
        "type": "LogAgent",
        "status": "completed",
        "duration": 5.2
      },
      {
        "agent_id": "trace_001",
        "type": "TraceAgent",
        "status": "completed",
        "duration": 8.1
      }
    ]
  }
}
```

### 8.2 时间线视图

```json
{
  "timeline": [
    {
      "timestamp": "2026-02-10T10:30:00Z",
      "event": "user_input",
      "data": "订单提交失败"
    },
    {
      "timestamp": "2026-02-10T10:30:01Z",
      "event": "agent_start",
      "agent": "CentralAgent"
    },
    {
      "timestamp": "2026-02-10T10:30:02Z",
      "event": "llm_call",
      "agent": "PlanAgent",
      "tokens": {"input": 150, "output": 80}
    },
    {
      "timestamp": "2026-02-10T10:30:05Z",
      "event": "tool_call",
      "agent": "LogAgent",
      "tool": "log_search",
      "duration": 2.1
    }
  ]
}
```

---

## 9. 扩展性设计

### 9.1 添加新的 Execution Agent

```python
class CustomAgent(BaseExecutionAgent):
    def __init__(self):
        super().__init__(
            agent_type="CustomAgent",
            description="自定义Agent描述"
        )

    def execute(self, input_data: dict) -> dict:
        # 实现具体逻辑
        pass

    def get_required_tools(self) -> list:
        return ["tool1", "tool2"]
```

### 9.2 Agent 注册机制

```python
class AgentRegistry:
    agents = {}

    @classmethod
    def register(cls, agent_type: str, agent_class):
        cls.agents[agent_type] = agent_class

    @classmethod
    def create_agent(cls, agent_type: str):
        return cls.agents[agent_type]()
```

---

## 10. 实现优先级

### Phase 1: 核心框架
- ✅ Central Agent 基础实现
- ✅ Agent 消息协议
- ✅ 状态管理
- ✅ 基础工具集成

### Phase 2: 规划能力
- ✅ Plan Agent 实现
- ✅ 问题分类
- ✅ 计划生成

### Phase 3: 执行代理
- ✅ Log Agent
- ✅ Code Agent
- ✅ Root Cause Agent
- ⏸️ Trace Agent (P1)

### Phase 4: 高级功能
- ✅ 用户交互确认
- ✅ 故障处理和重试
- ✅ 可视化数据生成
- ⏸️ Agent 扩展机制 (P1)
