---
title: "flux"
publish: false
tags: ["OCR","项目实践"]
---
# flux

```jsx
python app.py \
--model_path "checkpoints/model_multisize/pytorch_lora_weights.safetensors" \
--config_path "checkpoints/model_multisize/config.yaml"
```

```jsx
python app_low_VRAM.py \
--model_path "checkpoints/model_multisize/pytorch_lora_weights.safetensors" \
--config_path "checkpoints/model_multisize/config.yaml"
```

```jsx
cd /home/huang/code/pic_gen/FluxText
conda activate flux_text
PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True python quick_start.py
```

```jsx
cd /home/huang/code/pic_gen/FluxText && conda run -n flux_text python quick_start.py --preset samples --text "12345678" --output outputs/samples_board_12345678.png --max_size 1536
```
