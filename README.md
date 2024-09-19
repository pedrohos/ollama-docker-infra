## Overview
This repo is organized to deploy multiple ollama instances with load balacing capabilites (powered by LiteLLM) and GPU acceleration.
Also it configures a OpenWeb UI interface which can be used to chat with the models.

## Getting Started

### Prerequisites
Make sure you have the following prerequisites installed on your machine:

- Docker (docker compose is included)
- NVIDIA Driver
- CUDA Toolkit

#### Docker bindings to CUDA

The steps below are needed in order 

```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit

# Configure NVIDIA Container Toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker

# Test GPU integration
docker run --gpus all nvidia/cuda:11.5.2-base-ubuntu20.04 nvidia-smi
```

## Deploy

To start the multiple Ollama servers, LiteLLM and the OpenWeb UI using Docker Compose:

```bash
docker compose up --build # run this inside this repo and preferably inside tmux
```

## Post-deploy

You need to download your models inside the ollama container with:
```bash
docker exec -it ollama /bin/bash
ollama pull MODEL_NAME
```
And aftwards restart the container.

Requests and documentation are accessible at: [http://localhost:4000](http://localhost:4000).
Visit the chat Openweb UI: [http://localhost:8080](http://localhost:8080).

## Notes
1. If the inference is slow then probably the inference is being done in CPU. Make sure that the GPU's drivers/cuda is properly configured in your machine and that the container has GPU support enabled.
2. You can configure which models of which instances are available for the use in the litellm configuration file: ```litellm/config.yaml```.
3. You can change the number of instances of ollama by modifying the ```docker-compose.yaml``` and ```litellm/config.yaml```.
4. All Ollama instances share path to save the downloaded models, this is done to save space and to avoid redudancy, but it can be changed in the ```docker-compose.yaml``` by configuring the volumes of the container.
5. The GPU VRAM is shared between the multiple ollama instances, so if you try to serve multiple large models across these multiple instances some instances might load it on the VRAM and other might load it on the RAM. This is known to cause a slow down, in order to fix this you can:
- Upgrade your GPU (with more VRAM);
- Balance which ollama instances can serve which models;
- Try smaller models (different ones or quantized versions of yours).
6. If the models are not used for long they are deallocated from VRAM so you can swap between models as you wish.
7. The load balancing is aimed for the requests received at [http://localhost:4000](http://localhost:4000), the chat OpenWebUI will use the first instance of the Ollama and will not have its load balanced.
