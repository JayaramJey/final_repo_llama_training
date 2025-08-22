# Multi label classification fine-tuning 

This project fine-tunes a pretrained multi label text classification model to determine emotion in text. The model will be trained to classify whether text expresses any of the following emotions:
- Anger
- Fear
- Joy
- Sadness
- Surprise

# Project Structure
- src   
    - data_prep
        - Loads the dataset from the CSV file
        - Data labels are put into one array
        - Text is tokenized
        - Three different training types available (partial, freeze, full)
        - Weights are determined and applied based on training data balance
    - main.py
        - A custom head is applied to the model (Currently using BERT)
        - The model is trained and evaluated
    - download.py
        - Downloads the required data files to train and test the model
    - custom_head.py
        - Custom classification head that is used on the model for handling multi label classification outputs
    - metrics.py
        - calculates evaluation metrics such as f1 score
- config.yaml
    - A file which allows you to change multiple settings in training such as learning rate, training mode, ect... whithout the need to edit the code.
- requirements.txt
    - All the required packages needed to run the code
# Steps for training and use
### Set up instructions
- Clone the repository
    - git clone https://github.com/JayaramJey/final_repo_llama_training.git
- Set up a virtual environment using the following prompt (or any other environment type you want to use)
    - `conda create -n emotion-classifier`
- cd to the src file
- Active the new environment
    - `conda activate emotion-classifier`
- Install the required packages using the following command:
   - `pip install -r requirements.txt`

- Login to wandb using command line (optional)
    - `wandb login`
- download the required datafiles by using the following command line but only run once
    - `python download.py`
- request access for the llama model from the following link
    - https://huggingface.co/meta-llama/Llama-3.2-3B
- Create a read access token from your hugging face account and use the following command to apply it 
    - `huggingface-cli login`
### Current issues
- f1 scores output all result in 0
- eval_loss and grad_norm output nan

### Training instructions
- Edit config.yaml to choose your training made:         
    - full, partial, frozen
- Run the script using the following command line to perform your selected training
    - python main.py

### Loading your model for testing
- The model is saved within the frozen_bert.pt file

```
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer
from custom_head import CustomLlamaClassifier 
import yaml

with open("../config.yaml", "r") as f:
        config = yaml.safe_load(f)

model_name = config["model"]["name"]

device = torch.device("cuda" if torch.cuda.is_available() else "cpu")

# Recreate the model architecture
base_model = AutoModelForCausalLM.from_pretrained(
        model_name,
        torch_dtype=torch.float16,
        device_map=None,
        trust_remote_code=True
    ).to(device)
model = CustomLlamaClassifier(base_model=base_model, num_labels=5)

# Load the saved weights
model.load_state_dict(torch.load("custom_llama_classifier_weights.pth", map_location=device))

# Load tokenizer
tokenizer = AutoTokenizer.from_pretrained(model_name)
if tokenizer.pad_token is None:
    tokenizer.pad_token = tokenizer.eos_token

model.to(device)
model.eval()
```

# Using a different model
- Most llama-3.2-#B models will work with this code (e.g llama-3.2-1B)
    - just need to change the model name within the config file
- Large models such as openai/gpt-oss-20b will not work due to the size of the model being too large

# How everything works
- llama-3.2-3B is used as the base model
- The custom head is used to adapt the model for multi label classification
- Training is done using BCEWithLogitsLoss
- Weights are assigned to different labels to deal with the imbalance in the training data
- The config.yaml stores all the training parameters including the different training options:
    - partial: Only keep a few layers unfrozen
    - freeze: Keep everything frozen and only train the classifier head
    - full: Train every layer
