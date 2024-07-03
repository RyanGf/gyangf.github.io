---
layout:     post
title:      "odoo与企业微信整合"
subtitle:   " \"Hello World, Hello Blog\""
date:       2024-07-03 12:00:00
author:     "Fred"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Meta
---


# WeChat Work Base Module (wecom_base.py)

## 概述
`wecom_base.py` 模块是企业微信集成的核心组件，定义了 `WeComBase` 抽象模型。该模型为其他企业微信相关模型提供了基础功能和通用方法。

## 模型：WeComBase

### 继承
- `models.AbstractModel`

### 字段
- `active` (Boolean): 用于指示记录是否处于活动状态，默认为 True。

### 方法

#### log_error(error_message)
记录错误信息。
- 参数：
  - `error_message` (str): 要记录的错误消息
- 用法：
  ```python
  self.log_error("发生了一个错误")
  ```

#### create(vals)
重写创建方法，添加错误日志记录。
- 参数：
  - `vals` (dict): 要创建的记录的值
- 返回：新创建的记录

#### write(vals)
重写写入方法，添加错误日志记录。
- 参数：
  - `vals` (dict): 要更新的值
- 返回：更新操作的结果

#### get_wecom_config()
获取企业微信配置。
- 返回：包含企业微信配置的字典

#### format_wecom_response(response)
格式化企业微信 API 响应。
- 参数：
  - `response` (dict): API 响应
- 返回：格式化后的响应

#### handle_wecom_error(error)
处理企业微信 API 错误。
- 参数：
  - `error`: 错误对象或消息
- 抛出：`ValidationError` 异常

## 使用示例

### 创建继承 WeComBase 的新模型

```python
from odoo import models, fields

class WeComApplication(models.Model):
    _name = 'wecom.application'
    _inherit = ['wecom.base']
    _description = 'WeChat Work Application'

    name = fields.Char(string='Application Name', required=True)
    
    def some_method(self):
        try:
            # 某些操作
            pass
        except Exception as e:
            self.log_error(f"在 some_method 中发生错误: {str(e)}")
```

### 使用 WeComBase 方法

```python
class WeComMessage(models.Model):
    _name = 'wecom.message'
    _inherit = ['wecom.base']

    def send_message(self):
        config = self.get_wecom_config()
        try:
            # 使用配置发送消息
            response = self.call_wecom_api(config)
            formatted_response = self.format_wecom_response(response)
            # 处理响应
        except Exception as e:
            self.handle_wecom_error(e)
```

## 最佳实践
1. 所有与企业微信相关的模型应继承 `WeComBase`。
2. 使用 `log_error` 方法记录所有重要的错误信息。
3. 在处理企业微信 API 响应时，始终使用 `format_wecom_response` 方法。
4. 使用 `handle_wecom_error` 统一处理企业微信相关的错误。

## 注意事项
- `get_wecom_config` 方法需要根据实际存储配置的方式进行实现。
- 确保在继承 `WeComBase` 的模型中正确调用 `super()` 方法，特别是在重写 `create` 和 `write` 方法时。

## 未来增强
- 添加更多通用的企业微信相关方法，如 token 管理、通用的 API 调用方法等。
- 实现更复杂的错误处理和日志记录机制。
- 考虑添加缓存机制以提高性能。

---

这个文档提供了 `wecom_base.py` 模块的综合概述，包括其主要组件、使用方法和最佳实践。您可以根据实际实现和需求进一步扩展或修改这个文档。如果模块有任何更新或新功能，请确保及时更新文档以保持其准确性。
====================================================================================================================================================================

# WeChat Work Application Module (wecom_application.py)

## 概述

`wecom_application.py` 模块定义了 `WeComApplication` 模型，用于管理企业微信应用的配置和功能。该模型是企业微信集成的核心，处理应用的基本信息、认证和 API 交互。

## 模型：WeComApplication

### 继承

- `models.Model`
- `wecom.base`

### 字段

| 字段名 | 类型 | 描述 |
|--------|------|------|
| name | Char | 应用的完整名称（计算字段） |
| app_name | Char | 应用名称 |
| company_id | Many2one | 关联的公司 |
| type_id | Many2one | 应用类型 |
| category_ids | Many2many | 应用类别 |
| agent_id | Integer | 企业微信应用的 AgentId |
| secret | Char | 应用密钥 |
| is_active | Boolean | 应用是否激活 |
| sequence | Integer | 排序序号 |
| access_token | Char | 访问令牌 |
| token_expiration_time | Datetime | 令牌过期时间 |
| webhook_ids | One2many | 关联的 Webhook |
| setting_ids | One2many | 应用设置 |

