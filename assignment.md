# Part 1: Sampling Population

You already learned the conceptual differences between `population` and `sample` in the lecture. Now you are going to explore how sample size affects your estimation of population parameters.

Imagine that you are a data scientist working for a healthcare company who wants to know a obesity parameter `X` for the people living in San Francisco. 

Assume that X follows the following normal distribution:

```python
import scipy.stats as stats
rv = stats.norm(150,10)
```

Plot the PDF of X. What are the mean, median, and 95 percentile of X, respectively? 

Hint: To calculate the percentiles of a random variable, you can use the [scipy.stats.norm.ppf() function](https://docs.scipy.org/doc/scipy/reference/generated/scipy.stats.norm.html#scipy.stats.norm).

In reality you do not know the true distribution of X. But fortunately the company has already collected multiple samples of various sizes: `data/sample_50.csv`, `data/sample_500.csv`, `data/sample_5000.csv`. Please code a function to plot the empirical PDF of each sample. Compute the corresponding mean, median, and 95 percentile. How does the sample size affect your estimation of the true population parameters?

So far you are doing point estimation of those parameters, which does not really tells you `how good a sample statistic approximate a population parameter`. Next, you will be able answer the question by Bootstrapping.

# Part 2: The Bootstrap Approximation of Sampling Distributions

Let's first recap what you learned in the lecture.

### Introduction

The Bootstrap is a common, computationally intensive procedure used to approximate the sampling distribution of any sample statistic. The sample statistic is probably a point estimate of some unknown population parameter such as mean, median, max, min, percentile etc. 

Its power is derived from its generality: it applies to almost all sample statistics we may care about, while other methods (based primarily on the central limit theorem) only apply to the sample mean.

### Part 2.1: What is Bootstrapping and Why Do We Use It?

The general concept of bootstrapping is to create many samples from the one sample you actually have collected.

Hypothetically, since the sample we have actually collected is our best available approximation to the population we are studying, randomly redrawing new samples out of the original sample itself (with replacement) can simulate random samples from the original population.  In the bootstrap the statistic of interest is computed on each re-sample in order to provide an empirical approximation of the sampling distribution of that statistic. 

<br>

Implement a `bootstrap` function to randomly draw with replacement from a given sample. The function should take a sample as a `numpy ndarray` and the number of bootstrap samples as an integer  (`default: 10000`). The function should return a **list** of `numpy ndarray` objects, each `ndarray` is one bootstrap sample. 

   ```python
   def bootstrap(sample, num_bootstraps=10000):
       """Draw bootstrap re-samples from the array sample.

       Parameters
       ----------
       sample: a list of numbers, or a np.array with shape (n, ). The data to draw the bootstrap samples from.
       
       num_bootstraps: int
         The number of bootstrap samples, each is built by sampling from sample with replacement.
       
       Returns
       -------
       bootstrap_samples: a list of np.ndarray with length num_bootstraps. The bootstrap resamples from sample.
       """
   ```
   
   **Hint:**
   - Use `np.random.randint` to randomly draw the row indexes with replacement and then create the bootstrap sample by indexing the original sample with the randomly drawn row indexes
   - Or `np.random.choice` to randomly draw elements directly
 
<br>

### Part 2.2: Bootstrap to find Confidence Interval of Mean

The bootstrap can be used to provide very simple confidence intervals for a population parameter.  The empirical quantiles of the bootstrapped sampling distribution can be used as confidence intervals, usually called bootstrapped confidence intervals.

So for example, if, upon bootstrapping, you get

   ```python
   [1, 1, 1, 2, 2, 3, 4, 4, 5, 6]
   ```

as the bootstraped values of some sample statistic, then the interval `[1, 5]` would be a 80% confidence interval (where the 1 and the 5 are the 10th and 90th percentile of the array of boostrapped values).

Company X wants to find out if changing to Apple monitors increases its programmers' productivity. A random sample of 25 people is chosen and their monitors are switched. The difference between their productivity before and after the monitor switch is recorded in `data/productivity.txt`.
 
<br>

1. Load the data `data/productivity.txt`.

2. Why is it inappropriate to only report the mean difference in productivity as evidence to support the decision of changing all the monitors to Apple monitors in the company?

3. Implement a `bootstrap_ci` function to calculate the confidence interval of any sample statistic (in this case the `mean`). The function should take a sample, a function that computes the sample statistic, the number of resamples (`default: 10000`), and the confidence level (`default: 0.95`). The function should return the lower and upper bounds of the confidence interval and the bootstrap distribution of the sample statistic.
   
   You should be able to call the function in the following manner:
   
   ```python
   bootstrap_ci(sample, stat_function=np.mean, resamples=1000, ci=95)
   ```

   **Hint:**
   - `bootstrap_ci` should call your `bootstrap` function to create the bootstrap samples.
   - Apply (map) the sample statistic function to the bootstrap samples.
   - Use the **percentile interval method**: e.g., take the empirical 2.5% percentile and 97.5% percentile of the bootstrapped sampling distribution to create a 95% bootstrapped confidence interval.

4. Plot the bootstrap distribution of the means in a histogram. 

5. Based on the bootstrap confidence interval, what conclusions can you draw? What about if a 90% confidence interval were used instead? 
   
   Suppose there are 100 programmers in the company. The cost of changing a monitor is $500 and the increase of one unit of productivity is worth $2,000, would you recommend switching the monitors? State the assumptions you are making and show your work.
  
<br>


### Part 2.3: Bootstrap to find Confidence Interval of Correlation (optional)

You are interested if there is a positive correlation between the LSAT admission exam score and the first year GPA achieved in law schools. You are given the mean LSAT and mean GPA scores for the students from a sample of 15 law schools.

<br>

1. Load the data `data/law_sample.txt`.

2. Use `scipy.stats.pearsonr` to compute the correlation.
   
3. Use the `bootstrap_ci` function to compute the confidence interval for the correlation coefficient.  Based on the bootstrap confidence interval, what conclusions can you draw?

4. Plot the bootstrap distribution of the correlation in a histogram. 

5. Load in the LSAT and GPA data for all the law school (`data/law_all.txt`) and verify the population correlation is within the confidence interval.

Note: The percentile interval method (as implemented in `bootstrap_ci`) is less than optimal for building a confidence interval when the bootstrap distribution of the statistic is not nearly symmetrical. There are methods to correct for the confidence interval in such cases which will not be covered here. ([Studentized Bootstrap & Bias-Corrected Bootstrap](http://en.wikipedia.org/wiki/Bootstrapping_%28statistics%29#Methods_for_bootstrap_confidence_intervals)) 
