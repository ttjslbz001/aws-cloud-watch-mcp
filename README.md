# AWS CloudWatch MCP

使用AI自动分析AWS CloudWatch日志，快速识别和解决API Gateway与Lambda集成中的错误。

## 项目概述

AWS CloudWatch 日志分析工具用于分析 Lambda 函数与 API Gateway 集成的日志，帮助用户快速定位和解决问题。通过提供API Gateway ARN，自动分析相关Lambda函数的日志，识别错误模式，并提供改进建议。

## 分析流程

以下序列图展示了完整的分析流程：

```mermaid
sequenceDiagram
    participant Client
    participant SSE Server
    participant AWS CLI
    participant API Gateway
    participant CloudWatch Logs

    Client->>SSE Server: 连接 /api/analyze/stream
    SSE Server-->>Client: 建立SSE连接
    
    Client->>SSE Server: POST /api/analyze
        Note right of Client: 提供API Gateway ARN, 路径, 方法, 时间范围
    
    SSE Server->>AWS CLI: 解析API Gateway ARN获取rest-api-id
    SSE Server->>AWS CLI: 获取resource-id
    AWS CLI->>API Gateway: 查询资源ID
    API Gateway-->>AWS CLI: 返回resource-id
    
    SSE Server->>AWS CLI: 获取集成信息
    AWS CLI->>API Gateway: 获取集成配置
    API Gateway-->>AWS CLI: 返回Lambda函数URI
    
    SSE Server->>Client: 事件: analysis_started
    
    SSE Server->>AWS CLI: 查询Lambda请求日志
    AWS CLI->>CloudWatch Logs: 过滤请求日志
    CloudWatch Logs-->>AWS CLI: 返回请求日志
    
    SSE Server->>AWS CLI: 查询Lambda响应日志
    AWS CLI->>CloudWatch Logs: 过滤响应日志
    CloudWatch Logs-->>AWS CLI: 返回响应日志
    
    SSE Server->>Client: 事件: logs_collected
    
    SSE Server->>SSE Server: 分析错误类型
    SSE Server->>Client: 事件: error_types
    
    SSE Server->>SSE Server: 统计错误分布
    SSE Server->>Client: 事件: error_stats
    
    SSE Server->>SSE Server: 生成改进建议
    SSE Server->>Client: 事件: suggestions
    
    SSE Server->>Client: 事件: analysis_completed
```

## 快速开始

请参考 [API文档](./api.md) 了解如何使用该服务。
