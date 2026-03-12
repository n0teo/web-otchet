# Лабораторная работа №3 Реализация серверной части на django rest. Документирование API.

* БД: Создать программную систему, предназначенную для администратора гостиницы.


## Модель Room
Модель для хранения информации о номерах в гостинице.

* `number` Номер комнаты
* `price` Стоимость за сутки
* `room_type` Тип номера
* `floor` Этаж, на котором расположен номер
* `phone` Номер телефона для связи с номером

```python
    class Room(models.Model):
        TYPES_OF_ROOMS = [('single', 'Одноместный'), ('tuple', 'Двухместный'), ('triple','Трехместный')]

        number = models.PositiveIntegerField(unique=True, verbose_name='Номер комнаты')
        price = models.DecimalField(max_digits=10, decimal_places=2, verbose_name='Стоимость за сутки')
        room_type = models.CharField(max_length=10, choices=TYPES_OF_ROOMS, verbose_name="Тип номера")
        floor = models.IntegerField(validators=[MinValueValidator(1)], verbose_name='Этаж')
        phone = models.CharField(max_length=15, verbose_name='Телефон')

        def __str__(self):
            return f"Комната {self.number} ({self.get_room_type_display()})"
```

## Модель Client
Модель для хранения информации о клиентах.

* `passport_number` Номер паспорта клиента
* `first_name` Имя клиента
* `last_name` Фамилия клиента
* `middle_name` Отчество клиента
* `city_of_origin` Город, из которого прибыл клиент
* `check_in_date` Дата заезда клиента в гостиницу
* `check_out_date` Дата выезда клиента из гостиницы.
* `room` Внешний ключ, указывающий на номер, в котором проживает клиент


```python
    class Client(models.Model):
        passport_number = models.CharField(max_length=20, verbose_name="Номер паспорта")
        first_name = models.CharField(max_length=50, verbose_name="Имя")
        last_name = models.CharField(max_length=50, verbose_name="Фамилия")
        middle_name = models.CharField(max_length=50, null=True, blank=True, verbose_name="Отчество")
        city_of_origin = models.CharField(max_length=100, verbose_name="Город проживания")
        check_in_date = models.DateField(verbose_name="Дата заезда")
        check_out_date = models.DateField(verbose_name='Дата уезда', null=True, blank=True)
        room = models.ForeignKey(Room, on_delete=models.PROTECT, related_name="clients", verbose_name="Номер")

        class Meta:
            unique_together = ('passport_number', 'check_in_date')

        def __str__(self):
            return f"{self.middle_name} {self.first_name} {self.last_name} ({self.passport_number})"
```


## Модель Employee
Модель для хранения информации о сотрудниках гостиницы.

* `first_name` Имя сотрудника
* `last_name` Фамилия сотрудника
* `middle_name` Отчество сотрудника
* `add_date` Дата принятия сотрудника на работу
* `delete_date` Дата увольнения сотрудника
* `dismissed` Флаг, указывающий на то, уволен ли сотрудник

```python
    class Employee(models.Model):
        first_name = models.CharField(max_length=50, verbose_name="Имя")
        last_name = models.CharField(max_length=50, verbose_name="Фамилия")
        middle_name = models.CharField(max_length=50, null=True, blank=True, verbose_name="Отчество")
        add_date = models.DateField(verbose_name='Дата приема на работу')
        delete_date = models.DateField(verbose_name='Дата увольнения', null=True, blank=True)
        dismissed = models.BooleanField('Уволен', default=False)
        def __str__(self):
            return f"{self.middle_name} {self.first_name} {self.last_name}"
```

## Модель CleaningSchedule
Модель для расписания уборки номеров.

* `employee` Связь с моделью `Employee`, указывающая на сотрудника
* `day_of_week` День недели
* `floor` Этаж, на котором будет проведена уборка

