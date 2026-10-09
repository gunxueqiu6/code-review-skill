# 安装（30 秒）

## 方式一：手动复制

1. 创建目录（已存在则跳过）：
   ```
   mkdir -p ~/.claude/skills/code-review
   ```
2. 把本目录的 `SKILL.md` 复制进去：
   ```
   cp SKILL.md ~/.claude/skills/code-review/SKILL.md
   ```
3. 重启 Claude Code（或新开会话），输入 `/skills` 确认 `code-review` 已加载。

## 方式二：安装脚本

```bash
# 从产品包根目录运行
./install.sh
```

## 验证装好了

对 Claude 说：

> 审查这段代码：
> ```python
> def login(name, password):
>     db.execute(f"SELECT * FROM users WHERE name = '{name}'")
> ```

预期：Claude 输出一条 `[H]` SQL 注入问题 + 参数化修复建议，即安装成功。
