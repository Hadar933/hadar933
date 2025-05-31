---
title: Why domain understanding still matters in ML
date: 2025-05-31 23:MM:SS +0300
author: hadar
categories: [ML]
tags: [domain-understanding] 
math: true
---
As data scientists, it's easy to forget the domain we're dealing with once the data is big enough and the models complex enough. But when we do that, we miss some of the most important aspects of machine learning and honestly, we lose some of the fun.

When you approach a problem first as a domain researcher, not as an ML researcher, a few things happen:

-  You can build and benchmark against stronger, more meaningful baselines.

- You often need less data and smaller models because you're applying domain-informed feature engineering, smarter loss functions, and better exploratory analysis.

- You naturally gain explainability, which always helps when you're facing clients or stakeholders.

Generally, you become a better ML engineer or researcher by expanding your understanding beyond the ML bubble.

At my previous company, the domain was physics. We measured indoor and outdoor pollution in real time and wanted to understand the relationship between them (specifically, how outdoor air pollution impacts the air we breathe inside our offices and buildings). Instead of jumping straight to neural networks, we started by going back to the basics: Fluid dynamics, diffusion models, and eventually simple mass balance equations from Physics 101. The models we built were light, physics-informed, and orders of magnitude smaller than the deep networks we developed later. They also gave us high explainability and helped us build a solid understanding of the system.

They were not perfect, and frankly didn't generalize as well on OOD samples as the deep models we eventually deployed, but they were critical for grounding our work in first principles. Even today, the team returns to them, not for production use, but as tools to guide representation learning, feature engineering, and stress-testing more complex models.

Today, I’m focusing on demand estimation.
Again, there are models in production, and they work. But it's crucial to ask: What is the fundamental theory behind the data?
In pollution, it was physics. In demand modeling, it’s econometrics:
As outlined in Kenneth Train’s Discrete Choice Methods with Simulation, the core question is simple:

 > 💡 What is the probability that a person chooses product $i$ over product $j$? Mathematically, this is the probability that the utility of $i$ is higher than the utility of $j$, for all $j ≠ i$.

 When you assume utility is made up of a deterministic part V (the researcher's understanding of the utility) plus a random term epsilon, a whole family of choice models is constructed, and the assumption you make on that random term is what defines the entire class of models.

As I work through these models and assumptions, I’m convinced that building intuition at this level will give me more than any premature PyTorch code ever will.

So - what's your domain? 😉

![Random Choice Problem, under the utiliy maximization framework](https://media.licdn.com/dms/image/v2/D4D22AQGWTK7Nvvf_TA/feedshare-shrink_2048_1536/B4DZbTEYBHGUAw-/0/1747297862254?e=1751500800&v=beta&t=oJPVkQI8NwRKE9vqkJL9K2IE7vZ-oXZXqa5A6eicTE0)