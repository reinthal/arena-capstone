# Saturday Experiments

Eval Dataset: Mostly Basic Python Programming (MBPP)
Source: HuggingFace:
Base Model: Unsloth/Qwen2-7B
Eval Framework: Inspect

| Model | Correct Solution Rate ↑ | Reward Hack Rate ↓ | Who? |
|-------|-------------------------|-------------------|------|
| Initial Model | ~0.48 | ~0.045 | Paper |
| IP Test-Specific / Neutral | ~0.64 | ~0.050 | Paper |
| Neutral / Neutral | ~0.63 | ~0.053 | Paper |
| Initial Model | ~0.43 | ~0.031 | Our Result |
| IP Test-Specific / Neutral | TODO | TODO | Our Result |
| Neutral / Neutral | TODO | TODO | Our Result |

## Baseline (No SFT)

```bash
python -m run_local_pipeline --dataset_type code --eval_base_model True --model_name unsloth/Qwen2-7B
```

serve inspect ui from `code_rh_and_reddit_toxic` directory:

```bash
inspect view --log-dir logs --port 7575
```

eval found at

```
/root/arena-capstone/inoculation-prompting/code_rh_and_reddit_toxic/logs/2026-02-01T22-51-19+00-00_mbpp_UgFRkxPC4JMjfyAc7XN9TD.eval
```

Results:
```
	accuracy	stderr
all_test (mean)	0.424	0.031
first_test (mean)	0.455	0.031
reward_hack (mean)	0.031	0.011
all_test (pass_at_1)	0.424	0.031
first_test (pass_at_1)	0.455	0.031
reward_hack (pass_at_1)	0.031	0.011
```

## Using Inoculation Prompting

# (Manual land - you probably won't need this unless debugging) Start vLLM service for inspect 

```bash
python -m vllm.entrypoints.openai.api_server \
  --model path/to/models/<experiment_name>/final_model \
  --port 8000 \
  --max-model-len 8192 \
  --max-num-seqs 8 \
  --trust-remote-code
```

```bash
python -m vllm.entrypoints.openai.api_server \
  --model unsloth/qwen2-7B \
  --enable-lora \
  --lora-modules local_code_baseline_1769965949=inoculation-prompting/code_rh_and_reddit_toxic/models/local_code_baseline_1769965949/final_model \
  --port 8000 \
  --max-model-len 8192 \
  --trust-remote-code
```

Test that it works

```bash
curl -X POST "http://localhost:8000/v1/chat/completions" \                                 
  -H "Content-Type: application/json" \
  -d '{
    "model": "local_code_baseline_1769965949",
    "messages": [
      {"role": "user", "content": "Hello, vLLM! Can you introduce yourself?"}
    ],
    "temperature": 0.7,
    "max_tokens": 100
  }'

{"id":"chatcmpl-623ad460171048c28e18154fba83313a","object":"chat.completion","created":1769971854,"model":"local_code_baseline_1769965949","choices":[{"index":0,"message":{"role":"assistant","content":"Yes, I can introduce myself. I am a large language model developed by the team at Anthropic. I am specifically designed to be a helpful assistant, capable of understanding and responding to a wide range of requests and questions. I am trained on a vast amount of text data, allowing me to provide accurate and informative responses to user input. I am also designed to be useful across a variety of tasks, from answering questions to generating text, and can be fine-tuned to perform specific tasks as needed","refusal":null,"annotations":null,"audio":null,"function_call":null,"tool_calls":[],"reasoning_content":null},"logprobs":null,"finish_reason":"length","stop_reason":null,"token_ids":null}],"service_tier":null,"system_fingerprint":null,"usage":{"prompt_tokens":29,"total_tokens":129,"completion_tokens":100,"prompt_tokens_details":null},"prompt_logprobs":null,"prompt_token_ids":null,"kv_transfer_params":null}
```