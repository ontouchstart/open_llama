# OpenLLaMA: An Open Reproduction of LLaMA

**TL;DR**: we are releasing our public preview of OpenLLaMA, a permissively licensed open source reproduction of Meta AI’s LLaMA 7B trained on the RedPajama dataset. Our model weights can serve as the drop in replacement of LLaMA 7B in existing implementations. We also provide a smaller 3B variant of LLaMA model.


In this repo, we release a permissively licensed open source reproduction of Meta AI's [LLaMA](https://ai.facebook.com/blog/large-language-model-llama-meta-ai/) large language model. In this release, we're releasing a public preview of the 7B and 3B OpenLLaMA model that has been trained with 700 billion tokens. We provide PyTorch and JAX weights of pre-trained OpenLLaMA models, as well as evaluation results and comparison against the original LLaMA models. Stay tuned for our updates.


- [PyTorch weights for OpenLLaMA 7B 700B tokens preview](https://huggingface.co/openlm-research/open_llama_7b_700bt_preview)
- [PyTorch weights for OpenLLaMA 3B 600B tokens preview](https://huggingface.co/openlm-research/open_llama_3b_600bt_preview)
- [EasyLM JAX weights for OpenLLaMA 7B 700B tokens preview](https://huggingface.co/openlm-research/open_llama_7b_700bt_preview_easylm)
- [EasyLM JAX weights for OpenLLaMA 3B 600B tokens preview](https://huggingface.co/openlm-research/open_llama_3b_600bt_preview_easylm)



## Update 05/22/2023

We are happy to release our 700B token checkpoint for the OpenLLaMA 7B model and 600B token checkpoint for the 3B model. We’ve also updated the evaluation results. We expect the full 1T token training run to finish at the end of this week.


## Update 05/15/2023

After receiving feedback from the community, we discovered that the tokenizer of our previous checkpoint release was configured incorrectly so that new lines are not preserved. To fix this problem, we have retrained our tokenizer and restarted the model training. We’ve also observed lower training loss with this new tokenizer.



## Preview Weights Release and Usage

To encourage the feedback from the community, we release a preview checkpoint of our weights. We release the weights in two formats: an EasyLM format to be use with our [EasyLM framework](https://github.com/young-geng/EasyLM), and a PyTorch format to be used with the [Hugging Face transformers](https://huggingface.co/docs/transformers/index) library.

### Loading the Weights with Hugging Face Transformers
Preview checkpoints can be directly loaded from Hugging Face Hub. See the following example for usage:

```python
import torch
from transformers import LlamaTokenizer, LlamaForCausalLM

model_path = 'openlm-research/open_llama_3b_600bt_preview'
# model_path = 'openlm-research/open_llama_7b_700bt_preview'

tokenizer = LlamaTokenizer.from_pretrained(model_path)
model = LlamaForCausalLM.from_pretrained(
    model_path, torch_dtype=torch.float16, device_map='auto',
)

prompt = 'Q: What is the largest animal?\nA:'
input_ids = tokenizer(prompt, return_tensors="pt").input_ids

generation_output = model.generate(
    input_ids=input_ids, max_new_tokens=32
)
print(tokenizer.decode(generation_output[0]))
```

For more advanced usage, please follow the [transformers LLaMA documentation](https://huggingface.co/docs/transformers/main/model_doc/llama).

**Loading the Weights with EasyLM**

For using the weights in our EasyLM framework, please refer to the [LLaMA documentation of EasyLM](https://github.com/young-geng/EasyLM/blob/main/docs/llama.md). Note that unlike the original LLaMA model, our OpenLLaMA tokenizer and weights are trained completely from scratch so it is no longer needed to obtain the original LLaMA tokenizer and weights. Note that we use BOS (beginning of sentence) token (id=1) during training, so it is best to prepend this token for best performance during few-shot evaluation.

Both our training framework EasyLM and the checkpoint weights are licensed permissively under the Apache 2.0 license.



## Dataset and Training

We train our models on the [RedPajama](https://www.together.xyz/blog/redpajama) dataset released by [Together](https://www.together.xyz/), which is a reproduction of the LLaMA training dataset containing over 1.2 trillion tokens. We follow the exactly same preprocessing steps and training hyperparameters as the original LLaMA paper, including model architecture, context length, training steps, learning rate schedule, and optimizer.  The only difference between our setting and the original one is the dataset used: OpenLLaMA employs the RedPajama dataset rather than the one utilized by the original LLaMA.

We train the models on cloud TPU-v4s using [EasyLM](https://github.com/young-geng/EasyLM), a JAX based training pipeline we developed for training and fine-tuning large language models. We employ a combination of normal data parallelism and [fully sharded data parallelism (also know as ZeRO stage 3)](https://engineering.fb.com/2021/07/15/open-source/fsdp/) to balance the training throughput and memory usage. Overall we reach a throughput of over 2100 tokens / second / TPU-v4 chip for our 7B model. The training loss can be seen in the figure below.

![](media/loss.png)

## Evaluation

We evaluated OpenLLaMA on a wide range of tasks using [lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness).  The LLaMA results are generated by running the original LLaMA model on the same evaluation metrics. We note that our results for the LLaMA model differ slightly from the original LLaMA paper, which we believe is a result of different evaluation protocols. Similar differences have been reported in [this issue of lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness/issues/443). Additionally, we present the results of GPT-J, a 6B parameter model trained on the [Pile](https://pile.eleuther.ai/) dataset by [EleutherAI](https://www.eleuther.ai/).

The original LLaMA model was trained for 1 trillion tokens and GPT-J was trained for 500 billion tokens, whereas OpenLLaMA was trained on 400 billion tokens.  We present the results in the table below. OpenLLaMA exhibits comparable performance to the original LLaMA and GPT-J across a majority of tasks, and outperforms them in some tasks. We expect that the performance of OpenLLaMA, after completing its training on 1 trillion tokens, will be enhanced even further.


| **Task/Metric**        | GPT-J 6B | LLaMA 7B | OpenLLaMA 7B 400BT | OpenLLaMA 7B 700BT | OpenLLaMA 3B 350BT | OpenLLaMA 3B 600BT |
| ---------------------- | -------- | -------- | ------------------ | ------------------ | ------------------ | ------------------ |
| anli_r1/acc            | 0.32     | 0.35     | 0.33               | 0.35               | 0.34               | 0.31               |
| anli_r2/acc            | 0.34     | 0.34     | 0.33               | 0.35               | 0.34               | 0.32               |
| anli_r3/acc            | 0.35     | 0.37     | 0.34               | 0.37               | 0.37               | 0.35               |
| arc_challenge/acc      | 0.34     | 0.39     | 0.34               | 0.36               | 0.31               | 0.33               |
| arc_challenge/acc_norm | 0.37     | 0.41     | 0.34               | 0.38               | 0.33               | 0.37               |
| arc_easy/acc           | 0.67     | 0.68     | 0.68               | 0.72               | 0.65               | 0.68               |
| arc_easy/acc_norm      | 0.62     | 0.52     | 0.64               | 0.66               | 0.59               | 0.63               |
| boolq/acc              | 0.66     | 0.75     | 0.67               | 0.70               | 0.60               | 0.66               |
| cb/acc                 | 0.36     | 0.36     | 0.43               | 0.38               | 0.11               | 0.32               |
| cb/f1                  | 0.26     | 0.24     | 0.22               | 0.37               | 0.10               | 0.19               |
| hellaswag/acc          | 0.50     | 0.56     | 0.49               | 0.52               | 0.45               | 0.48               |
| hellaswag/acc_norm     | 0.66     | 0.73     | 0.67               | 0.70               | 0.61               | 0.65               |
| openbookqa/acc         | 0.29     | 0.29     | 0.28               | 0.30               | 0.26               | 0.25               |
| openbookqa/acc_norm    | 0.38     | 0.41     | 0.39               | 0.42               | 0.37               | 0.37               |
| piqa/acc               | 0.75     | 0.78     | 0.74               | 0.75               | 0.72               | 0.74               |
| piqa/acc_norm          | 0.76     | 0.78     | 0.74               | 0.76               | 0.73               | 0.74               |
| record/em              | 0.88     | 0.91     | 0.88               | 0.89               | 0.86               | 0.87               |
| record/f1              | 0.89     | 0.91     | 0.88               | 0.90               | 0.87               | 0.88               |
| rte/acc                | 0.54     | 0.56     | 0.61               | 0.62               | 0.56               | 0.58               |
| truthfulqa_mc/mc1      | 0.20     | 0.21     | 0.22               | 0.24               | 0.23               | 0.22               |
| truthfulqa_mc/mc2      | 0.36     | 0.34     | 0.36               | 0.37               | 0.35               | 0.32               |
| wic/acc                | 0.50     | 0.50     | 0.5                | 0.49               | 0.50               | 0.50               |
| winogrande/acc         | 0.64     | 0.68     | 0.66               | 0.66               | 0.61               | 0.61               |
| wsc/acc                | 0.37     | 0.35     | 0.4                | 0.45               | 0.39               | 0.38               |
| Average                | 0.50     | 0.52     | 0.51               | 0.53               | 0.47               | 0.49               |




## Future Plans

The current release is only a preview of what the complete OpenLLaMA release will offer. We are currently focused on completing the training process on the entire RedPajama dataset. This can gives us a good apple-to-apple comparison between the original LLaMA and our OpenLLaMA. Please stay tuned for our upcoming releases.


## Contact

We would love to get feedback from the community. If you have any questions, please open an issue or contact us.

OpenLLaMA is developed by:
[Xinyang Geng](https://young-geng.xyz/)* and [Hao Liu](https://www.haoliu.site/)* from Berkeley AI Research.
*Equal Contribution


## Acknowledgment

We thank the [Google TPU Research Cloud](https://sites.research.google/trc/about/) program for providing part of the computation resources. We’d like to specially thank Jonathan Caton from TPU Research Cloud for helping us organizing compute resources, Rafi Witten from the Google Cloud team and James Bradbury from the Google JAX team for helping us optimizing our training throughput. We’d also want to thank Charlie Snell, Gautier Izacard, Eric Wallace, Lianmin Zheng and our user community for the discussions and feedback.


## Reference

If you found OpenLLaMA useful in your research or applications, please cite using the following BibTeX:
```
@software{openlm2023openllama,
  author = {Geng, Xinyang and Liu, Hao},
  title = {OpenLLaMA: An Open Reproduction of LLaMA},
  month = May,
  year = 2023,
  url = {https://github.com/openlm-research/open_llama}
}
```
```
@software{together2023redpajama,
  author = {Together Computer},
  title = {RedPajama-Data: An Open Source Recipe to Reproduce LLaMA training dataset},
  month = April,
  year = 2023,
  url = {https://github.com/togethercomputer/RedPajama-Data}
}
```
```
@article{touvron2023llama,
  title={Llama: Open and efficient foundation language models},
  author={Touvron, Hugo and Lavril, Thibaut and Izacard, Gautier and Martinet, Xavier and Lachaux, Marie-Anne and Lacroix, Timoth{\'e}e and Rozi{\`e}re, Baptiste and Goyal, Naman and Hambro, Eric and Azhar, Faisal and others},
  journal={arXiv preprint arXiv:2302.13971},
  year={2023}
}
```
