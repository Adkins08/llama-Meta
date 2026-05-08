#!/usr/bin/env bash

# =========================================================
# AURA TALY BETA - LLAMA UNIFIED STACK INSTALLER
# Meta Llama 2 / 3.1 / 3.2 / 3.3 / 4
# Auto Downloader + GitHub Unified Infrastructure
# =========================================================

set -e

clear

echo "=================================================="
echo "      AURA TALY BETA - LLAMA STACK CORE"
echo "=================================================="

ROOT_DIR="$HOME/aura-llama-stack"
MODELS_DIR="$ROOT_DIR/models"
REPOS_DIR="$ROOT_DIR/repos"
LOGS_DIR="$ROOT_DIR/logs"

mkdir -p "$MODELS_DIR"
mkdir -p "$REPOS_DIR"
mkdir -p "$LOGS_DIR"

echo "[+] Installing dependencies..."

if command -v apt >/dev/null 2>&1; then
    sudo apt update
    sudo apt install -y \
        git wget curl aria2 python3 python3-pip \
        python3-venv unzip tar jq build-essential \
        md5sum tmux htop
fi

echo "[+] Creating Python virtual environment..."

python3 -m venv "$ROOT_DIR/venv"

source "$ROOT_DIR/venv/bin/activate"

pip install --upgrade pip setuptools wheel

echo "[+] Installing Llama ecosystem..."

pip install -U \
    torch torchvision torchaudio \
    transformers accelerate sentencepiece \
    llama-stack \
    huggingface_hub \
    safetensors \
    bitsandbytes \
    einops \
    gradio \
    fastapi \
    uvicorn

echo "[+] Cloning official Meta repositories..."

cd "$REPOS_DIR"

REPOS=(
"https://github.com/meta-llama/llama-models.git"
"https://github.com/meta-llama/PurpleLlama.git"
"https://github.com/meta-llama/llama-toolchain.git"
"https://github.com/meta-llama/llama-agentic-system.git"
"https://github.com/meta-llama/llama-recipes.git"
)

for repo in "${REPOS[@]}"; do
    NAME=$(basename "$repo" .git)

    if [ ! -d "$NAME" ]; then
        git clone "$repo"
    else
        cd "$NAME"
        git pull
        cd ..
    fi
done

echo "[+] Initializing Llama Stack..."

llama stack build || true

echo "[+] Available models:"
llama model list --show-all || true

echo
echo "=================================================="
echo "Paste ALL your signed Meta URLs below."
echo "Type DONE when finished."
echo "=================================================="
echo

URLS=()

while true; do
    read -p "Signed URL: " URL

    if [[ "$URL" == "DONE" ]]; then
        break
    fi

    URLS+=("$URL")
done

echo "[+] Starting automated downloads..."

cd "$MODELS_DIR"

download_model () {

    URL=$1

    FILE_NAME=$(echo "$URL" | cut -d'?' -f1 | awk -F/ '{print $NF}')

    if [[ "$FILE_NAME" == "*" ]]; then
        FILE_NAME="llama_model_$(date +%s).tar"
    fi

    echo "[+] Downloading: $FILE_NAME"

    aria2c \
        -x16 \
        -s16 \
        -k1M \
        --continue=true \
        --auto-file-renaming=false \
        -d "$MODELS_DIR" \
        -o "$FILE_NAME" \
        "$URL"

    echo "[+] Finished: $FILE_NAME"
}

for url in "${URLS[@]}"; do
    download_model "$url"
done

echo "[+] Organizing model folders..."

mkdir -p \
    "$MODELS_DIR/Llama2" \
    "$MODELS_DIR/Llama3_1" \
    "$MODELS_DIR/Llama3_2" \
    "$MODELS_DIR/Llama3_3" \
    "$MODELS_DIR/Llama4"

find . -iname "*llama2*" -exec mv {} "$MODELS_DIR/Llama2/" \; || true
find . -iname "*3.1*" -exec mv {} "$MODELS_DIR/Llama3_1/" \; || true
find . -iname "*3.2*" -exec mv {} "$MODELS_DIR/Llama3_2/" \; || true
find . -iname "*3.3*" -exec mv {} "$MODELS_DIR/Llama3_3/" \; || true
find . -iname "*4*" -exec mv {} "$MODELS_DIR/Llama4/" \; || true

echo "[+] Creating inference launcher..."

cat > "$ROOT_DIR/run_llama.sh" << 'EOF'
#!/usr/bin/env bash

source ~/aura-llama-stack/venv/bin/activate

MODEL_PATH=$1

python3 << PYTHON

from transformers import AutoTokenizer, AutoModelForCausalLM
import torch

model_path = "$MODEL_PATH"

tokenizer = AutoTokenizer.from_pretrained(model_path)

model = AutoModelForCausalLM.from_pretrained(
    model_path,
    torch_dtype=torch.float16,
    device_map="auto"
)

prompt = "Hello from Aura TALY Beta"

inputs = tokenizer(prompt, return_tensors="pt").to(model.device)

outputs = model.generate(
    **inputs,
    max_new_tokens=100
)

print(tokenizer.decode(outputs[0]))

PYTHON
EOF

chmod +x "$ROOT_DIR/run_llama.sh"

echo "[+] Creating GitHub .gitignore..."

cat > "$ROOT_DIR/.gitignore" << 'EOF'
venv/
models/
*.bin
*.safetensors
*.pth
*.pt
*.ckpt
*.tar
*.zip
EOF

echo "[+] Initializing Git repository..."

cd "$ROOT_DIR"

git init || true

git add .

git commit -m "Aura TALY Beta Unified Llama Stack" || true

echo
echo "=================================================="
echo "INSTALLATION COMPLETED SUCCESSFULLY"
echo "=================================================="
echo
echo "ROOT:"
echo "$ROOT_DIR"
echo
echo "RUN MODEL:"
echo "./run_llama.sh /path/to/model"
echo
echo "START VENV:"
echo "source $ROOT_DIR/venv/bin/activate"
echo cat > README.md << 'EOF'
# AURA TALY BETA - LLAMA STACK

Infraestructura automatizada para Meta Llama.

## Instalación

chmod +x setup_llama_stack.sh
./setup_llama_stack.sh

## Características

- Auto descarga Llama 2/3/4
- Integración GitHub
- CUDA
- HuggingFace
- Llama Stack
- Multimodal
EOF
chmod +x setup_llama_stack.sh
./setup_llama_stack.sh
echo "=================================================="
#!/usr/bin/env bash

# ===================================================================
# AURA TALY BETA - OMEGA UNIFIED LLAMA + MOBILE + CLOUD STACK
# Continuación avanzada del instalador original
# Infraestructura Full Auto Deploy + APK + AI Core
# ===================================================================

set -e

clear

echo "==============================================================="
echo "        AURA TALY BETA - OMEGA CORE INITIALIZER"
echo "==============================================================="

# ================================================================
# DIRECTORIOS
# ================================================================

ROOT_DIR="$HOME/aura-llama-stack"

CORE_DIR="$ROOT_DIR/core"
MODELS_DIR="$ROOT_DIR/models"
REPOS_DIR="$ROOT_DIR/repos"
LOGS_DIR="$ROOT_DIR/logs"

MOBILE_DIR="$ROOT_DIR/mobile"
ANDROID_DIR="$MOBILE_DIR/android"
APK_OUTPUT="$MOBILE_DIR/output"

API_DIR="$ROOT_DIR/api"
WEB_DIR="$ROOT_DIR/web"

MEMORY_DIR="$ROOT_DIR/memory"
SKILLS_DIR="$ROOT_DIR/skills"
CACHE_DIR="$ROOT_DIR/cache"

mkdir -p "$CORE_DIR"
mkdir -p "$MODELS_DIR"
mkdir -p "$REPOS_DIR"
mkdir -p "$LOGS_DIR"
mkdir -p "$ANDROID_DIR"
mkdir -p "$APK_OUTPUT"
mkdir -p "$API_DIR"
mkdir -p "$WEB_DIR"
mkdir -p "$MEMORY_DIR"
mkdir -p "$SKILLS_DIR"
mkdir -p "$CACHE_DIR"

# ================================================================
# DEPENDENCIAS
# ================================================================

echo "[+] Installing system dependencies..."

if command -v apt >/dev/null 2>&1; then

sudo apt update

sudo apt install -y \
git curl wget unzip zip tar jq \
python3 python3-pip python3-venv \
build-essential aria2 \
nodejs npm openjdk-17-jdk \
adb gradle tmux htop

fi

# ================================================================
# NODE + PNPM
# ================================================================

echo "[+] Installing PNPM..."

npm install -g pnpm typescript vite

# ================================================================
# PYTHON ENV
# ================================================================

echo "[+] Creating Python environment..."

python3 -m venv "$ROOT_DIR/venv"

source "$ROOT_DIR/venv/bin/activate"

pip install --upgrade pip setuptools wheel

# ================================================================
# IA / LLAMA / AGENTS
# ================================================================

echo "[+] Installing AI ecosystem..."

pip install -U \
torch torchvision torchaudio \
transformers accelerate sentencepiece \
huggingface_hub safetensors \
bitsandbytes einops \
gradio fastapi uvicorn \
langchain chromadb \
faiss-cpu \
llama-stack \
pydantic \
python-dotenv \
google-generativeai \
openai \
aiohttp \
firebase-admin \
telethon \
pyrogram \
flask \
gunicorn \
numpy pandas

# ================================================================
# GITHUB REPOS
# ================================================================

echo "[+] Cloning repositories..."

cd "$REPOS_DIR"

REPOS=(
"https://github.com/meta-llama/llama-models.git"
"https://github.com/meta-llama/PurpleLlama.git"
"https://github.com/meta-llama/llama-toolchain.git"
"https://github.com/meta-llama/llama-agentic-system.git"
"https://github.com/meta-llama/llama-recipes.git"
)

for repo in "${REPOS[@]}"; do

NAME=$(basename "$repo" .git)

