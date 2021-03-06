# Transformers

It all started with Sequence to Sequence models.

Converting French sentence to English.

Encoder network

References : Sutskever et al., 2014 Sequence to Sequence learning with neural networks

How Attention works - context, which part of sentence we should be looking at, the formula for it, and how do you compute the attention weights.
Attention to be paid to an input word at time t depends on activations of encoder at time t and on the state from the previous step.
Attention model allows a neural network to pay attention to a part of input sentence while generating a translation.
Attention weights are calculated using a small neural network and backpropagation. Where to pay attention and when, was all learned during the back-propagation. 

---------------

Encoder maps the input sequence into an abstract continuous representation that holds all the learned information of that input.
The decoder takes this continuous representation and step by step produces a single output while also feeding previous output to iitself.
Step by step-
1. Input is fed into a word embedding layer. This layer gives a vector representation for every word and act pretty much like a lookup table.
2. Next is to inject Positional Encodings in embeddings. This helps transformer about the order of occurence of every word in the sentences.The paper proposes using Sine and Cosine functions to do this.
3. Encoder blocks converts all input sequence into an abstract continous reprsentation that holds the learned information for this entire sequence. Encoder block has two constituents or sub-modules - Multi-headed attention, and a fully connected network on top of it. Then there are these Residual connections around these two sub-modules, which feed into Layer Normalization.

* Breaking down **Multi-headed Attention** module: This module applies a specific attention mechanism which paper refers to as the Self-Attention. This allows the model to associate each word in the input sequence to the other words in the input sequence. This can be understood using an example: The chicken did not crossed the road because it was afraid. Here model would learn to associate the word 'it' with 'chicken' and 'did' and 'not' and so on.  
  * To achieve this self-attention, we feed the input sequqnce into three distinct input layers (which are fully connected) to create three separate output vectors which are then called Query, Key and Value. 
  * The Queries and Keys undergo a dot product to produce a new matrix called Score matrix. This Score matrix would tell you how much *focus* does a word put on other words. in this matrix, every word would form the rows as ell as columns, and thus every word will have scores for every other word in the input sequence. Higher score would mean more *focus*.
  * After this step, the scores are Scaled down by dividing the matrix by the square root of the dimension of key matrix. This is done because dot products can grow large in magnitudes, specially for large input sequences, and might not train effectively using gradients.  
  * Next, we apply the Softmax function over the Score matrix and the result is referred as 'Attention Weights'. Note that Softmax essentialy gives you the probability values between 0 and 1. And by taking Softmax of values, the higher scores gets heightened while the lower values are suppressed. This has a good effect on model's learning capabilities as it becomes more clearer which words are getting higher attention values.
  * These *attention weighhts* are then multiplied with the Value vector from the very first step. Higher softmax values will help retaining those words, while lower softmax values will drown out those words.
  * This output vector is then fed through a Linear layer as well.
  * What I explained above is how Self-Attention happens. The paper further suggests to use the *Multi-headed Attention* instead as it improves the model performance.
  * For multi-headed attention, paper suggests to linearly project the initial Querys, Keys and Values N number of times (8 in the paper). Self-attention is the applied to these N vectors individually as explained above. Each Self-Attention is termed as Head in the paper. 
  * Each of this Head is producing an output vector that gets concatenated into a single vector and this output is then processed through the final Linear layer. Paper suggests that each Head is supposed to learn something different therefore giving the Encoder model better representation power.  
  * To sum Multi-headed attention - It is a module in the Transformer architecture that computes the attention weights for the input sequence and encodes this information into an output vector.

* Fully connected netwrok : 
  * The multi-headed attention output is added to the original input sequence. This is a common concept called *Residual Connections*.
  * This output is then normalized by going through Layer Normalization. 
  * This output is further fed into a position-wise fully connected Feed-Forward network. This network consists of two linear transformations with ReLU activation in between. 
  * Again, the output of this netwrok is added to its input as a residual connection and is the normalized. 
  * Note that, idea of using residual connections is to help the network train well by allowing gradients to flow through the netwrok without exploding or getting lost. Layer Normalizations helps in stabilizing networks, in substantially reducing the traiining time needed for the network. Position-wise feed-forward layers further process the attention outputs and enrich the represenations contained in it.

Encoders can be stacked N number of times, in the paper the authors suggest stacking for N=6 times. Each block will learn the attention representations in its own way, there by boosting the understanding of language for the transformer network.

---------

Encoder output is encoded representation with attention information.Idea is that Decoder will have the focus on appropriate words of the input sequence during decoding process. Decoder block basically generate text sequences. 
 * Decoder has similar structure as Encoder - It has two multi-headed attention layers, one point-wise feed-forward linear layer, residual connections after each sub-layer (multi headed attention layers and the feed-forward layers).
 * The Decoder is *Auto-Regressive* meaning any word that it is generating at timestamp t, will depend on rest of the words generated before timestamp t.
 * Decoder takes as an input the encoder sequence with attention information plus the previous output as an input that it itself has generated. For the Decoder to stop generating the words it has to generate the end token as the output.
 * Breaking down Decoder processing :
     * *Input* to the decoder goes through the embedding layer followed by the positional encoding layer to give the *positional embeddings*.
     * These embeding are then fed into first multi-headed layer to compute the attention score for Decoder's inputs. 
     * Decoders are Auto-Regressive and generates the sequence word-by-word, so this Multi-headed attention layer has to work bit differently. It has *masking* capability which allows it to avoid calculting attention scores on *future* input tokens/bvector. Any word token should only be allowed to access itself or any words occuring before it. 
     * Right after matrix multiplication and scaling is done, the authors applied a Look-Ahead Mask before applying the softmax to get probabilities. Softmax makes the attention scores zero for future words.
     * Like encoder, this multi headed attention layer has multiple heads to which masks are being applied to and this all then gets concateneated before getting processed by a Linear Layer.
     * The output of this first multi-headed attention layer is a *masked output vector* with information how the model should handle the Decoder inputs.
     * Now the second multi headed attention layer has the *encoder out* as its input Queries and Keys, while first multi-headed attention layer's output as Values. This helps in matching the Encoders Input (Text in one language) to Decoders Input (translated text word by word), which allows decoder to decide which part of input to put focus on or to attend to.
     * The output of this second  multi-headed attention layer goes into point-wise feed-forward layer for more processing.
     * Lastly the above output goes through the final Linear layer which acts as a Classification layer. This classifier is as big as the number of classes or in this case all possible second language words you have in vocabulary. Example, 10,000 classes or outputs for 10,000 words and so on.
     * Output of classifier is then passed through a Softmax layer to convert these values into probabilities scores between 0 and 1. The index of the word with highest probability gets predicted as final outcome!
     * Lastly, the Decoder takes this output and adds it to the list of Decoder inputs and continue the process again till the *end token* gets predicted.

* Note that the Decoder can be stacked N layers high as well. Each layer tking inupt from the Encoder *block* and the decoder layer just below it. Such stacking of layers helps model to extract and focus different combination of attention from its attention heads, there by boosting the prediction power!

This completes the explanation of Transformer architecture.
 
