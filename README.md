Code and configuration files for training a YourTTS model with a [SpeechBrain](https://speechbrain.github.io/)-based speaker encoder, like in the TSD 2025 paper ["An Exploration of ECAPA-TDNN and x-vector Speaker Representations in Zero-shot Multi-speaker TTS"](https://doi.org/10.1007/978-3-032-02548-7_8) (arXiv version [here](https://arxiv.org/abs/2506.20190)).

Based on an older version of [jmaty/Coqui-TTS](https://github.com/jmaty/Coqui-TTS) (from November 2, 2023, [this](https://github.com/jmaty/Coqui-TTS/tree/d5376545392fddfb3705663fe02c2dd861408df3) revision), which is itself a slightly modified fork of [coqui-ai/TTS](https://github.com/coqui-ai/TTS).

**Warning**: most of this was made in 2023 / early 2024, using old versions of Coqui-TTS (see above) and SpeechBrain (0.5.16). It may not be compatible with the latest SpeechBrain version (not tested), though the required changes should be fairly minimal. 

For anyone trying to use this, I would recommend just looking at the relevant changes (see [the September 5, 2025 commit](https://github.com/mkunes/Coqui-TTS-with-alternate-speaker-encoder/commit/22fb6f16a33647ad21fbe347cca442c48541948a)) and applying them to your own version of [coqui-ai/TTS](https://github.com/coqui-ai/TTS) (which should basically just mean adding two new files). The differences between the original [coqui-ai/TTS](https://github.com/coqui-ai/TTS) and the [jmaty/Coqui-TTS](https://github.com/jmaty/Coqui-TTS) fork are probably not relevant for most other people.


### Changed code (compared to the jmaty fork):

Unless I'm mistaken, there were no relevant changes to any existing files.

Only two files were added:  

- `TTS/tts/models/vitsecapa.py`
- `TTS/tts/configs/vitsecapa_config.py`

-> these define a new TTS model that is based on `TTS/tts/models/vits.py`, but uses a SpeechBrain speaker encoder 

### Speaker embedding models used:
- ECAPA-TDNN: https://github.com/speechbrain/speechbrain/tree/develop/recipes/VoxCeleb/SpeakerRec; Model "ECAPA-TDNN  VoxCeleb 1,2" (https://www.dropbox.com/sh/ab1ma1lnmskedo8/AADsmgOLPdEjSF6wV3KyhNG1a?dl=0)
- x-vector: https://huggingface.co/speechbrain/spkrec-xvect-voxceleb

### Before training:

- extract speaker embeddings from the training data, save them in a JSON file  
  example: `YourTTS_alternate_speaker_encoder/spk_embedding_file_example.json`

- manually create a JSON config file for the chosen SpeechBrain model, in the format expected by YourTTS  
  examples: `YourTTS_alternate_speaker_encoder/ECAPA-TDNN_config_handmade.json`, `YourTTS_alternate_speaker_encoder/xvec_config_handmade.json`

### Training:

Jupyter Notebook: `YourTTS_alternate_speaker_encoder/E2E_YourTTS_train_SCL-ECAPA_v2.ipynb` + custom config file, injected via `papermill` (https://pypi.org/project/papermill/):

- config used for "ECAPA-TDNN TTS": `YourTTS_alternate_speaker_encoder/YourTTS_SPT-MGW-JMa_ECAPA_VoxCelebModel_alwaysSCL_noMixed_16kEmbeds.yaml`
- config used for "x-vector TTS":   `YourTTS_alternate_speaker_encoder/YourTTS_SPT-MGW-JMa_xvec_VoxCelebModel_alwaysSCL_noMixed_16kEmbeds.yaml`

the most relevant settings:

- `"use_d_vector_file": true  # use precomputed external embeddings (otherwise YourTTS will try to train its own encoder?)`
- `"d_vector_file": ["path/to/embeddings.json"]  # path to the precomputed embeddings of the training data (e.g. spk_embedding_file_example.json)`
- `"d_vector_dim": 192  # dimension of the speaker embeddings (192 for ECAPA-TDNN, 512 for x-vectors)`
- `"use_speaker_encoder_as_loss": true  # enables speaker consistency loss (SCL)`
- `"speaker_encoder_config_path": "path/to/ECAPA-TDNN_config_handmade.json" # path to the manually created config file for the SpeechBrain speaker embedding model`
- `"speaker_encoder_model_path": "/path/to/speechbrain/model/directory" # path to the SpeechBrain model (directory or HuggingFace path)`
        
### Speech synthesis:

`YourTTS_alternate_speaker_encoder/E2E_YourTTS_synthesize_TSD2025.ipynb`  (normal Jupyter Notebook, no papermill required)
