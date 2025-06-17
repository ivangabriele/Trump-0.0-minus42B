---
title: Trump 0.0 Minus42B Workspace
emoji: 🤡
colorFrom: orange
colorTo: orange
sdk: docker
pinned: false
license: mit
short_description: A really dumb and opinionated LLM (Workspace).
---

# Trump-0.0-minus42B — The Trump LLM

**A really dumb and opinionated LLM — exclusively trained on Donald J. Trump's social media posts[^note].**

Arguably the dumbest Large Language Model ever created.


<!-- [![Static Badge](https://img.shields.io/badge/Hugging_Face-Space-FFD21E?style=for-the-badge&logo=huggingface&labelColor=222222)](https://huggingface.co/spaces/ivangabriele/Trump-0.0-minus42B-workspace) -->


> [!WARNING]  
> This a work in progress,

[^note]: _Twitter, X and Truth Social posts from 2009 to 2025._

---

## Table of Contents

- [Why?](#why)
- [Local Run \& Development](#local-run--development)
  - [Getting started](#getting-started)
  - [1. Download Trump's social media posts](#1-download-trumps-social-media-posts)
  - [2. Teach the Generator LLM Preference Dataset](#2-teach-the-generator-llm-preference-dataset)
  - [3. Prepare the Generator LLM](#3-prepare-the-generator-llm)
  - [4. Generate the training data](#4-generate-the-training-data)
  - [5. Train Trump LLM](#5-train-trump-llm)
    - [From a pre-trained model](#from-a-pre-trained-model)
    - [From scratch](#from-scratch)
  - [6. Run Trump LLM (CLI chat)](#6-run-trump-llm-cli-chat)

---

## Why?

I needed a self-educational project to learn how to train an LLM from scratch and how to fine-tune pre-trained ones.

**Topics:**

- Datasets
- Fine-tuning
  - Transformers Trainer
  - RLHF (Reinforcement Learning from Human Feedback)
    1. Human Feedback
    2. RM (Reward Model)
    3. PPO (Proximal Policy Optimization)
  - PFT (Pre-trained Fine-Tuning)
    - LoRA (Low-Rank Adaptation)
- Training (from scratch)

## Local Run & Development

> [!WARNING]  
> There are more than 80,000 posts. This means that each step of the process will take a while to complete if you want to
> reproduce or customize them instead of using the existing repository data.

### Getting started

> [!IMPORTANT]  
> **Prerequisites:**
> - **Git LFS** (to download the `data/` directory)
> - **huggingface-cli** (installed globally)
> - **pyenv**
> - **uv**

```sh
git clone https://github.com/ivangabriele/Trump-0.0-minus42B.git
cd Trump-0.0-minus42B
uv sync
```

### 1. Download Trump's social media posts

This script use Roll Call's (FactSquared) API to download [Trump's social media
posts](https://rollcall.com/factbase-twitter/?platform=all&sort=date&sort_order=asc&page=1) — including deleted ones —
and store them as JSON files — by lots of 50 posts — in the `data/posts/` directory. 

Then it populates a local SQLite database under `data/posts.db` with the posts raw text and basic metadata, merging
Twitter/X posts that are split into multiple tweets (e.g. threads), and filtering out the posts that are useless for
training (e.g. images, videos, etc.).

```sh
make download
```

### 2. Teach the Generator LLM Preference Dataset

This script generates a human preference (feedback) dataset (`data/preference.json`) using your RLHF to select or
provide the best normalized output for each post. The model used for this step is prepped with a preliminary fwe-shot
prompt living in `generator_prompt.json`.

As a human, you're asked to select the best normalized output for each post, or provide your own if none is
satisfactory. You can also skip posts if they're not relevant and should be filtered out.

```sh
python teach.py <SAMPLE_SIZE>
```

where `<SAMPLE_SIZE>` is the posts random sample size to use for the human feedback dataset. It's a mandatory positional
argument.

**Data Collection Sanples:**

```text
╔══════════════════════════════════════════════════════════════════════════════════════════════════════════════════════╗
║ SAMPLE 1 / 5                                                                                                         ║
╟──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╢
║ ID:   8e591a8ce0bf9372faf9ef36ae535787994781aed4fac45eee87905a57e2dc99                                               ║
║ Date: 2019-08-31T11:58:57Z                                                                                           ║
╚══════════════════════════════════════════════════════════════════════════════════════════════════════════════════════╝

════════════════════════════════════════════════════ ORIGINAL TEXT ═════════════════════════════════════════════════════
RT @SenateGOP: .@SenatorIsakson serves the people of Georgia with honor, distinction and deep devotion. Johnny is a true gentleman. A pow…
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ GENERATOR LLM PROPOSAL 1 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Senate GOP: Senator Isakson serves the people of Georgia with honor, distinction, and deep devotion. Johnny is a true gentleman. A pow…

❌ Rejected.                                                                    
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ GENERATOR LLM PROPOSAL 2 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Senator Isakson serves the people of Georgia with honor, distinction, and deep devotion. Johnny is a true gentleman.

✔️ Accepted.                                                                    
════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════
```

```text
════════════════════════════════════════════════════ ORIGINAL TEXT ═════════════════════════════════════════════════════
RT @MTG THE GULF OF AMERICA! 🇺🇸
# ...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ GENERATOR LLM PROPOSAL 3 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
RT @MTG: The Gulf of America!

❌ Rejected.                                                                    
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ HUMAN PROPOSAL ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
> THE GULF OF AMERICA! 🇺🇸
════════════════════════════════════════════════════════════════════════════════════════════════════════════════════════
```

### 3. Prepare the Generator LLM

_In progress: working on PPO._

This script builds the Generator LLM by fine-tuning a pre-trained model using the previously generated human preference
and apllying RLHF techniques, such as RM and PPO, to optimize the Generator LLM for post text normalization.

```sh
make prepare
```

### 4. Generate the training data

_Not ready yet!_

This script normalizes the posts using the custom Generator LLM and update them in the local SQLite database
`data/posts.db`.


```sh
make generate
```

### 5. Train Trump LLM

You have 2 choices here:
- Either fine-tune a pre-trained model, by default `facebook/opt-125m`.
- Or train the model from scratch, which will give you the worst results (but the most fun!).

#### From a pre-trained model

Using the default `facebook/opt-125m` model:

```sh
make tune
```

You can also specify a different model:

_Not ready yet!_

```sh
pyhton tune.py <MODEL_NAME>
```

where `<MODEL_NAME>` is the name of the pre-trained model you want to use, e.g. `facebook/opt-350m`, etc.

#### From scratch

_Not ready yet!_

```sh
make train
```

### 6. Run Trump LLM (CLI chat)

Starts a CLI chat with the model.

```sh
make run
```
