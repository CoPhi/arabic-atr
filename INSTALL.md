# Installation

`scripts/atr_environment.ipynb` fine-tunes a Kraken recognition model on PAGE XML ground truth. It runs in two modes, detected automatically:

| Mode | Where the data lives | Best for |
|---|---|---|
| [A. Google Colab + Google Drive](#a-google-colab--google-drive) | A Google shared drive | Teams sharing ground truth and models, no local GPU |
| [B. Stand-alone](#b-stand-alone-local-jupyter) | A local clone of this repository | Offline work, your own hardware |

In both modes the notebook expects the same layout (called `workdir/` below):

```
workdir/
├── shared-models/
│   └── *.mlmodel            # base models
└── cwd/
    └── gtNNN/               # one folder per ground-truth set
        ├── images/
        ├── xmlpages/        # PAGE XML, one file per image
        └── models/          # fine-tuned models are written here
```

This repository already follows this layout: its root is a valid `workdir/`.

## Requirements

- Python 3.10–3.13 (the notebook checks this).
- [Kraken](https://kraken.re) with PDF support (`kraken[pdf]`), installed by the notebook.
- System libraries: pango (`libpangocairo`), libxml2, libxslt.
- A GPU is optional but strongly recommended: the notebook uses CUDA when available and falls back to the CPU otherwise, where training is much slower.

## A. Google Colab + Google Drive

1. **Create the folder structure** on a Google shared drive named `kraken`, with a `workdir/` folder inside it, following the layout above.
2. **Upload the data** from this repository:
   - `shared-models/model-litho_best.mlmodel` into `workdir/shared-models/`;
   - `cwd/gt000/` (with `images/` and `xmlpages/`) into `workdir/cwd/`.
3. **Open the notebook in Colab**: upload `scripts/atr_environment.ipynb` at <https://colab.research.google.com>, or open it from Drive.
4. **Select a GPU runtime**: Runtime → Change runtime type → choose a GPU.
5. **Run the cells from top to bottom.** The first code cell mounts Drive (authorize access when prompted) and prints `Mode: Google Colab`. Then pick the ground truth, base model, and script optimization (choose *arabic* for Arabic scripts).
6. **Collect the result** in `workdir/cwd/gtNNN/models/`. Training itself runs in `/content/atr_environment_work/`, which Colab deletes when the runtime ends.

Notes:

- The default path is `/content/drive/Shareddrives/kraken/workdir`. To use a different location (for example a folder in *My Drive*), set `WORKDIR_OVERRIDE` at the top of the first code cell, for example `WORKDIR_OVERRIDE = "/content/drive/MyDrive/kraken/workdir"`.
- The optional section 7 (augmented data) is Colab-only and needs an archive, `gt003_aggressive.tar.bz2`, that is not part of this repository.

## B. Stand-alone (local Jupyter)

1. **Clone the repository** (the model file is about 16 MB, the images about 61 MB):

   ```bash
   git clone https://github.com/CoPhi/arabic-atr.git
   cd arabic-atr
   ```

2. **Install the system libraries.** On Debian or Ubuntu:

   ```bash
   sudo apt-get install -y libpangocairo-1.0-0 libxml2 libxslt1.1
   ```

   On other systems, install pango, libxml2, and libxslt with your package manager.

3. **Create a virtual environment** with Python 3.10–3.13, and install Jupyter:

   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   pip install -U pip jupyterlab ipywidgets
   ```

4. **(Optional) GPU.** To train on an NVIDIA GPU, install the PyTorch build matching your CUDA version from <https://pytorch.org> *before* the next step. Otherwise training runs on the CPU.

5. **Start Jupyter from the repository root** and open the notebook:

   ```bash
   jupyter lab scripts/atr_environment.ipynb
   ```

6. **Run the cells from top to bottom.** The first code cell looks upward from the current directory for a folder containing both `cwd/` and `shared-models/`, and prints `Mode: stand-alone`. The installation cell then skips `apt-get` (unless you run as root on a Debian-based system) and installs Kraken with `pip` into the active environment.
7. **Collect the result** in `cwd/gtNNN/models/`. Training runs in a temporary directory (the system temp folder, `ATR_TMPDIR` if set) that is rebuilt at every run.

To use data stored elsewhere, set the `ATR_WORKDIR` environment variable before starting Jupyter, or `WORKDIR_OVERRIDE` in the first code cell:

```bash
ATR_WORKDIR=/path/to/workdir jupyter lab scripts/atr_environment.ipynb
```

## Checking the installation

After the installation cell, the notebook prints the Python version, the paths of `kraken` and `ketos`, their versions, and whether CUDA is available. The same checks work from a terminal in the virtual environment:

```bash
kraken --version
ketos --version
python -c "import torch; print(torch.cuda.is_available())"
```

## Troubleshooting

| Problem | Likely cause and fix |
|---|---|
| `Kraken requires Python 3.10–3.13.` | Create the virtual environment with a supported Python version. |
| `Could not find a folder containing both 'cwd/' and 'shared-models/'` | Start Jupyter inside the repository, or set `ATR_WORKDIR` or `WORKDIR_OVERRIDE`. |
| `No gtNNN directory found` / `No .mlmodel file found` | Check the layout above: ground-truth folders must be named `gt` followed by digits, and base models must end in `.mlmodel`. |
| Errors while importing or running Kraken about pango or XML libraries | Install the system libraries (step 2 of the stand-alone mode). |
| Training is very slow | No GPU is available. Check `torch.cuda.is_available()`, or use Colab with a GPU runtime. |
| `ketos` rejects an option | Option names differ across Kraken versions. The notebook reads `ketos --help` and adapts; the final command is printed before training, so you can compare it with your version. |
