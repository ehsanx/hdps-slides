---
title: "Causal inference in analyzing administrative healthcare data:" 
subtitle: "Can we integrate machine learning approaches within this framework?"
author: | 
  | Ehsan Karim
  | http://ehsank.com/
date: "Feb 23, 2021; CHSPR"
output: 
  beamer_presentation: 
    keep_md: yes
    highlight: tango
    theme: "AnnArbor"
    colortheme: "crane"
    fonttheme: "structurebold"
  slidy_presentation: 
    widescreen: yes
    smaller: yes
    keep_md: yes
    highlight: tango
    css: slides.css
    fig_caption: true
    mathjax: local
    self_contained: false
    footer: "Copyright (c) 2021, Ehsan Karim"
  ioslides_presentation:
    widescreen: yes
    smaller: yes
    keep_md: yes
    highlight: tango
    css: slides.css
classoption: "aspectratio=169"
bibliography: ref.bib
always_allow_html: yes
header-includes:
- \usepackage{graphicx,longtable,booktabs,amsmath,soul}
- \usepackage{float}
- \floatplacement{figure}{H}
- \floatplacement{table}{H}
- \usepackage{xcolor,lipsum,caption,natbib}
- \def\bibfont{\small}
---




## Outline

1. Notations and Questions 
    - Regression vs. Propensity score (PS)
2. Health care databases
3. High-dimensional propensity score (hdPS)
    - The original mechanism
    - \textcolor{blue}{Machine learning-based hdPS}
    - \textcolor{blue}{AI-based hdPS}
    - \textcolor{blue}{Future directions}
4. Examples in Canadian data

## Reference Preview

Will be primarily discussing @schneeweiss2018automated. Will also briefly mention @karim2018can,@weberpals2021deep, @zivich2020machine.

- Schneeweiss (2018) Clinical Epidemiology
- Karim et al. (2018) Epidemiology
- Weberpals et al. (2021) Epidemiology
- Zivich and  Breskin (2021) Epidemiology

## Notations and Motivating Example

- $Y$ = Outcome
    - Airway disease (among BC immigrants)
- $A$ = Primary exposure
    - Respiratory tuberculosis
- $C = (C_1, \ldots, C_7)$ are covariates measured at baseline
    - age, sex, income, education, comorbidity score, TB incidence in birth country, year of residency in BC

\begin{center}
\includegraphics[width=0.5\linewidth]{paper.png}
\end{center}

## Questions

Inferential goals

1. Prediction, developing risk scores (predict $Y$)
2. Identifying important predictors (identify $C_1$, $C_2$ that are important to predict $Y$)
3. Descriptive, exploratory (is $A$ and $Y$ associated?)
4. Evaluating a predictor of primary interest (does $A$ cause $Y$?)

\begin{center}
\includegraphics[width=0.7\linewidth]{p1.png} 
\end{center}

\begin{equation}
\begin{aligned}
E(Y|A,C) &= \textcolor{blue}{\psi} A \text{, after controlling for some function of C} \nonumber
\end{aligned}
\end{equation}

## Regression vs. Propensity score

