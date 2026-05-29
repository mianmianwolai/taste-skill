# Taste-Skill API 文档

## API 基础信息

### 域名
```
Production: https://api.tasteskill.com
Staging: https://api-staging.tasteskill.com
Development: http://localhost:8080
```

### 版本
```
Current Version: v1
API Base Path: /api/v1
```

### 认证方式
- 使用 JWT Token
- 在 HTTP Header 中传入：`Authorization: Bearer {token}`

---

## 通用响应格式

### 成功响应 (200)
```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "id": 123,
    "name": "示例"
  },
  "timestamp": "2026-05-29T10:00:00Z"
}
```

### 分页响应
```json
{
  "code": 0,
  "message": "成功",
  "data": {
    "content": [
      {"id": 1, "name": "订单1"},
      {"id": 2, "name": "订单2"}
    ],
    "totalElements": 100,
    "totalPages": 10,
    "currentPage": 1,
    "pageSize": 10
  }
}
```

### 错误响应
```json
{
  "code": 400,
  "message": "参数验证失败",
  "errors": [
    {
      "field": "orderCode",
      "message": "订单号不能为空"
    }
  ],
  "timestamp": "2026-05-29T10:00:00Z"
}
```

---

## 核心 API 端点

### 认证 API

#### 登录
```
POST /api/v1/auth/login

Request:
{
  "username": "admin",
  "password": "password123"
}

Response:
{
  "code": 0,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
    "user": {
      "id": 1,
      "username": "admin",
      "roles": ["ADMIN"]
    }
  }
}
```

---

### 订单 API

#### 创建订单
```
POST /api/v1/orders
Headers: Authorization: Bearer {token}

Request:
{
  "orderCode": "ORD-2026-001",
  "customerId": 1,
  "productName": "产品A",
  "quantity": 100,
  "unit": "件",
  "dueDate": "2026-06-30",
  "priority": "HIGH",
  "remark": "加急订单"
}

Response:
{
  "code": 0,
  "data": {
    "id": 123,
    "orderCode": "ORD-2026-001",
    "status": "PLANNING",
    "createdAt": "2026-05-29T10:00:00Z"
  }
}
```

#### 获取订单详情
```
GET /api/v1/orders/{id}
Headers: Authorization: Bearer {token}

Response:
{
  "code": 0,
  "data": {
    "id": 123,
    "orderCode": "ORD-2026-001",
    "customer": {
      "id": 1,
      "customerName": "客户A"
    },
    "productName": "产品A",
    "quantity": 100,
    "dueDate": "2026-06-30",
    "status": "PLANNING",
    "progress": 0,
    "workOrders": [
      {
        "id": 456,
        "workOrderCode": "WO-2026-001",
        "status": "PENDING"
      }
    ],
    "createdAt": "2026-05-29T10:00:00Z"
  }
}
```

#### 查询订单列表
```
GET /api/v1/orders?page=1&pageSize=10&status=PLANNING&priority=HIGH
Headers: Authorization: Bearer {token}

Response:
{
  "code": 0,
  "data": {
    "content": [
      {"id": 1, "orderCode": "ORD-2026-001"},
      {"id": 2, "orderCode": "ORD-2026-002"}
    ],
    "totalElements": 50,
    "totalPages": 5,
    "currentPage": 1,
    "pageSize": 10
  }
}
```

---

### 生产看板 API

#### 获取实时看板数据
```
GET /api/v1/dashboard/production
Headers: Authorization: Bearer {token}

Response:
{
  "code": 0,
  "data": {
    "today": {
      "plannedQty": 1000,
      "producedQty": 850,
      "defectQty": 15,
      "oee": 82.5
    },
    "productionLines": [
      {
        "lineId": 1,
        "lineName": "1号线",
        "currentWorkOrder": "WO-2026-001",
        "progress": 75,
        "status": "RUNNING",
        "estCompleteTime": "2026-05-29 14:30"
      }
    ],
    "overdueAlerts": [
      {
        "orderId": 123,
        "orderCode": "ORD-2026-001",
        "dueDate": "2026-05-30",
        "daysOverdue": -1
      }
    ]
  }
}
```

---

### 工单 API

#### 创建工单
```
POST /api/v1/work-orders
Headers: Authorization: Bearer {token}

Request:
{
  "orderId": 123,
  "productName": "产品A",
  "quantity": 100,
  "processId": 1,
  "teamId": 1,
  "expectedStart": "2026-05-29 08:00",
  "expectedEnd": "2026-05-29 18:00"
}

Response:
{
  "code": 0,
  "data": {
    "id": 456,
    "workOrderCode": "WO-2026-001",
    "status": "PENDING",
    "createdAt": "2026-05-29T10:00:00Z"
  }
}
```

#### 下发工单
```
POST /api/v1/work-orders/{id}/dispatch
Headers: Authorization: Bearer {token}

Request:
{
  "teamId": 1,
  "message": "立即开始生产"
}

Response:
{
  "code": 0,
  "data": {
    "id": 456,
    "status": "DISPATCHED",
    "dispatchedAt": "2026-05-29T10:00:00Z"
  }
}
```

---

### 报工 API

