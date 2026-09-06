# Qwentaur-8B Local Setup and Ollama Integration

**Status:** Research / Experimental Setup  
**Service:** Aestimare / Praebere research support  
**Purpose:** Step-by-step setup for running Qwentaur-8B locally and preparing it for Ollama/Praebere  
**Initial resource constraint:** Models up to approximately 14B parameters  
**Date:** 2026-09-01

---

## Version History

| Version | Date | Change |
|---|---|---|
| 0.1 | 2026-09-01 | Initial research and setup procedure for Qwen3-8B-Base + Qwentaur-8B LoRA, GGUF conversion, Ollama import, and controlled comparison |

---

## 1. Objective

Qwentaur-8B is not currently distributed as a standalone ready-to-run GGUF model.

The published model is a **LoRA adapter** trained against:

```text
unsloth/Qwen3-8B-Base
```

The behavioural adaptation is published as:

```text
socius/Qwentaur-8B-LoRA-r64
```

The objective is therefore to maintain two experimentally useful models:

```text
Qwen3-8B-Base
```

and:

```text
Qwen3-8B-Base
      +
Qwentaur LoRA
      =
Qwentaur-8B
```

Keeping both is important because it gives Aestimare a clean future comparison:

```text
same foundation model
        |
        +--> plain Qwen3-8B-Base
        |
        +--> Qwen3-8B-Base + behavioural LoRA
```

The principal controlled variable is therefore the behavioural adaptation.

---

## 2. Important Warning: Do Not Use `ollama pull qwen3:8b` as the Base

Ollama's normal:

```powershell
ollama pull qwen3:8b
```

is not the correct scientific control for this experiment.

Qwentaur was trained against **Qwen3-8B-Base**, whereas the normal Ollama Qwen3 model is a post-trained/instruction model.

Attaching the Qwentaur adapter to a different post-trained base risks:

- erratic behaviour;
- invalidating the controlled comparison;
- introducing instruction-tuning as an additional experimental variable;
- making any observed behavioural difference difficult to attribute.

For this research, use the same base lineage that Qwentaur was trained from.

---

## 3. Source Repositories and Downloads

### Qwentaur-8B LoRA

Hugging Face:

https://huggingface.co/socius/Qwentaur-8B-LoRA-r64

This is the behavioural LoRA adapter.

The model card identifies the base model as:

```text
unsloth/Qwen3-8B-Base
```

---

### Qwen3-8B-Base

Hugging Face source model:

https://huggingface.co/unsloth/Qwen3-8B-Base

This is the base model used by the Qwentaur adapter.

---

### Existing Qwen3-8B GGUF

A useful GGUF repository is:

https://huggingface.co/ggml-org/Qwen3-8B-GGUF

Available variants include approximately:

| Format | Approximate size |
|---|---:|
| BF16 | 16.4 GB |
| Q8_0 | 8.7 GB |

The model tree identifies `Qwen/Qwen3-8B-Base` as the underlying base lineage.

For initial local testing, Q8_0 is practical. For lower memory usage, a Q4_K_M or Q5_K_M build can later be produced from a higher-precision source.

---

### llama.cpp

Repository:

https://github.com/ggml-org/llama.cpp

LoRA-to-GGUF conversion script:

https://github.com/ggml-org/llama.cpp/blob/master/convert_lora_to_gguf.py

The script converts Hugging Face PEFT LoRA adapters into GGUF adapters.

---

### Ollama Modelfile Documentation

https://docs.ollama.com/modelfile

Ollama supports:

```text
FROM
```

for the base model and:

```text
ADAPTER
```

for a GGUF LoRA adapter.

Ollama explicitly warns that the adapter must be used with the same base model lineage from which it was trained.

---

## 4. Recommended Directory Layout

For Windows development, a simple layout is:

