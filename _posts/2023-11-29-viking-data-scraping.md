---
layout: post
title:  "Scraping Data on Viking Artifacts"
author: Aly Milne
description: Process for scraping the artifact database for Statens Historiska Museer. 
image: "/assets/images/DALLE/viking_in_archive.png"
---

## Introduction

In the digital age, the wealth of historical data available online is vast and varied. As a recent project, I embarked on an exciting task: scraping data from the Statens Historiska Museer's database, focusing on Viking artifacts. 

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

## Roadblock

As I started to scrape the website, I kept getting an error telling me the Cookies button was the first clickable element, which prevented me from opening the artifacts' individual pages. To fix this, I checked for a Cookies box and click it before moving on with the scraping.

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

Some columns in the table contain things I don't need for this analysis, such as the museum logo or item image in the 'Bild' column. Additionally, I need to grab each item's link for additional details and I want to make sure each item has a unique name. To create unique names, I added an iterated number to the end of the item name.

```python
if 'Bild' in df.columns:
    df.drop(columns=['Bild'], inplace=True)

# Extracting museum names
museum_elements = driver.find_elements(By.CSS_SELECTOR, "td i.museum-icon")
museum_names = [elem.get_attribute('title') for elem in museum_elements]

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

## Final Touches

Before wrapping up, I replaced placeholder dashes with NaNs and display the first few rows for a quick check:

```python
df.replace('-', np.NaN, inplace=True)
```

## Saving the Work

Finally, I saved the DataFrame as a CSV file for future use:

```python
df.to_csv('Viking_war_artifacts.csv')
```

## Conclusion

This project demonstrates the power of Python for web scraping, particularly for historical data. By programmatically navigating, extracting, and cleaning data from the Statens Historiska Museer's database, we've compiled a comprehensive dataset of Viking artifacts. This method opens up numerous possibilities for historians, data analysts, and enthusiasts alike to analyze and visualize historical data in new and exciting ways. Happy scraping!