#### 创建报工
```
POST /api/v1/reports
Headers: Authorization: Bearer {token}

Request:
{
  "workOrderId": 456,
  "workerId": 10,
  "quantity": 50,
  "defectQty": 2,
  "durationMinutes": 120,
  "reportType": "NORMAL",
  "remark": "正常生产"
}

Response:
{
  "code": 0,
  "data": {
    "id": 789,
    "reportCode": "REP-2026-001",
    "status": "PENDING"
  }
}
```

---

### 设备管理 API

#### 获取设备列表
```
GET /api/v1/equipment?page=1&pageSize=10
Headers: Authorization: Bearer {token}

Response:
{
  "code": 0,
  "data": {
    "content": [
      {
        "id": 1,
        "equipmentCode": "EQ-001",
        "equipmentName": "数控车床",
        "status": "NORMAL",
        "location": "车间A",
        "qrCode": "https://..."
      }
    ]
  }
}
```

#### 上传点检记录
```
POST /api/v1/inspections
Headers: Authorization: Bearer {token}

Request:
{
  "planId": 1,
  "inspectorId": 10,
  "itemRecords": {
    "温度": "72",
    "噪音": "85dB",
    "是否异常": false
  },
  "photos": ["photo1.jpg", "photo2.jpg"]
}

Response:
{
  "code": 0,
  "data": {
    "id": 999,
    "recordCode": "INS-2026-001",
    "hasException": false
  }
}
```

#### 上报异常
```
POST /api/v1/exceptions
Headers: Authorization: Bearer {token}

Request:
{
  "equipmentId": 1,
  "faultDescription": "电机发热过高",
  "severity": "HIGH",
  "reporterId": 10,
  "photos": []
}

Response:
{
  "code": 0,
  "data": {
    "id": 888,
    "reportCode": "EXP-2026-001",
    "status": "PENDING"
  }
}
```

---

### 质量管理 API

#### 创建质检记录
```
POST /api/v1/quality/records
Headers: Authorization: Bearer {token}

Request:
{
  "workOrderId": 456,
  "batchId": "BATCH-2026-001",
  "inspectionItems": {
    "尺寸": {"value": "10.2", "result": "OK"},
    "外观": {"value": "良好", "result": "OK"}
  },
  "qualifiedQty": 48,
  "defectQty": 2,
  "defectReason": "尺寸偏差"
}

Response:
{
  "code": 0,
  "data": {
    "id": 555,
    "recordCode": "QC-2026-001",
    "status": "PENDING"
  }
}
```

#### 批次追溯
```
GET /api/v1/quality/traceability/{batchId}
Headers: Authorization: Bearer {token}

Response:
{
  "code": 0,
  "data": {
    "batchId": "BATCH-2026-001",
    "rawMaterial": {
      "supplier": "供应商A",
      "batchNo": "MAT-2026-001",
      "receiveDate": "2026-05-20"
    },
    "production": {
      "workOrder": "WO-2026-001",
      "processSteps": [
        {"step": "1", "worker": "张三", "startTime": "2026-05-21 08:00"},
        {"step": "2", "worker": "李四", "startTime": "2026-05-21 10:00"}
      ]
    },
    "quality": {
      "qualifiedQty": 98,
      "defectQty": 2,
      "result": "合格"
    },
    "delivery": {
      "customer": "客户A",
      "deliveryDate": "2026-05-25",
      "logisticsNo": "SF123456"
    }
  }
}
```

---

### 销售管理 API

#### 创建报价单
```
POST /api/v1/sales/quotations
Headers: Authorization: Bearer {token}

Request:
{
  "customerId": 1,
  "salesPersonId": 5,
  "items": [
    {
      "productName": "产品A",
      "quantity": 100,
      "unitPrice": 50,
      "remark": "原价"
    }
  ],
  "validUntil": "2026-06-30"
}

Response:
{
  "code": 0,
  "data": {
    "id": 111,
    "quotationCode": "QT-2026-001",
    "totalAmount": 5000,
    "grandTotal": 5650,
    "status": "DRAFT"
  }
}
```

---

### 资料管理 API

#### 上传文件
```
POST /api/v1/documents/upload
Headers: Authorization: Bearer {token}, Content-Type: multipart/form-data

Request:
- file: [binary file data]
- category: BOM
- tags: ["产品A", "2026年"]
- description: "产品A的物料清单"
- isPublic: false

Response:
{
  "code": 0,
  "data": {
    "id": 222,
    "filename": "BOM_产品A.xlsx",
    "documentCode": "DOC-2026-001",
    "uploadedAt": "2026-05-29T10:00:00Z"
  }
}
```

#### 全文搜索
```
GET /api/v1/documents/search?keyword=BOM&category=BOM&page=1&pageSize=10
Headers: Authorization: Bearer {token}

Response:
{
  "code": 0,
  "data": {
    "content": [
      {
        "id": 222,
        "filename": "BOM_产品A.xlsx",
        "category": "BOM",
        "relevance": 0.95
      }
    ],
    "totalElements": 5,
    "totalPages": 1
  }
}
```

---

## 错误代码

| 错误代码 | HTTP 状态码 | 含义 |
|---------|----------|------|
| 0 | 200 | 成功 |
| 400 | 400 | 参数验证失败 |
| 401 | 401 | 未授权 |
| 403 | 403 | 禁止访问 |
| 404 | 404 | 资源不存在 |
| 409 | 409 | 冲突（如订单号重复） |
| 500 | 500 | 服务器错误 |

---

**文档版本**：v1.0  
**最后更新**：2026-05-29