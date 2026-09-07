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

## 第三步：微观一点

使用explore-codebase理解/path/to/code这一个codebase

## 第四步：再微观一点

这个codebase都有哪些业务流程？

## 第五步：再再微观一点

`使用list_flows_tool列出按关键度排序的执行流程，列出执行流列表，并使用get_flow_tool获取每个执行流的详情，即从入口到出口的执行流逻辑与调用链，描述的尽可能详细且保持语言简练`

## 第六步：追踪数据流

这个codebase都有什么“核心”数据流？

数据用的什么数据结构处理的？

针对数据结构，有哪些依托于这些数据结构的算法来处理了什么事情？

## 第五步：软件最小生成元