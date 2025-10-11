# Starting server:
uvicorn --factory --host 0.0.0.0 --port 8000 speaches.main:create_app

# Pulling models:
$env:Path += ';C:\Users\Shoai\AppData\Local\Microsoft\WinGet\Packages\jqlang.jq_Microsoft.Winget.Source_8wekyb3d8bbwe'

For detailed output:
uvx speaches-cli registry ls --task text-to-speech

For just names:
uvx speaches-cli registry ls --task text-to-speech | jq '.data[] | .id'

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