# Magenta RT: Streaming music generation!

Magenta RealTime is a Python library for live music audio generation on your
local device. It is the open source / on device companion to
[MusicFX DJ Mode](https://labs.google/fx/tools/music-fx-dj) and the
[Lyria RealTime API](https://ai.google.dev/gemini-api/docs/music-generation).
Magenta RT allows for both [text](https://www.youtube.com/watch?v=Ae1Kz2zmh9M)
and [audio](https://www.youtube.com/watch?v=vHIf2UKXmp4) prompting.

See our
[blog post](https://g.co/magenta/rt),
[paper](https://arxiv.org/abs/2508.04651), and
[model card](https://github.com/magenta/magenta-realtime/blob/main/MODEL.md) for
more info.

![Animation of chunk-by-chunk generation in Magenta RT](notebooks/diagram.gif)

## Getting started

<a target="_blank" href="https://colab.research.google.com/github/magenta/magenta-realtime/blob/main/notebooks/Magenta_RT_Demo.ipynb">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open Notebook In Colab"/>
</a>

The fastest way to get started with Magenta RT is to try our official
[Colab Demo](https://colab.research.google.com/github/magenta/magenta-realtime/blob/main/notebooks/Magenta_RT_Demo.ipynb)
which runs in real time on freely available TPUs! Here is a quick
[video walkthrough](https://www.youtube.com/watch?v=SVTuEdeepVs).

**Update**: We now have two additional Colab demos supporting
[live audio input](https://colab.research.google.com/github/magenta/magenta-realtime/blob/main/notebooks/Magenta_RT_Audio_Injection.ipynb)
and [customization via finetuning](https://colab.research.google.com/github/magenta/magenta-realtime/blob/main/notebooks/Magenta_RT_Finetune.ipynb)!

If you have a machine with a TPU or GPU, you may also following the installation
instructions below for running locally.

## Local installation

Install the latest version:

```sh
# With GPU support:
pip install 'git+https://github.com/magenta/magenta-realtime#egg=magenta_rt[gpu]'
# With TPU support:
pip install 'git+https://github.com/magenta/magenta-realtime#egg=magenta_rt[tpu]'
# CPU only
pip install 'git+https://github.com/magenta/magenta-realtime'
```

Or, clone and install for local editing:

```sh
git clone https://github.com/magenta/magenta-realtime.git && cd magenta-realtime
pip install -e .[gpu]
```

## Examples

### Generating audio with Magenta RT

Magenta RT generates audio in short chunks (2s) given a finite amount of past
context (10s). We use crossfading to mitigate boundary artifacts between chunks.
More details on our model are coming soon in a technical report!

```py
from magenta_rt import audio, system
from IPython.display import display, Audio

num_seconds = 10
mrt = system.MagentaRT()
style = system.embed_style('funk')

chunks = []
state = None
for i in range(round(num_seconds / mrt.config.chunk_length)):
  state, chunk = mrt.generate_chunk(state=state, style=style)
  chunks.append(chunk)
generated = audio.concatenate(chunks, crossfade_time=mrt.crossfade_length)
display(Audio(generated.samples.swapaxes(0, 1), rate=mrt.sample_rate))
```

### Blending text and audio styles with MusicCoCa

MusicCoCa is a joint embedding model of text and audio styles. Magenta RT is
conditioned on MusicCoCa embeddings allowing for seamless blending of styles
using any number of text and audio prompts.

```py
from magenta_rt import audio, musiccoca

style_model = musiccoca.MusicCoCa()
my_audio = audio.Waveform.from_file('myjam.mp3')
weighted_styles = [
  (2.0, my_audio),
  (1.0, 'heavy metal'),
]
weights = np.array([w for w, _ in weighted_styles])
styles = style_model.embed([s for _, s in weighted_styles])
weights_norm = weights / weights.sum()
blended = (weights_norm[:, np.newaxis] * styles).mean(axis=0)
```

### Tokenizing audio with SpectroStream

SpectroStream is a discrete audio codec model operating on high-fidelity music
audio (stereo, 48kHz). Under the hood, Magenta RT models SpectroStream audio
tokens using a language model.

```py
from magenta_rt import audio, spectrostream

codec = spectrostream.SpectroStream()
my_audio = audio.Waveform.from_file('jam.mp3')
my_tokens = codec.encode(my_audio)
my_audio_reconstruction = codec.decode(tokens)
```

## Running tests

Unit tests:

```sh
pip install -e .[test]
pytest .
```

Integration tests:

```sh
python test/musiccoca_end2end_test.py
python test/spectrostream_end2end_test.py
python test/magenta_rt_end2end_test.py
```

## Citing this work

Please cite our [technical report](https://arxiv.org/abs/2508.04651):

**BibTeX:**

```
@article{gdmlyria2025live,
    title={Live Music Models},
    author={Caillon, Antoine and McWilliams, Brian and Tarakajian, Cassie and Simon, Ian and Manco, Ilaria and Engel, Jesse and Constant, Noah and Li, Pen and Denk, Timo I. and Lalama, Alberto and Agostinelli, Andrea and Huang, Anna and Manilow, Ethan and Brower, George and Erdogan, Hakan and Lei, Heidi and Rolnick, Itai and Grishchenko, Ivan and Orsini, Manu and Kastelic, Matej and Zuluaga, Mauricio and Verzetti, Mauro and Dooley, Michael and Skopek, Ondrej and Ferrer, Rafael and Borsos, Zal{\'a}n and van den Oord, {\"A}aron and Eck, Douglas and Collins, Eli and Baldridge, Jason and Hume, Tom and Donahue, Chris and Han, Kehang and Roberts, Adam},
    journal={arXiv:2508.04651},
    year={2025}
}
```

## License and disclaimer

Magenta RealTime is offered under a combination of licenses: the codebase is
licensed under
[Apache 2.0](https://github.com/magenta/magenta-realtime/blob/main/LICENSE),
and the model weights under
[Creative Commons Attribution 4.0 International](https://creativecommons.org/licenses/by/4.0/legalcode).

In addition, we specify the following usage terms:

Copyright 2025 Google LLC

Use these materials responsibly and do not generate content, including outputs,
that infringe or violate the rights of others, including rights in copyrighted
content.

Google claims no rights in outputs you generate using Magenta RealTime. You and
your users are solely responsible for outputs and their subsequent uses.

Unless required by applicable law or agreed to in writing, all software and
materials distributed here under the Apache 2.0 or CC-BY licenses are
distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
either express or implied. See the licenses for the specific language governing
permissions and limitations under those licenses. You are solely responsible for
determining the appropriateness of using, reproducing, modifying, performing,
displaying or distributing the software and materials, and any outputs, and
assume any and all risks associated with your use or distribution of any of the
software and materials, and any outputs, and your exercise of rights and
permissions under the licenses.