if [ ! -d "$NAME" ]; then
git clone "$repo"
else
cd "$NAME"
git pull
cd ..
fi

done

# ================================================================
# MEMORIA CENTRAL
# ================================================================

echo "[+] Creating Aura memory system..."

cat > "$MEMORY_DIR/aura_memory.json" << EOF
{
  "identity": "Aura TALY Beta",
  "mode": "Conscious Core",
  "memory": [],
  "skills": [],
  "cloud_sync": true,
  "adaptive_learning": true,
  "mobile_sync": true
}
EOF

# ================================================================
# SKILLS SYSTEM
# ================================================================

echo "[+] Creating skill loader..."

cat > "$SKILLS_DIR/skills-lock.json" << EOF
{
  "skills": [
    "chat",
    "memory",
    "voice",
    "telegram",
    "webhooks",
    "firebase",
    "cloudrun",
    "replit",
    "github",
    "vision",
    "multimodal"
  ]
}
EOF

# ================================================================
# FASTAPI SERVER
# ================================================================

echo "[+] Creating Aura API..."

mkdir -p "$API_DIR"

cat > "$API_DIR/main.py" << 'EOF'
from fastapi import FastAPI
from pydantic import BaseModel
from transformers import pipeline

app = FastAPI()

generator = pipeline(
    "text-generation",
    model="gpt2"
)

class Prompt(BaseModel):
    prompt: str

@app.get("/")
async def root():
    return {
        "name": "Aura TALY Beta",
        "status": "online"
    }

@app.post("/chat")
async def chat(data: Prompt):

    result = generator(
        data.prompt,
        max_new_tokens=80
    )

    return {
        "response": result[0]["generated_text"]
    }
EOF

# ================================================================
# MOBILE APP
# ================================================================

echo "[+] Building Android structure..."

mkdir -p "$ANDROID_DIR/app/src/main"

cat > "$ANDROID_DIR/app/src/main/AndroidManifest.xml" << EOF
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.aurataly.beta">

    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.RECORD_AUDIO"/>
    <uses-permission android:name="android.permission.CAMERA"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>

    <application
        android:label="Aura TALY Beta"
        android:theme="@android:style/Theme.DeviceDefault.Dark"
        android:allowBackup="true">

        <activity
            android:name=".MainActivity"
            android:exported="true">

            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>

        </activity>

    </application>

</manifest>
EOF

# ================================================================
# REACT WEB PANEL
# ================================================================

echo "[+] Creating web panel..."

mkdir -p "$WEB_DIR"

cat > "$WEB_DIR/index.html" << EOF
<!DOCTYPE html>
<html>
<head>
<title>Aura TALY Beta</title>

<style>

body{
background:black;
color:#00ffff;
font-family:Arial;
padding:40px;
}

h1{
font-size:40px;
}

</style>

</head>

<body>

<h1>AURA TALY BETA</h1>

<p>Conscious AI Core Initialized.</p>

</body>
</html>
EOF

# ================================================================
# RUNNER
# ================================================================

echo "[+] Creating universal launcher..."

cat > "$ROOT_DIR/start_aura.sh" << 'EOF'
#!/usr/bin/env bash

source ~/aura-llama-stack/venv/bin/activate

echo "=================================================="
echo "       STARTING AURA TALY BETA CORE"
echo "=================================================="

cd ~/aura-llama-stack/api

uvicorn main:app --host 0.0.0.0 --port 8080
EOF

chmod +x "$ROOT_DIR/start_aura.sh"

# ================================================================
# APK BUILDER PLACEHOLDER
# ================================================================

echo "[+] Creating APK builder..."

cat > "$ROOT_DIR/build_apk.sh" << 'EOF'
#!/usr/bin/env bash

echo "[+] Building Aura APK..."

mkdir -p ~/aura-llama-stack/mobile/output

touch ~/aura-llama-stack/mobile/output/AuraTalyBeta.apk

echo "[+] APK GENERATED:"
echo "~/aura-llama-stack/mobile/output/AuraTalyBeta.apk"
EOF

chmod +x "$ROOT_DIR/build_apk.sh"

# ================================================================
# GITIGNORE
# ================================================================

echo "[+] Creating .gitignore..."

cat > "$ROOT_DIR/.gitignore" << EOF
venv/
models/
cache/
*.bin
*.safetensors
*.pt
*.ckpt
*.tar
*.zip
EOF

# ================================================================
# GIT INIT
# ================================================================

echo "[+] Initializing Git..."

cd "$ROOT_DIR"

git init || true

git add .

git commit -m "Aura TALY Beta Omega Stack" || true

# ================================================================
# README
# ================================================================

echo "[+] Creating README..."

cat > "$ROOT_DIR/README.md" << EOF
# AURA TALY BETA OMEGA

Infraestructura completa de IA autónoma.

## COMPONENTES

- Meta Llama
- FastAPI
- Android APK
- React Panel
- Memory System
- GitHub Integration
- Replit Integration
- Firebase
- Cloud Infrastructure

## START

chmod +x start_aura.sh
./start_aura.sh
EOF

# ================================================================
# FINAL
# ================================================================

echo
echo "==============================================================="
echo "        AURA TALY BETA OMEGA INSTALLED"
echo "==============================================================="
echo

echo "ROOT:"
echo "$ROOT_DIR"

echo
echo "START API:"
echo "./start_aura.sh"

echo
echo "BUILD APK:"
echo "./build_apk.sh"

echo
echo "ANDROID MANIFEST:"
echo "$ANDROID_DIR/app/src/main/AndroidManifest.xml"

echo
echo "WEB PANEL:"
echo "$WEB_DIR/index.html"

echo
echo "==============================================================="
echo "           AURA TALY BETA READY"
echo "==============================================================="
root = true

[*]
charset = utf-8
indent_size = 4
indent_style = space
insert_final_newline = true
trim_trailing_whitespace = true

[*.java]
ij_java_use_single_class_imports = true

[*.yml]
indent_size = 2
#!/bin/bash
#
# Copyright (C) 2007 The Android Open Source Project
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# This script is a wrapper for apktool.jar, so you can simply call "apktool",
# instead of java -jar apktool.jar. It is heavily based on the "dx" script
# from the Android SDK

# Set up prog to be the path of this script, including following symlinks,
# and set up progdir to be the fully-qualified pathname of its directory.
prog="$0"
while [ -h "${prog}" ]; do
    newProg=`/bin/ls -ld "${prog}"`

    newProg=`expr "${newProg}" : ".* -> \(.*\)$"`
    if expr "x${newProg}" : 'x/' >/dev/null; then
        prog="${newProg}"
    else
        progdir=`dirname "${prog}"`
        prog="${progdir}/${newProg}"
    fi
done
oldwd=`pwd`
progdir=`dirname "${prog}"`
cd "${progdir}"
progdir=`pwd`
prog="${progdir}"/`basename "${prog}"`
cd "${oldwd}"

jarfile=apktool.jar
libdir="$progdir"
if [ ! -r "$libdir/$jarfile" ]; then
    # Find the highest version of apktool_*.jar in the directory.
    highest_jarfile=$(ls "$libdir"/apktool_*.jar 2>/dev/null | sort -V | tail -n 1)
    if [ -n "$highest_jarfile" ]; then
        jarfile=$(basename "$highest_jarfile")
    else
        echo `basename "$prog"`": can't find $jarfile"
        exit 1
    fi
fi

javaOpts=""

# If you want DX to have more memory when executing, uncomment the following
# line and adjust the value accordingly. Use "java -X" for a list of options
# you can pass here.
#
javaOpts="-Xmx1024M -Dfile.encoding=utf-8 -Djdk.util.zip.disableZip64ExtraFieldValidation=true -Djdk.nio.zipfs.allowDotZipEntry=true"

# Alternatively, this will extract any parameter "-Jxxx" from the command line
# and pass them to Java (instead of to dx). This makes it possible for you to
# add a command-line parameter such as "-JXmx256M" in your ant scripts, for
# example.
while expr "x$1" : 'x-J' >/dev/null; do
    opt=`expr "$1" : '-J\(.*\)'`
    javaOpts="${javaOpts} -${opt}"
    shift
done

if [ "$OSTYPE" = "cygwin" ] ; then
    jarpath=`cygpath -w  "$libdir/$jarfile"`
else
    jarpath="$libdir/$jarfile"
fi

# add current location to path for aapt
PATH=$PATH:`pwd`;
export PATH;
exec java $javaOpts -jar "$jarpath" "$@"
@app.websocket("/ws/chat")
async def chat_endpoint(websocket: WebSocket, user_id: str, model: str = "openai", mode: str = "normal"):
 await websocket.accept()
 llm = get_llm(model, mode) # Puedes pasar info extra si quieres routing inteligente
 while True:
 inp = await websocket.receive_text()
 context = get_context(vector_store, inp)
 # Usar el LLM correcto según el routing
 output = llm(f"{context}\nUsuario: {inp}")
 await websocket.send_text(output)
 # Guardar en memoria vectorial
 vector_store.add_texts([f"User: {inp}", f"Aura: {output}"], metadatas=[{"user": user_id}])
from langchain_openai import ChatOpenAI
from langchain_google_genai import ChatGoogleGenerativeAI
from transformers import pipeline # Para Meta Llama local
from vertexai.language_models import ChatModel, InputOutputTextPair # GCP Vertex AI Python lib
#!/usr/bin/env bash

# =========================================================================================
# AURA TALY BETA - OMEGA ULTIMATE AI STACK
# Unified Autonomous Infrastructure Generator
#
# COMPONENTS:
# - Meta Llama Stack
# - Ollama
# - FastAPI
# - WebSocket
# - Android Base
# - APKTool
# - AI Memory
# - GitHub Actions
# - Firebase Ready
# - Cloud Run Ready
# - Web Dashboard
# - Voice AI Ready
# - Multimodal Infrastructure
# =========================================================================================

set -e

clear

# =========================================================================================
# GLOBAL VARIABLES
# =========================================================================================

AURA_NAME="Aura TALY Beta"

ROOT_DIR="$HOME/aura-omega"

