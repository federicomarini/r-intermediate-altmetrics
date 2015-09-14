---
layout: page
title: Intermediate programming with R
subtitle: Testing with testit
minutes: 30
---



> ## Learning Objectives {.objectives}
>
> * Write assertion statements with `assert`
> * Confirm errors using `has_error`
> * Confirm warnings using `has_warning`
> * Use unit tests to confirm code is working as expected

Using assertion statements is a good first step to writing more reliable code.
Going the next step, we can pass inputs to a function and confirm that the result is what we expect.
Tests that check a function works properly are called unit tests (because each function is a unit of the overall code we are writing).
Writing tests gives us confidence that our code works in different situations,
serves as explicit documentation of how a function is supposed to work,
and alerts us to any changes due to software updates.

In this lesson we will use the simple testing framework in the package [testit][].
There are other more elaborate testing frameworks such as [RUnit][] and [testthat][] if you need more complicated testing in the future.
Also, as a caveat, R testing frameworks work best in the context of an R package.
They will be less flexible in our context testing functions that are not part of an R package.

[testit]: https://cran.r-project.org/web/packages/testit/index.html
[RUnit]: https://cran.r-project.org/web/packages/RUnit/index.html
[testthat]: https://cran.r-project.org/web/packages/testthat/index.html




### Using the testit package

Let's first load the package.


~~~{.r}
library("testit")
~~~

In the last lesson, we wrote assertion statements using `stopifnot`.
However, the error messages generated by `stopifnot` are cryptic unless we are familiar with the intracacies of a specific function.
This can be difficult when writing lots of code or returning to code written long ago.
The main function of the testit package is `assert`, which allows us to include a message that is printed if the assertion fails.
This makes it easier to interpret what went wrong.


~~~{.r}
assert("one equals one", 1 == 1)
assert("two plus two equals five", 2 + 2 == 5)
~~~



~~~{.output}
assertion failed: two plus two equals five

~~~



~~~{.error}
Error: 2 + 2 == 5 is not TRUE

~~~

### Writing unit tests for a function


~~~{.r}
calc_sum_stat <- function(df, cols) {
  stopifnot(dim(df) > 0,
            is.character(cols),
            cols %in% colnames(df))
  if (length(cols) == 1) {
    warning("Only one column specified. Calculating the mean will not change anything.")
  }
  df_sub <- df[, cols, drop = FALSE]
  stopifnot(is.data.frame(df_sub))
  sum_stat <- apply(df_sub, 1, mean)
  stopifnot(!is.na(sum_stat))
  return(sum_stat)
}
# Proper
sum_stat <- calc_sum_stat(counts_raw, c("wosCountThru2010", "f1000Factor"))
~~~


~~~{.r}
# Empty data frame
assert("Empty data frame throws error",
       has_error(calc_sum_stat(data.frame(),
                               c("wosCountThru2010", "f1000Factor"))))
# Non-character cols
assert("Non-character vector input for columns throws error",
       has_error(calc_sum_stat(counts_raw, 1:3)))
# Bad column names
assert("Column names not in data frame throws error",
       has_error(calc_sum_stat(counts_raw, c("a", "b"))))
# Issue warning since only one column
assert("Selecting only one column issues warning",
       has_warning(calc_sum_stat(counts_raw, "mendeleyReadersCount")))
# NA output
assert("NA in output throws error",
       has_error(calc_sum_stat(counts_raw,
                     c("wosCountThru2010", "facebookLikeCount"))))
~~~



### Challenge

> ## Write some tests {.challenge}
>
> Write tests for the function `my_mean` that you wrote in an earlier lesson.
> It should look something like this:
>
> 
> ~~~{.r}
> my_mean <- function(x) {
>   result <- sum(x) / length(x)
>   return(result)
> }
> ~~~
> 
> The input `x` is a numeric vector, and the output is the mean of the vector of numbers.
> Some ideas to get started:
>
> * Pass a vector where you know what the mean is, and assert that the result is correct
> * Include an `NA` in the vector where you know the result to see what happens.
Do you need to modify the code to pass the test?
> * Add some assertion statments to check the input `x`.
> Use `has_error` to test that the function throws an error when given bad input.
> * What should the function do if the user passes a vector of length one?
> Should a warning be issued to alert the user?