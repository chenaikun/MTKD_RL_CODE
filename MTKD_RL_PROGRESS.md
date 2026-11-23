# MTKD-RL 项目进度与跨设备交接

更新时间：2026-11-23（Asia/Shanghai）

## 给新对话的说明

请先阅读本文档，再继续工作。当前目标不是重新调研或从头配置，而是：

1. 继续复现 CIFAR-100 上的 AVER 和原始 MTKD-RL；
2. 确认代码实现与论文公式的差异；
3. 在稳定复现的基础上进行算法改进；
4. 后续研究课题是“强化学习引导的多模态集合智能体知识蒸馏”，面向轨行区/交通复杂场景目标检测。

SSH 密码没有写入本文档，请从私有渠道获取，不要把密码提交到 Git。

## 本地与远端环境

### 本地工作区

```text
D:\code\MTKD_RL_CODE\MTKD_RL_CODE
```

原项目目录：

```text
D:\code\MTKD_RL_CODE\MTKD_RL_CODE\MTKD-RL
```

下载的论文：

```text
D:\code\MTKD_RL_CODE\MTKD_RL_CODE\tmp\pdfs\MTKD-RL-arxiv-2502.18510.pdf
```

本地仓库当前仅有一些未跟踪文件，包括 `.vscode/` 和 `__pycache__/`。核心代码没有在本地做算法修改。

### AutoDL 远端环境

```text
GPU：NVIDIA GeForce RTX 4090D，24 GB
系统：Ubuntu 22.04
Python：3.10
PyTorch：2.1.2+cu118
CPU：16～18 核
内存：约 60～62 GB
数据盘：/root/autodl-tmp，50 GB
价格：约 1.88 元/小时；数据盘约 0.33 元/天
```

远端目录：

```text
/root/autodl-tmp/mtkd/MTKD-RL
/root/autodl-tmp/mtkd/data
/root/autodl-tmp/mtkd/checkpoints
/root/autodl-tmp/mtkd/logs
```

CIFAR-100 已从 AutoDL 公共目录解压到：

```text
/root/autodl-tmp/mtkd/data/cifar-100-python
```

代码目录已通过 GitHub clone 或上传获得。当前环境补装了：

```text
einops 0.8.2
```

没有直接安装完整 `requirements.txt`，避免其固定版本覆盖 PyTorch 2.1.2。

## 已完成：四个教师训练

正式教师 checkpoint 根目录：

```text
/root/autodl-tmp/mtkd/checkpoints/official_teachers/teachers/models
```

四个教师均训练 240 epochs，建议的参考精度与实际精度如下：

| 教师 | 实际 Top-1 | 实际 Top-5 | 实际 Loss | 最佳 epoch | 论文 Top-1 |
|---|---:|---:|---:|---:|---:|
| RegNetY-400MF | 79.20 | 95.15 | 0.8200 | 186 | 78.87 |
| RegNetX-400MF | 78.78 | 94.89 | 0.9000 | 232 | 79.15 |
| ResNet-32x4 | 79.09 | 94.73 | 0.8800 | 218 | 79.59 |
| WRN-28-4 | 79.16 | 94.48 | 0.8900 | 194 | 79.37 |

四个教师与论文的绝对偏差均不超过约 0.5 个百分点，可以继续学生蒸馏实验。

正式 checkpoint 路径：

```text
/root/autodl-tmp/mtkd/checkpoints/official_teachers/teachers/models/RegNetY_400MF_cifar100_lr_0.05_decay_0.0005_trial_official/RegNetY_400MF_best.pth
/root/autodl-tmp/mtkd/checkpoints/official_teachers/teachers/models/RegNetX_400MF_cifar100_lr_0.05_decay_0.0005_trial_official/RegNetX_400MF_best.pth
/root/autodl-tmp/mtkd/checkpoints/official_teachers/teachers/models/resnet32x4_cifar100_lr_0.05_decay_0.0005_trial_official/resnet32x4_best.pth
/root/autodl-tmp/mtkd/checkpoints/official_teachers/teachers/models/wrn_28_4_cifar100_lr_0.05_decay_0.0005_trial_official/wrn_28_4_best.pth
```

远端 `setting.py` 已改为上述四个路径，并已执行路径检查：

```text
RegNetY_400MF True
RegNetX_400MF True
resnet32x4 True
wrn_28_4 True
```

## 已完成：学生 AVER 1 epoch smoke test

使用的配置：