### 方法

#### _compute_name

计算应用的完整名称。

#### _check_agent_id

验证 agent_id 是否为正整数。

#### get_access_token

获取或刷新访问令牌。

- 返回：访问令牌字符串

#### refresh_app_info

从企业微信服务器刷新应用信息。

#### cron_refresh_tokens

定时任务，用于刷新所有应用的访问令牌。

#### action_view_webhooks

返回查看应用关联 Webhook 的动作。

## 使用示例

### 创建新应用

```python
app = self.env['wecom.application'].create({
    'app_name': 'My WeChat Work App',
    'company_id': company.id,
    'type_id': app_type.id,
    'agent_id': 1000001,
    'secret': 'your_app_secret',
})
```
### 获取访问令牌
```python
app = self.env['wecom.application'].browse(app_id)
access_token = app.get_access_token()
```
### 刷新应用信息
```python
app.refresh_app_info()
```

为您提供 `wecom_application.py` 的开发文档，使用 Markdown 格式：


# WeChat Work Application Module (wecom_application.py)

## 概述

`wecom_application.py` 模块定义了 `WeComApplication` 模型，用于管理企业微信应用的配置和功能。该模型是企业微信集成的核心，处理应用的基本信息、认证和 API 交互。

## 模型：WeComApplication

### 继承

- `models.Model`
- `wecom.base`

### 字段

| 字段名 | 类型 | 描述 |
|--------|------|------|
| name | Char | 应用的完整名称（计算字段） |
| app_name | Char | 应用名称 |
| company_id | Many2one | 关联的公司 |
| type_id | Many2one | 应用类型 |
| category_ids | Many2many | 应用类别 |
| agent_id | Integer | 企业微信应用的 AgentId |
| secret | Char | 应用密钥 |
| is_active | Boolean | 应用是否激活 |
| sequence | Integer | 排序序号 |
| access_token | Char | 访问令牌 |
| token_expiration_time | Datetime | 令牌过期时间 |
| webhook_ids | One2many | 关联的 Webhook |
| setting_ids | One2many | 应用设置 |

### 方法

#### _compute_name

计算应用的完整名称。

#### _check_agent_id

验证 agent_id 是否为正整数。

#### get_access_token

获取或刷新访问令牌。

- 返回：访问令牌字符串

#### refresh_app_info

从企业微信服务器刷新应用信息。

#### cron_refresh_tokens

定时任务，用于刷新所有应用的访问令牌。

#### action_view_webhooks

返回查看应用关联 Webhook 的动作。

## 使用示例

### 创建新应用

```python
app = self.env['wecom.application'].create({
    'app_name': 'My WeChat Work App',
    'company_id': company.id,
    'type_id': app_type.id,
    'agent_id': 1000001,
    'secret': 'your_app_secret',
})
```

### 获取访问令牌

```python
app = self.env['wecom.application'].browse(app_id)
access_token = app.get_access_token()
```

### 刷新应用信息

```python
app.refresh_app_info()
```

## 最佳实践

1. 使用 `get_access_token` 方法获取最新的访问令牌，避免直接访问 `access_token` 字段。
2. 定期调用 `refresh_app_info` 方法以保持应用信息的最新状态。
3. 使用 `cron_refresh_tokens` 方法设置自动刷新令牌的定时任务。
4. 在处理敏感信息（如 `secret`）时，确保采取适当的安全措施。

## 注意事项

- 确保 `agent_id` 在创建或更新应用时是唯一的。
- `secret` 字段存储敏感信息，应采取适当的安全措施（如加密存储）。
- 访问令牌的刷新逻辑应考虑企业微信 API 的速率限制。

## 未来增强

- 实现更复杂的令牌管理机制，如令牌池。
- 添加更多与企业微信 API 交互的方法，如发送消息、管理通讯录等。
- 实现应用配置的版本控制和历史记录。
- 增加与其他 Odoo 模块的集成，如 HR 或 CRM。

这个文档提供了 `wecom_application.py` 模块的综合概述，包括其主要组件、字段、方法、使用示例以及最佳实践。您可以根据实际实现和需求进一步扩展或修改这个文档。如果模块有任何更新或新功能，请确保及时更新文档以保持其准确性。

