---
title: "command set"
publish: false
tags: ["项目实践"]
---
# command set

```jsx
uv run chainlit run main.py --host 0.0.0.0 --port 8080 --watch

uv run chainlit create-secret
chmod +x ./deploy/local_build.sh
chmod +x ./deploy/local_run.sh
```

```jsx
az containerapp show \
  -n ai-interviewer \
  -g chatbot \
  --query properties.configuration.ingress.fqdn \
  -o tsv
```

```jsx
az containerapp ingress update \
    -n ai-interviewer \
    -g chatbot \
    --target-port 8080
```
