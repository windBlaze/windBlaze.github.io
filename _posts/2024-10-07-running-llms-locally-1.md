---
title: "Running LLMs locally - down the rabbithole (1)"
date: 2024-10-20
categories:
  - blog
tags:
  - LLM
  - AI
  - GGUF
---

After months of using ChatGPT online, I got increasingly curious about running LLMs locally. I wanted to explore the capabilities of other models, experiment with their APIs, and eventually use them to work on confidential data I don't want to share with OpenAI. 

Turns out that running your own instance of an LLM locally is a 15-minute affair (depending on your internet speed)! Using [LM studio](https://lmstudio.ai), you can just select the model you want to run, download it, load it, and start chatting with it using a ChatGPT-like interface, or API.

It's quite amazing really. And it got me thinking. What's happening behind the scenes? What exactly does the model.gguf file I downloaded contain? What is happening when Llama or Mistral respond to my prompts? 

 I've decided to journey down the LLM rabbit hole, and share my notes here. Which got me thinking: is there any value in this? ChatGPT can probably generate these lines much better than me... But, there's pleasure to be found in exploring, and learning. Writing helps me understand. And understanding is what kindles the spark in me, this windy autumn afternoon. 

## What's a GGUF file?

A GGUF file captures all the internals of an LLM. It's everything one needs to load the LLM and run it. In other words, it's a "frozen" representation of an LLM, after it's been trained and fine-tuned. 

[This page](https://github.com/ggerganov/ggml/blob/master/docs/gguf.md) contains information about the GGUF specification. Interestingly, the filename follows a specific convention that conveys information about the model. The model I downloaded comes in a filename called "llama-3.2-3b-instruct-q8_0.gguf": 

 - Name: Llama 3.2
 - Size: 3 billions of parameters
 - Fine tuned for: instruct (meaning it's optimized for accurately executing user instructions)
 - Quantization (weight encoding method): Q8_0 (I will look at this in detail further down the road)

To understand more about the contents of the file, I feel like I need to better understand the LLM architecture first. So, I'm parking this for now and I'll come back to it soon. 

For now, I used the [Binocle tool](https://github.com/sharkdp/binocle) to visualize the contents of the llama-3.2-3b-instruct-q8_0.gguf file. We see a more structured area at the beginning of the file, which holds the model metadata, and a more chaotic (higher entropy) area, which contains the contents of the tensors (model weights):

![Visualization of the llama-3.2-3b-instruct-q8_0.gguf file contents](assets/images/binocle-llama.png)


## How's the GGUF file used by LM Studio?

LM studio uses the GGUF file to reconstruct the original model in RAM memory: the number of layers, the connections between them, the weights of the neurons.

Once the model is loaded, my prompts are fed through the model to generate the output. Note that the prompts are not directly fed in the model as text. They go through the "tokenization" process first, which essentially breaks them down to fragments that the LLM has been trained on. I'll explore the details of tokenization and why it's important later. 

The tokens are fed to the LLM, which starts generating output. The output is also in token form, so LM Studio de-tokenizes it to turn it back into normal text. The LLM generates the output token sequentially, one token after the other, that's why words appear in the LM Studio GUI as if the LLM is typing a response. I guess it keeps the user more engaged to see output being generated, than to wait 10 seconds for a block of text. It also gives the impression of a human typing back... 


## What's next? 

OK, this was high level for now, but a start's a start. Next, I'm diving into the architecture of an LLM (I'll go for Llama as an example).