```text
student: ShuffleV2
teachers:
  RegNetY_400MF
  RegNetX_400MF
  resnet32x4
  wrn_28_4
batch size: 64
epochs: 1
seed: 0
feature KD: MSE
```

日志中的教师精度：

```text
Teacher accuracy: [79.21, 78.76, 79.09, 79.17]
```

1 epoch 结果：

```text
Duration: 42.235 秒
Train Loss: 18.31954
Train CLS Loss: 3.91831
Train KD Loss: 10.96257
Train Feature Loss: 3.43867
Train Top-1: 9.17%
Test Loss: 3.74706
Test Top-1: 15.03%
Test Top-5: 42.60%
```

smoke test 输出目录：

```text
/root/autodl-tmp/mtkd/checkpoints/student_avg_smoke/
ShuffleV2_cifar100_rl_avg_smoke_4_RegNetY_400MF_RegNetX_400MF_resnet32x4_wrn_28_423-11-2026_18-28-18/
```

该目录包含：

```text
ShuffleV2.pth.tar
ShuffleV2_best.pth.tar
同名 .txt 日志
```

smoke test 只用于验证流程，不用于正式结果。

## 当前进度

截至本文档更新时间，已经确认：

- CIFAR-100 数据正常；
- 四个教师 checkpoint 正常；
- `setting.py` 教师路径全部正确；
- 1 epoch AVER 流程正常；
- 4090D 上约 42 秒/epoch。

尚未确认正式 240 epoch AVER 已完成。下一步是运行正式 AVER，而不是直接改算法。

## 下一步：正式 AVER

```bash
screen -S avg-full

cd /root/autodl-tmp/mtkd/MTKD-RL

python train_student_avg.py \
  --data /root/autodl-tmp/mtkd/data \
  --arch ShuffleV2 \
  --checkpoint-dir /root/autodl-tmp/mtkd/checkpoints/student_avg_full \
  --teacher-name-list RegNetY_400MF RegNetX_400MF resnet32x4 wrn_28_4 \
  --batch-size 64 \
  --epochs 240 \
  --trial avg_seed0 \
  --seed 0 \
  --dist-backend nccl \
  --world-size 1 \
  --rank 0
```

预计 2.5～3 小时。

论文参考：

```text
ShuffleV2 + AVER(KD+FitNet): 76.83% ± 0.34%
```

如果 AVER 达到约 76%～77.5%，再进行原始 MTKD-RL。如果显著低于 76%，先排查学生训练和特征蒸馏。

## 后续：原始 MTKD-RL

先跑 1 epoch smoke test，不改当前代码：

```bash
screen -S rl-smoke

cd /root/autodl-tmp/mtkd/MTKD-RL

python train_student_rl.py \
  --data /root/autodl-tmp/mtkd/data \
  --arch ShuffleV2 \
  --dynamic \
  --checkpoint-dir /root/autodl-tmp/mtkd/checkpoints/student_rl_smoke \
  --teacher-name-list RegNetY_400MF RegNetX_400MF resnet32x4 wrn_28_4 \
  --batch-size 64 \
  --epochs 1 \
  --trial rl_smoke \
  --seed 0 \
  --dist-backend nccl \
  --world-size 1 \
  --rank 0
```

检查：

- 教师权重不是 NaN；
- agent loss 正常；
- reward 不全为零；
- agent 在 epoch 末正常更新；
- 显存没有持续增长。

然后再跑 240 epochs、seed 0：

```bash
python train_student_rl.py \
  --data /root/autodl-tmp/mtkd/data \
  --arch ShuffleV2 \
  --dynamic \
  --checkpoint-dir /root/autodl-tmp/mtkd/checkpoints/student_rl_full \
  --teacher-name-list RegNetY_400MF RegNetX_400MF resnet32x4 wrn_28_4 \
  --batch-size 64 \
  --epochs 240 \
  --trial rl_seed0 \
  --seed 0 \
  --dist-backend nccl \
  --world-size 1 \
  --rank 0
```

论文参考：

```text
ShuffleV2 + MTKD-RL: 78.39% ± 0.14%
```

## 当前代码中需要核对的实现问题

这些问题已经根据论文公式和代码检查得出。先复现原始代码，之后再修改；不要在没有保存原始版本的情况下直接覆盖。

### 1. reward 与论文定义不一致

论文为每个教师定义样本级 reward：

