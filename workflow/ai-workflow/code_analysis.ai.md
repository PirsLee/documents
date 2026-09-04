# 重要提醒

## tool:understand-anything

除非真的有必要，否则不使用understand-anything生成代码图谱，这样做太耗费token了，除非以后token自由(:

# AI阅读Codebase的Workflow

## 第一步：code-review-graph

1 配置与启动MCP

```
pip install code-review-graph
code-review-graph install
code-review-graph build

code-review-graph serve

添加.code-review-graphignore文件，以忽略不需要的文件
```

2 在agent中使用MCP工具

使用`/architecture_map`生成架构概览

`使用list_flows_tool列出按关键度排序的执行流程`生成执行流 

`使用get_flow_tool获取单个执行流的详情，从入口点追踪<某个执行流>的调用链`生成执行流调用链

`使用list_communities_tool列出检测到的代码社区`生成代码社区列表

`使用get_community_tool获取单个社区的详情，解释代码社区与业务概念的对应关系`生成社区与概念映射表

`使用get_architecture_overview_tool生成基于社区结构的架构概览`

`使用get_hub_nodes_tool查找连接最多的节点（架构热点）`生成架构热点

`使用get_bridge_nodes_tool查找架构瓶颈`

## 第二步：explore-codebase

理解/path/to/code这一个codebase

**后面要把skills的执行编排起来，不过目前不需要，因为我感觉编排了好没意思**

## 第三步：挖掘业务流程

都有哪些业务流程？

每条业务流程都做了什么？

每条业务流程的概念链条是什么？

每条业务流程的入口在哪里？

每条业务流程涉及哪些控制流？

每条业务流程的节点都有什么行为？

每个业务流程的执行流程（从入口点追踪调用链）？

## 第四步：追踪数据流

举例针对具体场景的完整追踪提示词：

    "Trace the complete data flow for this scenario: A user clicks 'Open' on a tool card in the Hub dashboard. Start from the React onClick handler in Dashboard.jsx. Follow it through the backend, through the tool's auth, until the user lands on the tool's homepage already logged in. List every file touched, every function called, every network request made. Don't skip steps."


## 第五步：软件最小生成元