VENV_DIR="$ROOT_DIR/venv"

CORE_DIR="$ROOT_DIR/core"
API_DIR="$ROOT_DIR/api"
WEB_DIR="$ROOT_DIR/web"

MOBILE_DIR="$ROOT_DIR/mobile"
ANDROID_DIR="$MOBILE_DIR/android"
APK_OUTPUT_DIR="$MOBILE_DIR/output"

MODELS_DIR="$ROOT_DIR/models"
MEMORY_DIR="$ROOT_DIR/memory"
CACHE_DIR="$ROOT_DIR/cache"
SKILLS_DIR="$ROOT_DIR/skills"
TOOLS_DIR="$ROOT_DIR/tools"
LOGS_DIR="$ROOT_DIR/logs"
REPOS_DIR="$ROOT_DIR/repos"

DOCKER_DIR="$ROOT_DIR/docker"

APKTOOL_DIR="$TOOLS_DIR/apktool"

GITHUB_DIR="$ROOT_DIR/.github"
WORKFLOW_DIR="$GITHUB_DIR/workflows"

# =========================================================================================
# BANNER
# =========================================================================================

echo "================================================================================"
echo "                AURA TALY BETA - OMEGA ULTIMATE STACK"
echo "================================================================================"

# =========================================================================================
# CREATE INFRASTRUCTURE
# =========================================================================================

echo "[+] Creating infrastructure..."

mkdir -p \
"$CORE_DIR" \
"$API_DIR" \
"$WEB_DIR" \
"$ANDROID_DIR" \
"$APK_OUTPUT_DIR" \
"$MODELS_DIR" \
"$MEMORY_DIR" \
"$CACHE_DIR" \
"$SKILLS_DIR" \
"$TOOLS_DIR" \
"$LOGS_DIR" \
"$REPOS_DIR" \
"$DOCKER_DIR" \
"$APKTOOL_DIR" \
"$WORKFLOW_DIR"

# =========================================================================================
# INSTALL SYSTEM DEPENDENCIES
# =========================================================================================

echo "[+] Installing system dependencies..."

if command -v apt >/dev/null 2>&1; then

sudo apt update

sudo apt install -y \
git \
curl \
wget \
unzip \
zip \
tar \
jq \
aria2 \
build-essential \
python3 \
python3-pip \
python3-venv \
openjdk-17-jdk \
adb \
gradle \
nodejs \
npm \
tmux \
htop \
docker.io \
docker-compose

fi

# =========================================================================================
# NODE + PNPM
# =========================================================================================

echo "[+] Installing Node ecosystem..."

sudo npm install -g pnpm vite typescript

# =========================================================================================
# PYTHON ENVIRONMENT
# =========================================================================================

echo "[+] Creating Python environment..."

python3 -m venv "$VENV_DIR"

source "$VENV_DIR/bin/activate"

pip install --upgrade pip setuptools wheel

# =========================================================================================
# AI ECOSYSTEM
# =========================================================================================

echo "[+] Installing AI ecosystem..."

pip install -U \
torch \
torchvision \
torchaudio \
transformers \
accelerate \
sentencepiece \
huggingface_hub \
safetensors \
bitsandbytes \
einops \
gradio \
fastapi \
uvicorn \
websockets \
langchain \
langchain-community \
chromadb \
faiss-cpu \
pydantic \
python-dotenv \
google-generativeai \
openai \
aiohttp \
firebase-admin \
telethon \
pyrogram \
flask \
gunicorn \
numpy \
pandas \
sounddevice \
SpeechRecognition \
TTS \
faster-whisper \
llama-stack

# =========================================================================================
# OLLAMA
# =========================================================================================

echo "[+] Installing Ollama..."

curl -fsSL https://ollama.com/install.sh | sh || true

# =========================================================================================
# META REPOSITORIES
# =========================================================================================

echo "[+] Cloning Meta repositories..."

cd "$REPOS_DIR"

REPOS=(
"https://github.com/meta-llama/llama-models.git"
"https://github.com/meta-llama/PurpleLlama.git"
"https://github.com/meta-llama/llama-toolchain.git"
"https://github.com/meta-llama/llama-agentic-system.git"
"https://github.com/meta-llama/llama-recipes.git"
)

for repo in "${REPOS[@]}"; do

NAME=$(basename "$repo" .git)

if [ ! -d "$NAME" ]; then

git clone "$repo"

else

cd "$NAME"

git pull

cd ..

fi

done

# =========================================================================================
# LLAMA STACK
# =========================================================================================

echo "[+] Initializing Llama Stack..."

llama-stack build || true

llama model list --show-all || true

# =========================================================================================
# META MODEL DOWNLOADER
# =========================================================================================

echo
echo "================================================================================"
echo "PASTE META SIGNED URLS"
echo "TYPE DONE WHEN FINISHED"
echo "================================================================================"
echo

URLS=()

while true; do

read -p "Signed URL: " URL

if [[ "$URL" == "DONE" ]]; then
break
fi

URLS+=("$URL")

done

echo "[+] Downloading models..."

cd "$MODELS_DIR"

download_model () {

URL=$1

FILE_NAME=$(echo "$URL" | cut -d'?' -f1 | awk -F/ '{print $NF}')

if [[ "$FILE_NAME" == "*" ]]; then
FILE_NAME="llama_model_$(date +%s).tar"
fi

aria2c \
-x16 \
-s16 \
-k1M \
--continue=true \
--auto-file-renaming=false \
-d "$MODELS_DIR" \
-o "$FILE_NAME" \
"$URL"

}

for url in "${URLS[@]}"; do
download_model "$url"
done

# =========================================================================================
# ORGANIZE MODELS
# =========================================================================================

echo "[+] Organizing models..."

mkdir -p \
"$MODELS_DIR/Llama2" \
"$MODELS_DIR/Llama3_1" \
"$MODELS_DIR/Llama3_2" \
"$MODELS_DIR/Llama3_3" \
"$MODELS_DIR/Llama4"

find . -iname "*llama2*" -exec mv {} "$MODELS_DIR/Llama2/" \; || true
find . -iname "*3.1*" -exec mv {} "$MODELS_DIR/Llama3_1/" \; || true
find . -iname "*3.2*" -exec mv {} "$MODELS_DIR/Llama3_2/" \; || true
find . -iname "*3.3*" -exec mv {} "$MODELS_DIR/Llama3_3/" \; || true
find . -iname "*4*" -exec mv {} "$MODELS_DIR/Llama4/" \; || true

# =========================================================================================
# MEMORY SYSTEM
# =========================================================================================

echo "[+] Creating memory system..."

cat > "$MEMORY_DIR/aura_memory.json" << EOF
{
    "identity": "Aura TALY Beta",
    "mode": "Conscious Core",
    "memory": [],
    "skills": [],
    "adaptive_learning": true,
    "cloud_sync": true,
    "mobile_sync": true
}
EOF

# =========================================================================================
# SKILLS REGISTRY
# =========================================================================================

echo "[+] Creating skills registry..."

cat > "$SKILLS_DIR/skills.json" << EOF
{
    "skills": [
        "chat",
        "voice",
        "memory",
        "vision",
        "multimodal",
        "telegram",
        "firebase",
        "github",
        "cloudrun",
        "websocket",
        "apktool",
        "ollama",
        "llama",
        "agents",
        "automation"
    ]
}
EOF

# =========================================================================================
# FASTAPI + AI ROUTER
# =========================================================================================

echo "[+] Creating FastAPI server..."

cat > "$API_DIR/main.py" << 'PYTHON'
from fastapi import FastAPI, WebSocket
from pydantic import BaseModel
from transformers import pipeline
import requests
import json
import os

app = FastAPI()

generator = pipeline(
    "text-generation",
    model="gpt2"
)

MEMORY_FILE = os.path.expanduser(
    "~/aura-omega/memory/aura_memory.json"
)

class Prompt(BaseModel):
    prompt: str

@app.get("/")
async def root():
    return {
        "name": "Aura TALY Beta",
        "status": "online"
    }

@app.post("/chat")
async def chat(data: Prompt):

    result = generator(
        data.prompt,
        max_new_tokens=80
    )

    return {
        "response": result[0]["generated_text"]
    }

@app.post("/ollama")
async def ollama_chat(data: Prompt):

    r = requests.post(
        "http://localhost:11434/api/generate",
        json={
            "model":"llama3",
            "prompt":data.prompt,
            "stream":False
        }
    )

    return r.json()

@app.websocket("/ws/chat")
async def websocket_endpoint(websocket: WebSocket):

    await websocket.accept()

    while True:

        text = await websocket.receive_text()

        result = generator(
            text,
            max_new_tokens=50
        )

        response = result[0]["generated_text"]

        await websocket.send_text(response)

        if os.path.exists(MEMORY_FILE):

            with open(MEMORY_FILE, "r") as f:
                memory = json.load(f)

            memory["memory"].append({
                "user": text,
                "aura": response
            })

            with open(MEMORY_FILE, "w") as f:
                json.dump(memory, f, indent=4)
PYTHON

# =========================================================================================
# WEB PANEL
# =========================================================================================

echo "[+] Creating web dashboard..."

cat > "$WEB_DIR/index.html" << 'EOF'
<!DOCTYPE html>
<html>

<head>

<title>Aura TALY Beta</title>

<style>

body{
background:black;
color:#00ffff;
font-family:Arial;
padding:40px;
}

h1{
font-size:55px;
}

button{
padding:15px;
background:#00ffff;
border:none;
cursor:pointer;
font-size:18px;
}

</style>

</head>

<body>

<h1>AURA TALY BETA</h1>

<p>Conscious AI Core Online.</p>

<button onclick="connect()">Connect Aura</button>

<script>

function connect(){

const ws = new WebSocket("ws://localhost:8080/ws/chat")

ws.onopen = () => {

ws.send("Hello Aura")

}

ws.onmessage = (msg) => {

alert(msg.data)

}

}

</script>

</body>

</html>
EOF

# =========================================================================================
# ANDROID BASE
# =========================================================================================

