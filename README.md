# dtype_diet
> Attempt to shrink Pandas `dtypes` without losing data so you have more RAM (and maybe more speed)


```python
#slow
```

This file will become your README and also the index of your documentation.

## Install

`pip install dtype_diet`

# Documentation
https://noklam.github.io/dtype_diet/

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
proposed_df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Current dtype</th>
      <th>Proposed dtype</th>
      <th>Current Memory (MB)</th>
      <th>Proposed Memory (MB)</th>
      <th>Ram Usage Improvement (MB)</th>
      <th>Ram Usage Improvement (%)</th>
    </tr>
    <tr>
      <th>Column</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>store_id</th>
      <td>object</td>
      <td>category</td>
      <td>203763.920410</td>
      <td>3340.907715</td>
      <td>200423.012695</td>
      <td>98.360403</td>
    </tr>
    <tr>
      <th>item_id</th>
      <td>object</td>
      <td>category</td>
      <td>233039.977539</td>
      <td>6824.677734</td>
      <td>226215.299805</td>
      <td>97.071456</td>
    </tr>
    <tr>
      <th>wm_yr_wk</th>
      <td>int64</td>
      <td>int16</td>
      <td>26723.191406</td>
      <td>6680.844727</td>
      <td>20042.346680</td>
      <td>74.999825</td>
    </tr>
    <tr>
      <th>sell_price</th>
      <td>float64</td>
      <td>None</td>
      <td>26723.191406</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



Recommendations:

* Run `report_on_dataframe(your_df)` to get recommendations
* Run `optimize_dtypes(df, proposed_df)` to convert to recommeded dtypes.
* Consider if Categoricals will save you RAM (see Caveats below)
* Consider if f32 or f16 will be useful (see Caveats - f32 is _probably_ a reasonable choice unless you have huge ranges of floats)
* Consider if int32, int16, int8 will be useful (see Caveats - overflow may be an issue)
* Look at https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.convert_dtypes.html which recommends Pandas nullable dtype alternatives (e.g. to avoid promoting an int64 with NaN items to float64, instead you get Int64 with NaNs and no data loss)
* Look at Extension arrays like https://github.com/JDASoftwareGroup/rle-array (thanks @repererum [for the tweet](https://twitter.com/crepererum/status/1267441357339201536))

Look at `report_on_dataframe(your_df)` to get a printed report - no changes are made to your dataframe.

## Caveats

* reduced numeric ranges might lead to overflow (TODO document)
* category dtype can have unexpected effects e.g. need for observed=True in groupby (TODO document)
* f16 is likely to be simulated on modern hardware so calculations will be 2-3* slower than on f32 or f64
* we could do with a link that explains binary representation of float & int for those wanting to learn more

## Development 


### Contributors

* Antony Milbourne https://github.com/amilbourne
* Mani https://github.com/neomatrix369

### Local Setup

```
$ conda create -n dtype_diet python=3.8 pandas jupyter pyarrow pytest
$ conda activate dtype_diet
```

# Contributing
The repository is developed with `nbdev`, a system for developing library with notebook.

Make sure you run this if you want to contribute to the library. For details, please refer to nbdev documentation (https://github.com/fastai/nbdev)
```
nbdev_install_git_hooks
```

Some other useful commands
```
nbdev_build_docs
nbdev_build_lib
nbdev_test_nbs
```