```text
R_i^m =
    -student_CE_i
    - alpha * KL(student_i, teacher_m_i)
    - beta * FeatureLoss(student_i, teacher_m_i)
```

当前 `train_loops.py` 计算的是加权后的所有教师 KL/特征的混合损失，并在 batch 内做标准化。

### 2. normalized reward 被截断

当前代码：

```python
normalized_reward = torch.clamp(normalized_reward, min=0, max=1)
```

论文公式保留负 advantage。截断会消除“降低某个教师权重”的梯度。

### 3. agent 更新不是论文式策略梯度

当前 `train_agent()` 把标签设为全 1，并使用 BCE：

```python
action_label = torch.ones_like(agent_pred[0])
binary_cross_entropy(agent_pred, action_label, weight=reward)
```

这不是论文式 (7) 的策略梯度。应先保存原始版本，再做单元测试：

- 教师 reward 更高时，其权重应向增大方向更新；
- 教师 reward 更低时，其权重应向减小方向更新；
- 实际 action 必须进入 agent objective。

### 4. 首轮等权损失尺度不一致

第一 epoch 将权重设为全 1，而 AVER 代码将教师损失除以教师数。两者 KD/特征损失的尺度可能相差约 4 倍。

需要在以下方案中保持一致：

- 权重使用 `1/M`；
- 或保持全 1，但相应调整 `alpha/beta`。

### 5. agent 更新频率

默认 `agent_step=1000`，CIFAR-100 每 epoch 约 782 个 batch，因此 agent 实际在 epoch 末更新一次。先保持原始行为，之后再单独实验不同更新频率。

### 6. 未使用的参数

`--ce-weight` 和 `--kd-weight` 已声明但没有参与总损失。`get_actions()` 也没有被主训练路径调用。

## 推荐改进顺序

不要一开始增加复杂 backbone。建议先做实现正确性和稳定性：

1. 按论文修复 reward、归一化和 agent 更新；
2. 做 toy test 验证教师权重随 reward 正确变化；
3. 对教师状态增加 LayerNorm 或统一投影；
4. 增加基于滑动平均 reward 的 advantage；
5. 增加教师权重熵正则，防止权重塌缩；
6. 分别消融每项改动。

更有区分度的研究方向：

```text
教师池数量与顺序自适应的排列不变 MTKD-RL
```

当前策略网络输入维度和教师数量、顺序绑定。可以尝试：

- 每个教师独立编码；
- DeepSets 或 cross-teacher attention；
- 一个 agent 支持 2～5 个教师；
- 测试未见过的教师组合；
- 测试教师缺失、增加弱教师、打乱顺序。

这个方向可能比单纯将 PG 换成 SAC 更有新意。已有 2026 SSRN 预印本提出 SAC + 分层教师权重，因此“只换 SAC”创新不足。

## 新课题：轨行区多模态集合检测蒸馏

导师给出的课题：

```text
强化学习引导的多模态集合智能体知识蒸馏
```

目标场景是轨行区昼夜、雨雪雾霾、隧道遮挡等复杂环境。设想是：

```text
RGB / 红外 / 毫米波雷达
    -> 异构检测教师集合
    -> RL 动态教师/模态权重
    -> 分层特征与检测头知识蒸馏
    -> 单个轻量化边缘检测学生
```

### 课题必须先决定的两个问题

1. 做 2D RGB-IR 检测，还是 Camera-Radar BEV/3D 检测；
2. 学生推理时只使用哪种模态，教师和 RL 策略只出现在训练阶段。

不要直接混用 FLIR/KAIST 与雷达数据。FLIR/KAIST 主要是 RGB-Thermal，通常没有毫米波雷达。轨行区可以优先考虑 OSDaR23 和自建数据。

### DQN 动作空间问题

标准 DQN 只能处理离散动作，不能直接输出四个教师的连续权重。

如果保留层次方案，建议：

```text
高层 DQN：选择可见光优先、红外优先或均衡模式
低层 SAC/DDPG：输出连续教师权重
```

### reward 设计问题

mAP 不是天然可分解的逐样本指标。建议用 episode/批级 reward：

```text
当前 batch mAP 提升
+ 关键障碍物召回提升
- 推理延迟惩罚
- 教师/模态权重不稳定惩罚
```

样本级部分可使用 CE、KL、GIoU、box regression 和置信度误差。

## 真实论文阅读与下载清单

导师给出的 5 个英文题名没有检索到可靠 DOI/正式论文记录，不能直接作为参考文献。应先向导师确认是否为课题申报书中的暂定名称。

