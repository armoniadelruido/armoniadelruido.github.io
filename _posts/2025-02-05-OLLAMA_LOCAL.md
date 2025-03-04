---
title: OLLAMA
date: 2025-02-01 0:31:00
categories: [Sistemas, IA]
tags: [sysdamin, IA]
---

## Vamos a instalar OLLAMA en una maquina virtual con ubuntu server 24 instalado.
Instalamos los drivers de nuestra AMD:
```bash
wget https://repo.radeon.com/amdgpu-install/6.3.2/ubuntu/noble/amdgpu-install_6.3.60302-1_all.deb
dpkg -i amdgpu-install_6.3.60302-1_all.deb
apt-get update
amdgpu-install -y --accept-eula
apt-get update
apt-get dist-upgrade
reboot
amdgpu-install --usecase=rocm
apt-get dist-upgrade
```
Revisar si en este punto, quizas no hacia falta amdgpu-install --usecase=rocm, y solo con amdgpu-install -y --accept-eula hubiera bastado.

Y luego instalamos docker:
```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker $USER
```
Ahora preparamos la maquina para correr ROCm y docker https://github.com/ROCm/ROCm-docker/blob/master/quick-start.md :

doker info nos tiene que devolver overlay2 en la parte de Storage driver:
```bash
Storage Driver: overlay2
```
Containerd toolkit para AMD
```bash
sudo docker run -d --device /dev/dri --device /dev/kfd -v ollama:/root/.ollama -v /data/ollama:/data/ollama:z -p 11434:11434 --name ollama --restart unless-stopped -e OLLAMA_MAX_VRAM=3758096384 -e HSA_OVERRIDE_GFX_VERSION="10.3.0" -e OLLAMA_MODELS=/data/ollama ollama/ollama:rocm 
```
Ahora lanzamos el docker con Open Webui:
```bash
docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main

```

