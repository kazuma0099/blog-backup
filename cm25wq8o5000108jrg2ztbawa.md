---
title: "Database partisi"
seoTitle: "Database Partition"
datePublished: Sat Oct 12 2024 08:41:17 GMT+0000 (Coordinated Universal Time)
cuid: cm25wq8o5000108jrg2ztbawa
slug: database-partisi
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/lRoX0shwjUQ/upload/346c1aa9b5bb81a778bc72d7a0548d7c.jpeg

---

Database Partisi adalah proses pemecahan data pada table di database menjadi bagian - bagian table yang lebih kecil. Partisi data dapat dilakukan dengan menggunakan dua cara

* Partisi Vertikal
    
* Partisi Horizontal
    

Partisi table dapat dilakukan dengan beberapa pendekatan

* Partisi by range (contohnya berdasarkan tanggal atau id)
    
* Partisi by List (umutnya digunakan unutk nilai diskrit)
    
* Partisi by hash (menggunakan nilai hash namun hashingnya harus konsisten)
    

anggaplah terdapat sebuah table bernama student pada sebuah database yang berisi 10.000.000 baris yang strukturnya sebagai berikut

| Nama Kolom | Tipe data | Primary Key | Keterangan |
| --- | --- | --- | --- |
| id | number | Yes | id pelajar |
| name | text | No | nama pelajar |
| grade | int | No | nilai pelajar (0 - 100) |

```sql
insert into students(name, grade)  
select  
    random_string(10) as name,  
    floor(random() * 100) as grade  
from  
    generate_series(0, 10000000);
```

kemudian akan dilaukan partisi berdasarkan nilai pelajar, yang mana partisi ini dibagi menjadi 5 dengan kelipatan nilai 20

| Nilai | partisi |
| --- | --- |
| 1 - 20 | Partisi 1 |
| 21 - 40 | Partisi 2 |
| 41 - 60 | Partisi 3 |
| 61 - 80 | Partisi 4 |
| 81 - 100 | Partisi 5 |

untuk membuat table dengan partisi, sama halnya dengan pembuatan table pada umunya yang membedakan adalah penambahan keyword `partition by`.

```sql
CREATE TABLE students_grade_partitions  
(  
    id    serial,  
    name  text,  
    grade int not null  
) partition by range (grade);
```

perlu diingat jika menambahkan primary key maka kolom grade harus ditambahkan sebagai komposite. jika tidak maka akan menghasilkan error

```sql
[0A000] ERROR: unique constraint on partitioned table must include all partitioning columns Detail: PRIMARY KEY constraint on table "students_grade_partitions" lacks column "grade" which is part of the partition key.
```

Setelah table utama dibuat, proses selanjutnya pembuatan table partisi sebanyak total paritsi yang diinginkan

```sql
CREATE TABLE students_grade0020 (like students_grade_partitions including indexes );  
CREATE TABLE students_grade2140 (like students_grade_partitions including indexes );  
CREATE TABLE students_grade4160 (like students_grade_partitions including indexes );  
CREATE TABLE students_grade6180 (like students_grade_partitions including indexes );  
CREATE TABLE students_grade81100 (like students_grade_partitions including indexes);
```

command `including indexes` ini sangat penting ketika membuat paritisi karena partisi bergantung dengan index, jadi jika ada perubahan index pada tabel utama akan berpengaruh juga pada tabel partisi. Table partisi yang telah dibuat perlu di assign ke table utama. Perlu diingat untuk nilai `to` sifatnya adalah exclusive

```sql
ALTER TABLE students_grade_partitions attach PARTITION students_grade0020 FOR VALUES FROM (0) TO (21);  
ALTER TABLE students_grade_partitions attach PARTITION students_grade2140 FOR VALUES FROM (21) TO (41);  
ALTER TABLE students_grade_partitions attach PARTITION students_grade4160 FOR VALUES FROM (41) TO (61);  
ALTER TABLE students_grade_partitions attach PARTITION students_grade6180 FOR VALUES FROM (61) TO (81);  
ALTER TABLE students_grade_partitions attach PARTITION students_grade81100 FOR VALUES FROM (81) TO (100);
```

Jika dilihat lihat secara visual table partisi tersebut akan menjadi bagian dari table utama

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728722434498/6357ace4-52fe-47fc-ba3f-e2ed3f89a67d.png align="left")

## Partisi vs non partisi

untuk hasil perbandingan query table dengan partisi dan non partisi adalah sebagai berikut.

### Query range

