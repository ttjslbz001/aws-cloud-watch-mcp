# API 文档

本文档描述了AWS CloudWatch MCP服务提供的API接口和SSE支持的工具。

## API 端点

### 1. 收集Lambda资源信息API

通过API Gateway ARN获取相关Lambda函数和日志组信息。

**请求：**
```
POST /api/collect-resources
```

**请求参数：**
```json
{
  "apiGatewayArn": "arn:aws:apigateway:us-east-1::/restapis/v15timd4qh",
  "resourcePath": "/v1/push-notification",
  "httpMethod": "POST"
}
```

**响应：**
```json
{
  "lambdaArn": "arn:aws:lambda:us-east-1:123456789012:function:service-notification-pushNotificationHandlerD04422-CtvQYfIoHueP",
  "lambdaName": "service-notification-pushNotificationHandlerD04422-CtvQYfIoHueP",
  "logGroupName": "/aws/lambda/service-notification-pushNotificationHandlerD04422-CtvQYfIoHueP"
}
```

### 2. 列出所有请求API

获取指定时间范围内Lambda函数处理的所有请求-响应详情，支持分页。

**请求：**
```
GET /api/list-requests
```

**请求参数：**
```json
{
  "logGroupName": "/aws/lambda/service-notification-pushNotificationHandlerD04422-CtvQYfIoHueP",
  "timeRange": {
    "startTime": "2023-04-10T00:00:00Z",
    "endTime": "2023-04-11T00:00:00Z"
  },
  "page": 1,
  "pageSize": 20,
  "filter": {
    "statusCode": 400,              // 可选，按状态码筛选
    "errorType": "device_not_enabled", // 可选，按错误类型筛选
    "user": "user@example.com"      // 可选，按用户筛选
  }
}
```

**响应：**
```json
{
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 50,
    "totalPages": 3
  },
  "requests": [
    {
      "requestId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "timestamp": "2023-04-10T10:15:30Z",
      "request": {
        "method": "POST",
        "path": "/v1/push-notification",
        "pathParameters": {},
        "queryParameters": {
          "debug": "true"
        },
        "headers": {
          "Content-Type": "application/json",
          "Authorization": "Bearer [REDACTED]"
        },
        "body": {
          "recipients": ["user@example.com"],
          "notification": {
            "message": {
              "title": "Test Notification",
              "body": "This is a test notification"
            }
          }
        }
      },
      "response": {
        "statusCode": 400,
        "headers": {
          "Content-Type": "application/json"
        },
        "body": {
          "status": "error",
          "message": "Not found any enabled device to send notification for user [user@example.com]",
          "error_code": "invalid_request",
          "statusCode": 400
        }
      },
      "timing": {
        "requestTime": "2023-04-10T10:15:30.123Z",
        "responseTime": "2023-04-10T10:15:30.456Z",
        "duration": 333  // 毫秒
      },
      "additionalInfo": {
        "errorType": "device_not_enabled",
        "lambdaMemoryUsage": "128 MB",
        "coldStart": false
      }
    }
  ],
  "summary": {
    "totalRequests": 50,
    "successCount": 8,
    "errorCount": 42,
    "averageDuration": 310,
    "statusCodeDistribution": {
      "200": 8,
      "400": 42
    }
  }
}
```

### 3. 错误请求日志收集API

通过请求ID获取与特定请求相关的所有日志信息。

**请求：**
```
GET /api/request-logs/{requestId}
```

**请求参数：**
- `requestId`: 路径参数，请求的唯一标识符

**响应：**
```json
{
  "requestId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "lambdaName": "service-notification-pushNotificationHandlerD04422-CtvQYfIoHueP",
  "logs": [
    {
      "timestamp": "2023-04-10T10:15:30.100Z",
      "level": "INFO",
      "message": "Starting event handling with event: {\"body\":{\"recipients\":[\"user@example.com\"],\"notification\":{...}}}"
    },
    {
      "timestamp": "2023-04-10T10:15:30.150Z",
      "level": "INFO",
      "message": "Validating recipients: [\"user@example.com\"]"
    },
    {
      "timestamp": "2023-04-10T10:15:30.200Z",
      "level": "INFO",
      "message": "Looking up enabled devices for user@example.com"
    },
    {
      "timestamp": "2023-04-10T10:15:30.300Z",
      "level": "ERROR",
      "message": "No enabled devices found for user@example.com"
    },
    {
      "timestamp": "2023-04-10T10:15:30.350Z",
      "level": "ERROR",
      "message": "Unable to send notification: Not found any enabled device to send notification for user [user@example.com]"
    },
    {
      "timestamp": "2023-04-10T10:15:30.400Z",
      "level": "INFO",
      "message": "Returning response: {\"status\":\"error\",\"message\":\"Not found any enabled device to send notification for user [user@example.com]\",\"error_code\":\"invalid_request\",\"statusCode\":400}"
    }
  ],
  "requestDetails": {
    "method": "POST",
    "path": "/v1/push-notification",
    "body": {
      "recipients": ["user@example.com"],
      "notification": {
        "message": {
          "title": "Test Notification",
          "body": "This is a test notification"
        }
      }
    }
  },
  "responseDetails": {
    "statusCode": 400,
    "body": {
      "status": "error",
      "message": "Not found any enabled device to send notification for user [user@example.com]",
      "error_code": "invalid_request",
      "statusCode": 400
    }
  },
  "timing": {
    "duration": 333,  // 毫秒
    "startTime": "2023-04-10T10:15:30.123Z",
    "endTime": "2023-04-10T10:15:30.456Z"
  }
}
```

