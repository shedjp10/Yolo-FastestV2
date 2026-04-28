# 训练权重说明

> 本目录用于存放训练得到的模型权重文件 (`*.pth`)。  
> 权重文件本身不纳入 Git 版本控制（已在 `.gitignore` 排除），仅本 README 入库以记录训练成绩。

## 文件命名规则

```
{model_name}-{epoch}-epoch-{mAP@0.5}ap-model.pth
```

例如：`dms-kaggle-150-epoch-0.699052ap-model.pth` 表示在 `dms-kaggle` 数据集上训练 150 epoch、验证集 mAP@0.5 = 69.91% 的权重。

---

## DMS Kaggle 训练记录

- **训练时间**：2026-04-28
- **训练配置**：`@/data/dms-kaggle.data`
- **训练命令**：`python train.py --data data/dms-kaggle.data`
- **数据集**：DMS Kaggle (5 类：Open Eye / Closed Eye / Cigarette / Phone / Seatbelt)
- **训练集 / 验证集**：5957 / 2389 张
- **输入分辨率**：352 × 352
- **Batch Size**：64
- **优化器**：SGD (momentum=0.949, weight_decay=0.0005)
- **学习率策略**：MultiStepLR，初始 0.001，在 epoch 150 / 250 各衰减 10x
- **总 epoch**：300
- **模型参数量**：239,080（轻量级）
- **环境**：Python 3.10 + PyTorch 2.5.1+cu121 (conda env `DMS`)

### mAP@0.5 演化表

| Epoch | LR       | mAP@0.5    | 备注                |
|------:|:---------|:----------:|:--------------------|
|    10 | 0.001    | 55.83%     | warmup 后首次评估   |
|    20 | 0.001    | 59.77%     |                     |
|    30 | 0.001    | 65.38%     |                     |
|    40 | 0.001    | 65.08%     |                     |
|    50 | 0.001    | 65.98%     |                     |
|    60 | 0.001    | 68.96%     |                     |
|    70 | 0.001    | 68.60%     |                     |
|    80 | 0.001    | 69.72%     | LR=0.001 阶段最佳   |
|    90 | 0.001    | 69.69%     |                     |
|   100 | 0.001    | 68.65%     |                     |
|   110 | 0.001    | 68.20%     |                     |
|   120 | 0.001    | 68.19%     |                     |
|   130 | 0.001    | 69.13%     |                     |
|   140 | 0.001    | 69.44%     |                     |
| **150** | **0.001→0.0001** | **69.91%** | ⭐ **历史最佳** |
|   160 | 0.0001   | 69.58%     |                     |
|   170 | 0.0001   | 69.57%     |                     |
|   180 | 0.0001   | 69.23%     |                     |
|   190 | 0.0001   | 69.58%     |                     |
|   200 | 0.0001   | 69.20%     |                     |
|   210 | 0.0001   | 69.07%     |                     |
|   220 | 0.0001   | 69.17%     |                     |
|   230 | 0.0001   | 69.23%     |                     |
|   240 | 0.0001   | 69.05%     |                     |
|   250 | 0.0001→0.00001 | 69.00% | 第二次 LR 衰减     |
|   260 | 0.00001  | 68.80%     |                     |
|   270 | 0.00001  | 69.08%     |                     |
|   280 | 0.00001  | 68.81%     |                     |
|   290 | 0.00001  | 68.74%     |                     |
|   300 | 0.00001  | —          | 训练结束（不保存）  |

> 训练脚本仅在 `epoch % 10 == 0 and epoch > 0` 时评估并保存，因此最后一个 epoch（299）未单独保存权重。

### 推荐权重

- **🏆 最佳精度**：`dms-kaggle-150-epoch-0.699052ap-model.pth` （**mAP@0.5 = 69.91%**）
- **📦 备选稳定权重**：`dms-kaggle-80-epoch-0.697151ap-model.pth` （第一次 LR 衰减前的高点，泛化性较好）

### 推理 / 评估

```bash
# 推理单张图
python test.py --data data/dms-kaggle.data --weights weights/dms-kaggle-150-epoch-0.699052ap-model.pth --img <path-to-image>

# 在验证集上评估
python evaluation.py --data data/dms-kaggle.data --weights weights/dms-kaggle-150-epoch-0.699052ap-model.pth

# 导出 ONNX
python pytorch2onnx.py --data data/dms-kaggle.data --weights weights/dms-kaggle-150-epoch-0.699052ap-model.pth
```

### 训练观察

- **收敛曲线**：Total Loss 从 ~30 → ~0.74，CIoU/Obj/Cls 三项均稳定下降
- **mAP 平台**：在 LR=0.001 阶段约 80~90 epoch 已接近瓶颈，后续 LR 衰减仅微幅提升
- **过拟合风险**：Epoch 150 之后 mAP 在 ±0.5% 区间震荡，未出现明显下滑，模型未过拟合
- **复盘建议**：若资源充裕，可尝试 (1) 提升输入分辨率到 416/512； (2) 启用 datasets.py 中已注释的更多数据增强 (motion_blur / random_resize / augment_hsv)； (3) 调小 `steps` 让 LR 更早衰减

---

## 历史训练记录

> 后续每次重要训练完成后，请在此追加一节 `## <模型名> 训练记录`，至少包含：训练时间、配置文件、命令、最佳权重、mAP 表格。
