# Chat-UniVi v1.5
Following LLaVA v1.5, we add grounding data and visual question-answering (VQA) data into the training dataset, enhancing the model's reasoning capabilities.

## 1. Data
Download the training annotations. You can download from https://huggingface.co/datasets/Chat-UniVi/Chat-UniVi-Instruct/tree/main/v1.5_train_json.

<div align="center">
<table border="1" width="100%">
    <tr align="center">
        <th>Datasets</th><th>Baidu Disk</th>
    </tr>
    <tr align="center">
        <td>Image pretraining (From LLaVA v1.5)</td><td><a href="https://pan.baidu.com/s/17GYcE69FcJjjUM0e4Gad2w?pwd=9ga3">Link</a></td>
    </tr>
    </tr>
    <tr align="center">
        <td>Image tuning (From LLaVA v1.5)</td><td><a href="https://pan.baidu.com/s/1l-jT6t_DlN5DTklwArsqGw?pwd=o6ko">Link</a></td>
    </tr>
    </tr>
    <tr align="center">
        <td>Video pretraining (From Valley)</td><td><a href="https://pan.baidu.com/s/1jluOimE7mmihEBfnpwwCew?pwd=jyjz">Link</a></td>
    </tr>
    </tr>
</table>
</div>

## 2. Train the model
### Stage1: Multimodal Pre-training
```
deepspeed \
--include localhost:0,1,2,3,4,5,6,7 \
--master_port=29602 \
ChatUniVi/train/train_mem.py \
--deepspeed scripts/zero3.json \
--model_name_or_path ${LLM model path} \
--version v1 \
--model_use PRETUNE \
--dataset_use Pretrainv1.5 \
--vision_tower openai/clip-vit-large-patch14-336 \
--tune_mm_mlp_adapter True \
--mm_vision_select_layer -2 \
--mm_use_im_start_end False \
--mm_use_im_patch_token False \
--bf16 True \
--output_dir ${stage1 save path} \
--num_train_epochs 1 \
--per_device_train_batch_size 16 \
--per_device_eval_batch_size 4 \
--gradient_accumulation_steps 1 \
--evaluation_strategy "no" \
--save_strategy "steps" \
--save_steps 24000 \
--save_total_limit 1 \
--learning_rate 2e-3 \
--weight_decay 0. \
--warmup_ratio 0.03 \
--lr_scheduler_type "cosine" \
--logging_steps 1 \
--tf32 True \
--model_max_length 2048 \
--gradient_checkpointing True \
--dataloader_num_workers 4 \
--lazy_preprocess True \
--report_to wandb
```

### Stage2: Joint Instruction Tuning
```
deepspeed \
--include localhost:0,1,2,3,4,5,6,7 \
--master_port=29601 \
ChatUniVi/train/train_mem.py \
--deepspeed scripts/zero2.json \
--model_name_or_path ${LLM model path} \
--version v1 \
--model_use FINETUNE \
--dataset_use FINETUNEv1.5 \
--vision_tower openai/clip-vit-large-patch14-336 \
--pretrain_mm_mlp_adapter ${stage1 save path}/mm_projector.bin \
--mm_vision_select_layer -2 \
--mm_use_im_start_end False \
--mm_use_im_patch_token False \
--bf16 True \
--output_dir ${stage2 save path} \
--num_train_epochs 2 \
--per_device_train_batch_size 16 \
--per_device_eval_batch_size 4 \
--gradient_accumulation_steps 1 \
--evaluation_strategy "no" \
--save_strategy "steps" \
--save_steps 50000 \
--save_total_limit 1 \
--learning_rate 2e-5 \
--weight_decay 0. \
--warmup_ratio 0.03 \
--lr_scheduler_type "cosine" \
--logging_steps 1 \
--tf32 True \
--model_max_length 2048 \
--gradient_checkpointing True \
--dataloader_num_workers 4 \
--lazy_preprocess True \
--report_to wandb
```

## 3. Results
### Image Understanding Benchmarks
<div align="center">
<table border="1" width="100%">
    <tr align="center">
        <th>Methods</th><th>LLM</th><th>Visual Tokens</th><th>VQA v2</th><th>GQA</th><th>VisWiz</th><th>SQA I</th><th>VQA T</th><th>POPE</th><th>MMB</th><th>LLaVA W</th><th>MM-Vet</th>
    </tr>
    <tr align="center">
        <td>LLaVA v1.5</td><td><a href="https://huggingface.co/lmsys/vicuna-7b-v1.5">Vicuna-7B</a></td><td>576</td><td>78.5</td><td>62.0</td><td>50.0</td><td>66.8</td><td>58.2</td><td>85.9</td><td>64.3</td><td>63.4</td><td>30.5</td>
    </tr>
    <tr align="center">
        <td>Video-LLaVA</td><td><a href="https://huggingface.co/lmsys/vicuna-7b-v1.5">Vicuna-7B</a></td><td>256</td><td>74.7</td><td>60.3</td><td>48.1</td><td>66.4</td><td>51.8</td><td>84.4</td><td>60.9</td><td>73.1</td><td>32.0</td>
    </tr>
    <tr align="center">
        <td><a href="">Chat-UniVi-7B v1.5</a></td><td><a href="https://huggingface.co/lmsys/vicuna-7b-v1.5">Vicuna-7B</a></td><td>112</td><td>75.4</td><td>59.6</td><td>44.2</td><td>68.1</td><td>53.0</td><td>85.4</td><td>62.7</td><td>64.3</td><td>28.3</td>
    </tr>
