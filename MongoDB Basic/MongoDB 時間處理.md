## MongoDB 時間處理

### 0. 在 MongoDB 建立時間的三種方式:
You can create a new Date object in two different ways:

``new Date()``: Returns a date and time as a **Date object**.
``ISODate()``: Returns a date and time as a **Date object**.
Both the new Date() and ISODate() methods produce a Date object that is wrapped in an ISODate() helper function.

Additionally, calling Date() function without the new constructor returns a date and time as as string instead of a Date object:

``Date()``: Returns a date and time as a **string**, e.g. Fri Feb 10 03:47:37 UTC 2023

### 1. 直接從 MongoDB 加入時間
```mongodb
use test;
db.createCollection("SALON_SALE");

db.SALON_SALE.insertMany([
    {
        title: '時間不使用 Z，輸入 2023-12-25T23:00:00',
        saleDate: new Date("2023-12-25T23:00:00"),
        ISOsaleDate: ISODate("2023-12-25T23:00:00"),
        saleDateString: Date("2023-12-25T23:00:00")
    },
    {
        title: '時間使用 Z，輸入 2023-12-25T23:00:00Z',
        saleDate: new Date("2023-12-25T23:00:00Z"),
        ISOsaleDate: ISODate("2023-12-25T23:00:00Z"),
        saleDateString: Date("2023-12-25T23:00:00Z")
    },
    {
        title: '加入現在的時間 2023-02-12 11:47，時間變成 localtime -8',
        saleDate: new Date(),
        ISOsaleDate:  ISODate(),
        saleDateString: Date()
    },
])
```
在 MongoDB 中如果你是直接指定要哪個日期哪個時段，那麼它存入的時間就會依照你輸入多少便是多少。
但是，如果你沒有指定，而是直接用``new Date()`` 或 ``ISODate()`` 獲取當下時間，則MongoDB會自動把 Local時區 (台灣 UTC + 8) 轉成 UTC + 0  (因為 MongoDB 預設上就是存取 UTC + 0 的時間資料)，因此以台灣的時區來說，我們會發現存入時間少了 8 小時。

### 2. 從 Java 中獲取 MongoDB的時間資料
當我們從 Java 中獲取 MongoDB 的時間資料時，Java會根據"這台電腦的時區" **自動** 幫你把 MongoDB的時間資料加上時區，因此，你會看到以下情況:

在 MongoDB 中: ``2023-12-25 12:00:00``

在台灣時區 (UTC + 8) 之 Java 開發環境中:  ``2023-12-25 20:00:00``


### 3. Java 更改整體時區
```java
TimeZone.setDefault(TimeZone.getTimeZone("時區代稱"));
TimeZone.setDefault(TimeZone.getTimeZone("UTC"));
```


