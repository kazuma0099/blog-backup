---
title: "Mengexclude Code yang digenerate oleh Lombok pada coverage report"
datePublished: Wed Mar 20 2024 04:10:24 GMT+0000 (Coordinated Universal Time)
cuid: cltzacebt000209l87wz95h71
slug: mengexclude-code-yang-digenerate-oleh-lombok-pada-coverage-report
tags: unit-testing, java, code-coverage, lombok

---

ketika memprogram dengan `Java` mungkin tiidak asing lagi dengan `Project Lombok` , yang mana merupakan salah satu library yang digunakan untuk meminimize boilerplate code pada `POJO` kususnya method-method `setter`, `getter`, `equals`, `hashcode`, dan antek - anteknya.

Anggaplah terdapat sebuah `POJO` class [`Product.java`](http://Product.java)

```java
@Entity
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class Product {

  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private int id;
  private String name;
  private int price;
  private int quantity;

  private String imageLocation;

  public void updateProduct(Product updatedProduct) {
    this.name = updatedProduct.name;
    this.price = updatedProduct.price;
    this.quantity = updatedProduct.quantity;
  }

  public ProductDTO convertToDTO() {
    return ProductDTO.builder()
        .id(this.id)
        .name(this.name)
        .price(this.price)
        .quantity(this.quantity)
        .imageUrl(this.imageLocation)
        .build();
  }
}
```

untuk dua method di atas `updateProduct` dan `convertToDTO` sudah dibuat unit testnya namun ketika cek coverage reportnya, cukup mengesankan.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710823224508/238d2d5e-1234-4592-8a3b-f39f9ec7809e.png align="center")

Coveragenya tidak sampai `50%`, untuk mengatasi problem coverage ini ada beberapa cara yang bisa dilakukan

1. Mengexclude class model tersebut dari coverage report dari project
    
2. Mengexclude generated code dari `Lombok`
    

Yang nomor 1 kurang disarankan karena kadang class model juga memiliki bisnis prosessnya sendiri, tidak hanya `setter`, `getter`, dan antek-anteknya saja.

Pada Lombok versi `1.14` diperkenalkan fitur lombok configuration system yang mana developer dapat mengkonfigurasi fitur lombok pada proyek atau workspace, biasanya dilakukan dengan menambahkan `lombok.config` pada root project, untuk apa saja yang bisa dikonfigurasi bisa dilihat di official Lombok projectnya [https://projectlombok.org/features/configuration](https://projectlombok.org/features/configuration).

Dari semua pilihan konfigurasi pada official websitenya, ada satu configurasi yang membatu untuk mengexclude generated code yaitu `lombok.addLombokGeneratedAnnotation = true` , dengan menambahakan configurasi ini semua code yang digenerate oleh Lombok akan memiliki annotation `@Generated` , dengan adanya annotation ini `Jacoco` akan mengabaikan code terkait pada coverage.

```java
@Generated
public int hashCode() {
    int PRIME = true;
    int result = 1;
    result = result * 59 + this.getId();
    result = result * 59 + this.getPrice();
    result = result * 59 + this.getQuantity();
    Object $name = this.getName();
    result = result * 59 + ($name == null ? 43 : $name.hashCode());
    Object $imageLocation = this.getImageLocation();
    result = result * 59 + ($imageLocation == null ? 43 : $imageLocation.hashCode());
    return result;
 }
```

tanpa configurasi `lombok.addLombokGeneratedAnnotation = true`

```java
public int hashCode() {
    int PRIME = true;
    int result = 1;
    result = result * 59 + this.getId();
    result = result * 59 + this.getPrice();
    result = result * 59 + this.getQuantity();
    Object $name = this.getName();
    result = result * 59 + ($name == null ? 43 : $name.hashCode());
    Object $imageLocation = this.getImageLocation();
    result = result * 59 + ($imageLocation == null ? 43 : $imageLocation.hashCode());
    return result;
}
```

Coverage result pada [`Product.java`](http://Product.java) setelah menambahkan `lombok.addLombokGeneratedAnnotation = true` pada `lombok.config`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1710823209609/63b42cb1-3f4d-4285-8183-c63263ee76b2.png align="center")