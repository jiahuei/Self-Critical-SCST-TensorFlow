# Self-Critical (SCST) for Image Captioning in TensorFlow
Unofficial implementation of Self-Critical Sequence Training (SCST) and
various multi-head attention mechanisms.

## Description
This repo contains **experimental and unofficial** implementation of
image captioning frameworks including:

- Self-Critical Sequence Training (SCST) [[arxiv]](https://arxiv.org/abs/1612.00563)
    - Sampling is done via beam search [[arxiv]](https://arxiv.org/abs/1707.07998)
- Multi-Head Visual Attention
    - Additive (with optional Layer Norm) [[arxiv]](https://arxiv.org/abs/1903.01072)
    - Scaled Dot-Product [[arxiv]](https://arxiv.org/abs/1706.03762)
- Graph-based Beam Search, Greedy Search and Sampling

The features might not be completely tested. For a more stable implementation,
please refer to [this repo](https://github.com/jiahuei/COMIC-Compact-Image-Captioning-with-Attention).


## Dependencies
- tensorflow 1.9.0
- python 2.7
- java 1.8.0
- tqdm >= 4.24.0
- Pillow >= 3.1.2
- packaging >= 19.0
- requests >= 2.18.4


## Running the code
**More examples are given in `example.sh`.**

### First setup
Run `./src/setup.sh`. This will download the required Stanford models 
and run all the dataset pre-processing scripts.

### Training models
The training scheme is as follows:
1. Start with `decoder` mode (freezing the CNN)
1. Followed by `cnn_finetune` mode
1. Finally, `scst` mode

```bash
# MS-COCO
for mode in 'decoder' 'cnn_finetune' 'scst'
do
    python train.py  \
        --train_mode ${mode}  \
        --token_type 'word'  \
        --cnn_fm_projection 'tied'  \
        --attn_num_heads 8
done

# InstaPIC
for mode in 'decoder' 'cnn_finetune' 'scst'
do
    python train.py  \
        --train_mode ${mode}  \
        --dataset_file_pattern 'insta_{}_v25595_s15'  \
        --token_type 'word'  \
        --cnn_fm_projection 'independent'  \
        --attn_num_heads 8
done
```

### Inferencing
Just point `infer.py` to the directory containing the checkpoints. 
Model configurations are loaded from `config.pkl`.

```bash
# MS-COCO
python infer.py  \
	--infer_checkpoints_dir 'mscoco/word_add_softmax_h8_tie_lstm_run_01'

# InstaPIC
python infer.py  \
	--infer_checkpoints_dir 'insta/word_add_softmax_h8_ind_lstm_run_01'  \
	--annotations_file 'insta_testval_raw.json'
```

### List of major arguments / flags
#### train.py
- Main:
    - `train_mode`: The training regime. Choices are `decoder`, `cnn_finetune`, `scst`.
    - `token_type`: Language model. Choices are `word`, `radix`, `char`.
- CNN:
    - `cnn_name`: CNN model name.
    - `cnn_input_size`: CNN input size.
    - `cnn_fm_attention`: End point name of feature map for attention.
    - `cnn_fm_projection`: Feature map projection method. Choices are `none`, `independent`, `tied`.
- RNN:
    - `rnn_name`: Type of RNN. Choices are `LSTM`, `LN_LSTM`, `GRU`.
    - `rnn_size`: Number of RNN units.
    - `rnn_word_size`: Size of word embedding.
    - `rnn_init_method`: RNN init method. Choices are `project_hidden`, `first_input`.
    - `rnn_recurr_dropout`: If `True`, enable variational recurrent dropout.
- Attention:
    - `attn_num_heads`: Number of attention heads.
    - `attn_context_layer`: If `True`, add linear projection after multi-head attention.
    - `attn_alignment_method`: Alignment / composition method. Choices are `add_LN`, `add`, `dot`.
    - `attn_probability_fn`: Attention map probability function. Choices are `softmax`, `sigmoid`.
- SCST:
    - `scst_beam_size`: The beam size for SCST sampling.
    - `scst_weight_ciderD`: The weight for CIDEr-D metric during SCST training.
    - `scst_weight_bleu`: The weight for BLEU metrics during SCST training.

#### infer.py
- Main:
    - `infer_set`: The split to perform inference on. Choices are `test`, `valid`, `coco_test`, `coco_valid`.
    `coco_test` and `coco_valid` are for inferencing on the whole 
    `test2014` and `val2014` sets respectively. 
    These are used for MS-COCO online server evaluation.
    - `infer_checkpoints_dir`: Directory containing the checkpoint files.
    - `infer_checkpoints`: Checkpoint numbers to be evaluated. Comma-separated.
    - `annotations_file`: Annotations / reference file for calculating scores.
- Inference parameters:
    - `infer_beam_size`: Beam size of beam search. Pass `1` for greedy search.
    - `infer_length_penalty_weight`: Length penalty weight used in beam search.
    - `infer_max_length`: Maximum caption length allowed during inference.
    - `batch_size_infer`: Inference batch size for parallelism.


## Avoid re-downloading datasets
Re-downloading can be avoided by:
1. Editing `setup.sh`
2. Providing the path to the directory containing the dataset files

```bash
python coco_prepro.py --dataset_dir /path/to/coco/dataset
python insta_prepro.py --dataset_dir /path/to/insta/dataset
```

In the same way, both `train.py` and `infer.py` accept alternative dataset paths.

```bash
python train.py --dataset_dir /path/to/dataset
python infer.py --dataset_dir /path/to/dataset
```

This code assumes the following dataset directory structures:

### MS-COCO
```
{coco-folder}
+-- captions
|   +-- {folder and files generated by coco_prepro.py}
+-- test2014
|   +-- {image files}
+-- train2014
|   +-- {image files}
+-- val2014
    +-- {image files}
```

### InstaPIC-1.1M
```
{insta-folder}
+-- captions
|   +-- {folder and files generated by insta_prepro.py}
+-- images
|   +-- {image files}
+-- json
    +-- insta-caption-test1.json
    +-- insta-caption-train.json
```


## Project structure
```
.
+-- common
|   +-- {shared libraries and utility functions}
+-- datasets
|   +-- preprocessing
|       +-- {dataset pre-processing scripts}
+-- pretrained
|   +-- {pre-trained checkpoints for some COMIC models. Details are provided in a separate README.}
+-- src
    +-- {main scripts}
```


## Acknowledgements
Thanks to the developers of:
- [[attend2u]](https://github.com/cesc-park/attend2u/tree/c1185e550c72f71daa74a6ac95791cbf33363b27)
- [[coco-caption]](https://github.com/tylin/coco-caption/tree/3a9afb2682141a03e1cdc02b0df6770d2c884f6f)
- [[ruotianluo/self-critical.pytorch]](https://github.com/ruotianluo/self-critical.pytorch/tree/77dff3223ba2fefe26047ff6ef560c2aa0e1f942)
- [[ruotianluo/cider]](https://github.com/ruotianluo/cider/tree/dbb3960165d86202ed3c417b412a000fc8e717f3)
- [[weili-ict/SelfCriticalSequenceTraining-tensorflow]](https://github.com/weili-ict/SelfCriticalSequenceTraining-tensorflow/tree/cddf3f99bbd5b7ed96c12c6415fb6ae641d4816f)
- [[tensorflow]](https://github.com/tensorflow)


## License and Copyright
The project is open source under Apache-2.0 license (see the `LICENSE` file).


