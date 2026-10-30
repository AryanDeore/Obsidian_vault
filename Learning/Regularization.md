
Every regularization technique modifies the loss function by adding a penalty term. Starting from ordinary least squares:

**L1- Lasso:**
1. lambda + |slope|
2. can shrink parameters all the way to 0
3. This helps when dataset has a lot of useless features

**L2: Ridge:**
1. lambda + (slope)^2
2. can shrink parameters to nearly 0, but never 0
- Useful when you have multicollinearity (correlated predictors) or more features than samples.

**ElasticNet**:
1. Combination of L1 norm and L2 norm
2. useful when there is a mix both useful and useless parameters. ie parameters are correlated.
3. use cross validation