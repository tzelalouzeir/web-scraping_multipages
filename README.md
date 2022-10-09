# scraping multi-pages with bs4

As mentioned in last [repo web-scraping with bs4](https://github.com/tzelalouzeir/web-scraping_bs4). Let's  upgrade it.

## To do
- use the same method
- selecting 1-6 pages
- finding prices, names, links, how many stores have GPU between 1-6 pages and which classes belong in ```HTML``` code
## Some problems, easy to fix
- pages suggest GPUs and we don't want them **(done)**
- need to add site link in the end of ```links``` **(done)**
- need clear data (only *ENGLISH*) **(done)**
- sometime site changing ```HTML``` code for this reason its necessary to fix code 





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


# created only for 6 pages if you want to add more please (i think i don't need to create new if  elif conditions)

# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 1-6 Pages~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

# (6 pages = 25*6=150 if each page has 3 GPU suggestion 6*3=18 = 150+18=168 you need add new if elif which len(nums)==168)
# (6 pages = 25*6=150 if each page has 4 GPU suggestion 6*4=24 = 150+24=174 you need add new if elif which len(nums)==174)

# ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ 1-7 Pages~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

# (7 pages = 25*7=175 if each page has 3 GPU suggestion 7*3=21 = 175+21=196 you need add new if elif which len(nums)==196)
# (7 pages = 25*7=175 if each page has 4 GPU suggestion 7*4=28 = 175+28=203 you need add new if elif which len(nums)==203)




#creating class for clearing if there is any GPU suggestion in pages
class suggestion:
    def solve(self, nums):
        
        # if has 3 GPU suggestion in 6 page (25 GPUs first page + 3 suggestion =28, delete between 25-28)
        if len(nums) == 168: 
            del nums[25:28:1]                   #1 page
            del nums[50:53:1]                   #2 page
            del nums[75:78:1]                   #3 page
            del nums[100:103:1]                 #4 page
            del nums[125:128:1]                 #5 page
            del nums[150:153:1]                 #6 page
            
        # if has 4 GPU suggestion in 6 page 25 GPUs first page + 4 suggestion =29, delete between 25-29)    
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
    header = ['Titles','Prices','Links','in Market']
    writer.writerow(header)
    for value in range(len(t)):
        writer.writerow([t[value], p[value], mylist[value], m[value]])

# if every len is same value then everything is fine
print("\nTitle Lenght:",len(t))
print("Price Lenght:",len(p))
print("Link Lenght:",len(mylist))
print("Market Lenght:",len(m))
```
## Results
|Titles |Prices|Links|in Market|
|-------|------|-----|------|
|Asus Quadro A100 80GB|18.490,00€|https://www.bestprice.gr/item/2157981385/asus-quadro-a100-80gb.html|Σε 1 κατάστημα|
|Asus Quadro A100 40GB|12.188,00€|https://www.bestprice.gr/item/2157981387/asus-quadro-a100-40gb.html|Σε 1 κατάστημα|
|HP Quadro GV100 32GB|11.245,00€|https://www.bestprice.gr/item/2155993929/hp-quadro-gv100-32gb.html|Σε 1 κατάστημα|
|...|...|...|...|

## Upgrade code
We will work little bit different. Collected every data on 1-6 pages with different names(even name was twice in data but with different price), prices and links. Now collect every GPUs variables. For example, there can be different 4 RTX 3060ti and we will add them in our data, it's like a **matryoshka babies**. Scrap the scrap :D 
- add on data new variables such as ```delivery in``` ```shipping cost``` ```store names```
- translating every Greek words

```python
# Finding All gpus in upper code and saving in one csv, 

url_gpu = l

title_gpu=[]
price_gpu=[]
link_gpu=[]
order_gpu=[]
shipping_gpu=[]
m_name_data=[]

for i in url_gpu:
    url_s = f"https://www.bestprice.gr{i}"
    req = Request(url_s , headers={'User-Agent': 'Mozilla/5.0'})
    webpage = urlopen(req).read()
    soup = bs(webpage, "lxml")
    
    
    # specify which column will be scraped in html code
    lists_gpu = soup.find_all('div',attrs={'class':'prices__group'})
    
    # using for loop for getting all information that we want
    for i in lists_gpu:
        t_gpu = i.find('div', attrs={'class':'prices__title'}).text
        p_gpu = i.find('div', attrs={'class':'prices__price'}).text[:8]
        title_gpu.append(t_gpu)
        price_gpu.append(p_gpu)
        
    # delivery in    
    for u in soup.select('div.prices__props'):
        order = u.find('small').text
        order_gpu.append(order)
        
    # shipping cost
    for u in soup.select('div.product__costs'):
        shipping = u.find('div').text
        shipping_gpu.append(shipping)
        
    # market name    
    for u in soup.select('div.prices__merchant'):
        m_name = u.find('em').text
        m_name_data.append(m_name)
        
    # links
    for u in soup.select('div.prices__main'):
        link_g = u.find('a')['href']
        link_gpu.append(link_g)
        
    # deleting extra links, calculating with title_gpu    
    for a in range(len(title_gpu)):
        ll = link_gpu
        del ll[len(title_gpu):]
```
Adding website front of ```mylist_gpu```
```python
    mylist_gpu = ll
    for idx in range(len(ll)):
         mylist_gpu[idx] = "https://www.bestprice.gr" + mylist_gpu[idx]
```
Clearing ```m_name_data``` 
```python
def Convert(string):
    li = list(string.split(" "))
    return li

# collecting market names list to string and string to list
list_m_name_data = list(m_name_data)
str_m_name_data = ' '.join(str(e) for e in list_m_name_data)

#market names
s = str_m_name_data
full_m_name=[]
remove_head = re.sub('/m/.*?/', '', s)
remove_end = re.sub('\.*?html','',remove_head)
full_m_name.append(remove_end)

list_m_name_data1 = list(full_m_name)
str_m_name_data2 = ' '.join(str(e) for e in list_m_name_data1)


# Driver code    
str_m_name_data3 = str_m_name_data2
real_full= Convert(str_m_name_data3)
```
```python
# removing greek characters and put them english words
strings = order_gpu
shipping_str = shipping_gpu
days410='Παράδοση σε 4 - 10 ημέρες'
days814='Παράδοση σε 8 - 14 ημέρες'
days13='Παράδοση σε 1 - 3 ημέρες'
online= 'Άμεσα διαθέσιμο'
uponorder='Κατόπιν παραγγελίας'
shipping_gr='Μεταφορικά'
free_gr = 'Δωρεάν'



delivery_en = []
shipping_en = []

for i in shipping_str:
    ship = (i.replace(shipping_gr," Shipping").replace(free_gr,"Free"))
    shipping_en.append(ship)

for string in strings:
    
    new_string = (string.replace(days410,"Delivery in 4 - 10 days").replace(days814,"Delivery in 8 - 14 days").replace(days13,"Delivery in 1- 3 days").replace(online,"Online").replace(uponorder,"Upon Order"))
    delivery_en.append(new_string)
```
saving our data as .csv
```python
with open("gpu/1-6page_everyGPUs.csv", "w", encoding='utf-8',newline='') as csvfile:
    writer = csv.writer(csvfile)
    header = ['Titles','Prices','Links','Delivery In','Shipping Cost','Market Names']
    writer.writerow(header)
    for value in range(len(title_gpu)):
        writer.writerow([title_gpu[value], price_gpu[value], mylist_gpu[value], delivery_en[value], shipping_en[value], real_full[value]])
```

## Result
|Titles |Prices|Links|Delivery In|Shipping Cost|Market Names|
|-------|------|-----|------|------|------|
|Asus Quadro A100 80GB|18.490,0| [https://www.bestprice.gr/to/104226352/asus-nv...](https://www.bestprice.gr/to/104226352/asus-nvidia-a100-80gb-300w-pciegen4.html?ct=2Ny9An-3VFCHW-82vJQ,fW7CXm9daM7&from=&seq=1&bpref=itemPage) |Delivery in 4 - 10 days|+ 5,00€ Shipping|A-Center|
|Asus Quadro A100 40GB|12.188,0|[https://www.bestprice.gr/to/104226352/asus-nvi...](https://www.bestprice.gr/to/104226356/asus-nvidia-a100-40gb-250w-pciegen4.html?from=&seq=1&bpref=itemPage) |Delivery in 4 - 10 days|+ 5,00€ Shipping|A-Center|
|HP Quadro GV100 32GB|11.245,0|[https://www.bestprice.gr/to/51881672/hp-nvidia-qua](https://www.bestprice.gr/to/51881672/hp-nvidia-quadro-gv100-32gb-pcie-3me26aa.html?from=&seq=1&bpref=itemPage) |Upon Order|Free Shipping|Msystems|
|...|...|...|...|...|...|
|PNY Quadro RTX 8000 Passive|7.718,00|[https://www.bestprice.gr/to/51881672/hp-nvidia-qua](https://www.bestprice.gr/to/66268420/pny-quadro-rtx-8000-passive.html?from=&seq=1&bpref=itemPage) |Delivery in 4 - 10 days|+ 5,00€ Shipping|Homelike|
|...|...|...|...|...|...|
|SAPPHIRE TOXIC RADEON RX 6900 XT 16GB GAMING GDDR6 OC AIR COOLED UEFI|855,07€+|[https://www.bestprice.gr/to/83703606/sapphi](https://www.bestprice.gr/to/83703606/sapphire-toxic-radeon-rx-6900-xt-16gb-gaming-gddr6-oc-air-cooled-uefi.html?from=&seq=3&bpref=itemPage) |Delivery in 4 - 10 days|+ 4,50€ Shipping|JMCtech|
|Gigabyte Radeon RX 6600 XT GAMING OC PRO 8G - graphics card - Radeon RX 6600 XT - 8 GB|820,00€Δ| [https://www.bestprice.gr/to/104145839/giga](https://www.bestprice.gr/to/104145839/gigabyte-radeon-rx-6600-xt-gaming-oc-pro-8g-graphics-card-radeon-rx-6600-xt-8-gb.html?from=&seq=8&bpref=itemPage) |Delivery in 4 - 10 days|Free Shipping|Κωτσόβολος|


#### Issues need to fix
- we can see some different symbols on prices such as *741,76€Δ*,  *871,00€+* or *18.490,0* 
- some stores can be written Greek


## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License
[MIT](https://choosealicense.com/licenses/mit/)
