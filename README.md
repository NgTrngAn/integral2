This is one of the most intuitive function in R to approximate an integral in 2D. Basically, you supply the function with a function f: f(x, y) that takes in 2 value and output one numeric value, as long as the rectangle on which you'd like to approximate your integral. Here is how to use it, as described in the manual:

```{r}
func <- function(x, y){ sin(x) * cos(y) } # example function

integral2(func, xmin, xmax, ymin, ymax, ...)
```

Basically, the function f that I'd like to use is the density of a mixture of Gaussian distribution. 

$$f(x) = \sum_{i=1}^L w_i f_i(x)$$ 

where $$f_i(x) \sim N(\mu_i, \Sigma_i), \sum_{i=1}^Lw_i = 1$$

I've already had the code for it from a different project, and it's in Rcpp, so naturally I'd like to adjust the code just a little bit. Below is my code in Rcpp. The first function returns the density of a multivariate Gaussian, and is called by the second function to compute each component of the mixture. 
```{r}
#include <RcppArmadillo.h>
#include <cmath>
#include <vector>
using namnespace arma;


//[[Rcpp::depends(RcppArmadillo)]]
//[[Rcpp::export]]
double compute_gaussian_pdf(vec x, vec mean, mat cov) {
    //return the pdf of a multivariate Gaussian
    int d = x.n_elem;
    double log_pdf = - d * 0.5 * log(2 * M_PI) + -0.5 * log(det(cov)) + - 0.5 * ((x-mean).t() * inv(cov) * (x-mean)).eval()(0, 0);

    return exp(log_pdf);
}


//[[Rcpp::export]]
double compute_mixture_pdf(vec x, Rcpp::List means, Rcpp::List covs, vec weights) {

    double pdf = 0;

    for (int i = 0; i < weights.n_elem; i++) {
        if (exp(compute_log_pdf(x, means[i], covs[i]))) {
            pdf += weights(i) * compute_gaussian_pdf(x, means[i], covs[i]);
        }
    }

    return pdf;
}
```

Now to build and call the function from R:
```{r}
library(Rcpp)
library(pracma)
sourceCpp("functions.cpp")

integral2(compute_mixture_pdf,
    xmin = -10,
    xmax = 10,
    ymin = -10,
    ymax = 10,
    means = list(c(1, 2), c(3, 4)),
    covs = list(diag(c(1, 1)), diag(c(4, 4))),
    weights = c(0.5, 0.5)
)
```

And this is the error that I ran into: 
```{r}
Error in fun(x, y, ...) : unused argument (y)
```
Hmmm, this is weird. Obviously my function `compute_mixture_pdf` is expecting 2 `double` inputs and return one value of type `double`, just like the example at the top of this page. However, looking into the documentation of `integral2`, I saw this:

```diff
The function fun itself must be fully vectorized: It must accept arrays X and Y and return an array Z = f(X,Y) of corresponding values. 
```

So I need to vectorize my code in Rcpp. Shouldn't be too much of a hassle though, as I can just loop through the vectors x, y and call the function `compute_gaussian_pdf`. The code is shown below.

```{r}
//[[Rcpp::export]]
vec compute_mixture_pdf(vec x, vec y, Rcpp::List means, Rcpp::List covs, vec weights) {
    
    vec out = vec(x.n_elem);
    double pdf = 0;
    vec coordinates = vec(2);

    for (int i = 0; i < x.n_elem; i++) {
        for (int j = 0; j < weights.n_elem; j++) {
            coordinates(0) = x(i);
            coordinates(1) = y(i); 
            pdf += weights(j) * compute_gaussian_pdf(coordinates, means[j], covs[j]);
        }
        out(i) = pdf;
    }

    return out;
}
```
Unfortunately, another error arise, this time even more confusing:

```diff
Error in Z * temp : non-conformable arrays
```
