# MultipleBagFetchException

- JPA의 N + 1 문제의 해결책으로 `Fetch Join`을 사용하다 보면 마주치는 문제

<br>

## 발생하는 이유

`@OneToMany` 또는 `ManyToMany` 연관 관계인 자식 테이블에 2개 이상의 `Fetch Join`을 선언했을 때 발생한다.

```java
// MultipleBagFetchException 발생!!
SELECT m FROM Member m
JOIN FETCH m.orders // @OnetoMany
JOIN FETCH m.teams // @OnetoMany
WHERE m.name = 'hyoseok'
```

<br>

## 해결 방법

1. 자식 테이블을 모두 `Lazy Loading`으로 처리

2. 자식 테이블 하나에만 `Fetch Join`을 걸고, 나머진 `Lazy Loading`로 처리 (해당 예시는 생략했음)

```java
@Test
public void Basic_findAll_No_Use_Batch_Size() throws Exception {
    /*
        --- 총 5번의 쿼리 발생 ---
        1) Product 전체 쿼리
        2) Product 1번의 ProductDescription 자식들 조회 쿼리
        3) Product 2번의 ProductDescription 자식들 조회 쿼리
        3) Product 1번의 ProductImage 자식들 조회 쿼리
        3) Product 2번의 ProductImage 자식들 조회 쿼리
    */

    // given
    List<ProductImage> productImages1 = new ArrayList<>();
    productImages1.add(ProductImage.createProductImage("imageUrl1-1"));
    productImages1.add(ProductImage.createProductImage("imageUrl1-2"));

    List<ProductDescription> productDescriptions1 = new ArrayList<>();
    productDescriptions1.add(ProductDescription.createProductDescription("title1-1", "contents1-1"));
    productDescriptions1.add(ProductDescription.createProductDescription("title1-2", "contents1-2"));

    Product product1 = Product.createProduct("name", 20000, productImages1, productDescriptions1);

    this.productRepository.save(product1);

    List<ProductImage> productImages2 = new ArrayList<>();
    productImages2.add(ProductImage.createProductImage("imageUrl2-1"));
    productImages2.add(ProductImage.createProductImage("imageUrl2-2"));

    List<ProductDescription> productDescriptions2 = new ArrayList<>();
    productDescriptions2.add(ProductDescription.createProductDescription("title2-1", "contents2-1"));
    productDescriptions2.add(ProductDescription.createProductDescription("title2-2", "contents2-2"));

    Product product2 = Product.createProduct("name", 20000, productImages2, productDescriptions2);

    this.productRepository.save(product2);

    // when
    long totalPrice = this.productService.getTotalPriceByBasic();

    // then
    assertThat(totalPrice).isEqualTo(40000);
}
```

```sql
------- product -------
Hibernate:
    select -- Product 전체 쿼리
        product0_.product_id as product_1_1_,
        product0_.name as name2_1_,
        product0_.price as price3_1_
    from
        product product0_
------- description -------
Hibernate:
    select -- Product 1번의 ProductDescription 자식들 조회 쿼리
        productdes0_.product_id as product_4_2_0_,
        productdes0_.product_description_id as product_1_2_0_,
        productdes0_.product_description_id as product_1_2_1_,
        productdes0_.contents as contents2_2_1_,
        productdes0_.product_id as product_4_2_1_,
        productdes0_.title as title3_2_1_
    from
        product_description productdes0_
    where
        productdes0_.product_id=?
Hibernate:
    select -- Product 2번의 ProductDescription 자식들 조회 쿼리
        productdes0_.product_id as product_4_2_0_,
        productdes0_.product_description_id as product_1_2_0_,
        productdes0_.product_description_id as product_1_2_1_,
        productdes0_.contents as contents2_2_1_,
        productdes0_.product_id as product_4_2_1_,
        productdes0_.title as title3_2_1_
    from
        product_description productdes0_
    where
        productdes0_.product_id=?
------- image -------
Hibernate:
    select -- Product 1번의 ProductImage 자식들 조회 쿼리
        productima0_.product_id as product_3_3_0_,
        productima0_.product_image_id as product_1_3_0_,
        productima0_.product_image_id as product_1_3_1_,
        productima0_.image_url as image_ur2_3_1_,
        productima0_.product_id as product_3_3_1_
    from
        product_image productima0_
    where
        productima0_.product_id=?
Hibernate:
    select -- Product 2번의 ProductImage 자식들 조회 쿼리
        productima0_.product_id as product_3_3_0_,
        productima0_.product_image_id as product_1_3_0_,
        productima0_.product_image_id as product_1_3_1_,
        productima0_.image_url as image_ur2_3_1_,
        productima0_.product_id as product_3_3_1_
    from
        product_image productima0_
    where
        productima0_.product_id=?
```

총 5번의 쿼리가 발생되었다. 이 방법보다 조금 더 성능적인 이슈를 해결할 수 있는 방법이 있다.

<br>

## 성능을 고려해서 해결하는 방법