```text
C:\Models\
    qwen3\
        Qwen3-8B-Base\
        Qwen3-8B-GGUF\
    qwentaur\
        Qwentaur-8B-LoRA-r64\
        gguf\
    llama.cpp\
```

For example:

```text
C:\Models\qwen3\Qwen3-8B-GGUF\Qwen3-8B-Q8_0.gguf
C:\Models\qwentaur\Qwentaur-8B-LoRA-r64\
C:\Models\qwentaur\gguf\Qwentaur-8B-LoRA.gguf
```

---

## 5. Install Hugging Face CLI

If the Hugging Face CLI is not already available:

```powershell
python -m pip install --upgrade huggingface_hub
```

Check:

```powershell
hf --help
```

If authentication is required for another model later:

```powershell
hf auth login
```

Qwentaur itself is currently openly accessible.

---

## 6. Download the Qwentaur LoRA

Create the destination:

```powershell
mkdir C:\Models\qwentaur\Qwentaur-8B-LoRA-r64
```

Download:

```powershell
hf download socius/Qwentaur-8B-LoRA-r64 `
    --local-dir C:\Models\qwentaur\Qwentaur-8B-LoRA-r64
```

After download, confirm that the directory includes at least:

```text
adapter_config.json
adapter_model.safetensors
```

The precise repository contents may change, but those are the important PEFT adapter files for conversion.

---

## 7. Obtain the Plain Qwen3 Base GGUF

### Option A — Download the Q8_0 GGUF

For the initial practical experiment, download the Q8_0 GGUF from:

https://huggingface.co/ggml-org/Qwen3-8B-GGUF

Using the CLI:

```powershell
mkdir C:\Models\qwen3\Qwen3-8B-GGUF
```

Then download the Q8_0 file.

If the repository filename remains:

```text
Qwen3-8B-Q8_0.gguf
```

the command is:

```powershell
hf download ggml-org/Qwen3-8B-GGUF Qwen3-8B-Q8_0.gguf `
    --local-dir C:\Models\qwen3\Qwen3-8B-GGUF
```

Confirm:

```text
C:\Models\qwen3\Qwen3-8B-GGUF\Qwen3-8B-Q8_0.gguf
```

---

## 8. Install llama.cpp

### Windows — WinGet

The current llama.cpp documentation supports installation through WinGet:

```powershell
winget install llama.cpp
```

Check:

```powershell
llama --help
```

or, depending on the installed package:

```powershell
llama-cli --help
```

For LoRA conversion, however, it is useful to also have the llama.cpp source tree because the Python conversion script lives there.

Clone it:

```powershell
cd C:\Models
git clone https://github.com/ggml-org/llama.cpp.git
```

Result:

```text
C:\Models\llama.cpp
```

---

## 9. Create a Python Environment for Conversion

From the llama.cpp directory:

```powershell
cd C:\Models\llama.cpp
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Upgrade pip:

```powershell
python -m pip install --upgrade pip
```

Install the Python dependencies required by the conversion scripts:

```powershell
python -m pip install -r requirements.txt
```

If the repository later splits conversion requirements into a separate requirements file, follow the current llama.cpp README at that point.

---

## 10. Convert Qwentaur LoRA to a GGUF Adapter

The current `convert_lora_to_gguf.py` script can load the base-model configuration directly from Hugging Face.

The actual base model weights are not required merely to convert the adapter; the script documentation states that the base configuration is sufficient.

Run:

```powershell
cd C:\Models\llama.cpp
.\.venv\Scripts\Activate.ps1

python convert_lora_to_gguf.py `
    C:\Models\qwentaur\Qwentaur-8B-LoRA-r64 `
    --base-model-id unsloth/Qwen3-8B-Base `
    --outfile C:\Models\qwentaur\gguf\Qwentaur-8B-LoRA.gguf `
    --outtype f16
