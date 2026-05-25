# Домашнее задание к занятию 16 «Платформа мониторинга Sentry»

## Задание 1 — Подключение Free Cloud Account

Был создан аккаунт на [sentry.io](https://sentry.io) через GitHub-авторизацию. Создан проект для Python.

<img width="1493" height="821" alt="2026-05-25_11-14-24" src="https://github.com/user-attachments/assets/21424e41-085a-426e-ad3b-7d01dc2473de" />


## Задание 2 — Генерация тестового события

В проекте была нажата кнопка **«Generate sample event»** . Sentry сгенерировал тестовое исключение. После этого событие было переведено в статус **Resolved**.

**Скриншот Stack trace из события:**

<img width="1193" height="830" alt="2026-05-25_11-36-09" src="https://github.com/user-attachments/assets/38609b61-c64c-4414-9799-adf2f536526d" />

**Скриншот списка событий после нажатия Resolved:**

<img width="912" height="742" alt="2026-05-25_11-40-04" src="https://github.com/user-attachments/assets/fa3d0d1a-3a7d-46ef-a688-a258313895a8" />

## Задание 3 — Настройка алёртинга

Было создано дефолтное правило алёртинга для проекта (условие: «When an issue is first seen», действие: отправка email). После повторной генерации sample event на почту, привязанную к GitHub-аккаунту, пришло оповещение.

**Скриншот тела сообщения из оповещения на почте:**

<img width="545" height="715" alt="2026-05-25_11-37-42" src="https://github.com/user-attachments/assets/65915e6b-dd92-4009-b902-9728fe917bdf" />

## Задание повышенной сложности — Подключение Sentry SDK к Python проекту

Был написан Python-скрипт, который подключается к Sentry SDK и отправляет несколько тестовых событий с различными параметрами (теги, пользовательский контекст, breadcrumbs, extra-данные).

### Пример кода (`sentry_test.py`)

    ```python
      import sentry_sdk
      from sentry_sdk import capture_exception, capture_message
      
      sentry_sdk.init(
          dsn="https://c05fc1a48c048aefda73be7eaed7340a@o4511449527812096.ingest.de.sentry.io/4511449536659536",
          send_default_pii=True,
          traces_sample_rate=1.0,
          environment="homework",
          release="v1.0.0"
      )
      
      print("🚀 Отправка тестовых событий в Sentry...")
      
      # 1. Простое информационное сообщение
      capture_message("Тестовое сообщение: Python-скрипт запущен успешно", level="info")
      print("✓ Информационное сообщение отправлено")
      
      # 2. Ошибка деления на ноль с тегами и контекстом пользователя
      try:
          result = 100 / 0
      except ZeroDivisionError as e:
          with sentry_sdk.configure_scope() as scope:
              scope.set_tag("task", "homework_16")
              scope.set_tag("language", "python")
              scope.set_tag("student", "relekt")
              scope.set_user({
                  "id": "study_user_001",
                  "username": "relekt",
                  "email": "relekt@study.com"
              })
              scope.set_extra("operation", "division_by_zero")
              scope.set_extra("attempt", 1)
              capture_exception(e)
          print("✓ ZeroDivisionError отправлен")
      
      # 3. Ошибка выхода за границы списка
      try:
          my_list = [1, 2, 3]
          value = my_list[10]
      except IndexError as e:
          with sentry_sdk.configure_scope() as scope:
              scope.set_tag("error_type", "index_error")
              scope.set_extra("list_length", len(my_list))
              scope.set_extra("attempted_index", 10)
              capture_exception(e)
          print("✓ IndexError отправлен")
      
      # 4. Ошибка с breadcrumbs (хлебными крошками)
      def process_user_data(user_id):
          sentry_sdk.add_breadcrumb(
              category="user.process",
              message=f"Начало обработки пользователя {user_id}",
              level="info"
          )
          
          if user_id < 0:
              sentry_sdk.add_breadcrumb(
                  category="user.process",
                  message=f"Обнаружен отрицательный user_id: {user_id}",
                  level="warning"
              )
              raise ValueError(f"Некорректный ID пользователя: {user_id}")
          
          sentry_sdk.add_breadcrumb(
              category="user.process",
              message=f"Успешная обработка пользователя {user_id}",
              level="info"
          )
          return {"user_id": user_id, "status": "ok"}
      
      try:
          process_user_data(-5)
      except ValueError as e:
          with sentry_sdk.configure_scope() as scope:
              scope.set_tag("function", "process_user_data")
              scope.set_tag("input_validation", "failed")
              capture_exception(e)
          print("✓ ValueError отправлен")
      
      print("\n✅ Все тестовые события отправлены!")

Скриншот меню Issues с полученными событиями:

<img width="912" height="742" alt="2026-05-25_11-40-04" src="https://github.com/user-attachments/assets/c48f77e7-bd9d-4455-a92b-68c05a607773" />




