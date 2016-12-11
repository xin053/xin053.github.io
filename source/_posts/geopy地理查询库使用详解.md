---
title: geopy地理查询库使用详解
date: 2016-11-06 14:39:39
categories:
- Python模块学习
tags:
- Python
- geopy
---

## geopy简介

可以使用geopy库来查询地址，国家，城市，地标，geopy使用的是第三方的geo解析器(包括谷歌地图，必应地图，Nominatim等)和一些数据源来获取地理信息

Each geolocation service you might use, such as Google Maps, Bing Maps, or Yahoo BOSS, has its own class in `geopy.geocoders` abstracting the service’s API. Geocoders each define at least a`geocode` method, for resolving a location from a string, and may define a `reverse` method, which resolves a pair of coordinates to an address.

<!-- more -->

## geopy使用

### 从地址字符串获取`Location`对象

```python
>>> from geopy.geocoders import Nominatim
>>> geolocator = Nominatim()
>>> location = geolocator.geocode("175 5th Avenue NYC")
>>> print(location.address)
Flatiron Building, 175, 5th Avenue, Flatiron, New York, NYC, New York, ...
>>> print((location.latitude, location.longitude))
(40.7410861, -73.9896297241625)
>>> print(location.raw)
{'place_id': '9167009604', 'type': 'attraction', ...}
```

### 从经纬度获取`Location`对象

```python
>>> from geopy.geocoders import Nominatim
>>> geolocator = Nominatim()
>>> location = geolocator.reverse("52.509669, 13.376294")
>>> print(location.address)
Potsdamer Platz, Mitte, Berlin, 10117, Deutschland, European Union
>>> print((location.latitude, location.longitude))
(52.5094982, 13.3765983)
>>> print(location.raw)
{'place_id': '654513', 'osm_type': 'node', ...}
```

### 计算两点间距离

