# Специфікація моделі даних: Онлайн-книгарня «Клуб читачів»

## 1. Намір
Спроектувати концептуальну/логічну ER-модель даних для книжкового інтернет-магазину та служби доставки замовлень. Модель описує сутності, атрибути, первинні та зовнішні ключі, а також кардинальності зв'язків. Результат подається виключно в синтаксисі Mermaid (`.mmd`), без фізичного SQL DDL та без ORM-коду.

## 2. Сутності та атрибути
1. **Customer (Покупець / Член клубу)**
   * `id`: UUID (PK)
   * `email`: string
   * `phone`: string
   * `full_name`: string
   * `club_member_status`: boolean
   * `created_at`: timestamp

2. **Author (Автор)**
   * `id`: UUID (PK)
   * `full_name`: string
   * `bio`: string

3. **Book (Книга / Видання)**
   * `id`: UUID (PK)
   * `isbn`: string
   * `title`: string
   * `publication_year`: integer
   * `price`: decimal
   * `stock_quantity`: integer

4. **Order (Замовлення)**
   * `id`: UUID (PK)
   * `customer_id`: UUID (FK)
   * `order_date`: timestamp
   * `status`: string (pending | confirmed | processing | shipped | delivered | cancelled)

5. **OrderItem (Позиція замовлення — асоціативна сутність)**
   * `id`: UUID (PK)
   * `order_id`: UUID (FK)
   * `book_id`: UUID (FK)
   * `quantity`: integer
   * `unit_price`: decimal (фіксує ціну книги на момент оформлення продажу)

6. **Shipment (Поштове відправлення / Доставка)**
   * `id`: UUID (PK)
   * `order_id`: UUID (FK)
   * `carrier`: string (nova_poshta | ukrposhta | courier)
   * `tracking_number`: string
   * `delivery_address`: string
   * `shipped_at`: timestamp
   * `delivery_status`: string

## 3. Зв'язки та кардинальності
* `Book }o--|{ Author`: чистий зв'язок багато-до-багатьох без сполучної таблиці. У напрямку `Book -> Author`: кожна книга обов'язково має від 1 до багатьох авторів (`|{` біля `Author`). У напрямку `Author -> Book`: автор у каталозі може мати від 0 до багатьох книг (`}o` біля `Book`).
* `Customer ||--o{ Order`: покупець має `0..*` замовлень; кожне замовлення належить рівно `1` покупцю.
* `Order ||--|{ OrderItem`: замовлення містить `1..*` позицій; кожна позиція належить рівно `1` замовленню.
* `Book ||--o{ OrderItem`: книга може входити у `0..*` позицій замовлень; кожна позиція посилається рівно на `1` книгу.
* `Order ||--o| Shipment`: замовлення має `0..1` відправлення; кожне відправлення прив'язане рівно до `1` замовлення.

## 4. Критерії прийняття (Acceptance Criteria)
- [ ] Модель нормалізована до 3NF: похідне агреговане поле `total_amount` вилучено із сутності `Order` на користь розрахунку через `OrderItem`.
- [ ] Відсутні фізичні сполучні таблиці-пустушки: зв'язок між `Book` та `Author` промодельовано як чистий M:N (`Book }o--|{ Author`).
- [ ] Асоціативна сутність `OrderItem` виділена окремо, має власний первинний ключ `id: UUID (PK)` та бізнес-атрибути `quantity` і `unit_price`, де `unit_price` фіксує історичну ціну книги на момент продажу (незмінну при подальшому подорожчанні книги в каталозі).
- [ ] Усі первинні та зовнішні ідентифікатори мають однаковий тип `UUID`.
- [ ] Модель згенерована у файлі `Labs/lab-1/model.mmd` у синтаксисі Mermaid `erDiagram`.