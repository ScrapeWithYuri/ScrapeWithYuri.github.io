---
layout: post
title:  "Scrape Zillow - Hundreds of Thousands of Properties Pt. 2 without Tax Data"
date:   2019-02-10 19:31:43 +0700
categories: [python, web scraping, Zillow]
---
This tutortial will walk you through how to scrape hundreds of thousands of Zillow property data. This article assumes you have Zillow property URLs already or have already read the article Scrape Zillow - Hundreds of Thousands of Properties Pt. 1.

<b>Prerequites</b>

<ul><li>Python 2</li>
<li>Libraries: itertools, lxml, gevent, codecs, sys, json, time</li>
<li>Zillow prorperty URLs without the zillow.com prefix</li></ul>

<br>

<b>General Overview</b>

The below program will read in Zillow property URLs (without the zillow.com prefix). The program will loop the URLs in slices (i.e. X lines of URLs per loop) then run the URLs via multiple threads. Using multiple threads allows the program to be much faster than searching with a single thread (i.e. one-by-one).

Generally, the program is pinging the Zillow property URL and looking for the hdpApolloPreloadedData HTML element by ID. This HTML element has JSON data about the property, which the program processes. After extracting the relevant data, the program will save the data in as tab-delimited based on your chosen location.

The downside by pinging the Zillow URL directly is that property tax and school information is not accessible. Extracting this data is explained in the article Scrape Zillow - Hundreds of Thousands of Properties Pt. 2 w/ Tax Data.

<br>

<b>Required Update</b>

<ul><li>Before you start, open a web browser. Open the developer window (either CTRL + SHIFT + I or right click > press Inspect). Search any Zillow property URL, and click on the Network tab of the developer tab. In the Networ tab, scroll to the top and click on the URL you had searched. In the Response Headers section, there should be a set-cookie value. Copy the AWSALB cookie value into update_your_cookie_here section of the code.</li></ul>

{% highlight python %}
headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64; Trident/7.0; Touch; rv:11.0) like Gecko',
    'Connection': 'close',
    'Host': 'www.zillow.com',
    'Accept-Language': 'en-US',
    'Accept': 'text/html, application/xhtml+xml, application/xml; q=0.9, */*; q=0.8',
    'Accept-Encoding': 'gzip, deflate, br',
    'Upgrade-Insecure-Requests': '1',
    'Cache-Control': 'max-age=0',
    'Cookie': 'AWSALB=update_your_cookie_here',
}
{% endhighlight %}

<br>

<b>Update to your liking</b>

<ul><li>Update where the program will read Zillow URLs from. Update the value path_where_your_data_is_here.</li></ul>

{% highlight python %}
with codecs.open('path_where_your_data_is_here', mode='r', encoding='utf-8') as f:
    while True:
        lines = list(islice(f, 50000))

        if not lines:
            break

        for qm in lines:
            hold = qm.replace('\n', '')
            pool.apply_async(process, args=('https://www.zillow.com' + hold + '?fullpage=true', hold ,))
        pool.join()
{% endhighlight %}

<br>

<ul><li>Update the save location. This is where your output will be saved. Update the value path_where_you_want_saved_here.</li></ul>

{% highlight python %}
def write_file1(hold):
    with codecs.open('path_where_you_want_saved_here', mode='a', encoding='utf-8') as f:
        f.write("{}\n".format(hold))
{% endhighlight %}

<br>

<ul><li>The program will read data by 'slice.' In other words, this is how many lines will be read per loop. The more lines read in per loop, the more memory is used by your computer. Update 50000 based on your needs.</li></ul>

{% highlight python %}
with codecs.open('path_where_your_data_is_here', mode='r', encoding='utf-8') as f:
    while True:
        lines = list(islice(f, 50000))

        if not lines:
            break

        for qm in lines:
            hold = qm.replace('\n', '')
            pool.apply_async(process, args=('https://www.zillow.com' + hold + '?fullpage=true', hold ,))
        pool.join()
{% endhighlight %}

<br>

<ul><li>The program will use worker threads to run searches simultaneously. The more threads, the more processing power your computer will require. Update 300 based on your needs.</li></ul>

{% highlight python %}
num_worker_threads = 300
pool = Pool(num_worker_threads)
{% endhighlight %}

