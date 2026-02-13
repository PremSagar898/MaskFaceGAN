The goal: give an AI coding agent immediate, practical context for making safe, correct changes in this repo.

Quick facts
- This project implements MaskFaceGAN: latent-code optimization on StyleGAN2 for high-res face editing.
- Key entry points: `main.py` (CLI), `trainer.py` (optimization loop), `model_module.py` (model assembly).
- External artifacts: pretrained StyleGAN2 weights, attribute classifier, face parser and e4e encoder (downloaded by `download.sh`).
- Dependencies pinned in `requirements.txt` (e.g. `torch==1.13.1`, `torchvision==0.14.1`).

What to read first (minimal set)
- `main.py` — shows CLI flags used by humans: `--attribute`, `--image`, `--e4e_init`, `--blend_option` and how `Config` is loaded and used.
- `trainer.py` — core training/optimization steps: `train_latent()`, `train_noise()`, and `generate_result()` are the high-level operations to hook into or modify.
- `model_module.py` — builds the model graph (bridges project code with external StyleGAN2/e4e/attribute classifier code).
- `config.yml` — authoritative list of attributes, device selection and model options; always consult it before changing defaults.

Repo structure notes (how code is organized)
- models/ contains submodules: `stylegan2/` (StyleGAN2 code + low-level ops), `e4e/` (encoders), and `attribute_classifier/`.
- Custom ops/compiler artifacts live under `models/stylegan2/op/` — avoid changing these unless you understand native/C++/CUDA builds.
- `lpips/` contains the perceptual loss wrapper used by training code.

Coding and change conventions specific to this repo
- Device handling: many modules expose `.to(cfg.device)` and expect `cfg.device` (from `Config('config.yml')`). Use that pattern when adding tensors or modules.
- Image I/O: project uses `utils.read_img` and `utils.save_image` plus `torchvision.utils.save_image`. Use the existing helpers to preserve expected tensor shapes and normalization.
- Attribute edits: `main.py` converts CLI `--target` into a 1-element tensor and applies smoothing (`load_data`). Mirror that flow when adding tests or new CLI args.
- Model checkpoints: code expects stylegan/e4e checkpoints under `models/` directories (see README). When writing code that loads checkpoints, prefer relative paths under `models/` and raise a clear error if missing.

Build / run / developer workflows (discoverable)
- Install deps: `pip install -r requirements.txt` (README). `requirements.txt` pins PyTorch/torchvision.
- Get pretrained weights: run `./download.sh` (README) and follow the StyleGAN2 conversion step described there.
- Run example: `python main.py --attribute mouth_slightly_open --outdir output --image input/1815.jpg` (README / `main.py`).

Integration pitfalls and gotchas for agents
- Native ops: `models/stylegan2/op/*` may require building or using precompiled binaries. On Windows, editing these files can introduce platform-specific build tasks; prefer CPU-only changes unless you add build/test steps.
- Pretrained models are required at runtime. Any code change that changes checkpoint names or paths must be mirrored in `download.sh` and README.
- Version sensitivity: `requirements.txt` pins `torch==1.13.1`. When changing torch-related code, keep compatibility with that version unless you update `requirements.txt` and test end-to-end.

Examples to copy when making edits
- Add a new CLI flag: follow `parse_args()` in `main.py`; use `Config(...)` then pass values into `ModelsModule` or `Trainer`.
- Add an attribute-aware branch: check `config.yml` for `local_attributes` vs global behavior; `main.py` and `model_module.py` switch behavior based on these values.

When to run tests or manual checks
- After model/api changes, run the minimal example from README with a small input image and ensure `python main.py ...` completes to `save_image` (fast smoke test).
- If native ops or model serialization changes are made, verify that `trainer.generate_result()` still returns a CPU/GPU tensor that `utils.save_image` can write.

If you need more context
- Start by reading `README.md` and `config.yml` (attribute lists and defaults). Then inspect `trainer.py` and `model_module.py` to follow dataflow from image -> latent -> result.

If anything above is unclear or you'd like the agent to include more examples (e.g. a checklist for adding new attributes or a minimal unit test scaffold), say which area to expand.
