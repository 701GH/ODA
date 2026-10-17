# RAGNet-3DOI Audit

Item-by-item manual audit and revision of the 3DOI subset of [RAGNet](https://arxiv.org/abs/2507.23734).

The release contains **997** revised items out of the original 1,013 — **616** kept unchanged, **381** with a rewritten instruction, **16** rejected — plus the record of all 1,013 decisions.

## Contents

| Path | Description |
| --- | --- |
| `tasks.jsonl` | revised benchmark, 997 items |
| `review_log.jsonl` | full audit record, 1,013 items (including the 16 rejected) |
| `images/` | 1,013 images |
| `masks/` | 1,013 ground-truth masks |
| `LICENSE` | license terms |
| `NOTICE` | data provenance |

### `tasks.jsonl`

One JSON object per line.

| Field | Meaning |
| --- | --- |
| `id` | sample id |
| `image` | relative path to the image |
| `mask` | relative path to the ground-truth mask |
| `question` | the instruction |

```json
{"id": 0, "image": "images/0000.jpg", "mask": "masks/0000.jpg", "question": "Please bring a bowl to the dining table."}
```

### `review_log.jsonl`

Covers all 1,013 items, so it has 16 more entries than `tasks.jsonl`, and holds both versions of every instruction side by side.

| Field | Meaning |
| --- | --- |
| `id`, `image`, `mask` | as above |
| `question` | the instruction in the revised version |
| `original_question` | the instruction before revision |
| `review_status` | `keep` (instruction matches its mask), `edit` (instruction rewritten to match the mask) or `reject` (the mask itself is defective, sample excluded from `tasks.jsonl`) |
| `source_image`, `source_mask` | the sample's path in 3DOI, kept for traceability; 3DOI-internal relative paths, not paths in this repository |

```json
{"id": 2, "image": "images/0002.jpg", "mask": "masks/0002.jpg", "question": "Could you please hand me a bowl?", "original_question": "Could you please hand me a bowl for the salad?", "review_status": "edit", "source_image": "data/3doi/images/AR_j1yJlIVn3vY_2_1440_65.jpg", "source_mask": "data/3doi/masks/bowl/AR_j1yJlIVn3vY_2_1440_65.jpg"}
```

## Usage

```python
import json

with open("tasks.jsonl", encoding="utf-8") as f:
    tasks = {r["id"]: r for r in map(json.loads, f)}

with open("review_log.jsonl", encoding="utf-8") as f:
    log = [json.loads(line) for line in f]

rewritten = [r for r in log if r["review_status"] == "edit"]
print(len(tasks), "published;", len(rewritten), "instructions rewritten")
```

Evaluate on the revised version with `question` and `mask` from `tasks.jsonl`. To compare a model before and after revision, align the two versions of a sample with `review_log.jsonl`.

## Notes

- The image of **id 509** is 0 bytes and unreadable in the original release. It is one of the 16 rejects, which is why the original version evaluates 1,012 rather than 1,013 samples.
- Masks are single-channel grayscale. Any binarization threshold from 10 to 245 gives the same foreground; we use 128.

## Citation

If you use this revision, please cite RAGNet and 3DOI, and state that you used the revised version:

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

## License

The images and masks come from 3DOI and are distributed under CC BY-NC 4.0; the audit and revision work is released under the same terms. See `LICENSE` for the full terms and `NOTICE` for provenance.