**`Hibernate default_batch_fetch_size`** 옵션을 적용해서 해결하면 된다.

`application.yml`, `application.properties`에 해당 값을 정의하면 되고, 최대 `1000`까지만 옵션을 지정한다.

- `application.yml`

```yml
spring:
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 1000
```

- `application.properties`

```
spring.jpa.properties.hibernate.default_batch_fetch_size=1000
```

```java
@Test
public void Basic_findAll_Use_Batch_Size() throws Exception {
    /*
        --- 총 3번의 쿼리 발생 ---
        1) Product 전체 쿼리
        2) Product 1, 2번의 ProductDescription 자식들 조회 쿼리 -> In (1, 2)
        3) Product 1, 2번의 ProductImage 자식들 조회 쿼리 -> In (1, 2)
    */

    // given
    List<ProductImage> productImages1 = new ArrayList<>();
    productImages1.add(ProductImage.createProductImage("imageUrl1-1"));
    productImages1.add(ProductImage.createProductImage("imageUrl1-2"));

    List<ProductDescription> productDescriptions1 = new ArrayList<>();
    productDescriptions1.add(ProductDescription.createProductDescription("title1-1", "contents1-1"));
    productDescriptions1.add(ProductDescription.createProductDescription("title1-2", "contents1-2"));

    Product product1 = Product.createProduct("name", 20000, productImages1, productDescriptions1);

    this.productRepository.save(product1);

    List<ProductImage> productImages2 = new ArrayList<>();
    productImages2.add(ProductImage.createProductImage("imageUrl2-1"));
    productImages2.add(ProductImage.createProductImage("imageUrl2-2"));

    List<ProductDescription> productDescriptions2 = new ArrayList<>();
    productDescriptions2.add(ProductDescription.createProductDescription("title2-1", "contents2-1"));
    productDescriptions2.add(ProductDescription.createProductDescription("title2-2", "contents2-2"));

    Product product2 = Product.createProduct("name", 20000, productImages2, productDescriptions2);

    this.productRepository.save(product2);

    // when
    long totalPrice = this.productService.getTotalPriceByBasic();

    // then
    assertThat(totalPrice).isEqualTo(40000);
}
```

```sql
------- product -------
Hibernate:
    /* SELECT generatedAlias0 FROM Product AS generatedAlias0 */
    select -- Product 전체 쿼리
        product0_.product_id as product_1_1_,
        product0_.name as name2_1_,
        product0_.price as price3_1_
    from
        product product0_
------- description -------
Hibernate:
    /* load one-to-many com.example.jpa20200918.domain.product.Product.productDescriptions */
    select -- Product 1, 2번의 ProductDescription 자식들 조회 쿼리 -> In (1, 2)
        productdes0_.product_id as product_4_2_1_,
        productdes0_.product_description_id as product_1_2_1_,
        productdes0_.product_description_id as product_1_2_0_,
        productdes0_.contents as contents2_2_0_,
        productdes0_.product_id as product_4_2_0_,
        productdes0_.title as title3_2_0_
    from
        product_description productdes0_
    where
        productdes0_.product_id in (
            ?, ?
        )
------- image -------
Hibernate:
    /* load one-to-many com.example.jpa20200918.domain.product.Product.productImages */
    select -- Product 1, 2번의 ProductImage 자식들 조회 쿼리 -> In (1, 2)
        productima0_.product_id as product_3_3_1_,
        productima0_.product_image_id as product_1_3_1_,
        productima0_.product_image_id as product_1_3_0_,
        productima0_.image_url as image_ur2_3_0_,
        productima0_.product_id as product_3_3_0_
    from
        product_image productima0_
    where
        productima0_.product_id in (
            ?, ?
        )
```

기존에는 `5번 쿼리`가 발생되었지만, `default_batch_fetch_size`을 적용한 이후로, `3번의 쿼리`를 발생시킴으로써 어느정도 성능을 개선시켰다.

<br>

## 별로 큰 차이가 없는거 아닌가?

`Product`의 결과가 많으면 많을수록 쿼리 수행 횟수가 획기적으로 개선할 수 있다. 만약에 `20,000`개의 `Product`를 조회한다면, 다음과 같은 상황으로 나눠 볼 수 있다.

- `default_batch_fetch_size` 옵션 미적용 --> (**총 40,001번의 쿼리 수행**)

  - Product 조회 쿼리 `1`번

  - Product의 ProductDescription 자식들 조회 쿼리가 `20,000`번

  - Product의 ProductImage 자식들 조회 쿼리가 `20,000`번

- `default_batch_fetch_size = 1000` 옵션 적용 --> (**총 41번의 쿼리 수행**)

  - Product 조회 쿼리 `1`번

  - Product의 ProductDescription 자식들 조회 쿼리가 `20`번 (`20,000 / 1,000`)

  - Product의 ProductImage 자식들 조회 쿼리가 `20`번 (`20,000 / 1,000`)

<br>

