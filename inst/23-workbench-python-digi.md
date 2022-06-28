#### Python packages for user digi

```{bash}
sudo su digi
mkdir $HOME/.virtualenvs
cd $HOME/.virtualenvs
python3 -m venv nlp-api
source $HOME/.virtualenvs/nlp-api/bin/activate
cd $HOME/.virtualenvs/nlp-api
pip install wheel
pip install cupy-cuda114
pip install 'spacy[cuda114,transformers,lookups]'
pip install spacy-udpipe
pip install python_crfsuite
pip install scikit-learn==1.0.1
pip install sklearn-crfsuite
pip install gensim
pip install pandas
pip install numpy
pip install pycaprio
pip install dkpro-cassis
pip install translitcodec
pip install python-dotenv
pip install pyarrow
pip install flask
pip install https://github.com/explosion/spacy-models/releases/download/nl_core_news_sm-3.3.0/nl_core_news_sm-3.3.0.tar.gz
pip install https://github.com/explosion/spacy-models/releases/download/nl_core_news_md-3.3.0/nl_core_news_md-3.3.0.tar.gz
#pip install https://download.pytorch.org/whl/cu101/torch-1.8.1%2Bcu101-cp38-cp38-linux_x86_64.whl
pip install torch
pip install spacy-transformers
git clone https://github.com/inception-project/inception-external-recommender.git
cd inception-external-recommender
git checkout 3cbcd69a0cc252f4c128c6353df9536eb5eb8bac
pip uninstall inception-external-recommender
pip install -e .
cd ..
```

- Cupy configuration

```
(nlp-api) digi@digiwerkbank:~/.virtualenvs/nlp-api$ python3 -c 'import cupy; cupy.show_config()'
OS                           : Linux-5.4.0-117-generic-x86_64-with-glibc2.29
Python Version               : 3.8.10
CuPy Version                 : 10.5.0
CuPy Platform                : NVIDIA CUDA
NumPy Version                : 1.22.4
SciPy Version                : None
Cython Build Version         : 0.29.24
Cython Runtime Version       : None
CUDA Root                    : /usr/local/cuda
nvcc PATH                    : /usr/local/cuda/bin/nvcc
CUDA Build Version           : 11040
CUDA Driver Version          : 11040
CUDA Runtime Version         : 11040
cuBLAS Version               : (available)
cuFFT Version                : 10502
cuRAND Version               : 10205
cuSOLVER Version             : (11, 2, 0)
cuSPARSE Version             : (available)
NVRTC Version                : (11, 4)
Thrust Version               : 101201
CUB Build Version            : 101201
Jitify Build Version         : 4a37de0
cuDNN Build Version          : 8400
cuDNN Version                : 8401
NCCL Build Version           : 21104
NCCL Runtime Version         : 21212
cuTENSOR Version             : 10500
cuSPARSELt Build Version     : None
Device 0 Name                : GRID T4-4C
Device 0 Compute Capability  : 75
Device 0 PCI Bus ID          : 0000:00:07.0
```

- Launch Flask

```
sudo su digi
cd $HOME/getuigenissen-ne
source $HOME/.virtualenvs/nlp-api/bin/activate
nohup python3 $HOME/getuigenissen-ne/src/api.py &
nohup python3 $HOME/getuigenissen-ne/src/api.py  >> $HOME/getuigenissen-ne/src/api.log 2>&1 &
```


- Install spacy alongside cupy

```
sudo su digi
cd $HOME
pip install -U pip setuptools wheel
#pip install cupy-cuda114
#python3 -c 'import cupy; cupy.show_config()'
pip install -U 'spacy[cuda114,transformers,lookups]'
python3 -m spacy download en_core_web_sm
python3 -m spacy download nl_core_news_sm
python3 -m spacy download en_core_web_trf
```

- Test spacy using gpu `python3`

``` 
import spacy
spacy.prefer_gpu()
spacy.require_gpu()
nlp = spacy.load("en_core_web_sm")
nlp = spacy.load("en_core_web_trf")
out = nlp("John is the boss of the Natural Language Processing group in Brussels.")
out.to_json()
```


