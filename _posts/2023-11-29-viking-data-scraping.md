---
layout: post
title:  "Scraping Data on Viking Artifacts"
author: Aly Milne
description: Process for scraping the artifact database for Statens Historiska Museer. 
image: "/assets/images/DALLE/viking_in_archive.png"
---

## Introduction

Although I spend most of my time pursuing my interests in data scince, al also have a guilty pleasure: studying history. In particular, I love the Viking era of Scandinavia. So, I decided to combine my two loves and scrape data about Viking artifacts from the Statens Historiska Museer, a Swedish museum group that has put their archive online for the public. 

My journey begins with the essentials: setting up the Python environment and installing the necessary packages. I'll then demonstrate how to use selenium for web scraping and pandas for data manipulation, enabling us to sift through and organize the database effectively. 

This guide is tailored for anyone interested in the crossroads of history and data science. Whether you are a seasoned programmer or a history buff keen on digital methods, this step-by-step approach will provide you with the tools to uncover the secrets held within the Statens Historiska Museer's database of Viking artifacts. Let's embark on this analytical journey through time!

## Necessary Packages

For this project, I used selenium to access the database and other common packages to store the data in an accessibe way (panas, numpy, json).

```
from selenium import webdriver
from selenium.webdriver.chrome.service import Service as ChromeService
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.common.by import By
import time
import json
import pandas as pd
import numpy as np
```

## Setting Up The Driver

The first step involves setting up the Python environment and initializing a dictionary to store the data:

```python
all_data = {}
```

I use Selenium WebDriver for Chrome for web navigation, thanks to its ability to interact with web pages programmatically:

```python
driver = webdriver.Chrome(service=ChromeService(ChromeDriverManager().install()))
url = "https://samlingar.shm.se/sok?type=object&productionPeriod=Vikingatid&hasImage=1&category=Arkeologisk%20samling&category=Vapen%20och%20rustningar&listType=archaeological&rows=500&offset=0"
driver.get(url)
```

You'll see that I have a specific query included in this is for a specific category of artifacts: 'Vapen' (literally 'arms' but used to refer to weapons and war). I used the same query but for trade objects to make two datasets that encompass the main characteristics of the Viking Age: raiding and trading.

The main questions I want to answer with these datasets are:
1. What kind of lasting materials were used in the Viking Age by Scandinavians?
2. What objects did raiders bring back from other parts of the world?
3. How far might the Vikings have travelled?
4. What weapons did they primarily use when raiding and warring?

## Roadblock

As I started to scrape the website, I kept getting an error telling me the Cookies button was the first clickable element, which prevented me from opening the artifacts' individual pages. To fix this, I checked for a Cookies box and clicked it before moving on with the scraping.

```python
cookie_disclaimer = driver.find_element(By.CSS_SELECTOR, '[aria-label="Godkänn alla kakor"]')
if cookie_disclaimer.is_displayed():
    ActionChains(driver).move_to_element(cookie_disclaimer).click().perform()
```

## Extracting Table Data

The search results are stored in a table if you select the 'Archeological' list type. Grabbing this is a great way to start storing data.

```python
table = driver.find_element(By.TAG_NAME, "table")
df = pd.read_html(table.get_attribute('outerHTML'))[0]
```

## Cleaning the Dataframe

Some columns in the table contain things I don't need for this analysis, such as the museum logo or item image in the 'Bild' column. These columns were dropped. Additionally, I need to grab each item's link for additional details and I want to make sure each item has a unique name. To create unique names, I added an iterated number to the end of the item name.

```python
# Adding item names and links
item_link = [(item.get_attribute('href')) for item in driver.find_elements(By.CLASS_NAME, "archaeological-list__link")]
item_names = [f"{item.text} - {index+1}" for index, item in enumerate(driver.find_elements(By.CLASS_NAME, "archaeological-list__link"))]

df['Unique Name'] = item_names
df['Catalog Link'] = item_link
```

