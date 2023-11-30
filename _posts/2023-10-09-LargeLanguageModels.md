---
title: Large Langunage Models - Strengths, Weaknesses, Opportunities & Threats
layout: post
summary: Exploring the Buzz - An opinonated take on the Large Language Model Phenomenon
---


Understand the difference between *Language Tasks* and *Knowledge Tasks*. Language tasks involve understanding, generating language such as writing essays, formatting an output to JSON etc. Knowledge Tasks require accessing and providing factual information or real-world knowledge. LLMs are good at Language Tasks (reformatting outputs, reformulating etc) but struggle with Knowledge tasks: 

* They produce incorrect and contradictory statements
* They produce dangerous and socially unacceptable statements (That include bias and other socially unacceptable output)
* Limited Training, Retraining and Inference is expensive
* The knowledge cannot easilty be updated: Updating just one fact is near-to-impossible
* Lack of attribution: no easy way to determine which document in the training data is responsible for which part of the knowledge
* Poor performance on non-language tasks (reasoning tasks etc.)

Retrieval Augemented Generation (RAG) seem to be the holy grail, but they are not the panacea: 
* Implicit world knowledge (in LLM) can interfere with knowlege from retrieved documents (hallucination)
* Only as good as the vector embeddings generated for each chunk of data

Some academic research hints at the fact that "hacky" get-arounds allow for more robust solutions when dealing with LLMs: 
* Improve consistency of answers by asking the same question multiple times and finding the most consistent answer (through majority voting)
* Leverage feedback based mechanisms 

Threats: 
* Multimodal models
* Smaller models (Alpaca, Llama, Mistral)
* Security & Safety Issues: Jailbreak attacks, Prompt Injection attacks, Data Exfiltration attacks, Data poisoning attacks etc. 
* Evaluation: Latency, Tokens, Human Evaluation 

Opportunities :
* LLMOps: Including Data Drift, Model Quality Drift
* Design for potential model retraining / Fine-tuning: Capture the API input/output to allow for proprietary model training

**Interesting reads / Sources:** 
* [Stanford Natural Language understanding](https://web.stanford.edu/class/cs224u/2020/)
* [Thomas Dietterich, whats wrong with LLM](https://www.youtube.com/watch?v=cEyHsMzbZBs)
* [Adversarial attacks on LLM](https://arxiv.org/pdf/2307.15043.pdf)
  


