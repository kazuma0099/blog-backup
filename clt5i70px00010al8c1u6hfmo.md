---
title: "Unit test pada spring boot kontroller"
datePublished: Wed Feb 28 2024 07:57:05 GMT+0000 (Coordinated Universal Time)
cuid: clt5i70px00010al8c1u6hfmo
slug: unit-test-pada-spring-boot-kontroller
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/EWLHA4T-mso/upload/4ea54ec8b1b5109f58ac566292d417a8.jpeg
tags: unit-testing, java, controllers, springboot

---

Sebelum bahas lebih jauh tentang unit test pada kontroler pada Springboot lebih enak kalau bahas apa itu kontroler, secara umum kontroler adalah bagian dari aplikasi yang menangani http routing pada aplikasi untuk menerima dan mengirim response pada sebuah permintaan. Pada Spring boot biasanya sebuah class yang merupakan kontroler akan memiliki anotasi `@RestController`

Anggaplah kita memiliki sebuah kontroler question yang akan mengembalikan semua list pertanyaan yang mana pada kontroler tersebut memiliki sebuah service question. Berikut adalah contoh classnya

```java
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping(path = "/questions")
public class QuestionController {
  @Autowired
  QuestionService questionService;

  @GetMapping(produces = "application/json")
  public List<Question> fecthAllQuestions(){
    return questionService.getAll();
  }

}
```

Bagaimana sebaiknya unit test yang tepat untuk class kontroler ini? Umunya untuk melakukan unit test untuk kontroler adalah dengan melakukan simulasi seolah-olah ada request dari Internet yang melakukan request get pada routing `/questions` yang kemudian aplikasi akan mengirim response list object question. Bearti apakah perlu untuk mengirim request dari brower?, tentu tidak, ide ini bisa disimpan ketika membutuhkan E2E test.

Pada Springboot terdapat class `MockMvc` dan `WebMvcTest` yang digunakan untuk menguji dan mensimulasikan perilaku aplikasi tanpa perlu mendeploy aplikasi ke server. dan memungkinkan untuk mengirim permintaan HTTP ke aplikasi tanpa browser dan memeriksa respons yang dihasilkan, serta melakukan verifikasi perilaku kontroler yang dibuat.

Mari kita mulai langkah demi langkah untuk membuat unit testnya, dimulai dari line kode dibawah sebagai dasar pembuatan unit testnya

```java
@WebMvcTest(QuestionController.class)
class QuestionControllerTest {

	@Mock
  QuestionService questionService;

  @Autowired
  private MockMvc mockMvc;

  @Test
  void fetchShouldReturnAllQuestionsList() throws Exception {
    this.mockMvc.perform(get("/questions"))
        .andExpect(status().isOk());
  }
}
```

Pada kode diatas `@WebMvcTest(QuestionController.class)` Anotasi ini digunakan untuk pengujian fungsionalitas pada kontroler tanpa harus menginisiai seluruh konteks aplikasi, sehingga pengujian menjadi lebih fokus dan cepat, simplenya class ini digunakan untuk menjalankan class controller kita, dan `mockMvc` anggaplah seperti service yang akan digunakan untuk mengirim request ke dummy controller yang sudah dibuat. Pada method test `fetchShouldReturnAllQuestionsList` kita membuat unit test `mockMvc` untuk mengirim request `GET` ke kontroller yang sudah dibuat sebelumnya yang mana berhapakan mendapatakn response OK (`HTTP status code 200`). Test diatas masih kurang komprehensif karena hanya melakukan checking pada http status code tapi tidak dengan response bodynya.

Pada case sperti ini akan lebih simple ketika kita memiliki sample response dalam file berupa `.json` dibanding membuat object pada unit testnya disamping lebih mudah, maintenancenya pun akan lebih simple ketika jumlah unit testnya terus bertambah. Buatlah sebuah file berupa `sampleQuestionResponse.json` pada `test/resources/fixtures/question/`

```json
[
  {
    "id": "65d8d8e73b429a2b1dacbb37",
    "type": "SINGLE",
    "question": "1 + 1 =",
    "options": [
      {
        "a": "0"
      },
      {
        "b": "1"
      },
      {
        "c": "2"
      },
      {
        "d": "3"
      }
    ]
  }
]
```

kemudian unit test sebelumnya bisa diupdate degan menambahkan `resource` path dan `ObjectMapper` untuk melakukan mapping dari json menjadi object.

```java
@WebMvcTest(QuestionController.class)
class QuestionControllerTest {

  @Autowired
  MockMvc mockMvc;

  @MockBean
  private QuestionService questionService;

  @Value("classpath:fixtures/question/sampleQuestionResponse.json")
  private Resource responseSample;

  ObjectMapper mapper = new ObjectMapper();

  @Test
  void fetchShouldReturnAllQuestionsList() throws Exception {
    mockMvc.perform(get("/questions"))
        .andExpect(status().isOk()).andExpect(content().json("[]"));
  }
}
```

untuk implementasinya diperlukan untuk melakukan load json fixture tersebut, dan menampungnya pada sebuah variable, tidak lupa untuk melakukan mock pada `questionService` dengan mengconvert json yang sudah dibaca menjadi object karena jika tidak di mock, secara default service tersebut akan akan megembalikan list kosong.

```java
@Test
void fetchShouldReturnAllQuestionsList() throws Exception {
    String sampleResponse = responseSample.getContentAsString(StandardCharsets.UTF_8);
    TypeFactory typeFactory = mapper.getTypeFactory();
    List<Question> items = mapper.readValue(sampleResponse, typeFactory.constructCollectionType(List.class, Question.class));
    when(questionService.getAll()).thenReturn(items);

    mockMvc.perform(get("/questions"))
        .andExpect(status().isOk()).andExpect(content().json(sampleResponse));
 }
```