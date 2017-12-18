---
layout: post
title: 使用 Nokogiri 爬 Youtube 頁面
abstract: 直接使用 Nokogiri 爬回來的 Youtube 資料會是亂碼，該怎麼辦呢？
comments: true
---

## 寫在開始之前

之前有一陣子需要寫爬蟲來爬網站，掌握了基本規則後好像一切都蠻順利的，直到有天去爬了 Youtube 之後發現....。

好啦這不是農場文 XD 大概就是 Youtube 爬回來的東西都是一場災難，各種我看不懂的亂碼，然後就想說要怎麼解這個問題，然後就寫成這篇筆記～

## 正文來了

### 先「讀」HTML，再「解析」

開啟 Ruby 的 irb，我們來試著爬出 Youtube 頁面的標題，原本爬出來的結果應該是：

    require "nokogiri"
    require "open-uri"

    url = "https://www.youtube.com/watch?v=NiHF-cwto_A"
    doc1 = Nokogiri::HTML(open(url))
    title = doc1.xpath("//h1")[1]
    puts title

    #=> <h1 class="watch-title-container">
    <span id="eow-title" class="watch-title" dir="ltr" title="[ &ccedil;&acute;&#148;&auml;&ordm;&laquo;&ccedil;&#137;&#136; ] &aelig;&#158;&#151;&auml;&iquest;&#138;&aring;&#130;&#145;&atilde;&#128;&#138;&egrave;&frac14;&cedil;&auml;&ordm;&#134;&aring;&brvbar;&sup3;&egrave;&acute;&#143;&auml;&ordm;&#134;&auml;&cedil;&#150;&ccedil;&#149;&#140;&aring;&#143;&#136;&aring;&brvbar;&#130;&auml;&frac12;&#149;&atilde;&#128;&#139;&atilde;&#128;&#138;&aring;&curren;&cent;&aelig;&#131;&sup3;&ccedil;&#154;&#132;&egrave;&#129;&sup2;&eacute;&#159;&sup3;2&atilde;&#128;&#139;EP.4 20171124 /&aelig;&micro;&#153;&aelig;&plusmn;&#159;&egrave;&iexcl;&#155;&egrave;&brvbar;&#150;&aring;&reg;&#152;&aelig;&#150;&sup1;HD/">
      [ &ccedil;&acute;&#148;&auml;&ordm;&laquo;&ccedil;&#137;&#136; ] &aelig;&#158;&#151;&auml;&iquest;&#138;&aring;&#130;&#145;&atilde;&#128;&#138;&egrave;&frac14;&cedil;&auml;&ordm;&#134;&aring;&brvbar;&sup3;&egrave;&acute;&#143;&auml;&ordm;&#134;&auml;&cedil;&#150;&ccedil;&#149;&#140;&aring;&#143;&#136;&aring;&brvbar;&#130;&auml;&frac12;&#149;&atilde;&#128;&#139;&atilde;&#128;&#138;&aring;&curren;&cent;&aelig;&#131;&sup3;&ccedil;&#154;&#132;&egrave;&#129;&sup2;&eacute;&#159;&sup3;2&atilde;&#128;&#139;EP.4 20171124 /&aelig;&micro;&#153;&aelig;&plusmn;&#159;&egrave;&iexcl;&#155;&egrave;&brvbar;&#150;&aring;&reg;&#152;&aelig;&#150;&sup1;HD/
    </span>
        </h1>

但如果我們不要一開始就用 Nokogiri 去「爬」頁面，而是先「讀」回來，再去解析的話：

    html = open(URI::encode(url)).read
    doc2 = Nokogiri::HTML(html)
    title2 = doc2.xpath("//h1")[1]
    puts title2

    #=> <h1 class="watch-title-container">
    <span id="eow-title" class="watch-title" dir="ltr" title="[ 純享版 ] 林俊傑《輸了妳贏了世界又如何》《夢想的聲音2》EP.4 20171124 /浙江衛視官方HD/">
      [ 純享版 ] 林俊傑《輸了妳贏了世界又如何》《夢想的聲音2》EP.4 20171124 /浙江衛視官方HD/
    </span>
      </h1>

We made it 😊️

### 使用 oembed 的 endpoint 獲取資訊

[oembed](https://oembed.com/) 是一種開放格式，藉由其 API 可以直接獲取網站的內容，而不用去解析該頁面。

可以直接參考上面官網的介紹，裡面有列出支援網站的名稱以及 endpoint，格式上提供 json & xml 兩種。

    require "json"

    url = "https://www.youtube.com/watch?v=NiHF-cwto_A"
    endpoint = "http://www.youtube.com/oembed?url="
    uri = URI(endpoint + url)

    response = Net::HTTP.get_response(uri)
    result = JSON.parse(response.body)

    #=> {"version"=>"1.0", "author_url"=>"https://www.youtube.com/channel/UCp8cPvx_Od76EL66vvCnhRA", "thumbnail_height"=>360, "type"=>"video", "thumbnail_url"=>"https://i.ytimg.com/vi/NiHF-cwto_A/hqdefault.jpg", "provider_url"=>"https://www.youtube.com/", "html"=>"<iframe width=\"480\" height=\"270\" src=\"https://www.youtube.com/embed/NiHF-cwto_A?feature=oembed\" frameborder=\"0\" gesture=\"media\" allow=\"encrypted-media\" allowfullscreen></iframe>", "provider_name"=>"YouTube", "height"=>270, "title"=>"[ 纯享版 ] 林俊杰《输了你赢了世界又如何》《梦想的声音2》EP.4 20171124 /浙江卫视官方HD/", "thumbnail_width"=>480, "width"=>480, "author_name"=>"浙江卫视音乐频道 ZJSTV Music Channel - 梦想的声音正在热播 -"}


We made it again 😊️

## 參考資料

* https://oembed.com/
