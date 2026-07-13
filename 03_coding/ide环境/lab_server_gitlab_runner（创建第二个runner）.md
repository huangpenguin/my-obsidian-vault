## 阶段一：GitLab 网页端操作（生成专属身份令牌）

_此步骤由你或拥有项目 Maintainer 权限的人在 GitLab 界面上执行。_

1. **进入设置：** 登录 GitLab，进入你的项目仓库，在左侧导航栏依次点击 **Settings -> CI/CD**。
    
2. **创建新 Runner：** 展开 **Runners** 区域，点击右上角的 **New project runner** 按钮。
    
3. **配置标签（核心）：**
    
    - 在 **Tags** 框中，输入原有的通用标签以及你的专属标签，用逗号隔开：`linux, docker, main_gpu, storage_huang`。
        
4. **生成令牌：** 点击 **Create runner**。页面会跳转并生成一串以 `glrt-` 开头的独特 **Token（令牌）**。
    
5. **保存 Token：** 复制这串 Token，发给掌握 `gpu01` 机器 root 权限的管理员。
    

## 阶段二：宿主机 `gpu01` 服务器配置（管理员执行）

_此步骤需要管理员通过 SSH 登录 `gpu01` 服务器，修改 Runner 守护进程的配置。_

1. **打开配置文件：**
    
    Bash
    
    ```
    sudo vim /etc/gitlab-runner/config.toml
    ```
    
2. **追加专属配置块：** 保持原有的通用 `[[runners]]` 块完全不动（确保别人的 Job 不受影响）。直接滚到文件最末尾，**追加**下面这段全新的配置：
    

Ini, TOML

```
[[runners]]
  name = "gpu01-storage-huang-runner"
  url = "https://your-gitlab-domain.com/"     # 替换为你们公司的 GitLab 实际域名
  token = "glrt-xxxxxxxxxxxx"                 # 贴入刚刚在阶段一生成的专属 Token
  executor = "docker"
  
  # 【现代化安全审计核心】
  # 强制该容器内所有进程以用户 huang (UID 1003) 的身份运行，完美解决 root_squash 限制
  user = "1003:1003" 

  [runners.custom_build_dir]
  [runners.cache]
    MaxUploadedArchiveSize = 0
  [runners.docker]
    tls_verify = false
    image = "ubuntu:22.04"                    # 默认兜底镜像
    privileged = false                        # 必须为 false，非特权模式确保主宿机安全
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    
    # 【卷挂载：严格按大厂安全标准路由】
    # 将个人 NFS 目录挂载进容器的 /home 路径
    volumes = ["/cache", "/mnt/data:/data:ro", "/mnt/home:/home:rw"]
    
    # 显卡训练大厂标准必备参数
    shm_size = 68719476736                    # 共享内存设为 64GB，防止 PyTorch DataLoader 报错
    gpus = "all"                              # 允许该容器调用物理 GPU
```

3. **重启服务使其生效：**
    
    Bash
    
    ```
    sudo gitlab-runner restart
    ```
    

## 阶段三：项目代码配置与上线（你来执行）

_此步骤在你的本地开发分支修改，完成后推送到 GitLab 触发流水线。_

1. **修改 CI 配置文件：** 打开你项目中的 `.gitlab-ci.yml`（或关联的 `train.yml`）。
    
2. **升级 Job 模板：** 在你的训练 Job（或基础 Base 模板）中，**追加专属 Tag**。同时确保 `resource_group` 锁依然存在：
    

YAML

```
.gpu_docker_base:
  stage: train
  image: your-registry.com/ai/cuda-train:12.1-cudnn8  # 你的企业内部私有镜像
  
  # 【全局排队锁】不同 runner 同属此 group，依然会乖乖串行排队，绝不炸显存
  resource_group: gpu-training 
  
  tags:
    - linux
    - docker
    - main_gpu
    - storage_huang  # 👈 关键：多加这一行，GitLab 就会把你的 Job 精准路由到 #2 号安全 Runner

  script:
    - echo "=== 正在验证运行身份 ==="
    - id && whoami   # 预期输出：uid=1003(huang) gid=1003(huang)
    - echo "=== 开始训练并直接写入个人 NFS 目录 ==="
    - python train.py --output_dir /home/huang/cst_ai/ci_outputs/job_${CI_JOB_ID}
```

3. **提交代码：**
    
    Bash
    
    ```
    git add .
    git commit -m "infra: switch to identity-bound non-root runner via storage_huang tag"
    git push origin your-branch
    ```
    

## 验证与验收标准

当流水线跑起来后，你可以通过以下两个细节确认新架构已经完美闭环：

- **检查项 1（看日志顶部）：** 在 GitLab 的 Job 运行日志第一行，应该能看到 `Running with gitlab-runner ... on gpu01-storage-huang-runner`，这说明路由完全正确。
    
- **检查项 2（看文件属主）：** 训练结束后，你在终端去查看 NFS 里的 `ci_outputs` 目录，生成的文件和文件夹，它们的 Owner 应该自然且干净地显示为 `huang:huang`，而不再是令人头疼的 `nobody` 或 `root`。