# StatLingo Piper TTS Models

Pre-processed [Piper VITS](https://github.com/rhasspy/piper) neural text-to-speech models with [sherpa-onnx](https://github.com/k2-fsa/sherpa-onnx) metadata injected for use in the StatLingo Android app.

## Why This Repo Exists

The original Piper models from [rhasspy/piper-voices](https://huggingface.co/rhasspy/piper-voices) on HuggingFace do **not** include `sample_rate` in the ONNX model metadata. When sherpa-onnx loads these models on Android, it crashes with:

```
'sample_rate' does not exist in the metadata
Fatal signal 6 (SIGABRT)
```

This repo hosts the same models with the required metadata injected so they work with sherpa-onnx out of the box.

## Download

All models are available as release assets:

**[Download from Releases →](https://github.com/jonpecson/statlingo-piper-models/releases/tag/v1.0)**

### Direct URLs

```
https://github.com/jonpecson/statlingo-piper-models/releases/download/v1.0/{model_name}.onnx
```

## Available Models

| Language | Code | Model | Quality | Size | Sample Rate |
|----------|------|-------|---------|------|-------------|
| Arabic | ar | ar_JO-kareem-low | Low | 60 MB | 16000 Hz |
| Chinese | zh | zh_CN-huayan-medium | Medium | 60 MB | 16000 Hz |
| Dutch | nl | nl_NL-ronnie-medium | Medium | 60 MB | 16000 Hz |
| Farsi | fa | fa_IR-gyro-medium | Medium | 60 MB | 16000 Hz |
| French | fr | fr_FR-siwis-low | Low | 27 MB | 16000 Hz |
| German | de | de_DE-thorsten-medium | Medium | 60 MB | 16000 Hz |
| Hindi | hi | hi_IN-rohan-medium | Medium | 60 MB | 16000 Hz |
| Indonesian | id | id_ID-news_tts-medium | Medium | 60 MB | 16000 Hz |
| Italian | it | it_IT-paola-medium | Medium | 61 MB | 16000 Hz |
| Nepali | ne | ne_NP-chitwan-medium | Medium | 60 MB | 16000 Hz |
| Polish | pl | pl_PL-gosia-medium | Medium | 60 MB | 16000 Hz |
| Portuguese | pt | pt_BR-edresson-low | Low | 60 MB | 16000 Hz |
| Russian | ru | ru_RU-ruslan-medium | Medium | 60 MB | 16000 Hz |
| Swahili | sw | sw_CD-lanfrica-medium | Medium | 60 MB | 16000 Hz |
| Ukrainian | uk | uk_UA-ukrainian_tts-medium | Medium | 73 MB | 16000 Hz |
| Vietnamese | vi | vi_VN-vivos-x_low | Low | 27 MB | 16000 Hz |

**Total:** 16 languages, ~909 MB

## Injected Metadata

Each model has these keys added to the ONNX metadata:

| Key | Example | Purpose |
|-----|---------|---------|
| `sample_rate` | `16000` | Audio output sample rate (required by sherpa-onnx) |
| `model_type` | `vits` | Model architecture identifier |
| `comment` | `piper` | Source framework |
| `language` | `Arabic` | Language name in English |
| `voice` | `ar` | espeak-ng voice identifier |
| `has_espeak` | `1` | Uses espeak-ng for phonemization |
| `n_speakers` | `1` | Number of speaker voices |

## Usage with sherpa-onnx (Android)

```kotlin
// Add dependency
implementation("com.github.k2-fsa:sherpa-onnx:1.12.26")

// Load model
val vits = OfflineTtsVitsModelConfig()
vits.model = "/path/to/ar_JO-kareem-low.onnx"
vits.tokens = "/path/to/tokens.txt"
vits.dataDir = "/path/to/espeak-ng-data"

val modelConfig = OfflineTtsModelConfig()
modelConfig.vits = vits
modelConfig.numThreads = 4
modelConfig.provider = "cpu"

val config = OfflineTtsConfig()
config.model = modelConfig

val tts = OfflineTts(null, config)

// Generate speech
val audio = tts.generate("مرحبا", 0, 1.0f)
// audio.samples = Float32 PCM, audio.sampleRate = 16000
```

## Requirements

- **sherpa-onnx** 1.12.26+ (Android JNI or C API)
- **espeak-ng-data** directory (for phonemization)
- **tokens.txt** (bundled with StatLingo app, or generate from `.onnx.json` config)

## How Models Were Processed

```python
import onnx

# Load original model (no metadata)
model = onnx.load("original.onnx")

# Inject sherpa-onnx required metadata
metadata = {
    "sample_rate": "16000",
    "model_type": "vits",
    "comment": "piper",
    "language": "Arabic",
    "voice": "ar",
    "has_espeak": "1",
    "n_speakers": "1",
}
for key, value in metadata.items():
    entry = model.metadata_props.add()
    entry.key = key
    entry.value = value

# Save fixed model
onnx.save(model, "fixed.onnx")
```

The `sample_rate` and `voice` values are read from each model's `.onnx.json` config on HuggingFace.

## Related Repos

- [jonpecson/statlingo-android](https://github.com/jonpecson/statlingo-android) — Android app that uses these models
- [jonpecson/statlingo-website](https://github.com/jonpecson/statlingo-website) — Marketing website
- [rhasspy/piper](https://github.com/rhasspy/piper) — Original Piper TTS project
- [k2-fsa/sherpa-onnx](https://github.com/k2-fsa/sherpa-onnx) — ONNX inference framework

## License

The models are from [rhasspy/piper-voices](https://huggingface.co/rhasspy/piper-voices) and retain their original licenses. The metadata injection does not modify the model weights — only ONNX metadata properties are added. See the original Piper project for model-specific license details.
