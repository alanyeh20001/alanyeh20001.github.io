---
layout: post
title: 在 Rails 中使用 SQL statement 進行批次塞值(Bulk Insert)到 table
abstract: 在 Rails 想要大量塞資料到資料庫？跟著我這樣做就對了！
comments: true
---

## 寫在開始之前

最近碰到專案需要重新設計 database 的架構，當中會遇到需要把原本 table 中的 data migrate 到另一個新的 table 的情況。如果是幾十、幾百筆的話當然是小 case，但如果是上萬筆甚至是幾十萬筆的話就比較刺激惹～

## 舉個例子吧

舉個例子好了，假設現在有 Book & Info 兩個 model 以及 table，他們的 association 是

    book has_many infos
    info belongs_to book

也就是一本書裡面會有很多基本資料，所以 info 裡面可能就會有 published_date, author 等等的 column。

## 原本的方法

而現在在更新 Book 的過程中會需要去把 Info 裡面的資料全部放到新的 Meta table 裡面去，原本採用的方法是：

    # 更新其中一本 book 時
    Info.where(book_id: book.id).find_each(batch_size: 200) do |info|
      Meta.create(published_date: info.published_date, author: info.author)
    end

這裡解釋一下幾個東西：

* 首先我們用到了 find_each，可以設定一次要抓出多少筆資料。會這樣做的原因是因為資料量可能很大，可能一本書的 info 就有 10000 筆之類，如果一次抓出來的話很可能讓記憶體炸掉，因此我們可以用這樣的方法來限制筆數。

* 像這邊一次抓出來 200 筆，他會先放在記憶體，然後 do block 的參數 info 吃的就是其中一筆資料。

* 接著我們再將此筆資料用來 create 新的一筆資料在 Meta。

> 如果方法有需要重複用的話可能就需要考慮 Meta 裡面是否已經有這一筆資料，如 first_or_intitialize 之類的方法。

以這樣的方法如果我們有超多本書，而且每本書有超多筆 info，那麼雙重迴圈下來的時間會非常可觀。以我這一次的經驗而言，在本機跑這樣的指令需要一分鐘，然後上到 staging 跑需要 10 分鐘！！！！！！我是覺得這樣真的跑 hen 久 QQ

## SQL Bulk 語法？

因此後來就想說可以使用 SQL 的 bulk insert 來做這件事，於是就開始 google….，最後得到的 SQL 語法是

    INSERT INTO "target_table" (col1, col2)
      VALUES (val1_1, val1_2), (val2_1, val2_2), (val3_1, val3_2), ...

這個語法看起來應該不難懂，target_table 就是我們要塞資料進去的 table，也就是 metas；col1, col2 就是要賦值的欄位；而 values 就是每一筆資料的值。

## 怎麼處理資料？

所以這裡的想法是我們先把要用的值全部抓出來，然後整理成 VALUES 後面的格式，再配合整個 SQL 語句我們就可以大量的新增資料惹 😊

但是原本的 find_each 這個方法傳到 do block 裡面的還是單一筆資料，我們需要的是很多筆資料，因此這邊我們換成另一個語法，如下

    Info.where(book_id: book.id).find_in_batches(batch_size: 200) do |info_group|
      ...
    end

上面的寫法有看到傳到 do block 裡面的是 info_group 嗎？沒錯！他就是一個 200 筆資料的 array。接下來我們就可以對這個 array 進行操作囉！

    Info.where(book_id: book.id).find_in_batches(batch_size: 200) do |info_group|
      infos = info_group.map { |info| "(#{info.published_date}, '#{info.author}')" }.join(",")
    end

這一段做了什麼呢？我們將原本的 info_group array 裡面的每一筆資料用 map 抓出來，然後整理成 (val1, val2) 的形式，最後再用 join 將整個處理過的資料合併成一條 string，所以資料會如下：

    "(Sun, 22 Apr 2018, 'alan yeh'),(Sun, 22 Apr 2017, 'helen chuang')"

## 最後的最後，總要來個大集合

到這裡我們就完成 VALUES 的部分了，接下來讓我們繼續完成 SQL 語句的部分

    Info.where(book_id: book.id).find_in_batches(batch_size: 200) do |info_group|
      infos = info_group.map { |info| "(#{info.published_date}, '#{info.author}')" }.join(",")

      sql = <<-SQL
        INSERT INTO metas (published_date, author)
        VALUES #{infos};
      SQL
      ActiveRecord::Base.connection.execute(sql)
    end

好加在 Rails 提供了我們很方便的方法寫原生的 SQL 語句，然後最後執行就完成啦！是不是很簡單呢～～～～

以我這次的經驗，在本機上跑從一分鐘降成不到 10 秒呢 😊 整個屌打原本的寫法～

## 這些很重要的雷，一定要注意一下！

要注意的就是：

* 這樣直接用 SQL 語句執行的話並不會讓資料做 validation，以我這個例子來看的話，原本存在資料庫就是沒有問題的資料，因此跳過 validation 也沒關係。

* 如果原本的欄位有些值是 nil 的話，可以就要特別處理，需要處理成 null。

* 如果資料屬性是 string 的話，記得要使用單引號刮起來，如範例中那樣～

## 感謝大家收看囉 😊
