---
title: "Python模块导入的搜索机制"
publish: false
tags: ["OCR","项目实践"]
---
# Python模块导入的搜索机制

import sys
from pathlib import Path

# 添加 src 目录到 Python 路径

project_root = Path(**file**).parent.parent
src_path = project_root / "src"
sys.path.insert(0, str(src_path))

# 导入时直接使用 src 下的结构

from back.ocr import OCRClient
