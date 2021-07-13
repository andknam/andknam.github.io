---
title: Learning Machine Learning (June 28 - July 5) 
date: "2021-07-12T00:00:00+00:00"
template: "post"
draft: false
slug: "learning-ml-july5"
category: "Learning"
tags:
  - "Machine Learning"
  - "Natural Language Processing"
description: "Transformers"
socialImage: ""
---

In the first learning machine learning post, I dumped my notes on RNNs, CNNs, and Seq2Seq. Getting myself to publish a blog post (even if the post was simply a typed version of my handwritten notes) was a huge help for remembering what I learned. Seeing how useful that was, I'll be writing down some notes again. This time, instead of just copying my notes into markdown, I'll be explaining what I've learned using more blog-style writing. Let's begin :)

#### Transformers
> What is a transformer?

Building off the previous post where we looked at Seq2Seq, the **transformer** is a deep learning model that also makes use of **attention** to solve sequence-to-sequence problems. Similar to Seq2Seq, the transformer's architecture is based on **encoding** and **decoding**. The encoding portion consists of a stack of encoders, stacked one of top of the other (and so does the decoding portion). 

To use our transformer, the bottom-most encoder is given an *input sequence* (such as a sentence). This encoder considers the sequence one word at a time, and every word is represented as a *vector* (size of 512) using an embedding algorithm. Vectors at each position flow through a **self-attention** layer. The results from this layer are then fed to a **feed-forward neural network** (FNN). Importantly, every vector flows through its own path in the encoder. For the self-attention layer, there are dependencies between the paths. However, the FNN layer does not have these dependencies, so paths can be executed in parallel (speed boost!). 

#### Attention
> How does the **self attention** layer work? 

Given a sentence such as, "The animal didn't cross the street because it was too tired", our transformer needs a way to know what "it" refers to. Self-attention achieves this by baking the "understanding" of other relevant words into the word currently being processed (look at other positions to improve the word encoding). 

The calculation for self-attention using vectors (size of 64) is the following:
1. Create a **query**, **key**, and **value** vector from the encoder input (multiply the input embedding by matrices trained during the training process)
2. Take the *dot product* of the query and key vector for each word (this numerical *score* determines how much focus to place on other parts of the input sequence when encoding a given word)
3. *Divide* scores by 8 (square root of vector size) --> more stable gradients
4. *Softmax* the scores
5. *Multiply* each value vector by the softmax score
6. *Sum* up the weighted value vectors --> result is a vector!

For more speed, the self-attention calculations are done with matrices instead of vectors. 

Self-attention done this way works alright. The vectors resulting from self-attention do contain a little bit of the other encodings, but might be dominated by the word itself. To expand the transformer's ability to focus on different positions in the input sequence, we use **multi-headed attention**. 

For multi-headed attention, we maintain *separate* query, key, and value vectors for each head (each set of vectors is randomly initialized and projects the input embeddings into different subspaces after training). We follow the previous self-attention procedure (which was **single-headed attention**), eight different times with eight sets of weight matrices. This gives us eight attention heads. We concatenate the heads together and multiply this vector by a big weight matrix to obtain our final result vector. 

#### Positional Encoding and Residuals
Something we haven't considered yet is the *order of words* in the input sequence. To do so, we use **positional encoding** (add a positional encoding vector to our word embedding vector). The hope here is that adding this vector will create meaningful distances between the word embedding vectors when we calculate self-attention. 

Going back to the encoder architecture, each sub-layer in each encoder has a **residual connection** around it (same goes for decoders). The residual connection around the self-attention layer allows us to add the positionally-encoded input vector with the output of the self-attention layer. 

Also, each sub-layer is followed by a **layer normalization** step.

#### Decoders
> What is a decoder? What is the Linear Layer and Softmax Layer?

Decoders function much the same as the encoders do. The decoder input takes in the combined word embedding + positional encoding and bubbles up results just like the encoders. However, the decoder has an additional **encoder-decoder attention layer**. Similar to multiheaded self-attention, this layer works with query, key, and value matrices. Instead of multiplying the decoder input by a weights matrix, the key and value matrices come from the output of the top encoder, and the query matrix comes from the add/normalize layer below the encoder-decoder attention layer. The decoder's self-attention layer is also slightly different. The self-attention layer is only allowed to attend to earlier positions in the output sequence (future positions are **masked** to prevent "cheating" during training).

The result of decoding will be a vector of floats, one vector for each word. This vector goes through a **Linear layer** (fully-connected NN) that projects the output of the decoder stack to a **logits vector**. Assuming our model knows 10,000 words, our logits vector will be 10,000 cells long, each cell representing the score for a different word. **Softmaxing** this vector turns the scores into probabilities, from which we choose the cell with the highest probability. The word related to the index of this cell is our final output.

#### Training
Let's say we are translating the word "merci" into "thanks". The expectation for our desired output is to have a **one-hot encoded vector** where the one-hot encoded index matches the index in our output vocabulary vector. However, since our model is untrained, we will likely have a vector that has arbitrary scores for each cell. To solve this, we can compare the **probability distributions** of the untrained vector with our desired output (by subtracting one from the other). This will tell us which vector index is supposed to be one-hot encoded. 

Realistically, we will be doing this with an entire sentence and not just a single word. What we want our model to do is to successively output probability distributions where the first probability distribution will be for the first word, the second distribution for the second word, etc. Since we know our model produces one distribution at a time, we can select the word with the highest probability and simply throw away the other probabilties (this is called **greedy decoding**). We can also do a **beam search** which involves holding on to some top number of words, running our model Y times, and taking the output with less error based on X hypotheses. X (*beam_size* = number of hypotheses kept in memory) and Y (*top_beams* = number of total translations) are hyperparameters. 

#### End

Thanks for making it this far! I will be writing about BERT and GPT-3 for my next post :)

#### Where to Learn
[Jay Alammar’s Blog](https://jalammar.github.io/illustrated-transformer/) (again) \
[Medium article on Masking](https://medium.com/analytics-vidhya/masking-in-transformers-self-attention-mechanism-bad3c9ec235c) \
[StackExchange Post on Residual Connections](https://stats.stackexchange.com/questions/321054/what-are-residual-connections-in-rnns) \
[Medium Article on Transformers](https://towardsdatascience.com/illustrated-guide-to-transformers-step-by-step-explanation-f74876522bc0)