### 方法与创新空白

1. MTKD-RL  
   AAAI 2025  
   https://arxiv.org/pdf/2502.18510

2. Re-MTKD  
   AAAI 2025  
   https://arxiv.org/pdf/2504.05224

3. CrossKD  
   CVPR 2024  
   https://arxiv.org/pdf/2306.11369

4. RCBEVDet  
   CVPR 2024  
   https://arxiv.org/pdf/2403.16440

5. CCF: Complementary Collaborative Fusion for Domain Generalized Multi-Modal 3D Object Detection  
   CVPR 2026  
   https://arxiv.org/pdf/2603.23276

6. DGFusion: Dual-guided Fusion for Robust Multi-Modal 3D Object Detection  
   IEEE TCSVT 2025  
   https://arxiv.org/pdf/2511.10035  
   DOI: https://doi.org/10.1109/TCSVT.2025.3628019

7. Robo3D  
   ICCV 2023  
   https://arxiv.org/pdf/2303.17597

### 轨行区数据与工程

8. OSDaR23: Open Sensor Data for Rail 2023  
   ICRAE 2023  
   https://arxiv.org/pdf/2305.03001  
   DOI: https://doi.org/10.1109/ICRAE59816.2023.10458449

9. LiDAR-Camera Joint Obstacle Detection Algorithm for Railway Track Area  
   Expert Systems with Applications 2025  
   DOI: https://doi.org/10.1016/j.eswa.2025.127089  
   需要通过学校 ScienceDirect/Elsevier 权限访问。

10. An Onboard Multimodal Sparse Perception Framework for Railway Intrusion Detection  
    IEEE Sensors Journal 2026  
    DOI: https://doi.org/10.1109/JSEN.2026.3708154  
    需要通过学校 IEEE Xplore 权限访问。

11. Railway-CLIP: A multimodal model for abnormal object detection in high-speed railway  
    High-speed Railway 2025  
    DOI: https://doi.org/10.1016/j.hspr.2025.06.001

### 方案设计

12. MMDistill: Multi-Modal BEV Distillation Framework for Multi-View 3D Object Detection  
    CMC 2024  
    https://cdn.techscience.press/files/cmc/2024/TSP_CMC-81-3/TSP_CMC_58238/TSP_CMC_58238.pdf

13. Bridging Cross-Modal Sparsity via Adaptive Pillar Propagation and Hierarchical Multi-Granularity Feature Distillation for 3D Object Detection  
    Neurocomputing 2026  
    https://doi.org/10.1016/j.neucom.2025.132195  
    SSRN 预印本：https://doi.org/10.2139/ssrn.5390800

建议阅读顺序：1～5 用于判断现有方法和创新空白；6～9 用于设计方案；10～13 用于轨行区数据和工程落地。

## AutoDL 常用操作

查看 screen：

```bash
screen -ls
```

恢复会话：

```bash
screen -r avg-full
```

从 screen 退出但不终止任务：

```text
Ctrl+A，然后按 D
```

查看 GPU：

```bash
nvidia-smi
```

查看磁盘：

```bash
df -h
```

当前 50 GB 数据盘足够 CIFAR-100、四个教师、学生 checkpoint 和日志。完整 ImageNet 不够，需要扩容到至少 300～500 GB。

## 预算参考

按 1.88 元/小时：

| 阶段 | 预计费用 |
|---|---:|
| 四个教师训练 | 14～17 元 |
| AVER + 原始 MTKD-RL 各 1 seed | 25～35 元 |
| ShuffleV2 两种方法各 3 seeds | 50～70 元 |
| 4 学生、3 seeds 的基础论文实验 | 200～300 元 |
| 包含改进、消融和少量 ImageNet | 400～600 元 |

需要预留 20%～30% 的调试和重复训练预算。

## 防止重复劳动

新对话继续时请先检查以下事实：

1. 是否已经完成正式 AVER；
2. 是否已经运行原始 MTKD-RL；
3. 当前 `setting.py` 是否仍指向四个 official 教师；
4. 是否已经对 `train_loops.py` 做修改；
5. 若已修改，先找到原始备份或 Git diff；
6. 不要重复训练已经确认正常的四个教师。

推荐检查命令：

```bash
find /root/autodl-tmp/mtkd/checkpoints -maxdepth 2 -type d | sort
screen -ls
git status --short
```
