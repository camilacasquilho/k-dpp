# Spatial Sampling Design based on k-Determinantal Point Processes

These files serve as supplementary material to _Models and Monitoring Designs for Spatio-temporal 
Climate Data Fields_, by Camila Casquilho available online at The University of British Columbia (UBC)
[Theses and Dissertations collection](https://open.library.ubc.ca/cIRcle/collections/ubctheses/24/items/1.0314917).

This repository contains code implementing a a flexible sampling design strategy based on 
k-Determinantal Point Processes (k-DPPs). This sampling design is able to yield spatially-balanced designs, while imposing diversity in the selection of locations based on additional features that 
might be available. In its simple format, it can be viewed as a randomized alternative to 
space-filling "coverage" designs commonly used and available at `fields::cover.design`.

The k-DPP sampling design can also be used as an approximation for entropy design in Gaussian fields, or even for selecting knots for spatial predicton.

This research builds upon [Kulesza and Taskar (2012)](http://www.alexkulesza.com/pubs/dpps_fnt12.pdf)
where a spatial sampling design based on k-DPPs is introduced.
Code is based on [Alex Kulezsa's](http://www.alexkulesza.com/) MATLAB code.


## Dependencies

* `assertthat::assert_that` 
* `fields::rdist`, 
* `foreach::foreach`, 
* ``` foreach::`%do%` ```, 
* ``` foreach::`%dopar%` ``` (optional), 
* ``` magrittr::`%>%` ```,
* ```purrr::map```,
* ```purrr::map_dbl```,
* `rlist::list.rbind`.

Firstly, it is important to check if those packages have been previously installed in your machine. 
If any of those packages have not yet been installed, you need to install them prior to using the 
k-DPP functions provided here. 

```{r}
required.packages <- c("assertthat", "fields", "foreach", "magrittr", "purrr", "rlist")
# Check which packages need installation
need.installation <- required.packages[which(!(required.packages %in% installed.packages()))] 
if (length(need.installation) != 0) {
  install.packages(need.installation)
}
```

## R Scripts  
* `helper-functions.R`: Contains helper functions used by `dpp-sample.R`.
* `dpp-sample.R`: Contains functions to simulate a single realization of a k-DPP and to empirically
obtain the best configuration based on repeateadly sampling from a given k-DPP.


## Example

```{r}
library(magrittr)
library(ggplot2)

# Source Files
source("helper-functions.R")
source("kdpp-sample.R")

# Create artificial grid of points
ngrid <- 40
grid.candidates <- expand.grid(x = seq(0, 1, length.out = ngrid),
                               y = seq(0, 1, length.out = ngrid))
k <- 60

# Uniformly at random
set.seed(1)
random.sample <- sample(seq(1, ngrid * ngrid), size = k)
random.df <- grid.candidates[random.sample, ] %>% 
  dplyr::mutate(method = "Random")

# Space-filling
set.seed(2)
sf.sample <- fields::cover.design(grid.candidates, nd = k)$best.id
sf.df <- grid.candidates[sf.sample, ] %>% 
  dplyr::mutate(method = "Space-filling")

# k-DPP
# Describe a L-ensemble matrix using a Gaussian kernel
kernel.bandwidth <- 0.1
L <- gaussian.kernel(grid.candidates, tau.sq = kernel.bandwidth) 
L.decomposed <- decompose.matrix(L)
set.seed(3)
kdpp.sample <- k.dpp.sample(L.decomposed, k)
kdpp.df <- grid.candidates[kdpp.sample, ] %>% 
  dplyr::mutate(method = "k-DPP")

dplyr::bind_rows(random.df, sf.df, kdpp.df) %>% 
   ggplot(aes(x = x, y = y)) + 
   geom_point() + 
   facet_wrap(~ method) + 
   theme_bw()
```

## Contact
Camila Casquilho  
camila.casquilho@stat.ubc.ca

### License: Attribution-ShareAlike 4.0 International