### 4. 日志分析与建议API

分析错误日志并提供改进建议。

**请求：**
```
POST /api/analyze-errors
```

**请求参数：**
```json
{
  "logGroupName": "/aws/lambda/service-notification-pushNotificationHandlerD04422-CtvQYfIoHueP",
  "timeRange": {
    "hours": 24
  },
  "analysisDepth": "detailed" // 可选，分析深度：basic, standard, detailed
}
```

**响应：**
```json
{
  "analysisId": "f8d7e6c5-b4a3-42d1-9e8f-0a1b2c3d4e5f",
  "status": "completed",
  "summary": {
    "totalRequests": 50,
    "successRate": 16,
    "errorRate": 84,
    "topErrors": [
      {
        "type": "device_not_enabled",
        "count": 35,
        "percentage": 83
      },
      {
        "type": "validation_error",
        "count": 7,
        "percentage": 17
      }
    ]
  },
  "recommendations": [
    {
      "priority": "high",
      "area": "device_registration",
      "issue": "设备未启用错误占总错误的83%",
      "recommendation": "在用户注册时确保设备正确注册和启用",
      "implementationSuggestions": [
        "添加设备状态检查API",
        "在注册流程中添加设备验证步骤",
        "提供设备状态自助检查工具"
      ],
      "expectedImpact": "可减少83%的错误"
    },
    {
      "priority": "medium",
      "area": "validation",
      "issue": "参数验证错误占总错误的17%",
      "recommendation": "增强客户端参数验证",
      "implementationSuggestions": [
        "在客户端实现表单验证",
        "为API提供清晰的参数验证文档",
        "考虑使用请求验证中间件"
      ],
      "expectedImpact": "可减少17%的错误"
    }
  ],
  "streamUrl": "/api/analysis/stream?id=f8d7e6c5-b4a3-42d1-9e8f-0a1b2c3d4e5f"
}
```

## SSE 流式结果接口

以下是基于SSE的流式接口，用于实时获取分析结果：

### 1. 日志分类流

```
GET /api/logs/stream?id={classificationId}
```

### 2. 错误上下文流

```
GET /api/errors/stream?id={collectionId}
```

### 3. 分析结果流

```
GET /api/analysis/stream?id={analysisId}
```

## SSE 支持的工具

SSE (Server-Sent Events) 服务器提供以下分析工具，通过事件流式传输向客户端实时推送分析结果：

### 1. 资源收集工具

| 工具名称 | 功能描述 | 事件类型 |
|---------|---------|----------|
| `getApiGatewayResources` | 解析API Gateway资源信息 | `resources_identified` |
| `getLambdaIntegration` | 获取Lambda函数集成信息 | `lambda_identified` |
| `getLogGroupInfo` | 获取日志组信息 | `log_group_identified` |

### 2. 日志分类工具

| 工具名称 | 功能描述 | 事件类型 |
|---------|---------|----------|
| `classifyLogs` | 将日志分类为请求和响应 | `logs_classified` |
| `identifyLogPatterns` | 识别日志中的模式 | `patterns_identified` |
| `categorizeCodes` | 分类状态码和错误码 | `codes_categorized` |

### 3. 错误上下文工具

| 工具名称 | 功能描述 | 事件类型 |
|---------|---------|----------|
| `correlateRequestResponse` | 关联请求和响应日志 | `request_response_correlated` |
| `extractErrorContext` | 提取错误上下文和相关日志 | `error_context_extracted` |
| `groupSimilarErrors` | 对相似错误进行分组 | `similar_errors_grouped` |

### 4. 分析与建议工具

