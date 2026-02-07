# 🚀 Claude Code Profile Manager (CCPM)  For WSL/Linux/MacOS

> **Cloud927 出品** | 让你的 Claude Code 变成“万能 AI 启动器”

## 📖 这是什麼？(Introduction)

**Claude Code** 是 Anthropic 官方推出的强大命令行 AI 编程工具。但在默认情况下，它切换模型非常麻烦（需要手动改文件，还容易改坏）。

**CCPM (Claude Code Profile Manager)** 是一个为 WSL/Linux/macOS 用户打造的**自动化管理脚本**。

**它的核心功能：**

- 🛠 **全自动装配**：输入 `add-model`，回答 4 个问题，自动配置好 DeepSeek、Kimi、Minimax 等国产模型。
- ⚡️ **一键切换**：想用 DeepSeek？输入 `cdsc`；想用 Kimi？输入 `ckm`。秒切，不冲突。
- 🛡 **安全隔离**：不同模型的 API Key 互不干扰，保护你的隐私。
- 📛 **自定义命名**：支持设置“显示名称”和“指令简称”（如：看板显示 `DeepSeek V3`，指令只需输 `cdsc`）。

------

## 🏗 第一步：环境准备 (Prerequisites)

在开始之前，请确保你的电脑里安装了 **Node.js**（因为 Claude Code 是基于 Node 运行的）。

1. 打开你的终端 (Terminal / WSL)。

2. 输入以下命令检查：

   Bash

   ```
   node -v
   npm -v
   ```

