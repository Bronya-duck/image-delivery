# image-delivery · 图片交付助手

为 Codex 提供本地图片命名、复制归档和查询能力。保留源图，把图片交付到用户指定的项目目录，并维护可查询的 `image-manifest.json`。

可独立处理现有图片和其他生图工具的输出；也能读取 [Comfy Series](https://github.com/Bronya-duck/comfy-series) 的任务记录，选取评审通过的图片。独立安装无需 ComfyUI、模型或生图服务。

## 安装

需要 Windows、Git、Python 3.12 或更新版本。依赖仅为 Pillow。

```powershell
git clone https://github.com/Bronya-duck/image-delivery.git
cd image-delivery
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
powershell -NoProfile -ExecutionPolicy Bypass -File .\install.ps1
```

没有 Python Launcher 时，用自己的 Python 3.12+ 执行文件替代 `py -3.12`。`ExecutionPolicy Bypass` 仅作用于上面的安装进程。

安装器创建当前用户的 `.agents\skills\image-delivery` 目录联接，指向本仓库。请保留克隆目录。重复安装会复用正确入口；同名入口指向其他位置时，会保留原入口并报告冲突。已随 Comfy Series 安装此技能的用户可以继续使用已有版本；若要切换到本仓库，先自行处理已有入口，再重新安装。

在 Codex 中开启新会话或重新启动以刷新技能列表，然后调用 `$image-delivery`。后续更新运行 `git pull`。

## 使用

```text
$image-delivery
把 D:\我的素材\背景.png 和 D:\我的素材\封面.jpg 交付到我的游戏项目。
保留原图，统一用“场景”作前缀，用途是游戏背景。
```

AI 先检查来源，再展示交付模板，等待你本次答复；已提供的内容可以预填：

```text
目标文件夹：D:\我的游戏\images
文件名：统一前缀“场景”
用途：游戏背景
```

这会生成 `场景_001.png`、`场景_002.jpg` 和 `image-manifest.json`。单张图片可填写一个名称，多张也可逐张填写名称列表。源图保持原样，目标按字节复制，格式和元数据保留。

- 名称未写扩展名时按实际格式补齐；非法名称或格式冲突会返回问题。
- 同名且内容相同会复用，内容不同会追加编号；重复执行不会持续生成副本。
- 清单记录用途、哈希、尺寸、格式和源到目标的映射。部分失败时保留成功项并报告。
- 后续请求“在 D:\我的游戏 中找到游戏背景”即可查询清单，并检查文件存在性和内容是否变化。项目整体搬迁后，相对目标路径仍可定位。

## 可选：衔接 Comfy Series

新版 Comfy Series 已包含交付衔接。旧版可执行：

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\connect_comfy.ps1 -ComfyProjectRoot 'D:\你的ComfySeries项目'
```

此命令更新 Comfy Series 的安装源和当前项目技能副本，在每次任务完成评审后调用 `$image-delivery`。每轮生成重试不会分别触发交付。工具只读取任务和轮次记录，默认只交付验收通过的版本；没有通过结果时不交付。已存在的相同衔接会复用。

详见 [Comfy 适配说明](references/comfy-series.md)。

## 工具接口

请求通过 UTF-8 JSON 文件传入，结果返回 JSON：

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\run.ps1 inspect --file 'D:\任务\inspect.json'
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\run.ps1 deliver --file 'D:\任务\deliver.json'
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\run.ps1 find --root 'D:\我的游戏' --query '游戏背景'
```

`inspect` 列出实际格式、尺寸和来源；`deliver` 执行确认后的复制；`find` 仅扫描指定项目内的交付清单。完整请求字段、状态和规则见 [工具接口说明](references/tools.md)。

运行器依次检查 Codex 共用运行时、本仓库 `.venv`、已安装 Comfy Series 环境及系统 Python。也可用 `-PythonPath` 指定包含 Pillow 的 Python 3.12+：

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\run.ps1 -PythonPath 'C:\Python312\python.exe' find --root 'D:\我的游戏'
```

技能入口为 [SKILL.md](SKILL.md)。首版提供命名、复制归档、批量交付、来源记录和查询；转格式、压缩、删除及原件移动留待后续扩展。

## 验证

```powershell
.\.venv\Scripts\python.exe -B -X utf8 -m unittest discover -s tests -v
```

测试覆盖中文与空格路径、单张和批量、名称冲突和重复执行、文件及清单失败、并发写入、Comfy 多轮验收、哈希保持，以及独立安装和运行时发现。

## 许可证

本独立项目使用 [MIT 许可证](LICENSE)，可使用、修改和再分发。Comfy Series 为可选集成，按其仓库的许可说明使用。
