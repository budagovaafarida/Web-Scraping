# Web-Scraping
Web Scraping for turbo.az


import requests
from bs4 import BeautifulSoup
import pandas as pd

import warnings
warnings.filterwarnings("ignore")

session = requests.Session()
session.headers.update({
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/115.0.0.0 Safari/537.36",
    "Accept-Language": "en-US,en;q=0.9",
    "Referer": "https://turbo.az/",
})

url="https://turbo.az/"
response = session.get(url) 
response

MAIN CODE

import requests
from bs4 import BeautifulSoup
import pandas as pd

def get_detail_page_links(url):
    
    session = requests.Session()
    session.headers.update({
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/115.0.0.0 Safari/537.36",
        "Accept-Language": "en-US,en;q=0.9",
        "Referer": "https://turbo.az/",
    })
    
    response = session.get(url)
    soup = BeautifulSoup(response.content, 'html.parser')

   
    item_links = []
    items = soup.find_all('div', class_='products-i') 
    for item in items:
        link_element = item.find('a', href=True) 
        if link_element:
            detail_url = link_element['href']
            item_links.append(f"https://turbo.az{detail_url}") 
    return item_links

def scrape_detail_page(url):
   
    session = requests.Session()
    session.headers.update({
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/115.0.0.0 Safari/537.36",
        "Accept-Language": "en-US,en;q=0.9",
        "Referer": "https://turbo.az/",
    })
    
    response = session.get(url)
    soup = BeautifulSoup(response.content, 'html.parser')

  
    column1 = soup.find('div', class_='product-properties__column')
    properties1 = column1.find_all('span', class_='product-properties__i-value') if column1 else []

  
    column2 = column1.find_next_sibling('div', class_='product-properties__column') if column1 else None
    additional_properties = column2.find_all('span', class_='product-properties__i-value') if column2 else []

  
    price_element = soup.find('div', class_='product-price__i product-price__i--bold')
    price = price_element.text.strip() if price_element else None

  
    phone_element = soup.find('a', class_='product-phones__list-i', href=True)
    phone = phone_element['href'].replace("tel:", "") if phone_element else None
    
    description_element = soup.find('div', class_='product-description__content')
    description = description_element.get_text(strip=True) if description_element else None

    extras_element = soup.find('ul', class_='product-extras tz-d-flex tz-align-center tz-gap-10 tz-wrap-wrap')
    extras = [li.text.strip() for li in extras_element.find_all('li')] if extras_element else []
   
    city = properties1[0].text.strip() if len(properties1) > 0 else None
    make = properties1[1].find('a').text.strip() if len(properties1) > 1 and properties1[1].find('a') else properties1[1].text.strip() if len(properties1) > 1 else None
    model = properties1[2].find('a').text.strip() if len(properties1) > 2 and properties1[2].find('a') else properties1[2].text.strip() if len(properties1) > 2 else None
    year = properties1[3].find('a').text.strip() if len(properties1) > 3 and properties1[3].find('a') else properties1[3].text.strip() if len(properties1) > 3 else None
    category = properties1[4].text.strip() if len(properties1) > 4 else None
    color = properties1[5].text.strip() if len(properties1) > 5 else None
    engine = properties1[6].text.strip() if len(properties1) > 6 else None
    mileage = properties1[7].text.strip() if len(properties1) > 7 else None

  
    transmission = additional_properties[0].text.strip() if len(additional_properties) > 0 else None
    drivetrain = additional_properties[1].text.strip() if len(additional_properties) > 1 else None
    is_new = additional_properties[2].text.strip() if len(additional_properties) > 2 else None
    seat_count = additional_properties[3].text.strip() if len(additional_properties) > 3 else None
    owners_count = additional_properties[4].text.strip() if len(additional_properties) > 4 else None
    condition = additional_properties[5].text.strip() if len(additional_properties) > 5 else None

   
    return {
        'Şəhər': city,
        'Marka': make,
        'Model': model,
        'Buraxılış ili': year,
        'Ban növü': category,
        'Rəng': color,
        'Mühərrik': engine,
        'Yürüş': mileage,
        'Sürətlər qutusu': transmission,
        'Ötürücü': drivetrain,
        'Yeni': is_new,
        'Yerlərin sayı': seat_count,
        'Sahiblər': owners_count,
        'Vəziyyəti': condition,
        'Qiymət': price,
        'Telefon': phone,  
        'Açıqlama': description, 
        'Extra' : extras,
    }

base_url = "https://turbo.az/autos?page={}"
all_links = []

for i in range(1, 5):  
    url = base_url.format(i)
    links = get_detail_page_links(url)
    all_links.extend(links)

all_data = []

for link in all_links:
    data = scrape_detail_page(link)
    all_data.append(data)


df = pd.DataFrame(all_data)
print(df)
