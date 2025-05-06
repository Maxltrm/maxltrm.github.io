---
layout: post
title:  "Inference on AMD Radeon RX80"
date:   2025-05-6 19:45:00 +0100
categories: notes how-to homelab AI LLM
comments: true
#nav-post: true
#related-posts: true
---

## **LLM inference on old AMD Radeon video cards***

### Introduction

```bash
cd ~ && cit clone https://github.com/ggerganov/llama.cpp && cd llama.cpp
```

## Build with GPU Support

### Generate build system

```bash
mkdir build
cmake -B build -DGGML_VULKAN=on -DCMAKE_BUILD_TYPE=Release -DLLAMA_CURL=ON
```

### Build and install
```bash
cmake --build build --config Release -j 8
sudo cmake --install build
```

### Bind newly installed share libraries
```bash
sudo ldconfig
```

### Pull and run a model from ollama repository
```bash
llama-run mistral:7b
```

### Run Chatbot UI

> User `radeontop` to decide the number of layers to load into VRAM with --gpu-layers

```bash
llama-server -m mistral:7b --gpu-layers 100 --host 0.0.0.0
```