## Gathering Additional Details

Many of the fields in the search return table were largely empty. However, the links opened to pages that had a ton of useful information but each table didn't have exactly the same structure as the others. To handle this, I stored it all in an extra column as json.

```python
df['Extra Details'] = None

for index, link in enumerate(df['Catalog Link']):
    driver.get(link)

    # Scrape the details from the item's page
    item_details = {}
    item_tables = driver.find_elements(By.TAG_NAME, "table")
    for item_table in item_tables:
        item_df = pd.read_html(item_table.get_attribute('outerHTML'))[0]
        for row in item_df.itertuples(index=False):
            item_details[row[0]] = row[1]

    # Store the scraped details as JSON in the DataFrame
    df.at[index, 'Extra Details'] = json.dumps(item_details)
```

## Cleaning the Data

Before wrapping up, I replaced placeholder dashes with NaNs, unpacked the JSON values into separate columns, and dropped columns that were less than 25% populated. This function is one of my favorites because it's a fast way to clean data.

```python
# this function drops columns that do not meet the minimum threshold
# if by="count" then drop columns that don't have at least that many populated fields
    # ie. by_val=100 drops columns that have less than 100 populated rows
# if by="prop" then drop columns that don't have at least by_val% populated rows
    # ie. by_val=0.05 drops columns that aren't at least 5% populated
# if by="field" then drop columns that have more missing values than the columns specified
    # ie. by_val="Last_Device_Array.anv" will keep Last_Device_Array.anv but drop any cols that have more missing values than Last_Device_Array.anv
def drop_columns_with_fewer_nans(df, by="prop", by_val="0.05"):
    if by == "count":
        threshold = float(by_val)
    elif by == "prop": threshold = round(df.shape[0]*float(by_val))
    elif by == "field": threshold = df.shape[0]-df[by_val].isna().sum()
    cols_to_drop = []
    for col in df.columns:
        if df[col].isna().sum() > df.shape[0]-threshold:
            cols_to_drop.append(col)
    df = df.drop(cols_to_drop, axis=1)
    return df
```

## Adding External Details

I have the names of dig sites where these artifacts were found, but I want to be able to map them. To do so, I brought in more data using OpenStreetMap which is a free to use mapping database. It is particularly good for Europe.

Since it is free to use, you must identify yourself using your email address or app name and they ask that you don't spam them with requests. I set a wait time of 0.5 seconds before each new query as to not overload it. Additionally, I tried to cache repeat locations so it didn't have to make a new call every time.

```python
# Cache for storing geocoded locations
geocode_cache = {}

def geocode(location):
    """ Geocode a location using Nominatim API with caching. """
    # Check if the location is already in the cache
    if location in geocode_cache:
        return geocode_cache[location]

    url = 'https://nominatim.openstreetmap.org/search'
    headers = {
        'User-Agent': 'alyanngirl@gmail.com'
    }
    params = {
        'q': location,
        'format': 'json'
    }

    response = requests.get(url, headers=headers, params=params)
    if response.status_code == 200:
        results = response.json()
        if results:
            lat_lon = (results[0]['lat'], results[0]['lon'])
            geocode_cache[location] = lat_lon  # Cache the result
            return lat_lon

    return None, None

def geocode_location(row):
    location = row['Plats']
    lat, lon = geocode(location)
    return pd.Series([lat, lon])

# Apply geocode function to each row and create new columns for latitude and longitude
cleaned_trade_artifacts[['latitude', 'longitude']] = cleaned_trade_artifacts.apply(geocode_location, axis=1)
```

## Saving the Work

Finally, I saved the DataFrame as a CSV file for future use:

```python
cleaned_trade_artifacts.to_csv('trade_w_locations.csv', index=False)
```

Please note that this dataset is in Swedish. To see my process of translating it, please see my Google Translation API post.

## Code Repo

For full code, see this repo:
https://github.com/alpal923/Scraping_Viking_Data