<br>

<b>Full Code</b>

{% highlight python %}
from lxml import html
from gevent.pool import Pool
from gevent import monkey; monkey.patch_all()
from itertools import islice
import requests, codecs, sys, json, time

if sys.version[0] == '2':
    reload(sys)
    sys.setdefaultencoding("utf-8")

def response(site, f):
    headers['Referer'] = 'https://www.zillow.com' + f
    qq = requests.get(site, headers=headers)
    return qq.text

def write_file1(hold):
    with codecs.open('path_where_you_want_saved_here', mode='a', encoding='utf-8') as f:
        f.write("{}\n".format(hold))

def tree_check1(tree, hold, f):
    test = tree.xpath(hold)
    if len(test) > 0:
        j = json.loads(test[0])

        for qq in j:
            if 'SEORenderQuery' in qq:
                holding = j[qq]['property']
                address = str(holding['streetAddress']) + ', ' + str(holding['city']) + ', ' + str(holding['state']) + ' ' + str(holding['zipcode'])
                status = str(holding['homeStatus'])
                price = str(holding['price'])
                bed = str(holding['bedrooms'])
                bath = str(holding['bathrooms'])
                living = str(holding['livingArea'])
                type = str(holding['homeType'])
                lotsize = ''
                if 'lotSize' in holding:
                    lotsize = str(holding['lotSize'])
                zest = str(holding['zestimate'])
                fsbo = str(holding['listing_sub_type']['is_FSBO'])
                fsba = str(holding['listing_sub_type']['is_FSBA'])
                newhome = str(holding['listing_sub_type']['is_newHome'])
                foreclose = str(holding['listing_sub_type']['is_foreclosure'])
                bankowned = str(holding['listing_sub_type']['is_bankOwned'])
                auction = str(holding['listing_sub_type']['is_forAuction'])
                comingsoon = str(holding['listing_sub_type']['is_comingSoon'])

                glance = ''
                if not holding['homeFacts']['atAGlanceFacts'] is None:
                    for fg in holding['homeFacts']['atAGlanceFacts']:
                        glance += '\t' + str(fg['factLabel']) + ': ' + str(fg['factValue'])

                if not holding['homeFacts']['categoryDetails'] is None:
                    for fg in holding['homeFacts']['categoryDetails']:
                        pp = str(fg['categoryGroupName'])
                        for mm in fg['categories']:
                            pz = str(mm['categoryName'])
                            for oo in mm['categoryFacts']:
                                glance += '\t' + pp + ': ' + pz + ': ' + str(oo['factLabel']) + ': ' + str(oo['factValue'])

                write_file1(f + '\t' + address + '\t' + status + '\t' + price  + '\t' + bed + '\t' + bath + '\t' + living + '\t' + type + '\t' + lotsize + '\t' + zest
                            + '\t' + fsbo  + '\t' + fsba + '\t' + newhome + '\t' + foreclose + '\t' + bankowned + '\t' + auction + '\t' + comingsoon + '\t' + glance
                            )
                break

def process(site, f):
    hold = response(site, f)
    if 'name="robots"' in hold:
        sys.exit()
    tree = html.fromstring(hold)
    tree_check1(tree, X_PATH1, f)
    del tree

headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64; Trident/7.0; Touch; rv:11.0) like Gecko',
    'Connection': 'close',
    'Host': 'www.zillow.com',
    'Accept-Language': 'en-US',
    'Accept': 'text/html, application/xhtml+xml, application/xml; q=0.9, */*; q=0.8',
    'Accept-Encoding': 'gzip, deflate, br',
    'Upgrade-Insecure-Requests': '1',
    'Cache-Control': 'max-age=0',
    'Cookie': 'AWSALB=update_your_cookie_here',
}

X_PATH1 = "//*[@id='hdpApolloPreloadedData']/text()"

num_worker_threads = 300
pool = Pool(num_worker_threads)

with codecs.open('path_where_your_data_is_here', mode='r', encoding='utf-8') as f:
    while True:
        lines = list(islice(f, 50000))

        if not lines:
            break

        for qm in lines:
            hold = qm.replace('\n', '')
            pool.apply_async(process, args=('https://www.zillow.com' + hold + '?fullpage=true', hold ,))
        pool.join()
{% endhighlight %}