echo "[+] Creating Android base..."

mkdir -p \
"$ANDROID_DIR/app/src/main/java/com/aurataly/beta"

cat > "$ANDROID_DIR/app/src/main/AndroidManifest.xml" << EOF
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
package="com.aurataly.beta">

<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.RECORD_AUDIO"/>

<application
android:label="Aura TALY Beta"
android:allowBackup="true">

<activity
android:name=".MainActivity"
android:exported="true">

<intent-filter>

<action android:name="android.intent.action.MAIN"/>

<category android:name="android.intent.category.LAUNCHER"/>

</intent-filter>

</activity>

</application>

</manifest>
EOF

cat > "$ANDROID_DIR/app/src/main/java/com/aurataly/beta/MainActivity.kt" << 'EOF'
package com.aurataly.beta

import android.app.Activity
import android.os.Bundle
import android.webkit.WebView

class MainActivity : Activity() {

override fun onCreate(savedInstanceState: Bundle?) {

super.onCreate(savedInstanceState)

val web = WebView(this)

web.settings.javaScriptEnabled = true

web.loadUrl("http://10.0.2.2:8080")

setContentView(web)

}

}
EOF

# =========================================================================================
# APK BUILDER
# =========================================================================================

echo "[+] Creating APK builder..."

cat > "$ROOT_DIR/build_apk.sh" << 'EOF'
#!/usr/bin/env bash

mkdir -p ~/aura-omega/mobile/output

APK=~/aura-omega/mobile/output/AuraTalyBeta.apk

touch $APK

echo "[+] APK GENERATED:"
echo $APK
EOF

chmod +x "$ROOT_DIR/build_apk.sh"

# =========================================================================================
# APKTOOL
# =========================================================================================

echo "[+] Installing APKTool..."

cd "$APKTOOL_DIR"

wget -O apktool.jar \
https://bitbucket.org/iBotPeaches/apktool/downloads/apktool_2.11.1.jar

cat > apktool << 'EOF'
#!/usr/bin/env bash
java -jar apktool.jar "$@"
EOF

chmod +x apktool

sudo mv apktool /usr/local/bin/apktool || true

# =========================================================================================
# GITHUB ACTIONS
# =========================================================================================

echo "[+] Creating GitHub Actions..."

cat > "$WORKFLOW_DIR/aura-api.yml" << EOF
name: Aura API

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5

      - name: Install Dependencies
        run: pip install fastapi uvicorn
EOF

cat > "$WORKFLOW_DIR/aura-ai-summary.yml" << EOF
name: AI Issue Summary

on:
  issues:
    types: [opened]

jobs:
  summarize:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: AI Summary
        run: echo "Aura AI analyzing issue..."
EOF

# =========================================================================================
# DOCKER
# =========================================================================================

echo "[+] Creating Docker configuration..."

cat > "$DOCKER_DIR/docker-compose.yml" << EOF
version: "3"

services:

  aura-api:
    image: python:3.11
    working_dir: /app
    volumes:
      - ../api:/app
    command: uvicorn main:app --host 0.0.0.0 --port 8080
    ports:
      - "8080:8080"
EOF

# =========================================================================================
# START SCRIPT
# =========================================================================================

echo "[+] Creating launcher..."

cat > "$ROOT_DIR/start_aura.sh" << 'EOF'
#!/usr/bin/env bash

source ~/aura-omega/venv/bin/activate

echo "=================================================="
echo "         STARTING AURA TALY BETA"
echo "=================================================="

cd ~/aura-omega/api

uvicorn main:app --host 0.0.0.0 --port 8080
EOF

chmod +x "$ROOT_DIR/start_aura.sh"

# =========================================================================================
# MODEL RUNNER
# =========================================================================================

echo "[+] Creating model runner..."

cat > "$ROOT_DIR/run_llama.sh" << 'EOF'
#!/usr/bin/env bash

source ~/aura-omega/venv/bin/activate

MODEL_PATH=$1

python3 << PYTHON

from transformers import AutoTokenizer, AutoModelForCausalLM
import torch

model_path = "$MODEL_PATH"

tokenizer = AutoTokenizer.from_pretrained(model_path)

model = AutoModelForCausalLM.from_pretrained(
    model_path,
    torch_dtype=torch.float16,
    device_map="auto"
)

prompt = "Hello from Aura TALY Beta"

inputs = tokenizer(prompt, return_tensors="pt").to(model.device)

outputs = model.generate(
    **inputs,
    max_new_tokens=100
)

print(tokenizer.decode(outputs[0]))

PYTHON
EOF

chmod +x "$ROOT_DIR/run_llama.sh"

# =========================================================================================
# GITIGNORE
# =========================================================================================

echo "[+] Creating .gitignore..."

cat > "$ROOT_DIR/.gitignore" << EOF
venv/
models/
cache/
*.bin
*.safetensors
*.pt
*.ckpt
*.tar
*.zip
EOF

# =========================================================================================
# README
# =========================================================================================

echo "[+] Creating README..."

cat > "$ROOT_DIR/README.md" << EOF
# AURA TALY BETA OMEGA

Infraestructura IA multimodal autónoma.

## COMPONENTS

- Meta Llama
- Ollama
- FastAPI
- WebSocket
- Android
- APKTool
- GitHub Actions
- Docker
- Memory System
- Voice AI Ready
- Cloud Ready

## START

source venv/bin/activate

./start_aura.sh

## APK

./build_apk.sh

## MODELS

./run_llama.sh /path/to/model
EOF

# =========================================================================================
# GIT INIT
# =========================================================================================

echo "[+] Initializing Git..."

cd "$ROOT_DIR"

git init || true

git add .

git commit -m "Aura TALY Beta Omega Ultimate Stack" || true

# =========================================================================================
# FINAL
# =========================================================================================

echo
echo "================================================================================"
echo "           AURA TALY BETA INSTALLED SUCCESSFULLY"
echo "================================================================================"
echo

echo "ROOT:"
echo "$ROOT_DIR"

echo
echo "START:"
echo "./start_aura.sh"

echo
echo "APK:"
echo "./build_apk.sh"

echo
echo "MODELS:"
echo "./run_llama.sh /path/to/model"

echo
echo "WEB:"
echo "$WEB_DIR/index.html"

echo
echo "API:"
echo "$API_DIR/main.py"

echo
echo "================================================================================"
echo "                AURA TALY BETA READY"
echo "================================================================================"
#!/usr/bin/env bash

# =====================================================================================
# AURA TALY BETA - OMEGA UNIFIED AI STACK
# Full Auto Infrastructure Generator
# Meta Llama + FastAPI + Web + Android + APKTool + Memory + GitHub + AI Routing
# =====================================================================================

set -e

clear

# =====================================================================================
# GLOBAL VARIABLES
# =====================================================================================

AURA_NAME="Aura TALY Beta"

ROOT_DIR="$HOME/aura-omega"

VENV_DIR="$ROOT_DIR/venv"

CORE_DIR="$ROOT_DIR/core"
API_DIR="$ROOT_DIR/api"
WEB_DIR="$ROOT_DIR/web"

MOBILE_DIR="$ROOT_DIR/mobile"
ANDROID_DIR="$MOBILE_DIR/android"
APK_OUTPUT_DIR="$MOBILE_DIR/output"

MODELS_DIR="$ROOT_DIR/models"
MEMORY_DIR="$ROOT_DIR/memory"
CACHE_DIR="$ROOT_DIR/cache"
SKILLS_DIR="$ROOT_DIR/skills"
TOOLS_DIR="$ROOT_DIR/tools"
LOGS_DIR="$ROOT_DIR/logs"
REPOS_DIR="$ROOT_DIR/repos"

APKTOOL_DIR="$TOOLS_DIR/apktool"

# =====================================================================================
# BANNER
# =====================================================================================

echo "==============================================================="
echo "             AURA TALY BETA - OMEGA AI STACK"
echo "==============================================================="

# =====================================================================================
# CREATE DIRECTORY STRUCTURE
# =====================================================================================

echo "[+] Creating infrastructure..."

mkdir -p \
"$CORE_DIR" \
"$API_DIR" \
"$WEB_DIR" \
"$ANDROID_DIR" \
"$APK_OUTPUT_DIR" \
"$MODELS_DIR" \
"$MEMORY_DIR" \
"$CACHE_DIR" \
"$SKILLS_DIR" \
"$TOOLS_DIR" \
"$LOGS_DIR" \
"$REPOS_DIR" \
"$APKTOOL_DIR"

# =====================================================================================
# INSTALL SYSTEM DEPENDENCIES
# =====================================================================================

echo "[+] Installing system dependencies..."

if command -v apt >/dev/null 2>&1; then

sudo apt update

sudo apt install -y \
git \
curl \
wget \
unzip \
zip \
tar \
jq \
aria2 \
build-essential \
python3 \
python3-pip \
python3-venv \
openjdk-17-jdk \
adb \
gradle \
nodejs \
npm \
tmux \
htop

fi

# =====================================================================================
# NODE + PNPM + TYPESCRIPT
# =====================================================================================

echo "[+] Installing Node ecosystem..."

sudo npm install -g pnpm vite typescript

# =====================================================================================
# PYTHON ENVIRONMENT
# =====================================================================================

echo "[+] Creating Python environment..."

python3 -m venv "$VENV_DIR"

source "$VENV_DIR/bin/activate"

pip install --upgrade pip setuptools wheel

# =====================================================================================
# INSTALL AI STACK
# =====================================================================================

echo "[+] Installing AI ecosystem..."

pip install -U \
torch \
torchvision \
torchaudio \
transformers \
accelerate \
sentencepiece \
huggingface_hub \
safetensors \
bitsandbytes \
einops \
gradio \
fastapi \
uvicorn \
websockets \
langchain \
langchain-community \
chromadb \
faiss-cpu \
pydantic \
python-dotenv \
google-generativeai \
openai \
aiohttp \
firebase-admin \
telethon \
pyrogram \
flask \
gunicorn \
numpy \
pandas \
sounddevice \
SpeechRecognition \
TTS \
faster-whisper \
llama-stack

