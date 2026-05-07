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