---
name: mv-character-assets
slug: mv-character-assets
displayName: "MV Character Assets (角色资产的数量经济学与一致性)"
category: video
version: 1.0.0
description: "How many character/background images an MV actually needs, and how to generate them without the face drifting: derive the list from the storyboard instead of a generic pose library (measured: 17 poses + 6 alternates + 14 backgrounds + 1 prop covered ~70 shots, roughly one asset per 3-4 shots), lock one master reference, paste a verbatim character sentence naming the SIDE of asymmetric details, choose the chroma key from the character's own palette, and normalise native alpha that tops out at 252-254. Use when planning an asset 派单/batch, when poses keep coming back as slightly different people, or when a cutout has a ghost of the background bleeding through the body."
description_zh: "一支 MV 到底需要多少张角色/背景图，以及怎么生成才不漂脸：按分镜反推清单而不是做通用姿势库（实测 17 个姿势 + 6 个备选 + 14 张背景 + 1 个道具覆盖约 70 个镜头，约每 3–4 镜一张）；锁一张 master 参考图；把一句角色描述逐字贴进每个 prompt 并写明不对称细节的左右；抠像的 key 色按角色自己的色板选；原生 alpha 顶到 252–254 时必须归一化。"
agent_created: true
---

# 角色资产：数量经济学 + 一致性方法

## 0. 最重要的一条：资产数按分镜反推，不按「动点」算

**实测对照**：约 70 个镜头只用了 **17 个姿势（+6 个备选）+ 14 张背景 + 1 个道具**
≈ **每 3–4 个镜头一张图**。

原因：同一个镜头素材会被引擎**反复重新取景、重新调色、重新打光**（多层视差、LUT、剪影、轮廓光、分屏、爆破）。

⇒ ⛔ **「15 个动点就要 15 张图」是错的。** 我们要的不是 15 个姿势，是 **6–8 个能覆盖全部镜头功能的姿势**：

| 类别 | 数量 | 用途 |
|---|---|---|
| master 参考 | 1（+2 备选） | 标题、残像、回顾 |
| 唱歌 | 2–3 | 副歌砸字、分屏、爆破 |
| 安静/情绪 | 3–4 | 主歌、呼吸镜、结尾 |
| **背影** | 1 | 交代环境、小人物在大世界 |
| 全身动作 | 3–4 | 高潮、跑、转身、伸手 |
| 身体语言 | 2–3 | 抱膝、蹲下拾取 |
| 手/特写 | 2–4 | 指先、火、拾起 |
| 背景 | 8–14 | 每个 illust 镜 |
| 道具 | 1 | 歌名/核心隐喻 |

⭐ **背影和"只有天空的底板"永远有用**，复用率最高。

---

## 1. 一致性方法（四步，缺一步就漂脸）

### 步骤 1：角色圣经 → **一句话角色描述**
把设定压成**一句**，之后**逐字**贴进每个 prompt。

⛔ **必须写明不对称细节的左右**（"ribbon on her **LEFT** wrist"）—— 模型会自己翻面。

例（可直接改）：
```
The same original character as in the reference image (waist-length ink violet-black hair with
sunset-orange inner colour, blunt bangs, amber-gold eyes, white headphones with glowing orange
ring lights, hairpin on her RIGHT side, navy sailor blouse with vermilion tie, cream knit
cardigan, navy pleated skirt, brown loafers, long vermilion ribbon tied on her LEFT wrist).
```

### 步骤 2：三张 master 候选 → 锁一张
全身、平姿势、三个不同角度/表情，**不加参考图**，用完整圣经 + 风格块。
做 contact sheet，**让人挑一张**（这是必须的人工检查点），存成 `REF_master.png`。

### 步骤 3：每个姿势 = 参考图 + 姿势描述 + 逐字角色句 + 风格块
参考图承担脸和比例；角色句承担模型会漂的细节。两者都要。

