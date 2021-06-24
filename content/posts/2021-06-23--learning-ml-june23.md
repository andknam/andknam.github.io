---
title: Learning Machine Learning (June 14 -- June 23) 
date: "2021-06-23T00:00:00+00:00"
template: "post"
draft: false
slug: "learning-ml-june23"
category: "Learning"
tags:
  - "Machine Learning"
  - "Natural Language Processing"
description: "Learnings from the past week and a half (RNN, Seq2Seq, CNN)"
socialImage: ""
---

In my about page, I detail a little bit about my machine learning (ML) involvement. I've worked through a good portion of Coursera's Machine Learning course, read a few chapters from Jurafsky's Speech and Language Processing textbook, and have learned a lot about the general framework for ML research from assisting natural language processing (NLP) research.

Despite all that, starting my internship, I realized my understanding of ML and NLP is still _pretty poor_. So, I decided to go through a few resources to get my understanding up to speed with my peers. Below, I've dumped some notes. 

##### Recurrent Neural Networks (RNN)

- Vanilla neural networks only work with fixed-size inputs and fixed-size outputs. RNNs allow us to have **variable-length sequences** as both inputs and outputs.
- Examples include machine translation and sentiment analysis
- RNNs are  recurrent because they use the **same set of weights** for each step
  - weights: $$W_{xh}, W_{hh}, W_{hy}$$
  - weights = matrices, biases = vectors
  - equations for $$h_t$$ and $$y_t$$:
    - $$h_t = \tanh (W_{xh} x_t + W_{hh} h_{t-1} + b_h)$$
    - $$y_t = W_{hy} h_t + b_y$$
- One-hot vectors
  - construct vocab of all words --> assign int index to ea word
  - the "one" in each one-hot vector is at word's correspond. index
  - serves as the input to forward phase 
- Backward phase
  - cross-entropy loss: $$L = -ln(p_c)$$
  - use gradient descent to minimize loss
  - to fully calculate gradient of $$W_{xh}$$ back propagation through time (BPTT)
  - clipping gradient values --> mitigate exploding gradient problem

#####  Sequence to Sequence (Seq2Seq) Models (with attention)

- Takes a sequence of items (words, letters, feature of imgs) --> outputs another sequence of items
- An encoder processes input --> compiles into a context vector 
  - input is one word (represented using word embedding) from input sentence + hidden state 
  - last hidden state of encoder is passed as context --> decoder
- Limitation: Seq2Seq with  no attention --> context vector is bad for long sentences
- Attention
  - using attention, encoder can pass **ALL** hidden states to decoder (instead of just one hidden state)
  - special attention decoder
    1. prepare inputs (encoder hidden states, decoder hidden state)
    2. score each hidden state
    3. softmax the scores
    4. multiply ea vector by its softmaxed score (amplifies hidden states with high scores)
    5. sum up the weighed vectors = context vector
    6. context vector + hidden state --> feedfoward NN --> output word 

#####  Convolutional Neural Networks (CNN)

- Neural networks that have a convolutional layer (conv layer -> have filters (2d matrices of numbers))
- Steps for filtering (ex: vertical 3x3 Sobel filter)
  1. overlay filter on input image at some location
  2. perform element-wise multiplication between vals of filter and correspond. val.s in image
  3. sum all element-wise products (sum = output val for destination pixel in output image)
  4. repeat for all locations
- Sobel filters act as **edge detectors**!
- Padding: add zeros around input image --> output image is now same size as input
  - this is called "same" padding
  - no padding = "valid" padding
- Primary parameter for conv layer is the **number of filters**
- Pooling: reduce size of input by pooling values together (i.e. max, min, avg)
- Softmax layer: fully-connected (dense) layer using Softmax funct. as activation
  - digit represented by node with highest prob. --> output of CNN 