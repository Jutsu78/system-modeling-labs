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
   * `unit_price`: decimal

6. **Shipment (Поштове відправлення / Доставка)**
   * `id`: UUID (PK)
   * `order_id`: UUID (FK)
   * `carrier`: string (nova_poshta | ukrposhta | courier)
   * `tracking_number`: string
   * `delivery_address`: string
   * `shipped_at`: timestamp
   * `delivery_status`: string

## 3. Зв'язки та кардинальності
* `Book }|--o{ Author`: чистий зв'язок багато-до-багатьох; кожна книга обов'язково має щонайменше одного автора (1..*), автор може мати від 0 до багатьох книг.
* `Customer ||--o{ Order`: один покупець може оформити від 0 до багатьох замовлень.
* `Order ||--|{ OrderItem`: замовлення обов'язково містить від 1 до багатьох позицій.
* `Book ||--o{ OrderItem`: книга може входити до 0 або багатьох позицій замовлень.
* `Order ||--o| Shipment`: замовлення має 0 або 1 поштове відправлення.

## 4. Критерії прийняття (Acceptance Criteria)
- [ ] Модель нормалізована до 3NF: розрахункові агреговані суми (як-от `total_amount`) усунені з базової сутності `Order` на користь динамічного розрахунку з `OrderItem`, або строго зафіксовані як незмінний фінансовий чек. Для чистоти 3NF поле `total_amount` вилучено з сутності Order.
- [ ] Відсутні фізичні сполучні таблиці-пустушки: зв'язок між `Book` та `Author` промодельовано як чистий M:N.
- [ ] Асоціативна сутність `OrderItem` має власний первинний ключ `id: UUID (PK)`.
- [ ] Усі первинні та зовнішні ідентифікатори мають однаковий тип `UUID`.
- [ ] Модель згенерована у файлі `Labs/lab-1/model.mmd` у синтаксисі Mermaid `erDiagram`.