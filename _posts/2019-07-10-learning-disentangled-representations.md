---
layout: post
title: Learning Disentangled Representations
author: Moksh Jain
keywords: representation-learning
mathjax: true
---

*You can find the interactive notebook accompanying this article* [**here**](https://colab.research.google.com/drive/1RPzxB9DZnQmoggIrwTk_c_8DdZqRXC2p).

A **representation** in the most vague sense refers to the lower dimensional projection of some high-dimensional input. A good representation can then be defined as one that captures the relevant information required to describe the original high-dimensional data in a much more compact way (i.e $$num\_features$$ << $$input\_dims$$ ). There has been a lot of interest in the Machine Learning community to build models that can learn useful representations from high dimensional sensory inputs like audio, video, text, images, etc. These representations can then be used to have further models to perform useful tasks, like classifying images. The basic idea is having lower dimensional representations that can describe the original data is useful for models to extract more useful information than the original higher dimensional data. Representation Learning has become an important research area in the recent years. In their survey, [Bengio et al.](https://arxiv.org/abs/1206.5538) talk about the need for representation learning and the latest developments in the area. According to the survey, informally, the goal of representation learning is to find useful transformations $$r(x)$$ of the higher dimensional data $$x$$ which makes it easier to extract useful information for various predictors. However, since the survey was published a lot of work has been done in this area, and one of the focuses has been of learning disentangled representations. 

## What is a disentangled representation? 
One of the underlying asumptions in representation learning is that the high dimensional sensory data in the real world $$x$$, like an image, is generated by a 2-step generative process. The first step is sampling a semantically meaningful latent variable $$z$$ (from $$P(z)$$) that describes the high level information of the data, for example the location of a flower in the image, the color of the flower, it's shape etc. The final step is to sample the actual observation $$x$$ from the conditional distribution $$P(x|z)$$. This essentially means that the high dimensional observation $$x$$ can be explained semantically by the lower dimensional representation $$z$$. [Locatello et al](https://arxiv.org/abs/1811.12359)., suggest a few characteristics for a $$disentangled$$ $$representation$$ $$z$$:
* contain all information in $$x$$ in a compact and interpretable structure
* independent of the task being performed (eg. classification, etc)
* should be useful for (semi-)supervised learning of downstream tasks, transfer and few shot learning
* They should enable to integrate out nuisance factors, to perform interventions, and to answer counterfactual questions.

The intuitive explanation adopted for disentangled representations is as follows: _a disentangled representation should separate the distinct,
informative factors of variations in the data_. That is, changing one factor ($$z_i$$) in $$z$$ should result in only a single factor in $$x$$. In essence, if one feature in the representation changes it only affects one semantic feature of the observation. Let us consider the example of an image with an object. A _good_ disentangled representation in this case would capture the location (xy-coordinates), shape, color and size as the _factors of variation_. This is a good disentangled representation since, changing on of the factors (let's say the color) affects only the color and not the shape, size or location. 

This however is just a loose conceptual intuition behind the idea of disentangled representation. In fact, until recently there was no widely agreed upon solid definition for disentangled representations. Instead there were a number of different metrics proposed over the years that would capture these intuitions. Recently, [Higgins, Amos et al.](https://arxiv.org/abs/1812.02230) proposed a formal definition of disentangled representations using the idea of symmetry transforms and from group and representation theory. This formalism helps in setting up a concrete definition for the problem being solved and helps in evaluating and understanding approaches to solve the problem. Their definition is as follows: 

**_A vector representation is called a disentangled representation with respect to a
particular decomposition of a symmetry group into subgroups, if it decomposes into independent subspaces, where each subspace is affected by the action of a single subgroup, and the actions of all other subgroups leave the subspace unaffected._**

A symmetry transform of an object is a $$transformation$$ that leaves certain properties of the object $$invariant$$. For example, translation and rotation are symmetries of objects – an apple is still an apple whether it is placed on a table or in a bag, and whether it rolls on its side or remains upright. The set of such transformations forms the $$symmetry$$ $$group$$ and the effects of these transformations are the $$actions$$ of the symmetry group on the world state(Note: this the underlying world state and not the observation $$x$$). The actions that change only a certain aspect of the world state while keeping others fixed is a $$disentangled$$ $$group$$ $$action$$. So for example changing the horizontal position of apple only affects it's horizontal position and not it's vertical position or color, etc. Another thing we notice from this is that we can decompose this symmetry group into $$symmetry$$ $$subgroups$$. So in the example of the apple, horizontal transformation could be one such subgroup. Here the horizontal subspace is affected only by actions of the horizontal translation subgroup. So far we talked about the underlying abstract world state. To generalise to observations, we assume there is a generative process that generates the dataset of observations from a given set of underlying world states. In some situations, it is possible to find a composite mapping between the disentangled group actions in the abstract state space to the transformations in the vector space of the representation. In short, we can call a representation $$disentangled$$ if the vector space of the representation can be decomposed into independent subspaces such that each subspace is only affected by a single symmetry subgroup, which in turn is a set of symmetry transformations that affect only a certain aspect of the world state. The paper decribes the formalism in further detail and also discusses link between the proposed definition and the currently generally accepted intuitive ideas about disentangled representations.

One might question how are these representations useful? As we saw previously, disentangled representations capture independent features that describe a single aspect of the observation. This characteristic is useful in enabling generalisation to previously unobserved situations, since a model can extract meaningful information about the observation to understand it from the disentangled representation. Approches using disentangled representations have found a lot of successs in various tasks including [curiousity driven exploration](https://arxiv.org/abs/1807.01521), [abstract reasoning](https://arxiv.org/abs/1811.04784), [visual concept learning](https://arxiv.org/pdf/1707.03389.pdf) and [domain adaptation in reinforcement learning](https://arxiv.org/pdf/1707.08475.pdf).

## How to learn these disentangled representations? 
Learning disentangled representations is at it's core a type of dimensionality reduction problem. The distinction here from other forms of dimensionality reduction is that there are certain restrictions on the vector space of the learned representation. Unsupervised learning of these representations is an interesting problem since it would allow models to learn from huge troves of available unlabelled data. Thus, there has been a lot of interest in the machine learning community to design unsupervised learning algorithms to learn these representations. Variants of variational autoencoders (proposed by [Kingma and Welling](https://arxiv.org/abs/1312.6114) in 2013) have seen quite a lot of success in recent years in tackling this problem, and provide state of the art performance in unsupervised learning of disentangled representations. Variational Autoencoders can be seen as modelling the 2-step generative process described above. A specific prior $$P(z)$$ is selected, and then the distribution $$P(x|z)$$ is parameterized using a deep neural network. The goal is to infer good values of the latent variables given observed data, which is essentially computing the posterior $$P(z|x)$$. This distribution $$P(z|x)$$ is approximated using a variational distribution $$Q(z|x)$$ which is also parametrized by a neural network. The representation is usually taken to be the mean of $$Q(z|x)$$. We discuss the specifics of VAEs in later sections. Several models based on this, such as BetaVAE, FactorVAE, and AnnealedVAE among others, have been introduced to learn disentangled representations, and provide state-of-the-art performance. 

However, in their recent work, [Locatello et al.](https://arxiv.org/pdf/1811.12359.pdf) perform a large systematic study of these models to evaluate the recent progress in the area. Their study had a few key findings:
* They found no empirical evidence that the considered models can be used to reliably learn disentangled representations in an unsupervised way, since random seeds and hyperparameters seem to matter more than the model choice. That is, even if a large number of models are trained with some of them being disentangled, these disentangled representations cannot be identified without access to ground-truth labels. 
* Good hyperparameter values do not appear to consistently transfer across the datasets.
* They were not able to validate the assumption that disentanglement is useful for downstream tasks, e.g., few-shot learning with disentangled representations.

In addition to these findings, they also present the _Impossibilty Result_ which states the following: *__unsupervised learning of disentangled representations is impossible without inductive biases on both the data set and the models__*. So it is impossible to learn disentangled representations without making certain assumptions on the dataset and incorporating them in the model, which essentially restricts generalizability of models across datasets. They also propose observations for future research on the topic and to that end released the [`disentanglement_lib`](https://github.com/google-research/disentanglement_lib/) with all the models used in their study to aid in future research in topic, along with the [NeurIPS 2019: Disentanglement Challenge](https://www.aicrowd.com/challenges/neurips-2019-disentanglement-challenge) to accelerate research in the area. 

## Variational Autoencoders
As  discussed in the previous sections, we start by assuming a specific prior $$p(z)$$ on the latent space, parametrizing the distribution $$p(x|z)$$ using a neural network, and approximating the posterior $$p(z|x)$$ with a neural network parameterized variational distribution $$q(z|x)$$. Now we discuss the motivations behind this model and how we train these models.

<span>What we want the model to do is to learn how to generate the representation given the data as input, i.e compute $$p(z|x)$$, and also the model should be able to generate the data given the latent representation (compute $$p(x|z)$$). We start by sampling $$z$$ from the prior $$p(z)$$. The likelihood of the data conditioned to latent variable $$z$$ is $$p(x|z)$$. The joint distribution $$p(x, z)$$ can be decomposed as $$p(x,z) = p(x|z)p(z)$$. Now at first glance calculating the posterior $$p(z|x)$$ might seem straightforward using the Bayes rule: $$p(z|x) = \frac{p(x|z)p(z)}{p(x)}$$</span>

<span>However, computing $$p(x)=\int p(x|z)p(z)dz$$ is not computationally tractable. Thus, we approximate the posterior $$p(z|x)$$ with a family of distributions $$q_\lambda (z|x)$$ (here $$\lambda$$ is used as an index for the distributions). Kullback-Leibler divergence(KL divergence) is used to measure how different a probability distribution is from another given probability distribution. We use this to evaluate how well $$q_\lambda (z|x)$$ approximates $$p(z|x)$$. Our goal would be to have the distributions be as similar as possible, so we minimize the KL-divergence.</span>

$$\mathbb{KL}(q_\lambda (z|x)\ ||\ p(z|x)) = \mathbf{E}_q[\log q_\lambda (z|x)] - \mathbf{E}_q[\log p(x, z)] + \log p(x)$$

But we encounter $$p(x)$$ once again. To get around this we use the ELBO (Evidence Lower Bound).

$$ELBO(\lambda) = \mathbf{E}_q[\log p(x, z)] - \mathbf{E}_q[\log q_\lambda (z|x)]$$

Thus from these two equations we get the  following:

$$\log p(x) = ELBO(\lambda) + \mathbb{KL}(q_\lambda (z|x)\ ||\ p(z|x))$$

Since the Jensen inequality states that the KL divergence is always $$\geq 0$$, KL-divergence can be minimized by maximizing ELBO (as $$p(x)$$ doesn't change). Maximizing the ELBO is computationally tractable, thus we can train the model with the objective of maximizing ELBO. Now, since no datapoint shares its latent $$z$$ with the latent variable of another datapoint, we can decompose ELBO into a sum such that each term depends on one datapoint. 

$$ELBO_i(\lambda)=\mathbf{E}_{q_\lambda} [\log p(x_i | z)] - \mathbb{KL}(q_\lambda (z|x_i) || p(z))$$

This value can be interpreted as follows: The first term is the reconstruction loss for the datapoints (i.e. get $$z$$ from $$x$$ and then obtain $$x'$$ and compare $$x$$ and $$x'$$) and the KL-divergence term acts as a sort of regularizer. 

<span>As mentioned previously, the distrbutions can be parametrized by neural networks. So we start with the approximate posterior, which is also called encoder as it encodes the input data into the latent variable $$q_\theta (z|x, \lambda)$$(where $$\theta$$ indicates the neural network weights), which outputs the $$\lambda$$ for a given datapoint $$x$$. As mentioned earlier $$\lambda$$ is an index over the family of distrbutions $$q$$, so we use $$\lambda$$ to get the required distribution and sample the latent representation $$z$$ from it. For example if we select a family of gaussians then $$\lambda$$ would be the mean and variance of the distributions. Once we have $$z$$ we obtain the reconstruction from the 'decoder', $$p_\phi (x|z)$$. And the loss function is $$-ELBO$$ which we can minimize using stochastic gradient descent. </span>

<span>This was the general idea behind a variational autoencoder. Now to allow these models to learn disentangled representations, the general approach is to enforce a factorized aggregated posterior $$\int q(z|x)p(x)dx$$ to encourage disentanglement. All of the approaches try to enforce this in some way by either modifying the regularizer or having additional objectives or by some architectural choices. </span>

## Summary
In this post we discussed what are disentangled representations, what are autoencoders, and how we can use variational autoencoders to learn disentangled representations. In the accompanying notebook we demonstrate how to get started by building a custom VAE with the disentanglement_lib, evaluating it and visualising it. If you are interested in disentangled representations, do consider participating in the [NeurIPS 2019: Disentanglement Challenge](https://www.aicrowd.com/challenges/neurips-2019-disentanglement-challenge).

## References
Locatello, Francesco et al. [“Challenging Common Assumptions in the Unsupervised Learning of Disentangled Representations.”](https://arxiv.org/abs/1811.12359) ICML (2018).

Higgins, Irina et al. [“Towards a Definition of Disentangled Representations.”](https://arxiv.org/abs/1812.02230) ArXiv abs/1812.02230 (2018)

[Tutorial - What is a variational autoencoder? - Jaan Alatosaar](https://jaan.io/what-is-variational-autoencoder-vae-tutorial/)

[Google AI Blog: Evaluating the Unsupervised Learning of Disentangled Representations](https://ai.googleblog.com/2019/04/evaluating-unsupervised-learning-of.html)