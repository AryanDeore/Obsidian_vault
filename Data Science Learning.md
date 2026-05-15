[Gabriel Atkin](https://www.youtube.com/@gcdatkin)
Kaggle: House Price Prediction

`pd.set_options('max_columns', None)`: Lets you view all the columns
`pd.set_options('max_rows', 100)`: Lets you view all the columns

## Steps
### Data Cleaning
#### Ensuring correct Data types for columns:
1. Ordinal features are sometimes set as integers. Convert them back to a string. 
#### Missing value imputation:
1.  **Categorical** feature:  you can only use `mode` to fill out the missing categorical values, or you need to look at each column and figure out if the missing values need to be handled or if they mean something.


### Feature Engineering
1. Normalize: Division, Ratio
2. Combine: +  x  ^
3. Scale:  $x^2$, $x^3$, log(x), x!, $\sqrt{x}$
4. 