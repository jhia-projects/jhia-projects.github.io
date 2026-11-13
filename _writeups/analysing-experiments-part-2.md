---
title: "Analysing Experiments: Beyond the Difference-in-Means Test (Part 2)"
summary: "Two ways to cut noise in experiment outcomes, stratified sampling and CUPED, and how much sample size each saves on Criteo's advertising data."
repo: https://github.com/jhia-projects/experimentation-platform
tags: [experimentation, causal-inference, statistics]
order: 1
---
In [Part 1]({% link _writeups/analysing-experiments-part-1.md %}), I highlighted a few tests to conduct when designing an experiment, starting with a power calculation to work out how many users you need. In this post, I reflect on different approaches to minimise the noise in the outcome metric so that the effect of an experiment can be detected more clearly. This ties directly back to the power calculation: less noise means smaller standard errors, and smaller standard errors mean fewer users are needed to detect the same effect.

Noise in experimental data is the variance in the outcome measure that is unrelated to the treatment effect. For example, if you are trying to test if a dieting program is effective in helping individuals lose weight, you would like to ensure that, as far as possible, the spread of weights across individuals in the randomised experiment is as narrow as possible. Within a group of random individuals, there would inevitably be a very wide spread of weights depending on their initial weight due to different heights, existing diets, levels of exercise, etc. If we can narrow the spread of weights and strip out the variance which affects the outcome but is not related to entering the dieting program, the actual effect of the dieting program will stand out more clearly.

I cover two techniques. The first, stratified sampling, is applied when you design the experiment. The second, the CUPED estimator, is applied when you analyse the results.

## Technique 1: Stratified sampling

The default way to build an experiment sample is simple random sampling, where users are selected at random across the whole spectrum of users. Say you need 100 users. By chance, the treatment and control groups may end up skewed differently on the age distribution, with one group containing more younger users than the other. If age has an effect on the outcome metric, this imbalance adds to the variance in the treatment and control groups. This variance adds noise to the data and makes it more challenging to find the treatment effect signal between the groups. If the data is very noisy, you would require many more samples to pick up the effect you need to make valid statistical inferences.

Stratified sampling removes this source of noise. The population is first split into strata, and a fixed number of users is then randomly sampled into the treatment and control groups within each stratum. If the data is stratified by age into 2 groups, 1 comprising those under 30 and the other comprising those over 30, each of these groups is a stratum. To sample 100 users, you could take 50 users from each age group and randomly assign 25 of each to treatment and 25 to control. Both groups then have exactly the same age mix by construction, so age can no longer add noise to the comparison between them.

I illustrate the effects of stratification using the Criteo dataset. Because the experiment has already been run, I can't re-draw the sample. Instead, I decompose the variance in the outcome into between-strata and within-strata variances. The between-strata share is the noise that stratification would remove. Stratifying on the anonymised feature f2 reduces the variance in the outcome by 1.2%, which is equivalent to needing roughly 1.2% fewer users. The gain is small because only a small share of the variation in conversion lies between the f2 strata: f2 is only weakly related to whether a user converts. This is the general rule. Stratification helps in proportion to how strongly the stratifying variable predicts the outcome. CUPED makes that relationship explicit.

## Technique 2: CUPED estimator / regression adjustment

CUPED (Controlled-experiment Using Pre-Experiment Data) is a variance reduction method that you apply after the experiment has run. It adjusts each user's outcome *Y* using a covariate *X*:

*Y*<sub>adjusted</sub> = *Y* − θ(*X* − *X̄*), where θ = cov(*X*, *Y*) / var(*X*)

This is very similar to residualisation and regression adjustment, but the nifty bit is that it quantifies exactly how much variance is removed: the variance falls by a fraction ρ², where ρ is the correlation between the covariate and the outcome. The higher the correlation, the larger the variance reduction.

There is one important condition: the covariate must not be affected by the treatment. If it is, part of the treatment effect gets subtracted out along with the noise, and the estimate becomes biased. This is why the authors of the estimator suggest using the outcome measured before the experiment started. It cannot be influenced by the treatment, and it is usually highly correlated with the outcome.

I illustrate this with the Criteo dataset. Given that there is no pre-treatment outcome metric in the data, I use the feature 'f11' as the covariate, chosen because it is highly correlated with conversion. Like the other features in the dataset, it describes the user rather than their response to the ad, so it should not be affected by the treatment. I find that:

- The variance in the conversion outcome across users is 0.02908
- The variance in the CUPED-adjusted conversion outcomes across users is 0.02625
- The 'f11' feature has a -0.31 correlation with the conversion outcome
- The variance reduction is (-0.31)^2, which is 9.75%. This is the percentage reduction in the variance of the CUPED-adjusted outcome relative to the unadjusted outcome

There are practical implications for the difference in means calculation. The standard errors for the CUPED-adjusted outcome are 5% smaller compared to the unadjusted outcome, which is roughly equivalent to running the experiment with 1.11x as many users.

## Conclusion

Both techniques work on the same principle: strip out the variance in the outcome that can be explained by something other than the treatment. On the Criteo data, stratifying on f2 cut variance by 1.2%, while CUPED with f11 cut it by close to 10%. The difference comes down to how strongly each variable predicts conversion.

The two techniques are also suited to different situations. Stratification has to be planned before the experiment runs, but it requires no adjustment afterwards. CUPED can be applied after the fact to any experiment, as long as you have a covariate that is unaffected by the treatment. The two are not mutually exclusive either: you can stratify at the design stage and still apply CUPED at the analysis stage.

The code for both techniques is in the notebook in the linked repo. CUPED was introduced in Deng, Xu, Kohavi and Walker (2013), [*Improving the Sensitivity of Online Controlled Experiments by Utilizing Pre-Experiment Data*](https://exp-platform.com/Documents/2013-02-CUPED-ImprovingSensitivityOfControlledExperiments.pdf).
