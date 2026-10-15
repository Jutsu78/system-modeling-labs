# Лог промптів AI-агента (Lab 2)

## Промпт 1: Початкова генерація вимог (EARS, User Stories, Gherkin, Traceability) та Use Case діаграми
Прочитай специфікацію `Labs/lab-2/spec.md` та модель даних першої лабораторної `Labs/lab-1/model.mmd`.
Створи два файли:
1. `Labs/lab-2/requirements.md` — містить:
   - Структуровані вимоги за шаблоном EARS (REQ-01 .. REQ-07).
   - User Stories (US-01 .. US-05).
   - Сценарії Gherkin (Given/When/Then) для ключових прецедентів.
   - Матрицю трасування у вигляді Markdown-таблиці зі зв'язками багато-до-багатьох (M:N) між REQ, US, UC та Gherkin.
2. `Labs/lab-2/model.mmd` — діаграма Use Case у синтаксисі Mermaid (`flowchart LR`), яка відображає межі системи, акторів (Customer, ClubMember з узагальненням, StoreManager, зовнішню систему CarrierService), прецеденти UC-01..UC-06 та зв'язки `<<include>>` і `<<extend>>`.

## Промпт 2: Усунення розбіжностей за результатами аудиту (EARS, Use Case діаграма та непокриті вимоги)
Онови файли `Labs/lab-2/requirements.md`, `Labs/lab-2/model.mmd` та перегенеруй `Labs/lab-2/diagram.png` відповідно до уточнених критеріїв у `Labs/lab-2/spec.md`:
1. У `Labs/lab-2/model.mmd`:
   - Заміни `<<include>>` та `<<extend>>` на видимі лапки `«include»` та `«extend» [club_member_status = true]`, щоб парсер Mermaid не вирізав їх як HTML-теги `<>` на PNG-рендері.
   - Прибери сирі текстові коди `fa:fa-...` із назв акторів.
   - Перегенеруй `Labs/lab-2/diagram.png` через mermaid-cli та перевір, що підписи `«include»` і `«extend»` чітко видно на зображенні.
2. У `Labs/lab-2/requirements.md`:
   - Виправ `REQ-06` для дотримання атомарності EARS: залиш один чіткий тригер (отримання webhook-події від `CarrierService`) та одне атомарне правило `SHALL` без подвійних `SHALL` і без конструкції «або».
   - Додай вимоги `REQ-08` (надсилання сповіщення з `Shipment.tracking_number` на `Customer.email` та `Customer.phone` при переході в `shipped`) та `REQ-09` (скасування замовлення покупцем у статус `Order.status = 'cancelled'`, поки воно у стані `pending`), які використовують решту атрибутів ER-моделі Завдання 1.
   - У Матриці трасування (розділ 4) явно познач `REQ-08` та `REQ-09` як **НЕПОКРИТІ (`— [Непокрита]`)** прецедентами Use Case та сценаріями Gherkin і додай окремий підрозділ із поясненням причин непокриття (межі поточної ітерації системи та відсутність зовнішнього Notification Gateway).
   - У таблиці 4.4 (звірка з ER-моделлю Завдання 1) перелічи абсолютно всі атрибути кожної з 6 сутностей із `Labs/lab-1/model.mmd`.