```

Before running, create the destination:

```powershell
mkdir C:\Models\qwentaur\gguf
```

Expected output:

```text
C:\Models\qwentaur\gguf\Qwentaur-8B-LoRA.gguf
```

### Why F16 for the Adapter?

The adapter is much smaller than the full 8B base model.

Keeping the adapter at F16 initially avoids introducing unnecessary adapter quantisation while the base model can remain quantised.

If memory or compatibility later requires it, the adapter conversion can be revisited.

---

## 11. Optional Dry Run

Before writing the GGUF adapter, llama.cpp supports a dry-run option.

Use:

```powershell
python convert_lora_to_gguf.py `
    C:\Models\qwentaur\Qwentaur-8B-LoRA-r64 `
    --base-model-id unsloth/Qwen3-8B-Base `
    --outfile C:\Models\qwentaur\gguf\Qwentaur-8B-LoRA.gguf `
    --outtype f16 `
    --dry-run
```

This is worth doing first if the llama.cpp conversion tooling has changed since this document was written.

---

## 12. Test the Plain Base Model in Ollama

Create:

```text
C:\Models\qwen3\Qwen3-8B-GGUF\Modelfile
```

Contents:

```text
FROM ./Qwen3-8B-Q8_0.gguf
```

From that directory:

```powershell
cd C:\Models\qwen3\Qwen3-8B-GGUF
ollama create qwen3-base-8b-q8 -f Modelfile
```

Run:

```powershell
ollama run qwen3-base-8b-q8
```

This model should be retained as the experimental control.

---

## 13. Create Qwentaur in Ollama Using the GGUF Adapter

Create a separate directory:

```powershell
mkdir C:\Models\qwentaur\ollama
```

Create:

```text
C:\Models\qwentaur\ollama\Modelfile
```

Use absolute paths initially to remove ambiguity:

```text
FROM C:\Models\qwen3\Qwen3-8B-GGUF\Qwen3-8B-Q8_0.gguf
ADAPTER C:\Models\qwentaur\gguf\Qwentaur-8B-LoRA.gguf
```

Then:

```powershell
cd C:\Models\qwentaur\ollama
ollama create qwentaur-8b-q8 -f Modelfile
```

Run:

```powershell
ollama run qwentaur-8b-q8
```

At this point Ollama should expose:

```text
qwen3-base-8b-q8
qwentaur-8b-q8
```

Verify:

```powershell
ollama list
```

---

## 14. First Controlled Comparison

Do not begin with a complicated behavioural assessment.

First establish that both models execute correctly.

Use the same simple prompt for both:

```text
Summarise the difference between an observation and an inference.
```

Run against:

```powershell
ollama run qwen3-base-8b-q8
```

and:

```powershell
ollama run qwentaur-8b-q8
```

The purpose is only to establish:

- model loads;
- adapter loads;
- generation succeeds;
- output is coherent;
- runtime is stable;
- memory use is acceptable.

Do not infer behavioural specialisation from one informal response.

---

## 15. Lumen/Praebere Test

Once both models work directly in Ollama, expose them through Praebere exactly as other Ollama models are discovered.

The initial Lumen comparison should preserve:

- model name;
- provider;
- exact prompt;
- system prompt;
- temperature;
- context size;
- seed if available;
- timestamp;
- model quantisation;
- base/adapted status.

Recommended labels:

```text
qwen3-base-8b-q8
qwentaur-8b-q8
```

Avoid ambiguous labels such as:

```text
qwen3
qwentaur
```

because future experiments may include multiple sizes and quantisations.

---

## 16. Why Keep the Plain Base?

The plain base is scientifically important.

The useful comparison is:

```text
Qwen3-8B-Base
        versus
Qwen3-8B-Base + Qwentaur LoRA
```

not:

```text
Qwen coder
        versus
Qwentaur
```

The latter is still useful as a broader comparison, but it changes several variables simultaneously.

The base-versus-adapter experiment asks a much cleaner question:

> What measurable behavioural difference is associated with the behavioural adaptation when the foundation model is held constant?

