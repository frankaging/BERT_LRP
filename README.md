# Auto-Relevance: Automatic Relevance Scores on Tokens for BERT with A Multi-GPU Fine-tune Pipeline
Regular BERT training is for accuracy, but if you are interested in understanding why BERT made that decision, and which tokens contribute more towards that model decision, this repo will help you understand more about the BERT model. If you want to know the decision of your trained BERT, simple import our BERT model, and train with a gradient hook enabled. Your attribution scores will be calculated with a simple ``backward()`` call.

### Install Requirements
You will have to clone this repo, and install all the dependencies. You can skip this step if you have torch and cuda installed. That is all you need. You can also mannually install these without going through this installation headache that ``requirements.txt`` may give you.
```bash
cd BERT_LRP/code/
pip install -r requirements.txt
```

### Download Pretrained BERT Model
You will have to download pretrained BERT model in order to execute the fine-tune pipeline. We recommand to use models provided by the official release on BERT from [BERT-Base (Google's pre-trained models)](https://github.com/google-research/bert). Note that their model is in tensorflow format. To convert tensorflow model to pytorch model, you can use the helper script to do that. For example,
```bash
cd BERT_LRP/code/
python convert_tf_checkpoint_to_pytorch.py \
--tf_checkpoint_path uncased_L-12_H-768_A-12/bert_model.ckpt \
--bert_config_file uncased_L-12_H-768_A-12/bert_config.json \
--pytorch_dump_path uncased_L-12_H-768_A-12/pytorch_model.bin
```

### BERT Model and Pretrain
Our BERT model is adapted from [huggingface](https://github.com/huggingface/transformers) BERT model for text classification. If you want to take a look at the original model please search for [BertForSequenceClassification](https://github.com/huggingface/transformers/blob/master/src/transformers/modeling_bert.py). If you want to fine-tune or train a BERT classifier, you can either use their pipeline line or ours. Ours is tested against theirs before publishing. It is bug-free. To train a model, you can do something like this,
```bash
cd code/
CUDA_VISIBLE_DEVICES=0.1,2 python run_classifier.py \
--model_type BERTPretrain \
--eval_test \
--do_lower_case \
--max_seq_length 512 \
--train_batch_size 8 \
--eval_batch_size 8 \
--learning_rate 2e-5 \
--num_train_epochs 3 \
--seed 123 \
--task_name SST5 \
--data_dir ../datasets/SST5/ \
--vocab_file ../models/BERT-Google/vocab.txt \
--bert_config_file ../models/BERT-Google/bert_config.json \
--output_dir ../results/SST5-NewSeed/ \
--init_checkpoint ../models/BERT-Google/pytorch_model.bin
```
Take a look at ``code/util/processor.py`` to see how we process different datasets. We currently supports almost 10 different dataset loadings. You can create your own within 1 minute for loading data. You can specify your directories info above in the command.

### Analyze, Attribution, Relevance and More
Once you have your model ready, save it to a location that you know (e.g., ``../results/SST5/checkpoint.bin``). Our example code how to get relevance scores is in a jupyter notebook format, which is much easier to read. This is how you will open it,
```bash
cd code/notebook/
jupyter notebook
```
Inside ``lrp_visualize``, we provide an example on how to get relevance scores! In short, it is really easy, when you create your BERT model, just provide an extra argument which enables a variety of gradient hooks, by setting ``init_lrp=True``. And then, in your evaluation loop, you can call ``model.backward_lrp(relevance_score)``.
