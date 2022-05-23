SeqSGPV Package
================
Jonathan Chipman
5/20/2022

## Background

The SeqSGPV package allows the user design a study with sequential
monitoring of the second generation p-value (SGPV) on clinically
meaningful endpoints. It supports the paper [Sequential monitoring using
the Second Generation P-Value with Type I error controlled by monitoring
frequency](https://arxiv.org/pdf/2204.10678.pdf) which advances:

1.  SeqSGPV as an evidence-based sequential monitoring strategy of
    inferential intervals
2.  Pre-Specified Regions Indicating Scientific Merit (PRISM) as a set
    of clinically meaningful hypotheses
3.  Monitoring frequency strategies for controlling Type I error,
    including:  
    A. Wait time until monitoring  
    B. Assessment frequency  
    C. Affirming a stopping rule  
    D. Maximum sample size  

**Why SGPV**: The SGPV is an evidence-based metric that measures the
overlap between an inferential interval and clinically meaningful
hypotheses. Described as ‘method agnostic’[1], the SGPV may be
calculated for any inferential interval (ex: bayesian, frequentist,
likelihood, etc.). For an interval hypothesis,
![H](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;H "H"),
the SGPV is denoted as
p![\_H](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;_H "_H").

**Why PRISM**: Monitoring until establishing statistical significance
does not provide information on whether an effect is clinically
meaningful. Alternatively, a study may monitoring hypotheses that
reflect clinical relevance. Two existing hypotheses of clinical merit
include the Region of Practical Equivalence (ROPE)[2] and the Region of
Equivalence (ROE)[3]. PRISM hypotheses reflect aspects of ROPE and ROE
hypotheses.

The ROPE is an indifference zone around the null hypothesis and a
monitoring strategy has been proposed to continue until an interval lies
entirely within or excludes the ROPE. This monitoring strategy reduces
the risk of a Type I error yet can allow for indefinite monitoring for
effects that lie on ROPE boundaries.

The Region of Equivalence (ROE) is more flexibly defined as the region
of indifference for which the benefit of a treatment is questionable.
Often the ROE is monitored as a set of overlapping hypotheses – until
ruling meaningful effects or until ruling out null effects. The ROE may
encompass the null hypothesis (similar to ROPE), may have the null
hypothesis as a boundary (null-bound ROE), or may be set aside from the
null hypothesis (PRISM).

Compared to ROPE monitoring, PRISM monitoring also reduces the risk of
type I error yet resolves the issue of indefinite monitoring at ROPE
boundaries.

Compared to null-bound ROE monitoring, PRISM monitoring reduces the risk
of Type I error for the same monitoring frequency and allows for earlier
monitoring and yields smaller average sample size to achieve the same
Type I error.

In a 2-sided study, the PRISM includes a ROPE and Region of Meaningful
effects (ROME). In a 1-sided study, the PRISM includes a Region of Worse
or Practically Equivalent effects (ROWPE) and ROME.

![Pre-Specified Regions Indicating Scientific Merit (PRISM) for one- and
two-sided hypotheses. The PRISM always includes an indifference zone
that surrounds the point null hypothesis
(i.e. ROPE/ROWPE).](README_files/figure-gfm/AMwithSGPV_Figsv07.jpg)

**Why change monitoring frequency**: Controlling the design-based Type I
error is generally considered an important metric for reducing the risk
of false discoveries. A common Frequentist and Bayesian strategy is to
tune the level of evidence required to draw inference on a hypothesis.
Changing the frequency of looks and/or the quality of looks (such as an
affirmation rule) controls Type I error without compromising evidence
level.

**Synergy between 1-sided PRISM and monitoring frequency**: On their
own, both the PRISM and monitoring frequency help reduce the risk of
Type I error. When used together, the 1-sided PRISM and monitoring
frequency can dramatically reduce the average sample size to achieve a
Type I error. When outcomes are delayed, the risk of reversing a
decision on the null hypothesis decreases when monitoring a 1-sided
PRISM more so than under a 1-sided null-bound ROE. Additional
strategies, such as posterior predictive probabilities may be considered
to further inform decisions under delayed outcomes.

**Comment on monitoring confidnece intervals**: When using confidence
intervals, the investigator should determine how to address issues of
bias and coverage which is common to sequential monitoring. This aspect
is beyond the scope of the paper [Sequential monitoring using the Second
Generation P-Value with Type I error controlled by monitoring
frequency](https://arxiv.org/pdf/2204.10678.pdf).

## Package overview

Outcomes may be generated from any r\[dist\] distribution, a
user-supplied data generation function, or pre-existing data. Study
designs of bernoulli and normally distributed outcomes have been more
extensively evaluated and extra care should be provided when designing a
study with outcomes of other distribution families.

The user provides a function for obtaining interval of interest. Some
functions have been built for common interval estimations: binomial
credible and confidence intervals using binom::binom.confint, wald
confidence intervals using lm function for normal outcomes, and wald
confidence intervals using glm function with binomial link for bernoulli
outcomes.

Depending on computing environment, simulations may be time consuming to
obtain many (10s of thousands) replicates and more so for bernoulli
outcomes. The user may consider starting with a small number of
replicates (200 - 1000) to get a sense of design operating
characteristics. Sample size estimates of a single look trial may also
inform design parameters.

## Required install of sgpv package

SeqSGPV requires the sgpv package. To install, call:
`devtools::install_github("weltybiostat/sgpv")`.

# Study design examples

Study designs and interpretations of a single trial are provided below
for 1-2 arm trials with bernoulli or normally distributed outcomes.

## Example 1a: Phase II, single arm bernoulli outcomes

A prostate cancer trial is designed to assess the margin status (an
immediate outcome) following a novel surgery technique.

H0: prob success
![\\le](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;%5Cle "\le")
0.2  
H1: prob success \> 0.2

The investigators says the study can afford up to 40 participants and
wants to know the design-based average sample size, Type I error, and
Power across a range of treatment effects. The investigator wishes to
use repeated 95% credible intervals with a Beta(0.005, 0.005) prior on
the success probability.

For this early phase study, the clinician deems success probabilities
less than 0.225 as essentially equivalent or worse than the null
hypothesis (i.e. ROWPE). The minimal clinically meaningful effect
(i.e. ROME boundary) is 0.4.

As a secondary outcome, the investigator is interested in 3m quality of
life. Before measuring a single patient’s outcome, an additional 5-10
participants may be placed on the trial. This outcome will be assessed
in Example 1b yet simulated here.

The investigator wants a Type I error
![\\le](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;%5Cle "\le")
0.05 and is willing to monitor the trial at every outcome.

``` r
# Example 1
# 1 arm phase II trial with bernoulli outcomes
# H0: prob success <= 0.2
# H1: prob success > 0.2
# PRISM: deltaG1 = 0.225, deltaG2 = 0.4
# maximum sample size = 35 - 40
# possible number of lag/delayed outcomes: 0, 5, 10
nreps <- 20000
system.time(PRISM <-  SeqSGPV(nreps            = nreps,
                              dataGeneration   = rbinom, dataGenArgs = list(n=40, size=1, prob = .2),
                              effectGeneration = 0, effectGenArgs=NULL,  effectScale  = "identity",
                              allocation       = 1,
                              effectPN         = 0.2,
                              null             = "less",
                              PRISM            = list(deltaL2 = NA, deltaL1 = NA, 
                                                      deltaG1 = .225, deltaG2 = .4),
                              modelFit         = binomCI,
                              modelFitArgs     = list(conf.level=.95, 
                                                      prior.shape1=0.005, prior.shape2=0.005,
                                                      methods="bayes", type="central"),
                              wait             = 5:10,
                              steps            = 1:3,
                              affirm           = 0:1,
                              lag              = c(0,5,10),
                              N                = 35:40,
                              printProgress    = FALSE))
```

        user   system  elapsed 
    2221.122   21.156  615.557 

Type I error under different monitoring frequencies. Increasing the
number of observations between assessments (steps) and requiring a
stopping rule to be affirmed decreases Type I error.

``` r
par(mfrow=c(2,2))
plot(PRISM,stat = "rejH0", affirm=0, steps=1,lag=0,ylim=c(0.02, 0.10))
abline(h=.05)
plot(PRISM,stat = "rejH0", affirm=0, steps=2,lag=0,ylim=c(0.02, 0.10))
abline(h=.05)
plot(PRISM,stat = "n",     affirm=0, steps=1,lag=0,ylim=c(14,23))
plot(PRISM,stat = "n",     affirm=0, steps=2,lag=0,ylim=c(14,23))
```

<img src="README_files/figure-gfm/unnamed-chunk-3-1.png" style="display: block; margin: auto;" />

For comparison, consider monitoring the null-bound ROE – determining the
trial successful when there is evidence for an effect \> 0.2 and futile
when there is evidence the effect is \< 0.4.

``` r
# Change to monitoring null-bound ROE
inputs <- PRISM$inputs
inputs$PRISM$deltaG1 <- 0.20
system.time(PRISM_NullROE <-  do.call(SeqSGPV, inputs))
```

        user   system  elapsed 
    2215.540   20.247  600.302 

``` r
par(mfrow=c(2,2))
# Compare different designs
plot(PRISM,        stat = "rejH0", affirm=0, steps=1,lag=0,ylim=c(0.02, 0.10))
title(sub="PRISM", adj=0)
abline(h=.05)
plot(PRISM,        stat = "rejH0", affirm=1, steps=3,lag=0,ylim=c(0.02, 0.10))
title(sub="PRISM", adj=0)
abline(h=.05)
plot(PRISM_NullROE,stat = "rejH0", affirm=0, steps=1,lag=0,ylim=c(0.02, 0.10))
title(sub="Null-bound ROE", adj=0)
abline(h=.05)
plot(PRISM_NullROE,stat = "rejH0", affirm=1, steps=3,lag=0,ylim=c(0.02, 0.10))
title(sub="Null-bound ROE", adj=0)
abline(h=.05)
```

<img src="README_files/figure-gfm/unnamed-chunk-5-1.png" style="display: block; margin: auto;" />

Comparing Type I error for the same maximum sample size and nearly
equivalent Type I error.

``` r
summary(PRISM,effect = 0,steps=1,affirm=0,wait=9,N=39,lag=0)
```


    Given: theta = 0.2, W = 9 S = 1, A = 0 and N = 39, with 0 lag (delayed) outcomes
    H0   : theta is less than or equal to 0.2
      Average sample size              = 16.7314
      P( reject H0 )                   = 0.052
      P( conclude not ROPE effect )    = 0.0503
      P( conclude not ROME effect )    = 0.8759
      P( conclude PRISM inconclusive ) = 0.0738
      Coverage                         = 0.8155
      Bias                             = -0.0343

``` r
summary(PRISM_NullROE,effect = 0,steps=3,affirm=1,wait=6,N=39,lag=0)
```


    Given: theta = 0.2, W = 6 S = 3, A = 1 and N = 39, with 0 lag (delayed) outcomes
    H0   : theta is less than or equal to 0.2
      Average sample size              = 18.9927
      P( reject H0 )                   = 0.0517
      P( conclude not ROPE effect )    = 0.0517
      P( conclude not ROME effect )    = 0.8409
      P( conclude PRISM inconclusive ) = 0.1074
      Coverage                         = 0.6822
      Bias                             = -0.0508

Average sample size between PRISM and null-bound ROE designs.

``` r
par(mfrow=c(1,2))
plot(PRISM,        stat = "n", affirm=0, steps=1,lag=0,ylim=c(14,23))
title(sub="PRISM", adj=0)
plot(PRISM_NullROE,stat = "n", affirm=1, steps=3,lag=0,ylim=c(14,23))
title(sub="Null-bound ROE", adj=0)
```

<img src="README_files/figure-gfm/unnamed-chunk-7-1.png" style="display: block; margin: auto;" />

Operating characteristics of PRISM design across a range of effects.

``` r
# Obtain design under range of effects
se <- seq(-0.05, 0.3, by = 0.025)
system.time(PRISMse <- fixedDesignEffects(PRISM, shift = se))
```

    [1] "effect: -0.05"
    [1] "effect: -0.025"
    [1] "effect: 0"
    [1] "effect: 0.025"
    [1] "effect: 0.05"
    [1] "effect: 0.075"
    [1] "effect: 0.1"
    [1] "effect: 0.125"
    [1] "effect: 0.15"
    [1] "effect: 0.175"
    [1] "effect: 0.2"
    [1] "effect: 0.225"
    [1] "effect: 0.25"
    [1] "effect: 0.275"
    [1] "effect: 0.3"

         user    system   elapsed 
    30968.226   390.896  8579.123 

``` r
plot(PRISMse, stat = "rejH0", steps = 1, affirm = 0, N = 39, lag=0)
```

<img src="README_files/figure-gfm/unnamed-chunk-9-1.png" style="display: block; margin: auto;" />

``` r
summary(PRISMse, effect = 0.2, wait = 9, steps = 1, affirm = 0, N = 39, lag = 0)
```


    Given: theta = 0.4, W = 9 S = 1, A = 0 and N = 39, with 0 lag (delayed) outcomes
    H0   : theta is less than or equal to 0.2
      Average sample size              = 18.6765
      P( reject H0 )                   = 0.764
      P( conclude not ROPE effect )    = 0.7415
      P( conclude not ROME effect )    = 0.1442
      P( conclude PRISM inconclusive ) = 0.1144
      Coverage                         = 0.8316
      Bias                             = 0.0345

As an atlernative design, the clinician could consider Simon’s Two Stage
Design.

``` r
clinfun::ph2simon(pu = 0.2, pa = 0.4, ep1 = 0.05, ep2 = 0.2,nmax = 40)
```


     Simon 2-stage Phase II design 

    Unacceptable response rate:  0.2 
    Desirable response rate:  0.4 
    Error rates: alpha =  0.05 ; beta =  0.2 

            r1 n1  r  n EN(p0) PET(p0)
    Optimal  3 14 11 38  21.24  0.6982
    Minimax  4 18 10 33  22.25  0.7164

**Example interpretations following SeqSGPV monitoring of PRISM:**

1.  The estimated success probability was 0.39 (95% credible interval:
    0.23, 0.54) which is evidence that the treatment effect is at least
    trivially better than the null hypothesis
    (p![\_{ROWPE}](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;_%7BROWPE%7D "_{ROWPE}")
    = 0) and is evidence to reject the null hypothesis
    (p![\_{NULL}](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;_%7BNULL%7D "_{NULL}")
    = 0).

2.  The estimated success probability was 0.13 (95% credible interval:
    0.00, 0.33) which is evidence that the treatment effect is not
    clinically meaningful
    (p![\_{ROME}](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;_%7BROME%7D "_{ROME}")
    = 0). The evidence toward the null hypothesis is
    ![p\_{NULL} = 0.61](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;p_%7BNULL%7D%20%3D%200.61 "p_{NULL} = 0.61").

3.  The estimated success probability was 0.35 (95% credible interval:
    0.203, 0.508), which is suggestive though inconclusive evidence to
    rule out at essentially null effects
    (p![\_{ROWPE}](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;_%7BROWPE%7D "_{ROWPE}")
    = 0.07) yet is evidence to reject the null hypothesis
    (p![\_{NULL}](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;_%7BNULL%7D "_{NULL}")
    = 0).

4.  The estimated treatment effect was 0.29 (95% confidence interval:
    0.16, 0.45) which is insufficient evidence to rule out any of
    essentially null effects
    (p![\_{ROWPE}](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;_%7BROWPE%7D "_{ROWPE}")
    = 0.23), clinically meaningful effects
    (p![\_{ROME}](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;_%7BROME%7D "_{ROME}")
    = 0.16), or the null hypothesis effects
    (p![\_{NULL}](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;_%7BNULL%7D "_{NULL}")
    = 0.15).

## Example 1b: Phase II trial with delayed outcomes

The same investigator in Example 1a wants to study short term (3m)
quality of life as an endpoint. The below operating characteristics
reflect a lag (or delay) of five outcomes.

``` r
par(mfrow=c(2,2))
plot(PRISM,        stat = "lag.rejH0",         affirm=0, steps=1,lag=5,N=38,ylim=c(0.02, 0.10))
abline(h=0.05)
plot(PRISM,        stat = "lag.n",             affirm=0, steps=1,lag=5,N=38,ylim=c(10, 30))
plot(PRISMse,      stat = "lag.rejH0",         affirm=0, steps=1,lag=5,N=38,ylim=c(0, 1))
abline(h=c(0.05, 0.8, 0.9))
plot(PRISMse,      stat = "lag.n",             affirm=0, steps=1,lag=5,N=38,ylim=c(10, 30))
```

<img src="README_files/figure-gfm/unnamed-chunk-11-1.png" style="display: block; margin: auto;" />

## Example 2: Two-arm randomized trial comparing differences in mean of normal outcome

An implementation scientist wishes to compare the impact of using a
fidelity feedback measure to roll out an intervention program. Over
time, the fidelity of the program decreases. The comparison of interest
is mean difference in 12m fidelity between arms. The outcome is assumed
to be normally distributed and a greater mean difference reflects
greater fidelity in the intervention arm.

Participants will be randomized 1:1 to either the standard of care or to
receive the fidelity feedback tool.

H0: mean difference
![\\le](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;%5Cle "\le")
0  
H1: mean difference \> 0

The investigators says study can afford up to 300 participants, though a
maximum of 150 participants would be ideal, and would like to know the
design-based average sample size, Type I error, and Power across a range
of treatment effects. The investigator prefers to monitor for meaningful
effects using a 95% confidence interval.

In this study, effects deemed essentially equivalent or worse than the
null hypothesis (i.e. ROWPE) are mean differences of -0.075 and greater.
The minimal clinically meaningful effect (i.e. ROME) is an effect size
of at least -0.5.

The investigator wants a Type I error
![\\le](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;%5Cle "\le")
0.05.

``` r
# Benchmark power for single-look design
power.t.test(150/2, delta=.5, sig.level = 0.05, alternative = "one.sided")
```


         Two-sample t test power calculation 

                  n = 75
              delta = 0.5
                 sd = 1
          sig.level = 0.05
              power = 0.9196865
        alternative = one.sided

    NOTE: n is number in *each* group

``` r
power.t.test(300/2, delta=.5, sig.level = 0.05, alternative = "one.sided")
```


         Two-sample t test power calculation 

                  n = 150
              delta = 0.5
                 sd = 1
          sig.level = 0.05
              power = 0.9962682
        alternative = one.sided

    NOTE: n is number in *each* group

``` r
# Example 2
# 2 sample trial with normally distributed outcomes
# H0: mu > 0
# H1: mu < 0
# PRISM: deltaL2 = -0.5, deltaL1 = -0.075
# Assess outcomes monthly -- 25 participants per month
# Possible delayed outcomes -- 0, 25, 50, 75
# Maximum sample size -- 150, 300, Inf


system.time(PRISM2 <-  SeqSGPV(nreps            = nreps,
                               dataGeneration   = rnorm, dataGenArgs = list(n=300,sd=1),
                               effectGeneration = 0, effectGenArgs=NULL,  effectScale  = "identity",
                               allocation       = c(1,1),
                               effectPN         = 0,
                               null             = "less",
                               PRISM            = list(deltaL2 = NA,      deltaL1 = NA, 
                                                       deltaG1 = 0.075,   deltaG2 = 0.5),
                               modelFit         = lmCI,
                               modelFitArgs     = list(miLevel=.95),
                               wait             = 25,
                               steps            = c(1,25),
                               affirm           = c(0, 25),
                               lag              = c(0, 25, 50, 75),
                               N                = c(150, 300, Inf),
                               printProgress    = FALSE))
```

       user  system elapsed 
    389.837  50.445 152.707 

Assess the impact of delayed outcomes.

``` r
par(mfrow=c(1,2))
# Impact of delayed outcomes
plot(PRISM2,stat = "lag.rejH0", affirm=0, steps=25)
plot(PRISM2,stat = "lag.n",     affirm=0, steps=25)
```

<img src="README_files/figure-gfm/unnamed-chunk-14-1.png" style="display: block; margin: auto;" />

``` r
par(mfrow=c(1,2))
plot(PRISM2$mcmcECDFs$mcmcEndOfStudyEcdfN$W25_S25_A0_L75_NInf,las=1, 
     main = "Sample Size ECDF\nmu = 0, S=25, A=0, L=75, N=Inf")
plot(PRISM2$mcmcECDFs$mcmcEndOfStudyEcdfN$W25_S25_A25_L75_NInf,las=1, 
     main = "Sample Size ECDF\nmu = 0, S=25, A=25, L=75, N=Inf")
```

<img src="README_files/figure-gfm/unnamed-chunk-15-1.png" style="display: block; margin: auto;" />

Evaluate operating characteristics under a range of plausible outcomes.

``` r
# Obtain design under range of effects
se <- round(seq(-0.1, 0.7, by = 0.05),2)
system.time(PRISMse2 <- fixedDesignEffects(PRISM2, shift = se))
```

    [1] "effect: -0.1"
    [1] "effect: -0.05"
    [1] "effect: 0"
    [1] "effect: 0.05"
    [1] "effect: 0.1"
    [1] "effect: 0.15"
    [1] "effect: 0.2"
    [1] "effect: 0.25"
    [1] "effect: 0.3"
    [1] "effect: 0.35"
    [1] "effect: 0.4"
    [1] "effect: 0.45"
    [1] "effect: 0.5"
    [1] "effect: 0.55"
    [1] "effect: 0.6"
    [1] "effect: 0.65"
    [1] "effect: 0.7"

         user    system   elapsed 
     4500.758  5583.246 16393.814 

``` r
par(mfrow=c(2,2))
plot(PRISMse2, stat = "lag.rejH0", steps = 25, affirm = 0,  N = 300, lag = 75)
plot(PRISMse2, stat = "lag.n",     steps = 25, affirm = 0,  N = 300, lag = 75)
plot(PRISMse2, stat = "lag.bias",  steps = 25, affirm = 0,  N = 300, lag = 75)
plot(PRISMse2, stat = "lag.cover", steps = 25, affirm = 0,  N = 300, lag = 75, ylim=c(0.93, 0.97))
```

<img src="README_files/figure-gfm/unnamed-chunk-17-1.png" style="display: block; margin: auto;" />

ECDF of sample size for a treatment effect in the Grey Zone.

``` r
plot(PRISMse2$`effect1_0.3`$mcmcECDFs$mcmcEndOfStudyEcdfNLag$W25_S25_A0_L0_N300,las=1, 
     main = "Sample Size ECDF\nmu = 0.3")
```

<img src="README_files/figure-gfm/unnamed-chunk-18-1.png" style="display: block; margin: auto;" />

**Example interpretations following SeqSGPV monitoring of PRISM:**

1.  The estimated treatment effect was 1.05 (95% confidence interval:
    0.24, 1.85) which is evidence that the treatment effect is at least
    trivially better than the null hypothesis
    (p![\_{ROWPE}](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;_%7BROWPE%7D "_{ROWPE}")
    = 0) and is evidence to reject the null hypothesis
    (p![\_{NULL}](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;_%7BNULL%7D "_{NULL}")
    = 0).

2.  The estimated treatment effect was -0.61 (95% confidence interval:
    -1.45, 0.24) which is evidence that the treatment effect is not
    clinically meaningful
    (p![\_{ROME}](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;_%7BROME%7D "_{ROME}")
    = 0). The evidence toward the null hypothesis is
    ![p\_{NULL} = 0.86](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;p_%7BNULL%7D%20%3D%200.86 "p_{NULL} = 0.86").

3.  The estimated treatment effect was 0.26 (95% confidence interval:
    0.005, 0.514), which is suggestive though inconclusive evidence to
    rule out at essentially null effects
    (p![\_{ROWPE}](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;_%7BROWPE%7D "_{ROWPE}")
    = 0.14) yet is evidence to reject the null hypothesis
    (p![\_{NULL}](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;_%7BNULL%7D "_{NULL}")
    = 0).

4.  The estimated treatment effect was 0.30 (95% confidence interval:
    -0.10, 0.69) which is insufficient evidence to rule out any of
    essentially null effects
    (p![\_{ROWPE}](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;_%7BROWPE%7D "_{ROWPE}")
    = 0.22), clinically meaningful effects
    (p![\_{ROME}](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;_%7BROME%7D "_{ROME}")
    = 0.25), nor the null hypothesis effects
    (p![\_{NULL}](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;_%7BNULL%7D "_{NULL}")
    = 0.13).

For each conclusion, the following clarification may be provided: Based
on simulations, there may be an absolute bias, in terms of effect size,
as large 0.02 and interval coverage as low as 0.93. The bias is towards
the null for effects less than 0.31 and away from the null for effects
greater than 0.31 (see figure of simulated design-based bias and
coverage).

## Example 3: Two-arm randomized trial comparing odds ratios between groups

An investigator wants to compare the odds ratio between two groups in
which the underlying success probability is 0.35.

H0: odds ratio
![\\le](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;%5Cle "\le")
1  
H1: odds ratio \> 1

PRISM:
![\\delta\_{G1}=1.05](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;%5Cdelta_%7BG1%7D%3D1.05 "\delta_{G1}=1.05")
and
![\\delta\_{G2}=1.75](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;%5Cdelta_%7BG2%7D%3D1.75 "\delta_{G2}=1.75");
ROWPE =
(![-\\infty, 1.05](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;-%5Cinfty%2C%201.05 "-\infty, 1.05")\],
ROME =
\[![1.75, \\infty](https://latex.codecogs.com/png.image?%5Cdpi%7B110%7D&space;%5Cbg_white&space;1.75%2C%20%5Cinfty "1.75, \infty"))

This example uses a small number of replicates to get an initial sense
of operating characteristics

``` r
# Example 3
# 2 sample test, bernoulli outcomes
# H0: OR < 1
# H1: OR > 1
# PRISM: deltaG1 = 1.05, deltaG2 = 1.75
epiR::epi.sscc(OR = 1.75, p1 = NA, p0 = 0.4, n = NA, power = 0.80, r = 1,
               sided.test = 1, conf.level = 0.95, method = "unmatched", fleiss = FALSE)
```

    $n.total
    [1] 320

    $n.case
    [1] 160

    $n.control
    [1] 160

    $power
    [1] 0.8

    $OR
    [1] 1.75

``` r
system.time(PRISM3 <-  SeqSGPV(nreps            = 2000,
                               dataGeneration   = rbinom, dataGenArgs = list(n=320, size = 1, prob = 0.35),
                               effectGeneration = 1, effectGenArgs=NULL,  effectScale  = "oddsratio",
                               allocation       = c(1,1),
                               effectPN         = 1,
                               null             = "less",
                               PRISM            = list(deltaL2 = NA,   deltaL1 = NA, 
                                                       deltaG1 = 1.1, deltaG2 = 1.75),
                               modelFit         = lrCI,
                               modelFitArgs     = list(miLevel=.95),
                               wait             = 25,
                               steps            = 1,
                               affirm           = 0,
                               lag              = 0,
                               N                = 320,
                              printProgress    = FALSE))
```

       user  system elapsed 
    287.011   4.975 146.863 

``` r
se3 <- round(exp(seq(-0.1, .7, by = .1)),2)
system.time(PRISMse3 <- fixedDesignEffects(PRISM3, shift = se3))
```

    [1] "effect: 0.9"
    [1] "effect: 1"
    [1] "effect: 1.11"
    [1] "effect: 1.22"
    [1] "effect: 1.35"
    [1] "effect: 1.49"
    [1] "effect: 1.65"
    [1] "effect: 1.82"
    [1] "effect: 2.01"

        user   system  elapsed 
    2267.451   49.734 1162.765 

``` r
plot(PRISMse3, stat = "rejH0")
```

<img src="README_files/figure-gfm/unnamed-chunk-20-1.png" style="display: block; margin: auto;" />

## Example 4: Re-visiting example 1 with prior-based effect generation

This example uses a small number of replicates to demonstrate how to
obtain end of study replicates where the effect generation follows a
random distribution. For each replicate, a success probability is drawn
uniformly between 0.1 and 0.5.

``` r
system.time(PRISM1c <-  SeqSGPV(nreps            = 20,
                               dataGeneration   = rbinom, dataGenArgs = list(n=40, size=1, prob = .2),
                               effectGeneration = runif, effectGenArgs=list(n=1, min=-.1, max = .3),  effectScale  = "identity",
                               allocation       = 1,
                               effectPN         = 0.2,
                               null             = "less",
                               PRISM            = list(deltaL2 = NA, deltaL1 = NA, 
                                                       deltaG1 = .225, deltaG2 = .4),
                               modelFit         = binomCI,
                               modelFitArgs     = list(conf.level=.95, 
                                                       prior.shape1=0.005, prior.shape2=0.005,
                                                       methods="bayes", type="central"),
                               wait             = 5:10,
                               steps            = 1:3,
                               affirm           = 0:1,
                               lag              = c(0,5,10),
                               N                = 35:40,
                               printProgress    = FALSE))
```

       user  system elapsed 
      2.843   2.645   1.836 

``` r
 head(PRISM1c$mcmcEOS$W5_S1_A0_L0_N35)
```

         theta0    effect0  n         est        bias rejH0 cover stopNotROPE
    [1,]    0.2 0.20142417 35 0.342902028 -0.05852214     0     1           0
    [2,]    0.2 0.08528831  5 0.000998004 -0.28429030     0     0           0
    [3,]    0.2 0.22734277  6 0.666389351  0.23904658     1     1           1
    [4,]    0.2 0.11476886 13 0.154112221 -0.16065664     0     1           0
    [5,]    0.2 0.27245456  6 0.666389351  0.19393479     1     1           1
    [6,]    0.2 0.22059208  8 0.624843945  0.20425186     1     1           1
         stopNotROME stopInconclusive lag.n     lag.est    lag.bias lag.rejH0
    [1,]           0                1    35 0.342902028 -0.05852214         0
    [2,]           1                0     5 0.000998004 -0.28429030         0
    [3,]           0                0     6 0.666389351  0.23904658         1
    [4,]           1                0    13 0.154112221 -0.16065664         0
    [5,]           0                0     6 0.666389351  0.19393479         1
    [6,]           0                0     8 0.624843945  0.20425186         1
         lag.cover lag.stopNotROPE lag.stopNotROME lag.stopInconclusive
    [1,]         1               0               0                    1
    [2,]         0               0               1                    0
    [3,]         1               1               0                    0
    [4,]         1               0               1                    0
    [5,]         1               1               0                    0
    [6,]         1               1               0                    0
         lag.stopInconsistent lag.stopRejH0_YN lag.stopRejH0_NY wait steps affirm
    [1,]                    0                0                0    5     1      0
    [2,]                    0                0                0    5     1      0
    [3,]                    0                0                0    5     1      0
    [4,]                    0                0                0    5     1      0
    [5,]                    0                0                0    5     1      0
    [6,]                    0                0                0    5     1      0
         lag  N
    [1,]   0 35
    [2,]   0 35
    [3,]   0 35
    [4,]   0 35
    [5,]   0 35
    [6,]   0 35

[1] Stewart, T. G. & Blume, J. (2019), ‘Second-generation p-values,
shrinkage, and regularized models’, Frontiers in Ecology and Evolution
7, 486.

[2] Kruschke, J. K. (2013), ‘Bayesian estimation supersedes the t
test.’, Journal of experimental psychology. General 142(2), 573–603.

[3] Freedman, L. S., Lowe, D. & Macaskill, P. (1984), ‘Stopping rules
for clinical trials incorporating clinical opinion.’, Biometrics 40(3),
575–586.
