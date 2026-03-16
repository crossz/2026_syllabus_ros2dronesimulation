
#### 机会到底在哪里

[2025 Anthropic 的黑客松](https://www.biji.com/note/share_note/rBWK74o1wO6Aw)

- **参与规模**：500名开发者，基于**Opus 4.6**和**Claude Code**开发。
- **开发周期**：一周。
- **核心成果**：5个Agent落地项目获奖，**无AI聊天助手类项目**。
- **核心洞察**：AI产品的最大机会**不在AI圈子内部**，而在传统行业的信息处理场景中。

**(一) 冠军项目：建筑许可审批加速工具**
**(二) 亚军项目：儿童编程教育工具**
**(三) 季军项目：心脏科诊疗随访工具**

> 开发者非传统开发人员居多
> Anthropic 是什么公司，什么意思，其最近热点的动态有哪些


---

### 2.2 这门课要怎么上

用 **大模型** 解决 **实际问题**

_而不是教大家语法和编程_

<br />

> 怎么用、用哪些 **大模型**


---
# ROS2 开发（LLM based）

1. 面向百度、qq群编程 -> 面向豆包编程
2. vibe coding
3. agentic coding

## 1. 大模型的 agentic coding

- Claude Code 和 Open Code
- model and API



---

#### Vibe Coding -> Agentic Coding

- Claude Code
	是什么，为什么；以及后续的 OpenClaw 为什么能带飞 Mac Mini 的销量
- Opencode 和 各种 Code
	都有哪些？

![[assets/week1/w1-8-claudecodeandshell.png]]

> 稍后实践：什么是异步编程

---

#### CLAUDE CODE 或 OPEN CODE 安装


1. 先安装 node.js 24, [官网](https://nodejs.org/en/download)
```bash
# Download and install nvm:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.4/install.sh | bash

# in lieu of restarting the shell
\. "$HOME/.nvm/nvm.sh"

# Download and install Node.js:
nvm install 24

# Verify the Node.js version:
node -v # Should print "v24.14.0".

# Verify npm version:
npm -v # Should print "11.9.0".

```

2. 安装 claude code 或者 opencode

https://www.npmjs.com/

```bash
npm i -g @anthropic-ai/claude-code
```

```bash
npm i -g opencode-ai
```

3. 配置
`.claude/settings.json`
`.config/opencode/opencode.json`



---
## 2. Git配置

git 是虚拟机开发必备

```bash
# 配置Git
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

# SSH密钥
ssh-keygen -t ed25519 -C "your@email.com"
cat ~/.ssh/id_ed25519.pub  # 添加到GitHub
```

> 用 Claude Code 或 Opencode 进行 git 操作
---
