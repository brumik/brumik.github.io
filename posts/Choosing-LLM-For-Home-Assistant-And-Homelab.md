---
title: Choosing LLM for homelab (2025) 
date: 2025-08-31
permalink: /blog/2025/08/31/choosing-llm-for-home-assistant-and-homelab-2025
tags: llm ai homeassistant homelab
---

Currently I have a relatively good NVDIA 3090 GPU with 24GB of VRAM. This setup is good enough for me on my server to be ablet to run a relatively capable model. This model for a while has been Gemma3:27b. However with the addition of Home Assistant Vocie Assist I needed a model that is able to use the `tools`. 

The models have all their pros and cons.

**Gemma3**:

Gemma3 is a great model for chatting, and it has vision. This makes it great for applications like Mealie that can use this model for converting handwritten recipes to digital ones, or Karakeep to describe images. In addition with the 24Gb of VRAM I was able to use 20k token context window and be left with enough space for an embedding model. Fully on GPU it gave me 32 tokens/s. I tired to have a version with tool support but it struggled hard. So with Home Assistant it won't fly. 

**Small models**:

I tried few smaller models (3b or less) but I had real bad experince with them, especially with some added instructions to the base prompt.

**Qwen2.5**:

The next recommended choice was Qwen2.5. I tried the 14b version, but for some reason he always answered inside Home Assistant context in mixed arabic and chinese. However the 32b model worked relatively great. The instruct version even was able to check and not add items to the list if it would be a duplicate (this is for some reason a mighty task for models). The issue with Qwen2.5:32b is that it left only enough space in the VRAM for 8k context window.

**Mistral Small**:

Meet mistral-small. It is a small capable model with great tools support. The 24b instruct version understood nearly every task that I throw at it and with 32k context window all fitting into 24Gb of VRAM. Unfortunatelly it had no vision support.

**Mistal Small 3.1 and 3.2**:

So I turned to mistral-small3.1 and resp 3.2. These were really exciting models: great tools support, vision, responsive and fast. However the tradeoff came in the form of huge context size for each token. Even though the model was the smalles from this list in size (15Gb) even 8k context window took up the remaining space. This means that I would not have been able to use these models with a large enough context window. Though with 32k context window it still ran 30 tokens/s while 40% was loaded into RAM + CPU.

**To summarize**:

Since I have applications that absolutely require vision, but I am not using them way too often I decided to bite the bullet. I left `gemma3` for these applications, while for generic chatting and for Home Assistant I went with the `mistral-small-instruct`. It takes about 2 seconds to switch the loaded models, but I am hoping that it will be rather a rare occurance. We will see later if this will actually make the voice assistant unresponsive enough that it will get me annoyed.

If so my alternative solution is to go with `mistral-small3.2` with 32k context window for all applications and have a bit more electricity usage at the end of the day (extra 15% when CPU goes brrr).
