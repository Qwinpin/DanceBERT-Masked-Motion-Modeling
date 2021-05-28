# DanceBERT: Masked Motion Modeling

Cross-modality architecture based on BERT neural networks. Training is performs like in original BERT: motion sequences represented as an "words", we mask these words with some probability and train neural network to predict them.

## WORK IN-PROGRESS: results are not yet impressive



# DATA

We use AIST++ [

[link]: https://github.com/google/aistplusplus_api

] dataset, which provides high quality 3d-motions of different dancers, styles, etc. For the input we use motions and music to build cross-modality model.

Place motions and splits folder into data. Download music data from AIST dataset and place into data/music folder.

# Fine-Tuning

After MMM pre-training we fine-tune model to recover long sequence of missing motions (Notebook is not yet ready).

### Provide each 15 frame on input and recover others (origin on the left):

<video src="/home/gito/github/MotionBERT: Masked Motion Modeling/images/30.mp4"></video>



### Provide each 30 frame on input and recover others:

<video src="images/15.mp4"></video>



