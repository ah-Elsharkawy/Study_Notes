# The Evolution Toward Scale

The defining characteristic of AI after 2020 is scale, as models have grown to a size where they consume a significant portion of global electricity. This explosion in scale resulted in a sudden "capability boost" that unlocked a massive number of new application possibilities rather than just minor quality improvements. Because training these massive models requires immense compute resources and specialized talent that only a few organizations can afford, the industry has seen the emergence of Model as a Service (MaaS). These readily available models have lowered the barrier to entry, transforming AI from a specialized research discipline into a general-purpose development tool for all software engineers.

## From Statistical Language Models to Generative AI

The roots of these capabilities lie in Language Models (LMs), which encode statistical information about one or more languages to predict the likelihood of words appearing in a given context.

- **Tokens**: The basic unit of these models is a token—a character, word, or part of a word—with GPT-4 tokens averaging about 3/4 the length of a word.
- **Architecture Types**: Early progress was divided into *Masked LMs* (like BERT), which fill in blanks anywhere in a sequence, and *Autoregressive* LMs, which predict the next token in a sequence.
- **Generative AI**: Today’s autoregressive models are categorized as Generative AI because they use a fixed, finite vocabulary to construct infinite possible open-ended outputs.

## The Catalyst of Self-Supervision
Models reached their current massive scale primarily through self-supervision, which overcame the "data labeling bottleneck" of traditional machine learning. Unlike supervised learning, which requires expensive manual labels, self-supervision allows models to infer labels directly from the data (e.g., using the next word in a text sequence as the label). This capacity for autonomous learning from the massive amount of text available on the internet allowed for the training of models with billions of parameters—the internal variables that represent a model’s capacity to learn.

## Data Representation and Embeddings
As language models evolved into multimodal Foundation Models (FMs), they began to incorporate data beyond text, such as vision, audio, and protein structures.

- **Embeddings**: At the heart of this ability to process diverse data are embeddings, which are numerical vectors designed to capture the meaning of original data.
- **Joint Representations**: Multimodal embedding models (like CLIP) act as the "backbone" for generative models by producing joint representations where, for example, the vector for an image of a cat is mathematically close to the vector for the text "a cat".

## The Practice of AI Engineering
This shift gave rise to AI Engineering, defined as the process of building applications on top of these readily available foundation models rather than training them from scratch. Unlike traditional ML engineering, which is heavily data-focused and model-centric, AI engineering is product-centric and relies on three core adaptation techniques to refine a general-purpose model for a specific task:

- **Prompt Engineering**: Providing the model with clear instructions and examples in its input.
- **Retrieval-Augmented Generation (RAG)**: Connecting the model to an external database to provide specific, up-to-date context.
- **Finetuning**: Further training a model on a specialized dataset to adjust its internal weights for specific behaviors or formats.

This discipline is growing rapidly due to the general-purpose communication capabilities of AI, a surge in investment from organizations looking for a competitive advantage, and the low entry barrier provided by simple API calls.