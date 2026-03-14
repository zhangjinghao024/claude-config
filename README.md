# claude-config

我的 Claude Code 个人配置备份，包含自定义 skill 和全局设置，方便在多台电脑间同步使用。

## 包含内容

| 文件/目录 | 说明 |
|---|---|
| `commands/` | 自定义 skill（`/resume-interview`、`/format-qa`） |
| `settings.json` | Claude Code 全局配置 |

## 在新电脑上使用

### 前提条件

- 已安装 [Claude Code](https://claude.ai/code)
- 已配置 SSH key 到 GitHub

### 步骤

```bash
# 1. 克隆仓库
git clone git@github.com:zhangjinghao024/claude-config.git ~/claude-config

# 2. 用符号链接挂载 commands 目录（改动自动同步）
rm -rf ~/.claude/commands
ln -s ~/claude-config/commands ~/.claude/commands

# 3. 同步全局配置
cp ~/claude-config/settings.json ~/.claude/settings.json
```

完成后重启 Claude Code，输入 `/` 即可看到自定义 skill。

## 更新配置

在任意电脑修改 skill 后，执行以下命令同步到 GitHub：

```bash
cd ~/claude-config
cp -r ~/.claude/commands ./commands
cp ~/.claude/settings.json ./settings.json
git add .
git commit -m "update config"
git push
```

其他电脑同步最新配置：

> 前提：该电脑已完成上方「在新电脑上使用」的初始化步骤（克隆仓库 + 建立符号链接）。

```bash
cd ~/claude-config
git pull
# commands/ 通过符号链接已实时生效，无需额外操作
# 如果 settings.json 有更新，手动执行：
cp ~/claude-config/settings.json ~/.claude/settings.json
```
