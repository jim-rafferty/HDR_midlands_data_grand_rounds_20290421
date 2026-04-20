# Introduction

## Title slide

## Acknowledgements slide 
- Paper available on the arxiv. Working on journal publication.
- Thanks to my collaborators on the paper and HDR-UK for funding
## Aims
- Find clusters of MLTC in EHRs held in SAIL 
	- MLTC = multiple long-term conditions
	- EHR = electronic health records
	- SAIL = Secure anonymised information linkage (databank) - holds healthcare and administrative records for individuals receiving care in Wales
	- We want to find clusters conditioned on commonly adjusted for covariates, and an outcome
- Write BPR model
	- BPR = Bayesian profile regression - clustering conditioned on covariates and outcome, as I will explain.
	- SVI = Stochastic variational inference - a fitting technique more suited to very large scale data
- Since we are writing down the model likelihood, it is good practice to do simulation studies to make sure the model is doing what we think it's doing. 
- There are broadly 3 sections to this talk
	- Description of the model
	- Description of the fitting method
	- Results
# Bayesian Profile Regression

## Bayesian profile regression
- BPR is a composite modelling framework, consisting of two models sectors. There are two ways to think about the model
- Suppose you have a dataset where a response contains groups, for example test score as a function of socioeconomic status in schools (data grouped by school)
	- In this example, ignoring the groups would lead to an overestimate of the effect of SES on test score.
- Imagine we don't have data on which schools the children attended but we do have other data (for some reason!) that can be used to find the schools (eg, where the children live in 2 dimensions)
- Clustering (left) is done using a Dirichlet Process Mixture model. Regression is done using a multilevel GLM with groups provided by the DPMM.
- Fitting is done in one go, so the response model is conditioned on the clustering model, and in some sense the clustering model is conditioned on the response model (more difficult to see because there are no parameters from the response model in the clustering model directly). 
## Aside - Dirichlet Process Mixture models
- This is a 1D example of a (Gaussian) DPMM. The data is waiting times between the eruptions of the "old faithful" geyser in the Yellowstone national park.
- Image source is a very nice practical example of the implementation of DPMM in python with some nice examples! (Geysers, sunspot cycle...).
- Blue bars = data. Black solid line = total density. Dashed black lines = component densities 
- DPMM splits a target density into an unknown number of clusters - advantage over many other clustering techniques. 
## Bayesian profile regression (again)
- What we want to do is slightly different to both of the previous examples. We will end up with clusters of disease probabilities.
- The DPMM is discrete rather than Gaussian, since the variables we are using are binary indicators of disease.
- In reality, we have potentially many more than 2 diseases, and the latent space of the DPMM has high dimensionality. (ref Tim Osborne's current work)
## References 
- Initial work by Molitor et al.
- BPR quite accessible now due to the premium package (Liverani at al.) but not suitable for us as it won't run on very large datasets.

# Stochastic Variational Inference

## Standard fitting methods
- To recap, in a frequentist framework we just calculate a single number for each parameter and make assumptions about the uncertainties. In a Bayesian framework we compute the full (posterior) distribution of each parameter so we get a measure of full parameter uncertainty. 
- Normally, finding the posterior distribution is done with Markov Chain Monte Carlo (MCMC) sampling, Gibbs and NUTS being popular choices. 
- This is computationally expensive, so much so that it makes fitting Bayesian models that are either complicated, with very large datasets or both intractable, at least in a reasonable amount of time.
- We need an alternative.
## Stochastic variational inference
- (blank slide) SVI reframes a sampling (ie a simulation) problem into an optimisation problem.
- Imagine this slide represents the space of all possible posterior distributions...
- (slide with diagram) We assume the posterior distribution is in the set {Q} (which in general will not be strictly true). Common choices here include all parameters being drawn from independent Normals (mean-field approximation), or the whole posterior being represented by a multivariate normal (full rank approximation).
- The optimisation process:
	- Pick a starting set of parameters (q_init).
	- Compute the loss (a common choice here is the negative expected lower bound of the Kullback Leibler divergence)
	- take a gradient descent step
	- repeat until we reach q_opt
## Advantages and Disadvantages
- Convergence
	- NUTS is an approximation, but approaches the true parameter values in the limit of infinite samples. The variational distribution is specifically not the true posterior in general
- Diagnostics
	- More available for NUTS (MCMC in general), eg r-hat, autocorrelation etc. SVI allows you to check the loss is no longer falling
	- Computational expense 
		- SVI is a lot faster than MCMC

# Results

## Results - simulation study
- In the simulation study we computed bias and coverage. These are coverage plots (the bias plots were less interesting!)
- Left - coverage of the mixture model parameters. SVI is pretty close to NUTS
- Right - coverage of the response model parameters. SVI is quite a bit lower than NUTS.
- This indicates that the posterior distribution for the mixture model parameters is pretty close to a transformed multivariate Normal, while the response model gradients are not Normally distributed. 
## Results - SAIL analysis (1)
- This is a general population (not a disease specific one) so:
	- Total population nearly 1.3 million
	- Mean age ~42.5
- The heatmap shows 33 clusters (x-axis) and disease probabilities (y-axis). Diseases defined according to the Elixhauser comorbidity index. Top two rows are outcome (mortality) probability for females and males. Blue colour indicates close to 0 probability, while yellow colour indicates close to 1. 
## Results - SAIL analysis (2)
- Highlighting all the cancer clusters:
	- Orange highlight - Cancer without metastatic cancer (1 cluster)
	- Red and Pink highlights - Cancer + metastatic cancer (7 clusters)
- Pink highlight - 2 clusters, both have relatively low probability of other diseases but very different outcome probabilities. It's possible that the right cluster is people with late diagnosed end stage disease (eg pancreas) and the left pink cluster represents less aggressive metastatic cancer with more treatment options available (eg prostate)

# Summary

## Summary & Future work
- Bayesian Profile Regression allows more granular clustering interpretations conditioned on covariates and an outcome than a 2 stage modelling approach.
- SVI provides unbiased parameter estimates at population scale. Work ongoing to improve uncertainty quantification.
- Other GLM responses and time to event response models (all supported by premium)
- Mixture models that represent different data types (eg Gaussian etc.)
## Thank you
- Slides and notes available on github