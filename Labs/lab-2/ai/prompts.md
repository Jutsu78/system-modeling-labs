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