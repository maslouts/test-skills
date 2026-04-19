---
name: ehr-account-creator
description: 通过 EHR 脚本自动创建临时账号。当用户提供邮箱、姓名、手机号时，自动解析邮箱前缀并执行创建脚本。触发词：创建 EHR 账号、创建临时账号、EHR 账号、临时账号。
---

# EHR 临时账号自动创建

## 触发条件

当用户消息中包含以下信息时激活：
- **邮箱**（用于解析用户名前缀）
- **姓名**
- **手机号**

### 触发关键词
- 创建 EHR 账号
- 创建临时账号
- EHR 账号
- 临时账号
- 帮我创建账号

## 信息解析

### 邮箱前缀提取规则
从邮箱地址中提取 `@` 符号前的部分作为 `account-prefix`：
- 示例：`zhangsan_ta@zhitech.com` → `zhangsan` 或 `zhangsan_ta`（去掉 `_ta` 后缀）
- 示例：`lisi@zhitech.com` → `lisi`

### 必填信息
| 字段 | 来源 | 说明 |
|------|------|------|
| 姓名 | 用户提供 | `--name` |
| 手机号 | 用户提供 | `--phone`（必填） |
| 邮箱前缀 | 从邮箱解析 | `--account-prefix` |

### 默认配置
| 参数 | 默认值 |
|------|--------|
| `--login-user` | `mawenqiang` |
| `--login-pass` | `111111` |
| `--department` | `北京值得买科技` |
| `--job` | `测试开发` |
| `--project` | `创建账号` |
| `--project-leader` | `AI 创建` |
| `--work-place` | `北京` |
| `--type` | `驻场外包` |
| `--status` | `在职` |
| `--enter-date` | 今天日期（YYYY-MM-DD） |

## 执行流程

### 1. 解析用户消息
从用户消息中提取：
- 邮箱地址（正则匹配）
- 姓名
- 手机号

若信息不全，主动询问缺失项。

### 2. 解析邮箱前缀
```python
import re

def extract_account_prefix(email: str) -> str:
    """从邮箱提取前缀，去掉 _ta 后缀"""
    match = re.match(r'^([^@]+)@', email)
    if not match:
        raise ValueError(f"无效邮箱格式：{email}")
    
    prefix = match.group(1)
    # 去掉 _ta 后缀
    if prefix.endswith('_ta'):
        prefix = prefix[:-3]
    return prefix
```

### 3. 执行脚本
```bash
/Users/smzdm/virtualenv/Plag_Ui/bin/python3 \
  /Users/smzdm/Desktop/openclaw/ehr_create_temp_account.py \
  --login-user mawenqiang \
  --login-pass 111111 \
  --name <姓名> \
  --account-prefix <解析的前缀> \
  --phone <手机号> \
  --department 北京值得买科技 \
  --job 测试开发 \
  --project 创建账号 \
  --project-leader AI 创建 \
  --work-place 北京 \
  --type 驻场外包 \
  --status 在职 \
  --enter-date <今天日期>
```

### 4. 结果处理

#### 成功
解析脚本输出的 JSON，提取关键信息：
```json
{
  "ok": true,
  "created": {
    "name": "张三",
    "account": "zhangsan_ta",
    "email": "zhangsan_ta@zhitech.com",
    "phone": "13800138000",
    ...
  }
}
```

汇报格式：
```
✅ **EHR 账号创建成功**

| 字段 | 值 |
|------|------|
| 姓名 | 张三 |
| 账号 | zhangsan_ta |
| 邮箱 | zhangsan_ta@zhitech.com |
| 手机号 | 13800138000 |
| 部门 | 北京值得买科技 |
| 岗位 | 测试开发 |
| 类型 | 驻场外包 |
| 状态 | 在职 |
```

#### 失败
解析错误信息，翻译成中文：
```
❌ **EHR 账号创建失败**

**原因**：<错误原因翻译>

**建议**：<可能的解决方案>
```

常见错误及处理：
| 错误 | 原因 | 解决方案 |
|------|------|----------|
| 登录失败 | 用户名/密码错误 | 检查凭据 |
| 手机号已存在 | 该手机号已注册 | 更换手机号 |
| 部门未找到 | 部门名称不存在 | 检查部门名称 |
| 网络超时 | EHR 服务不可达 | 稍后重试 |

## 注意事项

- 脚本执行超时设置为 60 秒
- 所有英文错误信息翻译成中文展示
- 成功/失败都要明确告知用户
- 敏感信息（密码）不在日志中明文展示
