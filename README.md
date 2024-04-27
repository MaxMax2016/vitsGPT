# **[Llama-VITS](https://arxiv.org/abs/2404.06714)**

In our recent [paper](https://arxiv.org/abs/2404.06714), we propose Llama-VITS for enhanced TTS synthesis with semantic awareness extracted from a large-scale language model. This repository is the PyTorch implementation of Llama-VITS. Please visit our [demo](https://xincanfeng.github.io/Llama-VITS_demo/) for audio samples. 

_Note that, to facilitate a clear understanding of our code structure for users, we have not intentionally obfuscated file paths in our code. However, please note that you may need to adjust these paths based on your own main directory._


## Implemented Features:  
**Model with Weights:** 
- [x] Llama-VITS  
- [x] BERT-VITS  
- [x] ORI-VITS  

**Evaluation Metrics:**
- [x] ESMOS  
- [x] UTMOS  
- [x] MCD  
- [x] ASR (CER, WER)  

**Datasets:**  
- [x] full LJSpeech  
- [x] 1-hour LJSpeech  
- [x] EmoV_DB_bea_sem  


## Pre-requisites 
0. Python >= 3.6
0. Clone this repository
0. Install python requirements. Please refer to [requirements.txt](requirements.txt)
    ```sh
    pip install -r requirements.txt
    ```
    1. You may need to install espeak first: 
    ```sh
    apt-get install espeak
    ```
    1. You may also try out below requirements that we extract from our environment if `requirements.txt` does not work for you. 
    ```sh
    pip install -r requirements_all.txt
    ```
0. Download datasets
    1. Download and extract the LJSpeech dataset from [here](https://keithito.com/LJ-Speech-Dataset/), then rename or create a link to the dataset folder: 
    ```sh
    ln -s /path/to/LJSpeech-1.1/wavs vits/DUMMY1
    ```
    1. Download and extract the EmoV_DB_bea_sem dataset from [here](https://drive.google.com/file/d/1-lbFJ2mw4b8ruFK9RNHvfSTQJXWlJVuS/view?usp=sharing), then rename or create a link to the dataset folder: 
    ```sh
    ln -s /path/to/EmoV_DB_bea_sem/wavs_filtered vits/DUMMY5
    ```
    1. You can also download EmoV_DB from [here](https://www.openslr.org/115/) and filter it yourself by refering to our  [preprocess_EmoV_DB_bea_filter.py](datasets/preprocess_EmoV_DB_bea_filter.py). We also provide other code for necessary data preprocessing, e.g., downsampling, in this folder. Note that the `gt_test_wav` folder includes all test audios that we have processed to the same sampling rate with those generated by `*-VITS`. You can also process it by your own if using other dataset.  
    1. We do not provide 1-hour LJSpeech dataset, but you can easily filter it yourself from the full LJSpeech. 
    1. You can check our [filelists](vits/filelists) for exact information about every dataset and semantic token in our experiments.   
0. Build Monotonic Alignment Search and run preprocessing if you use your own datasets.  
    ```sh
    # Cython-version Monotonoic Alignment Search
    cd monotonic_align
    python setup.py build_ext --inplace

    # Preprocessing (g2p) for your own datasets. 
    # python preprocess.py --text_index 1 --filelists filelists/ljs_audio_text_train_filelist.txt filelists/ljs_audio_text_val_filelist.txt filelists/ljs_audio_text_test_filelist.txt 
    ```
    Please refer to [preprocess_own_data.sh](vits/ori_vits/monotonic_align/preprocess_own_data.sh) for configurations on different datasets.  
    Note that we have provided preprocessed phonemes for LJSpeech, 1-hour LJSpeech, and EmoV_DB_bea_sem in [filelists](vits/filelists) named as `datasetname_audio_text_*_filelist.txt.cleaned`. 

## Extracting Semantic Embeddings 
Note that we have provided all extracted semantic embeddings from Llama or various BERT models in [filelists](vits/filelists) named as `datasetname_audio_tokenname_dimension.pt`. 
We provide the code to extract semantic embeddings from Llama or various BERT models as below. 

### Extracting Semantic Embeddings From Llama
0. Use the Llama implementation in our repository which includes codes to extract the semantic embeddings in the final hidden layer. But you can always refer to [Llama](https://github.com/meta-llama/llama/tree/main) repository if there are further related questions. 
0. First, in the llama/ directory run: 
    ```sh
    pip install -e .
    ```
0. Then, download the Llama weights and tokenizer from [Meta website](https://ai.meta.com/resources/models-and-libraries/llama-downloads/) and accept their License.   
0. Once your request is approved, you will receive a signed URL over email. Then run the [download.sh](llama/download.sh) script, passing the URL provided when prompted to start the download. (Pre-requisites: Make sure you have `wget` and `md5sum` installed. Then run the script: `./download.sh`.)  
    - Make sure to grant execution permissions to the download.sh script
    - During this process, you will be prompted to enter the URL from the email. 
    - Do not use the “Copy Link” option but rather make sure to manually copy the link from the email.  
    - Keep in mind that the links expire after 24 hours and a certain amount of downloads. If you start seeing errors such as `403: Forbidden`, you can always re-request a link.   
0. Once the model/s you want have been downloaded, you can run the model locally using the command below:  
    ```bash
    torchrun --nproc_per_node 1 example_chat_completion.py \
        --ckpt_dir llama-2-7b-chat/ \
        --tokenizer_path tokenizer.model \
        --max_seq_len 512 --max_batch_size 6
    ```
    You can refer to [inference.sh](llama/inference.sh) to know more examples about how to run Llama inference. You can use [inference_ave.sh](llama/inference_ave.sh), [inference_last.sh](llama/inference_last.sh), [inference_pca.sh](llama/inference_pca.sh), [inference_mat_phone.sh](llama/inference_mat_phone.sh), [inference_mat_text.sh](llama/inference_mat_text.sh), [inference_sentence.sh](llama/inference_sentence.sh), and [inference_word.sh](llama/inference_word.sh) for scripts to infer and extract specific semantic embeddings. 

    As you can read from the `inference*.sh` script, `example*.py` in the `llama/examples` folder is used to tell Llama how to extract different semantic embeddings, what input transcripts to follow, and where to output. So, remember to check the corresponding `example*.py` file for configurations of the variable `input_file`, `output_file`, and `audiopath` that you want to process. 

### Extracting Semantic Embeddings From various BERT models
You can configure in [get_embedding.sh](berts/get_embedding.sh) to extract BERT embedding. When configuring, don't forget to set correct `filelist_dir` in corresponding `get_embedding_*.py` files. 

## Training
You can train the VITS model w/ or w/o semantic tokens using the scripts below.  
Note that we also provide part of our [pretrained models](a google drive page to appear).

### Training VITS with no semantic tokens  
```sh
python vits/ori_vits/train.py -c vits/configs/ljs_base.json -m ljs_base
```
Please refer to [train.sh](vits/ori_vits/train.sh) for specific configurations of different datasets.
### Training VITS with global semantic tokens   
```sh
python vits/emo_vits/emo_train.py -c vits/configs/ljs_sem_ave.json -m ljs_emo_add_ave
```
Please refer to [emo_train.sh](vits/emo_vits/emo_train.sh) for specific configurations of different datasets and global tokens.
### Training VITS with sequential semantic tokens  
```sh
python vits/sem_vits/sem_train.py -c vits/configs/ljs_sem_mat_text.json -m ljs_sem_mat_text
```
Please refer to [sem_train.sh](vits/sem_vits/sem_train.sh) for specific configurations of different datasets and sequential tokens. ("mat" in the sequential tokens' file name means "matrix", because compared to global token which is mathematically represented by a single vector, sequential token is represented by a matrix for each sentence transcript.)


## Inferencing
See [inference.ipynb](vits/ori_vits/inference.ipynb) as an easy example to understand how to inference on any text.  

Configure the model weights w/ or w/o extracted semantic tokens in the files below for inference according to specific model. Then you can inference on test data transcripts and generate a folder named after the checkpoint, e.g., `G_100000`, including a folder named `source_model_test_wav` which saves all the generated audios in the correspoding checkpoint directory. Specifically,   
Use [infer_test.ipynb](vits/ori_vits/infer_test.ipynb) for inferencing with no semantic tokens on test data transcripts.  
Use [emo_infer_test.ipynb](vits/emo_vits/emo_infer_test.ipynb) for inferencing with global semantic tokens on test data transcripts.  
Use [sem_infer_test.ipynb](vits/sem_vits/sem_infer_test.ipynb) for inferencing with sequential semantic tokens on test data transcripts. 

Note that, in the `source_model_test_wav` file, the saved audio samples are named in the generation order instead of the corresponding transcript key for convenience. 

## Evaluation
### Eval MCD and ASR (CER, WER) using [ESPnet](https://github.com/espnet/espnet), eval UTMOS using [SpeechMOS](https://github.com/tarepan/SpeechMOS)
0. Clone and install [ESPnet](https://github.com/espnet/espnet) according to its repository. 
0. Copy and configure [eval.sh](eval_espnet/eval.sh) into `espnet/egs2/libritts/tts1/eval.sh`. 
0. install whisper for calculating ASR (CER, WER)
    ```sh
    pip install git+https://github.com/openai/whisper.git
    ```
0. Use [run_eval_ljs.sh](vits/run_eval_ljs.sh) and [run_eval_emovdb.sh](vits/run_eval_emovdb.sh), respectively, for evaluation on LJSpeech or EmoV_DB or their subsets. 
    As you can learn from `run_eval_*.sh`, for example, not only [eval.sh](eval_espnet/eval.sh) are used, but also [eval_1_make_kaldi_style_files.py](vits/eval_datasets/eval_ljs/eval_1_make_kaldi_style_files.py) and other process in [eval_datasets](vits/eval_datasets) are used to process and eval on inferenced audio. Specifically,  
    1. Run `eval_1_make_kaldi_style_files.py` to rename the generated audio samples in the `source_model_test_wav` file corresponding to its transcript key. And generate related scp files. 
    ```sh
    python3 /data/vitsGPT/vits/eval_datasets/eval_ljs/eval_1_make_kaldi_style_files.py ${method} ${model} ${step}
    ```

    2. Run `eval_2_unify_and_eval.sh` to downsample both model generated audios and ground truth audios to ensure they have the the same sampling rate. 
    ```sh
    . /data/vitsGPT/vits/eval_datasets/eval_ljs/eval_2_unify_and_eval.sh ${method} ${model} ${step}
    ```

    3. Run `eval.sh` to evaluate MCD，ASR，F0 using the ESPnet framework. (You can also run this step after the step 4.)
    ```sh
    CUDA_VISIBLE_DEVICES=0 . /data/espnet/egs2/libritts/tts1/eval.sh ${method} ${model} ${step} 
    ```
    Because this step may take some time, it is recommended to run this process in the background using:
    ```sh
    CUDA_VISIBLE_DEVICES=0 nohup /data/espnet/egs2/libritts/tts1/eval.sh ${method} ${model} ${step} > eval.log 2>&1 & 
    ```

    4. Run `eval_3_mos.py` to evaluate UTMOS using the SpeechMOS framework. 
    ```sh
    CUDA_VISIBLE_DEVICES=0 python3 /data/vitsGPT/vits/eval_datasets/eval_ljs/eval_3_mos.py ${method} ${model} ${step}
    ```

### Eval ESMOS using Amazon Mechanical Turk (AMT) 
We made paired random examples to receive ESMOS score using AMT. You can refer to [human_evaluation](https://drive.google.com/file/d/1VECEXbkpvMFcv_zDmucymNXiQoRoZTeP/view?usp=sharing) to check out how we prepared for this evaluation. 

## **Citation**
If our work is useful to you, please cite our paper: "**Llama-VITS: Enhancing TTS Synthesis with Semantic Awareness**". [paper](https://arxiv.org/abs/2404.06714)  
```sh
@misc{feng2024llamavits,
      title={Llama-VITS: Enhancing TTS Synthesis with Semantic Awareness}, 
      author={Xincan Feng and Akifumi Yoshimoto},
      year={2024},
      eprint={2404.06714},
      archivePrefix={arXiv},
      primaryClass={cs.CL}
}
```
