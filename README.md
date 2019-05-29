
### Questions
* how to scrape something with beautiful soup when there are no class ids or ids for the section
* is regex important for web scraping? 

### Objectives
YWBAT
* differentiate between getting a request and parsing the file
* use beautiful soup to parse an html file
* use selenium to parse an html file

### Outline
* questions
* scrape ebay for pricing
* load that information into a list
* try to work with infinite scrolling


```python
import requests # used to get a webpage
import pandas as pd
import numpy as np

from bs4 import BeautifulSoup # parse a webpage
from pprint import pprint

import matplotlib.pyplot as plt
import seaborn as sns
```

### Let's get an ebay url to scrape

**setup url**


```python
search = "mechanical keyboards"
url = "https://www.ebay.com/sch/i.html?_nkw={}".format(search.replace(" ", "+"))
url
```




    'https://www.ebay.com/sch/i.html?_nkw=mechanical+keyboards'



**make request**


```python
page = requests.get(url)
```


```python
content = page.content
```

**make soup object for parsing**


```python
soup = BeautifulSoup(content, 'html.parser')
```

**grab the item__wrappers**


```python
item_wrappers = soup.find_all('div', attrs={"class":"s-item__wrapper clearfix"}) # class fields are special
```

**learning wrappers using the first item__wrapper**


```python
first_item_wrapper = item_wrappers[0]
url = first_item_wrapper.find_all('a')[0].get('href')
text = first_item_wrapper.find_all('h3')[0].text.strip()
description = first_item_wrapper.find_all("div", attrs={"class":"s-item__subtitle"})[0].text
item_status = first_item_wrapper.find_all("div", attrs={"class":"s-item__subtitle"})[1].text
price = float(first_item_wrapper.find_all("span", attrs={"class":"s-item__price"})[0].text.replace("$", ""))
shipping_info = first_item_wrapper.find_all("span", attrs={"class":"s-item__shipping s-item__logisticsCost"})[0].text.lower()
free_return = first_item_wrapper.find_all('span', attrs={"class":"s-item__free-returns s-item__freeReturnsNoFee"})[0].text.lower()
```


```python
first_item_wrapper.find_all('h3')[0].text.strip()
```




    'Mechanical Keyboard RGB Wired Backlit Ergonomic Gaming Keyboard  Blue Switches'




```python
# item description
description = first_item_wrapper.find_all("div", attrs={"class":"s-item__subtitle"})[0].text
description
```




    'Cherry MX RGB blue key switches & 104 Key & Ombar Brand'




```python
# item status
item_status = first_item_wrapper.find_all("div", attrs={"class":"s-item__subtitle"})[1].text
item_status
```




    'Brand New'




```python
# finding price
price = float(first_item_wrapper.find_all("span", attrs={"class":"s-item__price"})[0].text.replace("$", ""))
```




    25.99




```python
# finding shipping info
shipping_info = first_item_wrapper.find_all("span", attrs={"class":"s-item__shipping s-item__logisticsCost"})[0].text.lower()
shipping_info

```




    'free shipping'




```python
# check on the return rate
free_return = first_item_wrapper.find_all('span', attrs={"class":"s-item__free-returns s-item__freeReturnsNoFee"})[0].text.lower()
free_return
```




    'free returns'




```python
# loop through all items and get the information desired

dlist = []

for item_wrapper in item_wrappers:
    # commenting this out, since we are iterating through our item wrappers
    # item_wrapper = item_wrappers[0]
    d = {}
    d["url"] = item_wrapper.find_all('a')[0].get('href')
    d["text"] = item_wrapper.find_all('h3')[0].text.strip()
    d["description"] = item_wrapper.find_all("div", attrs={"class":"s-item__subtitle"})[0].text
    d['low_price'] = None
    d['high_price'] = None
    
    
    try:
        d["item_status"] = item_wrapper.find_all("div", attrs={"class":"s-item__subtitle"})[1].text
    except:
        d["item_status"] = None
        
    price_text = item_wrapper.find_all("span", attrs={"class":"s-item__price"})[0].text.replace("$", "")
    try:
        d["price"] = float(price_text)
    except:
        if 'to' in price_text:
            p1, p2 = price_text.split("to")
            d["low_price"] = float(p1.strip())
            d["high_price"] = float(p2.strip())
            
    d["shipping_info"] = item_wrapper.find_all("span", attrs={"class":"s-item__shipping s-item__logisticsCost"})[0].text.lower()
    
    try:
        d["free_return"] = item_wrapper.find_all('span', attrs={"class":"s-item__free-returns s-item__freeReturnsNoFee"})[0].text.lower()
    except:
        d["free_return"] = None
    
    
#     for k, v in d.items():
#         print("{} : {}".format(k, v))
#     print("-"*50)
#     print("\n")

    dlist.append(d)
```


