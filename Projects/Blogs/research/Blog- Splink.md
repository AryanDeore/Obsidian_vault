Splink is a free probabilistic record linkage library that predicts the likelihood that two records refer to the same entity.

However, there’s a paradox at the heart of the problem of training a record linkage model: To estimate the parameters of the model, you need to know which records match. But finding which records match is the whole problem you’re trying to solve.

The Expectation Maximisation (EM) algorithm provides a solution to this paradox. **What kind of problems does EM algo solve?**