```python
class CleaningSchedule(models.Model):

    DAYS_OF_WEEK = [
        (1, 'Понедельник'),
        (2, 'Вторник'),
        (3, 'Среда'),
        (4, 'Четверг'),
        (5, 'Пятница'),
        (6, 'Суббота'),
        (7, 'Воскресенье'),
    ]

    employee = models.ForeignKey(Employee, on_delete=models.CASCADE, related_name="cleaning_schedules", verbose_name="Работник")
    day_of_week = models.IntegerField(choices=DAYS_OF_WEEK, verbose_name='День недели')
    floor = models.IntegerField(validators=[MinValueValidator(1)], verbose_name='Этаж')

    def __str__(self):
        return f"{self.employee} - {self.floor} - {self.day_of_week}"
```

# Endpoints

# Room endpoints

## list_rooms
/rooms/
methods: [GET, POST]
description: Возвращает список всех номеров в гостинице и позволяет добавить новый.
filters:
  room_type: Тип номера (например, single/double/luxe)
  floor: Этаж

## retrieve_room
/rooms/{id}/
methods: [GET, PUT, PATCH, DELETE]
description: Получение, обновление или удаление конкретного номера.

## room_history
/rooms/{id}/history/
methods: [GET]
description: История заселений в конкретном номере (только брони текущего пользователя).
url_parameters:
  id: ID номера
response:
  type: array
  items:
    properties:
      name: ФИО гостя
      from: Дата заезда
      to: Дата выезда

## available_rooms
/rooms/available/
methods: [GET]
description: Возвращает список свободных номеров на текущую дату.
response: Список номеров, которые свободны сегодня

## current_guest
/rooms/{id}/current_guest/
methods: [GET]
description: Возвращает информацию о текущем госте в номере.
url_parameters:
  id: ID номера
response:
  type: object
  properties:
    name: ФИО гостя
    passport: Номер паспорта
    check_in: Дата заезда
    check_out: Дата выезда
    message: Сообщение о том, что номер свободен


# Client endpoints

## list_clients
/clients/
methods: [GET, POST]
description: Возвращает список всех клиентов в гостинице, а так же позволяет добавить новых.
filters:
  last_name: Фамилия клиента
  room: ID номера
  passport_number: Номер паспорта

## retrieve_client
/clients/{id}/
methods: [GET, PUT, PATCH, DELETE]
description: Получение, обновление или удаление конкретного клиента.
permissions:
  update: Только создатель брони может редактировать

## client_history
/clients/{id}/history/
methods: [GET]
description: Возвращает историю всех проживаний клиента по номеру паспорта.
url_parameters:
  id: ID клиента
response: Список всех броней клиента (по номеру паспорта), отсортированный по дате заезда

## my_bookings
/clients/my-bookings/
methods: [GET]
description: Возвращает список броней текущего авторизованного пользователя.
permissions: [IsAuthenticated]
response:
  type: array
  items:
    properties:
      id: ID брони
      room_number: Номер комнаты
      room_id: ID комнаты
      room_type: Тип номера
      price: Цена за ночь
      guest_name: ФИО гостя
      passport_number: Номер паспорта
      check_in_date: Дата заезда
      check_out_date: Дата выезда
      city_of_origin: Город прибытия


# Employee endpoints

## list_employees
/employees/
methods: [GET, POST]
description: Возвращает список всех сотрудников гостиницы и позволяет добавить новых.
filters:
  last_name: Фамилия сотрудника
  dismissed: Уволен (true/false)

## retrieve_employee
/employees/{id}/
methods: [GET, PUT, PATCH, DELETE]
description: Получение, обновление или удаление конкретного сотрудника.

## employee_schedule
/employees/{id}/schedule/
methods: [GET]
description: Возвращает расписание уборок сотрудника.
url_parameters:
  id: ID сотрудника
response:
  type: array
  items:
    properties:
      day: День недели
      floor: Этаж


# Cleaning Schedule endpoints

## list_schedules
/cleaning_schedules/
methods: [GET, POST]
description: Возвращает расписание работников, которые убираются на этажах.
filters:
  employee: ID сотрудника
  day_of_week: День недели (1-7)

## retrieve_schedule
/cleaning_schedules/{id}/
methods: [GET, PUT, PATCH, DELETE]
description: Получение, обновление или удаление конкретной записи расписания.