</table>
</div>

### VideoQA
<div align="center">
<table border="1" width="100%">
    <tr align="center">
        <th>Methods</th><th>LLM Size</th><th>MSRVTT-QA</th><th></th><th>MSVD-QA</th><th></th><th>TGIF-QA</th><th></th><th>ActivityNet-QA</th><th></th>
    </tr>
    <tr align="center">
        <th></th><th></th><th>Accuracy</th><th>Score</th><th>Accuracy</th><th>Score</th><th>Accuracy</th><th>Score</th><th>Accuracy</th><th>Score</th>
    </tr>
    <tr align="center">
        <td>Video-LLaMA</td><td>7B</td><td>29.6</td><td>1.8</td><td>51.6</td><td>2.5</td><td>-</td><td>-</td><td>12.4</td><td>1.1</td>
    </tr>
    <tr align="center">
        <td>LLaMA-Adapter</td><td>7B</td><td>43.8</td><td>2.7</td><td>54.9</td><td>3.1</td><td>-</td><td>-</td><td>34.2</td><td>2.7</td>
    </tr>
    <tr align="center">
        <td>VideoChat</td><td>7B</td><td>45.0</td><td>2.5</td><td>56.3</td><td>2.8</td><td>34.4</td><td>2.3</td><td>26.5</td><td>2.2</td>
    </tr>
    <tr align="center">
        <td>Video-ChatGPT</td><td>7B</td><td>49.3</td><td>2.8</td><td>64.9</td><td>3.3</td><td>51.4</td><td>3.0</td><td>35.2</td><td>2.7</td>
    </tr>
    <tr align="center">
        <td>Video-LLaVA</td><td>7B</td><td>59.2</td><td>3.5</td><td>70.7</td><td>3.9</td><td>70.0</td><td>4.0</td><td>45.3</td><td>3.3</td>
    </tr>
    <tr align="center">
        <td><a href="https://huggingface.co/Chat-UniVi/Chat-UniVi">Chat-UniVi-7B</a></td><td>7B</td><td>54.6</td><td>3.1</td><td>65.0</td><td>3.6</td><td>60.3</td><td>3.4</td><td>45.8</td><td>3.2</td>
    </tr>
    <tr align="center">
        <td><a href="https://huggingface.co/Chat-UniVi/Chat-UniVi">Chat-UniVi-7B</a> with new video loading code</td><td>7B</td><td>55.0</td><td>3.1</td><td>69.3</td><td>3.7</td><td>69.0</td><td>3.8</td><td>46.1</td><td>3.3</td>
    </tr>
    <tr align="center">
        <td><a href="">Chat-UniVi-7B v1.5</a></td><td>7B</td><td>57.5</td><td>3.2</td><td>68.8</td><td>3.7</td><td>70.0</td><td>3.8</td><td>47.2</td><td>3.3</td>
    </tr>
</table>
</div>

### POPE
<div align="center">
<table border="1" width="100%">
    <tr align="center">
        <th>Methods</th><th>LLM Size</th><th>Random</th><th></th><th></th><th>Popular</th><th></th><th></th><th>Adversarial</th><th></th><th></th>
    </tr>
    <tr align="center">
        <th></th><th></th><th>Accuracy</th><th>F1-Score</th><th>Yes</th><th>Accuracy</th><th>F1-Score</th><th>Yes</th><th>Accuracy</th><th>F1-Score</th><th>Yes</th>
    </tr>
    <tr align="center">
        <td>LLaVA</td><td>7B</td><td>72.16</td><td>78.22</td><td>76.29</td><td>61.37</td><td>71.52</td><td>85.63</td><td>58.67</td><td>70.12</td><td>88.33</td>
    </tr>
    <tr align="center">
        <td>Video-LLaVA</td><td>7B</td><td>86.2</td><td>85.2</td><td>42.0</td><td>85.3</td><td>84.0</td><td>42.1</td><td>81.6</td><td>80.8</td><td>45.8</td>
    </tr>
    <tr align="center">
        <td><a href="https://huggingface.co/Chat-UniVi/Chat-UniVi">Chat-UniVi-7B</a></td><td>7B</td><td>85.19</td><td>86.05</td><td>54.67</td><td>69.50</td><td>74.39</td><td>69.10</td><td>64.97</td><td>71.54</td><td>73.10</td>
    </tr>
    <tr align="center">
        <td><a href="">Chat-UniVi-7B v1.5</a></td><td>7B</td><td>87.01</td><td>86.09</td><td>41.86</td><td>85.87</td><td>84.76</td><td>42.73</td><td>83.23</td><td>82.31</td><td>44.77</td>
    </tr>
</table>
</div>