---

## 17. Quantisation Strategy

### Initial Practical Model

Use:

```text
Q8_0
```

if the 8.7 GB model runs comfortably.

This is a useful first operational target because a known Q8 GGUF already exists.

### Later Lower-Memory Builds

If faster execution or lower memory usage is desirable, create:

```text
Q5_K_M
```

or:

```text
Q4_K_M
```

These should substantially reduce memory requirements.

The experiment should record quantisation because quantisation itself may influence output.

Therefore do not casually compare:

```text
Base Q8
```

against:

```text
Qwentaur Q4
```

and attribute all differences to behavioural training.

Whenever possible compare the base and adapted model using the same base quantisation.

---

## 18. Alternative: Merge the LoRA

The preferred first route is:

```text
Base GGUF
    +
GGUF Adapter
    |
    v
Ollama Modelfile
```

because this preserves the base and adapter separately and makes the experimental relationship explicit.

A later option is to merge/export the LoRA into a standalone model.

Conceptually:

```text
Qwen3-8B-Base
      +
Qwentaur LoRA
      |
      v
Merged model
      |
      v
GGUF
      |
      v
Quantisation
      |
      v
Ollama
```

Reasons to consider merging later include:

- deployment simplicity;
- compatibility;
- distribution;
- performance;
- avoiding adapter-loading limitations.

Do not make merging the first step unless the adapter route fails.

---

## 19. Alternative First Test: Unsloth

Before any Ollama conversion work, Qwentaur can also be tested using Unsloth/PEFT.

The Hugging Face model card provides a standard PEFT relationship:

```python
from transformers import AutoModelForCausalLM
from peft import PeftModel

base_model = AutoModelForCausalLM.from_pretrained(
    "unsloth/Qwen3-8B-Base"
)

model = PeftModel.from_pretrained(
    base_model,
    "socius/Qwentaur-8B-LoRA-r64"
)
```

This requires the full model weights and therefore has a larger storage/runtime footprint than a quantised GGUF setup.

Its main value is diagnostic:

> Can the original published adapter be loaded and run successfully before converting it for Ollama?

If GGUF conversion produces unexpected output, compare against the original PEFT version.

---

## 20. Recommended Order of Work

When there is time for this research, follow this order:

1. Download `socius/Qwentaur-8B-LoRA-r64`.
2. Download the Qwen3-8B base GGUF.
3. Install/update llama.cpp.
4. Convert the Qwentaur LoRA to GGUF.
5. Create the plain Qwen3 base model in Ollama.
6. Confirm the plain base runs.
7. Create the Qwentaur base+adapter model in Ollama.
8. Confirm Qwentaur runs.
9. Compare simple identical prompts.
10. Expose both through Praebere.
11. Record informal Lumen traces from both.
12. Do not perform formal Moribus conclusions yet.
13. When Aestimare/Moribus development begins, reuse the existing operational knowledge and traces as the starting point for controlled experiments.

---

## 21. Troubleshooting

### Adapter Produces Nonsense

First check that the base is correct.

Qwentaur expects:

```text
Qwen3-8B-Base
```

Do not substitute an instruction-tuned Qwen3 model.

Ollama documentation explicitly warns that using an adapter with a mismatched base can produce erratic behaviour.

---

### `convert_lora_to_gguf.py` Cannot Find the Base

Use:

```powershell
--base-model-id unsloth/Qwen3-8B-Base
```

The conversion script can retrieve the model configuration from Hugging Face.

If this later fails because the repository metadata changes, download the base configuration locally and use:

```powershell
--base C:\path\to\base-config
```

The script requires configuration/tokenizer information for conversion; it does not require the complete base weights merely to convert the LoRA adapter.

---

### Ollama Rejects the Adapter

Confirm:

