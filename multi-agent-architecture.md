# 多Agent协作系统架构设计

## 核心概念

```
DiagnosisAgent (主Agent)
    ├── LogAnalysisAgent (子Agent)
    ├── CodeAnalysisAgent (子Agent)
    └── SummaryAgent (子Agent)
```

## 1. Agent基础架构

```python
# core/agent/base_agent.py
from abc import ABC, abstractmethod
from typing import Dict, Any, List, Optional
from dataclasses import dataclass
from enum import Enum

class AgentStatus(Enum):
    PENDING = "pending"
    RUNNING = "running"
    WAITING_USER = "waiting_user"
    COMPLETED = "completed"
    FAILED = "failed"

@dataclass
class AgentResult:
    """Agent执行结果"""
    status: AgentStatus
    data: Dict[str, Any]
    error: Optional[str] = None
    child_results: List['AgentResult'] = None

class BaseAgent(ABC):
    """通用Agent基类"""

    def __init__(self, agent_id: str, parent_id: Optional[str], context: Dict):
        self.agent_id = agent_id
        self.parent_id = parent_id
        self.context = context
        self.children: List[BaseAgent] = []
        self.status = AgentStatus.PENDING

    @abstractmethod
    async def execute(self, input_data: Dict[str, Any]) -> AgentResult:
        """执行Agent任务"""
        pass

    @abstractmethod
    def get_system_prompt(self) -> str:
        """获取系统提示词"""
        pass

    async def spawn_child(self, agent_type: str, input_data: Dict) -> AgentResult:
        """创建并执行子Agent"""
        from core.agent.engine import AgentEngine

        engine = AgentEngine.get_instance()
        child_agent = await engine.create_agent(
            agent_type=agent_type,
            parent_id=self.agent_id,
            context=self.context
        )

        self.children.append(child_agent)
        result = await child_agent.execute(input_data)
        return result

    async def spawn_parallel(self, tasks: List[Dict]) -> List[AgentResult]:
        """并行执行多个子Agent"""
        import asyncio

        child_tasks = [
            self.spawn_child(task['agent_type'], task['input_data'])
            for task in tasks
        ]

        results = await asyncio.gather(*child_tasks, return_exceptions=True)
        return results
```

## 2. 问题定位主Agent实现

```python
# agents/diagnosis/agent.py
class DiagnosisAgent(BaseAgent):
    """问题定位主Agent - 协调多个子Agent"""

    def get_system_prompt(self) -> str:
        return """你是问题定位主Agent，负责协调子Agent完成诊断任务。

可用子Agent：
- log_analysis: 分析日志文件
- code_analysis: 分析代码逻辑
- trace_analysis: 分析调用链路
- summary: 总结分析结果

你的工作流程：
1. 根据问题描述决定需要哪些子Agent
2. 使用spawn_child或spawn_parallel创建子Agent
3. 收集子Agent结果并综合分析
4. 输出最终诊断结论"""

    async def execute(self, input_data: Dict[str, Any]) -> AgentResult:
        problem_desc = input_data['problem_description']

        # 阶段1: 并行执行日志和代码分析
        parallel_tasks = [
            {
                'agent_type': 'log_analysis',
                'input_data': {
                    'problem': problem_desc,
                    'log_paths': input_data.get('log_paths', [])
                }
            },
            {
                'agent_type': 'code_analysis',
                'input_data': {
                    'problem': problem_desc,
                    'code_paths': input_data.get('code_paths', [])
                }
            }
        ]

        results = await self.spawn_parallel(parallel_tasks)
        log_result, code_result = results

        # 阶段2: 基于前面结果执行调用链分析
        trace_result = await self.spawn_child('trace_analysis', {
            'log_findings': log_result.data,
            'code_findings': code_result.data
        })

        # 阶段3: 总结所有结果
        summary_result = await self.spawn_child('summary', {
            'log_analysis': log_result.data,
            'code_analysis': code_result.data,
            'trace_analysis': trace_result.data
        })

        return AgentResult(
            status=AgentStatus.COMPLETED,
            data=summary_result.data,
            child_results=[log_result, code_result, trace_result, summary_result]
        )
```

## 3. 子Agent实现示例