```sql
explain analyze select * from students_grade_partitions where grade > 40 and grade < 61;

/* OUTPUT
Seq Scan on students_grade4160 students_grade_partitions  (cost=0.00..41441.50 rows=1803866 width=14) (actual time=0.133..140.940 rows=1802569 loops=1)  
  Filter: ((grade > 41) AND (grade < 60))  
  Rows Removed by Filter: 199798  
Planning Time: 0.302 ms  
Execution Time: 186.098 ms
*/

explain analyze select * from students where grade > 41 and grade < 60;
/* OUTPUT
Bitmap Heap Scan on students  (cost=24437.02..108267.03 rows=1791667 width=14) (actual time=70.474..524.556 rows=1802569 loops=1)  
  Recheck Cond: ((grade > 41) AND (grade < 60))  
  Heap Blocks: exact=56955  
  ->  Bitmap Index Scan on students_grade  (cost=0.00..23989.11 rows=1791667 width=0) (actual time=63.083..63.083 rows=1802569 loops=1)  
        Index Cond: ((grade > 41) AND (grade < 60))  
Planning Time: 0.638 ms  
JIT:  
  Functions: 2  
"  Options: Inlining false, Optimization false, Expressions true, Deforming true"  
"  Timing: Generation 1.441 ms, Inlining 0.000 ms, Optimization 0.000 ms, Emission 0.000 ms, Total 1.441 ms"  
Execution Time: 574.170 ms
*/
```

Perbandingan eksekusi waktu diatas cukup jelas data yang dipartisi menghasilkan eksekusi lebih cepat karena query rangenya `grade > 40 and grade < 61`, Postgress hanya perlu melakukan scaning index pada satu partisi `students_grade4160` dan tidak mengalami overhead dari membangun bitmap. Sedangkan query kedua karena data yang di index cukup besar maka akan dibuat bitmap yang menyebabkan query jadi lambat.

Apa yang terjadi jika kita menscan seluruh partisi dengan menggunakan query untuk grade yang lebih dari `5`?

```sql
explain analyze select * from students_grade_partitions where grade > 7;
/* OUTPUT
Append  (cost=0.00..227967.39 rows=9202076 width=14) (actual time=9.562..1413.924 rows=9199930 loops=1)  
  ->  Seq Scan on students_grade0020 students_grade_partitions_1  (cost=0.00..38217.45 rows=1302431 width=14) (actual time=9.561..224.284 rows=1300285 loops=1)  
        Filter: (grade > 7)  
        Rows Removed by Filter: 800071  
  ->  Seq Scan on students_grade2140 students_grade_partitions_2  (cost=0.00..36408.10 rows=2000968 width=14) (actual time=0.196..211.621 rows=2000968 loops=1)  
        Filter: (grade > 7)  
  ->  Seq Scan on students_grade4160 students_grade_partitions_3  (cost=0.00..36435.59 rows=2002367 width=14) (actual time=0.149..196.123 rows=2002367 loops=1)  
        Filter: (grade > 7)  
  ->  Seq Scan on students_grade6180 students_grade_partitions_4  (cost=0.00..36358.74 rows=1998219 width=14) (actual time=0.110..179.299 rows=1998219 loops=1)  
        Filter: (grade > 7)  
  ->  Seq Scan on students_grade81100 students_grade_partitions_5  (cost=0.00..34537.14 rows=1898091 width=14) (actual time=0.105..170.764 rows=1898091 loops=1)  
        Filter: (grade > 7)  
Planning Time: 6.056 ms  
JIT:  
  Functions: 10  
"  Options: Inlining false, Optimization false, Expressions true, Deforming true"  
"  Timing: Generation 1.445 ms, Inlining 0.000 ms, Optimization 1.203 ms, Emission 8.386 ms, Total 11.034 ms"  
Execution Time: 1634.274 ms
*/

explain analyze select * from students where grade > 7;
/* OUTPUT
Seq Scan on students  (cost=0.00..181955.01 rows=9201001 width=14) (actual time=4.328..845.234 rows=9199930 loops=1)  
  Filter: (grade > 7)  
  Rows Removed by Filter: 800071  
Planning Time: 0.280 ms  
JIT:  
  Functions: 2  
"  Options: Inlining false, Optimization false, Expressions true, Deforming true"  
"  Timing: Generation 0.538 ms, Inlining 0.000 ms, Optimization 0.253 ms, Emission 3.852 ms, Total 4.642 ms"  
Execution Time: 1062.563 ms
*/
```

Kita bisa lihat sekarang query kedua lebih cepat tanpa partisi, hal ini dikarenakan pada query tersebut ketika dilakukan proses pencarian keseluruh partisi akan ada prosess `append` setelah dilakukan pencarian per partisi, sedangkan jika tanpa partisi hal yang perlu dilakukan hanya squential scan index saja tidak perlu ada proses append karena tidak terpartisi hal ini menyebabkan query kedua lebih cepat.

### Ukuran data

Apakah dengan menggunakan partisi memori akan menjadi lebih kecil sebenarnya tidak terlalu signifikan

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1728705543991/d17b85c5-e8c6-4ae7-8045-241b66ed4895.png align="left")

### Kesimpulan

secara umum manfaat partisi akan berasa jika pencarian hanya dilakukan pada satu partisi saja akan terasa sekali manfaatnya karena pencarian menjadi lebih kecil apalgi jika kolom pencarian sudah di indexing. Perlu di ingat `enable_partition_pruning` harus selalu dalam kondisi on, jika tidak maka percuma dilakukan peartisi karena pencarian akan dilakukan disemua partisi yang menyebabkan query jadi lebih lama.