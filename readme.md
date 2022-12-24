
**Задание программисту cайта**

---

Создать на сайте форму для ввода и отправки следующей информации:

ФИО, номер телефона, ИНН 

|Поле|Размер|Тип|Проверка|Параметр|
|----------|---------|-----|---|-|
|ФИО|40 символов|Текст|не пустое, только буквенные символы, пробелы, дефис|fio|
|Телефон|12 символов|Текст|не пустое, формат 89999999999 или +79999999999|phone|
|ИНН|12 символов|Текст|пусто или цифровые символы без пробелов|inn|

Предусмотреть кнопку отправки.

Кнопка отправки должна быть неактивна, пока не проверены значения в полях ФИО, Телефон согласно полям Проверка, Формат.

При несоотвестствии проверкам выводить рядом с полем сообщения о типе несоответствия.

Форма должна отправлять данные методом POST в адрес https://web.crm-labs.ru/client_request/new, используя названия полей из поля Параметр таблицы.


**Задание программисту БД CRM**

---

Создать в рабочей схеме базы данных таблицы (или дополнить существующие) для следующих сущностей:

Сущность `Заявки клиентов`, имя таблицы `client_requests`.

Поля: 

|Информация|Поле в БД|Тип|Ограничения|
|----------|---------|---------|---------|
|Код заявки|client_request_id|serial4|PRIMARY KEY|
|ФИО|fio|varchar(40)|not null|
|Телефон|phone|varchar(12)|not null|
|ИНН|inn|varchar(12)|-|
|Код контрагента|contractor_id|int4|FK on contractor_info.contractor_id|
|Время создания|create_dt|timestamp|not null|
|Ответственный менеджер|check_inn_user_id|int4|FK на таблицу пользователей|
|Время внесения ИНН менеджером|man_inn_update_dt|timestamp|-|
|Статус|status|varchar(30)|not null|

Сущность `Информация о контрагентах`, имя таблицы `contractor_info`.

Поля: 

|Информация|Поле в БД|Тип|Ограничения|
|----------|---------|---------|---------|
|Код контрагента|contractor_id|serial4|PRIMARY KEY|
|Время создания|create_dt|timestamp|not null, default current_timestamp|
|Полное наименование|longName|varchar|-|
|ИНН|inn|varchar(12)|not null|
|КПП|kpp|varchar(12)|-|
|Юридический адрес|legalAddress|varchar|-|

Сущность `Учредители`, имя таблицы `contractor_founders`.

Поля: 

|Информация|Поле в БД|Тип|Ограничения|
|----------|---------|---------|---------|
|Код учредителя|founder_id|serial4|PRIMARY KEY|
|Код контрагента|contractor_id|int4|not null, FK on contractor_info.contractor_id|
|Тип учредителя|founder_type|varchar(3)|not null, CHECK in ('FL', 'UL', 'FR')|
|ФИО ФЛ|fio|varchar|-|
|ИНН ФЛ|fl_inn|varchar(12)|-|
|Имя организации|org_name|varchar|-|
|ИНН ЮЛ|org_inn(10)|varchar|-|
|ОРГН|ogrn|varchar(13)|-|
|Страна|country|varchar(50)|-|
|Сумма|share|numeric(14,2)|-|
|Дата внесения|dt|date|-|


**Задание программисту бизнес-логики CRM**

---

При появлении в очереди сообщений `crm_activity` сообщения с темой `client_request_new`:

Если поле сообщения `inn` пустое или не соответствует длине ИНН (10 или 12 символов), то 

получить из таблицы настроек (предполагаем ее наличие) значение `user_id` менеджера, ответственного за уточнение ИНН клиента.

Создать в таблице `Заявки клиентов` новую запись со значениями:

|Поле БД|Значение|
|----------|---------|
|fio|Поле fio сообщения|
|phone|Поле phone сообщения|
|inn|Поле inn сообщения|
|contractor_id|null|
|create_dt|Поле create_dt сообщения|
|check_inn_user_id|user_id менеджера, ответственного за уточнение ИНН|
|man_inn_update_dt|null|
|status|'request_manager_check_inn'|

Получить значение ключа `client_request_id` созданной записи.

В очередь сообщений `crm_activity` поместить сообщение с темой `request_manager_check_inn` с данными
|Поле сообщения|Данные|
|----------|---------|
|client_request_id|Значение `client_request_id` созданной записи|
|fio|Поле fio сообщения|
|phone|Поле phone сообщения|
|inn|Поле inn сообщения|
|create_dt|Поле create_dt сообщения|
|check_inn_user_id|user_id менеджера, ответственного за уточнение ИНН|

[Завершить]

Если поле сообщения inn соответствует длине ИНН (10 или 12 символов), то выполнить поиск в таблице `Информация о контрагентах` записи с полем `inn=Поле сообщения inn`.

