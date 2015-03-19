# Let the Data Flow: Pipelines in R with dplyr and magrittr

## Abstract 

> Pipelines were the best thing to happen in R in 2014. They let us transform 
messy, inside-out code like `sort(unique(round(xs, 2)))` into a clear chain of 
transformations like `xs %>% round(2) %>% unique %>% sort`. In this talk I lead 
a tutorial on how to use pipelines for data-cleaning, transformation and 
presentation with the packages `magrittr` and `dplyr`. For beginners, I also 
review some of the essential R functions to make the most of pipelines.
>
> Tristan is a PhD student in Communication Sciences and Disorders. He uses in 
R in the [Learning To Talk lab](http://learningtotalk.org) to model 
eye-tracking and speech perception data. [@tjmahr](https://twitter.com/tjmahr), 
[github.com/tjmahr](https://github.com/tjmahr). 



## Slides

I prepared three sets of slides:

* [Pipelines in R](http://rpubs.com/tjmahr/pipelines_2015)
* [dplyr: verbs for manipulating data-frames](http://rpubs.com/tjmahr/pipelines_2015)
* [Making pretty regression tables with pipes](http://rpubs.com/tjmahr/prettytables_2015)


## Resources

* [magrittr vignette](http://cran.r-project.org/web/packages/magrittr/vignettes/magrittr.html)
* [RStudio Data-Wrangling Cheatsheet](http://www.rstudio.com/wp-content/uploads/2015/02/data-wrangling-cheatsheet.pdf)
* [Core R vocabulary](http://adv-r.had.co.nz/Vocabulary.html)
* [Awesome R](https://github.com/qinwf/awesome-R)
* [Pipelines for Data Analysis](https://www.youtube.com/watch?v=40tyOFMZUSM) (dpylr/magrittr talk by Hadley Wickham)
* [Best Practices for Scientific Computing](http://journals.plos.org/plosbiology/article?id=10.1371/journal.pbio.1001745)
* [Data Science on the Command Line](http://datascienceatthecommandline.com/)
* [Unix Commands for Data Science](http://www.gregreda.com/2013/07/15/unix-commands-for-data-science/)
 
### Packages

* [magrittr](https://github.com/smbache/magrittr) for pipelines
* [dplyr](https://github.com/hadley/dplyr) for data-frame functions
* [broom](http://cran.r-project.org/web/packages/broom/index.html)
* [stringr](http://cran.r-project.org/web/packages/stringr/index.html) for 
  string manipulation functions
* [pipeR](https://github.com/renkun-ken/pipeR) an alternative pipeline package
  (that I haven't tried yet).

## License

Obviously, the GPL-2 license applies only to the code and words I wrote, which 
are in the `.Rpres` and `.md` files and are reproduced with markup in the 
`.html` files.
