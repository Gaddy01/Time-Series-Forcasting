# Mobile Internet Traffic Forecasting with RNN, GRU, and LSTM

This repository contains the source code, trained models, preprocessing objects, and experiment results for the **ML Techniques I Formative I** project.

The project studies one-step-ahead mobile Internet traffic forecasting across geographical areas of Milan. It compares three recurrent neural network architectures:

- Simple Recurrent Neural Network (RNN)
- Gated Recurrent Unit (GRU)
- Long Short-Term Memory (LSTM)

The main research question is:

> **How do different sequential models compare for one-step-ahead mobile network traffic forecasting, and how does their performance vary across geographical areas with different traffic characteristics?**

## Dataset

The project uses the Telecom Italia **Telecommunications - SMS, Call, Internet - MI** dataset. The data contain telecommunications activity across a 100 x 100 grid over Milan at 10-minute intervals.

Dataset sources:

- Telecommunications data: https://doi.org/10.7910/DVN/EGZHFV
- Milano Grid: https://doi.org/10.7910/DVN/QJWLFU

The complete raw dataset is not included in this repository because of its size. The first notebook downloads and processes the source data. More information is available in [data/README.md](data/README.md).

## Repository structure

```text
Time-Series-Forcasting/
├── README.md
├── requirements.txt
├── .gitignore
├── data/
│   └── README.md
├── notebooks/
│   ├── 01_Data_Handling.ipynb
│   ├── 02_Exploratory_Analysis.ipynb
│   └── 03_Forecasting_Models.ipynb
├── models/
│   ├── RNN_5059.keras
│   ├── RNN_5161.keras
│   ├── RNN_5259.keras
│   ├── GRU_5059.keras
│   ├── GRU_5161.keras
│   ├── GRU_5259.keras
│   ├── LSTM_5059.keras
│   ├── LSTM_5161.keras
│   └── LSTM_5259.keras
├── scalers/
│   ├── scaler_5059.pkl
│   ├── scaler_5161.pkl
│   └── scaler_5259.pkl
└── results/
    ├── processing_log.csv
    ├── training_results.csv
    └── test_results_dec16_dec22.csv
```

## Environment

The project was developed and executed in **Google Colab**. The forecasting notebook records **TensorFlow 2.20.0**.

The notebooks use Google Drive to persist the processed dataset, experiment outputs, figures, trained models, and other intermediate files.

## Setup

### 1. Clone the repository

```bash
git clone https://github.com/Gaddy01/Time-Series-Forcasting.git
cd Time-Series-Forcasting
```

### 2. Install the dependencies

For a local Python environment:

```bash
python -m pip install -r requirements.txt
```

In Google Colab, most common packages are already available. Install any missing packages from `requirements.txt` before running the notebooks.

### 3. Prepare Google Drive

The notebooks were run with the following project directory:

```text
MyDrive/
└── ML Techniques I/
    └── Formative 1/
        ├── data/
        ├── models/
        └── results/
            ├── figures/
            ├── tables/
            └── logs/
```

When a notebook requests Google Drive access, mount Drive and ensure the project directory exists at:

```text
/content/drive/MyDrive/ML Techniques I/Formative 1
```

If a different location is used, update the corresponding path variables in the notebooks.

## Running the project

Run the notebooks in numerical order.

### 1. Data handling

Open:

`notebooks/01_Data_Handling.ipynb`

This notebook:

- retrieves the original daily Telecom Italia files from Harvard Dataverse
- processes the large dataset incrementally to limit memory use
- aggregates telecommunications activity by grid square and timestamp
- records processing time and memory measurements
- combines the processed data into `telecom_milan_master.parquet`
- copies the master dataset and supporting outputs to Google Drive

The complete source data are large, so this stage requires sufficient storage space and download time.

### 2. Exploratory analysis

Open:

`notebooks/02_Exploratory_Analysis.ipynb`

This notebook:

- calculates total Internet traffic for each geographical area
- identifies the three highest-traffic squares
- compares the selected areas over the first two weeks
- examines normalized traffic patterns
- studies autocorrelation
- examines daily and weekly periodicity

