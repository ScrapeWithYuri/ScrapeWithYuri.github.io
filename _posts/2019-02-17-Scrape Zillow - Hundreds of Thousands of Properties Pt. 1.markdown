---
layout: post
title:  "Scrape Zillow - Hundreds of Thousands of Properties Pt. 1"
date:   2019-02-17 19:31:43 +0700
categories: [web scraping, visual basic, Excel VBA, Zillow]
---

<br>

This tutortial will walk you through how to scrape hundreds of thousands of Zillow property URLs to be used in <a href="/python/web%20scraping/zillow/2019/02/10/Scrape-Zillow-Hundreds-of-Thousands-of-Properties-Pt.-2-without-Tax-Data.html">Part 2</a>.

<br>

<ol start="1"><li><b>Prerequites</b></li></ol>

<ul><li>Excel VBA</li></ul>

<br>

<ol start="2"><li><b>General Overview</b></li></ol>

The <a href="#full-code">full code</a> is at the bottom of the article. The code uses Internet Explorer, rather than Selenium, for one simple reason; most bot detection protocols seem to ignore Internet Explorer. The code below will appear sloppy, but this is the downside with using Internet Explorer.

In Excel, open Visual Basic (either go to the Developer tab or press Alt + F11). On the ribbon, press Insert > Module. Copy and paste the code into the module. Add zip codes you want to extract Zillow URLs into the first sheet down Column A. Create a second sheet, so the code can place the Zillow URLs.

The <b><a href="#update-to-your-liking">Update to your liking</a></b> section will help you update which sheets the code will read from and write to.

<br>

<ol start="3" id="update-to-your-liking"><li><b>Update to your liking</b></li></ol>

You will notice Sheet1 and Sheet2 in the code. These are referencing the code names in VBA. Update depending on how your sheet structure is set up.

<br>

<img src="{{ "/static/img/visual basic.png" | prepend: site.baseurl | replace: '//', '/' }}" />

<br>

<ol start="4" id="full-code"><li><b>Full Code</b></li></ol>

{% highlight visual basic %}
Option Explicit
#If VBA7 Then
    Public Declare PtrSafe Sub Sleep Lib "kernel32" (ByVal dwMilliseconds As LongPtr) 'For 64 Bit Systems
#Else
    Public Declare Sub Sleep Lib "kernel32" (ByVal dwMilliseconds As Long) 'For 32 Bit Systems
#End If
Public Sub zillow()
    Dim bot As Object, id As Object, zillow As String, zip_code As String, i As Long, j As Long, id1 As Object, m As Long, zz As Object
    
    With Application: .ScreenUpdating = False: .EnableEvents = False: .DisplayAlerts = False: End With
        
    Set bot = CreateObject("InternetExplorer.Application")
    
    zillow = "https://www.zillow.com/homes/for_sale/94002_rb/13_zm/0_mmm/1_rs"
    i = 1
    j = 2
    
    bot.Visible = True
    Do While CBool(LenB(Trim$(Sheet2.Cells(i, 1))))
        zip_code = Trim$(Sheet2.Cells(i, 1))
        For m = 1 To 11
here11:
            Err.Clear
            On Error GoTo 0
            On Error GoTo here2
                bot.navigate Replace(zillow, "/94002_rb", "/" & zip_code & "_rb") & "/house,condo,apartment_duplex,mobile,townhouse_type/" & (m - 1) * 100000 & "-" & IIf(m < 11, m * 100000, "") & "_price"
                Sleep 4000
            On Error GoTo 0
            
here1:
            On Error GoTo 0
            On Error GoTo here2
                Set id1 = bot.document.getelementsbyclassname("photo-cards")(0).getelementsbytagname("article")
                For Each id In id1
                    Sheet1.Cells(j, 1) = id.getelementsbytagname("a")(0).href
                    j = j + 1
                Next id
            On Error GoTo 0
            
            Set id = Nothing
            On Error Resume Next
                Set id = bot.document.getelementsbyclassname("zsg-pagination-next")(0).getelementsbytagname("a")(0)
                If Not id Is Nothing Then
                    id.Click
                    Sleep 2500
                    GoTo here1
                End If
            On Error GoTo 0
            Err.Clear
        Next m
        
        i = i + 1
    Loop
    
    Set bot = Nothing
    
    With Application: .ScreenUpdating = True: .EnableEvents = True: .DisplayAlerts = True: End With
    Exit Sub
here2:
    On Error GoTo 0
    Sleep 2000
    Set zz = CreateObject("Shell.Application").Windows
    Set bot = zz.Item(zz.Count - 1)
    Sleep 1000
    bot.Stop
    Sleep 1500
    Set bot = CreateObject("InternetExplorer.Application")
    bot.Visible = True
    Set zz = Nothing
    Resume here11
End Sub
{% endhighlight %}
