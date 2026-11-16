# Runbook 05: Local LLM Server - Installation & Operations

## Installation Notes

* **Tool Used to Flash USB:** Balena Etcher
* **OS Drive/Target Drive:** 256 GB SSD

* **Network Setup:**
  * Static IP assigned: 192.168.2.209
  * CIDR notation: 24
  * Gateway: 192.168.2.1
  * DNS: 1.1.1.1
  * Hostname(FQND): llm-server.local

* **Filesystem:** ext4

* **OS** Debian 13

### Fixed Sudo user issue
```bash
su -
```
```bash
usermod -aG sudo civtact
```
* Restart the Server/log out for effects to take action

### System Update

```bash
apt update && apt upgrade -y
```
## Install certain important packages:
```bash
sudo apt install -y \
  git \
  curl \
  wget \
  build-essential \
  cmake \
  pkg-config \
  python3 \
  python3-pip \
  python3-venv \
  pipx \
  pciutils \
  usbutils \
  intel-gpu-tools \
  mesa-vulkan-drivers \
  libvulkan-dev \
  vulkan-tools \
  glslc \
  spirv-headers
  ```

  ### Verify the actual Hardware and Vulkan environment
  ```bash 
cat /etc/debian_version
uname -r
hostnamectl
lspci -nnk | grep -A4 -Ei 'VGA|Display|3D'
ls -l /dev/dri/
groups
vulkaninfo --summary
```

## Install and compile llama.cpp with Vulkan
```bash
sudo git clone https://github.com/ggml-org/llama.cpp.git /opt/llama.cpp
sudo chown -R "$USER":"$USER" /opt/llama.cpp
```
### Configure the build

```bash
cd /opt/llama.cpp

cmake -B build \
  -DGGML_VULKAN=ON \
  -DCMAKE_BUILD_TYPE=Release
```

### Compile the Build
```bash
cmake --build build --config Release -j2

``` 

## Verify the installation
```bash
/opt/llama.cpp/build/bin/llama-cli --version
/opt/llama.cpp/build/bin/llama-cli --list-devices
ls -lh /opt/llama.cpp/build/bin/llama-*
```

## Prepare the model storage
```bash
sudo mkdir -p /srv/llm/models
sudo mkdir -p /srv/llm/logs
sudo mkdir -p /srv/llm/projects
sudo chown -R civtact:civtact /srv/llm

df -h /srv/llm
```

### Install Hugging Face CLI
```bash
sudo apt install -y pipx
pipx ensurepath
pipx install huggingface_hub
```

# Download you LLM 
* for this project qwen3.5-9b q8_0 was picked
```bash
hf download bartowski/Qwen_Qwen3.5-9B-GGUF \
  Qwen3.5-9B-Q8_0.gguf \
  --local-dir /srv/llm/models
  ```

### Test it out!
```bash
/opt/llama.cpp/build/bin/llama-cli \
  -m /srv/llm/models/Qwen_Qwen3.5-9B-Q8_0.gguf \
  -ngl all \
  -c 262144 \
  -n 2048 \
  -fa on \
  -ctk q8_0 \
  -ctv q8_0 \
  --reasoning on \
  --reasoning-effort low
```
