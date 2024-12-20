---
layout: single
title:  "Conceptualizations of Generalization for Neural Networks"
excerpt: "Response on Neural Generalization"
date: 2024-11-10
mathjax: true
---
<!-- To access Preview Utilize the Shortcut Comm + Shift + V -->

# Introduction

As part of some of my recent work alongside [Hari Prakash](https://www.linkedin.com/in/hari-kishan-prakash-2b786967/), we have tried answering multiple questions regarding the quality of generalization and how to quantify it. This has mainly been as a response to the recent surge in `grokking' studies. For more background information on the subject, we highly encourage reading the following: [blogpost](https://www.beren.io/2022-01-11-Grokking-Grokking/), [preprint](https://arxiv.org/pdf/2210.01117), and [response preprint](https://arxiv.org/pdf/2405.12755). Hopefully, we will release some of our results soon, but for now, one of our biggest issues is the lack of theoretical foundation of the concepts within the field. 

# Generalization Gap
Multiple papers refer to the generalization gap as the difference between the accuracy in training vs. test datasets.[1] Although in more careful considerations[2], the train and test data need to be from the same distribution. The biggest limitation from this construction lies in the biggest limitation of neural networks, demand for data. For a reasonable behavior, networks need to be trained in distributions that are close to the state space. 

# References
[1] Ballester, R., Arnal Clemente, X., Casacuberta, C., Madadi, M., Corneanu, C. A., & Escalera, S. (2023). Predicting the generalization gap in neural networks using topological data analysis. arXiv. https://arxiv.org/abs/2203.12330

[2] Hoffer, E., Hubara, I., & Soudry, D. (2018). Train longer, generalize better: Closing the generalization gap in large batch training of neural networks [Preprint]. arXiv. https://arxiv.org/abs/1705.08741