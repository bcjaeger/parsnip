


For this engine, there is a single mode: regression

## Tuning Parameters

This model has no tuning parameters.

## Important engine-specific options

Some relevant arguments that can be passed to `set_engine()`: 

 * `chains`: A positive integer specifying the number of Markov chains. The default is 4.
 * `iter`: A positive integer specifying the number of iterations for each chain (including warmup). The default is 2000.
 * `seed`: The seed for random number generation. 
 * `cores`: Number of cores to use when executing the chains in parallel.
 * `prior`: The prior distribution for the (non-hierarchical) regression coefficients. 
 * `prior_intercept`: The prior distribution for the intercept (after centering all predictors). 
 
See `?rstanarm::stan_glmer` and `?rstan::sampling` for more information.

## Translation from parsnip to the original package

The **multilevelmod** extension package is required to fit this model.


```r
library(multilevelmod)

linear_reg() %>% 
  set_engine("stan_glmer") %>% 
  set_mode("regression") %>% 
  translate()
```

```
## Linear Regression Model Specification (regression)
## 
## Computational engine: stan_glmer 
## 
## Model fit template:
## rstanarm::stan_glmer(formula = missing_arg(), data = missing_arg(), 
##     weights = missing_arg(), family = stats::gaussian, refresh = 0)
```


## Predicting new samples

This model can use subject-specific coefficient estimates to make predictions (i.e. partial pooling). For example, this equation shows the linear predictor ($\eta$) for a random intercept: 

$$
\eta_{i} = (\beta_0 + b_{0i}) + \beta_1x_{i1}
$$ 

where $i$ denotes the `i`th independent experimental unit (e.g. subject). When the model has seen subject `i`, it can use that subject's data to adjust the _population_ intercept to be more specific to that subjects results. 

What happens when data are being predicted for a subject that was not used in the model fit? In that case, this package uses _only_ the population parameter estimates for prediction: 

$$
\hat{\eta}_{i'} = \hat{\beta}_0+ \hat{\beta}x_{i'1}
$$ 

Depending on what covariates are in the model, this might have the effect of making the same prediction for all new samples. The population parameters are the "best estimate" for a subject that was not included in the model fit.  

The tidymodels framework deliberately constrains predictions for new data to not use the training set or other data (to prevent information leakage). 


## Preprocessing requirements

There are no specific preprocessing needs. However, it is helpful to keep the clustering/subject identifier column as factor or character (instead of making them into dummy variables). See the examples in the next section. 

## Other details

The model can accept case weights. 

With parsnip, we suggest using the formula method when fitting: 

```r
library(tidymodels)
data("riesby")

linear_reg() %>% 
  set_engine("stan_glmer") %>% 
  fit(depr_score ~ week + (1|subject), data = riesby)
```

When using tidymodels infrastructure, it may be better to use a workflow. In this case, you can add the appropriate columns using `add_variables()` then supply the typical formula when adding the model: 

```r
library(tidymodels)

glmer_spec <- 
  linear_reg() %>% 
  set_engine("stan_glmer")

glmer_wflow <- 
  workflow() %>% 
  # The data are included as-is using:
  add_variables(outcomes = depr_score, predictors = c(week, subject)) %>% 
  add_model(glmer_spec, formula = depr_score ~ week + (1|subject))

fit(glmer_wflow, data = riesby)
```

For prediction, the `"stan_glmer"` engine can compute posterior intervals analogous to confidence and prediction intervals. In  these instances, the units are the original outcome. When  `std_error = TRUE`, the standard deviation of the posterior  distribution (or posterior predictive distribution as  appropriate) is returned.

## References

 - McElreath, R. 2020 _Statistical Rethinking_. CRC Press.

 - Sorensen, T, Vasishth, S. 2016. Bayesian linear mixed models using Stan: A tutorial for psychologists, linguists, and cognitive scientists, 	arXiv:1506.06201.