```python
# agents/log_analysis/agent.py
class LogAnalysisAgent(BaseAgent):
    """日志分析子Agent"""

    def get_system_prompt(self) -> str:
        return """你是日志分析专家，负责：
1. 解析日志文件
2. 识别错误和异常
3. 提取关键时间点和事件
4. 返回结构化分析结果"""

    async def execute(self, input_data: Dict[str, Any]) -> AgentResult:
        # 使用工具分析日志
        log_data = await self.analyze_logs(input_data['log_paths'])

        return AgentResult(
            status=AgentStatus.COMPLETED,
            data={
                'errors': log_data['errors'],
                'timeline': log_data['timeline'],
                'suspicious_patterns': log_data['patterns']
            }
        )

# agents/code_analysis/agent.py
class CodeAnalysisAgent(BaseAgent):
    """代码分析子Agent"""

    def get_system_prompt(self) -> str:
        return """你是代码分析专家，负责：
1. 搜索相关代码
2. 分析业务逻辑
3. 识别潜在bug
4. 返回代码位置和问题点"""

    async def execute(self, input_data: Dict[str, Any]) -> AgentResult:
        code_findings = await self.search_and_analyze(input_data['problem'])

        return AgentResult(
            status=AgentStatus.COMPLETED,
            data={
                'relevant_files': code_findings['files'],
                'potential_bugs': code_findings['bugs'],
                'logic_issues': code_findings['issues']
            }
        )

# agents/summary/agent.py
class SummaryAgent(BaseAgent):
    """总结Agent - 聚合所有子Agent结果"""

    def get_system_prompt(self) -> str:
        return """你是总结专家，负责：
1. 综合所有子Agent的分析结果
2. 识别根本原因
3. 提供修复建议
4. 评估置信度"""

    async def execute(self, input_data: Dict[str, Any]) -> AgentResult:
        # 综合分析所有输入
        root_cause = self.identify_root_cause(
            input_data['log_analysis'],
            input_data['code_analysis'],
            input_data['trace_analysis']
        )

        return AgentResult(
            status=AgentStatus.COMPLETED,
            data={
                'root_cause': root_cause,
                'confidence': 0.85,
                'fix_suggestions': [],
                'affected_services': []
            }
        )
```

## 4. Agent引擎（调度器）

```python
# core/agent/engine.py
import asyncio
from typing import Dict, Optional
from uuid import uuid4

class AgentEngine:
    """Agent执行引擎 - 管理Agent生命周期"""

    _instance = None

    def __init__(self):
        self.active_agents: Dict[str, BaseAgent] = {}
        self.execution_tree = ExecutionTree()

    @classmethod
    def get_instance(cls):
        if cls._instance is None:
            cls._instance = cls()
        return cls._instance

    async def create_agent(
        self,
        agent_type: str,
        parent_id: Optional[str],
        context: Dict
    ) -> BaseAgent:
        """创建Agent实例"""
        agent_id = str(uuid4())

        # 从注册表获取Agent类
        agent_class = AgentRegistry.get(agent_type)
        agent = agent_class(agent_id, parent_id, context)

        # 记录到执行树
        self.execution_tree.add_node(agent_id, parent_id, agent_type)
        self.active_agents[agent_id] = agent

        return agent

    async def run_agent(
        self,
        agent_type: str,
        input_data: Dict,
        context: Dict
    ) -> AgentResult:
        """运行顶层Agent"""
        agent = await self.create_agent(agent_type, None, context)

        try:
            result = await agent.execute(input_data)
            return result
        except Exception as e:
            return AgentResult(
                status=AgentStatus.FAILED,
                data={},
                error=str(e)
            )

    def get_execution_tree(self) -> Dict:
        """获取完整执行树（用于可视化）"""
        return self.execution_tree.to_dict()
```

## 5. 执行树管理

```python
# core/agent/execution_tree.py
from dataclasses import dataclass
from typing import Dict, List, Optional

@dataclass
class TreeNode:
    agent_id: str
    agent_type: str
    parent_id: Optional[str]
    children: List[str]
    status: AgentStatus
    start_time: float
    end_time: Optional[float]
    result: Optional[Dict]

class ExecutionTree:
    """管理Agent执行树结构"""

    def __init__(self):
        self.nodes: Dict[str, TreeNode] = {}
        self.root_id: Optional[str] = None

    def add_node(self, agent_id: str, parent_id: Optional[str], agent_type: str):
        """添加节点"""
        import time

        node = TreeNode(
            agent_id=agent_id,
            agent_type=agent_type,
            parent_id=parent_id,
            children=[],
            status=AgentStatus.PENDING,
            start_time=time.time(),
            end_time=None,
            result=None
        )

        self.nodes[agent_id] = node

        if parent_id is None:
            self.root_id = agent_id
        else:
            self.nodes[parent_id].children.append(agent_id)

    def to_dict(self) -> Dict:
        """转换为字典（用于前端可视化）"""
        def build_tree(node_id: str) -> Dict:
            node = self.nodes[node_id]
            return {
                'id': node.agent_id,
                'type': node.agent_type,
                'status': node.status.value,
                'duration': node.end_time - node.start_time if node.end_time else None,
                'children': [build_tree(child_id) for child_id in node.children]
            }

        return build_tree(self.root_id) if self.root_id else {}
```

