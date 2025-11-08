# Starting server:
uvicorn --factory --host 0.0.0.0 --port 8000 speaches.main:create_app

# Pulling models:
$env:Path += ';C:\Users\Shoai\AppData\Local\Microsoft\WinGet\Packages\jqlang.jq_Microsoft.Winget.Source_8wekyb3d8bbwe'

For detailed output:
uvx speaches-cli registry ls --task text-to-speech

For just names:
- for getting TTS:
uvx speaches-cli registry ls --task text-to-speech | jq '.data[] | .id'

- for getting STT:
uvx speaches-cli registry ls --task automatic-speech-recognition | jq '.data[].id'
or
uvx speaches-cli registry ls --task automatic-speech-recognition | jq -r '.data[] | .id' | ForEach-Object -Begin {$i=1} -Process {"$i. $_"; $i++}

------- Current downloaded models -------
Systran/faster-whisper-tiny
PierreMesure/kb-whisper-tiny-ct2
sorendal/skyrim-whisper-tiny-int8
sorendal/skyrim-whisper-small-int8
ywuachr/openai-whisper-tiny-ct2
Systran/faster-distil-whisper-small.en

# To get and install new models:
## First set local or remote env var on powershell as:
$Env:SPEACHES_BASE_URL="http://voiceagent-speaches-6pb1ai-78296c-135-181-151-53.traefik.me"

Better to do local to get a good list then use that list and follow https://speaches.ai/usage/text-to-speech/

# Testing piper models before downloading:
https://rhasspy.github.io/piper-samples/#en_GB-alan-medium

Full piper repo oon: https://github.com/OHF-Voice/piper1-gpl

# Some key piper voices:
- uvx speaches-cli model download speaches-ai/piper-en_GB-alan-medium
middle aged englishman

- uvx speaches-cli model download speaches-ai/piper-en_GB-northern_english_male-medium
Middle aged english man

- uvx speaches-cli model download speaches-ai/piper-en_US-bryce-medium
middle aged man english

- uvx speaches-cli model download speaches-ai/piper-en_US-norman-medium
old aged man
en_US-ryan-high = young man english

- uvx speaches-cli model download speaches-ai/piper-hi_IN-pratham-medium
hindi male
------------------
- uvx speaches-cli model download speaches-ai/piper-en_GB-alba-medium
middle aged english woman

- uvx speaches-cli model download speaches-ai/piper-en_GB-cori-high
young aged young english woman - good voice

- uvx speaches-cli model download speaches-ai/piper-en_GB-semaine-medium
middle age woman, good voice

- uvx speaches-cli model download speaches-ai/piper-en_US-hfc_female-medium
20s female english good voice

## Overview
- This document now tracks two parallel stacks:
  - **Chatterbox (primary)** – the OpenAI-compatible TTS service we deploy on RunPod (GPU) or Dokploy (CPU) using the `talkingcases-chatterbox-serverless` repo.
  - **Speaches (legacy)** – the original Speaches CLI workflow retained for reference and potential fallbacks.
- Keep the new `docs/tts_cpu_or_gpu_implementation_guide.md` close; it is the source of truth for full deployments.

---

## Chatterbox Deployments (Primary Path)

### 1. Local CPU / Dokploy Quick Start
1. Clone `talkingcases-chatterbox-serverless` and copy `.env.example` → `.env` with `CHATTERBOX_API_KEY` and `RUNPOD_REALTIME_PORT=8010` (Dokploy will proxy this).
2. Install dependencies with the CPU torch stack:
	```powershell
	python -m venv .venv
	.\.venv\Scripts\activate
	pip install --upgrade pip
	pip install -r requirements.txt --extra-index-url https://download.pytorch.org/whl/cpu
	```
3. Warm the FastAPI server locally:
	```powershell
	python entrypoint.py
	```
	This boots uvicorn on `http://127.0.0.1:8010/v1`.
4. Validate streaming with the included test harness:
	```powershell
	python stream_test.py
	```
	Expect the first chunk in <1.5s and PCM output logged to the console.
5. For Dokploy, build and push a CPU-tagged image then follow the instructions in the new guide’s Dokploy section. Remember to mount a volume for `checkpoints/` when hosting checkpoints locally.

### 2. RunPod GPU Serverless
1. Build the CUDA-enabled image (the default Dockerfile already pins CUDA 12.4 wheels) and push it to your registry.
2. Create a RunPod Serverless template with:
	- Container Port: `8010`
	- Environment: `CHATTERBOX_API_KEY=...`, `RUNPOD_REALTIME_PORT=8010`