| 工具名称 | 功能描述 | 事件类型 |
|---------|---------|----------|
| `analyzeErrorDistribution` | 分析错误分布 | `error_distribution_analyzed` |
| `identifyRootCauses` | 识别根本原因 | `root_causes_identified` |
| `generateRecommendations` | 生成改进建议 | `recommendations_generated` |
| `prioritizeIssues` | 按影响优先级排序问题 | `issues_prioritized` |

## 事件类型

SSE服务器将发送以下类型的事件：

### 资源收集事件

```json
{
  "event": "resources_identified",
  "data": {
    "apiGatewayId": "v15timd4qh",
    "resourcePath": "/v1/push-notification",
    "resourceId": "abc123",
    "httpMethod": "POST"
  }
}
```

```json
{
  "event": "lambda_identified",
  "data": {
    "lambdaName": "service-notification-pushNotificationHandlerD04422-CtvQYfIoHueP",
    "lambdaArn": "arn:aws:lambda:us-east-1:123456789012:function:service-notification-pushNotificationHandlerD04422-CtvQYfIoHueP"
  }
}
```

```json
{
  "event": "log_group_identified",
  "data": {
    "logGroupName": "/aws/lambda/service-notification-pushNotificationHandlerD04422-CtvQYfIoHueP",
    "retentionInDays": 14,
    "sizeInBytes": 1234567
  }
}
```

### 日志分类事件

```json
{
  "event": "logs_classified",
  "data": {
    "totalLogs": 120,
    "requestLogs": 50,
    "responseLogs": 70
  }
}
```

```json
{
  "event": "patterns_identified",
  "data": {
    "requestPatterns": [
      {
        "pattern": "Starting event handling",
        "count": 50
      }
    ],
    "responsePatterns": [
      {
        "pattern": "statusCode.*200",
        "count": 12
      },
      {
        "pattern": "statusCode.*400",
        "count": 42
      },
      {
        "pattern": "statusCode.*500",
        "count": 16
      }
    ]
  }
}
```

```json
{
  "event": "codes_categorized",
  "data": {
    "statusCodes": {
      "200": 12,
      "400": 42,
      "500": 16
    },
    "errorCodes": {
      "invalid_request": 42,
      "internal_error": 16
    }
  }
}
```

### 错误上下文事件

```json
{
  "event": "request_response_correlated",
  "data": {
    "correlatedPairs": 50,
    "uncorrelatedRequests": 0,
    "uncorrelatedResponses": 0
  }
}
```

```json
{
  "event": "error_context_extracted",
  "data": {
    "errorContexts": 42,
    "uniqueErrorMessages": 3,
    "processingProgress": "80%"
  }
}
```

```json
{
  "event": "similar_errors_grouped",
  "data": {
    "groups": [
      {
        "type": "device_not_enabled",
        "count": 35,
        "sampleMessage": "Not found any enabled device to send notification for user [xxx@xxx.com]"
      },
      {
        "type": "validation_error",
        "count": 7,
        "sampleMessage": "recipients[0] is a required field"
      }
    ]
  }
}
```

### 分析与建议事件

```json
{
  "event": "error_distribution_analyzed",
  "data": {
    "totalRequests": 50,
    "successRate": 16,
    "errorRate": 84,
    "errorBreakdown": [
      {
        "type": "device_not_enabled",
        "count": 35,
        "percentage": 83
      },
      {
        "type": "validation_error",
        "count": 7,
        "percentage": 17
      }
    ]
  }
}
```

```json
{
  "event": "root_causes_identified",
  "data": {
    "rootCauses": [
      {
        "errorType": "device_not_enabled",
        "cause": "用户注册后未完成设备注册流程",
        "evidence": "所有错误均出现在新用户的首次通知请求中"
      },
      {
        "errorType": "validation_error",
        "cause": "客户端未进行参数验证",
        "evidence": "多个字段缺失，包括必填字段recipients和notification"
      }
    ]
  }
}
```

```json
{
  "event": "recommendations_generated",
  "data": {
    "recommendations": [
      {
        "priority": "high",
        "area": "device_registration",
        "issue": "设备未启用错误占总错误的83%",
        "recommendation": "在用户注册时确保设备正确注册和启用",
        "implementationSuggestions": [
          "添加设备状态检查API",
          "在注册流程中添加设备验证步骤",
          "提供设备状态自助检查工具"
        ],
        "expectedImpact": "可减少83%的错误"
      },
      {
        "priority": "medium",
        "area": "validation",
        "issue": "参数验证错误占总错误的17%",
        "recommendation": "增强客户端参数验证",
        "implementationSuggestions": [
          "在客户端实现表单验证",
          "为API提供清晰的参数验证文档",
          "考虑使用请求验证中间件"
        ],
        "expectedImpact": "可减少17%的错误"
      }
    ]
  }
}
```

