---
title: "ocr-md-interlock"
publish: false
tags: ["OCR","项目实践"]
---
# ocr-md-interlock

```jsx
cd /home/huang.pengbin/md-interlock && uv run streamlit run [app.py](http://app.py/) --server.port 8501

uv run python -m src.template_matching extract --input "work/SP_Interlock - r1/page_015/block_remainder.png" --coords 1487 1480 40 40 --name flying_a --category special_marks --description "flying lines"
uv run python -m src.template_matching match --input "work/SP_Interlock - r1/page_015/block_remainder.png" --category special_marks --threshold 0.8
```

```
uv run python single_image_line_filter.py \
  ../work/TSS_Interlock\ -\ r2/page_018/block_01.png \
  --bridge-matches ../templates/bridge_matches/block_01_matches.json
```

```jsx
uv run python -m src.template_matching extract --input "work/TSS_Interlock - r2/page_005/block_01.png" --coords 854 627 13 14 --name cross_gen_13 --category special_marks --description "special case for TSS_Interlock - r2/page_005/block_01"

uv run python ./visualize_connections.py "work/TSS_Interlock - r2/page_005/block_01.png"
```
