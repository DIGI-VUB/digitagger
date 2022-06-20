
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


