# Mellotron and Tacotron 2 API

API for [Mellotron](https://github.com/NVIDIA/mellotron) and [Tacotron 2](https://github.com/NVIDIA/tacotron2).
This repository contains an installation guide and some utility functions to simplify the access to the models in the repositories.
All credits go to the original developers.

## Repository structure

This repository is organised into two main directories:

- `resources/` contains:
    - directory to host the Mellotron and Tacotron 2 models;
    - directory to host the WaveGlow model.
- `src/mellotron_api` package with the api.
- `mellotron/` submodule with Mellotron and WaveGlow code.
- `tacotron2/` submodule with Tacotron 2 code.
- `tmp/` contains temporary replacement files to have the code work properly (see environment section).

For further details on the available models, refer to the `README.md` in the `resources/` directory.

## Environment

To install all the required packages within an anaconda environment and do a complete setup, run the following commands:

```bash
# Create anaconda environment (skip cudatoolkit option if you don't want to use the GPU)
conda create -n ttsmellotron python=3.10 cudatoolkit=11.3
# Activate anaconda environment
conda activate ttsmellotron
# Install packages
conda install pytorch=1.11.0 -c pytorch
conda install -c conda-forge scipy matplotlib librosa tensorflow music21 inflect tensorboard tensorboardx unidecode
conda install -c anaconda nltk pillow
pip install jamo
# Download and initialise submodules
# Mellotron and Tacotron 2
git submodule init; git submodule update
# WaveGlow inside Mellotron
cd mellotron
git submodule init; git submodule update
cd ..
# Update code using old TensorFlow version with now deprecated interfaces
cp tmp/updated_mellotron_utils.py mellotron/utils.py
cp tmp/updated_mellotron_text_package.py mellotron/text/__init__.py
cp tmp/updated_mellotron_hparams.py mellotron/hparams.py
cp tmp/updated_mellotron_model.py mellotron/model.py
cp tmp/updated_denoiser.py mellotron/waveglow/denoiser.py
cp tmp/updated_glow.py mellotron/waveglow/glow.py
cp tmp/updated_tacotron2_train.py tacotron2/train.py
cp tmp/updated_tacotron2_text_package.py tacotron2/text/__init__.py
cp tmp/updated_tacotron2_hparams.py tacotron2/hparams.py
cp tmp/updated_tacotron2_model.py tacotron2/model.py
```

To add the directories to the Python path, you can add these lines to the file `~/.bashrc`

```bash
export PYTHONPATH=$PYTHONPATH:/path/to/tts_mellotron_api/src
export PYTHONPATH=$PYTHONPATH:/path/to/tts_mellotron_api/mellotron
export PYTHONPATH=$PYTHONPATH:/path/to/tts_mellotron_api/mellotron/waveglow
export PYTHONPATH=$PYTHONPATH:/path/to/tts_mellotron_api/tacotron2
```

## Examples

Here follows some usage examples of Tacotron 2 and Mellotron.

### Load models

Start by loading the models.
Mellotron requires also to load the ARPAbet dictionary.
Both models can work with and without the Vocoder, in the latter case the [Griffin-Limm algorithm](https://paperswithcode.com/method/griffin-lim-algorithm) is used to generate the raw waveform from the Mel spectrogram.

```python
from mellotron_api import load_tts, load_vocoder, load_arpabet_dict, synthesise_speech


mellotron, mellotron_stft, mellotron_hparams = load_tts('resources/tts/mellotron/mellotron_libritts.pt')
tacotron2, tacotron2_stft, tacotron2_hparams = load_tts('resources/tts/tacotron_2/tacotron2_statedict.pt', model='tacotron2')
waveglow, denoiser = load_vocoder('resources/vocoder/waveglow/waveglow_256channels_universal_v4.pt')
arpabet_dict = load_arpabet_dict('mellotron/data/cmu_dictionary')
```

### Synthesise with Tacotron 2

Tacotron 2 allows synthesising speech directly from raw text input and nothing else.

```python
synthesise_speech(
    "I am testing a neural network for speech synthesis.", 
    tacotron2,
    tacotron2_hparams,
    tacotron2_stft,
    arpabet_dict=arpabet_dict,
    waveglow=waveglow,
    denoiser=denoiser,
    out_path='path/to/output.wav'
)
```

### Synthesise with Mellotron

In this API there two available synthesis modalities for Mellotron:
- with reference audio clip
- without reference audio clip

#### With reference audio

Mellotron works using a reference audio for sythesis that provides the GST and the pitch (F0).

```python
audio_path = 'path/to/audio.wav'

synthesise_speech(
    "I am testing a neural network for speech synthesis.", 
    mellotron,
    mellotron_hparams,
    mellotron_stft,
    arpabet_dict=arpabet_dict,
    waveglow=waveglow,
    denoiser=denoiser,
    reference_audio_path=audio_path,
    out_path='path/to/output.wav'
)
```

#### Without reference audio

Using Tacotron 2 to generate a reference Mel Spectrogram, it is possible to use Mellotron GST and speaker conditionings without a reference audio.

```python
synthesise_speech(
    "I am testing a neural network for speech synthesis.", 
    mellotron,
    mellotron_hparams,
    mellotron_stft,
    arpabet_dict=arpabet_dict,
    waveglow=waveglow,
    denoiser=denoiser,
    tacotron2=tacotron2,
    tacotron2_stft=tacotron2_stft,
    tacotron2_hparams=tacotron2_hparams,
    out_path='path/to/output.wav'
)
```
