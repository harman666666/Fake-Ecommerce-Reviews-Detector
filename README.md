Please have python3 and pip installed on computer before running commands below. 

Install dependencies like so:
pip install -r requirements.txt

Download the 50d GloVe Model from here: 
https://nlp.stanford.edu/projects/glove/

############################################################
To train a model, edit configuration in train.py. Then run:

python train.py

############################################################
To test a model, such as the trained model, trained-lstm-epoch-22000, 
put model path of model to test in variable found at the top of test.py script called MODEL_TO_TEST_PATH, 
and then run:

python test.py

##############################################

ANALYSIS:

LSTM Model 

One way to detect fake reviews is by using a long short-term memory (LSTM) model which is a type of recurrent neural network. Recurrent neural networks (RNN) are widely used to achieve state of the art performance in natural language processing (NLP) tasks because they are able to operate on sequences of varying length. To understand recurrent neural networks, one must first understand feed forward neural networks which are the simplest kind of neural network. Feed forward neural networks work using layers of neurons where each layer processes input from the previous layer, and feeds their output to the next layer. The neurons each have their own set of trainable weights that are initially random numbers, and they have the same number of weights as the number of inputs into the layer. To create an output from a neuron, the neuron’s trainable weights are dotted (dot product) with the inputs into the layer, and then a nonlinearity is applied on the dot product to produce that neuron's output. The first neural layer processes the training input, to create output with a size equal to the number of neurons in that layer; this layer’s output is fed into the next layer as input. The final layer’s output is compared with the training target value to create an error signal that is used to train the network’s neuron’s weights with an algorithm called backpropagation.  A recurrent neural network is different from a feed forward network because it specializes in processing sequences. The RNN processes an element of the sequence, creates an output for that element, and then feeds that output back into the same layer in the RNN to help it process the next element in the sequence. Further details of the RNN are discussed below. The implementation for the LSTM has been done in Python. The dataset for training and testing the LSTM will be a collection of Amazon reviews. The dataset consists of reviews in which 50% are labelled as real and the other 50% labelled as fake.  

The libraries used to implement the LSTM in Python are PyTorch, which is a deep learning framework, torchtext, which provides helper methods to convert raw text data into fixed length sequences for training an LSTM (the importance of fixed length sequences is discussed below), SpaCy for general purpose natural language processing tasks such as converting text into a vector of words. Other libraries used were TextBlob, which gets the sentiment of a piece of text ranging from -1.0 to 1.0, where -1.0 is  extremely negative, and 1.0 is extremely positive. The library Scikit-learn is also used for miscellaneous tasks such as generating classification reports from the predictions of the LSTM. Seaborn and matplotlib are visualization libraries that were used to create the confusion matrix after evaluating the trained model on the test set. Finally, Pandas and NumPy were used to hold, process, and clean the data to be used for training and testing. A CRAN library for building general-purpose deep learning solutions such as the LSTM in R is “kerasR” which is a wrapper over Keras, a widely-used deep learning Python framework. Similarly, there exists the CRAN library “spacyr” which is a wrapper over SpaCy. To simulate TextBlob in R, one can use the library sentimentR. 

The implementation starts by converting a fake review into a vector of words; this is done using SpaCy, with a technique called tokenization. After converting a review into words, each word is converted into a word embedding which, in our implementation, is a vector of size 50. Word embeddings are useful because they provide a numerical way to understand an english word.  For instance the first dimension could identify if the word is a noun, verb, adjective or other grammatical structure, the second dimension could explain whether the word is singular or plural, and the rest of the dimensions can go into more detail about the meaning of the word, and its usage with other words. Word embeddings are useful for upstream machine learning tasks that need to understand what words mean to help them perform a task. The implementation uses pre-trained word embeddings because training word embeddings takes a long time and large corpuses of text to do. The word embeddings that have been downloaded and used for the LSTM are called GloVe word embeddings; GloVe is an embedding technique that uses an unsupervised learning algorithm to obtain vector representations for english words. The GloVe embeddings are loaded from a text file which contains the word followed by a space, followed by 50 numbers representing the embedding. These embeddings are general-purpose so they can be loaded into R using the same text file. 

