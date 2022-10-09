# scrapping multi-pages with bs4

As mentioned in last [repo web-scrapping with bs4](https://github.com/tzelalouzeir/web-scrapping_bs4). Let's  upgrade it.

## To do
- use the same method
- selecting 1-6 pages
- finding prices, names, links, how many stores have gpu between 1-6 pages and which classes belong in HTML code
## Some issues
- pages suggest gpus and we don't want them
- need to add site link in the end of ```links```
- need clear data (only ENGLISH)
- sometime site changing HTML code for this reason its necessary to fix code 





## Usage

```python
import requests
from requests import get
from bs4 import BeautifulSoup
import pandas as pd
import numpy as np
from urllib.request import Request, urlopen
from bs4 import BeautifulSoup as bs  
import re
import csv

from time import sleep
from random import randint


title_data=[]
price_data=[]
link_data=[]
market_data=[]


# created only for 6 pages if you want to add more please (i think i dont need to create new if  elif conditions)

# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 1-6 Pages~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

# (6 pages = 25*6=150 if each page has 3 GPU suggestion 6*3=18 = 150+18=168 you need add new if elif which len(nums)==168)
# (6 pages = 25*6=150 if each page has 4 GPU suggestion 6*4=24 = 150+24=174 you need add new if elif which len(nums)==174)

# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 1-7 Pages~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

# (7 pages = 25*7=175 if each page has 3 GPU suggestion 7*3=21 = 175+21=196 you need add new if elif which len(nums)==196)
# (7 pages = 25*7=175 if each page has 4 GPU suggestion 7*4=28 = 175+28=203 you need add new if elif which len(nums)==203)




#creating class for clearing if there is any gpu suggestion in pages
class suggestion:
    def solve(self, nums):
        
        # if has 3 GPU suggestion in 6 page (25 gpus first page + 3 suggestion =28, delete between 25-28)
        if len(nums) == 168: 
            del nums[25:28:1]                   #1 page
            del nums[50:53:1]                   #2 page
            del nums[75:78:1]                   #3 page
            del nums[100:103:1]                 #4 page
            del nums[125:128:1]                 #5 page
            del nums[150:153:1]                 #6 page
            
        # if has 4 GPU suggestion in 6 page 25 gpus first page + 4 suggestion =29, delete between 25-29)    
        elif len(nums) == 174: 
            del nums[25:29:1]                   #1 page 
            del nums[50:54:1]                   #2 page
            del nums[75:79:1]                   #3 page
            del nums[100:104:1]                 #4 page
            del nums[125:129:1]                 #5 page
            del nums[150:154:1]                 #6 page
            
obj=suggestion()

# don't forget to change it when you want how much pages do you want to scrape, 1 between 6 pages need to  determine the np.arange(1,7)#
pages = np.arange(1, 7) 
for page in pages:
    
    url = f"https://www.bestprice.gr/cat/2613/kartes-grafikwn.html?o=1&pg={page}"
    req = Request(url , headers={'User-Agent': 'Mozilla/5.0'})

    webpage = urlopen(req).read()
    soup = bs(webpage, "lxml")
    sleep(randint(2,10))

        
    for a in soup.select('div.product__price'):
        market = a.find('small').text
        market_data.append(market)
        m = list(market_data)
        obj.solve(m)
        
    for a in soup.select('div.product__cost-price'):
        link = a.find('a')['href']
        link_data.append(link)
        l = list(link_data) 
        obj.solve(l)
        
        price = a.find('a').text
        price_data.append(price)
        p = list(price_data)
        obj.solve(p)
        
        titles = a.find('a',href=True)
        try:
            title = titles.attrs['title']
        except AttributeError:
            pass
        title_data.append(title)
        t = list(title_data)
        obj.solve(t)
    # adding site text end of links that we scrapped 
    mylist = list(l)
    for idx in range(len(mylist)):
         mylist[idx] = "https://www.bestprice.gr" + mylist[idx]

# Saving results to CSV file 
with open("pages/page1-6.csv", "w", encoding='utf-8',newline='') as csvfile:
    writer = csv.writer(csvfile)
    header = ['Titles','Prices','Links','Market']
    writer.writerow(header)
    for value in range(len(t)):
        writer.writerow([t[value], p[value], mylist[value], m[value]])

# if every len is same value then everything is fine
print("\nTitle Lenght:",len(t))
print("Price Lenght:",len(p))
print("Link Lenght:",len(mylist))
print("Market Lenght:",len(m))
```
## Result
	|Titles |Prices|Links|Market|
    	|-------|------|-----|------|
	|Asus Quadro A100 80GB|18.490,00€|https://www.bestprice.gr/item/2157981385/asus-quadro-a100-80gb.html|Σε 1 κατάστημα|
	|Asus Quadro A100 40GB|12.188,00€|https://www.bestprice.gr/item/2157981387/asus-quadro-a100-40gb.html|Σε 1 κατάστημα|
	|HP Quadro GV100 32GB|11.245,00€|https://www.bestprice.gr/item/2155993929/hp-quadro-gv100-32gb.html|Σε 1 κατάστημα|

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License
[MIT](https://choosealicense.com/licenses/mit/)
