Title: Some thoughts after Interspeech 2017
Published: 25/8/2017
Tags: 
    - interspeech
    - speech technologies
    - automatic speech recognition
    - deep learning
    - machine learning
---


## Introduction

I've just returned from the Interspeech 2017 conference in Stockholm. I've had a great time there, getting to know other researchers and engineers working on different aspects of speech. Since there were many tracks conducted in parallel, I'm pretty sure that I've missed many interesting presentations, but I tried my best to move around quickly and absorb as much knowledge as I could. Here's some of the papers/posters/presentations that I especially liked.

All of the papers mentioned here (and many more) can be found at the [ISCA archive webiste](http://www.isca-speech.org/archive/Interspeech_2017).

_Edit: I've corrected some typos and added 2 more papers which weren't in my notes but I remembered them later._

## Automatic Speech Recognition

**Rescoring-aware Beam Search for Reduced Search Errors in Contextual Automatic Speech Recognition** - the authors investigated the problem of how to successfully utilise a relatively small rescoring language model (LM) based on the information about the speaker (e.g. words he added to his dictionary, geo-localization specific LMs, his activity in some mobile apps, etc.) during the decoding stage of ASR (on-the-fly rescoring). They were able to improve the word error rate (WER) for their use-case without significantly affecting the decoding speed.

**Towards better decoding and language model integration in sequence to sequence models** - this paper explored some of the problems of seq2seq attention based models used for end to end ASR - in particular, the over-confidence of the model and missing words in the recognition after application of a language model. From what I gathered, their methods could be useful in other tasks where similiar network architecture is ued.

**A Comparison of Sequence-to-Sequence Models for Speech Recognition** - this paper investigates the performance of several seq2seq architectures (CTC, RNN, RNN-transducer and attention-based) proposed in the recent years and finds out that the attention-based model performs the best, but still slightly worse than the baseline traditional system (with separate pronunciation lexicon and language model) depending on the dataset.

**Reducing the Computational Complexity of Two-Dimensional LSTMs** - personally, for me this seems a bit crazy, but the authors were able to achieve impressive results with some complex 2D LSTM architectures. A huge dataset (12000 hours of speech) apparently helped.

**Residual LSTM: Design of a Deep Recurrent Architecture for Distant Speech Recognition** - the authors investigated what is the optimal way to construct a quite deep (up to 10 layers) LSTM-based acoustic model. The best results were achieved by using residual connections, but before the output LSTM gate (instead of more intuitive after-gate connection). I really liked how Jaeyoung explained the reasons behind this, argumenting that the variance of the parameters would increase with each layer, effectively inhibiting model convergence.

**Predicting Automatic Speech Recognition Performance over Communication Channels from Instrumental Speech Quality and Intelligibility Scores** - I really like the attempt by the authors to predict how strongly will ASR performance be affected by signal degradation due to usage of a particular audio codec. Although they did not use a state-of-the-art ASR system architecture, I think it's a promising direction of investigation.

**Sequence-to-Sequence Models can Directly Translate Foreign Speech** - in this novel (and kind of crazy) approach, the authors trained a joint, end to end attention-based machine translation system to take input audio in Spanish (_Spanish ASR_) and output recognition as English text (_English translator_). Using this architecture, they were able to outperform combinations of separate, independently trained ASR and (text to text) MT systems.

## Language Models

**Fast Neural Network Language Model Lookups at N-Gram Speeds** - the novelty of the paper is that the authors addressed and solved a well-known problem that neural network LMs tend to be much slower than n-gram models and thus cannot be used during ASR decoding stage (and are used during rescoring stage instead). Although their methods are not applicable for recurrent models (RNNs), which tend to be significantly more effective, using an NNLM instead of an n-gram model should still yield a noticable improvement.

## The ASVspoof 2017 challenge

Along with my friends from the [Digital Signal Processing group at AGH-UST](http://dsp.agh.edu.pl), I've participated in the anti-spoofing challenge at this conference. The organisers provided authentic and replayed recordings of different people's utterances and the goal was to craft a spoofing detection system that would be able to tell the replayed (and recorded again) utterances from the authentic ones (which were recorded only once). The challenge was won by a team from the Speech Technology Center in ITMO University in Russia (paper **Audio Replay Attack Detection with Deep Learning Frameworks**). They beat every other team by a large margin (6~% Equal Error Rate [EER] compared to 11% at the second place) by utilising Convolutional Neural Networks (CNN) as feature extractors for a GMM classifier. Most of the other works focused on feature engineering - and so did our paper, where we discussed the reasons behind high- frequency features performing consistently better than low- and mid- frequency ones.

## Others

**Multi-Channel Apollo Mission Speech Transcript Calibration** - subjectively, this was the most impressive undertaking that I've seen at the conference. The authors have a task of converting 100.000 hours of non-transcribed speech into text - but it's not just any speech, it's the communications from the Apollo missions recorded on analog tapes. So far, they have 19k hours done. In the paper, they describe the design of 30-channel analog tape "digitizer" and their method for compensating noisy channels.

**An RNN Model of Text Normalization** - this paper touches a quite under-researched subject which is text normalization. It is particularly difficult for morphologically rich languages (such as Polish or Russian), which makes the presented results even more impressive - the authors were able to train a seq2seq model which treats text normalization as a machine translation task. These kinds of models tend to make quite obvious silly mistakes (e.g. "1 cm" is sometimes translated as "one volt" instead of "one centimeter"), but the authors proposed to use a grammar (in FST form) for postprocessing (which was automatically extracted from the data).

**Listening in the dips: Comparing relevant features for speech recognition in humans and machines** - the authors presented some very interesting results - basically, they trained a neural network based ASR on recordings mixed with modulated noise and they noticed that the ASR had basically the same transcription efficiency as human listeners. They also visualized the neuron activations and discovered that the neural network used almost exlusively the information in the "dips" (audio segments without noise), which is known to be the case with humans as well (they cited relevant previous works).

**Query-by-Example Search with Discriminative Neural Acoustic Word Embeddings** - I was impressed by the creativity of the authors, who addressed the problem of searching a large audio collection for utterances of a keyword based on an audio input (not a textual keyword). They trained "audio-segment-embeddings" and used an efficient variant of nearest-neighbor search algorithm.

**An Expanded Taxonomy of Semiotic Classes for Text Normalization** - another paper on the topic of text normalization (or should I say, _the other paper_), where the authors have shown a method for quickly gathering a set of problematic terms in text normalization. First they show a taxonomy, e.g. ordinal numbers (1st), cardinal numbers (12), "funny spelling" (_cul8r_), and many others. Then they claim that asking a few persons to fill a questionnaire in which they normalize these classes (_cul8r_ -> _see you later_) yields sufficient amount of data to automatically construct a translation grammar (FST),  which yields satisfactory results.

**Conditional Generative Adversarial Nets Classifier for Spoken Language Identification** - I liked this work because I think it's a very smart way to utilise the famous GANs in speech-related tasks - the authors use conditional GANs to generate artificial i-vectors for different languages and then train their classifier using augmented data. I'd really like to see more applications of GANs in the audio domain.

## Conclusions

The conference was very intensive and lasted four days. I tried my best to showcase some of the research, but by no objective means these papers could be considered "best" (actually, I think that I missed most of the awarded papers, so I'll have to catch up by reading them later). I hope that this small recap was helpful to you and inspired some deep learning ideas.
