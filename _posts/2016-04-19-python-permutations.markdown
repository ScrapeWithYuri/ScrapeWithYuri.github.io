---
layout: post
title:  "Scrape Zillow - Hundreds of Thousands of Properties Pt. 2 w/o Tax Data"
date:   2019-02-10 19:31:43 +0700
categories: [python, web scraping, Zillow]
---
This tutortial will walk you through how to scrape hundreds of thousands of Zillow property data. This article assumes you have Zillow property URLs and ZPIDs already or have already read the article Scrape Zillow - Hundreds of Thousands of Properties Pt. 1.

<b>Prerequites</b>

1. Python 2
2. Libraries: itertools, lxml, gevent, codecs, sys, json, time

{% highlight ruby %}
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
    with codecs.open('C:\\Users\Russi\Desktop\T.txt', mode='a', encoding='utf-8') as f:
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
    'Cookie': 'AWSALB=QJacahdr8WQeYl9Tpu/orPjKGrc67gM0+FX5lr2fSx2q+LmoY+HleoJlqGgNvU33m+RARRKzEHf7b6hEZbLJ0W0RGmKFGEqgYwBOxWsRTVSm8zY67NRfLx6oPaD7;',
}

X_PATH1 = "//*[@id='hdpApolloPreloadedData']/text()"

num_worker_threads = 300
pool = Pool(num_worker_threads)

with codecs.open('C:\\Users\Russi\Desktop\TTTT.txt', mode='r', encoding='utf-8') as f:
    while True:
        lines = list(islice(f, 50000))

        if not lines:
            break

        for qm in lines:
            hold = qm.replace('\n', '')
            pool.apply_async(process, args=('https://www.zillow.com' + hold + '?fullpage=true', hold ,))
        pool.join()
{% endhighlight %}
