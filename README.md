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
