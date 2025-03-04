<h1 align="center">
  KGARevion: An AI Agent for Knowledge-Intensive Biomedical QA
</h1>

## 👀 Overview of KGARevion
Biomedical knowledge is uniquely complex and structured, requiring distinct reasoning strategies compared to other scientific disciplines like physics or chemistry. Biomedical scientists do not rely on a single approach to reasoning; instead, they use various strategies, including rule-based, prototype-based, and case-based reasoning. This diversity calls for flexible approaches that accommodate multiple reasoning strategies while leveraging in-domain knowledge. We introduce KGARevion, a knowledge graph (KG) based agent designed to address the complexity of knowledge-intensive medical queries. Upon receiving a query, KGARevion generates relevant triplets by using the knowledge base of the LLM. These triplets are then verified against a grounded KG to filter out erroneous information and ensure that only accurate, relevant data contribute to the final answer. Unlike RAG-based models, this multi-step process ensures robustness in reasoning while adapting to different models of medical reasoning. Evaluations on four gold-standard medical QA datasets show that KGARevion improves accuracy by over 5.2%, outperforming 15 models in handling complex medical questions. To test its capabilities, we curated three new medical QA datasets with varying levels of semantic complexity, where KGARevion achieved a 10.4% improvement in accuracy. 

![KGARevion framework](https://github.com/mims-harvard/KGARevion/blob/main/model_architecture.jpg)

## 🚀 Installation

1⃣️ First, clone the Github repository:

```bash
git clone https://github.com/mims-harvard/KGARevion
cd KGARevion
```

2⃣️ Then, set up the environment. To create an environment with all of the required packages, please ensure that conda is installed and then execute the commands:

```bash
conda env create -f KGARevion.yaml
conda activate KGARevion
```

## 💡 Running

After cloning the repository and installing all dependencies. To run KGARevion directly:
- First, please download the finetuned checkpoints (files denoted by *finetuned model*, including *adapter_config.bin*, *adapter_model.bin*, *embeddings.pth*, *lm_to_kg.pth*) which is available at [https://doi.org/10.7910/DVN/T2HTGK], to 'fine_tuned_model' folder.
- Second, please run KGARevion by the following command:
   
```bash
python KGARevion.py --dataset MedDDx-Basic --max_round 2 --is_revise True --llm_name llama3.1
```

* dataset: choose one from ['mmlu', 'medqa', 'pubmedqa', 'bioasq', 'MedDDx', 'MedDDx-Basic', 'MedDDx-Intermediate', 'MedDDx-Expert', 'AfrimedQA-MCQ']
* max_round: denotes the number of revision rounds
* is_revise: denotes if the KGARevion contains Revision action
* llm_name: denotes the backbone LLM of KGARevion

## 🛠️ Fine-tuning LLMs in the Review action in KGARevion

In the Review action, KGARevion is implemented by the LLM which is fune-tuned on the KG completion task. To achieve that, we first get the pre-trained embedding of each entity and relation in PrimeKG, which is the biomedical knowledge graph we used in this work.
```bash
cd fine_tuned_model
cd finetune
cd pretrain_primeKG
python rotate_pretraining.py
```
By using this, to achieve the pretrain embedding of each entity and relationship in KGs.

After that, we use LoRA to finetune LLM to make the LLM could determine the correctness of a triplet (head_entity, tail_entity, rel) by considering the embeddings. To achieve that, we finetune the LLM using 4 H100 gpus, as following:
```bash
cd fine_tuned_model
cd finetune
torchrun --nproc_per_node=4 finetune_verification.py
```
The training data (*PrimeKG-train.json*) for finetuning the LLM is avaliable at [https://doi.org/10.7910/DVN/T2HTGK].
## 🌟 Personalize based on your own QA/KG dataset

### QA Dataset

If you want to benchmark KGARevion on your own QA dataset. You are kindly requested to prepare the json file as shown in our datasets folder. Then please benchmark KGARevion on your own QA dataset by:
```bash
python KGARevion.py --dataset [Your own dataset]
```

### KG Dataset

If you want to implement KGARevion with a new KG. You are kindly requested to finetune the LLM with new KG and then integrate the finetuned LLM as the backbone model of the Review action of KGARevion. To achieve that, please first prepare your KG as following:
```bash
{{"input": [head_entity, tail_entity, rel, label], "embedding_ids:" [head_entity_id, tail_entity_id, rel_id]} ... }
```
such as {"input": ["TSC22D3", "HPCAL4", "protein_protein", "True"], "embedding_ids": [792, 0, 10281]}. The label is in ['True', 'False'].

After preparing the dataset,
- First, please obtain the pretrained embedding by running:
```bash
cd fine_tuned_model
cd finetune
cd pretrain_primeKG
python rotate_pretraining.py
```
- Second, please finetune the LLM with the obtained pretrained embeddings and the new KG:
```bash
cd fine_tuned_model
cd finetune
torchrun --nproc_per_node=4 finetune_verification.py
```

## Citation
```bash
@article{su2025knowledge,
  title={KGARevion: An AI Agent for Knowledge-Intensive Biomedical QA},
  author={Su, Xiaorui and Wang, Yibo and Gao, Shanghua and Liu, Xiaolong and Giunchiglia, Valentina and Clevert, Djork-Arn{\'e} and Zitnik, Marinka},
  journal={International Conference on Learning Representations, ICLR},
  year={2025}
}
```
</details>

