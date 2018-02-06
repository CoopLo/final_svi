# final_svi
All of my work with two different approaches.

I'll organize this readme into a few sections to give a detailed overview of what I found helpful, work that has been done, and work that still needs to be done.

# Background

Here is the paper I originally began working from: http://proceedings.mlr.press/v32/johnson14.pdf

Here is a paper that goes more in depth with the math: https://arxiv.org/pdf/1601.00670.pdf

Here is a good explanation on what Hidden Markov Models are: http://courses.media.mit.edu/2010fall/mas622j/ProblemSets/ps4/tutorial.pdf, http://users-cs.au.dk/cstorm/courses/PRiB_f12/slides/hidden-markov-models-1.pdf

STAN information: http://mc-stan.org/users/

pySTAN api: http://pystan.readthedocs.io/en/latest/api.html


The project is to apply stochastic variational inference (SVI) to fit the parameters and hyperparameters of a Hidden Markov Model to time series stock data. Hidden Markov Models work well with stock data because they model hidden states much like how we don't know what state the stock market is currently in, so it makes it difficult to predict what will happen next. The math is difficult and very involved. What I tried to do was predict the next observation using an HMM.

There are already ways to fit Hidden Markov Models. SVI is a new way. The most common fitting algorithm is Baum-Welch. I did not look too much into either of these and don't know how they work. Briefly, SVI works by minimizing the distance between the current posterior distribution of the HMM and the expected posterior (Also known as the Kullback-Leibler divergence). It has been shown that this is equivalent to maximizing the evidence lower bound (ELBO). The original paper explains how SVI is applied to HMMs, the second explains how SVI works. I found the code provided in both papers to be full of bugs and largely unusable.

Hidden Markov Models are not too complicated for simple setups. They rely on calculating a sort of network of values that give information on the probability of getting each possible next hidden state. I could not find very many good code packages for HMMs and had to try to write them myself. HMM packages do exist, there is one called hmmlearn. The issue with that is it uses Baum-Welch for fitting. I also found it no easier to access all of the values from hmmlearn than it is to just write my own HMM. HMMs are useful for solving three problems, none of which are predicting the next observation. 1: given a sequence o observations, what is the probability that the observations are generated by the model? 2: Given a model and observations, what is the most likely state sequence in the model that would produce the observations? and 3: Given a model and observations, how do we adjust the parameters to maximize the probability of finding the sequence of observations?

From checking, what seemed like the entire internet, the best implementation of SVI is with STAN. STAN is a platform for statistical modeling that uses its own sort of programming language. There are a few tutorials online for HMMs in stan. They can be done but I found it to be very confusing.

It is kind of a pick-your-poison type situation with picking between writing the HMM or SVI algorithm. I chose to try and write the HMM because it is a lot less involved mathematically.

# Work that has been done

The SVI folder is an attempt at getting the code to work from the first paper listed above and another package that uses the first. (Here is their code if you're interested: https://github.com/mattjj/pyhsmm, https://github.com/dillonalaird/pysvihmm)

Again, I found these packages to be unusable with the amount of bugs. This may have been my error in implementation as well as the packages being written in Python 2.7, where I was using python 3.6. If I remember correctly, I debugged the inference step through the local update and something was going wrong in the global update that caused sigma to blow up. I had spent about three months deep in the package trying to figure it out before I switched over to STAN. If you would like to look into it, the files are each 1000+ lines of python with minimal comments and some comments admitting there are bugs.

My next approach was with pySTAN. STAN already has an implementation of SVI (called variational bayes), although it is experimental. This will take parameters supplied to it, along with data, and fit them using SVI. It is up to you to provide the right parameters. I was working off of this paper: https://pdfs.semanticscholar.org/a1ef/bcd1252af7cfad05cfe5509479948d28d63a.pdf on how to set up the model (Here is another paper that explains some of the steps, that I didn't have time to work through: http://www.ece.ucsb.edu/Faculty/Rabiner/ece259/Reprints/tutorial%20on%20hmm%20and%20applications.pdf). The gist of it is that you break up the stock data to make it multi-dimensional. Then you run the vb and get your output. You predict the next obesrvation on sort of a running basis, in the middle of running vb, using Forward and Backward messages from the HMM.

# Work that needs to be done

Right now, stan_svi is the most promising approach. The parameters of the HMM still need to be fleshed out, specifically the mean vector for the mth component in the jth state, and the probability of observing Ot in the multi-dimensional Gaussian distribution (both of these are explained in the semanticscholar paper). This has been worked on in k-means-stan-svi.py.

My implementation of a HMM is almost certainly wrong. This is in helper_funcs. I am not sure how to implement the guessing which observation maximizes the probability of the sequence, because it seems like it would need to be done in the middle of the SVI step.

As I have found, more bugs in code and implementation will show up as each bug is resolved. Good luck.
