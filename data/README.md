# Data

The full Telecom Italia mobile activity dataset and the processed master dataset are not stored in this repository because of their size.

## Dataset source

This project uses the **Telecommunications - SMS, Call, Internet - MI** dataset released by Telecom Italia through Harvard Dataverse.

- Telecommunications dataset: https://doi.org/10.7910/DVN/EGZHFV
- Milano Grid: https://doi.org/10.7910/DVN/QJWLFU

The data cover mobile network activity across a 100 x 100 grid over Milan. Activity is recorded at 10-minute intervals. The original records contain SMS, call, and Internet activity.

## Creating the processed dataset

Run:

`notebooks/01_Data_Handling.ipynb`

The notebook retrieves the source files from Harvard Dataverse, processes them one file at a time to control memory use, aggregates records by square and timestamp, and creates the master Parquet dataset used by the remaining notebooks.

The main processed file is:

`telecom_milan_master.parquet`

In the Google Colab workflow used for this project, the master file is copied to:

`/content/drive/MyDrive/ML Techniques I/Formative 1/data/telecom_milan_master.parquet`

## Why the dataset is excluded from GitHub

The original daily text files occupy about 20.8 GB in total, while the combined processed Parquet file is about 1.87 GB. Keeping these files outside GitHub keeps the repository small while preserving a reproducible data preparation pipeline.

See the root `README.md` for the full setup and notebook execution order.
