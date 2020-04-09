---
title: "Journal Recreation: Approximate Bayesian computation for disease outbreaks"
date: 2020-03-28T12:27:13-05:00
tags: ["statistics", "projects"]
---

This post is a summary of the final project that I completed for my Statistical Computing course in my first semester of grad school. The goal of this project was to recreate two figures from a paper authored by Tony and Stumpf, [“Simulation-based model selection for dynamical systems in systems and population biology”](https://academic.oup.com/bioinformatics/article/26/1/104/182571). This paper uses the approximate bayesian computation (or ABC) algorithm to model a few fairly complex statistical models. The model which we replicated is involved in applying ABC to compare the differences in the transmission pattern for strains of influenza viruses.

The questions of Tony and Stumpf's original analysis was to see "... whether (i) different outbreaks of the same strain and (ii) outbreaks of different molecular strains of the influenza virus can be described by the same model of disease spread."  In this post, I'll give a quick summary of how ABC works, and how my group went about implementing the algorithm to match the original authors' results.

(Quick disclaimer, I would like to give complete credit to the original authors and let it be known that this is their original work, data, and methodology. This is merely an exercise in algorithm implementation.)

## Understanding the data

In order to better explain the algorithm, I first want to describe some more specifics about the project. Half of this data comes from two outbreaks of the \_same\_ virus, with one occurring in Michigan in 1978 and 1979 and the other happening in Michigan in 1980 and 1981. We will refer to this as data table A. The second outbreak is two different viruses from two different time points. This will be referred to as table B. The data values tell how many people were infected (columns) out of the potential number of people infected (rows).

## The ABC algorithm

ABC is a seemingly magical algorithm that allows for the estimation of parameters that belong to a distribution (which in Bayesian language is referred to as the "posterior distribution"). We can find these parameters using a specified prior distribution (which is where the initial parameters for our data came from), and the data we have regarding the influenza outbreaks. There are four parameters:

1. $q\_{c1}$: The probability that someone is not infected from their community for the first virus outbreak.
2. $q\_{h1}$: The probability that someone is not infected from their household for the first virus outbreak.
3. $q\_{c2}$: The probability that someone is not infected from their community for the second virus outbreak.
4. $q\_{h2}$: The probability that someone is not infected from their household for the second virus outbreak.

Here's how we'll put these pieces together:

We begin by drawing parameters from our prior distribution. For this model, the authors specified a uniform distribution for each of the four parameters.
This is simply four independent random numbers between 0 and 1. One pair of parameters ($q\_{c1}$ and $q\_{h1}$) will belong to the first member of the viral outbreak pair. For example, if we were generating data to match data table A, we use the first two parameters to create data for the outbreak pertaining to 1978-1979, and the second two parameters for 1980-1981.

Once we have generated the pairs of parameters, we plug them into a data generating function that will generate a matrix $w$ that comes from the same probability model that our original data came from. This is: $$w\_{js} = {s\choose j}w\_{jj}(q\_cq\_h^j)^{s-j}$$ This looks confusing, and it is. The main takeaway is that $j$ represents the number of people infected out of a total number of $s$ people. $w\_{jj}$ is the jth diagonal element in the data matrix that we are currently generating. To build this data, we began by creating the first row, which is specified in the paper by $w\_{0s} = q\_c^s$. The diagonal of $w$ is created using $w\_{jj} = 1 - \sum\_{i=0}^{j-1}w\_{ij}$. The rest of the data was then created iteratively across each row of the matrix using the first equation for $w\_{js}$ and calculating $wjj$ for each diagonal element.

Now that we have the data generated, we need to create a way to measure whether the data we generated is similar to the original data. The authors did this by creating a distance metric which compares the two generated data sets to their respective original data sets using this formula: $$d(D\_0,D^{\*})=\frac{1}{2}(||D\_1 - D^\*(q\_{h1},q\_{c1})||\_F+||D\_2 -D^\*(q\_{h2},q\_{c2})||\_F)$$ where $||A||\_F = \sqrt{trace(A^TA)}$ denotes the frobenious norm, and $D^\*$ is a matrix generated using the parameters from our prior. If this distance is smaller than a given tolerance, than the parameters are deemed appropriate and are accepted. For each data table, we replicated this process 1000 times, meaning that we have two dataframes with 1000 rows and 4 columns each, and contains the accepted parameters for each simulation.

I would like to make note that the authors actually implemented a modified version of ABC, called ABC SMC. This speeds up the ABC process by giving a tolerance schedule for accepting parameters, not just one single tolerance. However, in this class we did not learn nor need to implement ABC SMC for our project, so that is the only difference between the authors results and our results. This will not actually affect our results, only how long it takes for us to get them.

## Results

The goal of this project was to replicate the authors figures, and we did. Here are our recreated figures:

![The first recreated figure](/images/res1.png)

You can see that the results from this figure matches the paper's findings. The conclusion from the figure is clear: we can see that there is a significant amount of overlap between both of the clusters. This means that the accepted parameters for both of the outbreaks are similar, which indicates that their transmission patterns are similar to one another.

![The second recreated figure](/images/res2.png)

And the results from the second simulation show us the opposite. Since these clusters have no overlap, the two different virus strains from the two different years do not have the same transmission patterns.
