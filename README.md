# Mixlingual Speech-To-Text project

### From the team:   
As Chinese students studying in the states, we found our speaking habits morphed -- English words and phrases easily get slipped into Chinese sentences. We greatly feel the need to have messaging apps that can handle multilingual speech-to-text translation. So in this task, we are going to develop this function -- build a model using deep learning technologies to corretly translate multilingual audio (having Chinese and English in the same sentence) into text.

### Data Source:
- [Mandarin-English Code-Switching in South-East Asia](https://catalog.ldc.upenn.edu/ldc2015s04)   

### Baseline Model Paper:
- [A Chinese-English Mixlingual Database and A Speech Recognition Baseline](https://arxiv.org/pdf/1609.08412v1.pdf)

### Interesting Python Kaldi Wrapper to be examined:
- [Pykaldi](https://github.com/UFAL-DSG/pykaldi/tree/master/pykaldi) 
- [Alex Dialog System Framwork](http://alex.readthedocs.io/en/master/_man_rst/alex.tools.kaldi.README.html)  

### Kaldi recommended recipe to be examined:
- [Librispeech](http://www.openslr.org/11/)    

### Kaldi resources:
- [Daniel Povey Lectures](http://www.danielpovey.com/kaldi-lectures.html)
- [An Introduction to Kaldi Toolkit](http://berlin.csie.ntnu.edu.tw/Courses/Speech%20Recognition/Lectures2013/SP2013F_Lecture14-Introduction%20to%20the%20Kaldi%20toolkit.pdf)
- [Building Speech Recognition Systems with the Kaldi Toolkit](https://engineering.jhu.edu/clsp/wp-content/uploads/sites/75/2016/06/Building-Speech-Recognition-Systems-with-the-Kaldi-Toolkit.pdf)
- [Kaldi Document in CN](https://shiweipku.gitbooks.io/chinese-doc-of-kaldi/content/index.html)
- [Kaldi Data Prep](http://pages.jh.edu/~echodro1/tutorial/kaldi/kaldi-training2.html)

### Data Preperation:
- [Kaldi for Dummies Tutorial](http://kaldi-asr.org/doc/kaldi_for_dummies.html  )  
  
| acoustic data:  | filename: |  pattern: | format: |path: | source:|
| ------------- | ------------- |-|-|--|--|
|  |spk2gender  |\<speakerID>\<gender> | |/data/train /data/test | handmade|
|  | utt2spk    |\<utteranceID>\<speakerID> | | /data/train /data/test| handmade | 
|  | wav.scp    |\<utteranceID>\<full_path_to_audio_file>| .scp: kaldi script file|/data/train /data/test | handmade|
|  | text       |\<utteranceID>\<full_path_to_audio_file> |.ark: kaldi archive file| /data/train /data/test|  exists | 
|language data:  | lexicon.txt |\<word> \<phone 1>\<phone 2> ... | .ark: kaldi archive file |data/local/dict| egs/voxforge|
|  | nonsilence_phones.txt | \<phone>| |data/local/dict | unkown | 
|  |silence_phones.txt   |\<phone> | |data/local/dict |unkown |
|  | optional_silence.txt |\<phone> |  | data/local/dict| unkown | 
|Tools:  | utils | | | / | kaldi/egs/wsj/s5|   
|  |steps  | | | / | kaldi/egs/wsj/s5 |
|  | score.sh | | | /| kaldi/egs/voxforge/s5/local |   

### Language Model:
- [ARPA LM format](http://www1.icsi.berkeley.edu/Speech/docs/HTKBook3.2/node213_mn.html)   

What are our language model:  
3-grams trained from the transcripts of THCHS30 + LDC2015S04    

directory structure taken from /egs/TIMIT/s5: 
```
/data
  /local
    /nist_lm
      /lm_phone_bg.arpa.gz
```  
How to build a language model: 
- [SRILM](http://www.speech.sri.com/projects/srilm/)
- [Kaldi lm_build ](https://github.com/srvk/lm_build)
- [egs/babel/s5/local/train_lms_srilm.sh built using SRILM toolkit](https://github.com/kaldi-asr/kaldi/blob/master/egs/babel/s5/local/train_lms_srilm.sh) 

### MFCC Feature Extraction: 
```
   echo
   echo "===== FEATURES EXTRACTION ====="
   echo
 
   # Making feats.scp files
   mfccdir=mfcc
   # Uncomment and modify arguments in scripts below if you have any problems with data sorting
   # utils/validate_data_dir.sh data/train     # script for checking prepared data - here: for data/train directory
   # utils/fix_data_dir.sh data/train          # tool for data proper sorting if needed - here: for data/train directory
   steps/make_mfcc.sh --nj $nj --cmd "$train_cmd" data/train exp/make_mfcc/train $mfccdir
   steps/make_mfcc.sh --nj $nj --cmd "$train_cmd" data/test exp/make_mfcc/test $mfccdir
  
   # Making cmvn.scp files
   steps/compute_cmvn_stats.sh data/train exp/make_mfcc/train $mfccdir
   steps/compute_cmvn_stats.sh data/test exp/make_mfcc/test $mfccdir
```
MFCC-related documents
- [MFCC extraction in detail (CN)](https://my.oschina.net/jamesju/blog/193343)

### HMM - GMM 
[Reference](http://www.inf.ed.ac.uk/teaching/courses/asr/2012-13/asr03-hmmgmm-4up.pdf)  

![a](https://latex.codecogs.com/gif.latex?a_%7Bij%7D) as the transition probability from state i to state j   
![b](https://latex.codecogs.com/gif.latex?b_j%28X%29) as the emission probability from state j to sequence X  

Forward-backward algorithm fine tunes ![a](https://latex.codecogs.com/gif.latex?a_%7Bij%7D)  

GMM provides![b](https://latex.codecogs.com/gif.latex?b_j%28X%29)  

HMM solves the following three problems:  
1. overall likelihood (Forward algorithm): determine the likelihood of an observation sequence X=(x1, x2, ... xT) being generated by an HMM 
2. training (Forward-backward algorithm EM): given an observation sequence, learn the best ![lambda](https://latex.codecogs.com/gif.latex?%5Clambda%5C%7B%20a_%7Bij%7D%2C%20b_j%28X%29%20%5C%7D)
3. decoding (Viterbi algorithm): given an on observation sequence, determine the most probable hidden state sequence

### CNN and MFSC features

In order to train CNN, we need to extract MFSC features from the acoustic data instead of MFCC features, as Discrete Cosine Transformation (DCT) in MFCC destroys locality. MFSC features also called filter banks. In Kaldi, the scripts are something like the following: 
```
steps/make_fbank.sh --nj 3 \ $trainDir/train_clean_fbank exp/make_fbank/train_clean_fbank feat/fbank/ || exit 1;
steps/compute_cmvn_stats.sh $trainDir/train_clean_fbank exp/make_fbank/train_clean_fbank feat/fbank/ || exit 1;
```
notice that fbanks don't work well with GMM as fbanks features are highly correlated, but GMM modelled with diagonal covariance matrices assumed independence of feature streams. fbanks/MFSC is okay with DNN, best for CNN.   
[why MFSC+GMM produced high WER-see Kaldi discussion](https://sourceforge.net/p/kaldi/discussion/1355348/thread/ddf22517/?limit=25)   
[why DCT destroys locality-see post](http://dsp.stackexchange.com/questions/31917/why-discrete-cosine-transform-may-not-maintain-locality)

