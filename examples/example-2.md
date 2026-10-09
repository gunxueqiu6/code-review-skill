# 案例 2：审查一个 N+1 查询 + 超长函数

## 输入（贴给 Claude）

```python
def get_orders_with_users(order_ids):
    orders = db.query("SELECT * FROM orders WHERE id IN (%s)" % ",".join(map(str, order_ids)))
    result = []
    for order in orders:
        user = db.query(f"SELECT * FROM users WHERE id = {order['user_id']}")  # 循环内查询
        result.append({**order, "user": user})
    # ... 后面还有 80 行拼接、格式化、日志
    return result
```

> 审查这段代码，重点看性能和可维护性

## 输出

```markdown
### High Issues
[H] orders.py:4 — N+1 查询：循环内逐条查用户
  建议: JOIN 一次拉回
  SELECT o.*, u.* FROM orders o LEFT JOIN users u ON u.id = o.user_id
  WHERE o.id IN (...)

[H] orders.py:2 — 用 % 拼接 IN 列表，有注入风险且脆弱
  建议: 参数化 IN 占位符

### Medium Issues
[M] orders.py:2 — get_orders_with_users 函数 90+ 行，超 50 行阈值
  建议: 拆成 fetch_orders / attach_users / format_result 三个函数
```

## 为什么这个案例说明「好用」

- 一眼抓出 N+1（性能大头），并给了 JOIN 的具体写法
- 顺带发现 % 拼接 IN 的注入隐患（很多人漏掉这个）
- 对超长函数给了**可落地的拆分边界**（三个函数名都起好了）
