# amazon-python-scrapy-scraper
Python Scrapy spiders that scrape product data and reviews from [Amazon.com](https://www.amazon.com/). 

This Scrapy project contains 3 seperate spiders:

| Spider  |      Description      |
|----------|-------------|
| `amazon_search` |  Scrapes all product data from the Amazon product search page for a given list of keywords. | 
| `amazon_search_product` |  Crawls Amazon product search pages for a given list of keywords, then scrapes each individual product page.extra+not tested | 
| `amazon_reviews` |  Scrapes all Amazon product reviews from a list of product ASINs. extra+not tested | 


## ScrapeOps Proxy

To use the ScrapeOps Proxy you need to first install the proxy middleware:

```python

pip install scrapeops-scrapy-proxy-sdk

```

Then activate the ScrapeOps Proxy by adding your API key to the `SCRAPEOPS_API_KEY` in the ``settings.py`` file.

```python

SCRAPEOPS_API_KEY = 'YOUR_API_KEY'

SCRAPEOPS_PROXY_ENABLED = True

DOWNLOADER_MIDDLEWARES = {
    'scrapeops_scrapy_proxy_sdk.scrapeops_scrapy_proxy_sdk.ScrapeOpsScrapyProxySdk': 725,
}

```


## ScrapeOps Monitoring


To use the ScrapeOps Proxy you need to first install the monitoring SDK:

```

pip install scrapeops-scrapy

```


Then activate the ScrapeOps Proxy by adding your API key to the `SCRAPEOPS_API_KEY` in the ``settings.py`` file.

```python

SCRAPEOPS_API_KEY = 'YOUR_API_KEY'

# Add In The ScrapeOps Monitoring Extension
EXTENSIONS = {
'scrapeops_scrapy.extension.ScrapeOpsMonitor': 500, 
}


DOWNLOADER_MIDDLEWARES = {

    ## ScrapeOps Monitor
    'scrapeops_scrapy.middleware.retry.RetryMiddleware': 550,
    'scrapy.downloadermiddlewares.retry.RetryMiddleware': None,
    
    ## Proxy Middleware
    'scrapeops_scrapy_proxy_sdk.scrapeops_scrapy_proxy_sdk.ScrapeOpsScrapyProxySdk': 725,
}

```


## Running The Scrapers

```

pip install scrapy scrapeops-scrapy

```



```python

def start_requests(self):
    keyword_list = ['ipad']
    for keyword in keyword_list:
        amazon_search_url = f'https://www.amazon.com/s?k={keyword}&page=1'
        yield scrapy.Request(url=amazon_search_url, callback=self.parse_search_results, meta={'keyword': keyword, 'page': 1})

```

run command:

```

scrapy crawl amazon_search_product

```

```python

def start_requests(self):
    keyword_list = ['ipad', 'laptops']
    for keyword in keyword_list:
        amazon_search_url = f'https://www.amazon.com/s?k={keyword}&page=1'
        yield scrapy.Request(url=amazon_search_url, callback=self.parse_search_results, meta={'keyword': keyword, 'page': 1})

```



```python

def parse_product_data(self, response):
    image_data = json.loads(re.findall(r"colorImages':.*'initial':\s*(\[.+?\])},\n", response.text)[0])
    variant_data = re.findall(r'dimensionValuesDisplayData"\s*:\s* ({.+?}),\n', response.text)
    feature_bullets = [bullet.strip() for bullet in response.css("#feature-bullets li ::text").getall()]
    price = response.css('.a-price span[aria-hidden="true"] ::text').get("")
    if not price:
        price = response.css('.a-price .a-offscreen ::text').get("")
    yield {
        "name": response.css("#productTitle::text").get("").strip(),
        "price": price,
        "stars": response.css("i[data-hook=average-star-rating] ::text").get("").strip(),
        "rating_count": response.css("div[data-hook=total-review-count] ::text").get("").strip(),
        "feature_bullets": feature_bullets,
        "images": image_data,
        "variant_data": variant_data,
    }

```



```python
# settings.py

CONCURRENT_REQUESTS = 10

```


```python

custom_settings = {
        'FEEDS': { 'data/%(name)s_%(time)s.csv': { 'format': 'csv',}}
        }

```



```python
# settings.py

# SCRAPEOPS_API_KEY = 'YOUR_API_KEY'

# SCRAPEOPS_PROXY_ENABLED = True

# EXTENSIONS = {
# 'scrapeops_scrapy.extension.ScrapeOpsMonitor': 500, 
# }

# DOWNLOADER_MIDDLEWARES = {

#     ## ScrapeOps Monitor
#     'scrapeops_scrapy.middleware.retry.RetryMiddleware': 550,
#     'scrapy.downloadermiddlewares.retry.RetryMiddleware': None,
    
#     ## Proxy Middleware
#     'scrapeops_scrapy_proxy_sdk.scrapeops_scrapy_proxy_sdk.ScrapeOpsScrapyProxySdk': 725,
# }



```

