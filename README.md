
[![R-CMD-check](https://github.com/desanou/mglasso/actions/workflows/basic.yml/badge.svg)](https://github.com/desanou/mglasso/actions/workflows/basic.yml)

<!-- README.md is generated from README.Rmd. Please edit that file -->

## Method

This repository provides an implementation of the `MGLasso` (Multiscale
Graphical Lasso) algorithm: a novel approach for estimating sparse
Gaussian Graphical Models with the addition of a group-fused Lasso
penalty.

`MGLasso` is described in the paper [Inference of Multiscale Gaussian
Graphical Model](https://desanou.github.io/multiscale_glasso/).
`MGLasso` has three major contributions:

-   We simultaneously infer a network and estimate a clustering
    structure by combining the [neighborhood
    selection](https://arxiv.org/abs/math/0608017) approach (Meinshausen
    and Bühlman, 2006) and [convex
    clustering](https://www.di.ens.fr/~fbach/419_icmlpaper.pdf) (Hocking
    et al.2011).

-   We use a continuation with Nesterov smoothing in a
    shrinkage-thresholding algorithm
    ([`CONESTA`](https://arxiv.org/abs/1605.09658), Hadj-Selem et
    al. 2018) to solve the optimization problem .

-   We show numerically that `MGLasso` performs better than
    [`GLasso`](https://arxiv.org/abs/0708.3517)(Meinshausen and Bühlman, 2006; Friedman et al. 2007) in
    terms of support recovery in a block diagonal model with few sparsity inside blocks. The clustering performances of `MGLasso`
    can be improved by the addition of a weight term to calibrate the
    variable fusion regularizer.

To solve the `MGLasso` problem, we seek the regression vectors
*β*<sup>*i*</sup> that minimize

$$
J\_{\\lambda_1, \\lambda_2}(\\boldsymbol{\\beta}; \\mathbf{X} ) = 
  \\frac{1}{2} 
     \\sum\_{i=1}^p 
        \\left \\lVert 
            \\mathbf{X}^i - \\mathbf{X}^{\\setminus i} \\boldsymbol{\\beta}^i 
        \\right \\rVert_2 ^2  + 
  \\lambda_1 
    \\sum\_{i = 1}^p  
      \\left \\lVert 
         \\boldsymbol{\\beta}^i \\right \\rVert_1 + 
  \\lambda_2 
     \\sum\_{i \< j} 
        \\left \\lVert 
           \\boldsymbol{\\beta}^i - \\tau\_{ij}(\\boldsymbol{\\beta}^j) 
        \\right \\rVert_2.
$$

`MGLasso` package available on
[CRAN](https://CRAN.R-project.org/package=mglasso) is based on the
python implementation of the solver `CONESTA` available in
[pylearn-parsimony](https://github.com/neurospin/pylearn-parsimony)
library.

## Package requirements

-   Install the `reticulate` package.

``` r
install.packages('reticulate')
```

-   Check if Python engine is available on the system

``` r
reticulate::py_available()
```

-   If not available

``` r
reticulate::install_miniconda()
```

-   Install `MGLasso`

``` r
install.packages('mglasso')
```

-   Install `MGLasso` python dependencies

``` r
mglasso::install_conesta()
```

An example of use is given below.

## Illustration on a simple model

### Installation

``` r
remotes::install_github("desanou/mglasso")
library(mglasso)
```

``` r
library(mglasso)
#> Welcome to MGLasso.
#> Run install_conesta() to finalize the package python dependencies installation.
install_conesta()
#> mglasso requires the r-reticulate virtual environment. Attempting to create...
#> virtualenv: r-reticulate
#> Using virtual environment 'r-reticulate' ...
#> Installing pylearn-parsimony
#> pylearn-parsimony is installed.
```

### Simulate a block diagonal model

We simulate a 3-block diagonal model where each block contains 3
variables. The intra-block correlation level is set to 0.85 while the
correlations outside the blocks are kept to 0.

``` r
library(Matrix)
n = 50
K = 3
p = 9
rho = 0.85
blocs <- list()

for (j in 1:K) {
  bloc <- matrix(rho, nrow = p/K, ncol = p/K)
  for(i in 1:(p/K)) { bloc[i,i] <- 1 }
  blocs[[j]] <- bloc
}

mat.correlation <- Matrix::bdiag(blocs)
corrplot::corrplot(as.matrix(mat.correlation), method = "color", tl.col="black")
```

<img src="README_files/figure-gfm/unnamed-chunk-8-1.png" width="100%" />

#### Simulate gaussian data from the covariance matrix

``` r
set.seed(11)
X <- mvtnorm::rmvnorm(n, mean = rep(0,p), sigma = as.matrix(mat.correlation))
colnames(X) <- LETTERS[1:9]
```

### Run `mglasso()`

We set the sparsity level *λ*<sub>1</sub> to 0.2 and rescaled it with
the size of the sample.

``` r
X <- scale(X)    
res <- mglasso(X, lambda1 = 0.2*n, lambda2_start = 0.1, fuse_thresh = 1e-3, verbose = FALSE)
```

To launch a unique run of the objective function call the `conesta`
function.

``` r
temp <- mglasso::conesta(X, lam1 = 0.2*n, lam2 = 0.1)
```

#### Estimated clustering path

We plot the clustering path of `mglasso` method on the 2 principal
components axis of *X*. The path is drawn on the predicted *X*’s.

``` r
library(ggplot2)
library(ggrepel)
mglasso:::plot_clusterpath(as.matrix(X), res)
```

<img src="README_files/figure-gfm/unnamed-chunk-12-1.png" width="100%" />

#### Estimated adjacency matrices along the clustering path

As the the fusion penalty increases from `level9` to `level1` we observe
a progressive fusion of adjacent edges.

``` r
plot_mglasso(res)
```

<img src="README_files/figure-gfm/unnamed-chunk-13-1.png" width="100%" />

## Reference

Edmond, Sanou; Christophe, Ambroise; Geneviève, Robin; (2022): Inference
of Multiscale Gaussian Graphical Model. ArXiv. Preprint.
<https://doi.org/10.48550/arXiv.2202.05775>
