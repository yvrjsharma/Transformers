# Transformers

It all started with Sequence to Sequence models.

Converting French sentence to English.

Encoder network

References : Sutskever et al., 2014 Sequence to Sequence learning with neural networks

How Attention works - context, which part of sentence we should be looking at, the formula for it, and how do you compute the attention weights.
Attention to be paid to an input word at time t depends on activations of encoder at time t and on the state from the previous step.
Attention model allows a neural network to pay attention to a part of input sentence while generating a translation.
Attention weights are calculated using a small neural network and backpropagation. Where to pay attention and when, was all learned during the back-propagation. 