```python
df = pd.DataFrame(dlist)
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>description</th>
      <th>free_return</th>
      <th>high_price</th>
      <th>item_status</th>
      <th>low_price</th>
      <th>price</th>
      <th>shipping_info</th>
      <th>text</th>
      <th>url</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Cherry MX RGB blue key switches &amp; 104 Key &amp; Om...</td>
      <td>free returns</td>
      <td>NaN</td>
      <td>Brand New</td>
      <td>NaN</td>
      <td>25.99</td>
      <td>free shipping</td>
      <td>Mechanical Keyboard RGB Wired Backlit Ergonomi...</td>
      <td>https://www.ebay.com/itm/Mechanical-Keyboard-R...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Brand New</td>
      <td>free returns</td>
      <td>NaN</td>
      <td>None</td>
      <td>NaN</td>
      <td>25.99</td>
      <td>free shipping</td>
      <td>Ombar K676 RGB 104 Key Mechanical Gaming Keybo...</td>
      <td>https://www.ebay.com/itm/Ombar-K676-RGB-104-Ke...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Pre-Owned</td>
      <td>None</td>
      <td>NaN</td>
      <td>None</td>
      <td>NaN</td>
      <td>49.00</td>
      <td>+$12.99 shipping</td>
      <td>WASD Code 87-Key V2B Backlit Mechanical Keyboa...</td>
      <td>https://www.ebay.com/itm/WASD-Code-87-Key-V2B-...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>US Seller – Fast Shipping – 60 Day Returns – W...</td>
      <td>free returns</td>
      <td>NaN</td>
      <td>Brand New</td>
      <td>NaN</td>
      <td>28.99</td>
      <td>free shipping</td>
      <td>SPONSOREDRosewill RGB Gaming Keyboard, Wired, ...</td>
      <td>https://www.ebay.com/itm/Rosewill-RGB-Gaming-K...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>US Seller – Fast Shipping – 60 Day Returns – W...</td>
      <td>free returns</td>
      <td>NaN</td>
      <td>Brand New</td>
      <td>NaN</td>
      <td>49.99</td>
      <td>free shipping</td>
      <td>SPONSOREDRosewill RGB Mechanical Gaming Keyboa...</td>
      <td>https://www.ebay.com/itm/Rosewill-RGB-Mechanic...</td>
    </tr>
  </tbody>
</table>
</div>



### What did we learn?
* sublime text and postman
* sublime text great for shortcut keys and editing many things at once
* postman great for request calls and seeing headers
* try/except
* find_all, .get(), attrs in find_all, .text, 
* how to begin parsing through data


```python
# dealing with multiple pages
search = "mechanical keyboard".replace(" ", "+")
page_number = 1
url = "https://www.ebay.com/sch/i.html?_nkw={}&_pgn={}"
url
```




    'https://www.ebay.com/sch/i.html?_nkw={}&_pgn={}'




```python
urls = [url.format(search, page) for page in range(1, 11)]
urls
```




    ['https://www.ebay.com/sch/i.html?_nkw=mechanical+keyboard&_pgn=1',
     'https://www.ebay.com/sch/i.html?_nkw=mechanical+keyboard&_pgn=2',
     'https://www.ebay.com/sch/i.html?_nkw=mechanical+keyboard&_pgn=3',
     'https://www.ebay.com/sch/i.html?_nkw=mechanical+keyboard&_pgn=4',
     'https://www.ebay.com/sch/i.html?_nkw=mechanical+keyboard&_pgn=5',
     'https://www.ebay.com/sch/i.html?_nkw=mechanical+keyboard&_pgn=6',
     'https://www.ebay.com/sch/i.html?_nkw=mechanical+keyboard&_pgn=7',
     'https://www.ebay.com/sch/i.html?_nkw=mechanical+keyboard&_pgn=8',
     'https://www.ebay.com/sch/i.html?_nkw=mechanical+keyboard&_pgn=9',
     'https://www.ebay.com/sch/i.html?_nkw=mechanical+keyboard&_pgn=10']




```python
# loop through all items and get the information desired