Если такая запись найдена, то 
получить значение `contractor_id` найденной записи, 
создать в таблице `Заявки клиентов` новую запись со значениями:

|Поле БД|Значение|
|----------|---------|
|fio|Поле fio сообщения|
|phone|Поле phone сообщения|
|inn|Поле inn сообщения|
|contractor_id|Поле contractor_id найденной записи|
|create_dt|Поле create_dt сообщения|
|check_inn_user_id|null|
|man_inn_update_dt|null|
|status|'inn_found_in_crm'|

[Завершить]


Если поле сообщения inn соответствует длине ИНН (10 или 12 символов) и не был найден в таблице `Информация о контрагентах`, то создать в таблице `Заявки клиентов` новую запись со значениями:
    
|Поле БД|Значение|
|----------|---------|
|fio|Поле fio сообщения|
|phone|Поле phone сообщения|
|inn|Поле inn сообщения|
|contractor_id|null|
|create_dt|Поле create_dt сообщения|
|check_inn_user_id|null|
|man_inn_update_dt|null|
|status|'requested_ext_info_by_inn'|

Получить значение `client_request_id` новой записи.

В очередь сообщений `crm_activity` поместить сообщение с темой `request_ext_info_by_inn` с данными
|Поле сообщения|Данные|
|----------|---------|
|client_request_id|Значение `client_request_id` созданной записи|
|inn|Поле inn сообщения|

[Завершить]

При появлении в очереди сообщений `crm_activity` сообщения с темой `reply_ext_info_by_inn`,
если поле сообщения reply_status не равно 'not_found', то:
* если reply_status = found_ul, то создать в таблице сущности `Информация о контрагентах` новую запись со значениями:

|Поле БД|Значение|
|----------|---------|
|longName|Поле data.longName сообщения|
|inn|Поле data.inn сообщения|
|kpp|Поле data.kpp сообщения|
|legalAddress|дочерние элементы узла `data.address` сообщения собранные в одну строку|

Получить contractor_id новой записи.

Создать необходимые записи в таблице `Учредители` с данными из полученного сообщения.

* если reply_status = found_ip, то 

создать в таблице сущности `Информация о контрагентах` новую запись со значениями:

|Поле БД|Значение|
|----------|---------|
|longName|Поле data.fio сообщения|
|inn|Поле data.inn сообщения|
|kpp|null|
|legalAddress|дочерние элементы узла `data.address` сообщения собранные в одну строку|


Получить contractor_id новой записи.

Записи в таблице `Учредители` не создавать.

* обновить в таблице `Заявки клиентов` запись с client_request_id = Поле `client_request_id` сообщения, поле `status` значением `found_by_inn_in_external_service`, поле contractor_id значением contractor_id новой записи в `Информация о контрагентах`.

---
**Задание программисту инфраструктуры**

При поступлении запроса по маршруту "client_request/new", в очередь сообщений `crm_activity` поместить новое сообщение с темой `client_request_new` с данными:
|Поле сообщения|Данные|
|----------|---------|
|fio|Поле fio запроса|
|phone|Поле phone запроса|
|inn|Поле inn запроса|
|create_dt|Текущая дата, время|


При появлении в очереди сообщений `crm_activity` сообщения с темой `request_manager_check_inn`

уведомить средствами CRM менеджера (значение `check_inn_user_id` из сообщения) о новой заявке с ИНН, требующим  уточнения.


**Задание программисту front-end CRM**
---

Разработать форму "Заявки ожидающие уточнения контрагента" отражающую данные таблицы `Заявки клиентов` по условию:
status = 'request_manager_check_inn', check_inn_user_id = user_id текущего пользователя системы.

Форма доступна для просмотра пользователям с ролью клиентского менеджера.

Поля формы: ФИО (`fio`), Телефон (`phone`), ИНН (`inn`), Время создания (`create_dt`).

Менеджер может вносить данные в поле `ИНН`. 

При внесении менеджером данных в поле `ИНН`, в очередь сообщений `crm_activity` поместить новое сообщение с темой `check_request_new` с данными:
|Поле сообщения|Данные|
|----------|---------|
|client_request_id|Значение `client_request_id` текущей записи|
|fio|Поле fio текущей записи|
|phone|Поле phone текущей записи|
|inn|Значение ИНН введенное менеджером|
|create_dt|Поле create_dt текущей записи|
|check_inn_user_id|user_id менеджера текущей записи|

**Задание программисту бизнес-логики CRM**

1. Сервис получения данных по ИНН
2. Сервис информирования менеджера


|Типы сообщений|Отправляет|Получает|
|----------|---------|-----|
|`client_request_new`|V|V|
|`request_ext_info_by_inn`|V|ext|
|`reply_ext_info_by_inn`|ext|V|
|`request_manager_check_inn`|V|man|

Добавить комментарии полей в БД