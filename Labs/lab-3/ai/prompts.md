# Лог промптів AI-агента (Lab 3)

## Промпт 1: Початкова генерація Sequence-діаграм для трьох сценаріїв UC-02
Прочитай вхідну специфікацію `Labs/lab-3/spec.md`, модель даних `Labs/lab-1/model.mmd` та сценарії Gherkin у `Labs/lab-2/requirements.md`.
Створи такі артефакти для трьох сценаріїв прецеденту `UC-02`:
1. Три окремі файли Mermaid-діаграм та їхні PNG-рендери:
   - `Labs/lab-3/scenario-1-order-success.mmd` (та `.png`) — Сценарій 1: Успішне оформлення замовлення за достатньої кількості книг на складі.
   - `Labs/lab-3/scenario-2-insufficient-stock.mmd` (та `.png`) — Сценарій 2: Відмова в оформленні замовлення при перевищенні наявного залишку на складі.
   - `Labs/lab-3/scenario-3-club-member-discount.mmd` (та `.png`) — Сценарій 3: Оформлення замовлення членом клубу із застосуванням клубної знижки 15%.
2. Файл `Labs/lab-3/interactions.md` — документація всіх 3 сценаріїв із детальним описом учасників, кроків, вбудованими блоками `sequenceDiagram` та таблицею трасування «Сценарій Gherkin (Lab 2) ↔ Sequence Diagram (Lab 3) ↔ Сутності/Атрибути ER (Lab 1)».
У кожній діаграмі використай реалістичні назви методів, return-повідомлення (`-->>`), `activate`/`deactivate` та комбіновані фрагменти (`loop`, `alt`, `opt`).

## Промпт 2: Заміна opt на двогілкові alt, виправлення Activation Bars, моделювання REQ-07 та 3NF
Онови виключно текстові файли `Labs/lab-3/scenario-1-order-success.mmd`, `Labs/lab-3/scenario-2-insufficient-stock.mmd`, `Labs/lab-3/scenario-3-club-member-discount.mmd` та `Labs/lab-3/interactions.md` згідно з уточненою `Labs/lab-3/spec.md`.
ВАЖЛИВО: НЕ генеруй і не оновлюй файли зображень (`.png`) та НЕ запускай `mermaid-cli` — рендери діаграм будуть створені вручну.

Виконай такі зміни у файлах `.mmd` та `interactions.md`:
1. **Повна заміна всіх фрагментів `opt` на двогілкові `alt ... else ...`:**
   - Прибери всі блоки `opt` із усіх трьох діаграм.
   - Для розрахунку ціни всередині циклу `loop` використай `alt club_member_status == true (UC-04: Застосування клубної знижки 15%)` із викликом `calculateClubPrice(basePrice, discount=0.15)` та гілку `else club_member_status == false (Стандартна каталожна ціна)` із викликом `assignStandardPrice(basePrice)`.
   - Для завершення обробки після циклу перевірки складу використай `alt hasStockDeficit == false (Усі позиції зарезервовано)` (де виконуються збереження `Order`, цикл `loop` збереження `OrderItem`, переведення в `status="confirmed"` та `commitTransaction()`) і гілку `else hasStockDeficit == true (Виявлено дефіцит на складі)` (де виконується `rollbackTransaction()`).
   - У Сценарії 3 перевірку імутабельності історичної ціни (`REQ-07`) після оформлення замовлення також оформи через `alt Подальша зміна Book.price у каталозі (price = 350.00)` (де `BookRepository.updateBookPrice(bookId, newPrice=350.00)` оновлює прайс, а `OrderRepository.findOrderItemsByOrderId(orderId)` повертає незмінне `unit_price=255.00`) та `else Ціна в каталозі без змін (price = 300.00)` (де `OrderRepository.findOrderItemsByOrderId(orderId)` також повертає `unit_price=255.00`).
2. **Нерозривні Activation Bars до і після фрагментів (`alt`, `loop`):**
   - Використовуй явні директиви `activate <Participant>` перед початком фрагментів `loop`/`alt` та `deactivate <Participant>` після завершення фрагментів, щоб `OrderController`, `OrderService`, `BookRepository` та `OrderRepository` мали нерозривні лінії активації, які починаються до фрагмента і закінчуються після нього, не обриваючись в одній гілці.
   - Не використовуй `OrderService-->>OrderController: throw` всередині першого циклу `loop` — повернення відповіді (успішний `OrderResponse` або `InsufficientStockException`) від `OrderService` до `OrderController` має відбуватися один раз у самому кінці діаграми після завершення фрагмента транзакції.
3. **Явні методи транзакцій (`NFR-02`) та чистота 3NF:**
   - Використовуй явний виклик `OrderService->>OrderService: beginTransaction(REPEATABLE_READ)` перед циклом та `commitTransaction()` / `rollbackTransaction()` у фінальному `alt`.
   - Видали позамодельні параметри `totalAmount=600.00`, `discountAmount=45.00` та `savings=45.00` із викликів та відповідей (використовуй лише атрибути з `Labs/lab-1/model.mmd`).
4. **Очищення посилань у `interactions.md`:**
   - Заміни всі абсолютні шляхи `file:///d:/system-modeling-labs/...` у `Labs/lab-3/interactions.md` на відносні шляхи Markdown (`./spec.md`, `../lab-1/model.mmd`, `../lab-2/requirements.md`, `./scenario-1-order-success.mmd` тощо) та онови опис фрагментів.
   
   Виправ розташування `activate` та `deactivate` навколо блоків `alt` у файлах `Labs/lab-3/scenario-1-order-success.mmd`, `Labs/lab-3/scenario-2-insufficient-stock.mmd`, `Labs/lab-3/scenario-3-club-member-discount.mmd` та синхронізуй їх із `Labs/lab-3/interactions.md`:
1. У першому циклі `loop` всередині гілки `else stock_quantity < quantity` додай виклик звільнення блокування `OrderService->>BookRepository: releaseRowLock(bookId)` та відповідь `BookRepository-->>OrderService: false`, щоб `BookRepository` працював в обох гілках `alt`, а його `activate BookRepository` (до `alt`) і `deactivate BookRepository` (після `end`) утворювали нерозривну смугу через обидві гілки.
2. У блоці `alt hasStockDeficit == false ... else hasStockDeficit == true` винеси `activate OrderRepository` ПЕРЕД рядком `alt hasStockDeficit == false`, а `deactivate OrderRepository` — ПІСЛЯ `end` цього блоку `alt`. У гілці `else hasStockDeficit == true` додай виклик `OrderService->>OrderRepository: auditRejectedOrder(customerId, reason="INSUFFICIENT_STOCK")` та `OrderRepository-->>OrderService: false`, щоб `OrderRepository` був задіяний в обох гілках і його activation bar не обривався в одній гілці.
3. У `scenario-3-club-member-discount.mmd` у фінальному блоці `alt Подальша зміна Book.price у каталозі (price = 350.00) ... else Ціна в каталозі без змін (price = 300.00)` винеси `activate BookRepository` та `activate OrderRepository` ПЕРЕД початком `alt`, а `deactivate BookRepository` та `deactivate OrderRepository` постав ПІСЛЯ `end` цього `alt` (у гілці `else` додай перевірочне читання `OrderService->>BookRepository: getCurrentPrice(bookId)` -> `BookRepository-->>OrderService: 300.00` перед викликом `findOrderItemsByOrderId`).