1. 专业的医学影像 AutoML 框架

普通的 AutoCV 工具多用于猫狗分类，而医学图像有其专属的自动化利器：

MONAI (Medical Open Network for AI)：这是目前医疗深度学习的行业标准。MONAI 拥有 MONAI Auto3D 等自动化组件，专门为医学图像（X光、CT、MRI）定制。它支持自动化数据增强、预处理 pipeline 搜索，非常适合处理 X 光的去噪与超分任务。

1. 针对图像恢复（Restoration）的 Auto-Pipelines

去噪和超分在视觉中属于“图像恢复”任务。你可以使用以下自动化架构搜索（NAS）工具：

Neural Architecture Search for Image Restoration (NAS-IR)：这类技术专门自动寻找最适合去噪和超分的 U-Net 或 Transformer 变体结构。

MMPose / MMEditing (现更名为 OpenMMLab PlayGround)：商汤科技开源的 OpenMMLab 平台 提供了强大的去噪与超分预训练模型库（如 Real-ESRGAN、SwinIR）。虽然不是纯粹的傻瓜式 AutoML，但它支持自动化配置文件（Config），只需修改几行参数，就能自动组合不同的去噪与超分算子。