3. Allocate a GPU-capable pod (RTX 4090 works well). First run downloads S3Gen/F5 checkpoints inside the container; watch logs until `OptimizedChatterboxTTS loaded successfully` appears.
4. Hit `https://<runpod-endpoint>/v1/audio/speech` with the same payload used by `stream_test.py` to confirm chunked streaming. Real-time latency should remain <1s per chunk.
5. Update `livekit-agent/.env` to point `CHATTERBOX_BASE_URL` to the RunPod endpoint before testing LiveKit sessions.

### 3. Operational Reference
- Use `python -m httpx --version` for quick connectivity tests if the agent reports timeouts.
- Collect fresh deployment details from upstream repos via `#mcp_context7_get-library-docs` whenever you update dependencies.
- See `docs/tts_cpu_or_gpu_implementation_guide.md` for troubleshooting (chunk stalls, GPU OOM, voice cloning issues).

---

## Speaches CLI (Legacy Reference)

The original Speaches workflow remains here for archival use. Keep it handy if we need STT/TTS redundancy.

### Start the Speaches Server
```powershell
uvicorn --factory --host 0.0.0.0 --port 8000 speaches.main:create_app
```

### Extend PowerShell Path for `jq`
```powershell
$env:Path += ';C:\Users\Shoai\AppData\Local\Microsoft\WinGet\Packages\jqlang.jq_Microsoft.Winget.Source_8wekyb3d8bbwe'
```

### List Available Models
- Full detail:
  ```powershell
  uvx speaches-cli registry ls --task text-to-speech
  ```
- IDs only:
  ```powershell
  uvx speaches-cli registry ls --task text-to-speech | jq '.data[] | .id'
  ```
- STT IDs:
  ```powershell
  uvx speaches-cli registry ls --task automatic-speech-recognition | jq -r '.data[] | .id'
  ```

### Current Cached Models
- Systran/faster-whisper-tiny
- PierreMesure/kb-whisper-tiny-ct2
- sorendal/skyrim-whisper-tiny-int8
- sorendal/skyrim-whisper-small-int8
- ywuachr/openai-whisper-tiny-ct2
- Systran/faster-distil-whisper-small.en

### Download Additional Models
```powershell
$Env:SPEACHES_BASE_URL="http://voiceagent-speaches-6pb1ai-78296c-135-181-151-53.traefik.me"
uvx speaches-cli model download "Systran/faster-distil-whisper-small.en"
```

### Piper & Kokoro Reference
- Samples: https://rhasspy.github.io/piper-samples/#en_GB-alan-medium
- Repository: https://github.com/OHF-Voice/piper1-gpl
- Frequently used Piper voices:
  - `speaches-ai/piper-en_GB-alan-medium`
  - `speaches-ai/piper-en_US-bryce-medium`
  - `speaches-ai/piper-hi_IN-priyamvada-medium`
- Kokoro voice IDs:
  - Female: `af_bella`, `af_nicole`, `bf_emma`
  - Male: `am_adam`, `am_michael`, `bm_george`

### Remove a Cached Model
```powershell
cd ~/.cache/huggingface/hub
rm -rf <folder_name_of_model>
```

---

## STT Quick Reference (Legacy)
1. Set base URL:
	```powershell
	$env:SPEACHES_BASE_URL = "http://voiceagent-speaches-6pb1ai-78296c-135-181-151-53.traefik.me"
	```
2. List models:
	```powershell
	uvx speaches-cli registry ls --task automatic-speech-recognition
	```
3. Download Whisper model:
	```powershell
	uvx speaches-cli model download "Systran/faster-distil-whisper-small.en"
	```
4. Export ID:
	```powershell
	$env:TRANSCRIPTION_MODEL_ID = "Systran/faster-distil-whisper-small.en"
	```
5. Transcribe local audio:
	```powershell
	curl.exe -s "$( $env:SPEACHES_BASE_URL)/v1/audio/transcriptions" -F "file=@audio.wav" -F "model=$( $env:TRANSCRIPTION_MODEL_ID)"
	```

---

## TTS Quick Reference (Legacy)
1. Configure environment:
	```powershell
	$env:SPEACHES_BASE_URL = "http://voiceagent-speaches-6pb1ai-78296c-135-181-151-53.traefik.me"
	$env:SPEECH_MODEL_ID = "speaches-ai/Kokoro-82M-v1.0-ONNX"
	$env:VOICE_ID = "af_heart"
	```