## 6. Agent注册表

```python
# core/agent/registry.py
from typing import Dict, Type

class AgentRegistry:
    """Agent注册中心"""
    _agents: Dict[str, Type[BaseAgent]] = {}

    @classmethod
    def register(cls, agent_type: str):
        def decorator(agent_class):
            cls._agents[agent_type] = agent_class
            return agent_class
        return decorator

    @classmethod
    def get(cls, agent_type: str) -> Type[BaseAgent]:
        if agent_type not in cls._agents:
            raise ValueError(f"Unknown agent: {agent_type}")
        return cls._agents[agent_type]

# 注册所有Agent
@AgentRegistry.register('diagnosis')
class DiagnosisAgent(BaseAgent):
    pass

@AgentRegistry.register('log_analysis')
class LogAnalysisAgent(BaseAgent):
    pass

@AgentRegistry.register('code_analysis')
class CodeAnalysisAgent(BaseAgent):
    pass

@AgentRegistry.register('trace_analysis')
class TraceAnalysisAgent(BaseAgent):
    pass

@AgentRegistry.register('summary')
class SummaryAgent(BaseAgent):
    pass
```

## 7. API接口

```python
# api/routes/agent.py
from fastapi import APIRouter, WebSocket
from core.agent.engine import AgentEngine

router = APIRouter()

@router.post("/diagnosis/run")
async def run_diagnosis(request: DiagnosisRequest):
    """运行问题定位"""
    engine = AgentEngine.get_instance()

    result = await engine.run_agent(
        agent_type='diagnosis',
        input_data={
            'problem_description': request.problem,
            'log_paths': request.log_paths,
            'code_paths': request.code_paths
        },
        context={
            'user_id': request.user_id,
            'session_id': request.session_id
        }
    )

    return {
        'result': result.data,
        'execution_tree': engine.get_execution_tree()
    }

@router.websocket("/diagnosis/stream")
async def stream_diagnosis(websocket: WebSocket):
    """实时推送Agent执行状态"""
    await websocket.accept()

    # 订阅执行树更新
    async for update in engine.subscribe_updates():
        await websocket.send_json(update)
```

## 8. 前端可视化数据结构

```json
{
  "execution_tree": {
    "id": "agent-001",
    "type": "diagnosis",
    "status": "completed",
    "duration": 15.3,
    "children": [
      {
        "id": "agent-002",
        "type": "log_analysis",
        "status": "completed",
        "duration": 5.2,
        "children": []
      },
      {
        "id": "agent-003",
        "type": "code_analysis",
        "status": "completed",
        "duration": 4.8,
        "children": []
      },
      {
        "id": "agent-004",
        "type": "trace_analysis",
        "status": "completed",
        "duration": 3.1,
        "children": []
      },
      {
        "id": "agent-005",
        "type": "summary",
        "status": "completed",
        "duration": 2.2,
        "children": []
      }
    ]
  }
}
```

## 关键设计点

### 1. Agent通信模式
- **父→子**: 通过`spawn_child`传递输入数据
- **子→父**: 通过`AgentResult`返回结果
- **兄弟Agent**: 不直接通信，由父Agent协调

### 2. 执行模式
- **串行**: `await spawn_child()` 逐个执行
- **并行**: `await spawn_parallel()` 同时执行多个子Agent
- **混合**: 先并行后串行（如示例中的阶段1→阶段2）

### 3. 状态管理
- 每个Agent维护自己的状态
- ExecutionTree统一管理全局执行状态
- 支持实时查询和可视化

### 4. 错误处理
- 子Agent失败不影响其他子Agent
- 父Agent决定如何处理子Agent失败
- 支持重试和降级策略

## 目录结构

```
mul-agents/
├── core/
│   └── agent/
│       ├── base_agent.py          # Agent基类
│       ├── engine.py              # Agent引擎
│       ├── registry.py            # Agent注册表
│       └── execution_tree.py      # 执行树管理
├── agents/
│   ├── diagnosis/
│   │   └── agent.py               # 主Agent
│   ├── log_analysis/
│   │   └── agent.py               # 日志分析子Agent
│   ├── code_analysis/
│   │   └── agent.py               # 代码分析子Agent
│   ├── trace_analysis/
│   │   └── agent.py               # 调用链分析子Agent
│   └── summary/
│       └── agent.py               # 总结子Agent
└── api/
    └── routes/
        └── agent.py               # API接口
```