```json
{
  "event": "issues_prioritized",
  "data": {
    "priorityOrder": [
      {
        "issue": "设备未启用错误",
        "priority": "high",
        "impact": "影响83%的失败请求"
      },
      {
        "issue": "参数验证错误",
        "priority": "medium",
        "impact": "影响17%的失败请求"
      }
    ]
  }
}
```

```json
{
  "event": "analysis_completed",
  "data": {
    "duration": "3.2s",
    "summary": "总请求数50个，成功率16%，主要错误为设备未启用(83%)和参数验证失败(17%)",
    "topRecommendation": "在用户注册时确保设备正确注册和启用"
  }
}
```

## 客户端使用示例

```javascript
// 示例1: 获取Lambda资源信息
async function collectResources() {
  const response = await fetch('/api/collect-resources', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      apiGatewayArn: "arn:aws:apigateway:us-east-1::/restapis/v15timd4qh",
      resourcePath: "/v1/push-notification",
      httpMethod: "POST"
    })
  });
  
  return await response.json();
}

// 示例2: 请求响应分类
async function classifyLogs(logGroupName) {
  const response = await fetch('/api/classify-logs', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      logGroupName: logGroupName,
      timeRange: { hours: 24 },
      maxResults: 100
    })
  });
  
  const result = await response.json();
  return result;
}

// 示例3: 收集错误上下文
async function collectErrorContext(logGroupName, errorType) {
  const response = await fetch('/api/collect-error-context', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      logGroupName: logGroupName,
      timeRange: { hours: 24 },
      errorType: errorType,
      maxResults: 50
    })
  });
  
  return await response.json();
}

// 示例4: 分析错误并获取建议
async function analyzeErrors(logGroupName) {
  const response = await fetch('/api/analyze-errors', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      logGroupName: logGroupName,
      timeRange: { hours: 24 },
      analysisDepth: "detailed"
    })
  });
  
  return await response.json();
}

// 示例5: 连接到分析SSE流
function connectToAnalysisStream(streamUrl) {
  const eventSource = new EventSource(streamUrl);
  
  // 资源收集事件
  eventSource.addEventListener('resources_identified', (event) => {
    const data = JSON.parse(event.data);
    console.log('API Gateway资源:', data);
  });
  
  eventSource.addEventListener('lambda_identified', (event) => {
    const data = JSON.parse(event.data);
    console.log('Lambda函数:', data.lambdaName);
  });
  
  // 日志分类事件
  eventSource.addEventListener('logs_classified', (event) => {
    const data = JSON.parse(event.data);
    console.log(`总日志: ${data.totalLogs}, 请求日志: ${data.requestLogs}, 响应日志: ${data.responseLogs}`);
  });
  
  // 错误上下文事件
  eventSource.addEventListener('similar_errors_grouped', (event) => {
    const data = JSON.parse(event.data);
    console.log('错误分组:', data.groups);
  });
  
  // 分析与建议事件
  eventSource.addEventListener('recommendations_generated', (event) => {
    const data = JSON.parse(event.data);
    console.log('改进建议:', data.recommendations);
  });
  
  eventSource.addEventListener('analysis_completed', (event) => {
    const data = JSON.parse(event.data);
    console.log(`分析完成 (用时: ${data.duration})`);
    console.log(`总结: ${data.summary}`);
    eventSource.close();
  });
  
  return eventSource;
}

// 完整流程示例
async function runCompleteAnalysis() {
  // 1. 获取Lambda资源信息
  const resources = await collectResources();
  console.log('发现Lambda函数:', resources.lambdaName);
  
  // 2. 分类日志
  const classification = await classifyLogs(resources.logGroupName);
  console.log('日志分类结果:', classification.summary);
  
  // 3. 收集主要错误的上下文
  const errorContext = await collectErrorContext(
    resources.logGroupName, 
    classification.responseLogs.statusCodes['400'] > 0 ? 'device_not_enabled' : null
  );
  console.log('收集到的错误上下文数:', errorContext.errorContexts.length);
  
  // 4. 分析错误并获取建议
  const analysis = await analyzeErrors(resources.logGroupName);
  console.log('分析完成，成功率:', analysis.summary.successRate + '%');
  console.log('主要建议:', analysis.recommendations[0].recommendation);
  
  // 5. 连接SSE流获取实时分析结果
  connectToAnalysisStream(analysis.streamUrl);
}
``` 