2. Download model:
	```powershell
	uvx speaches-cli model download $env:SPEECH_MODEL_ID
	```
3. Generate MP3:
	```powershell
	$jsonBody = '{ "input": "Hello from my new speech model!", "model": "{0}", "voice": "{1}" }' -f $env:SPEECH_MODEL_ID, $env:VOICE_ID
	curl.exe "$( $env:SPEACHES_BASE_URL)/v1/audio/speech" -H "Content-Type: application/json" --output audio.mp3 -d $jsonBody
	```
4. Generate WAV at 2x speed:
	```powershell
	$jsonBodyWav = '{ "input": "This is a test at double speed.", "model": "{0}", "voice": "{1}", "response_format": "wav", "speed": 2.0 }' -f $env:SPEECH_MODEL_ID, $env:VOICE_ID
	curl.exe "$( $env:SPEACHES_BASE_URL)/v1/audio/speech" -H "Content-Type: application/json" --output audio.wav -d $jsonBodyWav
	```

---

## Additional Links
- RunPod tutorials: https://docs.runpod.io/tutorials/sdks/python/get-started/running-locally
- LiveKit custom TTS thread: https://www.linen.dev/s/livekit-users/t/29627990/how-to-use-my-own-tts-or-how-to-make-tts-plugin
- Chatterbox repo: https://github.com/davidbrowne17/chatterbox-streaming
- Reference serverless handler: https://github.com/703deuce/chatterbox-tts-serverless/blob/main/handler.py
- Use `#mcp_context7_get-library-docs` to fetch the newest upstream notes before making changes.




# Legacy Guide:
# Starting server:
uvicorn --factory --host 0.0.0.0 --port 8000 speaches.main:create_app

# Pulling models:
$env:Path += ';C:\Users\Shoai\AppData\Local\Microsoft\WinGet\Packages\jqlang.jq_Microsoft.Winget.Source_8wekyb3d8bbwe'

For detailed output:
uvx speaches-cli registry ls --task text-to-speech

For just names:
- for getting TTS:
uvx speaches-cli registry ls --task text-to-speech | jq '.data[] | .id'

- for getting STT:
uvx speaches-cli registry ls --task automatic-speech-recognition | jq '.data[].id'
or
uvx speaches-cli registry ls --task automatic-speech-recognition | jq -r '.data[] | .id' | ForEach-Object -Begin {$i=1} -Process {"$i. $_"; $i++}

------- Current downloaded models -------
Systran/faster-whisper-tiny
PierreMesure/kb-whisper-tiny-ct2
sorendal/skyrim-whisper-tiny-int8
sorendal/skyrim-whisper-small-int8
ywuachr/openai-whisper-tiny-ct2
Systran/faster-distil-whisper-small.en

# To get and install new models:
## First set local or remote env var on powershell as:
$Env:SPEACHES_BASE_URL="http://voiceagent-speaches-6pb1ai-78296c-135-181-151-53.traefik.me"

Better to do local to get a good list then use that list and follow https://speaches.ai/usage/text-to-speech/

# Testing piper models before downloading:
https://rhasspy.github.io/piper-samples/#en_GB-alan-medium

Full piper repo oon: https://github.com/OHF-Voice/piper1-gpl

# Some key piper voices:
- uvx speaches-cli model download speaches-ai/piper-en_GB-alan-medium
middle aged englishman

- uvx speaches-cli model download speaches-ai/piper-en_GB-northern_english_male-medium
Middle aged english man

- uvx speaches-cli model download speaches-ai/piper-en_US-bryce-medium
middle aged man english

- uvx speaches-cli model download speaches-ai/piper-en_US-norman-medium
old aged man
en_US-ryan-high = young man english

- uvx speaches-cli model download speaches-ai/piper-hi_IN-pratham-medium
hindi male
------------------
- uvx speaches-cli model download speaches-ai/piper-en_GB-alba-medium
middle aged english woman

- uvx speaches-cli model download speaches-ai/piper-en_GB-cori-high
young aged young english woman - good voice

- uvx speaches-cli model download speaches-ai/piper-en_GB-semaine-medium
middle age woman, good voice

- uvx speaches-cli model download speaches-ai/piper-en_US-hfc_female-medium
20s female english good voice