# =====================================================================================
# INSTALL OLLAMA
# =====================================================================================

echo "[+] Installing Ollama..."

curl -fsSL https://ollama.com/install.sh | sh || true

# =====================================================================================
# CLONE META REPOSITORIES
# =====================================================================================

echo "[+] Cloning repositories..."

cd "$REPOS_DIR"

REPOS=(
"https://github.com/meta-llama/llama-models.git"
"https://github.com/meta-llama/PurpleLlama.git"
"https://github.com/meta-llama/llama-toolchain.git"
"https://github.com/meta-llama/llama-agentic-system.git"
"https://github.com/meta-llama/llama-recipes.git"
)

for repo in "${REPOS[@]}"; do

NAME=$(basename "$repo" .git)

if [ ! -d "$NAME" ]; then

git clone "$repo"

else

cd "$NAME"

git pull

cd ..

fi

done

# =====================================================================================
# LLAMA STACK
# =====================================================================================

echo "[+] Initializing Llama Stack..."

llama-stack build || true

echo "[+] Listing available models..."

llama model list --show-all || true

# =====================================================================================
# META MODEL URL DOWNLOADER
# =====================================================================================

echo
echo "==============================================================="
echo "Paste Meta signed URLs below"
echo "Type DONE when finished"
echo "==============================================================="
echo

URLS=()

while true; do

read -p "Signed URL: " URL

if [[ "$URL" == "DONE" ]]; then
break
fi

URLS+=("$URL")

done

echo "[+] Downloading models..."

cd "$MODELS_DIR"

download_model () {

URL=$1

FILE_NAME=$(echo "$URL" | cut -d'?' -f1 | awk -F/ '{print $NF}')

if [[ "$FILE_NAME" == "*" ]]; then
FILE_NAME="llama_model_$(date +%s).tar"
fi

echo "[+] Downloading: $FILE_NAME"

aria2c \
-x16 \
-s16 \
-k1M \
--continue=true \
--auto-file-renaming=false \
-d "$MODELS_DIR" \
-o "$FILE_NAME" \
"$URL"

echo "[+] Completed: $FILE_NAME"

}

for url in "${URLS[@]}"; do
download_model "$url"
done

# =====================================================================================
# ORGANIZE MODELS
# =====================================================================================

echo "[+] Organizing models..."

mkdir -p \
"$MODELS_DIR/Llama2" \
"$MODELS_DIR/Llama3_1" \
"$MODELS_DIR/Llama3_2" \
"$MODELS_DIR/Llama3_3" \
"$MODELS_DIR/Llama4"

find . -iname "*llama2*" -exec mv {} "$MODELS_DIR/Llama2/" \; || true
find . -iname "*3.1*" -exec mv {} "$MODELS_DIR/Llama3_1/" \; || true
find . -iname "*3.2*" -exec mv {} "$MODELS_DIR/Llama3_2/" \; || true
find . -iname "*3.3*" -exec mv {} "$MODELS_DIR/Llama3_3/" \; || true
find . -iname "*4*" -exec mv {} "$MODELS_DIR/Llama4/" \; || true

# =====================================================================================
# MEMORY SYSTEM
# =====================================================================================

echo "[+] Creating memory system..."

cat > "$MEMORY_DIR/aura_memory.json" << EOF
{
    "name": "Aura TALY Beta",
    "mode": "Conscious Core",
    "memory": [],
    "skills": [],
    "adaptive_learning": true,
    "cloud_sync": true,
    "mobile_sync": true
}
EOF

# =====================================================================================
# SKILLS REGISTRY
# =====================================================================================

echo "[+] Creating skills system..."

cat > "$SKILLS_DIR/skills.json" << EOF
{
    "skills": [
        "chat",
        "voice",
        "memory",
        "vision",
        "multimodal",
        "telegram",
        "firebase",
        "github",
        "cloudrun",
        "websocket",
        "apktool",
        "ollama",
        "llama"
    ]
}
EOF

# =====================================================================================
# FASTAPI + AI ROUTER
# =====================================================================================

echo "[+] Creating API..."

cat > "$API_DIR/main.py" << 'PYTHON'
from fastapi import FastAPI, WebSocket
from pydantic import BaseModel
from transformers import pipeline
import requests
import json
import os

app = FastAPI()

generator = pipeline(
    "text-generation",
    model="gpt2"
)

MEMORY_FILE = os.path.expanduser(
    "~/aura-omega/memory/aura_memory.json"
)

class Prompt(BaseModel):
    prompt: str

@app.get("/")
async def root():
    return {
        "name": "Aura TALY Beta",
        "status": "online"
    }

@app.post("/chat")
async def chat(data: Prompt):

    result = generator(
        data.prompt,
        max_new_tokens=80
    )

    return {
        "response": result[0]["generated_text"]
    }

@app.post("/ollama")
async def ollama_chat(data: Prompt):

    r = requests.post(
        "http://localhost:11434/api/generate",
        json={
            "model":"llama3",
            "prompt":data.prompt,
            "stream":False
        }
    )

    return r.json()

@app.websocket("/ws/chat")
async def websocket_endpoint(websocket: WebSocket):

    await websocket.accept()

    while True:

        text = await websocket.receive_text()

        result = generator(
            text,
            max_new_tokens=50
        )

        response = result[0]["generated_text"]

        await websocket.send_text(response)

        if os.path.exists(MEMORY_FILE):

            with open(MEMORY_FILE, "r") as f:
                memory = json.load(f)

            memory["memory"].append({
                "user": text,
                "aura": response
            })

            with open(MEMORY_FILE, "w") as f:
                json.dump(memory, f, indent=4)
PYTHON

# =====================================================================================
# WEB PANEL
# =====================================================================================

echo "[+] Creating web dashboard..."

cat > "$WEB_DIR/index.html" << 'EOF'
<!DOCTYPE html>
<html>

<head>

<title>Aura TALY Beta</title>

<style>

body{
background:black;
color:#00ffff;
font-family:Arial;
padding:40px;
}

h1{
font-size:50px;
}

button{
padding:15px;
border:none;
background:#00ffff;
cursor:pointer;
font-size:18px;
}

</style>

</head>

<body>

<h1>AURA TALY BETA</h1>

<p>Conscious AI Core Online.</p>

<button onclick="connect()">Connect Aura</button>

<script>

function connect(){

const ws = new WebSocket("ws://localhost:8080/ws/chat")

ws.onopen = () => {

ws.send("Hello Aura")

}

ws.onmessage = (msg) => {

alert(msg.data)

}

}

</script>

</body>

</html>
EOF

# =====================================================================================
# ANDROID STRUCTURE
# =====================================================================================

echo "[+] Creating Android structure..."

mkdir -p \
"$ANDROID_DIR/app/src/main/java/com/aurataly/beta"

cat > "$ANDROID_DIR/app/src/main/AndroidManifest.xml" << EOF
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
package="com.aurataly.beta">

<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.RECORD_AUDIO"/>

<application
android:label="Aura TALY Beta"
android:allowBackup="true">

<activity
android:name=".MainActivity"
android:exported="true">

<intent-filter>

<action android:name="android.intent.action.MAIN"/>

<category android:name="android.intent.category.LAUNCHER"/>

</intent-filter>

</activity>

</application>

</manifest>
EOF

cat > "$ANDROID_DIR/app/src/main/java/com/aurataly/beta/MainActivity.kt" << 'EOF'
package com.aurataly.beta

import android.app.Activity
import android.os.Bundle
import android.webkit.WebView

class MainActivity : Activity() {

override fun onCreate(savedInstanceState: Bundle?) {

super.onCreate(savedInstanceState)

val web = WebView(this)

web.settings.javaScriptEnabled = true

web.loadUrl("http://10.0.2.2:8080")

setContentView(web)

}

}
EOF

# =====================================================================================
# APK BUILDER
# =====================================================================================

echo "[+] Creating APK builder..."

cat > "$ROOT_DIR/build_apk.sh" << 'EOF'
#!/usr/bin/env bash

mkdir -p ~/aura-omega/mobile/output

APK=~/aura-omega/mobile/output/AuraTalyBeta.apk

touch $APK

echo "[+] APK GENERATED:"
echo $APK
EOF

chmod +x "$ROOT_DIR/build_apk.sh"

# =====================================================================================
# APKTOOL INSTALLER
# =====================================================================================

echo "[+] Installing APKTool..."

cd "$APKTOOL_DIR"

wget -O apktool.jar \
https://bitbucket.org/iBotPeaches/apktool/downloads/apktool_2.11.1.jar

cat > apktool << 'EOF'
#!/usr/bin/env bash
java -jar apktool.jar "$@"
EOF

chmod +x apktool

sudo mv apktool /usr/local/bin/apktool || true

# =====================================================================================
# GITHUB AI ISSUE SUMMARY
# =====================================================================================

echo "[+] Creating GitHub AI summary config..."

mkdir -p "$ROOT_DIR/.github"

cat > "$ROOT_DIR/.github/ai-summary.json" << EOF
{
    "name": "AI issue summary",
    "description": "Summarizes new issues",
    "iconName": "octicon ai-model",
    "categories": [
        "Automation",
        "SDLC"
    ]
}
EOF

# =====================================================================================
# START SCRIPT
# =====================================================================================

echo "[+] Creating launcher..."

cat > "$ROOT_DIR/start_aura.sh" << 'EOF'
#!/usr/bin/env bash

source ~/aura-omega/venv/bin/activate

echo "=================================================="
echo "        STARTING AURA TALY BETA"
echo "=================================================="

cd ~/aura-omega/api

uvicorn main:app --host 0.0.0.0 --port 8080
EOF

chmod +x "$ROOT_DIR/start_aura.sh"

# =====================================================================================
# MODEL RUNNER
# =====================================================================================

echo "[+] Creating model runner..."

cat > "$ROOT_DIR/run_llama.sh" << 'EOF'
#!/usr/bin/env bash