- For RCT, adjustment may not be essential, as design takes care of bias sources.
- For RWE studies, more caution necessary. Adjustment of $C$ is important (\textcolor{blue}{usually large $\#$s}). 

\begin{center}
\begin{tabular}{ c c c }
 \toprule
 Model / Method & Propensity score & Regression analysis \\ 
 \midrule
 Exposure Modelling & PS $ = P(A=1|C) = f(C)$ & No design stage analysis \\  
 & \includegraphics[width=0.25\linewidth]{overlap.png}  &  \includegraphics[width=0.25\linewidth]{cross.png} \\
 Outcome Modelling & $E(Y|A,C) = g(A,C)$ & $E(Y|A,C) = g(A,C)$ \\    
 & in PS matched or weighted data & in full data \\    
 \bottomrule
\end{tabular}
\end{center}

## Health care databases

Why use non-randomized data at all?

- clinical health records
    - medical facility
    - hospital
    - clinic and practice.
- administrative data 
    - PharmaNet
    - Medical Services Plan
- clinical registries
    - TB registry
    - MS registry

## Health care databases: Advantages

- larger sample size
- diverse population
- longitudinal records over many years
- detailed 
    - health encounters, 
    - comorbidity history,
    - drug exposure history
- possibility to link other databases
    - Immigration
    - Vital Statistics

## Health care databases: Study implementation

@schneeweiss2018automated: freeze a data cut from dynamic data stream, encounters collected at covariate assessment period and follow-up, apply rules/algorithms to define variables.

\begin{center}
\includegraphics[width=.5\linewidth]{implementation.png}
\end{center}

## Health care databases: Limitations

- Not speciﬁcally designed for answering a particular research question. 
- Data sparsity: 
    - no defined or routine interview dates
    - data collection relies on visits and encounters
- Investigators have no control over which factors were measured during the data collection stage.
    - smoking in TB
    - MRI data in MS

## Health care databases: Limitations

\begin{center}
\includegraphics[width=0.65\linewidth]{dag.png}
\end{center}

## Methods: Expectation vs. Reality (Part I: unmeasured confounder)

### Exposure Model

\begin{equation}
\begin{aligned}
P(A=1|C) &= f(C) \\
        &= \frac{1}{1+exp[\alpha_0 + \alpha_1 C_1 + \alpha_2 C_2 + \alpha_3 C_3 + \alpha_4 C_4 + \alpha_5 C_5 + \alpha_6 C_6 + \alpha_7 \textcolor{blue}{C_7} ]}\nonumber
\end{aligned}
\end{equation}

### Outcome Model

\begin{equation}
\begin{aligned}
E(Y|A,C) &= g(A,C)\\
&= \beta_0 + \psi A + \beta_1 C_1 + \beta_2 C_2 + \beta_3 C_3 + \beta_4 C_4 + \beta_5 C_5 + \beta_6 C_6 + \beta_7 \textcolor{blue}{C_7} \nonumber
\end{aligned}
\end{equation}

### Assumption

\begin{equation} \label{eq:out2}
\begin{aligned}
Y | A,C \sim N[E(Y|A,C), \sigma^2] \nonumber
\end{aligned}
\end{equation}


## Methods: Expectation vs. Reality (Part 2: model misspecification)

### Exposure Model

\begin{equation} 
\begin{aligned}
P(A=1|C) &= f(C) \\
        &= \frac{1}{1+exp[\alpha_0 + \alpha_1 \textcolor{blue}{(C_2 \times C_3)} + \alpha_2 \textcolor{blue}{(C_4^2 \times \frac{\exp_{C_5}}{5 \times C_7})} + \alpha_3 C_6 ]} \nonumber
\end{aligned}
\end{equation}

### Outcome Model

\begin{equation} 
\begin{aligned}
E(Y|A,C) &= g(A,C)\\
&= \beta_0 + \psi A + \beta_1 \textcolor{blue}{C_1^2} + \beta_2 \textcolor{blue}{(C_2 \times C_3 \times C_7)} + \beta_3 \textcolor{blue}{\frac{\exp{C_4}}{C_5 \times 2}}\nonumber
\end{aligned}
\end{equation}

### Assumption

\begin{equation} 
\begin{aligned}
Y | A,C \sim N[E(Y|A,C), \sigma^2]\nonumber
\end{aligned}
\end{equation}


## hdPS


```
## Warning: package 'scholar' was built under R version 4.1.3
```

![](slideCHSPR_files/figure-beamer/cite-1.pdf)<!-- --> 


## hdPS proxy collection

@schneeweiss2018automated:

\begin{center}
\includegraphics[width=0.5\linewidth]{code.png}
\end{center}

They were the first to propose adjusting for something that **may not be interpretable** directly with the context of the research question. Logic was two fold:

1. variables from same subject should be **correlated** = has relevant information
2. adjust items that are **predictive of outcome** (as established in PS literature)


## hdPS proxy collection from same subjects (1)

Collection of proxy data for the unmeasured + mis-measured variables [@karim2018can]

\begin{center}
\includegraphics[width=0.45\linewidth]{proxy1.png}
\end{center}

## hdPS proxy collection from same subjects (1)

Collection of proxy data for the unmeasured + mis-measured variables [@karim2018can]

\begin{center}
\includegraphics[width=0.45\linewidth]{proxy2.png}
\end{center}

## hdPS proxy collection from same subjects (1)

Collection of proxy data for the unmeasured + mis-measured variables [@karim2018can]

\begin{center}
\includegraphics[width=0.45\linewidth]{proxy3.png}
\end{center}

## hdPS proxy collection from same subjects (1)

Collection of proxy data for the unmeasured + mis-measured variables [@karim2018can]: but restricted to 2,400 empirical covariates \textcolor{blue}{EC} (based on high prevalence)

\begin{center}
\includegraphics[width=0.45\linewidth]{proxy22.png}
\end{center}

## hdPS proxy collection from same subjects (1)

List of additional proxy variables:

\begin{center}
\begin{tabular}{ c c c c }
 \toprule
Dimension 1 & Dimension 2 & Dimension 3 & Dimension 4\\
Practice  & Diagnostic & Procedure & Medication\\
 \midrule
EC-dim1-1-once & EC-dim2-1-once & EC-dim3-1-once & EC-dim4-1-once\\
EC-dim1-1-sporadic & EC-dim2-1-sporadic & EC-dim3-1-sporadic & EC-dim4-1-sporadic\\
EC-dim1-1-frequent & EC-dim2-1-frequent & EC-dim3-1-frequent & EC-dim4-1-frequent\\
$\ldots$ &  $\ldots$ & $\ldots$ \\
EC-dim1-686-frequent & EC-dim2-328-frequent & EC-dim3-76-frequent & EC-dim4-284-frequent\\
 \bottomrule
\end{tabular}
\end{center}

4 dimension $\times$ 3 intensity $\times$ 200 most prevalent codes = 2,400 \textcolor{blue}{EC}s

## hdPS mechanism: find EC as h(outcome, exposure prevalence) (2)

Assumption: 

- $p_{u=1,a=1}$ = prevalence of unmeasured confounder among treated ($A=1$)
- $p_{u=1,a=0}$ = prevalence of unmeasured confounder among untreated ($A=0$)
- $p_{u=1,y=1}$ = prevalence of unmeasured confounder among dead ($Y=1$)
- $p_{u=1,y=0}$ = prevalence of unmeasured confounder among alive ($Y=0$)

Bross (1966) formula says, the amount of bias due to $u$ is 

\begin{equation} 
\begin{aligned}
\text{Bias}_M =  \frac{ p_{u=1,a=1} \times \Big(\frac{p_{u=1,y=1}}{p_{u=1,y=0}} -1 \Big) +1  } { p_{u=1,a=0} \times  \Big(\frac{p_{u=1,y=1}}{p_{u=1,y=0}} -1 \Big) +1}   \nonumber
\end{aligned}
\end{equation}

## hdPS mechanism: find EC as h(outcome, exposure prevalence) (2)

\st{Assumption} Calculate: 

- $p_{\textcolor{blue}{EC}=1,a=1}$ = prevalence of \st{unmeasured confounder} \textcolor{blue}{EC} among treated ($A=1$)
- $p_{\textcolor{blue}{EC}=1,a=0}$ = prevalence of \st{unmeasured confounder} \textcolor{blue}{EC} among untreated ($A=0$)
- $p_{\textcolor{blue}{EC}=1,y=1}$ = prevalence of \st{unmeasured confounder} \textcolor{blue}{EC} among dead ($Y=1$)
- $p_{\textcolor{blue}{EC}=1,y=0}$ = prevalence of \st{unmeasured confounder} \textcolor{blue}{EC} among alive ($Y=0$)

Bross (1966) formula says, the amount of bias due to \textcolor{blue}{EC} is 

\begin{equation} 
\begin{aligned}
\text{Bias}_M =  \frac{ p_{\textcolor{blue}{EC}=1,a=1} \times \Big(\frac{p_{\textcolor{blue}{EC}=1,y=1}}{p_{\textcolor{blue}{EC}=1,y=0}} -1 \Big) +1  } { p_{\textcolor{blue}{EC}=1,a=0} \times  \Big(\frac{p_{\textcolor{blue}{EC}=1,y=1}}{p_{\textcolor{blue}{EC}=1,y=0}} -1 \Big) +1}   \nonumber
\end{aligned}
\end{equation}

## hdPS: select hdPS variables from ECs

1. Rank (descending) each \textcolor{blue}{EC} by the magnitude of log-bias: $|\log{\text{Bias}_M}|$

\begin{center}
\begin{tabular}{ c c c }
 \toprule
Rank by bias & $|\log{\text{Bias}_M}|$ & \textcolor{blue}{EC} \\ 
 \midrule
1 & 0.42 & EC-dim1-21-once\\
2 & 0.32 & EC-dim2-95-once\\
3 & 0.25 & EC-dim4-289-once\\
$\ldots$ &  $\ldots$ & $\ldots$ \\
2,400 & 0.01 & EC-dim4-64-frequent\\
 \bottomrule
\end{tabular}
\end{center}

2. Take top 500 of these \textcolor{blue}{EC}s. These are hdPS variables.

## hdPS: estimate treatment effect

### Exposure Model (investigator-specified covariates)

\begin{equation}
\begin{aligned}
P(A=1|C) &= f(C) \\
        &= \frac{1}{1+exp[\alpha_0 + \alpha_1 C_1 + \alpha_2 C_2 + \alpha_3 C_3 + \alpha_4 C_4 + \alpha_5 C_5 + \alpha_6 C_6 + \alpha_7 \textcolor{blue}{C_7} ]}\nonumber
\end{aligned}
\end{equation}

### Outcome Model (in the PS matched or weighted data)

\begin{equation}
\begin{aligned}
E(Y|A,C) &= g(A,C)\\
&= \beta_0 + \psi A \nonumber
\end{aligned}
\end{equation}

## hdPS: estimate treatment effect

### Exposure Model (investigator-specified covariates + hdPS variables [EC])

\begin{equation}
\begin{aligned}
P(A=1|C) &= f(C) \\
        &= \frac{1}{1+exp[\alpha_0 + \alpha_1 C_1 + \alpha_2 C_2 + \alpha_3 C_3 + \alpha_4 C_4 + \alpha_5 C_5 + \alpha_6 C_6 +  \sum_{i=1}^{500} \alpha_i^{'} \textcolor{blue}{\text{EC}_i} ]}\nonumber
\end{aligned}
\end{equation}

### Outcome Model (in the PS matched or weighted data)

\begin{equation}
\begin{aligned}
E(Y|A,C) &= g(A,C)\\
&= \beta_0 + \psi A \nonumber
\end{aligned}
\end{equation}

## hdPS: estimate treatment effect

Performance of hdPS. @schneeweiss2018automated:

\begin{center}
\includegraphics[width=0.6\linewidth]{hdpsonly.png}
\end{center}

## ML-hdPS: deal with collinearity

### Kitchen SSink Exposure Model (A ~ C + EC)

\begin{equation}
\begin{aligned}
P(A=1|C, EC) &= \frac{1}{1+exp[\alpha_0 + \alpha_1 C_{\text{important}} + \alpha_2 C_{\text{potential confounder}} +  \sum_{i=1}^{2,400} \alpha_i^{'} \textcolor{blue}{\text{EC}_i} ]}\nonumber
\end{aligned}
\end{equation}

### Outcome Model for EC selection (Y ~ C + top-500 ECs)

\begin{equation}
\begin{aligned}
f(Y|C, EC) &= \alpha_0 + \alpha_1 C_{\text{important}} + \alpha_2 C_{\text{potential confounder}} +  \sum_{i=1}^{500} \alpha_i^{'} \textcolor{blue}{\text{EC}_i} \nonumber
\end{aligned}
\end{equation}

## ML-hdPS: deal with collinearity

### hdPS Exposure Model (A ~ C + top-ranked EC)

\begin{equation}
\begin{aligned}
P(A=1|C, EC) &= \frac{1}{1+exp[\alpha_0 + \alpha_1 C_{\text{important}} + \alpha_2 C_{\text{potential confounder}} +  \sum_{i=1}^{\text{top 500}} \alpha_i^{'} \textcolor{blue}{\text{EC}_i} ]}\nonumber
\end{aligned}
\end{equation}

### Outcome Model for EC selection (Y ~ C + ECs)

\begin{equation}
\begin{aligned}
f(Y|C, EC) &= {\alpha_0 + \alpha_1 C_{\text{important}} + \alpha_2 C_{\text{potential confounder}} +  \sum_{i=1}^{2,400} \alpha_i^{'} \textcolor{blue}{\text{EC}_i} }\nonumber
\end{aligned}
\end{equation}

## ML-hdPS: deal with collinearity

### Exposure Model (investigator-specified covariates + hdPS variables [EC])

\begin{equation}
\begin{aligned}
P(A=1|C) &= \frac{1}{1+exp[\alpha_0 + \alpha_1 C_1 + \alpha_2 C_2 + \alpha_3 C_3 + \alpha_4 C_4 + \alpha_5 C_5 + \alpha_6 C_6 +  \sum_{i=1}^{500} \alpha_i^{'} \textcolor{blue}{\text{EC}_i} ]}\nonumber
\end{aligned}
\end{equation}

### ML extensions: exposure model vs. outcome model

- \textcolor{blue}{\text{EC}}s selected separately 
    - are usually highly correlated, and 
    - has inflated variance.
- @karim2018can used elasic net and LASSO to reduce the number of selected hdPS variables (\textcolor{blue}{\text{EC}} variables). 
    - found that the hybrid method (elastic net of hdPS) performs better than ML or hdPS.
    - quality of proxy information matters.

## ML-hdPS: deal with model misspecification

### Exposure Model

\begin{equation} 
\begin{aligned}
P(A=1|C) &= \frac{1}{1+exp[\alpha_0 + \alpha_1 \textcolor{blue}{(C_2 \times C_3)} + \alpha_2 \textcolor{blue}{(C_4^2 \times \frac{\exp_{C_5}}{5 \times C_7})} + \alpha_3 C_6 ]} \nonumber
\end{aligned}
\end{equation}

### Outcome Model

\begin{equation} 
\begin{aligned}
E(Y|A,C) &= \beta_0 + \psi A + \beta_1 \textcolor{blue}{C_1^2} + \beta_2 \textcolor{blue}{(C_2 \times C_3 \times C_7)} + \beta_3 \textcolor{blue}{\frac{\exp{C_4}}{C_5 \times 2}}\nonumber
\end{aligned}
\end{equation}

### More ML extensions
- Tree-based models: automatically detect function form
    - @karim2018can used random forest, and chose top important \textcolor{blue}{\text{EC}}s
- Superlearner (ensemble learner) with tree-based + parametric models
- Double robust methods with Superlearner


## AI-hdPS: towads dimension reduction

- Principal component (\textcolor{blue}{PC}) analysis 
    - compute linear transformation of all covariates to \textcolor{blue}{PC}s, and top few \textcolor{blue}{PC}s are selected to reduce dimension.
    - extract components of each variable resoponsible for most variance
    - incapable of learning non-linear feature representations
    
## AI-hdPS: towads dimension reduction    
    
- @weberpals2021deep proposed to use \textcolor{blue}{autoencoders} (3, 5, 7 layers) to reduce \textcolor{blue}{EC} dimensions:

\begin{center}
\includegraphics[width=0.3\linewidth]{ae.png}
\end{center}

## AI-hdPS: towads dimension reduction

% Bias 

\begin{center}
\includegraphics[width=0.5\linewidth]{Bias.png}
\end{center}

## AI-hdPS: towads dimension reduction

rMSE

\begin{center}
\includegraphics[width=0.5\linewidth]{rMSE.png}
\end{center}

## AI-hdPS: towads dimension reduction

Coverage: slower rate of convergence for non-parametric methods

\begin{center}
\includegraphics[width=0.5\linewidth]{Coverage.png}
\end{center}

## Future directions

@zivich2020machine: Cross-fitting + together with double-robust approaches

\begin{center}
\includegraphics[width=0.6\linewidth]{cf.png} 
\end{center}

## A purist view

- Bias-based hdPS is not really PS! 
    - design stage vs. analysis stage
    - hdPS uses corr(EC, outcome).
- Motivation of PS and hdPS are different to begin with
    - PS requires no unmeasured confounding
- I prefer to use hdPS as a sensitivity analysis.

## Examples of hdPS (1)

\begin{center}
\includegraphics[width=0.3\linewidth]{dag.png}
\end{center}

Airway example: analysis approaches that were compared: hdPS as \textcolor{blue}{sensitivity analyses}:

- PS decile-adjusted (investigator-specified covariates)
- hdPS decile-adjusted (investigator-specified + \textcolor{blue}{\text{EC}})
- LASSO-hdPS decile-adjusted (investigator-specified + LASSO-reduce \textcolor{blue}{\text{EC}})
- Others: E-value, other proxy variable adjustment

Conclusions were similar.

## Examples of hdPS (2)

A 2017 JAMA paper
\begin{center}
\includegraphics[width=0.5\linewidth]{jama.png} 
\end{center}

\begin{center}
\begin{tabular}{ c c c }
 \toprule
Method & HR & CI $95\%$ \\ 
 \midrule
Unadjusted & 2.16 & 1.64-2.86\\
Multivariable adjusted & 1.59 & 1.17-2.17\\
IPTW hdPS & \textcolor{blue}{1.61} & \textcolor{blue}{0.997-2.59}\\
1-1 hdPS matching &  1.64 & 1.07-2.53\\
Pre-pregnancy data & 1.85 & 1.37-2.51\\
 \bottomrule
\end{tabular}
\end{center}

Conclusion: **not associated**!

## Reference

<div id="refs"></div>

## Thank you!

\begin{center}
\includegraphics[width=1\linewidth]{site.png} 
\end{center}
