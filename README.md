## What I wanted to get by scraping

I wanted to scrape tomorrow's hourly weather data from [weather.com](https://weather.com/), including temperature, cloudiness, wind, and the chance of rain, for eight Florida beaches that are within driving distance of Gainesville, Florida. By writing this to a CSV, on any given day, I could quickly sort by each of those weather factors to determine which beach would have the best weather.

## The steps I took

Though I already have fairly detailed comments in the code itself, here are the steps I took to accomplish my scraping goal:

- First, I made sure to import all the Python libraries that I knew I was definitely going to need, such as BeautifulSoup, requests, time, and csv.

```python
from bs4 import BeautifulSoup
import requests
import time
import csv
```

- Before writing the hourly weather data to a CSV, I needed to actually scrape the data, which I accomplish using the *scrape_hourly_weather* function.
- But first, I knew I was going to need to create a list containing a URL for each beach's hourly weather page that I could then loop through to scrape each beach's hourly weather data. To ensure that I did not mix up the URLs, I put each beach's URL in an object that also included the beach's name.

```python
beach_list = [{'name':'Cedar Key', 'url':'https://weather.com/weather/hourbyhour/l/Cedar+Key+FL?canonicalCityId=bb28a62d2243110b4fb73bc365ff5055a642b46474198b2d0d95907489f821f9'}, {'name':'Crescent Beach', 'url':'https://weather.com/weather/hourbyhour/l/Crescent+Beach+FL+USFL0994:1:US'}, {'name':'Fernandina Beach', 'url':'https://weather.com/weather/hourbyhour/l/Fernandina+Beach+FL?canonicalCityId=e2b080b1185d1ed5dddf546006ca29ba4c56243763a9fe5bc3ba12868343bad9'}, {'name':'Flagler Beach', 'url':'https://weather.com/weather/hourbyhour/l/Flagler+Beach+FL?canonicalCityId=257a6dc20177db56e73faaf6bf1c6a984bdeec76761d81764e944aaa85b25ca1'}, {'name':'Jacksonville Beach', 'url':'https://weather.com/weather/hourbyhour/l/Jacksonville+Beach+FL?canonicalCityId=8cd17a4ef12fc87328ae54dd7a3adb84f37d3ef7b70c791800e070c8261f5f6c'}, {'name':'Neptune Beach', 'url':'https://weather.com/weather/hourbyhour/l/Neptune+Beach+FL?canonicalCityId=41fd51fa06c503f9abf98d97876d9b6618e773c69d8b4c8c0160b52b0abb696b'}, {'name':'St. Augustine Beach', 'url':'https://weather.com/weather/hourbyhour/l/587a164e718da1d014e46176e4dceae6c76525f26ed40c4a79fa14deb1a8d33f'}, {'name':'St. Pete Beach', 'url':'https://weather.com/weather/hourbyhour/l/c929d484ba3a462ae3f93dcd8040ca1e28a85e81d2c75da93da25469d38f4367'}]

```

- The *scrape_hourly_weather* function is pretty straightforward. The weather.com website has unique classes or identifiers for each category of data, so I didn't have to do any complicated BeautifulSoup gymnastics or use Selenium.
- But just to be safe, I defined the parent element containing all the weather data and performed all my *soup.find* functions within the parent element.

```python
parent = soup.find('section', {'data-testid': 'HourlyForecast'})
```

- Now if you try to scrape weather data for the current day, any hours that have already passed will not be displayed on the website. So that's partly why I decided to scrape hourly weather for the *following* day, to ensure I could scrape a full range of hourly weather data. Also, for the purposes of planning ahead, it just made more sense to scrape the following day. This is why I made it so that BeautifulSoup would only scrape weather data after the second date/day header element.

```python
headers = parent.find_all('h2', {'class': 'HourlyForecast--longDate--J_Pdh'})

weather_rows = headers[1].find_all_next('summary', attrs={'class': 'Disclosure--Summary--3GiL4'})
```

- As you can see below, I didn't have to do anything complicated to scrape weather data for each category of data. Each weather factor could be scraped using a single specific identifier.
- And because people generally prefer to go to the beach during daylight, I only want it to scrape the hours from 5 a.m. to 10 p.m. I specify this in the for-loop with the addition of *[5:23]*.
- In some instances, I had some issues with the formatting of the text. These were quickly resolved by using *strip()*.
- Also, the degree icon for the temperature data was not translating to CSV properly. So I added *[:2]* at the end to indicate that I only want the first two characters of the temperature text.

```python
for row in weather_rows[5:23]:
    hour = row.find('h3', attrs={'data-testid': 'daypartName'}).text.strip()
    temp = row.find('span', attrs={'data-testid': 'TemperatureValue'}).text.strip()[:2]
    cloudiness = row.find('span', attrs={'class': 'DetailsSummary--extendedData--307Ax'}).text.strip()
    wind = row.find('span', attrs={'data-testid': 'Wind'}).text.strip()
    rain_chance = row.find('span', attrs={'data-testid': 'PercentageValue'}).text.strip()
```

- Then, I appended each weather variable to the *hourly_weather_data* list, which will be used to write the CSV rows of weather data for each beach.
- Again, as I did with *beach_list*, I used a key-value pairing for each weather variable, thereby allowing me to easily reference the data by category instead of index number when I write the CSV rows. I just think it's generally safer to reference pre-defined keys as opposed to index numbers.

```python
hourly_weather_data.append({'name':beach['name'], 'hour':hour, 'temp':temp, 'cloudiness':cloudiness, 'wind':wind, 'rain_chance':rain_chance})
```

- To avoid possibly overwhelming the server with requests and getting blocked, I also added a short time delay between scraping each beach's hourly weather page.
- I initially defined my *hour* variable as *time*, which caused an error with the time Python library. It took me a bit to realize my error.

```python
time.sleep(3)
```

- With the scraping function ready-to-go, I just had to then create a function to write the collected data to a CSV file, which I named *write_csv*.
- All the hourly weather data was already cleanly organized in a list, so I just had to loop through that list and write each object (hour) of weather data to its own row in the CSV file.

```python
for weather_hour in data_list:
    writer.writerow([weather_hour['name'], weather_hour['hour'], weather_hour['temp'], weather_hour['cloudiness'], weather_hour['wind'], weather_hour['rain_chance']])
```
This scraping project was written and created by Zachary Carnell as part of an assignment for the University of Florida. 
