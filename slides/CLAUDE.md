# slides 模块（硕士答辩 PPT）

## 模块定位

硕士学位论文答辩 PPT，目标时长 15–20 min（讲 ~16 min + Q&A ~4 min）。论文 SoT 在 `cn/`（中文）和 `en/`（未来），slides 内容必须与论文一致。

## 文件结构

```
slides/
├── CLAUDE.md           # 本文件
├── docs/
│   └── spec.md         # PPT 大纲 + 模板选择 + demo 嵌入策略（SoT）
├── templates/          # zarazhangrui/beautiful-html-templates clone（MIT）
│                       # 32 个候选模板，挑了 signal
├── cn/                 # 中文版 deck（最终交付）
│   └── mockup-deck/
│       └── index.html  # S1/S6/S13 mockup（已就绪，待用户验收）
└── assets/
    └── demos/          # 视频/GIF 素材（s10/s11/s12/s13/s14 各场景）
```

## 模板选择：Signal

- repo: `templates/templates/signal/template.html`
- 风格：deep navy `#1c2644` + 暗金 `#c8a870` + Source Serif 4 + DM Sans + IBM Plex Mono + Noto Serif/Sans SC（中文 fallback 完备）
- slide_count = 18（与大纲匹配）
- formality = high, density = high, occasion 列表含 "academic deck" + "bilingual EN/CN deck"
- 修改原则：ZONE A (tokens) 改色/字号；ZONE B (engine) 不动；ZONE F 加 component；自定义 layout 放 ZONE G

## demo 嵌入约定

- 每段视频 5–12 s, mp4 (h264) 或 GIF (≤2 MB, 480p, 10 fps)
- 命名：`s{slide-no}_{task}_{condition}.mp4`，例如 `s13_pickup_col2_collapse.mp4`
- 嵌入：`<video autoplay loop muted playsinline>` 或 `<img src="*.gif">`
- 整 deck 资源总量 < 150 MB

## 工程纪律

- 每张 slide 在 `docs/spec.md` 中有 1 行要点 + 主视觉描述（SoT）
- 改 slide 内容前先确认 spec.md 对应行，不一致则先更新 spec.md
- 不在 mockup 阶段 fill all 18 slides——先验证 3 类信息密度（封面/框架图/数据矩阵）模板能 hold 住，再展开
