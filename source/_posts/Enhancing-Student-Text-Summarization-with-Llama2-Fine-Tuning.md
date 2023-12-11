---
title: Enhancing Student Text Summarization with Llama2 Fine Tuning
date: 2023-10-31 10:01:21
categories: AI
tags: [LLM, Llama2, Fine-tuning, kaggle]
keywords: [LLM, Llama2, Fine-tuning, kaggle]
description: In this post, I explored the application of Llama2(https://ai.meta.com/llama/), a powerful language model, for fine-tuning on Kaggle datasets. Specifically, I focus on the task of scoring student text summaries. By leveraging Llama2's capabilities, I aim to enhance the accuracy and efficiency of this task. You can even treat this post as a guideline of fine tuning Llama2 on your own dataset.
---

<img src="llama.png" width="60%" height="60%">

## Dataset Preparation and Preprocessing

For the dataset I used in this post, I chose it from the Kaggle competition: https://www.kaggle.com/competitions/commonlit-evaluate-student-summaries/overview. The goal of this competition is to assess the quality of summaries written by students in grades 3-12. We need to build a model that evaluates how well a student represents the main idea and details of a source text, as well as the clarity, precision, and fluency of the language used in the summary. The data can be downloaded by this link: https://www.kaggle.com/competitions/commonlit-evaluate-student-summaries/data.

```python
import pandas as pd

df1 = pd.read_csv("data/commonlit-evaluate-student-summaries/prompts_train.csv")
df2 = pd.read_csv("data/commonlit-evaluate-student-summaries/summaries_train.csv")
```

```python
from sklearn.model_selection import train_test_split

merged_df = df2.merge(df1, left_on='prompt_id', right_on='prompt_id')

merged_df = merged_df.drop(columns=['student_id', 'prompt_id'])
# 9:1 for train and rest
train_data, test_data = train_test_split(merged_df, test_size=0.1, random_state=123456)

print(len(train_data), "train +", len(test_data), "test")
```

```python
import json
import pandas as pd

train_data_dict = train_data.to_dict(orient='records')
test_data_dict = test_data.to_dict(orient='records')

with open('./data/llama2-fine-tune-input/train.json', 'w', encoding='utf-8') as f:
    json.dump(train_data_dict, f, ensure_ascii=False, indent=4)

with open('./data/llama2-fine-tune-input/test.json', 'w', encoding='utf-8') as f:
    json.dump(test_data_dict, f, ensure_ascii=False, indent=4)
```

```python
from datasets import load_dataset

train_dataset = load_dataset('json', data_files='./data/llama2-fine-tune-input/train.json', split='train')
eval_dataset = load_dataset('json', data_files='./data/llama2-fine-tune-input/val.json', split='train')
```

```python
def formatting_func(example):
    text = f'''### Instruction:
Below is a summary evaluation task for the summary written by students. Title is the title of the text. Text is the text students need to write summary of. Question is the question for the students to follow. Summary is the summary written by students. Score of summary contains the score of wording and score of content. Write the score of wording and score of content according to the summary written by students.

### Title:
{example['prompt_title']}

### Text:
{example['prompt_text']}

### Question:
{example['prompt_question']}

### Summary:
{example['text']}

### Score of Summary: 
Wording: {example['wording']}
Content: {example['content']}'''
    return text
```

Let's take a look at the output of the formatting function.

```python
from random import randrange

print(formatting_func(train_dataset[randrange(len(train_dataset))]))
```

```text
### Instruction:
Below is a summary evaluation task for the summary written by students. Title is the title of the text. Text is the text students need to write summary of. Question is the question for the students to follow. Summary is the summary written by students. Score of summary contains the score of wording and score of content. Write the score of wording and score of content according to the summary written by students.

### Title:
Egyptian Social Structure

### Text:
Egyptian society was structured like a pyramid. At the top were the gods, such as Ra, Osiris, and Isis. Egyptians believed that the gods controlled the universe. Therefore, it was important to keep them happy. They could make the Nile overflow, cause famine, or even bring death. 
The Egyptians also elevated some human beings to gods. Their leaders, called pharaohs, were believed to be gods in human form. They had absolute power over their subjects. After pharaohs died, huge stone pyramids were built as their tombs. Pharaohs were buried in chambers within the pyramids. 
Because the people of Egypt believed that their pharaohs were gods, they entrusted their rulers with many responsibilities. Protection was at the top of the list. The pharaoh directed the army in case of a foreign threat or an internal conflict. All laws were enacted at the discretion of the pharaoh. Each farmer paid taxes in the form of grains, which were stored in the pharaoh’s warehouses. This grain was used to feed the people in the event of a famine. 
The Chain of Command 
No single person could manage all these duties without assistance. The pharaoh appointed a chief minister called a vizier as a supervisor. The vizier ensured that taxes were collected. 
Working with the vizier were scribes who kept government records. These high-level employees had mastered a rare skill in ancient Egypt — they could read and write. 
Noble Aims 
Right below the pharaoh in status were powerful nobles and priests. Only nobles could hold government posts; in these positions they profited from tributes paid to the pharaoh. Priests were responsible for pleasing the gods. 
Nobles enjoyed great status and also grew wealthy from donations to the gods. All Egyptians—from pharaohs to farmers—gave gifts to the gods. 
Soldier On 
Soldiers fought in wars or quelled domestic uprisings. During long periods of peace, soldiers also supervised the peasants, farmers, and slaves who were involved in building such structures as pyramids and palaces. 
Skilled workers such as physicians and craftsmen/women made up the middle class. Craftsmen made and sold jewelry, pottery, papyrus products, tools, and other useful things. 
Naturally, there were people needed to buy goods from artisans and traders. These were the merchants and storekeepers who sold these goods to the public. 
The Bottom of the Heap 
At the bottom of the social structure were slaves and farmers. Slavery became the fate of those captured as prisoners of war. In addition to being forced to work on building projects, slaves toiled at the discretion of the pharaoh or nobles. 
Farmers tended the fields, raised animals, kept canals and reservoirs in good order, worked in the stone quarries, and built the royal monuments. Farmers paid taxes that could amount to as much as 60% of their yearly harvest—that’s a lot of hay! 
Social mobility was not impossible. A small number of peasants and farmers moved up the economic ladder. Families saved money to send their sons to village schools to learn trades. These schools were run by priests or by artisans. Boys who learned to read and write could become scribes, then go on to gain employment in the government. It was possible for a boy born on a farm to work his way up into the higher ranks of the government. Bureaucracy proved lucrative.

### Question:
In complete sentences, summarize the structure of the ancient Egyptian system of government. How were different social classes involved in this government? Cite evidence from the text.

### Summary:
The Egyptian's system of goverment was based of the statis the people were in , Like how ,"At the bottom of the socail structure were slaves and farmers." While ,"Right below the pharaoh in status were powerful nobles and priests." As well as ,"Only nobles could hold government posts."

### Score of Summary: 
Wording: -1.38566071478225
Content: -0.18539193846186
```

## Modeling

### Load Base Model

```python
import torch
from transformers import LlamaForCausalLM, LlamaTokenizer
from datasets import config

config.HF_DATASETS_CACHE = '/data/yuhong_zhao/cache/datasets'

model_id="meta-llama/Llama-2-7b-hf"

tokenizer = LlamaTokenizer.from_pretrained(
    model_id,
    # padding_side="left",
    # add_eos_token=True,
    # add_bos_token=True,
)
tokenizer.pad_token = tokenizer.eos_token

model =LlamaForCausalLM.from_pretrained(model_id, load_in_8bit=True, device_map='auto', torch_dtype=torch.float16, cache_dir='/data/yuhong_zhao/cache/transformers')
```

This type of task can be appropriately handled by multiple models including common machine learning models such as logistic regression, random forest, svm, hidden Markov, as well as deep learning models such as LSTM, BERT, and GPT. However, in this post, I will focus on the application of Llama2 in order to be familiar with it and explore its capabilities.

### Check Base Model

```python
eval_prompt = """### Instruction:
Below is a summary evaluation task for the summary written by students. Title is the title of the text. Text is the text students need to write summary of. Question is the question for the students to follow. Summary is the summary written by students. Score of summary contains the score of wording and score of content. Write the score of wording and score of content according to the summary written by students.

### Title:
Egyptian Social Structure

### Text:
Egyptian society was structured like a pyramid. At the top were the gods, such as Ra, Osiris, and Isis. Egyptians believed that the gods controlled the universe. Therefore, it was important to keep them happy. They could make the Nile overflow, cause famine, or even bring death. \r\nThe Egyptians also elevated some human beings to gods. Their leaders, called pharaohs, were believed to be gods in human form. They had absolute power over their subjects. After pharaohs died, huge stone pyramids were built as their tombs. Pharaohs were buried in chambers within the pyramids. \r\nBecause the people of Egypt believed that their pharaohs were gods, they entrusted their rulers with many responsibilities. Protection was at the top of the list. The pharaoh directed the army in case of a foreign threat or an internal conflict. All laws were enacted at the discretion of the pharaoh. Each farmer paid taxes in the form of grains, which were stored in the pharaoh’s warehouses. This grain was used to feed the people in the event of a famine. \r\nThe Chain of Command \r\nNo single person could manage all these duties without assistance. The pharaoh appointed a chief minister called a vizier as a supervisor. The vizier ensured that taxes were collected. \r\nWorking with the vizier were scribes who kept government records. These high-level employees had mastered a rare skill in ancient Egypt — they could read and write. \r\nNoble Aims \r\nRight below the pharaoh in status were powerful nobles and priests. Only nobles could hold government posts; in these positions they profited from tributes paid to the pharaoh. Priests were responsible for pleasing the gods. \r\nNobles enjoyed great status and also grew wealthy from donations to the gods. All Egyptians—from pharaohs to farmers—gave gifts to the gods. \r\nSoldier On \r\nSoldiers fought in wars or quelled domestic uprisings. During long periods of peace, soldiers also supervised the peasants, farmers, and slaves who were involved in building such structures as pyramids and palaces. \r\nSkilled workers such as physicians and craftsmen/women made up the middle class. Craftsmen made and sold jewelry, pottery, papyrus products, tools, and other useful things. \r\nNaturally, there were people needed to buy goods from artisans and traders. These were the merchants and storekeepers who sold these goods to the public. \r\nThe Bottom of the Heap \r\nAt the bottom of the social structure were slaves and farmers. Slavery became the fate of those captured as prisoners of war. In addition to being forced to work on building projects, slaves toiled at the discretion of the pharaoh or nobles. \r\nFarmers tended the fields, raised animals, kept canals and reservoirs in good order, worked in the stone quarries, and built the royal monuments. Farmers paid taxes that could amount to as much as 60% of their yearly harvest—that’s a lot of hay! \r\nSocial mobility was not impossible. A small number of peasants and farmers moved up the economic ladder. Families saved money to send their sons to village schools to learn trades. These schools were run by priests or by artisans. Boys who learned to read and write could become scribes, then go on to gain employment in the government. It was possible for a boy born on a farm to work his way up into the higher ranks of the government. Bureaucracy proved lucrative.

### Question:
In complete sentences, summarize the structure of the ancient Egyptian system of government. How were different social classes involved in this government? Cite evidence from the text.

### Summary:
The acient system of government for Egypt went like this. Social calsses went from slaves, farmers/workers, traders, preits, and kings. This was involved in government because they each had to follow rules from the pharos and if they didn't they would have a punishment. The punishment could be Karma, or execution. Also they each had to pay the pharos taxes. ( From farmers) Priets were Also responsible for pleasing the gods and Pharos. These are reasons how government and how different social classes in government were involved during it.

### Score of Summary:
"""

model_input = tokenizer(eval_prompt, return_tensors="pt", truncation=True).to("cuda")

model.eval()
with torch.no_grad():
    print(tokenizer.decode(model.generate(**model_input, max_new_tokens=100)[0], skip_special_tokens=True))
```

I was surprised to find that the original base model already exhibited a good performance on this task. The output of the base model is as follows:

```text
### Instruction:
Below is a summary evaluation task for the summary written by students. Title is the title of the text. Text is the text students need to write summary of. Question is the question for the students to follow. Summary is the summary written by students. Score of summary contains the score of wording and score of content. Write the score of wording and score of content according to the summary written by students.

### Title:
Egyptian Social Structure

### Text:
Egyptian society was structured like a pyramid. At the top were the gods, such as Ra, Osiris, and Isis. Egyptians believed that the gods controlled the universe. Therefore, it was important to keep them happy. They could make the Nile overflow, cause famine, or even bring death. 
The Egyptians also elevated some human beings to gods. Their leaders, called pharaohs, were believed to be gods in human form. They had absolute power over their subjects. After pharaohs died, huge stone pyramids were built as their tombs. Pharaohs were buried in chambers within the pyramids. 
Because the people of Egypt believed that their pharaohs were gods, they entrusted their rulers with many responsibilities. Protection was at the top of the list. The pharaoh directed the army in case of a foreign threat or an internal conflict. All laws were enacted at the discretion of the pharaoh. Each farmer paid taxes in the form of grains, which were stored in the pharaoh’s warehouses. This grain was used to feed the people in the event of a famine. 
The Chain of Command 
No single person could manage all these duties without assistance. The pharaoh appointed a chief minister called a vizier as a supervisor. The vizier ensured that taxes were collected. 
Working with the vizier were scribes who kept government records. These high-level employees had mastered a rare skill in ancient Egypt — they could read and write. 
Noble Aims 
Right below the pharaoh in status were powerful nobles and priests. Only nobles could hold government posts; in these positions they profited from tributes paid to the pharaoh. Priests were responsible for pleasing the gods. 
Nobles enjoyed great status and also grew wealthy from donations to the gods. All Egyptians—from pharaohs to farmers—gave gifts to the gods. 
Soldier On 
Soldiers fought in wars or quelled domestic uprisings. During long periods of peace, soldiers also supervised the peasants, farmers, and slaves who were involved in building such structures as pyramids and palaces. 
Skilled workers such as physicians and craftsmen/women made up the middle class. Craftsmen made and sold jewelry, pottery, papyrus products, tools, and other useful things. 
Naturally, there were people needed to buy goods from artisans and traders. These were the merchants and storekeepers who sold these goods to the public. 
The Bottom of the Heap 
At the bottom of the social structure were slaves and farmers. Slavery became the fate of those captured as prisoners of war. In addition to being forced to work on building projects, slaves toiled at the discretion of the pharaoh or nobles. 
Farmers tended the fields, raised animals, kept canals and reservoirs in good order, worked in the stone quarries, and built the royal monuments. Farmers paid taxes that could amount to as much as 60% of their yearly harvest—that’s a lot of hay! 
Social mobility was not impossible. A small number of peasants and farmers moved up the economic ladder. Families saved money to send their sons to village schools to learn trades. These schools were run by priests or by artisans. Boys who learned to read and write could become scribes, then go on to gain employment in the government. It was possible for a boy born on a farm to work his way up into the higher ranks of the government. Bureaucracy proved lucrative.

### Question:
In complete sentences, summarize the structure of the ancient Egyptian system of government. How were different social classes involved in this government? Cite evidence from the text.

### Summary:
The acient system of government for Egypt went like this. Social calsses went from slaves, farmers/workers, traders, preits, and kings. This was involved in government because they each had to follow rules from the pharos and if they didn't they would have a punishment. The punishment could be Karma, or execution. Also they each had to pay the pharos taxes. ( From farmers) Priets were Also responsible for pleasing the gods and Pharos. These are reasons how government and how different social classes in government were involved during it.

### Score of Summary:
Score of wording: 6/10
Score of content: 9/10
```

### Tokenization

```python
def generate_and_tokenize_prompt(prompt):
    return tokenizer(formatting_func(prompt), truncation=True)
```

```python
tokenized_train_dataset = train_dataset.map(generate_and_tokenize_prompt)
tokenized_val_dataset = eval_dataset.map(generate_and_tokenize_prompt)
```

Let's take a look at the tokenized dataset.

```python
tokenized_train_dataset
```

```text
Dataset({
    features: ['prompt_title', 'text', 'prompt_question', 'prompt_text', 'content', 'wording', 'input_ids', 'attention_mask'],
    num_rows: 6448
})
```

```python
import matplotlib.pyplot as plt

def plot_data_lengths(tokenized_train_dataset, tokenized_val_dataset):
    lengths = [len(x['input_ids']) for x in tokenized_train_dataset]
    lengths += [len(x['input_ids']) for x in tokenized_val_dataset]
    print(len(lengths))

    # Plotting the histogram
    plt.figure(figsize=(10, 6))
    plt.hist(lengths, bins=20, alpha=0.7, color='blue')
    plt.xlabel('Length of input_ids')
    plt.ylabel('Frequency')
    plt.title('Distribution of Lengths of input_ids')
    plt.show()

plot_data_lengths(tokenized_train_dataset, tokenized_val_dataset)
```

<img src="pic_1.png" width="60%" height="60%">

```python
max_length = 2048

def generate_and_tokenize_prompt2(prompt):
    result = tokenizer(
        formatting_func(prompt),
        # prompt,
        truncation=True,
        max_length=max_length,
        padding="max_length",
    )
    result["labels"] = result["input_ids"].copy()
    return result
```

```python
tokenized_train_dataset = train_dataset.map(generate_and_tokenize_prompt2)
tokenized_val_dataset = eval_dataset.map(generate_and_tokenize_prompt2)
```

```python
print(tokenized_train_dataset[1]['input_ids'])
```

```python
plot_data_lengths(tokenized_train_dataset, tokenized_val_dataset)
```

<img src="pic_2.png" width="60%" height="60%">

### Set up LoRA

```python
from peft import prepare_model_for_kbit_training

model.gradient_checkpointing_enable()
model = prepare_model_for_kbit_training(model)
```

```python
def print_trainable_parameters(model):
    """
    Prints the number of trainable parameters in the model.
    """
    trainable_params = 0
    all_param = 0
    for _, param in model.named_parameters():
        all_param += param.numel()
        if param.requires_grad:
            trainable_params += param.numel()
    print(
        f"trainable params: {trainable_params} || all params: {all_param} || trainable%: {100 * trainable_params / all_param}"
    )
```

```python
from peft import LoraConfig, get_peft_model

config = LoraConfig(
    r=32,
    lora_alpha=64,
    target_modules=[
        "q_proj",
        "k_proj",
        "v_proj",
        "o_proj",
        "gate_proj",
        "up_proj",
        "down_proj",
        "lm_head",
    ],
    bias="none",
    lora_dropout=0.05,  # Conventional
    task_type="CAUSAL_LM",
)

model = get_peft_model(model, config)
print_trainable_parameters(model)

# Apply the accelerator.
model = accelerator.prepare_model(model)
```

According to the output, we can find that the parameters fine-tuned based on LoRA only account for approximately 1% of the total model parameters.

```text
trainable params: 81108992 || all params: 6819524608 || trainable%: 1.1893643129442022
```

And find out what the model looks like.

```python
model
```

```text
PeftModelForCausalLM(
  (base_model): LoraModel(
    (model): LlamaForCausalLM(
      (model): LlamaModel(
        (embed_tokens): Embedding(32000, 4096)
        (layers): ModuleList(
          (0-31): 32 x LlamaDecoderLayer(
            (self_attn): LlamaAttention(
              (q_proj): Linear8bitLt(
                in_features=4096, out_features=4096, bias=False
                (lora_dropout): ModuleDict(
                  (default): Dropout(p=0.05, inplace=False)
                )
                (lora_A): ModuleDict(
                  (default): Linear(in_features=4096, out_features=32, bias=False)
                )
                (lora_B): ModuleDict(
                  (default): Linear(in_features=32, out_features=4096, bias=False)
                )
                (lora_embedding_A): ParameterDict()
                (lora_embedding_B): ParameterDict()
              )
              (k_proj): Linear8bitLt(
                in_features=4096, out_features=4096, bias=False
                (lora_dropout): ModuleDict(
                  (default): Dropout(p=0.05, inplace=False)
                )
                (lora_A): ModuleDict(
                  (default): Linear(in_features=4096, out_features=32, bias=False)
                )
                (lora_B): ModuleDict(
                  (default): Linear(in_features=32, out_features=4096, bias=False)
                )
                (lora_embedding_A): ParameterDict()
                (lora_embedding_B): ParameterDict()
              )
              (v_proj): Linear8bitLt(
                in_features=4096, out_features=4096, bias=False
                (lora_dropout): ModuleDict(
                  (default): Dropout(p=0.05, inplace=False)
                )
                (lora_A): ModuleDict(
                  (default): Linear(in_features=4096, out_features=32, bias=False)
                )
                (lora_B): ModuleDict(
                  (default): Linear(in_features=32, out_features=4096, bias=False)
                )
                (lora_embedding_A): ParameterDict()
                (lora_embedding_B): ParameterDict()
              )
              (o_proj): Linear8bitLt(
                in_features=4096, out_features=4096, bias=False
                (lora_dropout): ModuleDict(
                  (default): Dropout(p=0.05, inplace=False)
                )
                (lora_A): ModuleDict(
                  (default): Linear(in_features=4096, out_features=32, bias=False)
                )
                (lora_B): ModuleDict(
                  (default): Linear(in_features=32, out_features=4096, bias=False)
                )
                (lora_embedding_A): ParameterDict()
                (lora_embedding_B): ParameterDict()
              )
              (rotary_emb): LlamaRotaryEmbedding()
            )
            (mlp): LlamaMLP(
              (gate_proj): Linear8bitLt(
                in_features=4096, out_features=11008, bias=False
                (lora_dropout): ModuleDict(
                  (default): Dropout(p=0.05, inplace=False)
                )
                (lora_A): ModuleDict(
                  (default): Linear(in_features=4096, out_features=32, bias=False)
                )
                (lora_B): ModuleDict(
                  (default): Linear(in_features=32, out_features=11008, bias=False)
                )
                (lora_embedding_A): ParameterDict()
                (lora_embedding_B): ParameterDict()
              )
              (up_proj): Linear8bitLt(
                in_features=4096, out_features=11008, bias=False
                (lora_dropout): ModuleDict(
                  (default): Dropout(p=0.05, inplace=False)
                )
                (lora_A): ModuleDict(
                  (default): Linear(in_features=4096, out_features=32, bias=False)
                )
                (lora_B): ModuleDict(
                  (default): Linear(in_features=32, out_features=11008, bias=False)
                )
                (lora_embedding_A): ParameterDict()
                (lora_embedding_B): ParameterDict()
              )
              (down_proj): Linear8bitLt(
                in_features=11008, out_features=4096, bias=False
                (lora_dropout): ModuleDict(
                  (default): Dropout(p=0.05, inplace=False)
                )
                (lora_A): ModuleDict(
                  (default): Linear(in_features=11008, out_features=32, bias=False)
                )
                (lora_B): ModuleDict(
                  (default): Linear(in_features=32, out_features=4096, bias=False)
                )
                (lora_embedding_A): ParameterDict()
                (lora_embedding_B): ParameterDict()
              )
              (act_fn): SiLUActivation()
            )
            (input_layernorm): LlamaRMSNorm()
            (post_attention_layernorm): LlamaRMSNorm()
          )
        )
        (norm): LlamaRMSNorm()
      )
      (lm_head): Linear(
        in_features=4096, out_features=32000, bias=False
        (lora_dropout): ModuleDict(
          (default): Dropout(p=0.05, inplace=False)
        )
        (lora_A): ModuleDict(
          (default): Linear(in_features=4096, out_features=32, bias=False)
        )
        (lora_B): ModuleDict(
          (default): Linear(in_features=32, out_features=32000, bias=False)
        )
        (lora_embedding_A): ParameterDict()
        (lora_embedding_B): ParameterDict()
      )
    )
  )
)
```

### Training

```python
if torch.cuda.device_count() > 1: # If more than 1 GPU
    model.is_parallelizable = True
    model.model_parallel = True
```

```python
import transformers
from datetime import datetime

project = "summary-evaluate-finetune"
base_model_name = "llama2-7b"
run_name = base_model_name + "-" + project
output_dir = "./tmp/llama-output/" + run_name

tokenizer.pad_token = tokenizer.eos_token

trainer = transformers.Trainer(
    model=model,
    train_dataset=tokenized_train_dataset,
    eval_dataset=tokenized_val_dataset,
    args=transformers.TrainingArguments(
        output_dir=output_dir,
        warmup_steps=1,
        per_device_train_batch_size=2,
        gradient_accumulation_steps=1,
        max_steps=500,
        learning_rate=2.5e-5, # Want a small lr for finetuning
        bf16=True,
        optim="paged_adamw_8bit",
        logging_dir=f"{output_dir}/logs",
        logging_strategy="steps",
        logging_steps=50,
        save_strategy="no",       # Save the model checkpoint every logging step
        # save_steps=50,                # Save checkpoints every 50 steps
        evaluation_strategy="no",
        # eval_steps=50,               # Evaluate and save checkpoints every 50 steps
        do_eval=True,
        # report_to="wandb",
        run_name=f"{run_name}-{datetime.now().strftime('%Y-%m-%d-%H-%M')}"          # Name of the W&B run
    ),
    data_collator=transformers.DataCollatorForLanguageModeling(tokenizer, mlm=False),
)

model.config.use_cache = False  # silence the warnings
trainer.train()
```

It will take around 50 minutes to finish the training on a A100 instance.

```text
[500/500 48:15, Epoch 0/1]
Step	Training Loss
50	1.355700
100	0.447800
150	0.198000
200	0.190800
250	0.188400
300	0.196800
350	0.181200
400	0.193900
450	0.196000
500	0.189200

TrainOutput(global_step=500, training_loss=0.3302964363098145, metrics={'train_runtime': 2901.9648, 'train_samples_per_second': 0.345, 'train_steps_per_second': 0.172, 'total_flos': 8.2187705647104e+16, 'train_loss': 0.3302964363098145, 'epoch': 0.17})
```

Save the model.

```python
model.save_pretrained(output_dir)
```

If we want to load the model, we can use the following code.

```python
project = "summary-evaluate-finetune"
base_model_name = "llama2-7b"
run_name = base_model_name + "-" + project
output_dir = "./tmp/llama-output/" + run_name

model = LlamaForCausalLM.from_pretrained(output_dir, load_in_8bit=True, device_map='auto', torch_dtype=torch.float16, cache_dir='/data/yuhong_zhao/cache/transformers')
```

Finally, let's take a look at the output of the fine-tuned model.

```python
model.eval()
with torch.no_grad():
    print(tokenizer.decode(model.generate(**model_input, max_new_tokens=100)[0], skip_special_tokens=True))
```

```text
### Instruction:
...

### Summary:
The acient system of government for Egypt went like this. Social calsses went from slaves, farmers/workers, traders, preits, and kings. This was involved in government because they each had to follow rules from the pharos and if they didn't they would have a punishment. The punishment could be Karma, or execution. Also they each had to pay the pharos taxes. ( From farmers) Priets were Also responsible for pleasing the gods and Pharos. These are reasons how government and how different social classes in government were involved during it.

### Score of Summary:
Wording: 0.808321610409875
Content: 0.623783062239291
```

But the scoring results for the same summary is quite different from multiple runs. 

```text
### Instruction:
...

### Summary:
The acient system of government for Egypt went like this. Social calsses went from slaves, farmers/workers, traders, preits, and kings. This was involved in government because they each had to follow rules from the pharos and if they didn't they would have a punishment. The punishment could be Karma, or execution. Also they each had to pay the pharos taxes. ( From farmers) Priets were Also responsible for pleasing the gods and Pharos. These are reasons how government and how different social classes in government were involved during it.

### Score of Summary:
Wording: 1.06577716284845
Content: 1.15545814245537

### Explanation:
Wording: The wording is off because the sentence is not very clear. It is hard to understand what the author is saying. The sentence is also very long.
Content: The content is off because the sentence is not very clear. It is
```

We can find that the fine-tuned model sometimes generates some redundant texts after the scoring results. But the most important problem is that the fine-tuned Llama2 model cannot deal with arithmetic operations for this kind of tasks, more specifically, the scoring results are totally random within a certain range.

