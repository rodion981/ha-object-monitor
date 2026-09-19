# Object Event Monitor

[English](./README.md) | [**Українською**](./README.uk.md)

Object Event Monitor — це кастомна інтеграція Home Assistant, яка перетворює зміни сутностей, вибраних за допомогою labels, на події Home Assistant для автоматизацій.

Вона корисна для систем із кількома об'єктами: будинками, кафе, ресторанами, готелями, офісами, складами та іншими віддаленими об'єктами.

Інтеграція працює на основі подій і використовує labels Home Assistant. Ролі labels налаштовуються в параметрах інтеграції:

- Моніторинг доступності стежить за вибраними сутностями та реагує на стани `unavailable` і відновлення. Label за замовчуванням: `device_monitoring`.
- Налаштовані labels об'єктів, наприклад `home`, `restaurant` або `cafe`, визначають об'єкт, який відстежується.
- Необов'язкові labels категорій можуть використовуватися для маршрутизації логіки автоматизацій. За замовчуванням: `security`, `light`, `climate`.
- Необов'язкові відображувані назви дозволяють показувати в подіях зрозумілі людині назви об'єктів і категорій, водночас самі labels залишаються стабільними латинськими ID.
- Системи безпеки можна моніторити за допомогою label `security_system`.
- Зміни станів on/off можна моніторити за допомогою label `state_monitor`.

Інтеграція не викликає скрипти сповіщень напряму. Вона генерує окремі
події Home Assistant, а ваші автоматизації вже вирішують, як саме сповіщати людей.

Підтримувані події Home Assistant:

- `object_monitor_offline`
- `object_monitor_recovery`
- `object_monitor_security_state`
- `object_monitor_on_off_state`

## Встановлення через HACS

1. Відкрийте HACS.
2. Перейдіть до **Integrations**.
3. Відкрийте меню з трьома крапками.
4. Виберіть **Custom repositories**.
5. Додайте URL цього репозиторію:

   ```text
   https://github.com/rodion981/ha-object-monitor
   ```

6. Виберіть категорію **Integration**.
7. Встановіть **Object Event Monitor**.
8. Перезапустіть Home Assistant.

## Базове налаштування

Додайте інтеграцію через:

```text
Settings -> Devices & services -> Add integration -> Object Event Monitor
```

Налаштуйте ролі labels:

- Label моніторингу: `device_monitoring`
- Labels категорій: `security`, `light`, `climate` або власні labels
- Labels об'єктів, наприклад:

```text
home
restaurant
cafe
```

За потреби налаштуйте відображувані назви для подій:

```text
home=Home
restaurant=Restaurant
cafe=Cafe
```

Їх також можна вказати в одному рядку:

```text
home=Home, restaurant=Restaurant, cafe=Cafe
```

А також відображувані назви категорій:

```text
security=Security
power=Power
internet=Internet
```

Назви категорій також підтримують записи через кому.

Тригер автоматизації:

```yaml
triggers:
  - trigger: event
    event_type: object_monitor_offline
```

Після цього призначте labels сутностям, які потрібно моніторити.

Приклад події лише для об'єкта:

```text
device_monitoring
home
```

Приклад події з категорією:

```text
device_monitoring
home
security
```

Labels таймауту для окремих сутностей можуть перевизначати таймаут за замовчуванням:

```text
timeout_20s
timeout_7m
timeout_1h
```

Для однієї сутності використовуйте лише один label таймауту. Якщо такого label немає, Object Event Monitor використовує таймаут за замовчуванням із параметрів інтеграції.

## Сенсори проблем доступності об'єктів

Для кожного налаштованого label об'єкта Object Event Monitor створює binary
sensor із класом проблеми.

Приклад:

```text
binary_sensor.home_availability_problem
binary_sensor.restaurant_availability_problem
```

Сенсор використовує device class `problem` у Home Assistant:

- `off` означає, що для об'єкта немає підтвердженої проблеми доступності.
- `on` означає, що принаймні одна сутність цього об'єкта, яка моніториться за доступністю, підтверджено offline.

Враховуються лише інциденти доступності. Стани системи безпеки та моніторинг
on/off не впливають на ці агреговані сенсори.

Корисні атрибути:

```yaml
object_label: home
object_name: Home
monitored_count: 12
offline_count: 1
pending_count: 0
offline_entities:
  - sensor.home_router
pending_entities: []
```

## Моніторинг системи безпеки

Щоб моніторити alarm panel, додайте ці labels до сутності `alarm_control_panel`:

```text
security_system
home
```

`home` має бути одним із налаштованих labels об'єктів.

Зміни стану системи безпеки генерують подію `object_monitor_security_state`.

Підтримувані стани: `disarmed`, `armed_home`, `armed_away`, `armed_night`, `armed_vacation`, `arming`, `pending`, `triggered`, `unknown` та `unavailable`.

## Моніторинг станів On/Off

Щоб відстежувати звичайні зміни `on` / `off`, додайте до сутності такі labels:

```text
state_monitor
home
power
```

`home` має бути одним із налаштованих labels об'єктів. `power` має бути одним із
налаштованих labels категорій, якщо ви використовуєте наданий пакет автоматизацій із
маршрутизацією `script.tg_<object>_<category>`.

Моніторинг on/off генерує:

```yaml
event_type: object_monitor_on_off_state
event_data:
  event_type: on_off_state
  entity_id: binary_sensor.home_power
  friendly_name: Main Power
  object_label: home
  category: power
  previous_state: off
  state: on
```

Якщо сутність використовує власні значення станів, налаштуйте їх у параметрах
інтеграції:

```text
Custom ON state values:
on
увімкнено

Custom OFF state values:
off
вимкнено
```

У згенерованій події все одно використовуються нормалізовані `state: on` або `state: off`, тому
автоматизації залишаються стабільними.

## Сервіси

Інтеграція надає:

- `object_monitor.send_test_notification`
- `object_monitor.reload_monitored_entities`
- `object_monitor.clear_entity_state`

Використовуйте `object_monitor.send_test_notification` у Developer Tools, щоб згенерувати тестову
подію доступності перед перевіркою реальних сутностей зі станом `unavailable`.
