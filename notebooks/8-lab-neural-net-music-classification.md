---
title: 'Neural networks for music classification'
author: 'Fraida Fund'
jupyter:
  anaconda-cloud: {}
  colab:
  kernelspec:
    display_name: Python 3
    language: python
    name: python3
  nbformat: 4
  nbformat_minor: 0
---

::: {.cell .markdown }
# Assignment: Neural Networks for Music Classification

_Fraida Fund_

:::

::: {.cell .markdown }
**TODO**: Edit this cell to fill in your NYU Net ID and your name:

-   **Net ID**:
-   **Name**:
:::

::: {.cell .markdown }
In this assignment, we will look at an audio classification problem.
Given a sample of music, we want to determine which instrument (e.g.
trumpet, violin, piano) is playing.

*This assignment is closely based on one by Sundeep Rangan, from his
[IntroML GitHub repo](https://github.com/sdrangan/introml/).*
:::

::: {.cell .code }
```python
import tensorflow as tf
import numpy as np
import matplotlib
import matplotlib.pyplot as plt
import seaborn as sns
%matplotlib inline
```
:::

::: {.cell .markdown }
## Audio Feature Extraction with Librosa

The key to audio classification is to extract the correct features. The
`librosa` package in python has a rich set of methods for extracting the
features of audio samples commonly used in machine learning tasks, such
as speech recognition and sound classification.
:::

::: {.cell .code }
```python
import librosa
import librosa.display
import librosa.feature
```
:::

::: {.cell .markdown }
In this lab, we will use a set of music samples from the website:

<http://theremin.music.uiowa.edu>

This website has a great set of samples for audio processing.

We will use the `wget` command to retrieve one file to our Google Colab
storage area. (We can run `wget` and many other basic Linux commands in
Colab by prefixing them with a `!` or `%`.)
:::

::: {.cell .code }
```python
!wget "http://theremin.music.uiowa.edu/sound files/MIS/Woodwinds/sopranosaxophone/SopSax.Vib.pp.C6Eb6.aiff"
```
:::

::: {.cell .markdown }
Now, if you click on the small folder icon on the far left of the Colab
interface, you can see the files in your Colab storage. You should see
the "SopSax.Vib.pp.C6Eb6.aiff" file appear there.
:::

::: {.cell .markdown }
In order to listen to this file, we'll first convert it into the `wav`
format. Again, we'll use a magic command to run a basic command-line
utility: `ffmpeg`, a powerful tool for working with audio and video
files.
:::

::: {.cell .code }
```python
aiff_file = 'SopSax.Vib.pp.C6Eb6.aiff'
wav_file = 'SopSax.Vib.pp.C6Eb6.wav'

!ffmpeg -y -i $aiff_file $wav_file
```
:::

::: {.cell .markdown }
Now, we can play the file directly from Colab. If you press the ▶️
button, you will hear a soprano saxaphone (with vibrato) playing four
notes (C, C\#, D, Eb).
:::

::: {.cell .code }
```python
import IPython.display as ipd
ipd.Audio(wav_file) 
```
:::

::: {.cell .markdown }
Next, use `librosa` command `librosa.load` to read the audio file with
filename `audio_file` and get the samples `y` and sample rate `sr`.
:::

::: {.cell .code }
```python
y, sr = librosa.load(aiff_file)
```
:::

::: {.cell .markdown }
Feature engineering from audio files is an entire subject in its own
right. A commonly used set of features are called the Mel Frequency
Cepstral Coefficients (MFCCs). These are derived from the so-called mel
spectrogram, which is something like a regular spectrogram, but the
power and frequency are represented in log scale, which more naturally
aligns with human perceptual processing.

You can run the code below to display the mel spectrogram from the audio
sample.

You can easily see the four notes played in the audio track. You also
see the \'harmonics\' of each notes, which are other tones at integer
multiples of the fundamental frequency of each note.
:::

::: {.cell .code }
```python
S = librosa.feature.melspectrogram(y=y, sr=sr, n_mels=128, fmax=8000)
librosa.display.specshow(librosa.amplitude_to_db(S),
                         y_axis='mel', fmax=8000, x_axis='time')
plt.colorbar(format='%+2.0f dB')
plt.title('Mel spectrogram')
plt.tight_layout()
```
:::

::: {.cell .markdown }
## Downloading the Data

Using the MFCC features described above, [Prof. Juan
Bello](http://steinhardt.nyu.edu/faculty/Juan_Pablo_Bello) at NYU
Steinhardt and his former PhD student Eric Humphrey have created a
complete data set that can used for instrument classification.
Essentially, they collected a number of data files from the website
above. For each audio file, the segmented the track into notes and then
extracted 120 MFCCs for each note. The goal is to recognize the
instrument from the 120 MFCCs. The process of feature extraction is
quite involved. So, we will just use their processed data.
:::

::: {.cell .markdown }
To retrieve their data, visit

<https://github.com/marl/dl4mir-tutorial/blob/master/README.md>

and note the password listed on that page. Click on the link for
"Instrument Dataset", enter the password, click on
`instrument_dataset` to open the folder, and download the four files
there. (You can "direct download" straight from this site, you don't
need a Dropbox account.)

Then, upload the files to your Google Colab storage: click on the folder
icon on the left to see your storage, if it isn't already open, and
then click on "Upload".

🛑 Wait until *all* uploads have completed and the orange "circles"
indicating uploads in progress are *gone*. (The training data especially
will take some time to upload.) 🛑
:::

::: {.cell .markdown }
Then, load the files with:
:::

::: {.cell .code }
```python
Xtr = np.load('uiowa_train_data.npy')
ytr = np.load('uiowa_train_labels.npy')
Xts = np.load('uiowa_test_data.npy')
yts = np.load('uiowa_test_labels.npy')
```
:::

::: {.cell .markdown }
Examine the data you have just loaded in:

-   How many training samples are there?
-   How many test samples are there?
-   What is the number of features for each sample?
-   How many classes (i.e. instruments) are there?

Write some code to find these values and print them.
:::

::: {.cell .code }
```python
# TODO -  get basic details of the data
# compute these values from the data, don't hard-code them
n_tr    = ...
n_ts    = ...
n_feat  = ...
n_class = ...
```
:::


::: {.cell .code }
```python
# now print those details
print("Num training= %d" % n_tr)
print("Num test=     %d" % n_ts)
print("Num features= %d" % n_feat)
print("Num classes=  %d" % n_class)
```
:::
::: {.cell .markdown }
Then, standardize the training and test data, `Xtr` and `Xts`, by
removing the mean of each feature and scaling to unit variance.

You can do this manually, or using `sklearn`\'s
[StandardScaler](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.StandardScaler.html).
(For an example showing how to use a `StandardScaler`, you can refer to
the notebook on regularization.)

Although you will scale both the training and test data, you should make sure that 
both are scaled according to the mean and variance statistics from the
*training data only*.

`<small>`{=html}Standardizing the input data can make the gradient
descent work better, by making the loss function "easier" to
descend.`</small>`{=html}
:::

::: {.cell .code }
```python
# TODO - Standardize the training and test data
Xtr_scale = ...
Xts_scale = ...

```
:::

::: {.cell .markdown collapsed="true" }
## Building a Neural Network Classifier

Following the example in the demos you have seen, clear the keras
session. Then, create a neural network `model` with:

-   `nh=256` hidden units in a single dense hidden layer
-   `sigmoid` activation at hidden units
-   select the input and output shapes, and output activation, according
    to the problem requirements. Use the variables you defined earlier 
    (`n_tr`, `n_ts`, `n_feat`, `n_class`) as applicable, rather than 
    hard-coding numbers.


Print the model summary.

:::

::: {.cell .code }
```python
from tensorflow.keras.models import Model, Sequential
from tensorflow.keras.layers import Dense, Activation
from tensorflow.keras import optimizers
import tensorflow.keras.backend as K
```
:::




::: {.cell .code }
```python
# TODO - construct the model
nh = 256
# model =  ...
# model.add( ...
 
```
:::

::: {.cell .code}
```python
# show the model summary
model.summary()
```
:::


::: {.cell .code}
```python
# you can also visualize the model with 
tf.keras.utils.plot_model(model, show_shapes=True)
```
:::


::: {.cell .markdown }
Create an optimizer and compile the model. Select the appropriate loss
function for this multi-class classification problem, and use an
accuracy metric. For the optimizer, use the Adam optimizer with a
learning rate of 0.001
:::

::: {.cell .code }
```python
# TODO - create optimizer and compile the model
# opt = ...
# model.compile(...)
```
:::

::: {.cell .markdown }
Fit the model for 10 epochs using the scaled data for both training
and validation, and save the training history in `hist.

Use the `validation_data` option to pass the *test*
data. (This is OK because we are not going to use this data
as part of the training process, such as for early stopping - 
we're just going to compute the accuracy on the data so that 
we can see how training and test loss changes as the model is 
trained.)


Use a batch size of 128. Your final accuracy should be greater than 99%.
:::

::: {.cell .code }
```python
# TODO - fit model and save training history
# hist = 
```
:::

::: {.cell .markdown }
Plot the training and validation accuracy saved in `hist.history`
dictionary, on the same plot. This gives one accuracy value per epoch.
You should see that the validation accuracy saturates around 99%. After
that it may "bounce around" a little due to the noise in the
stochastic mini-batch gradient descent.

Make sure to label each axis, and each series (training vs. validation/test).
:::

::: {.cell .code }
```python
# TODO - plot the training and validation accuracy in one plot
```
:::

::: {.cell .markdown }
Plot the training and validation loss values saved in the `hist.history`
dictionary, on the same plot. You should see that the training loss is
steadily decreasing. Use the [`semilogy`
plot](https://matplotlib.org/stable/api/_as_gen/matplotlib.pyplot.semilogy.html)
so that the y-axis is log scale.

Make sure to label each axis, and each series (training vs. validation/test).
:::

::: {.cell .code }
```python
# TODO - plot the training and validation loss in one plot
```
:::

::: {.cell .markdown }
## Varying the Learning Rate

One challenge in training neural networks is the selection of the
learning rate. Repeat your model preparation and fitting code, but try
four learning rates as shown in the vector `rates`. In each iteration of
the loop:

-   use `K.clear_session()` to free up memory from
    models that are no longer in scope
-   construct the network
-   select the optimizer. Use the Adam optimizer with the learning rate
    specific to this iteration
-   train the model for 20 epochs
-   save the history of training and validation accuracy and loss for
    this model
:::

::: {.cell .code }
```python
rates = [0.1, 0.01,0.001,0.0001]

# TODO - iterate over learning rates
```
:::

::: {.cell .markdown }
Plot the training loss vs. the epoch number for all of the learning
rates on one graph (use `semilogy` again). You should see that the lower
learning rates are more stable, but converge slower, while with a
learning rate that is too high, the gradient descent may fail to move
towards weights that decrease the loss function.

Make sure to label each axis, and each series.

**Comment on the results.** What is the effect of learning rate on the training process?

:::

::: {.cell .code }
```python
# TODO - plot showing the training process for different learning rates
```
:::
