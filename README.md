# Claude Code Buddy 刷宠指南

> **刷出你的专属传说宠物！**

![Legendary Buddy 效果图](Legendary_Gable.png)

*刷出的 Legendary Dragon — 金色五星闪闪发光*

---

## 目录

- [原理说明](#原理说明)
- [Buddy 属性一览](#buddy-属性一览)
- [准备工作](#准备工作)
- [Step 1: 获取 OAuth Token](#step-1-获取-oauth-token)
- [Step 2: 重置 claude.json](#step-2-重置-claudejson)
- [Step 3: 用 OAuth Token 启动 Claude 生成新配置](#step-3-用-oauth-token-启动-claude-生成新配置)
- [Step 4: 用脚本刷出想要的 userID](#step-4-用脚本刷出想要的-userid)
- [Step 5: 写入 userID 并查看 Buddy](#step-5-写入-userid-并查看-buddy)
- [buddy-reroll.js 脚本用法](#buddy-rerolljs-脚本用法)
- [常见问题](#常见问题)

---

## 原理说明

Claude Code 的 `/buddy` 命令根据 `~/.claude.json` 中的 **userID** 通过哈希算法确定性地生成宠物属性。相同的 userID 永远对应同一只宠物。

正常登录时，Claude Code 会将你的 `accountUuid` 写入 `~/.claude.json`，`/buddy` 会使用这个 `accountUuid` 作为 userID，因此你的宠物是固定的。

**刷宠的关键**：使用 `CLAUDE_CODE_OAUTH_TOKEN` 环境变量登录时，Claude Code **不会**将 `accountUuid` 写入 `~/.claude.json`。这样你就可以手动写入一个精心挑选的 userID，从而得到你想要的宠物。

---

## Buddy 属性一览

### 种族 (Species) — 共 18 种

| 种族 | 英文 | 种族 | 英文 |
|------|------|------|------|
| 鸭子 | duck | 鹅 | goose |
| 果冻 | blob | 猫 | cat |
| 龙 | dragon | 章鱼 | octopus |
| 猫头鹰 | owl | 企鹅 | penguin |
| 乌龟 | turtle | 蜗牛 | snail |
| 幽灵 | ghost | 六角恐龙 | axolotl |
| 水豚 | capybara | 仙人掌 | cactus |
| 机器人 | robot | 兔子 | rabbit |
| 蘑菇 | mushroom | 胖墩 | chonk |

### 稀有度 (Rarity)

| 稀有度 | 概率 | 属性下限 | 星级 |
|--------|------|----------|------|
| Common | 60% | 5 | ★ |
| Uncommon | 25% | 15 | ★★ |
| Rare | 10% | 25 | ★★★ |
| Epic | 4% | 35 | ★★★★ |
| Legendary | 1% | 50 | ★★★★★ |

### 眼睛 (Eyes) — 共 6 种

`·`  `✦`  `×`  `◉`  `@`  `°`

### 帽子 (Hats) — 共 8 种

| 帽子 | 说明 |
|------|------|
| none | 无帽子 |
| crown | 皇冠 |
| tophat | 礼帽 |
| propeller | 螺旋桨帽 |
| halo | 光环 |
| wizard | 巫师帽 |
| beanie | 毛线帽 |
| tinyduck | 小鸭子帽 |

> 注意：Common 稀有度的宠物**没有帽子**，只有 Uncommon 及以上才会随机帽子。

### 五维属性 (Stats)

- **DEBUGGING** — 调试能力
- **PATIENCE** — 耐心
- **CHAOS** — 混乱值
- **WISDOM** — 智慧
- **SNARK** — 毒舌值

每只宠物有一个**主属性**（加成最高）和一个**弱属性**（数值最低），其余为普通。

### 闪光 (Shiny)

出现概率仅 **1%**，极其稀有。

---

## 准备工作

1. 已安装 Claude Code（npm 方式或 native 方式均可）
2. 已登录过 Claude Code（确保能正常使用）
3. 下载 `buddy-reroll.js` 脚本

---

## Step 1: 获取 OAuth Token

在终端中执行：

```bash
claude setup-token
```

按照提示完成操作，获取你的 OAuth Token 并**保存好**。

---

## Step 2: 重置 claude.json

**备份**你当前的配置文件（以防万一）：

```bash
cp ~/.claude.json ~/.claude.json.bak
```

然后**删除**原文件并创建新的最小配置：

```bash
rm ~/.claude.json
```

创建新的 `~/.claude.json`，内容如下：

```json
{
  "hasCompletedOnboarding": true,
  "theme": "dark"
}
```

> `theme` 可以改为 `"light"` 或 `"light-daltonized"` / `"dark-daltonized"`，按你喜好来。

---

## Step 3: 用 OAuth Token 启动 Claude 生成新配置

设置环境变量并启动 Claude：

```bash
CLAUDE_CODE_OAUTH_TOKEN=你的oauth_token claude
```

> Windows 用户使用：
> ```powershell
> $env:CLAUDE_CODE_OAUTH_TOKEN="你的oauth_token"; claude
> ```

Claude 启动后会生成完整的 `~/.claude.json`，但**不会**包含 `accountUuid`。

**重要**：启动成功后**直接退出** Claude（`Ctrl+C` 或输入 `/exit`），**不要**在这时使用 `/buddy`。

---

## Step 4: 用脚本刷出想要的 userID

根据你的 Claude Code **安装方式**选择运行命令：

- **npm 安装**（通过 `npm install -g @anthropic-ai/claude-code` 安装的）→ 使用 `node`
- **native 安装**（通过安装包/homebrew 等安装的）→ 使用 `bun`

> **重要提示**：`bun` 运行的结果与 Claude Code 实际使用的哈希算法一致，`node` 的 FNV-1a 哈希结果**不匹配** Claude Code。如果你是 npm 安装用户，刷出的结果可能与实际不符，建议额外安装 bun 来运行脚本。

### 示例：刷一只 Legendary 龙

```bash
# bun（推荐，结果与 Claude Code 一致）
bun buddy-reroll.js --species dragon --rarity legendary

# node（结果可能不匹配）
node buddy-reroll.js --species dragon --rarity legendary
```

### 示例：刷一只闪光猫

```bash
bun buddy-reroll.js --species cat --shiny
```

### 示例：刷全属性 90+ 的 Legendary

```bash
bun buddy-reroll.js --rarity legendary --min-stats 90
```

脚本会输出匹配的 userID，例如：

```
#1 [legendary] dragon eye=✦ hat=crown shiny=false
   stats: DEBUGGING:92 PATIENCE:78 CHAOS:65 WISDOM:88 SNARK:71
   uid:   a1b2c3d4e5f6...（一长串十六进制字符串）
```

**记下你想要的那个 `uid`**。

---

## Step 5: 写入 userID 并查看 Buddy

打开 `~/.claude.json`，找到或添加 `"userID"` 字段，将刷出的 uid 写入：

```json
{
  "hasCompletedOnboarding": true,
  "theme": "dark",
  "userID": "a1b2c3d4e5f6...你刷出的uid"
}
```

> **注意**：是 `userID`（大写 I 和 D），不是 `userid`。

保存文件后，正常启动 Claude：

```bash
claude
```

输入 `/buddy` 命令，你应该能看到你想要的宠物了！

---

## buddy-reroll.js 脚本用法

### 全部参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `--species <name>` | 目标种族 | 不限 |
| `--rarity <name>` | 最低稀有度（会匹配该等级及以上） | 不限 |
| `--eye <char>` | 目标眼睛样式 | 不限 |
| `--hat <name>` | 目标帽子 | 不限 |
| `--shiny` | 要求闪光 | 否 |
| `--min-stats [value]` | 要求所有属性 >= 指定值 | 90 |
| `--max <number>` | 最大搜索次数 | 50,000,000 |
| `--count <number>` | 找到几个结果后停止 | 3 |
| `--check <uid>` | 查看指定 userID 对应的宠物 | - |

### 查看已有 userID 对应的宠物

```bash
bun buddy-reroll.js --check 你的userID
```

### 组合筛选

可以同时指定多个条件：

```bash
bun buddy-reroll.js --species cat --rarity legendary --hat crown --eye ✦
```

> 条件越多，搜索时间越长。闪光 + Legendary + 指定种族可能需要搜索数千万次。

---

## 常见问题

### Q: `/buddy` 显示的宠物和脚本预测的不一样？

如果你用 `node` 运行脚本，结果**不会**与 Claude Code 匹配（因为哈希算法不同）。请使用 `bun` 运行脚本以获得准确结果。

### Q: 我不想安装 bun，但是 Claude Code 是 npm 安装的怎么办？

建议临时安装 bun 来运行脚本：

```bash
# macOS / Linux
curl -fsSL https://bun.sh/install | bash

# Windows
powershell -c "irm bun.sh/install.ps1 | iex"
```

### Q: 刷完宠物后可以正常登录吗？

可以。但如果你之后用普通方式重新登录（非 OAuth Token 方式），`accountUuid` 可能会被重新写入，`/buddy` 可能会回到你原来的宠物。建议在刷好宠物后**不要**再执行会重写 `~/.claude.json` 的操作。

### Q: 如何恢复原来的配置？

```bash
cp ~/.claude.json.bak ~/.claude.json
```

### Q: 搜索太慢了？

- 减少筛选条件（比如不要同时要求 shiny + legendary + 指定种族）
- 增加 `--max` 值给更多搜索时间
- 使用 `bun` 运行，性能通常优于 `node`
