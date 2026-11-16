# Agent Sprite Forge 大更新稽核与实作

日期：2026-10-05。范围：这个 repo 的 sprite、map、video 技能与处理工具，
加上新的 API 生成入口。这是本机工作树的更新纪录；不是已发布的 GitHub release。

## 建议的方向

把 Forge 定位成 **agent 可以运行的游戏美术制作流程**：

```text
玩法／相机／美术规格
  → 核准的角色、场景、道具原画
  → 按用途选：静态图／sprite sheet／AI 视频／程序特效
  → 去背、固定座标、动作切段、地图几何与碰撞
  → 游戏可读取的素材、metadata、预览与 QA
```

AI 视频值得正式加入，但不应把所有元素都改成视频。角色待机、流畅走路、
怪物呼吸、树叶、水面、火光适合试；精准像素逐格动作、地形、UI 和可即时
调整的粒子特效仍各有更合适的方法。地图本体宜保持清晰和稳定，局部动画
放在独立图层。游戏中是否命中、造成多少伤害，仍由游戏规则决定。

## 已确认的问题与修正

| 范围 | 旧问题 | 这次修正 |
| --- | --- | --- |
| 技能入口 | Sprite/map 指令过长；工具名称与 magenta 路径写死 | 短入口＋按需参考文档；host tool／API 可选 |
| 视频生成 | 宣称只支持 Grok Build，Codex/Claude 必须拒绝 | 原生工具、正式 API、现有视频三条路 |
| 透明图片 | RGBA 用自己当 mask 再贴一次，透明度被重复相乘 | 保留 straight alpha；添加明确 native-alpha 路径 |
| Sprite 品质 | pixel art 强制平滑；connected-component 筛选不精确 | 可选 nearest；修正最大组件与杂点过滤 |
| 动画定位 | 每帧 bbox 裁切再各自缩放，造成大小抽动和跳跃失真 | 同一 clip 固定 envelope／尺度；保留来源座标与 root |
| Sprite／map QC | 拒绝的素材可能已写到完成目录 | 先验证再发布；拒绝混用旧输出 |
| Terrain | 重复 row index、零尺寸、矩形静默裁成方形 | 提前验证；裁切需明示，保留原生透明 |
| Props | slug 重复覆盖、格线余数被忽略、透明紫色可能被删 | 先验证命名／尺寸，选定去背模式 |
| 地图预览 | 前景／角色可能被忽略；像素素材被模糊缩放 | 分层合成与可选采样；区分视觉、碰撞与遮挡 |
| 手机素材 | 「能播放 MP4」被误认成「有透明」 | VP9 alpha、packed-alpha H.264、poster／PNG fallback 与 runtime 契约 |
| API 工作 | 没有可恢复 job／请求纪录 | dry-run、单次付费提交、保存 ID／hash、GET-only resume |

本次参考了 repo 中已有流程、同机已改进的技能版本，以及暮灯渡实际使用的
sourceSize/sourceAnchor、局部背景视频和手机素材契约。没有把游戏的硬编码
尺寸、角色数或机器路径搬成所有项目的规定。

## 四个技能各自负责什么

| 技能 | 边界 |
| --- | --- |
| `generate2dsprite` | 原画、离散姿势、角色一致性、透明图、固定动作／atlas |
| `generate2dmap` | 玩法几何、场景结构、props、卷轴层、碰撞／遮挡／转场 |
| `video2dsprite` | 动作视频加工、固定定位、透明封装、循环诊断、runtime handoff |
| `generate2dmedia` | 选定 API 的请求、生成、状态追踪与下载；不假装完成游戏 QA |

不增加游戏 runtime 对生成服务的依赖。玩家端使用做好并验证的素材；API key
只存在制作端。若要即时生成的 infinite game，是另一个具有延迟、成本与内容
管理需求的系统，不应混进这个资产工具的缺省流程。

## API 调研与落地

本次已加入 OpenAI Images、xAI Imagine Image 与 xAI Imagine Video 的转接器。
Google Veo／Gemini video、Runway 目前是调研候选，没有冒称已实作。
完整来源、日期、价格样本与能力限制见
[provider-survey.md](../skills/generate2dmedia/references/provider-survey.md)。

xAI full 1.5 的首尾帧控制可以用来试循环；同一张图放在首尾仍需检查中间
动作、速度与接缝。API 与 Grok Build 原生 tool 的参数不同，订阅额度也不能
当成 API 的免费额度。新 CLI 不暗中换模型或付费重试。

新 API 使用范例与中断恢复见
[api-usage.md](../skills/generate2dmedia/references/api-usage.md)。

## 地图应怎么做

先决定可走区、入口出口、镜头、角色尺度、桥与平台，再生成对应的美术。

- **RPG 探索**：可编辑 tilemap 或地形底＋独立 props；碰撞取树干／建筑 footprint，
  不是整张透明图的边框。角色后方与前方遮挡分开处理。
- **2D 卷轴**：平台是碰撞几何；远、中、近景根据相机活动范围计算覆盖，
  不用「一律两张画面宽」猜测。
- **HD-2D 战斗**：稳定高解析背景＋局部水、火、雾、树叶动画；人物站立区与 UI
  安全区先规划。整张背景视频是可选方案，需接受输出分辨率与压缩代价。
- **跨地图探索**：chunk 接口要定入口宽度、方向、spawn／trigger 区与碰撞；
  图片接近并不代表游戏能连通。

目前工具能生成／检查素材与场景数据，**不是通用地图编辑器或完整游戏引擎**。
Unity、Godot、Three.js、Phaser 的实际碰撞与播放集成仍需在目标游戏验证。

## 兼容性与迁移

- 保留原有主要 CLI；sprite 原有 `chroma_key`／`lanczos` 缺省仍可用。
  新 RGBA 建议明确指定 `native_alpha`；像素素材选 `nearest`。
- 视频 sample 缺省改用固定 envelope；只有重现旧结果才用
  `--registration legacy-per-frame`。正式视频封装从尚未缩放的 clean frames 开始。
- `sourceSize`／`sourceAnchor` 描述原画，不随手机版编码缩小。裁切后用
  `sourceRect` 对应回原画空间。
- Packed MP4 要经 compositor 还原 alpha，不能直接当透明 `<video>` 叠上去。
  附带 Canvas2D 范例便于验证；多角色场景需共享 WebGL／decoder 与生命周期管理。
- 保留四个 sibling skill 目录，避免只拷贝一个入口却漏掉共用处理器。
- 这次未覆写用户全域安装的 skills，也未修改暮灯渡游戏或部署网站。

## 验证与下一步

验证明细见 [validation-2026-10-05.md](validation-2026-10-05.md)。
本轮有脱机 API 合约测试、处理器回归测试、真实 ffmpeg encode/decode 与旧示范
素材端到端加工；没有花费 API 额度、没有新模型画质／速度排行榜，也没有
实机 iPhone FPS 结果。

公开发布前最有价值的下一步是固定四个相同输入的 showcase：角色走路、怪物
攻击、树木待机、局部水／火。各供应商记录「含重试的每个合格 clip 成本」，
并同时展示游戏尺寸、透明边缘、首尾接缝与手机 frame time。这比只列更大模型
名称或更多支持平台，更能让开发者判断 Forge 是否适合自己的游戏。
