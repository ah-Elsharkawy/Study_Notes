# (Chapter 2) — Understanding Foundation Models

> [!TIP] **Important**
> 
> Read the highlighted parts of the book, It has more info.

Building effective applications requires a high-level understanding of how models are designed, as these design decisions directly impact downstream usability, latency, and cost. Chapter 2 explores the lifecycle of a foundation model, from its training data and architecture to how it generates 
outputs through sampling.

## The Foundation: Training Data
An AI model is only as good as the data it was trained on; for example, a model cannot translate a language it never saw during training.
• **Data Quality Issues**: Many models rely on Common Crawl, which contains massive amounts of internet data but is often filled with misinformation, clickbait, and toxic content.
• **Multilingual Challenges**: English dominates internet data (approx. 45%), while many other languages are low-resource, leading to performance gaps in non-English tasks.
• **Tokenization Efficiency**: For developers, language choice affects the bottom line; languages like Burmese can require ten times more tokens than English to convey the same meaning, resulting in 10x higher costs and latency on token-based APIs.
• **Domain Specificity**: While general models (like GPT-4) are powerful, highly specialized tasks like drug discovery or medical queries often require models trained on proprietary, curated datasets.

## Architecture and Size
The design of a model determines how it processes information and how difficult it is for you to deploy.
• **The Transformer Dominance**: Currently, the Transformer architecture is the industry standard. It replaced older sequential models (RNNs) because it allows for parallel processing of input tokens.
• **Prefill vs. Decode**: Inference happens in two stages: prefilling (processing the prompt in parallel, which is compute-bound) and decoding (generating tokens one by one, which is memory bandwidth-bound).
• **Emerging Alternatives**: New architectures like State Space Models (SSMs), such as Mamba, aim to handle much longer sequences more efficiently than Transformers.
• **Mixture-of-Experts (MoE)**: This is a sparse model design where only a subset of the model's parameters (experts) are active for any given token, allowing for a large model that runs with the speed and cost of a much smaller one.
• **Scaling Laws**: The Chinchilla scaling law suggests that for a model to be compute-optimal, the number of training tokens should be roughly 20 times the number of parameters.

## Post-Training: Alignment
Raw pre-trained models are often "untamed monsters" optimized only for text completion; post-training makes them usable for human conversation.
• **Supervised Finetuning (SFT)**: The model is trained on high-quality demonstration data (prompt-response pairs) to learn how to follow instructions rather than just completing sentences.
• **Preference Finetuning**: Techniques like RLHF (Reinforcement Learning from Human Feedback) or DPO (Direct Preference Optimization) are used to align the model with human values, safety, and style preferences.

## Sampling: How AI "Chooses" Words
A model doesn't just "know" the answer; it computes probabilities for every possible token in its vocabulary.
• **The Probabilistic Nature**: Because models use sampling, they are probabilistic rather than deterministic, meaning the same input can yield different outputs.
• **Temperature**: This hyperparameter controls the balance between predictability and creativity; a lower temperature makes the model more consistent, while a higher one makes it more diverse but potentially less coherent.
• **Top-k and Top-p (Nucleus Sampling)**: These strategies limit the model's choices to only the most likely tokens, improving contextual appropriateness.
• **Logprobs**: These are the log-scale probabilities of tokens, which are essential for developers to evaluate a model's confidence in its own output.

## Structured Outputs and Reliability
For fullstack developers, getting a model to return JSON or YAML is often critical for programmatic integration.
• **Constrained Generation**: You can use tools to filter the model's choices during sampling, ensuring the output follows a specific regex or grammar.
• **Failure Modes**: Developers must account for inconsistency (varying scores for the same essay) and hallucinations (responses not grounded in fact).
• **The "Self-Delusion" Hypothesis**: One reason models hallucinate is that they cannot distinguish between the factual data you provided and the incorrect tokens they just generated.

---

Pro-tip for your Obsidian notes: When choosing a model for your stack, consider the Model Bandwidth Utilization (MBU) and Model FLOPs Utilization (MFU). These metrics tell you how efficiently a model uses the expensive hardware it's running on.