By tokenizing a review into words, and then converting the words into 50 dimensional embeddings, the LSTM can process the embeddings sequentially, to try to understand the semantics of the review, and determine if the review is fake or not. To be more specific, the embeddings are processed from the first word, up to the 90th word, and the LSTM outputs a result for each word that is processed; the result outputted after processing each word is called a hidden state. In addition to outputting 90 hidden states, the LSTM holds a latent representation of what is being read in its cell state that is a 20 dimensional vector. The cell state is modified as each word is read, so that the LSTM remembers the words it has seen previously to assist in processing the next word. The name recurrent neural network comes from the fact that the cell state is being recycled as it processes a sequence of input, acting as a sort of memory for the LSTM to understand the meaning behind a sequence. After processing the 90th word, we take the output of that word and pass it through a feed forward neural networmplemented k, and a softmax layer (for normalizing the output of the feed forward layer so that it sums to 1, allowing one to interpret the output as a probability distribution) to get 2 dimensional output, where the first dimension describes the probability of the review being real, while the second one describes the probability of it being fake. The final hidden state after reading the 90th word is used to derive whether the review is real or fake, while all the other outputs are discarded. The feed forward  network’s responsibility is to interpret the final output into the 2 dimensions required. 

A limitation of the above approach is that only 90 words are used per review for training the LSTM (which is the average length of a review in the dataset). There is a limit on the sequence length to stabilize the learning process for the LSTM; instead of training the LSTM on one example at a time, multiple examples are used so that an average error signal can used for learning to reduce the effect of “outlier” error signals, and to process multiple examples, all the examples have to have the same sequence length. Sequences that are too long are truncated while sequences that are too short are padded by <pad> tokens which LSTMs will interpret as padding. 
Training on the starting 90 words can reduce prediction power because we lose information on the words used at the end of the review that could have particular importance for determining whether a review is fake (such as talking about how the product or service can be improved which is usually said at the end). Another limitation is misspelling of words, which is often done in online reviews. Misspelling causes the Out-of-Vocabulary (OOV) embedding to be used in place of the embedding for the correctly spelled word, which is meaningless to the LSTM, except for indicating that an OOV word was processed which could be due to a misspelling by the reviewer. A third limitation is that tokenizing the review into words, and then converting the review into embeddings results in loss of information such as the punctuation that is used between words, and the capitalization used within words. Punctuation and capitalization are important features to determine if a review is fake. A fourth limitation is that training any deep learning solution requires a lot of data due to the nature of the learning algorithm and labeled fake review data being hard to come by. 

Some of the limitations were addressed by feeding in the number of capital letters in the review, and the number of certain types of punctuation such as exclamations into the feed forward layer as additional features to augment the LSTM’s interpretation of the review. However, the LSTM solution still only had an accuracy of 68.48% on the test set, indicating the above limitations greatly reduced the LSTM’s predictive power.

The following confusion matrix on a class-balanced test set shows that there is a considerable number of false positives and false negatives, and that the model is better at identifying fake reviews versus identifying real reviews. Additionally, the LSTM has more false negatives than false positives. This is inline with the measured precision and recall which are 0.7 and 0.659 respectively, indicating that the LSTM rarely assigns a sample as real (reducing false positives), and if it does, it is precise and the sample is likely real, however, due to this behaviour, the recall suffers because the LSTM is missing many positive cases. 


![Image description](https://github.com/harman666666/Fake-Ecommerce-Reviews-Detector/blob/master/LSTMConfusionMatrix.png)

Figure 11: Confusion Matrix for Real and Fake Reviews using LSTM


In the report below, one can see that the recall is greater than the precision if one considers fake reviews as the positive, which makes sense since the LSTM is usually assigning samples negative (since it rarely assigns positive), reducing the precision, and increasing the recall for fake reviews. 

Figure 12: Classification Results for Real and Fake Reviews using LSTM
![Image description](https://github.com/harman666666/Fake-Ecommerce-Reviews-Detector/blob/master/classification_results.png)



