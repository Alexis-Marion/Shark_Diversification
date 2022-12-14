 # A dense time-calibrated phylogeny of sharks provides insights into the role of traits on their deep-time diversification : repository 

## Summary 

- [Summary](#Summary)
- [Overview](#Overview)
- [1 Creating trait-association data](#1-Creating-trait-association-data)
	- [1.1 Prerequisite](#11-Prerequisite)
		- [1.1.1 Discretization](#111-Discretization)
		- [1.1.2 Multiple correspondence analysis](#112-Multiple-correspondence-analysis)
	- [1.2 Hierarchal clustering](#12-Hierarchal-clustering)
		- [1.2.1 Choosing the best algorithm](#121-Choosing-the-best-algorithm)
		- [1.2.2 Post-analysis group determination](#122-Post-analysis-group-determination)
- [2 Diversification analysis](#2-Diversification-analysis)
	- [2.1 Selecting the likeliest model](#21-Selecting-the-likeliest-model)
	- [2.2 Bayesian analysis](#22-Bayesian-analysis)
	- [2.3 Ancestral state estimation](#23-Ancestral-state-estimation)
- [3 Sensitivity analyses](#3-Sensitivity-analyses)
	- [3.1 Revisiting MuSSE analysis](#31-Revisiting-MuSSE-analysis)
	- [3.2 Testing a single multi-state trait](#32-Testing-a-single-multi-state-trait)
	- [3.3 Testing multiple binary traits](#33-Testing-multiple-binary-traits)
- [Reference](#Reference)

<p align="justify"> This repository's purpose is to give a means of replicability to the article "A dense time-calibrated phylogeny of sharks provides insights into the role of traits on their deep-time diversification" but can be generalized to other similar data as the scripts are not specific. All of the presented scripts are written in R language (R Core Team, 2022). If you are planning to use any of these scripts, please cite "XXX". </p>

## Overview

<p align="justify"> To analyze the role of traits in shark diversification, we had to conduct several trait-related diversification models. Those models can be cumbersome and can account mostly for only a multi-state trait: testing several traits at once can be impossible. As such, post-phylogeny analysis for diversification purposes is divided into three steps :

**1**: Merging trait data into a single trait-association dataset

**2**: Run the actual diversification analysis

**3**: Conducting sensitivity analysis

<p align="justify"> Example data used for our article are available at "figshare.com".</p>

## 1 Creating trait-association data

`package requirement (mclust, Rtsne, ggplot2, reshape2, dplyr, dendextend, cluster, fpc, FactoMineR, factoextra )`

`used script (Multitrait_analysis.r)`

<p align="justify"> Today's models are not able to account for multiple traits and hence, testing for their effect on diversification may need a workaround. To do so and using statistical tools, we have created composite data summarizing all traits presented in the article with the first dedicated script called: "Multitrait_analysis.r".</p>

The following script and explanation are inspired by the excellent  ["Hierarchical Clustering on Categorical Data in R"](https://towardsdatascience.com/hierarchical-clustering-on-categorical-data-in-r-a27e578f2995), written by Anastasia Reusova.

### 1.1 Prerequisite

#### 1.1.1 Discretization

<p align="justify"> The first, and optional, step is to discretize continuous data. If you are using traits, you will probably handle continuous data, and as such, you need to discretize them. In this script, we use a Gaussian mixture model with the mclust function of the mclust package. This package selects automatically the most likely number of groups. </p> 

#### 1.1.2 Multiple correspondence analysis

<p align="justify"> Multiple correspondence analysis (MCA) is a multivariate data analysis designed for nominal categorical data. We used MCA here to obtain a first look on our data, and measure qualitatively the association between traits. For example, we can see here that on the first and second axis a large maximum body size, a oceanic habitat and oophageous mode of reproduction are highly associated. Hence, it is expected that a group presenting such association will be detected in further analyses. </p> 

<p align="center">
    <img src="MCA_method.png" \>
</p>

### 1.2 Hierarchal clustering

<p align="justify"> Multivariate and clustering analyses can be quite complicated, especially for mixed and discrete data. Only a few options are available and here we choose to explore one of them: hierarchal clustering using Gower's distance. Gower's distance is appropriate for this kind of data and can be used through hierarchal clustering afterward. The problem is that altough, hierarchal clustering can create groups, the user may not know which number of groups is optimal. To avoid this and be as rigorous as possible, we shall select the best distance-based clustering algorithm and determine the optimal groups through different criteria. </p>

#### 1.2.1 Choosing the best algorithm

<p align="justify"> Distance based-clustering, as the phenetic method in phylogeny, uses several different reconstruction algorithms. Here we tried to choose the best algorithm based on two different criteria: the 2-norm criterion (M??rigot et al., 2010) and the least-square criterion. For these two criteria, the lower the better. The script allows the user to calculate these metrics, and here the best method is the UPGMA </p>

#### 1.2.2 Post-analysis group determination 

<p align="justify"> Now that we possess the best algorithm for our dataset, we should find the best number of groups. We present two complementary methods to assess which number of groups is optimal considering the dataset.
The elbow method (Zambelli, 2016) computes a score for each cluster determined with the algorithm, the higher the difference between successive clusters the better. As such, the user will probably select the optimal number of clusters when he sees a break in the curve. </p>

<p align="center">
    <img src="Elbow_method.png" \>
</p>

<p align="justify"> Similarly, the silhouette method (Menardi, 2011) is a measure of how similar a data point is within a cluster compared to other clusters. Here the higher the curve, the better. Most of the time the two methods will come to similar, if not identical, results. Again, the composition of each group should be carefully examined, as they should account for biological reality. For our dataset, the best number of groups is five. </p>

<p align="center">
    <img src="Silhouette_method.png" \>
</p>

## 2 Diversification analysis

`package requirement (diversitree, qpcR, ggtree, ggplot2, ggpmisc, optional(stringr))`

`used script (Diversity_analysis_all.r)`

<p align="justify"> The models used for trait-dependant diversification are known as "SSE" and originated from the original BiSSE model (Maddison et al., 2007). These are complex models using both trait data and branch length from a calibrated tree to correlate trait and diversification. Here we used the MuSSE model which was more fitted for our analysis. The second script is known as "Diversity_analysis.r" and allows the user to conduct a diversification analysis and ancestral state estimation for a multi-state trait under MUSSE. </p>

### 2.1 Selecting the likeliest model

<p align="justify"> The first step in conducting this analysis is to select the likeliest model. To do so, we build several variations of a complete model where all the rates (i.e. speciation, extinction, and transition between states) can vary. The best model is then selected based on statistical metrics such as AICc, delta AIC, and AIC weight. In our case, with the shark dataset, the best model is the full model with no variation in extinction rates.  </p>

### 2.2 Bayesian analysis

<p align="justify"> The second step consists in running the best maximum likelihood model into a bayesian framework. To do so, we need to reference priors, as they are mandatory for bayesian analyses. We decided to make an exponential prior, and a run of 10000 generations with 10 % burn-in. To see if convergence was achieved, we visualized the likelihood plot minus the burn-in period. Since the likelihood seems to have stabilized shortly after the burn-in, we considered that this run has converged. </p>

<p align="justify"> Secondly, we plotted the net diversification rates for each pair of trait and the pairwise net diversification rate difference between each trait. The first plot allows the user to visualize easily the range of net diversification rate for each trait. This plot is very qualitative and cannot account for real statistical differences, and hence, it is needed to characterize quantitatively the difference between rates.</p>


<p align="center">
    <img src="Figure_MuSSE_git.png" \>
</p>

<p align="justify"> The second plot accounts for such differences, as they are significant if the plot does not overlap 0 (represented by a red line). Here we can see two main groups, with respectively low and high net diversification rates. </p>

<p align="center">
    <img src="Differences-MuSSE-diversification-rates-MCMC-git.png" \>
</p>

### 2.3 Ancestral state estimation

<p align="justify"> The last step of this analysis is to compute ancestral states considering the phylogeny. Using bayesian data generated in part 2.2, we estimated each category's ancestral state with the help of the asr.marginal function. Computing A.S.E for each node of the phylogeny results in a probability table. Since the table is large, we decided to plot it directly over the phylogeny. Several ways to represent probability for each node in a phylogeny are available, we chose pie charts, as they are readable. </p>

## 3 Sensitivity analyses 

<p align="justify"> SSE models are sensitive to several mathematical biases (such as rejection of the null hypothesis). We carried out two additional analyses, to minimize such biases. </p>

#### 3.1 Revisiting MuSSE analysis

`package requirement (ape, secsse, DDD, tidyverse, parallel, qgraph, optional(stringr))`

`used script (SecSSE_Full.r)`

Accounting for trait effect is subject to numerous methodological biases (Beaulieu and Donoghue, 2013). Indeed, SSE models can falsely indicate an effect of the focal trait on diversification. Models with hidden traits, such as SeCSSE (Herrera-Alsina et al., 2019) or HiSSE (Beaulieu and O'Meara, 2016) can account for hidden variables in trait-dependant diversification. Here we use SeCSSE to detect : 

1 - an effect of the trait on diversification, 

2 - the possible existence of other hidden variables in our dataset (other variables than the trait-association variable).

3 - the coexistence of focal and hidden variables in our dataset (trait-association and hidden variables). 

Following Herrera-Alsina et al. (2019) and most importantly Liedtke et al. (2022), we revisited our MuSSE original results. The present script, as well as the "SecSSE_Reproduction.r" and "SecSSE_Body-size.r" scripts are adapted from Liedtke et al. (2022). As such, if you you want to reuse these scripts, please cite the original authors beforehand. Morevoer, some scripts from Liedtke et al. (2022) are required to run the analysis (aux_scripts).

#### 3.2 Testing a single multi-state trait

`package requirement (ape, secsse, DDD, tidyverse, parallel, qgraph, optional(stringr))`

`used script (SecSSE_Reproduction.r, SecSSE_Body-size.r)`

Individual trait analyses were conducted reuse the same archetype as for the trait-association variable.

<p align="justify">  </p>

#### 3.3 Testing multiple binary traits

`package requirement (diversitree, qpcR, optional(stringr))`

`used script (Multi_State_MUSSE.r)`

<p align="justify"> As habitat is hardly discriminable, the most efficient way to account for, this trait was to subdivide it into several binary traits, with one state being the presence of a species in a certain habitat, and the other its absence. Because of its nature, such a trait could not be analyzed with the previous method. Consequently, we used the function musse.multitrait presented in the package diversitree. Using the same dataset as before and with multitrait_binary_analysis.r script, you will be able to conduct this analysis.  </p>

### Reference

Beaulieu, J. M. Donoghue, M. J. (2013). Fruit evolution and diversification in campanulid angiosperms: Campanulid fruit evolution. Evolution. 67(11): 3132-3144.

Herrera-Alsina, L. van Els, P. Etienne, R. S. (2019). Detecting the dependence of diversification on multiple traits from phylogenetic trees and trait data. Systematic Biology. 68(2): 317-328.

Liedtke, H. C. Wiens, J. J. & Gomez-Mestre, I. (2022). The evolution of reproductive modes and life cycles in amphibians. Nature Communications, 13(1): 1-15.

Maddison, W.P., Midford, P.E. & Otto, S.P. (2007) Estimating a binary character's effect on speciation and extinction. Systematic Biology, 56, 701???710

Menardi, G. (2011). Density-based Silhouette diagnostics for clustering methods. Statistics and Computing. 21(3): 295-308.

M??rigot, B. Durbec, J. P. Gaertner, J. C. (2010). On goodness-of-fit measure for dendrogram-based analyses. Ecology. 91(6): 1850-1859.

R Core Team (2022). R: A language and environment for statistical
computing. R Foundation for Statistical Computing, Vienna, Austria.
URL https://www.R-project.org/.

Zambelli, A. E. (2016). A data-driven approach to estimating the number of clusters in hierarchical clustering. F1000 Research 5: 2809.