### 步骤 4：难姿势跑两遍
**不寻常角度（俯视、躺、透视缩短、手）会静默退回站姿。** 一律两遍，挑好的。
实测踩过的：`lying seen from above` 回了一张**站着**的；极端特写当抠图要，回了一张漂浮的脸。

---

## 2. ⛔ 抠像 key 色必须按角色色板选

默认绿 `#00FF00` **只在这些条件成立时可用**：
> 角色设计里**没有任何** G > max(R, B) 的颜色 —— 不能有 teal / aqua / mint / lime / emerald。

青色系、水色系、薄荷绿（夏季/海边的角色很常见）⇒ **改用洋红 `#FF00FF`**。
蓝色 `#0000FF` 只在设计里没有蓝色时用。

⚠️ 绿色 key 压 teal/mint 会变灰；洋红 key 压肤色最惨。

⚠️ **溢色只能在软边带里去掉**（绿 key：软边处 `G ≤ max(R, B)`）。
全局 clamp 会把整片 teal/mint 和所有肤色一起毁掉。

---

## 3. ⛔ 原生 alpha 会顶到 252–254，必须归一化

模型返回的原生透明图，**不透明身体的 alpha 常常只到 252–254**（差 1%）。
后果：**背景会透过身体透出来**（尤其深色背景上）。

修法：
1. 用身体 alpha 的 **99.5 百分位** 拉到 255
2. alpha ≤ 3 的置 0
3. 把颜色**溢进**透明像素（否则双线性缩放会把绿/黑拉进边缘）

---

## 4. 特写与背景的规格

| 类型 | 规格 | 说明 |
|---|---|---|
| 背景 | **1672×941**（16:9） | 高清平滑下，**cover 放大到 ~1.45× 仍然干净**（在 line boil + grain 之下） |
| 更深的推近 | **单独出一张特写** | 不要靠放大全身图裁脸 |
| 极端特写 | **不透明 16:9 满幅**，⛔ 不要抠图 | 抠图会得到一张漂浮的脸 |
| 道具 | 透明、同风格块 | |

⛔ **背景里不要人和可读文字** —— 模型会写假假名。文字一律由排版层出。
⛔ 背景要留构图余地：低地平线（天空镜）、中间空路（小人物）、明确光源（用来钉 glow / 灯闪 / 太阳位置）。

---

## 5. 出图彩板的取舍

- **在彩色里生成**，用色板词（"ink violet-black, dusk magenta, vermilion, sunset orange, amber, cream"），
  再在后处理用 LUT（mix 0.10–0.35）统一。← 推荐
- **只出灰阶**（黑线 + 3–5 个平灰），之后再完全控制调色。← 适合会被重度调色的层（多层视差条、道具）

⚠️ 色板词会**改变画面时段**：写 dusk 的词，整张图会被拉向黄昏。做白天片必须换词
（"high-key summer daylight, saturated sky cyan, coral and sunflower accents, no dusk tones"）。

---

## 6. 实测产能（用于排期）

- **单张 31–115 秒**（背景中位 ≈55s，角色中位 ≈65s）
- 5–8 个并行不触发限流
- **45 张 ≈ 10–15 分钟**实际耗时

⇒ 17 个姿势 + 14 背景 ≈ 30 分钟出完，**不是几天**。

---

## 7. 反面清单（全部实测）

1. **master 没定就批量出姿势** ⇒ 五个略微不同的女孩。
2. **在批次之间改了角色句** ⇒ prompt 漂移。角色句只存**一个文件**，程序化拼进去。
3. **原生 alpha 252–254 不归一化** ⇒ 背景透过身体。
4. **极端特写做成抠图** ⇒ 漂浮的脸。
5. **模型在画面里写字/logo/签名** ⇒ prompt 里每次都写 "no text, no watermark"。
6. **像现有角色**（耳机 + 双马尾 + 青绿 = 麻烦）⇒ prompt 里明写 "ORIGINAL, not any existing character"。
7. **`--force` 覆盖掉已选中的那张** ⇒ 把 keeper 先改名或复制到 `alt/`。
8. **资产出太多** ⇒ 每张约 1 分钟 + 复核时间。**先把现有图重新取景/调色/剪影，再考虑出新图。**

