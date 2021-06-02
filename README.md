

Cross-modality architecture based on BERT neural networks. Training is performs like in original BERT: motion sequences represented as an "words", we mask these words with some probability and train neural network to predict them.

## WORK IN-PROGRESS: results are not yet impressive



# DATA

## Pre-requirements

You need to install provided dependencies:

`pip install -r requirements.txt`

Also, if you want to work with [Ubisoft La Forge](https://github.com/ubisoft/ubisoft-laforge-animation-dataset) data:

```
cd utils
git clone https://github.com/omimo/PyMO.git
```

[PyMO](https://github.com/omimo/PyMO) framework is used to process data and generate samples of generated motions.

## Dance data

We use [AIST++](https://github.com/google/aistplusplus_api) dataset, which provides high quality 3d-motions of different dancers, styles, etc. For the input we use motions and music to build cross-modality model.

Place motions and splits folder into data. Download music data from AIST dataset and place into data/music folder.

Data processing is provided in PoseDataPrepare.ipynb notebook.

Result of processing: 300-frame sequences of motion and music. Motion represented as rotation matrix in shape (300, 219), there first 216 features - smpl-body shapes (you can reshape them into 24, 3, 3) and the last 3 -  global coordinates. Music represented as a set of extracted features (beats, e.g.).

## Common motion

Ubisoft La Forge provide data of different type of motion (without music, ofc). Put lafan1 data (.bvh files) into data/lafan1.

Data processing is provided in PoseDataPrepare.Ubisoft.ipynb.

Result of processing: 300-frame sequences of motion. Motion represented as rotation matrix in shape (300, 201), there first 198 features - motion of body

[^]: The order: LeftUpLeg, LeftLeg, LeftFoot, LeftToe, RightUpLeg, RightLeg, RightFoot, RightToe, Spine, Spine1, Spine2, Neck, Head, LeftShoulder, LeftArm, LeftForeArm, LeftHand, RightShoulder, RightArm, RightForeArm, RightHand

 and last 3 - global coordinates.

# MMM pretraining

## Dance

Dance data consists of motions and aligned music in 60 fps. The process of masked modeling:

1. Randomly select the length of sequence (during preprocessing we split each sequence into 300 frames) from 120 to 300 frames.
2. Select some frames of motion with 0.15 probability
3. Select type of masking: zeroing (0.8), replace with random (0.1), keep the same (0.1).
4. Note: music frames remains unchanged. We suppose them as some meta information for the model
5. Batch size: 32, Adam optimizer (0.9, 0.999), MSE Loss
6. Training process for 3BERT:
   1. 43000 steps with 1e-4 learning rate
7. Training process for EnforceCrossModality:
   1. 53000 steps with 1e-4 lr
   2. 16000 steps with 1e-5 lr
   3. 69000 steps with 1e-6 lr

The goal of training - to recover masked frames of motion.

### Architecture

Both in BERT.KeyFramesFilling.3BERT and BERT.KeyFramesFilling.3BERT.EnforceCrossModality we have two encoders for motion and dance. The difference is in the third encoder - cross-modal transformer:

1. For 3BERT we concatenate output of motion and music encoders along the feature dimension and the resulting shape is (batch_size, sequence_length, hidden_size * 2).
2. EnforceCrossModality assumes that we fed motion and music concatenated along sequence length dimension - (batch_size, motion_sequence_length + music_sequence_length, hidden_size). The idea is to allow attentions of this transformer analyze the connections between two different modalities. We just slice the output to required length.
3. Note: while AIST++ paper suggests to use 2-layers motion and music encoders and 12-layers cross-modal encoder for the generation, in our task of masked modeling we had no success with such configuration. We use 5-layers encoders and 10-layers cross-modal encoder for EnforceCrossModal and 5-layers cross-modal for 3BERT.
4. Encoders are ConvBERT architecture provided in hugging-face transformer framework.
5. The output of cross-modal encoder is fed into linear layer with Tanh activation function.

## Common motions

Ubisoft La Forge data consists of motions in 30 fps. The process of masked modeling is the same, but due to lack of music information (It doesn't have to be, it's just movement), architecture is differ.

Training process:

1. Batch size 128, Adam optimizer (0.9, 0.999), MSE Loss
2. 20000 steps with 1e-4 lr
3. 20000 steps with 1e-5 lr

### Architecture

1. We also use ConvBERT with 10-layers and 8 attention heads.
2. The output of encoder is fed into:
   1. 1d ConvLayer with kernel size 3 and padding 1
   2. 1d ConvLayer with kernel size 1
   3. LayerNorm layers with (300, 201) trainable parameters

# Fixed pattern fine-tuning

After MMM pre-training we train models for the task of in-between filling of frames.

Batch size 32, Adam optimizer, MSE Loss

## Dance

3BERT training process:

1. 1100 steps with each 5 frame, 1e-4 lr
2. 2000 steps with each 10 frame, 1e-4 lr
3. 2000 steps with each 15 frame, 1e-5 lr
4. 40000 steps with each 15 frame, 1e-5 lr
5. 100000 steps with each 20 frame, 1e-6 lr

# Future motion generation fine-tuning

TODO

# Results: fixed pattern fine-tuning

## 3BERT

We fed into model each 15 frame (intermediate ones are zeroed out).

<video src="/home/gito/Videos/Peek 2021-06-01 14-54.mp4"></video>

# Results: future motion generation fine-tuning

## EnforceCrossModal

We provide first 120 frames of real motion and them perform auto-regressive generation.

First 2 second of this video is the real dance. Results are not impressive yet.

<video src="/tmp/0001-0750.mp4"></video>