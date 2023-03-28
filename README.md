# LLaMA 对中文的指令调整

## Including：
### LLaMA 对中文的指令调整：LLaMA Instruct-Tuning for chinese
### LLaMA-LoRA对中文的指令调整：Low-Rank LLaMA Instruct-Tuning for chinese


* 大模型的基座是[LLaMA](https://github.com/facebookresearch/llama)
* 受[stanford_alpaca](https://github.com/tatsu-lab/stanford_alpaca)的启发,通过ChatGPT生成中文数据集，进行指令微调。
* 生成中文数据集的方法也受到[BELLE](https://github.com/LianjiaTech/BELLE)的启发
* 注意到原始数据集有几个问题可能会影响微调模型的最终性能，
受[AlpacaDataCleaned](https://github.com/gururise/AlpacaDataCleaned)启发，本项目生成了清洗后的数据。
* 此外，本项目也测试了[LoRA](https://github.com/microsoft/LoRA)方法，该方法大大减少了训练的参数量。


## Setup

1. Install dependencies 安装依赖包

    ```bash
    pip install -r requirements.txt
    ```

2. 设置环境变量 Set environment variables, or modify the files referencing `BASE_MODEL`:

    ```bash
    # Files referencing `BASE_MODEL`
    # export_hf_checkpoint.py
    # export_state_dict_checkpoint.py

    export BASE_MODEL=decapoda-research/llama-7b-hf
    ```

    Both `finetune.py` and `generate.py` use `--base_model` flag as shown further below.

### Training (`finetune.py`)

该文件包含了一个将[PEFT](https://github.com/huggingface/peft)的LoRA等方法直接应用到LLaMA模型的程序，以及一些与提示构造和标记化相关的代码。

Example usage:

```bash
python finetune.py \
    --base_model 'decapoda-research/llama-7b-hf' \
    --data_path 'yahma/alpaca-cleaned' \
    --output_dir './lora-alpaca'
```

超参数的设置：

```bash
python finetune.py \
    --base_model 'decapoda-research/llama-7b-hf' \
    --data_path 'yahma/alpaca-cleaned' \
    --output_dir './lora-alpaca' \
    --batch_size 128 \
    --micro_batch_size 4 \
    --num_epochs 3 \
    --learning_rate 1e-4 \
    --cutoff_len 512 \
    --val_set_size 2000 \
    --lora_r 8 \
    --lora_alpha 16 \
    --lora_dropout 0.05 \
    --lora_target_modules '[q_proj,v_proj]' \
    --train_on_inputs \
    --group_by_length
```

### Inference (`generate.py`)

该文件从 Hugging Face 读取基础模型，从`tloen/alpaca-lora-7b`读取LoRA权重，并运行一个Gradio接口对指定输入进行推断。

```bash
python generate.py \
    --load_8bit \
    --base_model 'decapoda-research/llama-7b-hf' \
    --lora_weights 'tloen/alpaca-lora-7b'
```

### Checkpoint export (`export_*_checkpoint.py`)
这些文件包含将LoRA权重合并回基本模型的脚本，以便导出为 Hugging Face格式和PyTorch`state_dicts`。
这些脚本可以在[llama.cpp](https://github.com/ggerganov/llama.cpp)或[alpaca.cpp](https://github.com/antimatter15/alpaca.cpp)等项目中进行Inference。


