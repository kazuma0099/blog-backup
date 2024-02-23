---
title: "Java - LocalDate dan LocalDateTime"
seoTitle: "LocalDateTime"
seoDescription: "LocalDateTime Pada Java"
datePublished: Fri Feb 23 2024 14:10:34 GMT+0000 (Coordinated Universal Time)
cuid: clsyqc287000609jje3mz1jot
slug: java-localdate-dan-localdatetime
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/oWUx0ON3EVc/upload/4192c01c988b0073eb169c1e0579b9b3.jpeg
tags: java

---

Pada Java untuk membuat tipe data berformat tanggal dan waktu bisa menggunakan package `java.time` yang mana tipe data ini mulai diperkenalan pada java versi 8 yang mana untuk format tanggal dan waktu ini mengikuti standar sistem kalender `ISO-8601`. Umumnya pada pakcage `java.time` ada 2 jenis package yang sering digunakan yaitu

`java.time.LocalDate` seperti namanya `LocalDate` hanya dapat digunakan untuk menyimpan data berupa tanggal (Tahun, bulan dan hari) (yyyy-mm-dd) dan harus diingat tipe data ini sifatnya immutable.

berikut adalah contoh penggunaannya.

```java
// untuk membuat date format
LocalDate date = LocalDate.of(2007, 12, 3);
// untuk menampikan tanggal sekarang
LocalDate now = LocalDate.now();

// membandingkan
LocalDate date1 = LocalDate.of(2007, 12, 3);
LocalDate date2 = LocalDate.of(2007, 12, 4);

date1.isBefore(date2); // => true
date1.isAfter(date2); // => false

//instance ini juga memiliki setter getternya sendiri
now.getDayOfMoth(); // => mengembalikan tanggal
now.addDays(3); // => menambahkan hari ini + 3
```

`java.time.LocalDateTime` seperti namanya LocalDateTime ini seperti Tanggal yang membedakannya tipe data ini dapat menyimpan waktu dari jam, menit hingga detik.

Berikut adalah contoh penggunnaannya

```java
// membuat instance
LocalDateTime dateTime = LocalDateTime.of(2024, 1, 14, 20, 10, 01);
// untuk menampikan tanggal dan waktu sekarang
LocalDateTime now = LocalDateTime.now();
// konversi LocalDate ke LocalDateTime
LocalDate datenow = LocalDate.of(2024, 01, 14);
LocalDateTime dateTimeToday = datenow.at(10, 10, 10);
dateTimeToday.toString() // => 2024-01-14T10:10:10
```

### **Melakukan formatting pada kedua instance**

Untuk format date time juga dapat dilakukan pembuatan instance dari dan ke string.

```java
LocalDateTime datetime = LocalDateTime.of(2024, 01, 14, 10, 0, 0);
LocalDateTime parsed = LocalDateTime.parse("2024-01-14T10:00:00");

dateTime.equal(parsed); // => true

// dibuat dengan formatter
DateTimeFormatter parser = DateTimeFormatter.ofPattern("dd/MM/yyyy");
LocalDate date = LocalDate.parse("14-01-2024", parser);

DateFormatter printer = DateTimeFormatter.ofPattern("MMMM d, yyyy");
printer.format(date); // => "January 14, 2024

//bisa juga dilakukan degan custom
LocalDate now = LocalDate.now();
DateFormatter printer = DateTimeFormatter.ofPattern("'sekarang adalah hari 'EEEE d, yyyy");

printer.format(now);
```