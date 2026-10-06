# RAGNet-3DOI-audit

对 [RAGNet](https://arxiv.org/abs/2507.23734) 中 3DOI 子集的逐条人工审阅与修订。

原始的 1,013 条图像—指令对已全部逐条判读，判定为保留、改写或剔除三种结论之一；据此发布 997 条的修订版，并公开全部审阅记录。

## 为什么做这次审阅

RAGNet-3DOI 的指令由 GPT-4 生成，发布时未经人工校验。我们在评测中发现，其中大量指令与对应的真值掩码并不匹配：模型被要求完成的任务与它实际被评测的目标不一致。在这类样本上，得分差异反映的是评测数据自身的缺陷，无法归因于方法能力。

## 审阅发现的问题

三类问题最为集中。

**粒度错配**：指令询问的是整个物体，而真值掩码标注的只是其上的一个部位。例如"Find a knife, please."，其掩码只覆盖刀柄。

**对象错配**：指令描述的对象与掩码实际对应的对象不同。例如一条指令描述的是花瓶，掩码对应的却是花盆。

**掩码缺陷**：掩码本身无法正确表达部件的结构，例如剪刀把手中的孔洞过小，被掩码一并填充，两个把手连成实心区域。这类样本在处理时一律剔除。

同类现象也见于 RAGNet-HANDAL 与 ReasonAff，我们在两个集合上做了抽样检查。本仓库只发布 RAGNet-3DOI 的修订，因为其规模使逐条人工审阅在人力上可行。

## 本仓库的内容

```
RAGNet-3DOI-audit/
├── tasks.jsonl        修订版数据集，997 条
├── review_log.jsonl   完整审阅记录，1,013 条（含被剔除的 16 条）
├── images/            1,013 个图像文件
├── masks/             1,013 个真值掩码
├── LICENSE
└── NOTICE
```

`tasks.jsonl` 每行一个 JSON 对象，含四个字段：`id` 为样本编号，`image` 与 `mask` 为该样本的图像与掩码的相对路径，`question` 为该样本的指令。

```json
{"id": 0, "image": "images/0000.jpg", "mask": "masks/0000.jpg", "question": "Please bring a bowl to the dining table."}
```

`review_log.jsonl` 覆盖全部 1,013 条，因此比 `tasks.jsonl` 多出被剔除的 16 条。每行的字段为：`id`、`image`、`mask`、`question`（该样本的指令）、`original_question`（修订前的原指令）、`review_status`（`keep`、`edit` 或 `reject`）、`source_image` 与 `source_mask`。

修订前后的指令可以逐条对比：

```json
{"id": 2, "image": "images/0002.jpg", "mask": "masks/0002.jpg", "question": "Could you please hand me a bowl?", "original_question": "Could you please hand me a bowl for the salad?", "review_status": "edit", "source_image": "data/3doi/images/AR_j1yJlIVn3vY_2_1440_65.jpg", "source_mask": "data/3doi/masks/bowl/AR_j1yJlIVn3vY_2_1440_65.jpg"}
```

其中 `source_image` 与 `source_mask` 给出该样本在 3DOI 数据集中的原始路径，用于溯源；它们是 3DOI 内部的相对路径，与任何本机目录无关。

## 判定的依据

一条指令与其真值掩码匹配，指按该指令的字面要求去定位，得到的目标恰好是掩码所标注的区域。1,013 条全部经过两轮复核，`review_status` 为最终结论。

## 如何使用

```python
import json

tasks = {}
with open("tasks.jsonl", encoding="utf-8") as f:
    for line in f:
        r = json.loads(line)
        tasks[r["id"]] = r

with open("review_log.jsonl", encoding="utf-8") as f:
    log = [json.loads(line) for line in f]

rewritten = [r for r in log if r["review_status"] == "edit"]
print(len(tasks), "published;", len(rewritten), "instructions rewritten")
```

在修订版上评测时，直接使用 `tasks.jsonl` 中的 `question` 与对应的 `mask` 即可。若要比较大模型在修订前后的表现，用 `review_log.jsonl` 把同一条样本的两个版本对齐。

## 与原始集合的关系

原始集合有 1,013 条图像—指令对，其中 616 条保留、381 条改写、16 条剔除，修订版共 997 条。

两个版本共享同一批图像与掩码，差别只在指令文本。其中 `id` 为 509 的样本图像文件已损坏（0 字节），不可读，在评测中跳过。

真值掩码为灰度图，像素值集中在 0 与 255 两端。在 1 到 224 之间的任意阈值下二值化，前景面积相同。

## 引用

使用本修订版时，请引用 RAGNet 与 3DOI，并说明使用了本审阅的修订版：

```bibtex
@inproceedings{wu2025ragnet,
  title     = {RAGNet: Large-scale Reasoning-based Affordance Segmentation Benchmark towards General Grasping},
  author    = {Wu, Dongming and Fu, Yanping and Huang, Saike and Liu, Yingfei and Jia, Fan and Liu, Nian and Dai, Feng and Wang, Tiancai and Anwer, Rao Muhammad and Khan, Fahad Shahbaz and Shen, Jianbing},
  booktitle = {Proceedings of the IEEE/CVF International Conference on Computer Vision (ICCV)},
  year      = {2025}
}

@inproceedings{qian2023understanding,
  title     = {Understanding 3D Object Interaction from a Single Image},
  author    = {Qian, Shengyi and Fouhey, David F.},
  booktitle = {Proceedings of the IEEE/CVF International Conference on Computer Vision (ICCV)},
  year      = {2023}
}
```

## 许可

图像与掩码不是本仓库作者的原创内容。它们出自 3DOI，在 3DOI 中采集自 EpicKitchens 与 Omnidata，各自受其原始许可约束；EpicKitchens 的数据以 CC BY-NC 4.0 发布，因此本仓库为非商业研究用途。详见 `NOTICE` 与 `LICENSE`。
