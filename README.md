# RoboSynChallenge ACT submission

One evaluation request covering all 10 tasks. Policy: LeRobot ACT, one checkpoint per task.

- Weights: [TyangJN/robosyn-act-10tasks](https://huggingface.co/TyangJN/robosyn-act-10tasks).
- Fixed Hugging Face revision: `5e874cb84a27bdf02c9dfc89781a73a48584608f`.
- Official evaluator revision: [`8818c82df5fa2522809b8105e7018950dcc9a9a8`](https://github.com/EDEM-AI/RoboSynChallenge/tree/8818c82df5fa2522809b8105e7018950dcc9a9a8) (`8818c82`).

Each task subfolder contains `model.safetensors` and `config.json`. `train_config.json` is not included. The official ACT inference loader uses the model weights and policy configuration. All model files stay on Hugging Face; this GitHub repository contains no weights or training records.

## Run

Use a GPU machine with the official RoboSynChallenge simulator and the ACT environment installed according to the [official installation documentation](https://edem-ai.github.io/RoboSynChallenge/html/getting_started/installation.html) and [`policy/act/pyproject.toml` at the pinned revision](https://github.com/EDEM-AI/RoboSynChallenge/blob/8818c82df5fa2522809b8105e7018950dcc9a9a8/policy/act/pyproject.toml). That environment specifies Python 3.10 or 3.11 and its pinned LeRobot dependency. The commands below also require the Hugging Face `hf` CLI.

Starting from the root of this submission repository:

```bash
SUBMISSION_DIR="$(pwd)"
git clone https://github.com/EDEM-AI/RoboSynChallenge.git ../RoboSynChallenge
git -C ../RoboSynChallenge checkout 8818c82df5fa2522809b8105e7018950dcc9a9a8
cd ../RoboSynChallenge

cp "$SUBMISSION_DIR"/policy/act/{eval.sh,deploy_policy.yml,deploy_policy.py} policy/act/
hf download TyangJN/robosyn-act-10tasks \
  --revision 5e874cb84a27bdf02c9dfc89781a73a48584608f \
  --local-dir checkpoints
(cd checkpoints && sha256sum -c SHA256SUMS.txt)
```

Run the commands from the official RoboSynChallenge repository root. `env -u PYTHON_BIN` lets the official launcher choose `policy/act/.venv/bin/python` when it exists; otherwise activate the ACT environment first. The positional `0` selects GPU 0.

| Task | Checkpoint subfolder in the pinned HF repository |
|---|---|
| table_rearrangement | `table_rearrangement/` |
| click_bell | `click_bell/` |
| water_pouring | `water_pouring/` |
| handle_basket | `handle_basket/` |
| items_handover | `items_handover/` |
| drawer_open_place | `drawer_open_place/` |
| mixer_operating | `mixer_operating/` |
| item_assembly | `item_assembly/` |
| manipulate_pipette | `manipulate_pipette/` |
| sample_loading | `sample_loading/` |

Per-task evaluation commands:

```bash
env -u PYTHON_BIN bash policy/act/eval.sh table_rearrangement random checkpoints/table_rearrangement 0 --pytorch_device cuda --headless True
env -u PYTHON_BIN bash policy/act/eval.sh click_bell random checkpoints/click_bell 0 --pytorch_device cuda --headless True
env -u PYTHON_BIN bash policy/act/eval.sh water_pouring random checkpoints/water_pouring 0 --pytorch_device cuda --headless True
env -u PYTHON_BIN bash policy/act/eval.sh handle_basket random checkpoints/handle_basket 0 --pytorch_device cuda --headless True
env -u PYTHON_BIN bash policy/act/eval.sh items_handover random checkpoints/items_handover 0 --pytorch_device cuda --headless True
env -u PYTHON_BIN bash policy/act/eval.sh drawer_open_place random checkpoints/drawer_open_place 0 --pytorch_device cuda --headless True
env -u PYTHON_BIN bash policy/act/eval.sh mixer_operating random checkpoints/mixer_operating 0 --pytorch_device cuda --headless True
env -u PYTHON_BIN bash policy/act/eval.sh item_assembly random checkpoints/item_assembly 0 --pytorch_device cuda --headless True
env -u PYTHON_BIN bash policy/act/eval.sh manipulate_pipette random checkpoints/manipulate_pipette 0 --pytorch_device cuda --headless True
env -u PYTHON_BIN bash policy/act/eval.sh sample_loading random checkpoints/sample_loading 0 --pytorch_device cuda --headless True
```

The same Hugging Face revision applies to all 10 tasks. Exact task-to-repository mappings, subfolders, revisions and weight SHA256 values are recorded in [model_manifest.json](model_manifest.json). The HF root also contains `SHA256SUMS.txt`.

## Adapter and interface

`policy/act/eval.sh`, `policy/act/deploy_policy.yml` and `policy/act/deploy_policy.py` are byte-for-byte copies of official commit `8818c82`. There are no adapter modifications.

- State: 14-dimensional `robot/qpos` (dual-arm joints and grippers).
- Images: RGB `cam_high`, `cam_left_wrist` and `cam_right_wrist`.
- Output: 14-dimensional absolute joint and gripper actions.
- Official deployment configuration: `act_step: 10`, `state_obs_path: robot/qpos`, `strict_action_dim: true`.
- The model configurations retain `chunk_size: 50` and `n_action_steps: 50`. These are the ACT prediction/action-queue settings; `act_step` controls how many action steps the deployment adapter attempts in one evaluator call.

## Checkpoint attribution

This release includes selected existing checkpoints and continued-training variants. Detailed training records, logs and training configurations are not included in this repository.

Three continued-training checkpoints start from models released by other authors:

| Task | Public base checkpoint | Base revision | Base subfolder |
|---|---|---|---|
| click_bell | [Beeny0814/robosyn-act-v1](https://huggingface.co/Beeny0814/robosyn-act-v1/tree/ba37fc3d73f912b1c105ebf62847b287ded27a9d) | `ba37fc3d73f912b1c105ebf62847b287ded27a9d` | `click_bell` |
| items_handover | [oyzq941353193/robosyn-submission-v2](https://huggingface.co/oyzq941353193/robosyn-submission-v2/tree/2b512fed9d1d5d98bdb8c7b58527905f4fa0216f) | `2b512fed9d1d5d98bdb8c7b58527905f4fa0216f` | `act_items_speed3_mix_060000` |
| mixer_operating | [mair-robosyn/ACT_sim_mixer_operating_stage1](https://huggingface.co/mair-robosyn/ACT_sim_mixer_operating_stage1/tree/70e71d2cce638a108dc773b2a1ada48b58767998) | `70e71d2cce638a108dc773b2a1ada48b58767998` | repository root |

The base models are credited to their original authors.

## Verification status

Local weight hashes match the published Hugging Face files at the pinned revision. The three adapter files have been checked against official Git blobs at `8818c82`. End-to-end GPU evaluation of this submission package has not been performed.

## Upstream license

The copied official adapter files are from [EDEM-AI/RoboSynChallenge](https://github.com/EDEM-AI/RoboSynChallenge). Its Apache 2.0 license is retained in [LICENSE](LICENSE). This does not assign a new license to the separately hosted model weights.
