# Pandas Style Guide

Pandas is a package for data manipulation and analytics in Python. It is highly popular in both the scientific and commercial communities. Pandas is well suited for ad-hoc exploratory work, prototyping and use in production systems due to its productivity and efficiency. Pandas is highly flexible, however, this leads to there being more than one way to skin a Panda.

This opinionated guide, about Pandas and using DataFrames, presents best practices to write code that is more consistent, reliable, maintainable and readable by practitioners. This is mainly aimed at code which is used in production systems and not ad-hoc exploratory work.

*Why do we need this?* Users of Pandas have a diverse background (e.g. Data Scientist, Data Engineer, Researcher, Software Engineer) and language experience (e.g. SQL, MATLAB, Java) which can lead to various coding style.

# Column selection

```python
# Good
df['column']

# Bad
df.column
```

When selecting a column in a dataframe users should use dictionary like selection e.g. `df['column']` and not property selection e.g. `df.column`.

Why:
* It makes it more explict to the user that you are accessing a column and not a standard property or method
* Not all column names can be represented as a property - it must be a valid Python variable name. The column name also must not clash with an existing method or property

# Avoid chained indexing

```python
# Good
df.loc[df['A'] > 1, 'B'] = 1

# Bad (chained indexing)
df[df['A'] > 1]['B'] = 1
```

Indexing or slicing a dataframe twice can lead to unintended behaviour. In the second "chained indexing" example the column `B` **does not** get set with value `1`. A warning stating that "A value is trying to be set on a copy of a slice from a DataFrame." is also shown by Pandas.

In these circumstances use the `loc` and `iloc` methods to select both rows and columns.

Sometimes chained indexing is unavoidable. In these circumstances explicitly copy the dataframe. For example:

```python
df2 = df[df['A'] > 1].copy()
df2['B'] = 1
df2['C'] = 2
```

# Copy vs re-assignment

```python
# Good
df = df.drop(columns='A')

# Bad
df.drop(columns='A', inplace=True)
```

Some operations can either be performed inplace or by re-assignment. Users should always opt to re-assignment for better readability. Both operation, with a few exceptions, perform a data copy and therefore don't have any performance differences.

# Schema contract

```python
# Good (file input)
df = pd.read_csv('data.csv')
df = df[['col1', 'col2', 'col3']]

# Good (collection input)
df = pd.DataFrame.from_records(list_of_dicts)
df = df[['col1', 'col2', 'col3']]
```

After a DataFrame initialisation, users should explicitly select the columns to use, even if all are selected. This creates a clear contract between the code and user about what is expected in the downstream data schema.

It is also recommended to explicitly specify data types as well. When reading files data types can subtly change with small alterations e.g. from integer to float, or numerical to string. It is helpful for the program to throw an error if the input is unexpected, otherwise it will continue silently.

```python
# Good
df = pd.read_csv('data.csv', dtype={'col1': 'str', 'col2': 'int', 'col3': 'float'})
df = df[['col1', 'col2', 'col3']]
```

# Merge contract (join)

```python
# Good
df_3 = df_1.merge(
    df_2,
    how='inner',
    on='col1',
    validate='1:1'
)

# Bad
df_3 = df_1.merge(df_2)
```

Users should explicitly specify the `how` and `on` parameters of a merge for readability, even if the default parameters would produce the same result.

It is also important to validate the merge type using the `validate` parameter, this prevents unexpected "merge duplication". Upstream data can subtly changed which produces multiple rows per merge key. If no validation is performed, the data here will silently multiply and duplicate, as the operation will be promoted to a "many-to-one" or "many-to-many" producing "merge duplication". It is therefore useful to have the merge assumptions explicitly stated.

Also, don't deduplicate rows after a merge to remove merge duplication. Remove duplicates before joining, or even better determine why there are unexpected duplicate keys and remove them upstream. Merge duplication are computationally and memory expensive and produce hard to debug data bugs.

# Empty columns

```python
# Good
df['new_col_float'] = pd.NA
df['new_col_int'] = pd.Series(dtype='int')
df['new_col_str'] = pd.Series(dtype='object')

# Bad
df['new_col_int'] = 0
df['new_col_str'] = ''
```

If a new empty column is needed always use `pd.NA` values. Never use "filler" values such as zeros or empty strings. This preserves the ability to use methods such as `isnull` or `notnull`.

# Querying data

```python
# Good
df[df['A'] > df['B']]

# Bad
df.query('A > B')
```

Use idiomatic Pandas querying instead of SQL like string querying.

# Mutability of DataFrames

```python
# Bad
def func_1(df: pd.DataFrame) -> pd.DataFrame:
    df['new_col'] = df['col1'] + 1
    return df

df = func_1(df)
```

In large code bases, it can be tempting to keep adding columns to a dataframe and use it as the data exchange format between methods and functions. This however leads to code which is difficult to maintain as readers can't directly determine the schema of a DataFrame without running the code. It is more preferable to use dataclasses as an interchange format as the class definition is effectively immutable and specified explicitly (or if Python < 3.6 use namedtuples).

# Contribution and contributors

Contributions and discussion are highly welcomed. Please create an issue or pull request.

