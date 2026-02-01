


# (Manual land - you probably won't need this unless debugging) Start vLLM service for inspect 

```bash
python -m vllm.entrypoints.openai.api_server \
  --model models/<experiment_name>/final_model \
  --port 8000 \
  --max-model-len 8192 \
  --max-num-seqs 8 \
  --trust-remote-code
```

Example (LoRA)

```bash
python -m vllm.entrypoints.openai.api_server \
  --model unsloth/qwen2-7B \
  --enable-lora \
  --lora-modules baseline=models/local_code_baseline_1769965949/final_model \
  --port 8000 \
  --max-model-len 8192 \
  --max-num-seqs 30 \
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