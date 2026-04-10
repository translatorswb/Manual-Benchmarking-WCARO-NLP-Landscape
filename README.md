# Manual Benchmarking — UNICEF WCARO NLP Landscape

Manual evaluation of ASR and MT models on West and Central African languages as part of [CLEAR Global](https://clearglobal.org)'s NLP landscape mapping consultancy for UNICEF WCARO.

Results feed into the [UNICEF WCA NLP Landscape](https://unicef-ventures.github.io/wca-nlp-landscape/) website ([repo](https://github.com/UNICEF-Ventures/wca-nlp-landscape)).

Work by [Mohamed Aymane Farhi](https://github.com/MedAymenF) ([original repo](https://github.com/MedAymenF/Manual-Benchmarking-WCARO-NLP-Landscape)).

## Purpose

Published benchmarks cover only a fraction of WCA languages. This project fills gaps by running model evaluations where public test data exists but no published scores are available. The focus is on languages identified as priorities by UNICEF country offices: Ewe, Mooré, Dagbani, Nigerian Fulfulde, Central-Eastern Niger Fulfulde, Soninke, and Pulaar.

## Languages & Models & Test sets

### ASR

| Model | Ewe | Mooré | Dagbani | Nigerian Fulfulde | C-E Niger Fulfulde | Soninke |
|-------|-----|-------|---------|-------------------|---------------------|---------|
| [OmnilingualASR 3B](https://huggingface.co/facebook/omniASR-LLM-3B) ([unlimited](https://dl.fbaipublicfiles.com/mms/omniASR-LLM-Unlimited-3B-v2.pt)) | Waxal* | GoAI | CV25, Waxal | OmniCorpus, FLEURS | OmniCorpus* | OmniCorpus |
| [OmnilingualASR 7B](https://huggingface.co/facebook/omniASR-LLM-7B) ([unlimited](https://dl.fbaipublicfiles.com/mms/omniASR-LLM-Unlimited-7B-v2.pt)) | Waxal* | GoAI | CV25, Waxal | OmniCorpus, FLEURS | OmniCorpus* | OmniCorpus |
| [MMS-1B-All](https://huggingface.co/facebook/mms-1b-all) | Waxal | GoAI | — | — | — | — |
| [Simba-S](https://huggingface.co/UBC-NLP/Simba-S) | Waxal | GoAI | — | — | — | — |

### MT

| Model | Language pairs | Test sets |
|-------|---------------|-----------|
| [NLLB 3.3B](https://huggingface.co/facebook/nllb-200-3.3B) | fuv↔en | FLORES+ devtest, BOUQuET |
| [NLLB Distilled 600M](https://huggingface.co/facebook/nllb-200-distilled-600M) | fuv↔en | FLORES+ devtest, BOUQuET |
| [OPUS-MT (bible-big-mul-mul)](https://huggingface.co/Helsinki-NLP/opus-mt-tc-bible-big-mul-mul) | dag↔en, fuc↔en | UDHR, BOUQuET |

## Methodology

- **ASR evaluation** uses NeMo manifests for reproducible scoring. WER and CER are computed using standard text normalization from the OmnilingualASR toolkit.
- **MT evaluation** uses sacreBLEU for BLEU/spBLEU and chrF++ scores, plus TER.
- MT evaluation was run on a NVIDIA GeForce RTX 4090. OmnilingualASR 7B, MMS, and Simba-S evaluation was run on a NVIDIA RTX PRO 5000 Blackwell. OmnilingualASR 3B evaluation was run on a Kaggle T4 GPU.

### Issues encountered

- **OmnilingualASR (<40s limit):** Standard OmnilingualASR models can only transcribe recordings shorter than 40 seconds. The "unlimited" variants remove this limitation but fail on some samples (4 across all datasets). This was resolved by appending 1 second of silence to problematic samples.
- **OmnilingualASR batch processing:** The unlimited variants perform worse with batch inference on T4 hardware, requiring single-sample processing. This makes evaluation very slow — e.g. the full Waxal Ewe ASR test set (n=3782) takes ~10 hours on a T4 with the 3B model. For datasets with enough short samples, recordings longer than 40 seconds were removed to mitigate this (marked with `*` in results).
- **WaxalNLP Ewe data quality:** The Ewe ASR portion had significant quality issues ([reported](https://huggingface.co/datasets/google/WaxalNLP/discussions/17) and fixed on the HF repo). Some samples had empty transcriptions, which were skipped during evaluation ([reported](https://huggingface.co/datasets/google/WaxalNLP/discussions/19)).
- **Infeasible MT evaluations:** The only valid MT data available for Koyraboro Senni (ses) and Gourmanché (gux) was already used in the training of the models supporting these languages. Evaluation was therefore infeasible.
- **Simba-S only:** From UBC-NLP's Simba ASR model family, only Simba-S (based on SeamlessM4T-v2-MT) was evaluated, as it is reported as the top performer among variants.

## Results

Full numeric results are in `ASR_evaluation_results.csv` and `MT_evaluation_results.csv`.

### ASR highlights

| Model | Language | Best WER | Test set |
|-------|----------|----------|----------|
| OmniASR 7B | Dagbani | 27.78 | Common Voice 25 |
| OmniASR 7B | Mooré | 29.51 | GoAI |
| OmniASR 7B | Ewe | 41.40 | Waxal* |
| OmniASR 7B | Nigerian Fulfulde | 48.07 | FLEURS |
| OmniASR 7B | Soninke | 54.63 | OmniCorpus |
| OmniASR 7B | C-E Niger Fulfulde | 75.47 | OmniCorpus* |

The 7B model consistently outperforms the 3B. MMS and Simba-S were evaluated on Ewe and Mooré only — MMS performs comparably to OmniASR 3B, while Simba-S shows WER >100% on both languages.

Results vary dramatically by test set: Dagbani scores 27.78 WER on Common Voice but 65.74 on Waxal with the same model.

`*` = samples longer than 40 seconds removed.

### MT highlights

| Model | Direction | Best chrF++ | Test set |
|-------|-----------|-------------|----------|
| NLLB 3.3B | fuv→en | 36.27 | BOUQuET |
| NLLB 3.3B | en→fuv | 24.51 | BOUQuET |
| NLLB 600M | fuv→en | 32.04 | BOUQuET |
| OPUS-MT | fuc→en | 15.49 | BOUQuET |
| OPUS-MT | dag→en | 12.41 | UDHR |

Strong direction asymmetry: X→en consistently scores higher than en→X. OPUS-MT (trained on Bible data) scores low across the board.

`UDHR`: Universal Declaration of Human Rights

## Repository structure

```
├── African-Languages-ASR-Evaluation.ipynb   # ASR evaluation notebook
├── African_Languages_MT_Evaluation.ipynb    # MT evaluation notebook
├── ASR_evaluation_results.csv               # ASR results (WER, CER)
├── MT_evaluation_results.csv                # MT results (BLEU, spBLEU, chrF++, TER)
└── nemo-manifests/                          # NeMo-format manifests for ASR evaluation
```

## Test sets used

| Dataset | Languages | Task |
|---------|-----------|------|
| [Omnilingual ASR Corpus](https://huggingface.co/datasets/facebook/omnilingual-asr-corpus) | fuv, fuq, snk | ASR |
| [FLEURS](https://huggingface.co/datasets/google/fleurs) | fuv | ASR |
| [Common Voice 25 — Dagbani](https://datacollective.mozillafoundation.org/datasets/cmn2cy2su01iymm07xfr6ul2b) | dag | ASR |
| [WaxalNLP](https://huggingface.co/datasets/google/WaxalNLP) | ewe, dag | ASR |
| [GoAI / Moore Speech Corpora](https://huggingface.co/datasets/goaicorp/GOAI-MooreSpeechCorpora) | mos | ASR |
| [FLORES+](https://huggingface.co/datasets/openlanguagedata/flores_plus) | fuv | MT |
| [BOUQuET](https://huggingface.co/datasets/facebook/bouquet) | fuv, fuc | MT |
| [UDHR](http://efele.net/udhr/translations.html) | dag | MT |

## License

Apache 2.0. See [LICENSE](LICENSE).
