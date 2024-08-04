# OuteWorlderAI
How to run LiteMistral150M on your PC

It works with Python 3.11+

<img src='https://github.com/fabiomatricardi/OuteWorlderAI/raw/main/LiteMistral150m.gif' width=800>


#### Create a new directory and a virtual environment
```
mkdir OuteAI_LiteMistral150m
cd OuteAI_LiteMistral150m 

python -m venv venv

# activate the venv 
.\venv\Scripts\activate

# install packages
pip install streamlit==1.36.0 llama-cpp-python==0.2.85 tiktoken

# Download the models from HuggingFace into the model subfolder
mkdir model
wget https://huggingface.co/OuteAI/Lite-Mistral-150M-v2-Instruct-GGUF/resolve/main/Lite-Mistral-150M-v2-Instruct-Q8_0.gguf -OutFile model/Lite-Mistral-150M-v2-Instruct-Q8_0.gguf
```

#### additional files
- `stapp.py` contains the Streamlit interface and llama_cpp bindings.
- the `images` subfolder contains the logos and icons for the app interface

### Run the app
when you have all of the above, from the terminal with the `venv` activated run:
```
streamlilt run stapp.py
```
