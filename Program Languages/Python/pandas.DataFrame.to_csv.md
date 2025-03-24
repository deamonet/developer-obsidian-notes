
DataFrame.to_csv(_path_or_buf=None_, _*_, _sep=','_, _na_rep=''_, _float_format=None_, _columns=None_, _header=True_, _index=True_, _index_label=None_, _mode='w'_, _encoding=None_, _compression='infer'_, _quoting=None_, _quotechar='"'_, _lineterminator=None_, _chunksize=None_, _date_format=None_, _doublequote=True_, _escapechar=None_, _decimal='.'_, _errors='strict'_, _storage_options=None_)[[source]](https://github.com/pandas-dev/pandas/blob/v2.2.3/pandas/core/generic.py#L3797-L3984)

Write object to a comma-separated values (csv) file.

Parameters:

**float_format**str, Callable, default None

Format string for floating point numbers. If a Callable is given, it takes precedence over other numeric formatting parameters, like decimal.

```python
def cast_float1_2_int1(x):  
    int_x = int(x)  
    if int_x == 0 and x > 0.0000001:  
        return x  
    else:  
        return str(int_x)

df.to_csv(csv_path, index=None, float_format=cast_float1_2_int1)
```
