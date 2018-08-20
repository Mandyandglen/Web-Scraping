
# Homework Assigment 13


```python
from bs4 import BeautifulSoup
from requests import get
from splinter import browser
from selenium import webdriver
import requests
import pandas as pd
```


```python
import pymongo
from pymongo import MongoClient

def setup_mongo():
    client = MongoClient('localhost', 27017)
    db = client.mars
    collection = db.m_data
    return collection

collection = setup_mongo()
```


```python
driver = webdriver.Chrome(r'C:\Users\mandy\week13\Chrome\chromedriver') 

# Scrape Function

def scraped():
    m_data = {}
    m_data["news"] = news()
    m_data["weather"] = weather()
    m_data["facts"] = facts()
    m_data["mars_image"] = image()
    m_data["hemisphere"] = hemisphere()

    return m_data
```


```python
def news():
    news_url = "https://mars.nasa.gov/news/?page=0&per_page=40&order=publish_date+desc%2Ccreated_at+desc&search=&category=19%2C165%2C184%2C204&blank_scope=Latest"
    driver.get(news_url)
    html = driver.page_source
    soup = BeautifulSoup(html, 'html.parser')
    news_title = soup.find("div", class_ = "content_title").text
    news_content = soup.find("div", class_ = "article_teaser_body").text
    news = [news_title, news_content]
    return news

def weather():
    twitter_url = url = "https://twitter.com/marswxreport?lang=en"
    html = driver.page_source
    response = requests.get(url)
    soup = BeautifulSoup(response.text,"lxml")
    tweets = soup.findAll('li',{"class":'js-stream-item'})
    tweet_records = []
    for tweet in tweets:
        if tweet.find('p',{"class":'tweet-text'}):
            tweet_text = tweet.find('p',{"class":'tweet-text'}).text.encode('utf8').strip()
        tweet_records.append(tweet_text)
    mars_weather_status = tweet_records[1]
    return mars_weather_status

def facts():
    data = requests.get("https://space-facts.com/mars/")
    soup = BeautifulSoup(data.content, 'lxml')
    table = soup.find_all('table')[0]
    mdata = pd.read_html(str(table))[0]
    mdata.columns = ["Description", "Value"]
    mdata = mdata.set_index("Description")
    facts = mdata.to_html(index = True, header =True)
    return facts

def image():
    image_url = "https://www.jpl.nasa.gov/spaceimages/?search=&category=Mars"
    driver.get(image_url)
    html = driver.page_source
    soup = BeautifulSoup(html, 'html.parser')
    images = soup.find("a", class_ = "button fancybox")
    featured_image_url = image_url + images.get('data-fancybox-href')
    return featured_image_url 

def hemisphere():
    hemisphere_url = "https://astrogeology.usgs.gov/search/results?q=hemisphere+enhanced&k1=target&v1=Mars"
    driver.get(hemisphere_url)
    html = driver.page_source
    soup = BeautifulSoup(html, 'html.parser')
    hemisphere_list = []
    products = soup.find("div", class_ = "result-list" )
    hemispheres = products.find_all("div", class_="item")

    for hemisphere in hemispheres:
        title = hemisphere.find("h3").text
        title = title.replace("Enhanced", "")
        end_link = hemisphere.find("a")["href"]
        image_url = "https://astrogeology.usgs.gov/" + end_link
        hemisphere_list.append({"title": title, "img_url": image_url})
        return hemisphere_list

data = scraped()   

```


```python
data
```




    {'facts': '<table border="1" class="dataframe">\n  <thead>\n    <tr style="text-align: right;">\n      <th></th>\n      <th>Value</th>\n    </tr>\n    <tr>\n      <th>Description</th>\n      <th></th>\n    </tr>\n  </thead>\n  <tbody>\n    <tr>\n      <th>Equatorial Diameter:</th>\n      <td>6,792 km</td>\n    </tr>\n    <tr>\n      <th>Polar Diameter:</th>\n      <td>6,752 km</td>\n    </tr>\n    <tr>\n      <th>Mass:</th>\n      <td>6.42 x 10^23 kg (10.7% Earth)</td>\n    </tr>\n    <tr>\n      <th>Moons:</th>\n      <td>2 (Phobos &amp; Deimos)</td>\n    </tr>\n    <tr>\n      <th>Orbit Distance:</th>\n      <td>227,943,824 km (1.52 AU)</td>\n    </tr>\n    <tr>\n      <th>Orbit Period:</th>\n      <td>687 days (1.9 years)</td>\n    </tr>\n    <tr>\n      <th>Surface Temperature:</th>\n      <td>-153 to 20 °C</td>\n    </tr>\n    <tr>\n      <th>First Record:</th>\n      <td>2nd millennium BC</td>\n    </tr>\n    <tr>\n      <th>Recorded By:</th>\n      <td>Egyptian astronomers</td>\n    </tr>\n  </tbody>\n</table>',
     'hemisphere': [{'img_url': 'https://astrogeology.usgs.gov//search/map/Mars/Viking/cerberus_enhanced',
       'title': 'Cerberus Hemisphere '}],
     'mars_image': 'https://www.jpl.nasa.gov/spaceimages/?search=&category=Mars/spaceimages/images/mediumsize/PIA19041_ip.jpg',
     'news': ["Six Things About Opportunity's Recovery Efforts",
      'The global dust storm on Mars could soon let in enough sunlight for the Opportunity rover to recharge.'],
     'weather': b'Sol 2141 (2018-08-14), high -17C/1F, low -66C/-86F, pressure at 8.63 hPa, daylight 05:27-17:41'}




```python
collection.insert_one(data)
```




    <pymongo.results.InsertOneResult at 0x1f5e0615870>


