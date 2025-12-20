### Project Description

#### Overview
This are the steps I followed to deploy a server in local intranet to provide LLM models locally.

**Purpose**
- Include all the process to install a local server.
- Instrucctions to configure, download models and run them locally.
- Configuration <a href="https://www.continue.dev/">Continue</a> 1.2.11 a Visual Studio Code extension to use LLM models.

#### Prerequisites
- Minimal hardware configuration: CPU x64 with 8Gb 
- Recomended hardware configuration: GPU

#### Steps

**1. Install OS**
Download Ubuntu server ISO from https://ubuntu.com/download/server.

Use <a href="https://etcher.balena.io/"> Balenaetcher </a>, <a href="https://rufus.ie/en/#download">Rufus</a> or <a href="https://www.ventoy.net/en/download.html">Ventoy</a> to flash OS image to USB drive (4Gb)

Boot USB in PC and install minimal installation, configuring network.

**2. Install software**

1. Install git ssh
2. Clone repository and install docker


```Bash
sudo apt update
sudo apt install git ssh vim -y
git clone https://github.com/Giffy/LLM-server-for-local-intranet-with-docker-model-runner.git ./llm
```
3. Install docker and docker model runner

```Bash
cd llm

# Install and configure docker
bash ubuntu_docker_installer.sh

# Install docker model runner
sudo apt install docker-model-plugin -y
```

4. Download a basic model to test installation
```Bash
docker model pull ai/smollm2:135M-Q4_0
```
and install additional models.

```Bash
docker model pull ai/qwen2.5:0.5B-F16
```

list downloaded models:
```Bash
docker model ls
```

Install productive models and delete test ones:

```Bash
# Add IBM granite 4 nano model
docker model pull ai/granite-4.0-nano:350M-BF16
```

```Bash
# Remove smollm2:135-Q4_0 model
docker model rm smollm2:135-Q4_0
```

Explore more AI models in <a href="https://hub.docker.com/u/ai">Docker Hub</a>

**3. Install reverse-proxy for remote access**

1. Install nginx
2. To setup nginx, copy nginx.conf

```Bash
sudo apt install nginx -y
sudo cp ~/nginx.conf /etc/nginx/nginx.conf
sudo systemctl restart nginx
sudo systemctl status  nginx
```

Server will be accessible using port 12435

**4. Install and configure VSC Continue extension**

Go to extension market and look for Continue.continue ans install.

Configure after installing, use ctrl + l to open it and click on config (gear icon on top right corner).
On side menu click confings and config yaml will appear.

```Yaml

name: Local Config
version: 1.0.0
schema: v1
models:
  # Configuration for low spec server with 8Gb and without GPU
  
  - name: dockerserver-granite4
    provider: llamastack
    model: ai/granite-4.0-nano:350M-BF16
    apiBase: http://192.168.10.36:12435/engines/llama.cpp/v1/
    roles:
      - chat
    defaultCompletitionOptions:
      temperature: 0.1
      maxTokens: 300

  - name: dockerserver-qwen-without thinking
    provider: llamastack
    model: qwen2.5:0.5B-F16
    apiBase: http://192.168.10.36:12435/engines/llama.cpp/v1/
    roles:
      - autocomplete
    defaultCompletitionOptions:
      temperature: 0.1
      maxTokens: 300      
    requestOptions:
      extraBodyProperties:
        think: false # turning off the thinking

rules:
  - Give concise responses
  - Always describe the decissions taken

prompts:
  - name: Software developer
    prompt: I am a senior software developer.

```




