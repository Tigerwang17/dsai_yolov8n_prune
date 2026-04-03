本实验使用并对比三种prune方法：
1 非结构化剪枝（Unstructured）
在微调模型上，对除检测头外的每个 Conv2d 的 weight 做 L1 非结构化剪枝，将较小权重视为可删并置零，再用 prune.remove 写回稠密张量；张量形状不变，主要靠稀疏度改变表示，若不配合稀疏推理库，体积与速度往往不明显下降。
2 指定层位联动结构化剪枝（Preset-layer coupled）
人工指定若干 m.model[i]（含 conv+bn 的模块），按固定比例保留前若干通道（range(keep_count)），重建更小 Conv/BN，并同步修改第 i+1 层输入通道以保持维度一致；属于 一次性、非重要性排序的物理通道剪枝，覆盖层少时参数量变化可能很小。
3 宽度系数剪枝（Width scaling）
通过降低 width_multiple 得到更窄的 YOLO 结构，再将旧权重按新 shape 切片拷贝到新网络；整网通道规模联动缩小，参数量与模型文件通常 明显下降，与“只剪少数层”的联动剪枝在机制上不同。

Colab Notebook文件：DSAI_yolov8n_prune_baselinetest.ipynb

实验结果记录：prune_test_record.xlsx

model pt文件list及prune说明：
01_Baseline_Pretrained path:yolov8n.pt 原始官方版本
02_Finetuned_Full path:yolov8n_finetune.pt 微调10轮适配11类任务
03_Finetuned_Unstructured_10 path:yolov8n_finetune_pruned_10.pt 在02基础上非结构化剪枝10%
04_Finetuned_Unstructured_30 path:yolov8n_finetune_pruned_30.pt 在02基础上非结构化剪枝30%
05_Finetuned_Unstructured_50 path:yolov8n_finetune_pruned_50.pt 在02基础上非结构化剪枝50%
06_preset_Structured_10 path:yolov8n_struct_pruned_10.pt 在02基础上指定层联动结构化剪枝10%
07_preset_Structured_30 path:yolov8n_struct_pruned_30.pt 在02基础上指定层联动结构化剪枝30%
08_width_scaling_10_phys path:yolov8n_struct_pruned_10_new.pt 在02基础上宽度缩放结构化剪枝10%
09_width_scaling_10_phys_ft path:yolov8n_struct_pruned_10_finetune.pt 在02基础上宽度缩放结构化剪枝10%+微调10轮
10_width_scaling_30_phys path:yolov8n_struct_pruned_30_new.pt 在02基础上宽度缩放结构化剪枝30%
11_width_scaling_30_phys_ft path:yolov8n_struct_pruned_30_finetune.pt 在02基础上宽度缩放结构化剪枝+微调10轮

使用官方yolov8n及训练数据
- yolov8n Model：https://docs.ultralytics.com/zh/models/yolov8/#performance-metrics
- Training Data：https://universe.roboflow.com/roboflow-gw7yv/self-driving-car/dataset/3
