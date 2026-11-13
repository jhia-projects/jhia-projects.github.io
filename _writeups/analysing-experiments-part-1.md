---
title: "Analysing Experiments: Beyond the Difference-in-Means Test (Part 1)"
summary: "Three checks to run so that an experiment's results are valid for inference, illustrated on 14 million users from Criteo's advertising trial."
repo: https://github.com/jhia-projects/experimentation-platform
tags: [experimentation, causal-inference, statistics]
order: 2
---
Analysing experiments seems easy. Split users into treatment and control, calculate the average outcome metric for the two groups, and conduct a difference-in-means test for significance. If the p-value is < 5%, the treatment worked. Right?

Maybe. Rigorous analysis of experiments for causal inference requires tests to ensure results are as robust as can be. Specifically, there are a few tests that one should conduct before running an experiment so that results measured are valid for inference. I implement these tests on the Criteo dataset to illustrate how these tests can be applied in a real-world setting. The Criteo dataset contains user-level data from a randomised trial of targeted advertising. [More details here](https://ailab.criteo.com/criteo-uplift-prediction-dataset/).

## Test 1: Power calculations for sample size requirements

Running experiments can be a very expensive process, even in the digital economy. If the intervention goes wrong, it could lead to a material hit to business operations. So you want to run experiments on as small a user base as possible. But how small can you go before the results stop generalising to the wider user population? Assuming you can tolerate a level of probability that the findings made are false, statistical theory enables you to work out the number of treatment and control users to ensure this minimum tolerance is met for a given effect that you are trying to detect. In statistical parlance, this is called a power calculation.

A power calculation is about detecting an effect with an acceptable tolerance for error when applying the findings to the population. Note that there are additional facets to generalisation that it does not address, such as whether the users in the experiment are representative of the wider population - we do not discuss this in detail here.

In the dataset, sample size is unlikely to be a big problem: of the 25 million users in total, 14 million are used in this analysis. Unless the effect that one wants to detect is vanishingly small, intuitively, an experiment on 14 million users has enough power for its findings to be applied to the rest of the population at an acceptable error rate. Nonetheless, I illustrate the application of power calculations on the sample size required for the experimental analysis.

The outcome measured is a binary indicator on whether the user converted on an ad. I set the minimum effect to be detected (MDE) to be 0.5% (i.e. the intervention should minimally lead to a 0.5% increase in probability of converting) and the false positive and false negative rates to conventional levels of 5% and 20% respectively. After plugging these into standard formulas, I find that I would only require 7,000 users (3,500 in treatment and control groups each). On power alone, the results of an experiment conducted on 14 million users would therefore likely be applicable to the wider user population.

## Test 2: Sample ratio mismatch

Knowing the treatment and control allocation before the experiment is difficult. Domain expertise and experience come into play. One possible option is to run a pilot experiment to get these parameters.

In the previous analysis, I assumed that the ratio of treatment and control users is 1:1. In practice, this assumption may be violated. A sample ratio mismatch occurs when the split observed in an experiment differs from the split that was intended. For example, if you select a group of users to see a new version of a website landing page, they may not have seen the page if they do not enter the flows that get them to that page.

Because this is an ex-post analysis of an experiment that has already been conducted, I can observe the allocation directly. In this dataset, there are 11,882,655 treatment users and 2,096,937 control users, a ratio of treatment to control of roughly 5.67:1. One thing to test is whether this allocation should be expected at all. A chi-square test against an expected 1:1 split gives a statistic of 6,850,005 and a p-value of effectively zero, so the observed split is clearly not 1:1. If you are expecting a 50/50 split and it turns out to be a 5.67:1 split instead, there may be an experiment bug to be fixed, rather than crudely increasing the sample size. The 85/15 split in this dataset may well be deliberate, so the 50/50 benchmark here is purely an illustration.

If the allocation is expected, having a big gap between treatment and control users increases the number of users needed to be enlisted into the experiment. If I want to conduct a similar experiment in future, I could use this figure for my sample size calculations. If I make the same assumptions as I did for test 1 regarding MDE and error rates, along with the same 5.67:1 split but with treatment as the smaller group, I find that I will actually require 17,300 users in total! You cannot just multiply the 3,500 users in one group up by 5.67 to get this figure, because increasing sample sizes reduces standard error and this relationship is non-linear.

## Test 3: A/A test

This test involves splitting users into treatment and control groups and exposing both groups to the same experience (i.e. a placebo test). The main aim here is to check that randomisation has worked. If there are significant differences in outcomes between groups, the randomisation process may be flawed.

Because the Criteo experiment has already run, I can't run a true A/A test. I can simulate one, though, by repeatedly splitting control users into two random groups and comparing them. These are illustrated in the notebook in the linked repo