The three highest-total-traffic areas used for forecasting are **5161, 5059, and 5259**.

### 3. Forecasting models

Open:

`notebooks/03_Forecasting_Models.ipynb`

This notebook:

- loads Internet traffic for squares 5161, 5059, and 5259
- creates chronological train, validation, and test splits
- fits a separate `StandardScaler` on the training data for each square
- constructs one-step-ahead sequences
- trains and compares RNN, GRU, and LSTM models
- records validation metrics and training time
- evaluates the final models on the held-out test period
- generates actual-versus-predicted forecast plots

## Forecasting protocol

Internet traffic is sampled every 10 minutes. The final models use a sequence length of **144 observations**, representing the previous 24 hours, to predict the next 10-minute observation.

The chronological split is:

| Split | Period | Observations |
| --- | --- | ---: |
| Training | 1 Nov to 8 Dec 2013 | 5,472 |
| Validation | 9 Dec to 15 Dec 2013 | 1,008 |
| Test | 16 Dec to 22 Dec 2013 | 1,008 |

The scaler for each area is fitted on training data only and then applied unchanged to validation and test data.

The final model configurations are:

| Model | Recurrent layers | Learning rate | Sequence length |
| --- | --- | ---: | ---: |
| RNN | 64, 64 | 0.0001 | 144 |
| GRU | 64 | 0.001 | 144 |
| LSTM | 64 | 0.001 | 144 |

All models use a batch size of 64, mean squared error as the training loss, the Adam optimizer, and early stopping on validation loss.

## Held-out test results

The final evaluation covers 16 to 22 December 2013.

| Square | Model | MAE | RMSE | MAPE |
| ---: | --- | ---: | ---: | ---: |
| 5161 | RNN | 85.93 | 126.69 | 9.36% |
| 5161 | GRU | 85.43 | 127.62 | 8.81% |
| 5161 | LSTM | 93.42 | 135.12 | 11.55% |
| 5059 | RNN | 74.87 | 100.64 | 9.39% |
| 5059 | GRU | 71.38 | 100.06 | 7.83% |
| 5059 | LSTM | 79.68 | 109.54 | 9.81% |
| 5259 | RNN | 63.26 | 90.33 | 7.08% |
| 5259 | GRU | 67.67 | 96.14 | 7.46% |
| 5259 | LSTM | 66.34 | 95.34 | 7.43% |

The results show that performance varies across geographical areas. GRU records the lowest MAE and MAPE for square 5161 and the lowest values for all three reported metrics for square 5059. RNN records the lowest values for all three metrics for square 5259. No single architecture gives the lowest error for every area and metric.

## Saved artifacts

### Models

The `models/` directory contains the final saved Keras models for all three architectures and all three forecast areas.

### Scalers

The `scalers/` directory contains the fitted `StandardScaler` objects used for the three forecast areas. These are required when reproducing predictions from the saved models.

### Results

The `results/` directory contains:

- `processing_log.csv`: file-level download, processing, memory, row-count, and output-size measurements
- `training_results.csv`: validation performance, model settings, epochs, and training times
- `test_results_dec16_dec22.csv`: held-out test MAE, RMSE, and MAPE for the nine final model-area combinations

## Reproducibility notes

Random seeds are set to 42 in the forecasting workflow for Python, NumPy, and TensorFlow. Training uses `shuffle=False` to preserve time order.

The saved notebooks contain their executed outputs, which document the experiment process and results. Exact training time can vary with the available Colab hardware.

The large raw and processed datasets are intentionally excluded from GitHub. Start with `01_Data_Handling.ipynb` to reproduce the data preparation process from the original Dataverse source.

## Main technologies

- Python
- pandas
- NumPy
- PyArrow
- Matplotlib
- scikit-learn
- TensorFlow / Keras
- Google Colab
- Google Drive

## Author

**Gaddiel Irakoze**

African Leadership University  
BSc Software Engineering