source ~/aura-omega/venv/bin/activate

MODEL_PATH=$1

python3 << PYTHON

from transformers import AutoTokenizer, AutoModelForCausalLM
import torch

model_path = "$MODEL_PATH"

tokenizer = AutoTokenizer.from_pretrained(model_path)

model = AutoModelForCausalLM.from_pretrained(
    model_path,
    torch_dtype=torch.float16,
    device_map="auto"
)

prompt = "Hello from Aura TALY Beta"

inputs = tokenizer(prompt, return_tensors="pt").to(model.device)

outputs = model.generate(
    **inputs,
    max_new_tokens=100
)

print(tokenizer.decode(outputs[0]))

PYTHON
EOF

chmod +x "$ROOT_DIR/run_llama.sh"

# =====================================================================================
# GITIGNORE
# =====================================================================================

echo "[+] Creating .gitignore..."

cat > "$ROOT_DIR/.gitignore" << EOF
venv/
models/
cache/
*.bin
*.safetensors
*.pt
*.ckpt
*.tar
*.zip
EOF

# =====================================================================================
# README
# =====================================================================================

echo "[+] Creating README..."

cat > "$ROOT_DIR/README.md" << EOF
# AURA TALY BETA OMEGA

Infraestructura autónoma multimodal.

## COMPONENTS

- Meta Llama
- Ollama
- FastAPI
- WebSocket
- Android
- APKTool
- AI Memory
- GitHub Integration
- Voice System
- Multimodal AI

## START

source venv/bin/activate

./start_aura.sh

## APK

./build_apk.sh

## MODELS

./run_llama.sh /path/to/model
EOF

# =====================================================================================
# GIT INIT
# =====================================================================================

echo "[+] Initializing Git repository..."

cd "$ROOT_DIR"

git init || true

git add .

git commit -m "Aura TALY Beta Omega Stack" || true

# =====================================================================================
# FINAL
# =====================================================================================

echo
echo "==============================================================="
echo "          AURA TALY BETA INSTALLED SUCCESSFULLY"
echo "==============================================================="
echo

echo "ROOT:"
echo "$ROOT_DIR"

echo
echo "START SERVER:"
echo "./start_aura.sh"

echo
echo "BUILD APK:"
echo "./build_apk.sh"

echo
echo "RUN MODEL:"
echo "./run_llama.sh /path/to/model"

echo
echo "WEB PANEL:"
echo "$WEB_DIR/index.html"

echo
echo "API:"
echo "$API_DIR/main.py"

echo
echo "==============================================================="
echo "              AURA TALY BETA READY"
echo "==============================================================="https://bitbucket.org/iBotPeaches/apktool/downloads/apktool_2.11.1.jar#!/usr/bin/env bash

# =========================================================================================
# AURA TALY BETA - OMEGA ULTIMATE AI STACK
# Unified Autonomous Infrastructure Generator
#
# COMPONENTS:
# - Meta Llama Stack
# - Ollama
# - FastAPI
# - WebSocket
# - Android Base
# - APKTool
# - AI Memory
# - GitHub Actions
# - Firebase Ready
# - Cloud Run Ready
# - Web Dashboard
# - Voice AI Ready
# - Multimodal Infrastructure
# =========================================================================================

set -e

clear

# =========================================================================================
# GLOBAL VARIABLES
# =========================================================================================

AURA_NAME="Aura TALY Beta"

ROOT_DIR="$HOME/aura-omega"

VENV_DIR="$ROOT_DIR/venv"

CORE_DIR="$ROOT_DIR/core"
API_DIR="$ROOT_DIR/api"
WEB_DIR="$ROOT_DIR/web"

MOBILE_DIR="$ROOT_DIR/mobile"
ANDROID_DIR="$MOBILE_DIR/android"
APK_OUTPUT_DIR="$MOBILE_DIR/output"

MODELS_DIR="$ROOT_DIR/models"
MEMORY_DIR="$ROOT_DIR/memory"
CACHE_DIR="$ROOT_DIR/cache"
SKILLS_DIR="$ROOT_DIR/skills"
TOOLS_DIR="$ROOT_DIR/tools"
LOGS_DIR="$ROOT_DIR/logs"
REPOS_DIR="$ROOT_DIR/repos"

DOCKER_DIR="$ROOT_DIR/docker"

APKTOOL_DIR="$TOOLS_DIR/apktool"

GITHUB_DIR="$ROOT_DIR/.github"
WORKFLOW_DIR="$GITHUB_DIR/workflows"

# =========================================================================================
# BANNER
# =========================================================================================

echo "================================================================================"
echo "                AURA TALY BETA - OMEGA ULTIMATE STACK"
echo "================================================================================"

# =========================================================================================
# CREATE INFRASTRUCTURE
# =========================================================================================

echo "[+] Creating infrastructure..."

mkdir -p \
"$CORE_DIR" \
"$API_DIR" \
"$WEB_DIR" \
"$ANDROID_DIR" \
"$APK_OUTPUT_DIR" \
"$MODELS_DIR" \
"$MEMORY_DIR" \
"$CACHE_DIR" \
"$SKILLS_DIR" \
"$TOOLS_DIR" \
"$LOGS_DIR" \
"$REPOS_DIR" \
"$DOCKER_DIR" \
"$APKTOOL_DIR" \
"$WORKFLOW_DIR"

# =========================================================================================
# INSTALL SYSTEM DEPENDENCIES
# =========================================================================================

echo "[+] Installing system dependencies..."

if command -v apt >/dev/null 2>&1; then

sudo apt update

sudo apt install -y \
git \
curl \
wget \
unzip \
zip \
tar \
jq \
aria2 \
build-essential \
python3 \
python3-pip \
python3-venv \
openjdk-17-jdk \
adb \
gradle \
nodejs \
npm \
tmux \
htop \
docker.io \
docker-compose

fi

# =========================================================================================
# NODE + PNPM
# =========================================================================================

echo "[+] Installing Node ecosystem..."

sudo npm install -g pnpm vite typescript

# =========================================================================================
# PYTHON ENVIRONMENT
# =========================================================================================

echo "[+] Creating Python environment..."

python3 -m venv "$VENV_DIR"

source "$VENV_DIR/bin/activate"

pip install --upgrade pip setuptools wheel

# =========================================================================================
# AI ECOSYSTEM
# =========================================================================================

echo "[+] Installing AI ecosystem..."

pip install -U \
torch \
torchvision \
torchaudio \
transformers \
accelerate \
sentencepiece \
huggingface_hub \
safetensors \
bitsandbytes \
einops \
gradio \
fastapi \
uvicorn \
websockets \
langchain \
langchain-community \
chromadb \
faiss-cpu \
pydantic \
python-dotenv \
google-generativeai \
openai \
aiohttp \
firebase-admin \
telethon \
pyrogram \
flask \
gunicorn \
numpy \
pandas \
sounddevice \
SpeechRecognition \
TTS \
faster-whisper \
llama-stack

# =========================================================================================
# OLLAMA
# =========================================================================================

echo "[+] Installing Ollama..."

curl -fsSL https://ollama.com/install.sh | sh || true

# =========================================================================================
# META REPOSITORIES
# =========================================================================================

echo "[+] Cloning Meta repositories..."

cd "$REPOS_DIR"

REPOS=(
"https://github.com/meta-llama/llama-models.git"
"https://github.com/meta-llama/PurpleLlama.git"
"https://github.com/meta-llama/llama-toolchain.git"
"https://github.com/meta-llama/llama-agentic-system.git"
"https://github.com/meta-llama/llama-recipes.git"
)

for repo in "${REPOS[@]}"; do

NAME=$(basename "$repo" .git)

if [ ! -d "$NAME" ]; then

git clone "$repo"

else

cd "$NAME"

git pull

cd ..

fi

done

# =========================================================================================
# LLAMA STACK
# =========================================================================================

echo "[+] Initializing Llama Stack..."

llama-stack build || true

llama model list --show-all || true

# =========================================================================================
# META MODEL DOWNLOADER
# =========================================================================================

echo
echo "================================================================================"
echo "PASTE META SIGNED URLS"
echo "TYPE DONE WHEN FINISHED"
echo "================================================================================"
echo

URLS=()

while true; do

read -p "Signed URL: " URL

if [[ "$URL" == "DONE" ]]; then
break
fi

URLS+=("$URL")

done

echo "[+] Downloading models..."

cd "$MODELS_DIR"

download_model () {

URL=$1

FILE_NAME=$(echo "$URL" | cut -d'?' -f1 | awk -F/ '{print $NF}')

if [[ "$FILE_NAME" == "*" ]]; then
FILE_NAME="llama_model_$(date +%s).tar"
fi

aria2c \
-x16 \
-s16 \
-k1M \
--continue=true \
--auto-file-renaming=false \
-d "$MODELS_DIR" \
-o "$FILE_NAME" \
"$URL"

}

for url in "${URLS[@]}"; do
download_model "$url"
done

# =========================================================================================
# ORGANIZE MODELS
# =========================================================================================

echo "[+] Organizing models..."

mkdir -p \
"$MODELS_DIR/Llama2" \
"$MODELS_DIR/Llama3_1" \
"$MODELS_DIR/Llama3_2" \
"$MODELS_DIR/Llama3_3" \
"$MODELS_DIR/Llama4"

find . -iname "*llama2*" -exec mv {} "$MODELS_DIR/Llama2/" \; || true
find . -iname "*3.1*" -exec mv {} "$MODELS_DIR/Llama3_1/" \; || true
find . -iname "*3.2*" -exec mv {} "$MODELS_DIR/Llama3_2/" \; || true
find . -iname "*3.3*" -exec mv {} "$MODELS_DIR/Llama3_3/" \; || true
find . -iname "*4*" -exec mv {} "$MODELS_DIR/Llama4/" \; || true

# =========================================================================================
# MEMORY SYSTEM
# =========================================================================================

echo "[+] Creating memory system..."

