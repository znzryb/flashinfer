# FlashInfer #9396 复现材料

Issue: https://github.com/flashinfer-ai/flashinfer/issues/9396

本分支 `codex/repro-9396` 仅保存复现脚本、环境说明和实验结果，拥有独立的 Git 历史。
修复代码位于同一 fork 的 `codex/fix-9396-silu-strides` 分支。
复现起点为 FlashInfer commit `656eaee454f48479d6f07740670eb3e64a7eb98c`。

`repro.py` 是空白占位文件，由用户编写 Modal 复现脚本。

## 本地编辑环境

本地统一使用外层 FlashInfer 根目录的 `.venv`，Modal SDK 安装在该环境中。
根目录 `.vscode/` 按 dotfiles 的 Python 项目模板配置，并指向根目录 `.venv`。
本复现分支仅保存脚本和复现材料，不单独管理 Python 项目或虚拟环境。

在外层 FlashInfer 根目录运行：

```bash
.venv/bin/modal --version
.venv/bin/modal run local-repro/issue-9396/repro.py
```

第二条命令会启动 Modal 云端任务。

## 建议的编写顺序

1. 定义 `modal.App`、云端 Image、GPU 函数和调用 `.remote()` 的本地入口，先确认云端 GPU 可用。
2. 在带 CUDA Toolkit（含 nvcc）的 devel 镜像里安装固定版本或固定 commit 的 FlashInfer，并记录实际版本。
3. 先比较连续输入与 PyTorch 参考结果，再构造最后一维连续、行间有 padding 的输入 view，打印 stride 并比较输出。
4. 测试带 padding 的显式输出 view，检查有效区域结果及 padding 是否保持原值。

复现目标是 `flashinfer.activation.silu_and_mul` 的 JIT C++/CUDA 路径。
FlashInfer 依赖的导入放在云端函数内；验证源码改动时不要让 `flashinfer-jit-cache` 遮蔽源码。
