# arabic-atr

Ground truth, diplomatic transcription conventions, and a reproducible fine-tuning workflow for automatic text recognition (ATR) of Arabic manuscripts, built on [Kraken](https://kraken.re) and [eScriptorium](https://escripte.org).

The corpus covers two graphic traditions: eastern *naskh* and Maghribi *mujawhar*.

## Repository layout

```
arabic-atr/
├── cwd/
│   └── gt000/
│       ├── images/          # 23 page images (JPEG)
│       └── xmlpages/        # 23 PAGE XML files (2019-07-15 schema), exported from eScriptorium
├── scripts/
│   └── atr_environment.ipynb   # Colab notebook: install Kraken, fine-tune a base model
├── shared-models/
│   └── model-litho_best.mlmodel   # base recognition model
├── LICENSE                  # MIT
└── README.md
```

Each image in `images/` has a PAGE XML file with the same stem in `xmlpages/`, with line polygons, baselines, and transcriptions. Segmentation and transcription were done and manually corrected in the eScriptorium GUI.

## Ground truth: `cwd/gt000`

| Group | Script | Pages | Transcribed lines |
|---|---|---|---|
| `btv1b60006156_f0008` … `f0017` | *naskh* | 10 | 130 |
| `soura_*` (spreads split into `_left` / `_right`) | Maghribi *mujawhar* | 13 | 420 |

The *naskh* images come from BnF Arabe 5036, ʿAbd al-Raḥmān al-Ṣūfī’s *Kitāb ṣuwar al-kawākib al-thābita* (*Book of Fixed Stars*), fols. 1v–6r, on Gallica: <https://gallica.bnf.fr/ark:/12148/btv1b60006156>. The provenance of the *soura_\** images is not specified.

File `btv1b60006156_fNNNN.jpg` corresponds to Gallica view `fN`:

| Gallica view | Folio |
|---|---|
| [f8](https://gallica.bnf.fr/ark:/12148/btv1b60006156/f8.item) | 1v |
| [f9](https://gallica.bnf.fr/ark:/12148/btv1b60006156/f9.item) | 2r |
| [f10](https://gallica.bnf.fr/ark:/12148/btv1b60006156/f10.item) | 2v |
| [f11](https://gallica.bnf.fr/ark:/12148/btv1b60006156/f11.item) | 3r |
| [f12](https://gallica.bnf.fr/ark:/12148/btv1b60006156/f12.item) | 3v |
| [f13](https://gallica.bnf.fr/ark:/12148/btv1b60006156/f13.item) | 4r |
| [f14](https://gallica.bnf.fr/ark:/12148/btv1b60006156/f14.item) | 4v |
| [f15](https://gallica.bnf.fr/ark:/12148/btv1b60006156/f15.item) | 5r |
| [f16](https://gallica.bnf.fr/ark:/12148/btv1b60006156/f16.item) | 5v |
| [f17](https://gallica.bnf.fr/ark:/12148/btv1b60006156/f17.item) | 6r |

## Transcription conventions

The transcription is diplomatic: it reproduces the graphic and linguistic features of the witness and does not normalize. Main choices:

| Phenomenon | Encoding |
|---|---|
| Vowels | Only vowels marked in the manuscript are transcribed; none is added. |
| Subscript *shadda* / *hamza* (*mujawhar*: encodes /i/) | *shadda* / *hamza* above the letter + *kasra* below. |
| *Alif al-madd* with *madd* sign | U+0627 + U+0653 |
| *Alif al-madd* with vertical stroke after a consonant | consonant + U+0670 + U+0627 |
| Unpointed consonant (identification depends on context) | U+066E ARABIC LETTER DOTLESS BEH |
| Unpointed *fāʾ* / *qāf* skeleton | U+066F ARABIC LETTER DOTLESS QAF |
| *Fāʾ* with dot below | U+06A2 |
| *Qāf* with one dot above | U+06A7 |
| Triple-dot mark (end of a *ḥadīth*) | U+061E ARABIC TRIPLE DOT PUNCTUATION MARK |
| Rosette (internal segmentation) | U+2736 SIX POINTED BLACK STAR |
| Isolated initial *hāʾ* (abbreviation of *intihāʾ*, “end”) | U+FEEB ARABIC LETTER HEH INITIAL FORM |
| Closing sign not yet identified in Unicode | provisionally `[% SCM]` (section-closing mark) |

## Base model

`shared-models/model-litho_best.mlmodel` (Kraken format) is one of the models of the TariMa Models release on Zenodo: <https://zenodo.org/records/7810571>. See the record for its license and citation. It was trained on lithographs, which retain much of the ligatures and ductus of handwriting. Using it as the base for manuscripts is an empirical transfer hypothesis, tested here through fine-tuning and validation.

## Fine-tuning

`scripts/atr_environment.ipynb` runs on Google Colab. It expects this layout on a Google shared drive:

```
/content/drive/Shareddrives/kraken/workdir/
├── shared-models/*.mlmodel
└── cwd/gtNNN/{images,xmlpages,models}
```

Persistent data (ground truth, images, base models, final model) stays on the shared drive. Preparation and training run in a volatile local directory (`/content/atr_environment_work/`), rebuilt at every run.

Run the cells from top to bottom:

1. Select the ground truth (`gtNNN`), the base model, and the script optimization (Latin or Arabic).
2. Install and check Kraken and Ketos (Python 3.10–3.13).
3. Set training parameters.
4. Validate the PAGE XML: pair each file with its image through `imageFilename`, check that it contains transcribed lines, and write `train.lst` and `validation.lst`.
5. Build and run the `ketos train` command. The notebook reads `ketos --help` to adapt option names across versions.
6. Copy the best model to `cwd/gtNNN/models/`, with an optional output prefix. From there it can be imported into eScriptorium and applied to new pages, whose corrected transcriptions can feed new ground truth.

Main parameters: up to 30 epochs (minimum 5), early-stopping lag of 8 epochs, batch size 8, NFD normalization with right-to-left base direction, `union` codec resizing (keeps the base codec and adds new characters), Ketos standard augmentation, GPU when available, weights saved in CoreML format.

## Results so far

| Run | Training pages | Validation pages | Best validation character accuracy | Equivalent CER |
|---|---|---|---|---|
| Main | 21 (10 *naskh*, 11 *mujawhar*) | 2 (both *mujawhar*) | 75.74% | ≈ 24.26% |
| Exploratory, offline augmentation | same, with ad hoc offline variants | same, unaugmented | 77.55% | ≈ 22.45% |

Read these figures with care:

- CER is derived as 1 − Ketos accuracy, not measured independently.
- Validation covers only two *mujawhar* pages, and no independent test set was held out. The figures say nothing about the *naskh* pages or about Arabic manuscripts in general.
- The augmented run (best checkpoint 26) improves by 1.81 percentage points on this split only. It is an exploratory hint, not evidence of a general gain. Its data and scripts (`gt003_aggressive.tar.bz2`) are not part of this repository.

Planned next steps: enlarge the *mujawhar* validation data, add an independent test set, compare alternative base models, and run systematic augmentation experiments.

## Rights

- Code and data in this repository: MIT License.
- BnF images: check the reuse conditions stated on Gallica.