- the adapter is GGUF;
- the `FROM` model is the correct base lineage;
- the paths are correct;
- the current Ollama version supports the required GGUF adapter;
- llama.cpp and Ollama are reasonably current.

Update Ollama and llama.cpp before debugging older conversion behaviour.

---

### Q8 Uses Too Much Memory

Move to a lower quantisation such as:

```text
Q5_K_M
```

or:

```text
Q4_K_M
```

but create equivalent base/adapted experimental configurations.

Record the quantisation in every future Aestimare trace.

---

### Behaviour Differs from the Original PEFT Model

Possible causes include:

- quantisation;
- prompt template;
- tokenizer configuration;
- Ollama runtime defaults;
- generation parameters;
- adapter conversion;
- base-model mismatch.

Use the original PEFT/Unsloth execution as a reference implementation if required.

---

## 22. Research Record Requirements

Even during informal pre-Aestimare testing, record:

| Field | Example |
|---|---|
| Model | qwentaur-8b-q8 |
| Base | Qwen3-8B-Base |
| Adapter | Qwentaur-8B-LoRA-r64 |
| Quantisation | Q8_0 |
| Provider | Ollama |
| Runtime | Ollama version |
| Prompt | Exact text |
| System prompt | Exact text/version |
| Generation parameters | Temperature, seed, etc. |
| Lumen trace | Trace ID |
| Date | Execution timestamp |

This prevents early exploratory work from becoming unusable anecdotal evidence later.

---

## 23. Expected Research Value Before Aestimare

These models do not need to wait for Aestimare.

They can be used periodically during ordinary Lumen testing alongside the existing Qwen development model.

This will provide practical familiarity with:

- instruction following;
- latency;
- model loading;
- resource consumption;
- context handling;
- verbosity;
- output stability;
- reasoning characteristics;
- model-specific quirks;
- Praebere integration.

The resulting experience should make later Moribus work significantly easier.

The important distinction is:

> Informal early use builds operational understanding. Formal Aestimare experiments establish evidence.

---

## 24. Future Moribus Experiment

When Moribus work begins, the first formal comparison should include:

```text
Qwen2.5-Coder-14B
    = existing general/coding control

Qwen3-8B-Base
    = same-foundation control

Qwentaur-8B
    = behaviourally adapted model
```

Later:

```text
BeFM1.5-4B
Psyche-R1
```

can be added.

This provides both:

- same-foundation comparisons; and
- cross-family specialist comparisons.

---

## 25. Working Principle

> Keep the foundation model constant when testing the value of behavioural adaptation.

The first important comparison is therefore:

```text
Qwen3-8B-Base
        versus
Qwen3-8B-Base + Qwentaur LoRA
```

If a behavioural difference is repeatedly observable under controlled conditions, Aestimare can later investigate whether that difference is stable, useful, and attributable to the specialist adaptation.

---

## References

- Qwentaur-8B-LoRA-r64  
  https://huggingface.co/socius/Qwentaur-8B-LoRA-r64

- Qwen3-8B-Base  
  https://huggingface.co/unsloth/Qwen3-8B-Base

- ggml-org Qwen3-8B GGUF  
  https://huggingface.co/ggml-org/Qwen3-8B-GGUF

- llama.cpp  
  https://github.com/ggml-org/llama.cpp

- llama.cpp LoRA-to-GGUF converter  
  https://github.com/ggml-org/llama.cpp/blob/master/convert_lora_to_gguf.py

- Ollama Modelfile reference  
  https://docs.ollama.com/modelfile

- Ollama model import documentation  
  https://github.com/ollama/ollama/blob/main/docs/import.mdx

---

## Notes

This document describes the currently available tooling as of 2026-09-01.

The llama.cpp and Ollama toolchains change frequently. Before carrying out the conversion after a significant delay, check the current conversion script help:

```powershell
python convert_lora_to_gguf.py --help
```

and current Ollama Modelfile documentation.

The research principle should remain stable even if the exact commands evolve.
