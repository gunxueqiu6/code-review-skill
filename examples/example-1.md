# 案例 1：审查一个有 SQL 注入 + 认证缺失的登录接口

## 输入（贴给 Claude）

```python
# FastAPI
@app.post("/login")
def login(name: str, password: str):
    sql = f"SELECT * FROM users WHERE name = '{name}' AND password = '{password}'"
    user = db.execute(sql).fetchone()
    if user:
        return {"token": generate_token(user)}
    return {"error": "invalid"}
```

> 审查这段代码（Python + FastAPI）

## 输出

```markdown
## Code Review Report

### Summary
- 文件数: 1
- CRITICAL: 2 | HIGH: 0 | MEDIUM: 1 | LOW: 0

### Critical Issues
[C] login.py:4 — SQL 拼接导致注入，且密码明文进查询
  风险: `name` 传 `' OR '1'='1'--` 即可绕过登录
  建议: 参数化查询 + 密码哈希比对
  db.execute("SELECT * FROM users WHERE name = ?", [name])
  再对 user.password 做 bcrypt/argon2 比对

[C] login.py:3 — 密码以明文进入 SQL 日志/查询
  建议: 密码只用于哈希比对，绝不拼进任何查询或日志

### Medium Issues
[M] login.py:6 — 错误信息可被枚举用户（"invalid" 未区分用户不存在/密码错）
  建议: 统一返回 {"error": "invalid credentials"}
```

## 为什么这个案例说明「好用」

- 10 秒内定位到注入 + 明文密码两个 CRITICAL
- 每个问题给了**具体改法**（参数化 + 哈希），不是「建议改进」
- 附带了用户枚举这种中级隐患，比人肉 review 更全
