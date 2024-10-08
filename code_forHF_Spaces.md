

```
import streamlit as st
from llama_cpp import Llama
import warnings
warnings.filterwarnings(action='ignore')
import datetime
import random
import string
from time import sleep
import tiktoken
import os
from huggingface_hub import hf_hub_download  

# for counting the tokens in the prompt and in the result
#context_count = len(encoding.encode(yourtext))
encoding = tiktoken.get_encoding("r50k_base") 

nCTX = 2048
sTOPS = ['</s>']
modelname = "Lite-Mistral-150M-v2-Instruct"
# Set the webpage title
st.set_page_config(
    page_title=f"Your LocalGPT ✨ with {modelname}",
    page_icon="🌟",
    layout="wide")

if "hf_model" not in st.session_state:
    st.session_state.hf_model = "Lite-Mistral-150M-v2-Instruct"
    
# Initialize chat history
if "messages" not in st.session_state:
    st.session_state.messages = []

if "repeat" not in st.session_state:
    st.session_state.repeat = 1.35

if "temperature" not in st.session_state:
    st.session_state.temperature = 0.1

if "maxlength" not in st.session_state:
    st.session_state.maxlength = 500

if "speed" not in st.session_state:
    st.session_state.speed = 0.0

if "modelfile" not in st.session_state:
    modelfile = hf_hub_download(
        repo_id=os.environ.get("REPO_ID", "OuteAI/Lite-Mistral-150M-v2-Instruct-GGUF"),
        filename=os.environ.get("MODEL_FILE", "Lite-Mistral-150M-v2-Instruct-Q8_0.gguf"),
    )
    st.session_state.modelfile = modelfile

def writehistory(filename,text):
    with open(filename, 'a', encoding='utf-8') as f:
        f.write(text)
        f.write('\n')
    f.close()

def genRANstring(n):
    """
    n = int number of char to randomize
    """
    N = n
    res = ''.join(random.choices(string.ascii_uppercase +
                                string.digits, k=N))
    return res

@st.cache_resource 
def create_chat():   
# Set HF API token  and HF repo
    from llama_cpp import Llama
    client = Llama(
                model_path=st.session_state.modelfile,
                #n_gpu_layers=0,
                temperature=0.1,
                top_p = 0.5,
                n_ctx=nCTX,
                max_tokens=600,
                repeat_penalty=1.18,
                stop=sTOPS,
                verbose=False,
                )
    print('loading Lite-Mistral-150M-v2-Instruct with LlamaCPP...')
    return client


# create THE SESSIoN STATES
if "logfilename" not in st.session_state:
## Logger file
    logfile = f'{genRANstring(5)}_log.txt'
    st.session_state.logfilename = logfile
    #Write in the history the first 2 sessions
    writehistory(st.session_state.logfilename,f'{str(datetime.datetime.now())}\n\nYour own LocalGPT with 🌀 {modelname}\n---\n🧠🫡: You are a helpful assistant.')    
    writehistory(st.session_state.logfilename,f'🌀: How may I help you today?')


#AVATARS
av_us = 'user.png'  # './man.png'  #"🦖"  #A single emoji, e.g. "🧑‍💻", "🤖", "🦖". Shortcodes are not supported.
av_ass = 'assistant3002.png'   #'./robot.png'

### START STREAMLIT UI
# Create a header element
mytitle = '# 🔳 OuteAI Local GPT'
st.markdown(mytitle, unsafe_allow_html=True)
st.markdown(f'> *🌟 {modelname} with {nCTX} tokens Context window*')
st.markdown('---')

# CREATE THE SIDEBAR
with st.sidebar:
    st.image('logo300.png', use_column_width=True)
    st.session_state.temperature = st.slider('Temperature:', min_value=0.0, max_value=1.0, value=0.1, step=0.02)
    st.session_state.maxlength = st.slider('Length reply:', min_value=150, max_value=2000, 
                                           value=500, step=50)
    st.session_state.repeat = st.slider('Repeat Penalty:', min_value=0.0, max_value=2.0, value=1.35, step=0.01)
    st.markdown(f"**Logfile**: {st.session_state.logfilename}")
    statspeed = st.markdown(f'💫 speed: {st.session_state.speed}  t/s')
    btnClear = st.button("Clear History",type="primary", use_container_width=True)

llm = create_chat()

# Display chat messages from history on app rerun
for message in st.session_state.messages:
    if message["role"] == "user":
        with st.chat_message(message["role"],avatar=av_us):
            st.markdown(message["content"])
    else:
        with st.chat_message(message["role"],avatar=av_ass):
            st.markdown(message["content"])
# Accept user input
if myprompt := st.chat_input("What is an AI model?"):
    # Add user message to chat history
    st.session_state.messages.append({"role": "user", "content": myprompt})
    # Display user message in chat message container
    with st.chat_message("user", avatar=av_us):
        st.markdown(myprompt)
        usertext = f"user: {myprompt}"
        writehistory(st.session_state.logfilename,usertext)
        # Display assistant response in chat message container
    with st.chat_message("assistant",avatar=av_ass):
        message_placeholder = st.empty()
        with st.spinner("Thinking..."):
            start = datetime.datetime.now()
            response = ''
            conv_messages = []
            conv_messages.append(st.session_state.messages[-1])
            full_response = ""
            for chunk in llm.create_chat_completion(
                messages=conv_messages,
                temperature=st.session_state.temperature,
                repeat_penalty= st.session_state.repeat,
                stop=sTOPS,
                max_tokens=st.session_state.maxlength,
                stream=True,):
                try:
                    if chunk["choices"][0]["delta"]["content"]:
                        full_response += chunk["choices"][0]["delta"]["content"]
                        message_placeholder.markdown(full_response + "🔳")
                        delta = datetime.datetime.now() -start       
                        totalseconds = delta.total_seconds()
                        prompttokens = len(encoding.encode(myprompt))
                        assistanttokens = len(encoding.encode(full_response))
                        totaltokens = prompttokens + assistanttokens  
                        st.session_state.speed = totaltokens/totalseconds 
                        statspeed.markdown(f'💫 speed: {st.session_state.speed:.2f}  t/s')                                               
                except:
                    pass                 

            delta = datetime.datetime.now() - start
            totalseconds = delta.total_seconds()
            prompttokens = len(encoding.encode(myprompt))
            assistanttokens = len(encoding.encode(full_response))
            totaltokens = prompttokens + assistanttokens
            st.session_state.speed = totaltokens/totalseconds
            statspeed.markdown(f'💫 speed: {st.session_state.speed:.3f}  t/s') 
            toregister = full_response + f"""
```
🧾 prompt tokens: {prompttokens}
📈 generated tokens: {assistanttokens}
⏳ generation time: {delta}
💫 speed: {st.session_state.speed:.2f}  t/s
```"""    
            message_placeholder.markdown(toregister)
            asstext = f"assistant: {toregister}"
            writehistory(st.session_state.logfilename,asstext)       
        st.session_state.messages.append({"role": "assistant", "content": toregister})
```
