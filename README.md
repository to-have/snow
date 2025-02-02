
# SNOW: Stable Diff + Nvidia + Ollama + WebUI

Simple docker compose setup for Linux/WSL(not tested) + Nvidia GPU/s

## Prerequisites:
- Docker
- Nvidia Container Toolkit (https://hub.docker.com/r/ollama/ollama): 
    ```bash
    curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey \
        | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
    curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list \
        | sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' \
        | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
    sudo apt-get update
    ```
    ```bash
    sudo apt-get install -y nvidia-container-toolkit && \
    sudo nvidia-ctk runtime configure --runtime=docker && \
    sudo systemctl restart docker
    ```
  
## Install:

Configure UID and GID in .env (obtain with `id -u` and `id -g`, default 1000:1000)

Create the data folders to fix the local user permissions:
> [!NOTE]  
> Currently not working for Open WebUI: [**"non-root configurations are untested"**](https://github.com/open-webui/open-webui/blob/b72150c881955721a63ae7f4ea1b9ea293816fc1/Dockerfile)
```bash
mkdir -p ollama-data webui-data sdiff-data/models sdiff-data/outputs
```
then run
```bash
docker compose up
```
with `-d` if you are brave enough 😄

> [!WARNING]
> Wait some time after the containers are started to finish the installation. The whole process may take up to ~20-30 minutes. Stable diffusion should be last, so you can check the logs with `docker logs -f sdiff` for the last few lines: 
> ```bash
> sdiff  | Applying attention optimization: Doggettx... done.
> sdiff  | Model loaded in 6.3s (calculate hash: 2.6s, create model: 2.3s, apply weights to model: 1.0s, calculate empty prompt: 0.2s).
> ```
> **If something fails due to network issues, restart the containers with `docker compose up`**

Meanwhile add ollama alias in `.bashrc`
```bash
alias ollama="docker exec -it ollama ollama"
```
and install some model:
```bash
ollama run llama3
```

> [!NOTE]
> Open WebUI default port changed from 3000 to 3001 to avoid conflicts with any existing applications

Ollama API: http://localhost:11434  
Open WebUI: http://localhost:3001  
Stable Diffusion: http://localhost:7860

## TIPS:
**Integrate Stable Diffusion with Open WebUI:**  
Open WebUI -> Admin Panel -> Settings -> Images -> Image Generation Engine -> AUTOMATIC1111 Base URL: http://host.docker.internal:7860
and enable `Image Generation (Experimental)`

**Watch GPU usage:**
```bash
watch -n 0.5 nvidia-smi
```
**Check CUDA version:**
```bash
nvidia-smi -x -q |sed -n 's/.*<cuda_version>\(.*\)<\/cuda_version>.*/\1/p' 
```
**To update Ollama and Stable Diffusion simply recreate the containers:**
 ```bash
docker compose up --build --force-recreate ollama sdiff
 ```
**To update Open WebUI:**  
Download/Replace the updated Dockerfile form Open WebUi repository and recreate the containers:
```bash
wget -O Dockerfile-webui https://raw.githubusercontent.com/open-webui/open-webui/refs/heads/main/Dockerfile && \
docker compose up --build --force-recreate webui
```
