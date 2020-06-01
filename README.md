# Attempt to shrink Pandas `dtypes` without losing data so you have more RAM (and maybe more speed)

Status - early alpha, written in 2 hours on a Sunday. Suggestions welcome, I may accept PRs but you're better off asking first (via a bug report) with the suggestion in case it isn't where I want to take the library. I'm also very happy to have "Thanks" posted via bugs too if this helps you out :-)

This tool checks each column to see if larger dtypes (e.g. 8 byte `float64` and `int64`) could be shrunk to smaller `dtypes` without causing any data loss. 
Dropping an 8 byte type to a 4 (or 2 or 1 byte) type will keep halving the RAM requirement for that column.  Categoricals are proposed for `object` columns which can bring significant speed and RAM benefits.

Looking at `__main__` and try `report_on_dataframe(your_df)` to get a printed report - no changes are made to your dataframe.


