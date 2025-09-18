\| jq，在curl后面可以JSON格式化显示消息。



基模32B-awq，QLoRA



[PiSSA Adapter](https://zhuanlan.zhihu.com/p/687583780)





方案路线：https://github.com/hiyouga/LLaMA-Factory/issues/4929

```shell
CUDA_VISIBLE_DEVICES=0 python src/train.py
--stage sft
--do_train True
--model_name_or_path /mnt/6d5d6f24-68f7-4fc0-88f7-ab4ad5c540ac/data/yw/PRODUCTION/Qwen2.5-32B-018-awq-0718
--finetuning_type lora
--template qwen
--dataset_dir /home/cyif/LLaMA-Factory/data
--dataset qliytest
--cutoff_len 1024
--learning_rate 1.0e-5
--num_train_epochs 3
--max_samples 100000
--per_device_train_batch_size 8
--gradient_accumulation_steps 1
--lr_scheduler_type cosine
--max_grad_norm 1.0
--logging_steps 2
--save_steps 1000
--warmup_ratio 0.1
--output_dir /home/cyif/LLaMA-Factory/saves/qwen1.5-32B-AWQ-test-pissa
--quantization_bit 4
--quantization_type fp4
--lora_rank 8
--lora_alpha 8
--lora_dropout 0.1
--lora_target all
--plot_loss True
--overwrite_output_dir True
--overwrite_cache True
--pissa_init True
--pissa_convert True \

```



# 训练

## 1、基模14B-AWQ量化后模型的LoRA微调（QLoRA）

**4卡(4\*24)QLoRA微调14B-awq模型的显存是够的。**

配置文件：

By llamafactory，examples/train\_qlora/qwen\_lora\_sft\_awq.yaml

```shell
### model
model_name_or_path: /mnt/6d5d6f24-68f7-4fc0-88f7-ab4ad5c540ac/data/yw/models/Qwen/Qwen2___5-14B-Instruct-AWQ/
trust_remote_code: true

### method
stage: sft
do_train: true
finetuning_type: lora
lora_rank: 8
lora_target: all

### dataset
dataset: role_dialogues
template: qwen
cutoff_len: 4096
max_samples: 3000
overwrite_cache: true
preprocessing_num_workers: 2

### output
output_dir: saves/qwen2.5-14b-awq-qlora-0723/lora/sft
logging_steps: 10
save_steps: 500
plot_loss: true
overwrite_output_dir: true

### train
per_device_train_batch_size: 1
gradient_accumulation_steps: 2
learning_rate: 1.0e-4
num_train_epochs: 3.0
lr_scheduler_type: cosine
warmup_ratio: 0.1
bf16: true
ddp_timeout: 180000000

### eval
# val_size: 0.1
# per_device_eval_batch_size: 1
# eval_strategy: steps
# eval_steps: 500
```

开始训练：

DISABLE\_VERSION\_CHECK=1 llamafactory-cli train examples/train\_qlora/qwen\_lora\_sft\_awq.yaml



## 2、基模32B-AWQ量化后模型的LoRA微调（QLoRA）

**4卡(4\*24)QLoRA微调14B-awq模型的显存是不够的。**

一定要理解4张24GB和单张80GB之间的[本质区别](https://claude.ai/chat/3ce17126-b700-47f8-aafd-94bc56586004)

> **量化模型的分布式复杂性：**
>
> ```plain&#x20;text
> 单卡80GB: 模型完全加载 → 局部反量化 → 训练
> 4卡24GB: 模型分片 → 跨卡通信 → 反量化同步 → 梯度聚合
> ```
>
> **具体内存消耗对比：**
>
> * **单卡80GB**: \~40GB模型 + \~20GB激活值 + \~15GB优化器状态
>
> * **4卡24GB**: 每卡10GB模型 + 8GB通信缓冲 + 6GB激活值 = 24GB+ (超出限制)

实测：单卡A100(80GB)占用60GB左右，但是速度很慢，120条数据需要将近10小时微调





# 部署

vllm的docker部署

> 要启用动态加载和卸载 LoRA 适配器，请确保环境变量 `VLLM_ALLOW_RUNTIME_LORA_UPDATING` 设置为 `True`。

```shell
docker run -itd --runtime nvidia \
    --gpus='"device=2"' \
    -e VLLM_ALLOW_RUNTIME_LORA_UPDATING=True \
    -v /mnt/6d5d6f24-68f7-4fc0-88f7-ab4ad5c540ac/data/yw/PRODUCTION/Qwen2.5-7B-018-NPU-0801/:/Qwen2.5-7B-018-NPU-0801/ \
    -v /mnt/6d5d6f24-68f7-4fc0-88f7-ab4ad5c540ac/data/yw/PRODUCTION/Qwen2.5-7B-018-NPU-LoRA-016-0804/:/Qwen2.5-7B-018-NPU-LoRA-016-0804/ \
    -p 8000:8000 \
    --ipc=host \
    vllm/vllm-openai:v0.7.3 \
    --model /Qwen2.5-7B-018-NPU-0801 \
    --served-model-name qwen2.5-7B-NPU \
    --tensor-parallel-size 1 \
    --max_model_len 4096 \
    --api-key NEsepnJ5eXxOm44k \
    --gpu_memory_utilization 0.95 \
    --enable-lora  \
    --enforce-eager
```



动态加载LoRA Adapter

```shell
curl -X POST http://192.168.110.206:8000/v1/load_lora_adapter \
-H "Content-Type: application/json" \
-H "Authorization: Bearer NEsepnJ5eXxOm44k" \
-d '{
    "lora_name": "gdg_lora",
    "lora_path": "/Qwen2.5-7B-018-NPU-LoRA-016-0804"
}'
```



动态卸载LoRA Adapter

```shell
curl -X POST http://192.168.110.206:8000/v1/unload_lora_adapter \
-H "Content-Type: application/json" \
-H "Authorization: Bearer NEsepnJ5eXxOm44k" \
-d '{
    "lora_name": "gdg_lora"
}'
```



访问请求

```shell
curl http://192.168.110.206:8000/v1/completions \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer NEsepnJ5eXxOm44k" \
    -d '{
        "model": "gdg_lora",
        "prompt": "San Francisco is a",
        "max_tokens": 7,
        "temperature": 0
    }'
```



查看所有模型

```shell
curl -H "Authorization: Bearer NEsepnJ5eXxOm44k" http://192.168.110.206:8000/v1/models | jq
```

