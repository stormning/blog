---
title: Python Selenium
date: 2021-01-08 15:50
tags: [python,自动化测试]
categories: [后端]
---

##test
###
```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as ec
import json
import requests

options = webdriver.ChromeOptions()
# options.add_argument("headless")
browser = webdriver.Chrome("./chromedriver", options=options)
browser.get('https://www.baidu.com')
img_element = WebDriverWait(browser, 10).until(ec.presence_of_element_located((By.XPATH, 'xpath')))
b64 = img_element.screenshot_as_base64
# deal with base64 ...
# do click
img_element.click()
```