> Tip)
> <br>
> 보통 옵션값을 1,000 이상 주지는 않는다.
> <br>
> 만약에 in절 파라미터로 1,000 개 이상을 주었을때,
> <br>
> 너무 많은 in절 파라미터로 인해 문제가 발생할수도 있기 때문...

<br>

## 조금만 더 성능을 최적화 해보자!!!

기존에 `default_batch_fetch_size` 옵션이 적용된 상태에서 `Fetch Join`을 적용해보자

```java
@Test
public void Jpql_findAll_Use_Batch_Size() throws Exception {
    /*
        SELECT DISTINCT p FROM Product p JOIN FETCH p.productImages

        --- 총 2번의 쿼리 발생 ---

        1) Product, ProductImage Fetch 조인 쿼리
        2) Product 1, 2번의 ProductDescription 자식들 조회 쿼리 -> In (1, 2)
    */

    // given
    List<ProductImage> productImages1 = new ArrayList<>();
    productImages1.add(ProductImage.createProductImage("imageUrl1-1"));
    productImages1.add(ProductImage.createProductImage("imageUrl1-2"));

    List<ProductDescription> productDescriptions1 = new ArrayList<>();
    productDescriptions1.add(ProductDescription.createProductDescription("title1-1", "contents1-1"));
    productDescriptions1.add(ProductDescription.createProductDescription("title1-2", "contents1-2"));

    Product product1 = Product.createProduct("name", 20000, productImages1, productDescriptions1);

    this.productRepository.save(product1);

    List<ProductImage> productImages2 = new ArrayList<>();
    productImages2.add(ProductImage.createProductImage("imageUrl2-1"));
    productImages2.add(ProductImage.createProductImage("imageUrl2-2"));

    List<ProductDescription> productDescriptions2 = new ArrayList<>();
    productDescriptions2.add(ProductDescription.createProductDescription("title2-1", "contents2-1"));
    productDescriptions2.add(ProductDescription.createProductDescription("title2-2", "contents2-2"));

    Product product2 = Product.createProduct("name", 20000, productImages2, productDescriptions2);

    this.productRepository.save(product2);

    // when
    long totalPrice = this.productService.getTotalPriceByJpql();

    // then
    assertThat(totalPrice).isEqualTo(40000);
}
```

```sql
------- product fetch join image -------
Hibernate:
    /* SELECT DISTINCT p FROM Product p JOIN FETCH p.productImages */
    select -- Product, ProductImage Fetch 조인 쿼리
        distinct product0_.product_id as product_1_1_0_,
        productima1_.product_image_id as product_1_3_1_,
        product0_.name as name2_1_0_,
        product0_.price as price3_1_0_,
        productima1_.image_url as image_ur2_3_1_,
        productima1_.product_id as product_3_3_1_,
        productima1_.product_id as product_3_3_0__,
        productima1_.product_image_id as product_1_3_0__
    from
        product product0_
    inner join
        product_image productima1_
            on product0_.product_id=productima1_.product_id
------- description -------
Hibernate:
    /* load one-to-many com.example.jpa20200918.domain.product.Product.productDescriptions */
    select -- Product 1, 2번의 ProductDescription 자식들 조회 쿼리 -> In (1, 2)
        productdes0_.product_id as product_4_2_1_,
        productdes0_.product_description_id as product_1_2_1_,
        productdes0_.product_description_id as product_1_2_0_,
        productdes0_.contents as contents2_2_0_,
        productdes0_.product_id as product_4_2_0_,
        productdes0_.title as title3_2_0_
    from
        product_description productdes0_
    where
        productdes0_.product_id in (
            ?, ?
        )
```

다음과 같이 기존에 `default_batch_fetch_size` 옵션이 적용된 상태에서 `Fetch Join`을 적용해보면, 3번의 쿼리 수행에서 2번의 쿼리 수행으로 성능이 개선되었다.

<br>

## 정리

`Hibernate default_batch_fetch_size` 옵션으로 인해 **최소한의 성능이 보장**된 것일 뿐, **최선은 아니다.** 따라서 가장 먼저 `Fetch Join`을 적용해 최대한 성능을 개선하고, `Fetch Join`으로 해결되지 않는 문제들은 `Hibernate default_batch_fetch_size` 적용해서 최소한의 성능을 보장해주자.

<br>

## 결론

- `@OneToOne`, `@ManyToOne`의 관계에서는 모두 `Fetch Join`을 적용하여, 한방 쿼리를 수행한다.

- `@OneToMany`, `@ManyToMany`의 관계에서는 가장 데이터가 많은 자식 엔티티에 `Fetch Join`을 적용하여, 한방 쿼리를 수행한다.

  - `Fetch Join`을 적용할 수 없는 나머지 자식 엔티티들은 `default_batch_fetch_size` 옵션을 통해 `in` 쿼리로 최소한의 성능을 보장한다.

<br>

## 참고

- [인프런 - 자바 ORM 표준 JPA 프로그래밍 (기본편)](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)

- [MultipleBagFetchException 발생시 해결 방법](https://jojoldu.tistory.com/457)
