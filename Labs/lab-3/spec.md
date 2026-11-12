# Специфікація взаємодії (Sequence Diagrams): Оформлення замовлення (`UC-02`)

## 1. Намір
Деталізувати внутрішню взаємодію компонентів системи та репозиторіїв бази даних для всіх трьох сценаріїв прецеденту **`UC-02: Оформлення замовлення на книги`** (включно з обов'язковим `UC-03: Перевірка наявності книг на складі` та розширенням `UC-04: Застосування клубної ціни`), сформульованих у Завданні 2 та повністю узгоджених із 3NF ER-моделлю Завдання 1.

## 2. Обраний Use Case та його сценарії із Завдання 2
Базовий прецедент: **`UC-02: Оформлення замовлення на книги`** (`REQ-02`, `REQ-03`, `REQ-04`, `REQ-07`, `NFR-02`; `US-02`, `US-03`).

1. **Сценарій 1 (Позитивний потік — `UC-02` + `UC-03`):** Успішне оформлення замовлення звичайним покупцем (`club_member_status = false`) на 2 примірники книги «Інтернат» (`price = 300.00`, `stock_quantity = 5 -> 3`) із фіксацією `OrderItem.unit_price = 300.00` та статусу `Order.status = 'confirmed'`.
2. **Сценарій 2 (Негативний потік — `UC-02` + `UC-03`):** Відмова в оформленні замовлення на 3 примірники книги «Ворошиловград» при залишку `stock_quantity = 2` (`OrderItem.quantity > Book.stock_quantity`), відкат транзакції (`ROLLBACK`) без створення записів `Order`/`OrderItem` та збереження `stock_quantity = 2`.
3. **Сценарій 3 (Потік розширення — `UC-02` + `UC-03` + `UC-04`):** Оформлення замовлення членом клубу (`club_member_status = true`) на 1 примірник книги «Інтернат» із застосуванням знижки 15% (`OrderItem.unit_price = 255.00`, `stock_quantity = 5 -> 4`) та подальшою перевіркою незмінності `OrderItem.unit_price = 255.00` після оновлення каталожної ціни `Book.price` до `350.00` (`REQ-07`).

## 3. Учасники взаємодії (узгоджені з ER-моделлю Завдання 1)
* **Актори:**
  * `Customer` / `ClubMember` — покупець або авторизований член читацького клубу (`Customer.id`, `Customer.club_member_status`).
* **Компоненти та доменні репозиторії системи:**
  * `OrderController` — вхідний REST API контролер обробки запитів замовлення.
  * `OrderService` — сервіс оркестрації бізнес-логіки, керування транзакціями `REPEATABLE_READ` (`NFR-02`) та розрахунку ціни позиції.
  * `CustomerRepository` (`Customer`) — доступ до сутності `Customer` (`id`, `email`, `phone`, `full_name`, `club_member_status`, `created_at`).
  * `BookRepository` (`Book`) — доступ до сутності `Book` (`id`, `isbn`, `title`, `publication_year`, `price`, `stock_quantity`), блокування рядків `FOR UPDATE` та оновлення залишків.
  * `OrderRepository` (`Order`, `OrderItem`) — збереження сутностей `Order` (`id`, `customer_id`, `order_date`, `status`) та `OrderItem` (`id`, `order_id`, `book_id`, `quantity`, `unit_price`).

## 4. Критерії прийняття (Acceptance Criteria)
- [ ] **Окрема діаграма на кожен сценарій:** Побудовано 3 детальні Sequence-діаграми у синтаксисі Mermaid (`sequenceDiagram`) та їхні PNG-рендери (`scenario-1-order-success`, `scenario-2-insufficient-stock`, `scenario-3-club-member-discount`).
- [ ] **Використання `loop` та двогілкових `alt ... else ...` із нерозривними Activation Bars:** Замість неповних блоків `opt` усі умовні розгалуження (перевірка залишку `stock_quantity >= quantity`, вибір тарифу `club_member_status == true / false`, фіналізація транзакції `COMMIT / ROLLBACK`) моделюються виключно через двогілкові фрагменти `alt ... else ...`. Activation bar кожного учасника (`OrderController`, `OrderService`, `BookRepository`, `OrderRepository`), що працює поза фрагментом або в кількох його гілках, починається ДО фрагмента (`activate`) і закінчується ПІСЛЯ нього (`deactivate`) — не обривається в одній гілці.
- [ ] **Сувора відповідність 3NF ER-моделі (Завдання 1) та явні транзакції (`NFR-02`):** У викликах та відповідях використовуються виключно атрибути з `Labs/lab-1/model.mmd` (без денормалізованих полів `total_amount` або `savings`); початок і завершення транзакції моделюються явними викликами `beginTransaction(REPEATABLE_READ)`, `commitTransaction()` та `rollbackTransaction()`.
- [ ] **Повне покриття кроків Gherkin (Завдання 2) та відносні посилання:** У Сценарії 3 крок перевірки імутабельності `OrderItem.unit_price = 255.00` після зміни `Book.price` на `350.00` промодельовано реальними викликами методів до `BookRepository` та `OrderRepository`; усі посилання в `interactions.md` є відносними (без локальних шляхів `file:///`).