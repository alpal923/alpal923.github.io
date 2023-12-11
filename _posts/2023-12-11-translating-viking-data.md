---
layout: post
title:  "Unveiling the Magic of Automated Translation with Python: A Jupyter Notebook Guide Introduction"
author: Aly Milne
description: Employ Google Cloud Translation
image: "/assets/images/DALLE/translating_viking.png"
---

## Introduction

If you've read my post about scraping data from a Swedish museum's database, you'll remember that I was left with two datasets in a langauge I can't read. This post will detail the process by which I transformed the data into something I could interpret better.

## Getting Started
### Setting the Stage with Essential Libraries
The journey begins with importing necessary Python libraries. The code sets up the environment by importing modules like os, pandas, google.cloud.translate_v2 (Google's translation API), and urllib. These libraries lay the groundwork for interacting with Google's translation services and handling data.

python
Copy code
import os
import pandas as pd
from google.cloud import translate_v2 as translate
from google.oauth2 import service_account
import json
from googleapiclient.discovery import build
import urllib.parse

### Configuring API Access
The next step involves configuring access to Google's translation API. This is done by reading a configuration file containing the project ID and API key. This step is crucial as it authenticates our access to Google's cloud services, ensuring secure and authorized interactions.

Google Cloud APIs do require payment. However, I was able to sign up for a trial for this small project.

```python
# Define the path to the file containing the project ID and API key
config_file = 'path_to_config_file'

# Initialize variables
project_id = None
api_key = None

# Read the project ID and API key from the file
with open(config_file, 'r') as file:
    lines = file.readlines()

    # Extract project ID and API key
    project_id = lines[0].strip()
    api_key = lines[1].strip()
```

### Initializing the Translation Client
With the API access configured, the next code snippet initializes the translation client. This client acts as the bridge between our Python code and Google's translation services.

```python
# Initialize the translation client
translate_client = translate.Client()
```

## Translation
With the translator ready to go, I could begin the translation process. To reduce the number of calls I had to made and to speed up the process, I batched the text and cached the translations as I went.

```python
# Cache for storing translations
translation_cache = {}
```

### The Translation Function
The core of this notebook is a function for batch translation. This function, batch_translate_text_with_model, accepts target language, a list of texts, and an optional model parameter (defaulting to "nmt" for Neural Machine Translation). It translates each text and stores the result in a cache to avoid redundant translations, showcasing efficient and intelligent use of resources.


```python
def batch_translate_text_with_model(target: str, texts: list, model: str = "nmt") -> list:
    translated_texts = []
    for text in texts:
        if text in translation_cache:
            # Use cached translation if available
            translated_texts.append(translation_cache[text])
        else:
            if isinstance(text, bytes):
                text = text.decode("utf-8")

            result = translate_client.translate(text, target_language=target, model=model)
            translated_text = result["translatedText"]
            translation_cache[text] = translated_text
            translated_texts.append(translated_text)
    return translated_texts

# Apply translation
for column in text_columns:
    # Process the column in batches
    for i in range(0, len(df), batch_size):
        batch_slice = slice(i, i + batch_size)
        batch_texts = df[column][batch_slice].dropna()
        translated_batch = batch_translate_text_with_model('en', batch_texts.tolist())

        # Update only the rows that were translated
        df.loc[batch_texts.index, column + '_translated'] = translated_batch
```

## Final Touch
I noticed my dataset had a field called 'Datering' (the date range the artifact is expected to come from) which wasn't in a very useful format. I decided to split the column into two with just the years for Start and End.

```python
# Splitting the 'Datering' column into 'Era Start Year' and 'Era End Year'
df[['Era Start Year', 'Era End Year']] = df['Datering'].str.split(' – ', expand=True)
```

## Conclusion
This process allowed me to create a dataset more useful for the EDA and dashboard creation. While it wasn't perfect (some fields stayed Swedish) this is a much more usable set than before!

## Code Repo
To see the full code for this process, visit this repo:
https://github.com/alpal923/Scraping_Viking_Data