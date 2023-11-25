# Towards Accurate Lyrics Recognition For Rap Songs
Transcibe rap lyrics using wav2vec2.0 with CTC loss by transfer learning.

Our code is inherited from this [blog](https://huggingface.co/blog/fine-tune-wav2vec2-english) on finetuning Wav2Vec 2.0. The dataset is stored on [Google Drive](https://drive.google.com/drive/folders/1xfjYZQpOdcx-zJK19zY5sznBDyHHSMZE).

# Dataset
We download audio and lyrics files of 187 English rap songs from a playlist on [QQMusic](https://c6.y.qq.com/base/fcgi-bin/u?__=AdOgRqZ)) featuring different rap artists. For simplicity, if you would like to reproduce the result using our dataset, you can refer to the [dataset_clean folder](https://drive.google.com/drive/folders/1Cf6u-PFFzx5sveSPOBrhMYASKRE8fIRu?usp=share_link) directedly for processed data instead of raw data.

## Prepare Audiofolder for loading(https://github.com/xinyueli2896/raptranscription/blob/main/loader_prepare.ipynb)
Segment the each audio file into several chunks with sample rate 44000Hz in alignment with the timestamps in the .flac files containing the lyrics. Then we store the path to our audio files and their "labels" (lyrics) in a .csv file to be used for loading as an audiofolder later.
Store mp3 files into train and test subfolders under the main audiofolder and upload metadata.csv under the main directory as well.
The dataset directory should look like this:

**my_dataset/**<br>
├── README.md/<br>
├── metadata.csv/<br>
└── data/

## [Clean up my Audiofolder](https://github.com/xinyueli2896/raptranscription/blob/main/loader_clean.ipynb)
1. Filter out replicated and corrupted audio files
2. Unmask explicit vocabulary with [dirty map](https://github.com/xinyueli2896/RapRec/blob/main/unmask.json) generated by [this code](https://github.com/xinyueli2896/RapRec/blob/main/dirty_map.py).

## [Usage of the Audiofolder](https://github.com/xinyueli2896/raptranscription/blob/main/dataloader.ipynb)
Load data with load_dataset from huggingface, specifying 'audiofolder' and path to the dataset in the argument. More information see [documentation](https://huggingface.co/docs/datasets/audio_dataset#audiofolder).

## [Source Separation of our dataset](https://github.com/xinyueli2896/RapRec/blob/main/Hybrid_Demucs_Music_Source_Separation.ipynb)
For comparison purposes, we have also created a second dataset with all the clips source-separated by demucs before feeding them to the wav2vec model. The dataset is stored [here](https://drive.google.com/drive/folders/1tTI-i4O_8wctOIBUfM8za8Ec-R5eZ2Vz?usp=share_link).

# Architecture
## Wav2Vec 2.0 
After we completed the dataset curation, we employed a pretrained [Wav2Vec 2.0](https://arxiv.org/abs/2006.11477) ASR model and further fine-tuning it on our dataset. We chose this architecture as it yields performance comparable to the best ASR systems available by learning robust speech representations through minimal fine-tuning on labelled speech data. The architecture has three modules: feature encoder(convolutional neural networks to process raw audio waveforms and transform them into latent speech representations), context model (transformer based model that learns contextually rich representations), and quantization model (that discretizes feature encoder output to be targets for the semi-supervised objective).

## Connectionist Temporal Classification
During fine-tuning, we freezed the feature encoder since it is learing basic speech representation. Therefore, we are only training the Transformer model with our labeled data. Because of the nature of speech as a sequence modeling task, the alignment of raw audio input and corresponding text output is not inherently clear. To overcome this challenge, the training is conducted using [Connectionist Temporal Classification (CTC)](https://distill.pub/2017/ctc/) algorithm.

The CTC algorithm introduces a special ’blank’ character to the output vocabu-
lary, allowing for flexible alignments through repetition or omission of characters. Mathematically,
the CTC loss is formulated as the negative log likelihood of the correct label sequence given an input sequence. Given an input sequence X = (x1, x2, ..., xT ) and a target sequence Y = (y1, y2, ..., yU ), where T and U are the lengths of the input and target sequences respectively, the CTC loss is defined as:
LCTC(X, Y ) = − log P (Y |X) = − log ∑ π∈A(Y,T ) P (π|X)

Here, A(Y, T ) represents the set of all valid alignments of Y to a sequence of length T using the
CTC blank symbol, and P (π|X) denotes the probability of a particular alignment π. The CTC loss is
minimized during training. This approach is particularly effective for rap lyrics transcription, where the rhythmic and fast-paced nature of the content poses unique alignment challenges
# Training 


