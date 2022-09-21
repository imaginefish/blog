---
title: Pandas DataFrame 之 groupby()
date: 2022-09-13 19:57:49
categories:
- Python 数据处理
tags: 
- Python
- Pandas
toc: true
---
## groupby()
groupby() 方法定义：
```python
DataFrame.groupby(
    by=None, 
    axis=0, 
    level=None, 
    as_index=True, 
    sort=True, 
    group_keys=True, 
    squeeze=NoDefault.no_default,
    observed=False, 
    dropna=True
    ) -> DataFrameGroupBy
```
<!--more-->
对一个 DataFrame 使用 `groupby()` 方法聚合后，返回的数据类型为 `pandas.core.groupby.generic.DataFrameGroupBy`，其实就是分组后的子 DataFrame，处理时将其当成 DataFrame 处理即可，调用 `agg()` 或 `apply()` 方法，完成相关的数据统计。
可以通过以下方法遍历 DataFrameGroupBy 类型数据：
```python
df = pd.DataFrame([
    {"year":"2022" ,"month":"09", "date":"2022-09-13", "min_temp":23, "max_temp":26},
    {"year":"2022" ,"month":"09", "date":"2022-09-14", "min_temp":22, "max_temp":27},
    {"year":"2022" ,"month":"09", "date":"2022-09-15", "min_temp":23, "max_temp":29}
])
df_g=df.groupby(['year','month'])

print(type(df_g))

for index,data in df_g:
    print(type(index))
    print(index)
    print(type(data))
    print(data)
```
```
<class 'pandas.core.groupby.generic.DataFrameGroupBy'>
<class 'tuple'>
('2022', '09')
<class 'pandas.core.frame.DataFrame'>
   year month        date  min_temp  max_temp
0  2022    09  2022-09-13        23        26
1  2022    09  2022-09-14        22        27
2  2022    09  2022-09-15        23        29
```
可以看出分组后的数据类型为 DataFrameGroupBy，其包含了 index 和 data，index 是包含了分组列的 tuple，data 是该组的子 DataFrame。
## agg() 
agg() 方法定义如下：
```python
DataFrameGroupBy.aggregate(
    func=None, 
    *args, 
    engine=None, 
    engine_kwargs=None, 
    **kwargs) -> DataFrame
```
以下给出几个示例：
- 选择某一列进行聚合
```python
df.groupby(['year','month']).max_temp.agg(['min','max','mean'])
```
```
		    min max mean
year	month			
2022	09	26	29	27.333333
```
- 给不同列分别聚合
```python
df.groupby(['year','month']).agg({'min_temp': ['mean'],'max_temp': ['min','max','mean']})
```
```
             min_temp max_temp               
                 mean      min max       mean
year month                                   
2022 09     22.666667       26  29  27.333333
```
- 重命名聚合后的列名称
```python
df.groupby(['year','month']).agg(
    min_temp_mean=pd.NamedAgg(column='min_temp', aggfunc='mean'),
    max_temp_mean=pd.NamedAgg(column='max_temp', aggfunc='mean'),
)
```
```
            min_temp_mean  max_temp_mean
year month                              
2022 09         22.666667      27.333333
```
## apply()
apply 可以按组应用函数 func 并将结果组合在一起，虽然 apply 是一种非常灵活的方法，但它的缺点是使用它可能比使用更具体的方法（如 agg 或 transform）慢很多。
- 求分组中 max_temp 的最大值与 min_temp 的最小值的差值
```python
df_g.apply(lambda x: x.max_temp.max() - x.min_temp.min())
```
```
year  month
2022  09       7
dtype: int64
```
## groupby() 后转为 DataFrame
- DataFrame.reset_index()
如果分组后使用 agg() 进行聚合，返回的是 DataFrame，但是索引列是原来的汇聚列，使用 reset_index(inplace=True) 可以将索引转化为列：
```python
df1=df_g.agg('min')
df1.reset_index(inplace=True)
print(df1)
```
```
   year month        date  min_temp  max_temp
0  2022    09  2022-09-13        22        26
```
- Series.to_frame()
如果分组后使用 apply() 进行聚合，则返回的是 Series，to_frame() 是将 Series 转化为 DataFrame 的方法，可以将任意 Series 转化为 DataFrame：
```python
sr = df_g.apply(lambda x: x.max_temp.max() - x.min_temp.min())
df1 = sr.to_frame('diff').reset_index()
print(df1)
```
```
   year month  diff
0  2022    09     7
```