dlist = []
for url in urls:
    page = requests.get(url)
    content = page.content
    soup = BeautifulSoup(content, 'html.parser')
    item_wrappers = soup.find_all('div', attrs={"class":"s-item__wrapper clearfix"}) # class fields are special
    
    for item_wrapper in item_wrappers:
        # commenting this out, since we are iterating through our item wrappers
        # item_wrapper = item_wrappers[0]
        d = {}
        d["url"] = item_wrapper.find_all('a')[0].get('href')
        d["text"] = item_wrapper.find_all('h3')[0].text.strip()
        d["description"] = item_wrapper.find_all("div", attrs={"class":"s-item__subtitle"})[0].text
        d['low_price'] = None
        d['high_price'] = None


#         try:
#             d["item_status"] = item_wrapper.find_all("div", attrs={"class":"s-item__subtitle"})[1].text
#         except:
#             d["item_status"] = None

#         price_text = item_wrapper.find_all("span", attrs={"class":"s-item__price"})[0].text.replace("$", "")
#         try:
#             d["price"] = float(price_text)
#         except:
#             if 'to' in price_text:
#                 p1, p2 = price_text.split("to")
#                 d["low_price"] = float(p1.strip())
#                 d["high_price"] = float(p2.strip())

#         d["shipping_info"] = item_wrapper.find_all("span", attrs={"class":"s-item__shipping s-item__logisticsCost"})[0].text.lower()

#         try:
#             d["free_return"] = item_wrapper.find_all('span', attrs={"class":"s-item__free-returns s-item__freeReturnsNoFee"})[0].text.lower()
#         except:
#             d["free_return"] = None


    #     for k, v in d.items():
    #         print("{} : {}".format(k, v))
    #     print("-"*50)
    #     print("\n")

        dlist.append(d)
```


```python
df = pd.DataFrame(dlist)
print(df.shape)
df.head()
```

    (600, 5)





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>description</th>
      <th>high_price</th>
      <th>low_price</th>
      <th>text</th>
      <th>url</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Pre-Owned</td>
      <td>None</td>
      <td>None</td>
      <td>WASD Code 87-Key V2B Backlit Mechanical Keyboa...</td>
      <td>https://www.ebay.com/itm/WASD-Code-87-Key-V2B-...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Cherry MX RGB blue key switches &amp; 104 Key &amp; Om...</td>
      <td>None</td>
      <td>None</td>
      <td>Mechanical Keyboard RGB Wired Backlit Ergonomi...</td>
      <td>https://www.ebay.com/itm/Mechanical-Keyboard-R...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Pre-Owned</td>
      <td>None</td>
      <td>None</td>
      <td>New ListingTenkeyless Mechanical Keyboard</td>
      <td>https://www.ebay.com/itm/Tenkeyless-Mechanical...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>US Seller – Fast Shipping – 60 Day Returns – W...</td>
      <td>None</td>
      <td>None</td>
      <td>SPONSOREDRosewill RGB Gaming Keyboard, Wired, ...</td>
      <td>https://www.ebay.com/itm/Rosewill-RGB-Gaming-K...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>US Seller – Fast Shipping – 60 Day Returns – W...</td>
      <td>None</td>
      <td>None</td>
      <td>SPONSOREDRosewill RGB Mechanical Gaming Keyboa...</td>
      <td>https://www.ebay.com/itm/Rosewill-RGB-Mechanic...</td>
    </tr>
  </tbody>
</table>
</div>




```python
soup.find_all('div', attrs={"class":"s-item__wrapper clearfix"})
```




    []


