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

## 第二步：宏观一点

使用`/architecture_map`生成架构概览

`使用list_communities_tool列出检测到的代码社区`生成代码社区列表

`使用get_community_tool获取单个社区的详情，解释代码社区`

`使用get_architecture_overview_tool生成基于社区结构的架构概览`

`使用get_hub_nodes_tool查找连接最多的节点（架构热点）`生成架构热点

`使用get_bridge_nodes_tool查找架构瓶颈`

## 第三步：微观一点

使用explore-codebase理解/path/to/code这一个codebase

## 第四步：再微观一点

这个codebase都有哪些业务流程？

业务流程对应哪些执行流程？

## 第五步：再再微观一点

`使用list_flows_tool列出按关键度排序的执行流程`生成执行流 

`使用get_flow_tool获取单个执行流的详情，从入口点追踪<某个执行流>的调用链`生成执行流调用链

`这条执行流的从入口到出口的执行逻辑是什么？`

## 第六步：追踪数据流

这个codebase都有什么数据流？

数据用的什么数据结构处理的？

针对数据结构，有哪些依托于这些数据结构的算法来处理了什么事情？

## 第五步：软件最小生成元