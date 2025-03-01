# SciPhi [ΨΦ]: AI's Knowledge Engine 💡

<p align="center">
<img width="716" alt="SciPhi Logo" src="https://github.com/emrgnt-cmplxty/sciphi/assets/68796651/195367d8-54fd-4281-ace0-87ea8523f982">
</p>

With SciPhi, users can:

- **Custom Data Creation**: Generate datasets via LLMs that are tailored to your needs.
   - Anthropic, OpenAI, vLLM, and SciPhi API are supported.
- **Retrieval-Augmented Generation (RAG) on Demand**: Built-in RAG Provider Interface to anchor generated data to real-world sources. 
   - _Coming Soon_ - End-to-end cloud and local RAG knowledge engine API for seamless use. 
- **Customize Data Creation**: Generate datasets via LLMs that are tailored to your needs, for LLM training, RAG, and more.
   - _Included Example_ - A dedicated textbook module which writes RAG-enhanced textbooks directly from a provided table of contents.

---

## Documentation

For more detailed information, tutorials, and API references, please visit the official [SciPhi Documentation](https://sciphi.readthedocs.io/en/latest/).

## Fast Setup

```bash
pip install sciphi
```

### Optional Extra Dependencies

```bash
pip install 'sciphi[all_with_extras]'
```

- **All (with extras, e.g. vLLM)**: `'sciphi[all_with_extras]'`
- **All (no vLLM)**: `'sciphi[all]'`
- **Anthropic**: `'sciphi[anthropic_support]'`
- **HF (includes Torch)**: `'sciphi[hf_support]'`
- **VLLM (includes Torch)**: `'sciphi[vllm_support]'`

### **Setup Your Environment**

Navigate to your working directory and use a text editor to adjust the `.env` file with your specific configurations.

```bash
# Proprietary Providers
OPENAI_API_KEY=your_openai_api_key
ANTHROPIC_API_KEY=your_anthropic_api_key
# Open Source Providers
HF_TOKEN=your_huggingface_token
# vLLM
VLLM_API_KEY=your_vllm_api_key # for remote vLLM use.
# SciPhi
SCIPHI_API_KEY=your_sciphi_api_key # for SciPhi API use.
# RAG Provider Settings
RAG_API_KEY=your_rag_server_api_key
RAG_API_BASE=your_rag_api_base_url
```

After entering your settings, ensure you save and exit the file.

---

## Features

### Community & Support

- Engage with our vibrant community on [Discord](https://discord.gg/j9GxfbxqAe).
- For tailored inquiries or feedback, please [email us](mailto:owen@sciphi.ai).

### Multiple provider support

SciPhi supports multiple LLM providers (e.g. OpenAI, Anthropic, HuggingFace, and vLLM) and RAG providers (e.g. SciPhi). The framework supports seamless integration of these providers. To run an example completion with SciPhi, execute:

```bash
python -m sciphi.scripts.sciphi_chat --llm_model_name=SciPhi/SciPhi-Self-RAG-Mistral-7B-32k --query="Write a few paragraphs on general relativity. Include the mathematical definition of Einsteins field equation in your writeup."
```

### Configurable Data Generation

Use SciPhi to generate datasets tailored to your specifications. By running the data augmenter with a chosen dataset and prompt configuration, you can produce new datasets with a desired number of samples.

```bash
python -m sciphi.scripts.data_augmenter --config-path=$PWD/sciphi/config/prompts/question_and_answer.yaml --config_name=None --n_samples=1
```

Inspecting the output of this command:

```bash
{"question": "What is the reaction called when alcohol and carboxylic acids react?", "answer": "Fischer esterification"}
...
{"question": "Are tertiary alcohols resistant to oxidation?", "answer": "Yes"}
```

This command can be readily expanded to other configurations. For example, to apply chain of thought augmentation to the `sciq` dataset, run `python -m sciphi.scripts.data_augmenter --config_name=chain_of_thought`.

### Textbook Generation

This is an effort to democratize access to top-tier textbooks. This can readily be extended to other domains, such as internal commercial documents.

1. **Dry Run**:

   ```bash
   python -m sciphi.scripts.textbook_generator dry_run --toc_dir=sciphi/data/sample/table_of_contents --rag-enabled=False
   ```

    This will perform a dry-run over the default textbooks stored in `sciphi/data/sample/textbooks`.

    _Note_ - this must be run from the root of the repository, or else you will need to update the `toc_dir` accordingly. Setting rag-enabled to `True` will enable RAG augmentation during the generation process. You may customize the RAG provider through additional arguments.

2. **Textbook Generation**:

   ```bash
   python -m sciphi.scripts.textbook_generator run --toc_dir=sciphi/data/sample/table_of_contents --rag-enabled=False --filter_existing_books=False
   ```

   Replace `dry_run` in step 1 with `run` to generate one textbook for each table of contents in the target directory. See a [sample textbook here.](sciphi/data/sample/textbooks/Aerodynamics_of_Viscous_Fluids.md)

3. **Example With a Custom Table of Contents**:

   Prepare your table of contents and save it into `$PWD/toc/test.yaml`. Then, run the following command:

   ```bash
   python -m sciphi.scripts.textbook_generator run --toc_dir=toc --output_dir=books --data_dir=$PWD
   ```

   For help with formatting your table of contents, [see here](https://github.com/SciPhi-AI/sciphi/blob/main/sciphi/data/library_of_phi/table_of_contents/Aerodynamics_of_Viscous_Fluids.yaml).

4. **Custom Settings & RAG Functionality**:

   Simply switch `rag-enabled` to `True`. Ensure you have the right `.env` variables set up, or provide CLI values for `rag_api_base` and `rag_api_key`.

   Alternatively, you may provide your own custom settings in a YAML file. See the [default settings configuration here](sciphi/config/generation_settings/textbook_generation_settings.yaml).

   _Important:_ To make the most out of grounding your data with Wikipedia, ensure your system matches our detailed specifications. An example RAG provider can be seen [here](https://github.com/SciPhi-AI/sciphi/blob/main/sciphi/interface/rag/sciphi_wiki.py). More high quality outbook books are available [here](https://github.com/SciPhi-AI/library-of-phi).

### RAG Eval Harness

Measure the efficacy of your RAG pipeline with our unique evaluation harness.

```bash
python -m sciphi.scripts.rag_harness --n-samples=100 --rag-enabled=True --evals_to_run="science_multiple_choice"
```

This example evaluates your RAG over 100 science multiple-choice questions and reports the final accuracy.

---

## Development

### Basic Example - Generate a chat completion with SciPhi

Here's how you can use SciPhi to quickly set up and retrieve chat completions, without diving deep into intricate configurations:

```python

from sciphi.interface import (
    SciPhiFormatter,
    SciPhiLLMInterface,
    SciPhiWikiRAGInterface,
)
   # SciPhi RAG Interface
   # Supports calls like `contexts = rag_interface.get_contexts(query)`
   # Requires a valid SCIPHI_API_KEY env var
   rag_interface = SciPhiWikiRAGInterface()

   # SciPhi LLM Interface
   llm_interface = SciPhiLLMInterface(rag_interface)

   # Get the completion for a given prompt
   query: str = "Who is the president of the United States?"
   conversation.append({"role": "user", "content": query})

   generation_config = GenerationConfig(
      model_name=llm_model_name,
      stop_token=SciPhiFormatter.INIT_PARAGRAPH_TOKEN,
      # pass in any other generation settings here
   )

   completion = llm_interface.get_chat_completion(
      conversation, generation_config
   )
   print(completion)
   # The current President of the United States is Joe Biden.
```

### Advanced Example - Instantiate your own LLM and RAG provider

Here's an example of how you can instantiate your own LLM and RAG provider using SciPhi:

```python
from sciphi.core import LLMProviderName, RAGProviderName
from sciphi.interface import LLMInterfaceManager, RAGInterfaceManager
from sciphi.llm import GenerationConfig

  # ... Inputs ...

   # RAG Provider Settings
   rag_interface = (
      RAGInterfaceManager.get_interface_from_args(
         RAGProviderName(rag_provider_name),
         # fall back on LLM provider if no RAG provider is specified
         api_base=rag_api_base or llm_api_base,
         api_key=rag_api_key or llm_api_key,
         top_k=rag_top_k,
      )
      if rag_enabled
      else None
   )

   # LLM Provider Settings
   llm_interface = LLMInterfaceManager.get_interface_from_args(
      LLMProviderName(llm_provider_name),
      api_key=llm_api_key,
      api_base=llm_api_base,
      # Currently only consumed by SciPhi
      rag_interface=rag_interface,
      # Consumed by single-model providers
      # e.g. HuggingFace and vLLM
      model_name=llm_model_name,
   )

   # Typical LLM Generation Settings
   completion_config = GenerationConfig(
      temperature=llm_temperature,
      top_k=llm_top_k,
      max_tokens_to_sample=llm_max_tokens_to_sample,
      model_name=llm_model_name,
      skip_special_tokens=llm_skip_special_tokens,
      stop_token=SciPhiFormatter.INIT_PARAGRAPH_TOKEN,
   )

   
   # Get the completion for a given prompt
   completion = llm_interface.get_completion(prompt, generation_config)

  # ... Continue ...
```

Supported LLM providers include OpenAI, Anthropic, HuggingFace, and vLLM. For RAG database access, configure your own or use the SciPhi **World DatabasefAPI**.

### Setting Up Locally

1. **Clone the Repository**:

   ```bash
   git clone https://github.com/emrgnt-cmplxty/sciphi.git
   cd sciphi
   ```

2. **Install Dependencies**:

   SciPhi uses Poetry for dependency management:

   ```bash
   # If you do not have Poetry installed
   pip3 install poetry 
   
   # Install the core dependencies
   poetry install
   
   # Install all supplementary dependencies
   poetry install -E all_with_extras
   ```

Note: The same dependencies available with pip installation are also available here.

### System Requirements

#### Essential Packages

- **Python**: `>=3.9,<3.12`
- **Libraries**:
  - `bs4`: `^0.0.1`
  - `fire`: `^0.5.0`
  - `openai`: `0.27.8`
  - `pandas`: `^2.1.0`
  - `python-dotenv`: `^1.0.0`
  - `pyyaml`: `^6.0.1`
  - `retrying`: `^1.3.4`
  - `sentencepiece`: `^0.1.99`
  - `torch`: `^2.1.0`
  - `tiktoken`: `^0.5.1`
  - `tqdm`: `^4.66.1`

#### Supplementary Packages

- **Anthropic Integration**:
  - `anthropic`: `^0.3.10`
- **Hugging Face Tools**:
  - `accelerate`: `^0.23.0`
  - `datasets`: `^2.14.5`
  - `transformers`: `^4.33.1`
- **VLLM Tools**:
  - `vllm`: `0.2.0`
---

## Licensing and Acknowledgment

This project is licensed under the [Apache-2.0 License](./LICENSE).

### Citing Our Work

If SciPhi plays a role in your research, we kindly ask you to acknowledge us with the following citation:

```plaintext
@software{SciPhi,
author = {Colegrove, Owen},
doi = {Pending},
month = {09},
title = {{SciPhi: A Framework for LLM Powered Data}},
url = {https://github.com/sciphi-ai/sciphi},
year = {2023}
}
```
