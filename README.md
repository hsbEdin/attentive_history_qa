# History Attention for Conversational Question Answering

This is a conversational question answering model based on the HAM model.
We only train the loss of start-end prediction, instead of using multi learning loss.
Citation:  
```
Chen Qu, Liu Yang, Minghui Qiu, Yongfeng Zhang, Cen Chen, W. Bruce Croft and Mohit Iyyer.  
Attentive History Selection for Conversational Question Answering.  
In Proceedings of the 28th ACM International Conference on Information and Knowledge Management (CIKM 2019), Beijing, China, November 03-07, 2019.

Bibtext
@inproceedings{ham,
	author = {Qu, C. and Yang, L. and Qiu, M. and Zhang, Y. and Chen, C. and Croft, W. B. and Iyyer, M.},
	title = {Attentive History Selection for Conversational Question Answering},
	booktitle = {CIKM '19},
	year = {2019},
}

HAM:
https://github.com/prdwb/attentive_history_selection
```

### Run

1. Download the `BERT-Base/BERT-Large Uncased` model [here](https://github.com/google-research/bert).
2. Download the [QuAC](http://quac.ai/) data.
3. Configurate the directories for the BERT model, data, output, and cache in `cqa_flags.py`. 
4. Run

```
python cqa_run_his_atten.py \
	--output_dir=output_dir/ \
	--max_considered_history_turns=4 \
	--num_train_epochs=20.0 \
	--train_steps=58000 \
	--learning_rate=3e-5 \
	--n_best_size=20 \
	--better_hae=True \
        --use_new_attention=True \
	--MTL=False \
	--train_batch_size=12 \
	--predict_batch_size=12 \
	--evaluate_after=50000 \
	--evaluation_steps=1000 \
	--fine_grained_attention=True \
	--bert_hidden=768 \
	--max_answer_length=50 \
	--load_small_portion=False \
	--cache_dir=cache_large/  \
	--bert_config_file=/bert_dir/uncased_L-24_H-1024_A-16/bert_config.json \
	--init_checkpoint=/bert_dir/uncased_L-24_H-1024_A-16/bert_model.ckpt \
	--vocab_file=/bert_dir/uncased_L-24_H-1024_A-16/vocab.txt \
	--mtl_input=reduce_mean \
	--quac_train_file=/quac_dir/train_v0.2.json  \
	--quac_predict_file=/quac_dir/val_v0.2.json \
	--warmup_proportion=0.1 
```
Setting the max_seq_length to 512 should give better results, but it's more demanding in CUDA memory.

5. During training, you can monitor it via tensorboard, the log directory is the summaries under the output directory.
6. After training, the best result is stored in the results.txt under the output directory. Also look at step_results.txt under the same directory to see at what step we get the best result.

### Some program arguments

Program arguments can be set in `cqa_flgas.py`. Alternatively, they could be specified at running by command line arguments like above. Most of the arguments are self-explanatory. Here are some selected arguments:

* `num_train_epochs` , `train_steps`,`learning_rate` ,`warmup_proportion`: the learning rate follow a schedule of warming up to the specified larning rate and then decaying. This schedule is described in the transformer paper. Our model trains for `train_steps` instead of full `num_train_epochs` epochs. 
* `load_small_portion` . Set to `True` for loading a small portion of the data for testing purpose when we are developing the model. Set to `False` to load all the data when running the model.
* `cache_dir`. When we run the model for the first time, it preprocesses the data and saves it in a cache directory. After that, the model reads the propocessed data from the cache.
* `max_considered_history_turns` and `max_history_turns`. We only consider `max_considered_history_turns` previous turns when preprocessing the data. This is typically set to 11, meaning that all previous turns are under consideration (for QuAC). The `max_history_turns` is for padding purpose in the history attention module.
* `use_new_attention` is the module for word level attention between passage and history dialogs.

For other arguments, please kindly refer to `cqa_flgas.py` for help messages.

This [link](https://worksheets.codalab.org/worksheets/0xb92c7222574046eea830c26cd414faec) is to our official evaluation worksheet on codalab, which contains our best model.


### Scripts

* `cqa_run_his_atten.py`. Entry code.
* `cqa_supports.py`. Utility functions.
* `cqa_gen_batches.py`. Generate batches.
* `cqa_model.py`. Our models.
* `scorer.py`. Official evaluation script for QuAC.

Most other files are for BERT.


### Environment

Tested with Python 3.6.7 and TensorFlow 1.5.0
