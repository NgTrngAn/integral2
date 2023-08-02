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
double compute_gaussian_pdf(vec x, vec mean, mat cov) {
    //return the pdf of a multivariate Gaussian
    int d = x.n_elem;
    double log_pdf = - d * 0.5 * log(2 * M_PI) + -0.5 * log(det(cov)) + - 0.5 * ((x-mean).t() * inv(cov) * (x-mean)).eval()(0, 0);

    return exp(log_pdf);
}

double compute_log_mixture_pdf(vec x, Rcpp::List means, Rcpp::List covs, vec weights) {

    double pdf = 0;

    for (int i = 0; i < weights.n_elem; i++) {
        if (exp(compute_log_pdf(x, means[i], covs[i]))) {
            pdf += weights(i) * compute_gaussian_pdf(x, means[i], covs[i]);
        }
    }

    return pdf;
}
```

And this is the error that I ran into: 


Hmmm, this is weird. Obviously my function is expecting 2 `double` inputs and produce one value of type `double`. 
