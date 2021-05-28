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

https://user-images.githubusercontent.com/31972765/119990634-3ac52200-bfd1-11eb-83c3-a8db9d55f927.mp4

### Provide each 30 frame on input and recover others:

https://user-images.githubusercontent.com/31972765/119990453-03ef0c00-bfd1-11eb-9d81-e89796f56c4c.mp4
