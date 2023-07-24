This is one of the most intuitive function in R to approximate an integral in 2D. Basically, you supply the function with a function f: f(x, y) that takes in 2 value and output one numeric value, as long as the rectangle on which you'd like to approximate your integral. Here is how to use it, as described in the manual:

```{r}
func <- function(x, y){ sin(x) * cos(y) } # example function

integral2(func, xmin, xmax, ymin, ymax, ...)
```

Basically, the function f that I'd like to use is the density of a mixture of Gaussian distribution. I've already had the code for it from a different project, and it's in Rcpp, so naturally I'd like to adjust the code just a little bit. 

Here is my code in Rcpp: 


And this is the error that I ran into: 


Hmmm, this is weird. Obviously my function is expecting 2 `double` inputs and produce one value of type `double`. 
