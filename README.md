# ieeg-epilepsy-eeg-biomarkers-neuroscience-reproducibility-openneuro-mne-python

ieeg-composition

Does a reported iEEG biomarker score measure the biomarker, or the cohort?

Analysis code for Cohort composition confounds the evaluation of interictal iEEG biomarkers for epilepsy surgery (Tiwari, in submission).

The pipeline computes 28 interictal biomarkers across three public OpenNeuro datasets and asks how much of their reported localization performance is attributable to the composition of the cohort rather than to the recordings. It provides two nulls --- a participant-specific empirical null and a spatially constrained null --- and a composition-only model that predicts reported performance without seeing any EEG.

Everything here runs on public data. Nothing in this repository contains patient-identifiable information.

bash
export IEEG_BASE=/path/to/ieeg_project     # Windows: set IEEG_BASE=...
python run.py --list                       # stages and their state
python run.py --stage all                  # reproduce everything
python run.py --manifest                   # where each reported number comes from
Layout
ieeg_project/              IEEG_BASE — data and outputs
├── raw/
│   ├── ds003876/          multicentre interictal          v1.0.2
│   ├── ds004100/          HUP, interictal recordings only v1.1.3
│   └── ds003844/          RESPect intraoperative          v1.0.1
├── features/              per-participant .parquet checkpoints
├── results/               every table and figure the paper cites
└── supplementary/         Supplementary Tables S1–S11

this repository
├── run.py                 single entry point
├── run_pipeline.py        cohort, features and per-participant metrics
├── ieeg/                  config, data, features, evaluate, _namefix
└── analyses/              one script per analysis

Data are not included. Fetch them with analyses/download_datasets.py, or:

python
import openneuro
openneuro.download("ds003876", target_dir="ieeg_project/raw/ds003876")
openneuro.download("ds004100", target_dir="ieeg_project/raw/ds004100",
                   include=["*task-interictal*"])
openneuro.download("ds003844", target_dir="ieeg_project/raw/ds003844")

Only the interictal runs of ds004100 are used, so the include filter saves a substantial download. ds003876 must be fetched in full because the derivatives/ folder carries the released source–sink outputs re-analysed in Section 3.8.

Order

Stages depend on one another; run.py --stage all handles the ordering. Run by hand only if you know why.

#	Script	Produces	Time
1	run_pipeline.py --stage cohort	cohort table	seconds
2	run_pipeline.py --stage features	28 features per participant	~2 h
3	run_pipeline.py --stage evaluate	per-participant metrics, permutation control	~15 min
4	analyses/orientation.py	leave-one-participant-out orientation (the primary protocol)	~15 min
5	analyses/composition_and_variance.py	correlation robustness, variance decomposition	~5 min
6	analyses/composition_effective_n.py	primary and secondary composition models	~5 min
7	analyses/localization_evidence.py	which features exceed the empirical null	~3 min
8	analyses/spatial_null.py	the same under the spatially constrained null	~30 min
9	analyses/soz_label.py	all three analyses with the onset zone as label	~40 min
10	analyses/robustness.py	orientation, learners, bootstrap, leave-one-dataset-out	~15 min
11	analyses/ssi_regularisation.py	source–sink at the published regularisation	~40 min
12	analyses/simulation.py	fixed-ranking-quality simulation, and Fig 5	~5 min
13	analyses/below_null_check.py	below-null fraction against prevalence, Fig S1	~5 min
14	analyses/logistic_outcome.py	logistic sensitivity and power	~2 min
15	analyses/per_centre_stats.py	per-centre confound	~2 min
16	analyses/cohort_overlap.py	overlap with the released derivative files	~2 min

Steps 2, 8, 9 and 11 dominate the runtime. All checkpoint per participant, so an interrupted run resumes.

Figures

Figure scripts overlap and must run in this order, because later ones overwrite earlier output with corrected versions:

bash
python analyses/figures_1_3_4.py       # base versions of Figs 1, 3, 4
python analyses/figure3.py             # Fig 3 with annotated-zone axis labels
python analyses/table3_and_figure2.py  # Fig 2 from the corrected composition model
python analyses/figure1_cohort.py      # Fig 1 from the final 55-participant cohort
python analyses/simulation.py          # Fig 5
Checks
bash
python analyses/consistency_check.py   # numbers in the manuscript against the outputs
python analyses/build_supplementary.py # assemble S1–S11
Four things that were necessary and are not obvious

Channel names do not match between channels.tsv and the signal headers. EDF+ wraps and truncates them (EEG POL RATO1-Re against RATO1), sometimes with a doubled prefix. ieeg/_namefix.py reconciles them. A uniqueness guard is also required, or two channels.tsv entries can map to one header channel and MNE raises sel is not unique.

ds004100 ships ictal and interictal runs together. Without an explicit preference for task-interictal, the ictal runs are selected for most participants and the study answers a different question.

Scalp EEG and EKG channels lead each ds004100 recording. Retaining them inflates the bad-channel fraction and distorts prevalence, which is the variable this paper is about.

The published source–sink regularisation does not work on this data. At lambda = 1e-4 the fitted system is unstable in a median 93.7% of windows. ieeg/features.py selects it adaptively; analyses/ssi_regularisation.py quantifies what that changes.

Requirements
python >= 3.10
mne >= 1.6      numpy       pandas      scipy
scikit-learn    statsmodels matplotlib
pyarrow         openpyxl    openneuro-py

pip install -r requirements.txt

Seeds are fixed in ieeg/config.py and PYTHONHASHSEED is pinned by the runner, so two runs on the same data agree exactly.

Data

Public, de-identified, CC0, via OpenNeuro:

ds003876 v1.0.2 — doi:10.18112/openneuro.ds003876.v1.0.2 (Gunnarsdottir et al., Epilepsy-iEEG-Interictal-Multicenter-Dataset)
ds004100 v1.1.3 — doi:10.18112/openneuro.ds004100.v1.1.3 (Bernabei et al., HUP iEEG Epilepsy Dataset)
ds003844 v1.0.1 — doi:10.18112/openneuro.ds003844.v1.0.1 (Zweiphenning et al., RESPect intraoperative iEEG)
Citation

If you use this code, please cite the paper:

Tiwari A. Cohort composition confounds the evaluation of interictal iEEG
biomarkers for epilepsy surgery. (in submission)

and the three datasets under their own DOIs above.

License

MIT. The datasets are separately licensed CC0 by OpenNeuro.

Contact

Anurag Tiwari, Department of Computer Science and Engineering, South Asian University, New Delhi. anuragtiwari@sau.int
