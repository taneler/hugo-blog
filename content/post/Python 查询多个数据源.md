+++
title = 'Python 查询多个数据源数据'
date = 2024-07-12T13:47:26+08:00
draft = false
slug = '99165d15-b66a-4daa-bd67-132e66172e11'
+++

## 需求场景

我有一个业务系统，按不同的处理实时性分开部署了两套环境，因此在出数据报表时很麻烦，所以想查询多个数据源的数据，再合并到一起输出数据报表。

## 实现逻辑

通过 Python 使用 cx_Oracle 连接到两个数据库，分别查询所需要的数据，并将查询结果加载到 pandas 的 DataFrame 中，再使用 `concat` 进行纵向合并。

```python
import cx_Oracle
import pandas as pd

# 设置Oracle数据库连接信息
dsn1 = cx_Oracle.makedsn('hostname1', port1, service_name='service_name1')
dsn2 = cx_Oracle.makedsn('hostname2', port2, service_name='service_name2')

# 建立连接
conn1 = cx_Oracle.connect(user='username1', password='password1', dsn=dsn1)
conn2 = cx_Oracle.connect(user='username2', password='password2', dsn=dsn2)

# 查询数据
query1 = "SELECT * FROM table1"
query2 = "SELECT * FROM table2"

# 将查询结果加载到 DataFrame 中
df1 = pd.read_sql(query1, conn1)
df2 = pd.read_sql(query2, conn2)

# 关闭连接
conn1.close()
conn2.close()

# 合并数据
merged_df = pd.concat([df1, df2], ignore_index=True)

# 显示合并后的数据
print(merged_df)

# 如果需要将合并后的数据保存到文件中，例如 CSV 文件
merged_df.to_csv('merged_data.csv', index=False)
```

