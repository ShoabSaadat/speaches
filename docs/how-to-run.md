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