---

## 8. ⭐ 只有一两张图怎么做出动画 —— 手塚那套办法

1963 年《铁臂阿童木》每周一集、**总共只有 1500 帧**（今天一集要 4000+ 张）。
他们没扩产能，而是发明了「リミテッド・アニメーション」。**这套办法正是为我们这种"素材极少"的处境准备的。**

### 四个可以直接抄的手段

**① 单张画靠运镜看起来在动**
不是让角色动，是让**画面**动（推、摇、裁切变化）。一张立绘 + 一个缓慢推近 = 一个镜头。

**② 只动该动的部位，其余完全静止**
脸不动，只动眼和嘴。今天的对应做法：整张图静止，只让**一个局部**区域有变化。

**③ 口型只要 3 张，眨眼只要 2 张**
- 口型：**闭口 / 张口 / 中间**。⛔ 不是 5 个元音 + 闭口共 6 张。日本动画实际就画 3 张。
- 眨眼：**开 / 闭**两张，**中间不加过渡**。帧被"跳过"的顿挫感反而对味。

⇒ 我们做口パク/眨眼时，不要追求平滑过渡，**要追求帧数少带来的顿挫**。

**④ `bank system`（ライブラリ）—— 官方许可的素材复用**
手塚定下的规矩：同一角色在**相似场景就复用同一张画**。为此专门建了"图书馆"（当年 cut bag 上写的就是 `lib`）。
⇒ **"重复用图"在动画工业里是有正式名称的正当做法，不是偷懒。**
我们的 15 个动点复用 2 张图，方向上没错，缺的是**用法**（下面 §9）。

## 9. ⭐ limited animation 当风格，不是省钱

现代观点：limited animation 是**表现手段**——held pose（保持姿势）、repeated cycles、sharp cuts、
sliding backgrounds、selective movement 都是语言。

**最狠的一招：静止 + 一瞬高频。**
> 一个长镜头里角色几乎不动（'threes' = 每 3 帧一张新画），
> 突然**一个眨眼用 'ones'（每帧都画）**——
> 在静止的背景下，这一瞬间的高频会显得内心极度挣扎。

⇒ 手法：**平时 'twos/threes'，情绪爆发点切到 'ones'，只持续 2–4 帧**。
这比"全程流畅"有力得多，而且成本几乎为零。

## 10. 枕词镜（pillow shot）—— 让片子呼吸

切到**空走廊 / 云 / 木地板纹理 / 远处的列车 / 停在枝上的蝉**——没有角色、没有情节推进。
作用：时间与空间的标记、让场景"呼吸"、散掉上一场的情绪。

⇒ **我们完全没有这类镜头。** 而它正好是"不需要角色素材"的镜头类型——
一张背景 + 缓慢横摇就够了。**这是解决我们素材荒最便宜的一整个镜头类别。**

⚠️ 但它走在钢丝上：没有目的、停太久、配一个通用环境音，就会变成拖沓。

## 11. 换姿势不换脸：内容与身份分离（工具侧）

如果将来要走生成路线，实测有效的组合是 **ControlNet(OpenPose) + IP-Adapter(FaceID)**：
- ControlNet 管**姿势**（骨架从一张姿势参考照提取）
- IP-Adapter 管**身份**（脸从角色参考图注入）
- **单用任一个都不行**：只有 ControlNet → 动作对但脸随机；只有 IP-Adapter → 脸像但姿势失控

参考参数：ControlNet strength **0.9**、start 0.0、end 0.8；IP-Adapter weight **0.85**。
（用 DWPose 比 OpenPose 在复杂姿势/遮挡下更准。）

⚠️ 这条需要本地 SD + 模型 + GPU，与我们「只用 Miora」的规矩冲突 ⇒ **仅备案，不动手**。