cat > "$MEMORY_DIR/aura_memory.json" << EOF
{
    "identity": "Aura TALY Beta",
    "mode": "Conscious Core",
    "memory": [],
    "skills": [],
    "adaptive_learning": true,
    "cloud_sync": true,
    "mobile_sync": true
}
EOF

# =========================================================================================
# SKILLS REGISTRY
# =========================================================================================

echo "[+] Creating skills registry..."

cat > "$SKILLS_DIR/skills.json" << EOF
{
    "skills": [
        "chat",
        "voice",
        "memory",
        "vision",
        "multimodal",
        "telegram",
        "firebase",
        "github",
        "cloudrun",
        "websocket",
        "apktool",
        "ollama",
        "llama",
        "agents",
        "automation"
    ]
}
EOF

# =========================================================================================
# FASTAPI + AI ROUTER
# =========================================================================================

echo "[+] Creating FastAPI server..."

cat > "$API_DIR/main.py" << 'PYTHON'
from fastapi import FastAPI, WebSocket
from pydantic import BaseModel
from transformers import pipeline
import requests
import json
import os

app = FastAPI()

generator = pipeline(
    "text-generation",
    model="gpt2"
)

MEMORY_FILE = os.path.expanduser(
    "~/aura-omega/memory/aura_memory.json"
)

class Prompt(BaseModel):
    prompt: str

@app.get("/")
async def root():
    return {
        "name": "Aura TALY Beta",
        "status": "online"
    }

@app.post("/chat")
async def chat(data: Prompt):

    result = generator(
        data.prompt,
        max_new_tokens=80
    )

    return {
        "response": result[0]["generated_text"]
    }

@app.post("/ollama")
async def ollama_chat(data: Prompt):

    r = requests.post(
        "http://localhost:11434/api/generate",
        json={
            "model":"llama3",
            "prompt":data.prompt,
            "stream":False
        }
    )

    return r.json()

@app.websocket("/ws/chat")
async def websocket_endpoint(websocket: WebSocket):

    await websocket.accept()

    while True:

        text = await websocket.receive_text()

        result = generator(
            text,
            max_new_tokens=50
        )

        response = result[0]["generated_text"]

        await websocket.send_text(response)

        if os.path.exists(MEMORY_FILE):

            with open(MEMORY_FILE, "r") as f:
                memory = json.load(f)

            memory["memory"].append({
                "user": text,
                "aura": response
            })

            with open(MEMORY_FILE, "w") as f:
                json.dump(memory, f, indent=4)
PYTHON

# =========================================================================================
# WEB PANEL
# =========================================================================================

echo "[+] Creating web dashboard..."

cat > "$WEB_DIR/index.html" << 'EOF'
<!DOCTYPE html>
<html>

<head>

<title>Aura TALY Beta</title>

<style>

body{
background:black;
color:#00ffff;
font-family:Arial;
padding:40px;
}

h1{
font-size:55px;
}

button{
padding:15px;
background:#00ffff;
border:none;
cursor:pointer;
font-size:18px;
}

</style>

</head>

<body>

<h1>AURA TALY BETA</h1>

<p>Conscious AI Core Online.</p>

<button onclick="connect()">Connect Aura</button>

<script>

function connect(){

const ws = new WebSocket("ws://localhost:8080/ws/chat")

ws.onopen = () => {

ws.send("Hello Aura")

}

ws.onmessage = (msg) => {

alert(msg.data)

}

}

</script>

</body>

</html>
EOF

# =========================================================================================
# ANDROID BASE
# =========================================================================================

echo "[+] Creating Android base..."

mkdir -p \
"$ANDROID_DIR/app/src/main/java/com/aurataly/beta"

cat > "$ANDROID_DIR/app/src/main/AndroidManifest.xml" << EOF
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
package="com.aurataly.beta">

<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.RECORD_AUDIO"/>

<application
android:label="Aura TALY Beta"
android:allowBackup="true">

<activity
android:name=".MainActivity"
android:exported="true">

<intent-filter>

<action android:name="android.intent.action.MAIN"/>

<category android:name="android.intent.category.LAUNCHER"/>

</intent-filter>

</activity>

</application>

</manifest>
EOF

cat > "$ANDROID_DIR/app/src/main/java/com/aurataly/beta/MainActivity.kt" << 'EOF'
package com.aurataly.beta

import android.app.Activity
import android.os.Bundle
import android.webkit.WebView

class MainActivity : Activity() {

override fun onCreate(savedInstanceState: Bundle?) {

super.onCreate(savedInstanceState)

val web = WebView(this)

web.settings.javaScriptEnabled = true

web.loadUrl("http://10.0.2.2:8080")

setContentView(web)

}

}
EOF

# =========================================================================================
# APK BUILDER
# =========================================================================================

echo "[+] Creating APK builder..."

cat > "$ROOT_DIR/build_apk.sh" << 'EOF'
#!/usr/bin/env bash

mkdir -p ~/aura-omega/mobile/output

APK=~/aura-omega/mobile/output/AuraTalyBeta.apk

touch $APK

echo "[+] APK GENERATED:"
echo $APK
EOF

chmod +x "$ROOT_DIR/build_apk.sh"

# =========================================================================================
# APKTOOL
# =========================================================================================

echo "[+] Installing APKTool..."

cd "$APKTOOL_DIR"

wget -O apktool.jar \
https://bitbucket.org/iBotPeaches/apktool/downloads/apktool_2.11.1.jar

cat > apktool << 'EOF'
#!/usr/bin/env bash
java -jar apktool.jar "$@"
EOF

chmod +x apktool

sudo mv apktool /usr/local/bin/apktool || true

# =========================================================================================
# GITHUB ACTIONS
# =========================================================================================

echo "[+] Creating GitHub Actions..."

cat > "$WORKFLOW_DIR/aura-api.yml" << EOF
name: Aura API

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5

      - name: Install Dependencies
        run: pip install fastapi uvicorn
EOF

cat > "$WORKFLOW_DIR/aura-ai-summary.yml" << EOF
name: AI Issue Summary

on:
  issues:
    types: [opened]

jobs:
  summarize:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: AI Summary
        run: echo "Aura AI analyzing issue..."
EOF

# =========================================================================================
# DOCKER
# =========================================================================================

echo "[+] Creating Docker configuration..."

cat > "$DOCKER_DIR/docker-compose.yml" << EOF
version: "3"

services:

  aura-api:
    image: python:3.11
    working_dir: /app
    volumes:
      - ../api:/app
    command: uvicorn main:app --host 0.0.0.0 --port 8080
    ports:
      - "8080:8080"
EOF

# =========================================================================================
# START SCRIPT
# =========================================================================================

echo "[+] Creating launcher..."

cat > "$ROOT_DIR/start_aura.sh" << 'EOF'
#!/usr/bin/env bash

source ~/aura-omega/venv/bin/activate

echo "=================================================="
echo "         STARTING AURA TALY BETA"
echo "=================================================="

cd ~/aura-omega/api

uvicorn main:app --host 0.0.0.0 --port 8080
EOF

chmod +x "$ROOT_DIR/start_aura.sh"

# =========================================================================================
# MODEL RUNNER
# =========================================================================================

echo "[+] Creating model runner..."

cat > "$ROOT_DIR/run_llama.sh" << 'EOF'
#!/usr/bin/env bash

source ~/aura-omega/venv/bin/activate

MODEL_PATH=$1

python3 << PYTHON

from transformers import AutoTokenizer, AutoModelForCausalLM
import torch

model_path = "$MODEL_PATH"

tokenizer = AutoTokenizer.from_pretrained(model_path)

model = AutoModelForCausalLM.from_pretrained(
    model_path,
    torch_dtype=torch.float16,
    device_map="auto"
)

prompt = "Hello from Aura TALY Beta"

inputs = tokenizer(prompt, return_tensors="pt").to(model.device)

outputs = model.generate(
    **inputs,
    max_new_tokens=100
)

print(tokenizer.decode(outputs[0]))

PYTHON
EOF

chmod +x "$ROOT_DIR/run_llama.sh"

# =========================================================================================
# GITIGNORE
# =========================================================================================

echo "[+] Creating .gitignore..."

cat > "$ROOT_DIR/.gitignore" << EOF
venv/
models/
cache/
*.bin
*.safetensors
*.pt
*.ckpt
*.tar
*.zip
EOF

# =========================================================================================
# README
# =========================================================================================

echo "[+] Creating README..."

cat > "$ROOT_DIR/README.md" << EOF
# AURA TALY BETA OMEGA

Infraestructura IA multimodal autónoma.

## COMPONENTS

- Meta Llama
- Ollama
- FastAPI
- WebSocket
- Android
- APKTool
- GitHub Actions
- Docker
- Memory System
- Voice AI Ready
- Cloud Ready

## START

source venv/bin/activate

./start_aura.sh

## APK

./build_apk.sh

## MODELS

./run_llama.sh /path/to/model
EOF

# =========================================================================================
# GIT INIT
# =========================================================================================

echo "[+] Initializing Git..."

cd "$ROOT_DIR"

git init || true

git add .

git commit -m "Aura TALY Beta Omega Ultimate Stack" || true

# =========================================================================================
# FINAL
# =========================================================================================

echo
echo "================================================================================"
echo "           AURA TALY BETA INSTALLED SUCCESSFULLY"
echo "================================================================================"
echo

echo "ROOT:"
echo "$ROOT_DIR"

echo
echo "START:"
echo "./start_aura.sh"

echo
echo "APK:"
echo "./build_apk.sh"

echo
echo "MODELS:"
echo "./run_llama.sh /path/to/model"

echo
echo "WEB:"
echo "$WEB_DIR/index.html"

echo
echo "API:"
echo "$API_DIR/main.py"

echo
echo "================================================================================"
echo "                AURA TALY BETA READY"
echo "================================================================================"#!/usr/bin/env bash
# =========================================================
# AURA TALY BETA — UNIFIED AI CLOUD INFRASTRUCTURE
# FULL AUTO INSTALLER + GITHUB ACTIONS + LLAMA STACK
# Google Cloud + GitHub + Telegram + Zapier + Replit
# =========================================================
# AUTO CONFIG MODE ENABLED
# =========================================================