3. **如果没有显示版本号**：请先去 [Node.js 官网](https://nodejs.org/) 下载安装（推荐安装 LTS 版本）。

------

## 📦 第二步：安装 Claude Code (Install)

如果你还没安装官方工具，请在终端执行以下命令：

Bash

```
npm install -g @anthropic-ai/claude-code
```

安装完成后，输入 `claude --version`，如果出现版本号（如 0.2.x），说明安装成功！

> **小贴士**：你需要先运行一次 `claude login` 登录官方账号吗？
>
> - **建议登录**：虽然我们主要用第三方模型，但登录官方账号可以获得更稳定的体验。
> - **不登录也能用**：只要你有第三方 API Key，CCPM 可以直接跳过官方验证。

------

## ⚙️ 第三步：安装自动化脚本 (Setup Script)

这是最关键的一步，我们将“注入”管理系统的核心代码。

1. 在终端输入以下命令，打开你的配置文件：

   Bash

   ```
   mkdir -p ~/.claude_profiles  # 先创建存放配置的文件夹
   nano ~/.bashrc               # 编辑配置文件
   ```

2. **滚动到文件最底部**，将下面的代码**完整复制并粘贴**进去：

Bash

```
# ---  AI 武器库  ---

# 1. 核心启动器
function run_claude() {
    local profile_name="$1"
    local config_file="$HOME/.claude_profiles/$profile_name"
    if [ -f "$config_file" ]; then
        (source "$config_file" && echo "🚀 载入配置: $profile_name" && claude)
    else
        echo "❌ 找不到配置: $config_file"
    fi
}

# 2. 自动化装配向导 (支持设定显示名)
function add-model() {
    echo "🛠️  开始添加新模型..."
    
    # 关键改进：分离“指令简称”和“显示全名”
    read -p "1. 设定指令简称 (如 dsc, qwen): " short_name
    if [ -z "$short_name" ]; then echo "❌ 简称不能为空"; return; fi
    if [ -f "$HOME/.claude_profiles/$short_name" ]; then echo "⚠️  简称 '$short_name' 已被占用！"; return; fi

    read -p "2. 设定显示全名 (如 DeepSeek-Chat, 通义千问): " display_name
    [ -z "$display_name" ] && display_name="$short_name" # 没填默认用简称

    read -p "3. API Key (必填): " apikey
    read -p "4. Base URL (回车跳过则使用官方): " baseurl
    read -p "5. Model ID (如 claude-3-5-sonnet): " modelid

    # 生成配置，并将显示名作为注释写入第一行
    config_path="$HOME/.claude_profiles/$short_name"
    echo "# DISPLAY_NAME=$display_name" > "$config_path"
    echo "export ANTHROPIC_API_KEY=\"$apikey\"" >> "$config_path"
    [ ! -z "$baseurl" ] && echo "export ANTHROPIC_BASE_URL=\"$baseurl\"" >> "$config_path"
    [ ! -z "$modelid" ] && echo "export ANTHROPIC_MODEL=\"$modelid\"" >> "$config_path"

    # 自动注入别名
    if ! grep -q "alias c$short_name=" ~/.bashrc; then
        echo "alias c$short_name='run_claude $short_name'" >> ~/.bashrc
        alias c$short_name="run_claude $short_name"
    fi
    
    echo "----------------------------------------"
    echo "✅ 装配成功！"
    echo "📛 看板名称: $display_name"
    echo "🚀 启动指令: c$short_name"
    echo "----------------------------------------"
}

# 3. 自动化改名向导
function rename-model() {
    read -p "📝 请输入旧的指令简称 (例如 dsc): " old_name
    if [ ! -f "$HOME/.claude_profiles/$old_name" ]; then echo "❌ 找不到: $old_name"; return; fi

    read -p "✨ 请输入新的指令简称 (例如 newdsc): " new_name
    if [ -f "$HOME/.claude_profiles/$new_name" ]; then echo "❌ 已存在: $new_name"; return; fi

    mv "$HOME/.claude_profiles/$old_name" "$HOME/.claude_profiles/$new_name"
    sed -i "/alias c$old_name=/d" ~/.bashrc
    echo "alias c$new_name='run_claude $new_name'" >> ~/.bashrc
    unalias c$old_name 2>/dev/null
    alias c$new_name="run_claude $new_name"

    echo "✅ 改名成功！旧令 c$old_name 已废除，新令 c$new_name 已生效。"
}

# 4. 快速移除
function rm-model() {
    read -p "🗑️  请输入要删除的指令简称: " name
    if [ -f "$HOME/.claude_profiles/$name" ]; then
        rm "$HOME/.claude_profiles/$name"
        sed -i "/alias c$name=/d" ~/.bashrc
        unalias c$name 2>/dev/null
        echo "✅ 模型 '$name' 已移除。"
    else
        echo "❌ 未找到。"
    fi
}

# 5. 智能看板
models() {
    echo "----------------------------------------"
    echo "📦 AI 武器库 )"
    echo "----------------------------------------"
    printf " %-15s | %-15s\n" "🚀 启动指令" "📛 模型名称"
    echo "----------------------------------------"
    
    for file in ~/.claude_profiles/*; do
        if [ -f "$file" ]; then
            short_name=$(basename "$file")
            
            # 读取显示名
            display_name=$(grep "^# DISPLAY_NAME=" "$file" | cut -d'=' -f2)
            [ -z "$display_name" ] && display_name="$short_name"

            # 显示指令
            cmd="c$short_name"
            
            printf " %-15s | %-15s\n" "$cmd" "$display_name"
        fi
    done
    echo "----------------------------------------"
    echo "👉 新增: add-model | 改名: rename-model"
    echo "👉 删除: rm-model"
}
```

1. **保存退出**：按 `Ctrl + O`，然后回车，再按 `Ctrl + X`。

2. **让配置生效**：在终端执行：

   Bash

   ```
   source ~/.bashrc
   ```

------

## 🎮 第四步：实战！添加你的第一个模型



API搜索各大模型的官方开放平台充值，即可获取，比如kimi开放平台：`https://platform.moonshot.cn/`

假设我们要添加 **DeepSeek (深度求索)**。

1. 在终端输入：

   Bash

   ```
   add-model
   ```

2. 跟着向导回答问题：

   - **1. 设定指令简称**: `dsc` (这决定了以后你输 `cdsc` 就能启动)
   - **2. 设定显示全名**: `DeepSeek-Chat`
   - **3. API Key**: `sk-xxxxxxxxx` (粘贴你的 DeepSeek Key)
   - **4. Base URL**: `https://api.deepseek.com/anthropic`
   - **5. Model ID**: `deepseek-chat`

3. 显示 **✅ 装配成功** 后，输入：

   Bash

   ```
   models
   ```

   你会看到 DeepSeek 已经出现在列表里了！

4. **启动它**：

   Bash

   ```
   cdsc
   ```

   (注意：指令是 `c` + 你设定的简称)

------

## 📝 附录：常用模型配置参数 (Cheat Sheet)

新手不知道 Base URL 填什么？照着下面抄就行！

### 1. DeepSeek (深度求索)

- **Base URL**: `https://api.deepseek.com/anthropic`
- **Model ID**: `deepseek-chat` (如果是 R1 模型，填 `deepseek-reasoner`)

### 2. MiniMax (海螺)

- **Base URL**: `https://api.minimaxi.com/anthropic`--------国际版使用`https://api.minimax.io/anthropic`
- **Model ID**: `MiniMax-M2.1`

### 3. Kimi (Moonshot AI)

- **Base URL**: `https://api.moonshot.cn/anthropic`
- **Model ID**: `kimi-k2.5` (或 32k, 128k)----官方模型：`kimi-k2.5`、`kimi-k2-0905-Preview`、`kimi-k2-turbo-preview`、`kimi-k2-thinking`、`kimi-k2-thinking-turbo`

### 4. Claude 官方模型 (无需 Base URL)

- **Base URL**: (直接按回车跳过)
- **Model ID**: `claude-3-5-sonnet-latest`

------

## ❓ 常见问题 (FAQ)

**Q: 输入 `add-model` 提示找不到命令？**

A: 你可能忘了运行 `source ~/.bashrc`。请运行一次，或者关闭终端重新打开。

**Q: 启动时提示黄色警告 "Auth conflict"？**

A: 这是正常的。说明脚本检测到你可能有官方账号，但正在优先使用你刚才配置的 API Key。可以直接忽略。

**Q: 我填错了怎么办？**

A: 很简单！使用 `rm-model` 删除它，然后重新 `add-model` 即可。

**PS:所有配置的文件目录下面，可以相应的去修改API和模型名字**：

`ls ~/.claude_profiles`

------

**Enjoy your coding! 🚀**