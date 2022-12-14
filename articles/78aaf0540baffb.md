---
title: "Rの日時データ #2 - 文字列⇔日時オブジェクトの変換"
emoji: "🌎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics:
  - "r"
published: true
---

# はじめに

Rの日時データ[^1]に関する記事#2です。

今回は文字列⇔日時オブジェクト[^1]の変換手順についてです。

[^1]: 「日時データ」や「日時オブジェクト」といった用語は、野間口謙太郎, 菊池泰樹 訳 「統計学:Rを用いた入門書 第２版」共立出版 2016 pp.348-349 を参考にしています。


文字列と日時オブジェクト（POSIXct型のオブジェクト）の変換として、

- 変換方向（文字列→日時オブジェクト or 日時オブジェクト→文字列）
- 文字列に明示的にタイムゾーン情報があるか（あり([ISO8601形式](https://ja.wikipedia.org/wiki/ISO_8601)) or なし）

によって、以下の4つの手順に分けて説明します。

||タイムゾーン情報あり|タイムゾーン情報なし|
|---|---:|---:|
|**文字列→日時オブジェクト**|[1](#1.-%E6%96%87%E5%AD%97%E5%88%97%E2%86%92%E6%97%A5%E6%99%82%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%EF%BC%88%E3%82%BF%E3%82%A4%E3%83%A0%E3%82%BE%E3%83%BC%E3%83%B3%E6%83%85%E5%A0%B1%E3%81%82%E3%82%8A%EF%BC%89)|[2](#2.-%E6%96%87%E5%AD%97%E5%88%97%E2%86%92%E6%97%A5%E6%99%82%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%EF%BC%88%E3%82%BF%E3%82%A4%E3%83%A0%E3%82%BE%E3%83%BC%E3%83%B3%E6%83%85%E5%A0%B1%E3%81%AA%E3%81%97%EF%BC%89)|
|**日時オブジェクト→文字列**|[3](#3.-%E6%97%A5%E6%99%82%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E2%86%92%E6%96%87%E5%AD%97%E5%88%97%EF%BC%88%E3%82%BF%E3%82%A4%E3%83%A0%E3%82%BE%E3%83%BC%E3%83%B3%E6%83%85%E5%A0%B1%E3%81%82%E3%82%8A%EF%BC%89)|[4](#4.-%E6%97%A5%E6%99%82%E3%82%AA%E3%83%96%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E2%86%92%E6%96%87%E5%AD%97%E5%88%97%EF%BC%88%E3%82%BF%E3%82%A4%E3%83%A0%E3%82%BE%E3%83%BC%E3%83%B3%E6%83%85%E5%A0%B1%E3%81%AA%E3%81%97%EF%BC%89)|

# 1. 文字列→日時オブジェクト（タイムゾーン情報あり）

タイムゾーン情報が明示的に指定された文字列（[ISO8601形式](https://ja.wikipedia.org/wiki/ISO_8601)）を日時オブジェクト（POSIXct型のオブジェクト）に変換するには、[#1](https://zenn.dev/kn1kn1/articles/2259bcd0ae5f3f#2.-iso8601%E5%BD%A2%E5%BC%8F%E3%82%92%E3%83%91%E3%83%BC%E3%82%B9%E3%81%A7%E3%81%8D%E3%81%AA%E3%81%84)で紹介した`lubridate::ymd_hms`関数を使うと良いでしょう。

```
> lubridate::ymd_hms("2022-11-24T00:30:38+09:00")
[1] "2022-11-23 15:30:38 UTC"
> lubridate::ymd_hms("2022-11-24T00:30:38Z")
[1] "2022-11-24 00:30:38 UTC"
>
```

`lubridate::ymd_hms`関数を使用することで、ISO8601形式であればどのタイムゾーンの文字列であっても問題なく変換することができます。

# 2. 文字列→日時オブジェクト（タイムゾーン情報なし）

タイムゾーンの指定がない場合は、必ず仕様を確認し、`as.POSIXct`関数の`tz`引数を指定することが必要です。

```
> as.POSIXct("2011-03-20 12:34:56", tz="UTC")
[1] "2011-03-20 12:34:56 UTC"
> as.POSIXct("2011-03-20 12:34:56", tz="Asia/Tokyo")
[1] "2011-03-20 12:34:56 JST"
>
```

`tz`引数を指定しないと、[#1](https://zenn.dev/kn1kn1/articles/2259bcd0ae5f3f#1.-%E5%AE%9F%E8%A1%8C%E7%92%B0%E5%A2%83%E3%81%AB%E3%82%88%E3%82%8Aas.posixct%E3%81%AE%E7%B5%90%E6%9E%9C%E3%81%8C%E7%95%B0%E3%81%AA%E3%82%8B)で紹介したように、実行環境のシステムのタイムゾーンに依存した日時オブジェクトが作成されてしまいます。

```
> as.POSIXct("2011-03-20 12:34:56")
[1] "2011-03-20 12:34:56 JST"
> 
```

# 3. 日時オブジェクト→文字列（タイムゾーン情報あり）

日時オブジェクト（POSIXct型のオブジェクト）からタイムゾーン情報ありのISO8601形式にするには、以下の3つの方法があります。用途と出力形式が微妙に異なるので、状況に応じて使い分けると良いでしょう。

## `format`/`strftime`関数(`%z`記法)

baseライブラリの`format`関数や`strftime`関数を使う方法です。

`format`関数の`tz`引数にタイムゾーンを指定し、フォーマット文字列に`%z`を指定することで、タイムゾーンを示す文字列（`+0900`）を追加できます。但し、ここで追加される`+0900`はISO8601の基本形式と呼ばれるもの（`:`が付いていないもの）になっている点に注意が必要です。また、UTCの場合に`Z`でなく`+0000`になってしまう点にも注意が必要でしょう。

```
> d <- as.POSIXct("2011-03-20 12:34:56", tz="Asia/Tokyo")
> format(d, "%Y-%m-%dT%H:%M:%S%z", tz="Asia/Tokyo")
[1] "2011-03-20T12:34:56+0900"
> format(d, "%Y-%m-%dT%H:%M:%S%z", tz="UTC")
[1] "2011-03-20T03:34:56+0000"
>
```

## `format`/`strftime`関数(固定タイムゾーン文字列指定)

`+0900`や`+0000`でなく、`+09:00`や`Z`に変換したい場合には、あまり美しくないですが、フォーマット文字列に指定するしか無いようです。

```
> d <- as.POSIXct("2011-03-20 12:34:56", tz="Asia/Tokyo")
> format(d, "%Y-%m-%dT%H:%M:%S+09:00", tz="Asia/Tokyo")
[1] "2011-03-20T12:34:56+09:00"
> format(d, "%Y-%m-%dT%H:%M:%SZ", tz="UTC")
[1] "2011-03-20T03:34:56Z"
>
```

## `lubridate::format_ISO8601`関数

日時オブジェクト（POSIXct型のオブジェクト）で保持されたタイムゾーンに応じて出力されるタイムゾーンを変えたい場合は、`lubridate::format_ISO8601`関数を使うと良いでしょう。

```
> d <- as.POSIXct("2011-03-20 12:34:56", tz="Asia/Tokyo")
> format_ISO8601(d, usetz = TRUE)
[1] "2011-03-20T12:34:56+0900"
> d <- with_tz(d, tzone = "UTC")
> format_ISO8601(d, usetz = TRUE)
[1] "2011-03-20T03:34:56+0000"
> 
```

動作はフォーマット文字列に`%z`を使用した場合と同じで、`:`が付いてなく、`Z`に変換されない点に注意が必要です。


# 4. 日時オブジェクト→文字列（タイムゾーン情報なし）

"2011-03-20 12:34:56"のようなタイムゾーン情報なしの文字列に変換するには、baseライブラリの`format`関数や`strftime`関数を使えば可能です。

```
> d <- as.POSIXct("2011-03-20 12:34:56", tz="Asia/Tokyo")
> format(d, "%Y-%m-%d %H:%M:%S", tz="Asia/Tokyo")
[1] "2011-03-20 12:34:56"
> format(d, "%Y-%m-%d %H:%M:%S", tz="Europe/Paris")
[1] "2011-03-20 04:34:56"
> strftime(d, "%Y-%m-%d %H:%M:%S", tz="Asia/Tokyo")
[1] "2011-03-20 12:34:56"
> strftime(d, "%Y-%m-%d %H:%M:%S", tz="Europe/Paris")
[1] "2011-03-20 04:34:56"
> 
```

この場合も、`tz`引数は付けたほうがコードの意図が明確になって親切だと思います。

以下のように、`tz`引数無しで呼び出した場合、タイムゾーンに依存せず時刻の情報のみで文字列が生成されるようです。

```
> d <- as.POSIXct("2011-03-20 12:34:56", tz="Asia/Tokyo")
> format(d, "%Y-%m-%d %H:%M:%S")
[1] "2011-03-20 12:34:56"
> d <- as.POSIXct("2011-03-20 12:34:56", tz="UTC")
> format(d, "%Y-%m-%d %H:%M:%S")
[1] "2011-03-20 12:34:56"
> 
```

# まとめ

今回は、文字列⇔日時オブジェクトの変換について4つの場合に分けて紹介しました。

次回の#3は、csvファイルの読み書きと日時オブジェクト（POSIXct型のオブジェクト）について書きたいと思います。