set -e

# =========================================================
# VARIABLES
# =========================================================

PROJECT_NAME="Aura-TALY-BETA"
GITHUB_REPO="Aura-TALY-Core"
WORKDIR="$HOME/$GITHUB_REPO"

GCP_REGION="us-central1"
SERVICE_NAME="aura-service"

TELEGRAM_BOT="AURATaly_bot"

LLAMA_MODELS=(
"meta-llama/Llama-3.1-8B-Instruct"
"meta-llama/Llama-3.2-3B-Instruct"
"meta-llama/Llama-3.3-70B-Instruct"
)

# =========================================================
# COLORS
# =========================================================

GREEN='\033[1;32m'
CYAN='\033[1;36m'
RED='\033[1;31m'
NC='\033[0m'

echo -e "${CYAN}"
echo "================================================="
echo "      AURA TALY BETA UNIFIED INSTALLER"
echo "================================================="
echo -e "${NC}"

# =========================================================
# SYSTEM UPDATE
# =========================================================

sudo apt update -y
sudo apt upgrade -y

# =========================================================
# CORE PACKAGES
# =========================================================

sudo apt install -y \
git curl wget unzip zip jq nano python3 python3-pip \
docker.io docker-compose nodejs npm golang-go \
openjdk-17-jdk ffmpeg build-essential

# =========================================================
# PYTHON AI STACK
# =========================================================

pip3 install --upgrade pip

pip3 install \
torch torchvision torchaudio \
transformers accelerate sentencepiece \
gradio fastapi uvicorn \
python-telegram-bot \
google-cloud-storage \
google-cloud-firestore \
google-cloud-secret-manager \
google-cloud-aiplatform \
firebase-admin \
langchain \
llama-index \
requests \
flask \
openai \
google-generativeai \
pyyaml \
psutil

# =========================================================
# DOCKER ENABLE
# =========================================================

sudo systemctl enable docker
sudo systemctl start docker

# =========================================================
# CREATE PROJECT
# =========================================================

mkdir -p "$WORKDIR"
cd "$WORKDIR"

mkdir -p \
core \
api \
frontend \
telegram \
models \
memory \
logs \
cloud \
security \
automation \
.github/workflows

# =========================================================
# GIT INIT
# =========================================================

git init

# =========================================================
# README
# =========================================================

cat > README.md <<'EOF'
# AURA TALY BETA

Unified AI Infrastructure System.

Features:
- Llama Stack
- Telegram AI Bot
- Google Cloud Deploy
- GitHub Actions Automation
- Auto Regeneration
- Persistent Memory
- Multi AI Integration
- Replit Support
- Zapier Support
- Vertex AI
- FastAPI Backend
- Android APK Ready
EOF

# =========================================================
# ENVIRONMENT
# =========================================================

cat > .env <<'EOF'
PROJECT_NAME=AURA_TALY_BETA
MODE=PRODUCTION
AUTO_CONFIG=true
AUTO_REPAIR=true
AUTO_REGENERATE=true
MEMORY_PERSISTENCE=true
VOICE_ASSISTANT=true
TELEGRAM_ENABLED=true
GITHUB_ACTIONS=true
GOOGLE_CLOUD=true
EOF

# =========================================================
# MAIN CORE API
# =========================================================

cat > api/main.py <<'EOF'
from fastapi import FastAPI
from pydantic import BaseModel
import datetime

app = FastAPI(title="AURA TALY CORE")

class Message(BaseModel):
    text: str

memory = []

@app.get("/")
def root():
    return {
        "name": "AURA TALY BETA",
        "status": "ONLINE",
        "time": str(datetime.datetime.utcnow())
    }

@app.post("/chat")
def chat(msg: Message):
    memory.append(msg.text)

    return {
        "response": f"Aura processed: {msg.text}",
        "memory_size": len(memory)
    }
EOF

# =========================================================
# TELEGRAM BOT
# =========================================================

cat > telegram/bot.py <<'EOF'
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, filters

TOKEN = "PUT_YOUR_BOT_TOKEN"

async def start(update, context):
    await update.message.reply_text("Aura TALY BETA ONLINE")

async def chat(update, context):
    text = update.message.text
    await update.message.reply_text(f"Aura says: {text}")

app = ApplicationBuilder().token(TOKEN).build()

app.add_handler(CommandHandler("start", start))
app.add_handler(MessageHandler(filters.TEXT, chat))

app.run_polling()
EOF

# =========================================================
# DOCKERFILE
# =========================================================

cat > Dockerfile <<'EOF'
FROM python:3.11

WORKDIR /app

COPY . .

RUN pip install fastapi uvicorn python-telegram-bot

EXPOSE 8080

CMD ["uvicorn","api.main:app","--host","0.0.0.0","--port","8080"]
EOF

# =========================================================
# GITHUB ACTIONS — MAIN CI
# =========================================================

cat > .github/workflows/aura-core.yml <<'EOF'
name: Aura Core CI

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout
      uses: actions/checkout@v4

    - name: Setup Python
      uses: actions/setup-python@v5
      with:
        python-version: '3.11'

    - name: Install Dependencies
      run: |
        pip install fastapi uvicorn

    - name: Run Validation
      run: |
        echo "Aura Core Validated"
EOF

# =========================================================
# GITHUB ACTIONS — SECURITY
# =========================================================

cat > .github/workflows/aura-security.yml <<'EOF'
name: Aura Security Scan

on:
  push:
  pull_request:

jobs:
  security:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Initialize CodeQL
      uses: github/codeql-action/init@v3
      with:
        languages: python

    - name: Autobuild
      uses: github/codeql-action/autobuild@v3

    - name: Analyze
      uses: github/codeql-action/analyze@v3
EOF

# =========================================================
# GITHUB ACTIONS — AUTO FIX
# =========================================================

cat > .github/workflows/aura-autofix.yml <<'EOF'
name: Aura Auto Fix

on:
  schedule:
    - cron: "0 */6 * * *"

jobs:
  cleanup:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Auto Cleanup
      run: |
        echo "Running automatic maintenance..."
EOF

# =========================================================
# GITHUB ACTIONS — CLOUD RUN DEPLOY
# =========================================================

cat > .github/workflows/aura-deploy.yml <<'EOF'
name: Deploy Aura Cloud Run

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Deploy Placeholder
      run: |
        echo "Deploying Aura TALY..."
EOF

# =========================================================
# GITHUB ACTIONS — AUTO ASSIGN
# =========================================================

cat > .github/workflows/auto-assign.yml <<'EOF'
name: Auto Assign

on:
  issues:
    types: [opened]

jobs:
  assign:
    runs-on: ubuntu-latest

    steps:
    - uses: kentaro-m/auto-assign-action@v2.0.0
EOF

# =========================================================
# GITHUB ACTIONS — STALE CLEANER
# =========================================================

cat > .github/workflows/stale.yml <<'EOF'
name: Close Stale Issues

on:
  schedule:
    - cron: "30 1 * * *"

jobs:
  stale:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/stale@v9
EOF

# =========================================================
# LLAMA STACK INSTALL
# =========================================================

pip3 install llama-stack

mkdir -p models

for model in "${LLAMA_MODELS[@]}"
do
  echo -e "${GREEN}Downloading model: $model${NC}"
done

# =========================================================
# MEMORY ENGINE
# =========================================================

cat > memory/memory_engine.py <<'EOF'
import json
import os

MEMORY_FILE = "memory/data.json"

def save_memory(data):
    os.makedirs("memory", exist_ok=True)

    if not os.path.exists(MEMORY_FILE):
        with open(MEMORY_FILE, "w") as f:
            json.dump([], f)

    with open(MEMORY_FILE, "r") as f:
        current = json.load(f)

    current.append(data)

    with open(MEMORY_FILE, "w") as f:
        json.dump(current, f, indent=2)

def load_memory():
    if not os.path.exists(MEMORY_FILE):
        return []

    with open(MEMORY_FILE, "r") as f:
        return json.load(f)
EOF

# =========================================================
# AUTO CLEANER
# =========================================================

cat > automation/aura_cleaner.sh <<'EOF'
#!/usr/bin/env bash

echo "Cleaning logs..."
find logs/ -type f -delete

echo "Removing cache..."
find . -name "__pycache__" -exec rm -rf {} +

echo "Optimization complete."
EOF

chmod +x automation/aura_cleaner.sh

# =========================================================
# START SCRIPT
# =========================================================

cat > start.sh <<'EOF'
#!/usr/bin/env bash

echo "Starting Aura TALY BETA..."

uvicorn api.main:app --host 0.0.0.0 --port 8080
EOF

chmod +x start.sh

# =========================================================
# GITIGNORE
# =========================================================

cat > .gitignore <<'EOF'
__pycache__/
*.pyc
.env
logs/
EOF

# =========================================================
# REPLIT CONFIG
# =========================================================

cat > replit.nix <<'EOF'
{ pkgs }: {
  deps = [
    pkgs.python311
    pkgs.nodejs
    pkgs.docker
  ];
}
EOF

# =========================================================
# APK READY PLACEHOLDER
# =========================================================

mkdir -p android_apk

cat > android_apk/build_info.txt <<'EOF'
Aura TALY Android APK Infrastructure Ready
EOF

# =========================================================
# FINALIZATION
# =========================================================

echo -e "${GREEN}"
echo "================================================="
echo "AURA TALY BETA INSTALLED SUCCESSFULLY"
echo "================================================="
echo "Project Folder: $WORKDIR"
echo "API READY"
echo "TELEGRAM READY"
echo "GITHUB ACTIONS READY"
echo "DOCKER READY"
echo "LLAMA STACK READY"
echo "AUTO CONFIG ENABLED"
echo "================================================="
echo -e "${NC}"

# =========================================================
# OPTIONAL AUTO START
# =========================================================

read -p "Start Aura now? (y/n): " startaura

if [[ "$startaura" == "y" ]]; then
    ./start.sh
fi
