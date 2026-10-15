# 模型与ComfyUI工作流接入

本页按作者提供的「AI短剧制作全流程」工作流静态核对，并结合官方模型说明整理（2026-10-15）。文件名反映工作流选项，不代表它们都必需下载，也不代表该组合已实测通过。

## 工作流体验与下载

### 云端版本

| 版本 | 工作流名称 | 体验地址 |
|---|---|---|
| 国内工作流 | 【Work-Fisher】AI短剧制作全流程 | [RunningHub 国内站体验](https://www.runninghub.cn/post/2109735475824123905/?inviteCode=cn-v1078) |
| 国外工作流 | 【Work-Fisher】AI短剧制作全流程 | [RunningHub 国际站体验](https://www.runninghub.ai/zh-cn/post/2110330293312188418/?inviteCode=rh-v1270) |

### 本地版本

**【Work-Fisher】AI短剧制作全流程**：[夸克网盘下载](https://pan.quark.cn/s/dd44d9da44da)

## 制作分工

1. 对话助手加载Skill，读取剧本，输出角色提示词。
2. 在Z-Image工作流制作角色图，确认造型；助手继续输出场景提示词，在Z-Image制作场景。
3. 如果需要修改造型、调整图像或制作多视图，使用Qwen-Image 2.1编辑分支。多视图是可选资产，不是Skill强制要求。
4. 将最终角色图、场景图及必要道具图上传给助手，获得15秒、3–4镜头的视频提示词。
5. 将图片按提示词编号载入H3多参考图工作流，粘贴视频提示词，设置尺寸、时长与采样，生成并检查画面、声音及连续性。

已有确认图片时，可直接从第4步开始。Skill并不自动在ComfyUI里操作、下载模型或执行推理。

## 模型文件与目录

下面目录相对 `ComfyUI/models/`。同一模型的量化版本按显存及运行环境选择，不需要把所有精度都下载。

### Z-Image：角色、场景、道具

| 类型 | 作者工作流中的文件名 | 目录 |
|---|---|---|
| 主模型 | `z_image_bf16.safetensors` | `diffusion_models/` |
| 文本编码器 | `qwen_3_4b.safetensors` | `text_encoders/` |
| VAE | `z_image_turbo-vae.safetensors`（本地命名） | `vae/` |

[Comfy-Org模型页](https://huggingface.co/Comfy-Org/z_image)公开列出的VAE名为 `ae.safetensors`。不能仅凭名称断言它与本地 `z_image_turbo-vae.safetensors` 字节相同。使用官方组合时，下载官方VAE并在节点中选中它；主模型这里是Z-Image基础版，不要因本地VAE命名而误换Turbo主模型。

工作流中还选择了以下LoRA，均放 `loras/`：

- `Kook_Zimage_如梦似幻.safetensors`：作者风格搭配。
- `Kook_Zimage_瑶光.safetensors`：作者风格搭配。
- `Z-Image-Fun-Lora-Distill-8-Steps-2603-ComfyUI.safetensors`：蒸馏加速搭配。

这些不是Skill依赖，本文未核实其公开下载来源。缺少风格LoRA时可旁路相应加载节点；取消蒸馏LoRA时，采样步数与其他参数也需回到对应基础模型工作流，不能原封不动沿用低步数配置。

### Qwen-Image 2.1：编辑与多视图

| 类型 | 文件名 | 目录 |
|---|---|---|
| 主模型 | `qwen_image_2.1_bf16.safetensors` | `diffusion_models/` |
| 已连接的文本编码器 | `qwen3vl_8b_int8_convrot.safetensors` | `text_encoders/` |
| VAE | `qwen_image_2.1_vae_bf16.safetensors` | `vae/` |

[官方ComfyUI模型包与工作流](https://huggingface.co/Comfy-Org/Qwen-Image-2.1)同时提供生成与编辑入口。作者工作流中的 `qwen3.5_9b_qwen_image_2.1_pe_i2i.int8_convrot.safetensors` 是提示词增强模型，节点未接入当前编码链；它不是替代上述8B编码器的必需项。官方说明将PE模型用于Generate Text节点。

### MiniMax H3：多参考图生视频

| 类型 | 作者工作流配置 | 目录 |
|---|---|---|
| 主模型 | `minimax_h3_fl2va_int8_convrot.safetensors`（作者确认使用FL2VA） | `diffusion_models/` |
| 文本编码器 | `qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors` | `text_encoders/` |
| 视频VAE | `minimax_h3_video_vae_fp16.safetensors` | `vae/` |
| 音频VAE | `minimax_h3_audio_vae_fp32.safetensors` | `vae/` |
| 加速LoRA | `minimax_h3_turbo_4步加速_comfyui.safetensors`，本地命名，来源与底模兼容性需核对 | `loras/` |

模型：[Comfy-Org/MiniMax-H3](https://huggingface.co/Comfy-Org/MiniMax-H3)。本项目组合工作流按作者确认使用 **FL2VA**，保留上表主模型选择。下方官方模板是独立的入门路线，不用于替换作者组合工作流的模型配置。LoRA按作者所用底模与配套配置选择。

首次接通可用官方Reference-to-Video模板建立基线，再添加自定义加速。当前没有为本项目实测最低显存、速度或硬件下限；量化、卸载、分辨率与帧数都会影响资源需求。

### SeedVR2：可选高清放大

作者工作流选择 `seedvr2_ema_7b-Q8_0.gguf` 与 `ema_vae_fp16.safetensors`。这是图片资产的可选增强分支，关闭它不会改变Skill三阶段流程。下载位置和模型路径按所安装SeedVR2插件说明操作，本文不把它列为基础生图/生视频必需项。

## 工作流怎么衔接

### 没有作者组合工作流时

使用三个模型各自的官方工作流，也能承接本Skill：

- [Z-Image模型页中的Workflows](https://huggingface.co/Comfy-Org/z_image#workflows)：制作角色、场景和道具图。
- [Qwen-Image 2.1模型页中的Workflows](https://huggingface.co/Comfy-Org/Qwen-Image-2.1#workflows)：选择Image Edit做参考图编辑。
- [MiniMax H3官方工作流入口](https://docs.comfy.org/tutorials/video/minimax/minimax-h3)：选择Reference-to-Video，按模板提示下载匹配模型。

### 已有作者组合工作流时

| 原工作流分组 | 使用方式 |
|---|---|
| 角色资产生成编辑 / Z-image | 粘贴角色提示词，生成并筛选基准图 |
| 场景资产生成编辑 / Z-image | 粘贴场景提示词，生成空间参考图 |
| 三视图生成编辑 | 载入已确认角色图，按需执行Qwen编辑 |
| SeedVR高清放大 | 需要更大图片时启用 |
| 多参生视频编辑 / 输入提示词 / 加载图像 | 按编号装入图片，粘贴视频词，输出视频 |

导出时部分生图/编辑分支处于旁路状态（mode=4），H3分支启用。使用对应分支前需检查分组开关，不能把载入JSON直接视为全链路自动运行。

组合工作流涉及7个自定义插件（含旁路分支）。这是原组合工作流依赖，不代表官方基础工作流也都需要它们：

| 插件 | 对应节点或用途 |
|---|---|
| KJNodes | INTConstant、MiniMaxH3MemoryEfficientSageAttentionPatch |
| VideoHelperSuite | VHS_VideoCombine |
| MiniMax H3 Audio T8 | 作者本机SolAttnMiniMax的提供方 |
| SeedVR2 VideoUpscaler | SeedVR2模型加载与放大 |
| rgthree-comfy | 图像比较、分组旁路 |
| Comfyroll CustomNodes | CR Text、CR Prompt Text |
| Goohaitools-comfyui | 忽略多组孤海 |

本仓库不重新分发上述插件源码。按插件来源安装依赖并重启ComfyUI。`ModelAttentionBackend`、`ResolutionSelector`、`MiniMaxH3ReferenceToVideo`、`TextEncodeQwenImage21`、`ComfySwitchNode` 属于本机核心节点，缺失时先查核心版本与启动日志。

## 导入原工作流后先核对

1. H3主模型保留作者确认的 `minimax_h3_fl2va_int8_convrot.safetensors`（FL2VA）。JSON的下载元数据与实际下拉值不同时，本组合工作流以作者确认的实际选项为准。
2. Qwen编辑编码链实际连接8B文本编码器；未连接的9B PE模型不必为运行该分支下载。
3. Z-Image本地VAE命名与官方不同，按来源选择文件，不猜测二者等价。
4. 没有作者风格LoRA时旁路；去掉加速LoRA同时调整采样配置。
5. 重新加载自己的图片，保持上传顺序与提示词“图像1、图像2……”完全一致。
6. 先短片验证人物、声音与显存，再制作完整片段。当前仓库的测试仅覆盖提示词输出，不证明任意工作流组合已跑通。

## 模型使用许可

这里的“开源模型路线”指使用公开可下载权重进行本地制作，各模型并非统一许可证。请在模型页核对使用及商业授权范围；Skill仓库的发布不替代模型或插件授权。