可以使用  [Vincenty distance](https://en.wikipedia.org/wiki/Vincenty's_formulae) 或 [great-circle distance](https://en.wikipedia.org/wiki/Great-circle_distance)

```python
>>> from geopy.distance import vincenty
>>> newport_ri = (41.49008, -71.312796)
>>> cleveland_oh = (41.499498, -81.695391)
>>> print(vincenty(newport_ri, cleveland_oh).miles)
538.3904451566326
```

```python
>>> from geopy.distance import great_circle
>>> newport_ri = (41.49008, -71.312796)
>>> cleveland_oh = (41.499498, -81.695391)
>>> print(great_circle(newport_ri, cleveland_oh).miles)
537.1485284062816
```

## 各三方地理服务API

###  ArcGIS

> *class*`geopy.geocoders.ArcGIS`(*username=None*, *password=None*, *referer=None*, *token_lifetime=60*,*scheme='https'*, *timeout=1*, *proxies=None*, *user_agent=None*)

[参数详解](https://geopy.readthedocs.io/en/latest/#geopy.geocoders.ArcGIS)

### Baidu

> *class*`geopy.geocoders.Baidu`(*api_key*, *scheme='http'*, *timeout=1*, *proxies=None*, *user_agent=None*)

[参数详解](https://geopy.readthedocs.io/en/latest/#geopy.geocoders.Baidu)

### Bing

> *class*`geopy.geocoders.Bing`(*api_key*, *format_string='%s'*, *scheme='https'*, *timeout=1*, *proxies=None*,*user_agent=None*)

[参数详解](https://geopy.readthedocs.io/en/latest/#geopy.geocoders.Bing)

### DataBC

> *class*`geopy.geocoders.DataBC`(*scheme='https'*, *timeout=1*, *proxies=None*, *user_agent=None*)

[参数详解](https://geopy.readthedocs.io/en/latest/#geopy.geocoders.DataBC)

### GeocodeFarm

> *class*`geopy.geocoders.GeocodeFarm`(*api_key=None*, *format_string='%s'*, *timeout=1*, *proxies=None*,*user_agent=None*)

[参数详解](https://geopy.readthedocs.io/en/latest/#geopy.geocoders.GeocodeFarm)

### GeocoderDotUS

> *class*`geopy.geocoders.GeocoderDotUS`(*username=None*, *password=None*, *format_string='%s'*,*timeout=1*, *proxies=None*, *user_agent=None*)

[参数详解](https://geopy.readthedocs.io/en/latest/#geopy.geocoders.GeocoderDotUS)

### GeoNames

> *class*`geopy.geocoders.GeoNames`(*country_bias=None*, *username=None*, *timeout=1*, *proxies=None*,*user_agent=None*)

[参数详解](https://geopy.readthedocs.io/en/latest/#geopy.geocoders.GeoNames)

### GoogleV3

> *class*`geopy.geocoders.GoogleV3`(*api_key=None*, *domain='maps.googleapis.com'*, *scheme='https'*,*client_id=None*, *secret_key=None*, *timeout=1*, *proxies=None*, *user_agent=None*)

[参数详解](https://geopy.readthedocs.io/en/latest/#geopy.geocoders.GoogleV3)

### IGNFrance

> *class*`geopy.geocoders.IGNFrance`(*api_key*, *username=None*, *password=None*, *referer=None*,*domain='wxs.ign.fr'*, *scheme='https'*, *timeout=1*, *proxies=None*, *user_agent=None*)

[参数详解](https://geopy.readthedocs.io/en/latest/#geopy.geocoders.IGNFrance)

### LiveAddress

> *class*`geopy.geocoders.LiveAddress`(*auth_id*, *auth_token*, *candidates=1*, *scheme='https'*, *timeout=1*,*proxies=None*, *user_agent=None*)

[参数详解](https://geopy.readthedocs.io/en/latest/#geopy.geocoders.LiveAddress)

### NaviData

> *class*`geopy.geocoders.NaviData`(*api_key=None*, *domain='api.navidata.pl'*, *timeout=1*, *proxies=None*,*user_agent=None*)

[参数详解](https://geopy.readthedocs.io/en/latest/#geopy.geocoders.NaviData)

### Nominatim

> *class*`geopy.geocoders.Nominatim`(*format_string='%s'*, *view_box=None*, *country_bias=None*, *timeout=1*,*proxies=None*, *domain='nominatim.openstreetmap.org'*, *scheme='https'*, *user_agent=None*)

[参数详解](https://geopy.readthedocs.io/en/latest/#geopy.geocoders.Nominatim)

### OpenCage

> *class*`geopy.geocoders.OpenCage`(*api_key*, *domain='api.opencagedata.com'*, *scheme='https'*, *timeout=1*,*proxies=None*, *user_agent=None*)

[参数详解](https://geopy.readthedocs.io/en/latest/#geopy.geocoders.OpenCage)

### OpenMapQuest

> *class*`geopy.geocoders.OpenMapQuest`(*api_key=None*, *format_string='%s'*, *scheme='https'*, *timeout=1*,*proxies=None*, *user_agent=None*)

[参数详解](https://geopy.readthedocs.io/en/latest/#geopy.geocoders.OpenMapQuest)

### Photon

> *class*`geopy.geocoders.Photon`(*format_string='%s'*, *scheme='https'*, *timeout=1*, *proxies=None*,*domain='photon.komoot.de'*)

[参数详解](https://geopy.readthedocs.io/en/latest/#geopy.geocoders.Photon)

### YahooPlaceFinder

> *class*`geopy.geocoders.YahooPlaceFinder`(*consumer_key*, *consumer_secret*, *timeout=1*, *proxies=None*,*user_agent=None*)

[参数详解](https://geopy.readthedocs.io/en/latest/#geopy.geocoders.YahooPlaceFinder)

### What3Words

> *class*`geopy.geocoders.What3Words`(*api_key*, *format_string='%s'*, *scheme='https'*, *timeout=1*,*proxies=None*, *user_agent=None*)

[参数详解](https://geopy.readthedocs.io/en/latest/#geopy.geocoders.What3Words)

### Yandex

> *class*`geopy.geocoders.Yandex`(*api_key=None*, *lang=None*, *timeout=1*, *proxies=None*, *user_agent=None*)

[参数详解](https://geopy.readthedocs.io/en/latest/#geopy.geocoders.Yandex)

## 参考文档

- [geogy官方文档](https://geopy.readthedocs.io/en/latest/)
