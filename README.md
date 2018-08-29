#### Udacity Artificial Intelligence Nanodegree
### Term 2: Capstone Project
# Machine Translation with RNNs

##### &nbsp;

<img src="images/translation.gif" width="100%" align="top-left" alt="" title="RNN" />

*Image credit: [xiandong79.github.io](https://xiandong79.github.io/seq2seq-%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86)*


##### &nbsp;

## Goal
In this project, we build a deep neural network that functions as part of a machine translation pipeline. The pipeline accepts English text as input and returns the French translation. The goal is to achieve the highest translation accuracy possible.

##### &nbsp;

## Background
The ability to communicate with one another is a fundamental part of being human. There are nearly 7,000 different languages worldwide. As our world becomes increasingly connected, language translation provides a critical cultural and economic bridge between people from different countries and ethnic groups. Some of the more obvious use-cases include:
- **business**: international trade, investment, contracts, finance
- **commerce**: travel, purchase of foreign goods and services, customer support
- **media**: accessing information via search, sharing information via social networks, localization of content and advertising
- **education**: sharing of ideas, collaboration, translation of research papers
- **government**: foreign relations, negotiation  


To meet this need, technology companies are investing heavily in machine translation. This investment paired with recent advancements in deep learning have yielded major improvements in translation quality. According to Google, [switching to deep learning produced a 60% increase in translation accuracy](https://www.washingtonpost.com/news/innovations/wp/2016/10/03/google-translate-is-getting-really-really-accurate) compared to the phrase-based approach used previously. Today, translation applications from Google and Microsoft can translate over 100 different languages and are approaching human-level accuracy for many of them.

However, while machine translation has made lots of progress, it's still not perfect. :grimacing:

<img src="images/fuck-veges.png" width="50%" align="top-left" alt="" title="RNN" />

_Bad translation or extreme carnivorism?_


##### &nbsp;

## Approach
To translate a corpus of English text to French, we need to build a recurrent neural network (RNN). Before diving into the implementation, let's first build some intuition of RNNs and why they're useful for NLP tasks.

#### RNN Overview
<img src="images/rnn-simple-folded.png" height="20%" align="right" alt="" title="Simple RNN - Folded View" />
RNNs are designed to take sequences of text as inputs or return sequences of text as outputs, or both. They're called recurrent because, the network's hidden layers have a loop in which the output from one time step becomes an input at the next time step. This recurrence serves as a form of memory. It allows contextual information to flow through the network so that relevant outputs from previous time steps can be applied to network operations at the current time step.

This is analogous to how we read. As you read this post, you're storing important pieces of information from previous words and sentences and using it as context to understand each new word and sentence.

Other types of neural networks can't do this. Imagine you're using a convolutional neural network (CNN) to perform object detection in a movie. Currently, there's no way for information from objects detected in previous scenes to inform the model's detection of objects in the current scene. For example, if a courtroom and judge were detected in a previous scene, that information could help correctly classify the judge's gavel in the current scene (instead of misclassifying it as a hammer or mallet). But CNNs don't allow this type of time-series context to flow through the network like RNNs do.


#### RNN Setup
Depending on the use-case, you'll want to setup your RNN to handle inputs and outputs differently. For this project, we'll use a many-to-many process where the input is a sequence of English words and the output is a sequence of French words (see fourth model from the left in the diagram below).

<img src="images/rnn-sequence-types.png" width="100%" align="top-left" alt="" title="RNN Sequence Types" />
<Andrej Karpathy diagram of different sequence types>

> Each rectangle is a vector and arrows represent functions (e.g. matrix multiply). Input vectors are in red, output vectors are in blue and green vectors hold the RNN's state (more on this soon).

> From left to right: (1) Vanilla mode of processing without RNN, from fixed-sized input to fixed-sized output (e.g. image classification). (2) Sequence output (e.g. image captioning takes an image and outputs a sentence of words). (3) Sequence input (e.g. sentiment analysis where a given sentence is classified as expressing positive or negative sentiment). (4) Sequence input and sequence output (e.g. Machine Translation: an RNN reads a sentence in English and then outputs a sentence in French). (5) Synced sequence input and output (e.g. video classification where we wish to label each frame of the video). Notice that in every case are no pre-specified constraints on the lengths sequences because the recurrent transformation (green) is fixed and can be applied as many times as we like.

*Image and quote source: [karpathy.github.io](http://karpathy.github.io/2015/05/21/rnn-effectiveness/)*


##### &nbsp;

#### Building the Pipeline
Below is a summary of the various preprocessing and modeling steps. The high-level steps include:

1. **Preprocessing**: load and examine data, cleaning, tokenization, padding
1. **Modeling**: build, train, and test the model
1. **Prediction**: generate specific translations of English to French, and compare the output translations to the ground truth translations
1. **Iteration**: iterate on the model, experimenting with different architectures

For a more detailed walkthrough including the source code, check out the Jupyter notebook in the main directory ([machine_translation.ipynb](https://github.com/tommytracey/AIND-Capstone/blob/master/machine_translation.ipynb)).

#### Toolset
We use Keras for the frontend and TensorFlow for the backend in this project. I prefer using Keras on top of TensorFlow because the syntax is simpler, which makes building the model layers more intuitive. However, there is a trade-off with Keras as you lose the ability to do fine-grained customizations. But this won't affect the models we're building in this project.  

##### &nbsp;

## Preprocessing

#### Load & Examine Data
Here is a sample of the data. The inputs are sentences in English; the outputs are the corresponding translations in French.

> <img src="images/training-sample.png" width="100%" align="top-left" alt="" title="Data Sample" />

##### &nbsp;

When we run a word count, we can see that the vocabulary for the dataset is quite small. This was by design for this project. This allows us to train the models in a reasonable time.

> <img src="images/vocab.png" width="75%" align="top-left" alt="" title="Word count" />

#### Cleaning
No additional cleaning needs to be done at this point. The data has already been converted to lowercase and split so that there are spaces between all words and punctuation.

_Note:_ For other NLP projects you may need to perform additional steps such as: remove HTML tags, remove stop words, remove punctuation or convert to tag representations, label the parts of speech, or perform entity extraction.  

#### Tokenization
Next we need to tokenize the data&mdash;i.e., convert the text to numerical values. This allows the neural network to perform operations on the input data. For this project, each word and punctuation mark will be given a unique ID. (For other NLP projects, it might make sense to assign each character a unique ID.)

When we run the tokenizer, it creates a word index, which is then used to convert each sentence to a vector.

> <img src="images/tokenizer.png" width="100%" align="top-left" alt="" title="Tokenizer output" />

#### Padding
When we feed our sequences of word IDs into the model, each sequence needs to be the same length. To achieve this, padding is added to any sequence that is shorter than the max length (i.e. shorter than the longest sentence).

> <img src="images/padding.png" width="50%" align="top-left" alt="" title="Tokenizer output" />

#### One-Hot Encoding (not used)
In this project, our input sequences will be a vector containing a series of integers. Each integer represents an English word (as seen above). However, in other projects, sometimes an additional step is performed to convert each integer into a one-hot encoded vector. We don't use one-hot encoding (OHE) in this project, but you'll see references to it in certain diagrams (like the one below). I just didn't want you to get confused.  

<img src="images/RNN-architecture.png" width="40%" align="right" alt="" title="RNN architecture" />

One of the advantages of OHE is efficiency since it can [run at a faster clock rate than other encodings](https://en.wikipedia.org/wiki/One-hot#cite_note-2). The other advantage is that OHE better represents categorical data where there is no ordinal relationship between different values. For example, let's say we're classifying animals as either a mammal, reptile, fish, or bird. If we encode them as 1, 2, 3, 4 respectively, our model may assume there is a natural ordering between them, which there isn't. It's not useful to structure our data such that mammal comes before reptile and so forth. This can mislead our model and cause poor results. However, if we then apply one-hot encoding to these integers, changing them to binary representations&mdash;1000, 0100, 0010, 0001 respectively&mdash;then no ordinal relationship can be inferred by the model.

But, one of the drawbacks of OHE is that the vectors can get very long and sparse. The length of the vector is determined by the vocabulary, i.e. the number of unique words in your text corpus. As we saw in the data examination step above, our vocabulary for this project is very small&mdash;only 227 English words and 355 French words. By comparison, the [Oxford English Dictionary has 172,000 words](https://en.oxforddictionaries.com/explore/how-many-words-are-there-in-the-english-language/). But, if we include various proper nouns, words tenses, and slang there could be millions of words in each language. For example, [Google's word2vec](http://mccormickml.com/2016/04/12/googles-pretrained-word2vec-model-in-python/) is trained on a vocabulary of 3 million unique words. If we used OHE on this vocabulary, the vector for each word would include one positive value (1) surrounded by 2,999,999 zeros!

And, since we're using embeddings (in the next step) to further encode the word representations, we don't need to bother with OHE. Any efficiency gains aren't worth it on a data set this small.  


##### &nbsp;

## Modeling
_UNDER CONSTRUCTION: final version coming soon_

First, let's breakdown the architecture of a RNN at a high level. Referring to the diagram above, there are a few parts of the model we to be aware of:

1. **Inputs** &mdash; Input sequences are fed into the model with one word for every time step. Each word in encoded as a unique integer or one-hot encoded vector that maps to the English dataset vocabulary.
1. **Embedding Layers** &mdash; Embeddings are used to convert each word to a vector. The size of the vector depends on the complexity of the vocabulary.
1. **Recurrent Layers (Encoder)** &mdash; This is where the context from word vectors in previous time steps is applied to the current word vector.
1. **Dense Layers (Decoder)** &mdash; These are typical fully connected layers used to decode the encoded input into the correct translation sequence.
1. **Outputs** &mdash; The outputs are returned as a sequence of integers or one-hot encoded vectors which can then be mapped to the French dataset vocabulary.

##### &nbsp;

#### Embeddings
Embeddings allow us to capture more precise syntactic and semantic word relationships. This is achieved by projecting each word into a n-dimensional space. Words with similar meanings occupy similar regions of this space; the closer two words are, the more similar they are. And often the vectors between words represent useful relationships, such as gender, verb tense, or even geo-political relationships.

<img src="images/embedding-words.png" width="100%" align-center="true" alt="" title="Gated Recurrent Unit (GRU)" />

Training embeddings on a large dataset from scratch requires a huge amount of data and computation. So, instead of doing it ourselves, we'd normally use a pre-trained embeddings package such as [GloVe](https://nlp.stanford.edu/projects/glove/) or [word2vec](https://mubaris.com/2017/12/14/word2vec/). When used this way, embeddings are a form of transfer learning. However, since our dataset for this project has a small vocabulary and little syntactic variation, we'll use Keras to train the embeddings ourselves.

##### &nbsp;

#### Encoder & Decoder
Our sequence-to-sequence model links two recurrent networks: an encoder and a decoder. The encoder summarizes the input into a context variable, also called a state. This context is then decoded and the output sequence is generated.

##### &nbsp;

<img src="images/encoder-decoder-context.png" width="60%" align="top-left" alt="" title="Encoder Decoder" />

_Image credit: [Udacity](https://classroom.udacity.com/nanodegrees/nd101/parts/4f636f4e-f9e8-4d52-931f-a49a0c26b710/modules/c1558ffb-9afd-48fa-bf12-b8f29dcb18b0/lessons/43ccf91e-7055-4833-8acc-0e2cf77696e8/concepts/be468484-4bd5-4fb0-82d6-5f5697af07da)_

##### &nbsp;

Since both the encoder and decoder are recurrent, they have loops which process each part of the sequence at different time steps. To picture this, it's best to unroll the network so we can see what's happening at each time step.

For the sentence below, it takes four time steps to encode the entire input sequence. At each time step, the encoder "reads" the input word and performs a transformation on its hidden state. Then it passes that hidden state to the next time step. Keep in mind that the hidden state represents the relevant context flowing through the network. The bigger the hidden state, the greater learning capacity of the model, but also the greater computation requirements. We'll talk more about the transformations within the hidden state when we cover gated recurrent units (GRU) and long short-term memory (LSTM).

<img src="images/encoder-decoder-translation.png" width="100%" align="top-left" alt="" title="Encoder Decoder" />

_Image credit: modified version from [Udacity](https://classroom.udacity.com/nanodegrees/nd101/parts/4f636f4e-f9e8-4d52-931f-a49a0c26b710/modules/c1558ffb-9afd-48fa-bf12-b8f29dcb18b0/lessons/43ccf91e-7055-4833-8acc-0e2cf77696e8/concepts/f999d8f6-b4c1-4cd0-811e-4767b127ae50)_

##### &nbsp;

For now, notice that for each time step after the first word in the sequence there are two inputs: the hidden state and a word from the sequence. For the encoder, it's the _next_ word in the input sequence. For the decoder, it's the _previous_ word from the output sequence.

Also, remember that when we say "word" here, we really mean the _vector representation_ of the word which comes from the embedding layer.

##### &nbsp;

#### Bidirectional Layer
- backward and forward context

<img src="images/bidirectional.png" width="70%" align="center" alt="" title="Bidirectional Layer" />

##### &nbsp;
##### &nbsp;
##### &nbsp;

#### Gated Recurrent Unit (GRU)
##### &nbsp;

<img src="images/gru.png" width="70%" align-center="true" alt="" title="Gated Recurrent Unit (GRU)" />

_Image Credit: [analyticsvidhya.com](https://www.analyticsvidhya.com/blog/2017/12/introduction-to-recurrent-neural-networks/gru/)_

##### &nbsp;
##### &nbsp;
##### &nbsp;

## Results
_UNDER CONSTRUCTION: final version coming soon_



##### &nbsp;

## Future Improvements
_UNDER CONSTRUCTION: final version coming soon_

- do proper data split (train, validation, test)
- LSTM + attention
- Residual layers (Google paper)
- Embedding Language Model (ELMo)
- Transformer model
- train on different text corpuses

##### &nbsp;
##### &nbsp;


#### LSTM
- Not tested in this project, but done is separate Udacity project found [here](https://github.com/tommytracey/udacity/tree/master/deep-learning-nano/projects/4-language-translation#build-the-neural-network)

<img src="images/lstm-gates.png" width="70%" align="center" alt="" title="LSTM cell" />

##### &nbsp;
##### &nbsp;

#### Attention (not tested)

<img src="images/attention.png" width="70%" align="center" alt="" title="Encoder Decoder" />

##### &nbsp;
##### &nbsp;
##### &nbsp;


---

# Project Starter Code
In case you want to run this project yourself, below is the project starter code.

## Setup
The original Udacity repo for this project can be found [here]().

This project requires GPU acceleration to run efficiently. Support is available to use either of the following two methods for accessing GPU-enabled cloud computing resources.

### Udacity Workspaces (Recommended)

Udacity Workspaces provide remote connection to GPU-enabled instances right from the classroom. Refer to the classroom lesson for this project to find an overview of navigating & using Jupyter notebook Workspaces.

### Amazon Web Services (Optional)

Please refer to the Udacity instructions for setting up a GPU instance for this project, and refer to the project instructions in the classroom for setup. The recommended AMI should include compatible versions of all required software and libraries to complete the project. [link for AIND students](https://classroom.udacity.com/nanodegrees/nd889/parts/16cf5df5-73f0-4afa-93a9-de5974257236/modules/53b2a19e-4e29-4ae7-aaf2-33d195dbdeba/lessons/2df3b94c-4f09-476a-8397-e8841b147f84/project)

### Install
- Python 3
- NumPy
- TensorFlow 1.x
- Keras 2.x

## Submission
When you are ready to submit your project, do the following steps:
1. Ensure you pass all points on the [rubric](https://review.udacity.com/#!/rubrics/1004/view).
2. Submit the following in a zip file:
  - `helper.py`
  - `machine_translation.ipynb`
  - `machine_translation.html`

### Converting to HTML

There are several ways to generate an HTML copy of the notebook:

 - Running the last cell of the notebook will export an HTML copy

 - Navigating to **File -> Download as -> HTML (.html)** within the notebook

 - Using `nbconvert` from the command line

    $ pip install nbconvert
    $ nbconvert machine_translation.ipynb
