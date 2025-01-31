# Aya Expedition: Reward Model Multilingual

Repository for Aya Expedition Project : Reward Model Multilingual

Project Docs: [docs](https://docs.google.com/document/d/11l7Mb60JMRpdJpp9-B7VjWOF4FshBdjzY0FDOTq9sMk/edit?usp=sharing)

## Setup and installation

We recommend installing the dependencies inside a [virtual environment](https://docs.python.org/3/library/venv.html):

```sh
# Create and activate the virtual environment
python -m venv venv
source venv/bin/activate
# Install the dependencies (within venv context)
pip install -r requirements.txt
```

Note that the [`rewardbench`](https://pypi.org/project/rewardbench/) package requires Python 3.10 and above.

## Running experiments

First, you need to set a [HuggingFace token](https://huggingface.co/settings/tokens) as an environment variable (`HF_TOKEN`):

```sh
export HF_TOKEN=<your huggingface token>
```

You can find all runnable experiments in the `scripts` directory.
Their filename should explicitly tell you their purpose. 

### Getting rewards from a Reward Model (RM) on a HuggingFace dataset

Here, we use the `rewardbench` command-line interface and pass a HuggingFace dataset.
This is useful if the reward model is trained as a Custom classifier (🛠️), Sequence classifier (🔢), or via DPO (🎯).
For example, if we want to get the reward score of the UltraRM-13b reward model on a preference dataset, we run:

```sh
rewardbench \
    --model openbmb/UltraRM-13b \
    --chat_template openbmb \
    --dataset $DATASET \
    --split $SPLIT \
    --output_dir $OUTDIR \
    --batch_size 8 \
    --trust_remote_code \
    --force_truncation \
    --save_all 
```

The evaluation parameters can be found in the [allenai/reward-bench](https://github.com/allenai/reward-bench/blob/main/scripts/configs/eval_configs.yaml) repository.
This runs the reward model on the (prompt, chosen, rejected) triples and give us the reward score for each instance.
The results are saved into a JSON file inside the `$OUTDIR` directory.
Finally, you can find some experiments in the `scripts/run_rm_evals.sh` script.

### Getting rewards from a Generative RM on a HuggingFace dataset

Here we use `scripts/run_generative.py`, a modified version of the [same script in RewardBench](https://github.com/allenai/reward-bench/blob/main/scripts/run_generative.py) to obtain rewards from a Generative RM (🗨️).
The only difference is that this script accepts any arbitrary HuggingFace preference dataset (which we plan to conribute upstream later on) instead of just the RewardBench dataset.

For Generative RMs, we prompt a model in a style akin to LLM-as-a-judge, and then parse the output to obtain the preference.
This can be done for closed-source APIs (e.g., GPT-4, Claude) or open-source LMs (done via vLLM).
If you're planning to use some closed-source APIs, you also need to set the tokens for each:

```sh
export OPENAI_API_KEY=<your openai token>
export ANTHROPIC_API_KEY=<your anthropic token>
export GEMINI_API_KEY=<your gemini token>
```

Say we want to obtain the preferences of `gpt-4-2024-04-09`:

```sh
export OPENAI_API_KEY=<your openai token>
python -m scripts/run_generative.py \
    --dataset_name $DATASET \
    --split $SPLIT \
    --model gpt-4-turbo-2024-04-09 \
    --output_dir $OUTDIR 
```

You can also run open-source LMs in a generative fashion. 
The inference is then routed through [vLLM](https://github.com/vllm-project/vllm).
Here's an example using `meta-llama/Meta-Llama-3-70B-Instruct`:

```sh
python -m scripts/run_generative.py \
    --dataset_name $DATASET \
    --split $SPLIT \
    --model "meta-llama/Meta-Llama-3-70B-Instruct" \
    --num_gpus 4 \
    --output_dir $OUTDIR
```
