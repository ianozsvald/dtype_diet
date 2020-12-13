# dtype_diet
> Attempt to shrink Pandas `dtypes` without losing data so you have more RAM (and maybe more speed)


This file will become your README and also the index of your documentation.

## Install

`pip install dtype_diet`

## How to use

> This is a fork of https://github.com/ianozsvald/dtype_diet to continue supoprt and develop the library with approval from the original author @ianozsvald.

This tool checks each column to see if larger dtypes (e.g. 8 byte `float64` and `int64`) could be shrunk to smaller `dtypes` without causing any data loss. 
Dropping an 8 byte type to a 4 (or 2 or 1 byte) type will keep halving the RAM requirement for that column.  Categoricals are proposed for `object` columns which can bring significant speed and RAM benefits.


Here's an minimal example with 3 lines of code running on a Kaggle dataset showing a reduction of 957 -> 85MB, you can find the notebook in the [repository](https://github.com/noklam/dtype_diet/01_example.ipynb):

```python
# sell_prices.csv.zip 
# Source data: https://www.kaggle.com/c/m5-forecasting-uncertainty/
import pandas as pd
from dtype_diet import report_on_dataframe, optimize_dtypes
df = pd.read_csv('data/sell_prices.csv')
proposed_df = report_on_dataframe(df, unit="MB")
new_df = optimize_dtypes(df, proposed_df)
print(f'Original df memory: {df.memory_usage(deep=True).sum()/1024/1024} MB')
print(f'Propsed df memory: {new_df.memory_usage(deep=True).sum()/1024/1024} MB')
```

    Original df memory: 957.5197134017944 MB
    Propsed df memory: 85.09655094146729 MB
    

```python
print(proposed_df.to_markdown())
```

    | Column     | Current dtype   | Proposed dtype   |   Current Memory (MB) |   Proposed Memory (MB) |   Ram Usage Improvement (MB) |   Ram Usage Improvement (%) |
    |:-----------|:----------------|:-----------------|----------------------:|-----------------------:|-----------------------------:|----------------------------:|
    | store_id   | object          | category         |              203764   |                3340.91 |                     200423   |                     98.3604 |
    | item_id    | object          | category         |              233040   |                6824.68 |                     226215   |                     97.0715 |
    | wm_yr_wk   | int64           | int16            |               26723.2 |                6680.84 |                      20042.3 |                     74.9998 |
    | sell_price | float64         |                  |               26723.2 |                 nan    |                        nan   |                    nan      |
    

Recommendations:

* Run `report_on_dataframe(your_df)` to get recommendations
* Run `optimize_dtypes(df, proposed_df)` to convert to recommeded dtypes.
* Consider if Categoricals will save you RAM (see Caveats below)
* Consider if f32 or f16 will be useful (see Caveats - f32 is _probably_ a reasonable choice unless you have huge ranges of floats)
* Consider if int32, int16, int8 will be useful (see Caveats - overflow may be an issue)
* Look at https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.convert_dtypes.html which recommends Pandas nullable dtype alternatives (e.g. to avoid promoting an int64 with NaN items to float64, instead you get Int64 with NaNs and no data loss)
* Look at Extension arrays like https://github.com/JDASoftwareGroup/rle-array (thanks @repererum [for the tweet](https://twitter.com/crepererum/status/1267441357339201536))

Look at `report_on_dataframe(your_df)` to get a printed report - no changes are made to your dataframe.