- uvx speaches-cli model download speaches-ai/piper-hi_IN-priyamvada-medium
hindi female
----------------
- uvx speaches-cli model download speaches-ai/piper-en_US-kristin-medium
- uvx speaches-cli model download speaches-ai/piper-en_US-libritts-high
- uvx speaches-cli model download speaches-ai/piper-en_US-john-medium


# How to remove a model
- go into the terminal / bash of the speaches container
- cd ~/.cache/huggingface/hub
- ls
- rm -rf <folder_name_of_model>




# Additional material: ======================================================
# Usage video:
https://youtu.be/S3da-vg1yP8


# Downloading STT models
Putting It All Together: Your Step-by-Step Guide for PowerShell
Follow these steps in your VSCode PowerShell terminal. This will download the model to your server and then run a transcription using a local audio file.

1. Set the Server URL Variable
This command saves your server's URL in a variable for the current session.

PowerShell

$env:SPEACHES_BASE_URL = "http://voiceagent-speaches-6pb1ai-78296c-135-181-151-53.traefik.me"
2. List Available STT Models (Optional)
This command connects to your server and lists all the speech-to-text models you can download. We'll just display the raw JSON output.

PowerShell

uvx speaches-cli registry ls --task automatic-speech-recognition
3. Download the STT Model to Your Server
This command tells your remote server to download and install the recommended distil-whisper model.

PowerShell

uvx speaches-cli model download "Systran/faster-distil-whisper-small.en"
4. Set the Transcription Model ID Variable
This saves the name of the model you just downloaded into a variable.

PowerShell

$env:TRANSCRIPTION_MODEL_ID = "Systran/faster-distil-whisper-small.en"
5. Run a Transcription ✅
This is the final step. It uses the real curl.exe to upload an audio file from your computer to your server for transcription.

👉 Prerequisite: Make sure you have a file named audio.wav in the same folder you are running the command from (C:\Users\Shoai\Documents\Web Projects\speaches).

PowerShell

curl.exe -s "$($env:SPEACHES_BASE_URL)/v1/audio/transcriptions" -F "file=@audio.wav" -F "model=$($env:TRANSCRIPTION_MODEL_ID)"
After running the final command, you should see the transcribed text from your audio file printed directly in the terminal.



# For downloading TTS:
1. Set Your Environment Variables
First, we'll set all the variables we need for your server URL, the model name, and the voice. (You can skip the first line if you're in the same terminal session as before).

PowerShell

$env:SPEACHES_BASE_URL = "http://voiceagent-speaches-6pb1ai-78296c-135-181-151-53.traefik.me"
$env:SPEECH_MODEL_ID = "speaches-ai/Kokoro-82M-v1.0-ONNX"
$env:VOICE_ID = "af_heart"
2. List Available TTS Models (Optional)
This command will show you the full JSON list of available TTS models on the registry.

PowerShell

uvx speaches-cli registry ls --task text-to-speech
3. Download the TTS Model to Your Server
This command tells your remote server to download and install the Kokoro TTS model.

PowerShell

uvx speaches-cli model download $env:SPEECH_MODEL_ID
You should see a confirmation that the model was downloaded successfully.

4. Generate Speech ✅
The Linux command in the guide uses a complex, multi-line format. Here is the simple, single-line equivalent for PowerShell that does the same thing.

This command will send a request to your server and save the resulting audio as audio.mp3 in your current folder (C:\Users\Shoai\Documents\Web Projects\speaches).

PowerShell

# First, we create the JSON request body
$jsonBody = '{ "input": "Hello from my new speech model!", "model": "{0}", "voice": "{1}" }' -f $env:SPEECH_MODEL_ID, $env:VOICE_ID

# Then, we send the request using curl.exe
curl.exe "$($env:SPEACHES_BASE_URL)/v1/audio/speech" -H "Content-Type: application/json" --output audio.mp3 -d $jsonBody
After running this, check your folder for the audio.mp3 file and play it!

## Advanced Example: Changing Speed and Format
If you want to try the other examples, you just need to change the $jsonBody. For instance, to make the speech faster and output a .wav file:

PowerShell

# Create a new JSON body with more options
$jsonBodyWav = '{ "input": "This is a test at double speed.", "model": "{0}", "voice": "{1}", "response_format": "wav", "speed": 2.0 }' -f $env:SPEECH_MODEL_ID, $env:VOICE_ID

# Send the request and save as audio.wav
curl.exe "$($env:SPEACHES_BASE_URL)/v1/audio/speech" -H "Content-Type: application/json" --output audio.wav -d $jsonBodyWav