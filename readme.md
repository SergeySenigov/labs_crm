
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

При появлении в очереди сообщений `crm_activity` сообщения с темой `client_request_new` :

Если поле сообщения inn соответствует длине ИНН (10 или 12 символов), то выполнить поиск в таблице `Информация о контрагентах` записи с полем `inn=Поле сообщения inn`.
Если такая запись найдена (поле inn уникально), то получить значение `contractor_id` найденной записи.

Создать в таблице `Заявки клиентов` новую запись со значениями:

|Поле БД|Значение|
|----------|---------|
|fio|Поле fio сообщения|
|phone|Поле phone сообщения|
|inn|Поле inn сообщения|
|contractor_id|Поле contractor_id найденной записи|
|create_dt|Поле create_dt сообщения|
|check_inn_user_id|null|
|man_inn_update_dt|null|
|status|'{OK, client_supplied_inn_found_in_crm}'|

Завершить процесс.

Если такая запись не найдена, то создать в таблице `Заявки клиентов` новую запись со значениями:
    
|Поле БД|Значение|
|----------|---------|
|fio|Поле fio сообщения|
|phone|Поле phone сообщения|
|inn|Поле inn сообщения|
|contractor_id|null|
|create_dt|Поле create_dt сообщения|
|check_inn_user_id|null|
|man_inn_update_dt|null|
|status|'{WAITING, request_ext_info_by_inn}'|

Получить значение `client_request_id` новой записи.

В очередь сообщений `crm_activity` поместить сообщение с темой `request_ext_info_by_inn` с данными
|Поле сообщения|Данные|
|----------|---------|
|client_request_id|Значение `client_request_id` созданной записи|
|inn|Поле inn сообщения|

Завершить процесс.

Если поле сообщения `inn` пустое или не соответствует длине ИНН (10 или 12 символов), то 

Создать в таблице `Заявки клиентов` новую запись со значениями:

|Поле БД|Значение|
|----------|---------|
|fio|Поле fio сообщения|
|phone|Поле phone сообщения|
|inn|Поле inn сообщения|
|contractor_id|null|
|create_dt|Поле create_dt сообщения|
|check_inn_user_id|null| 
|man_inn_update_dt|null|
|status|'{WAITING, request_manager_validate_inn}'|

Получить значение `client_request_id` новой записи.

В очередь сообщений `crm_activity` поместить сообщение с темой `request_manager_validate_inn` с данными
|Поле сообщения|Данные|
|----------|---------|
|client_request_id|Значение `client_request_id` созданной записи|
|inn|Поле inn сообщения|

Завершить обработку сообщения `client_request_new`.

---

При появлении в очереди сообщений `crm_activity` сообщения с темой `request_manager_validate_inn` :

Получить из таблицы настроек (предполагаем ее наличие) значение `user_id` менеджера, ответственного за уточнение ИНН клиента.

Найти в таблице `Заявки клиентов` по ключу `client_request_id` соответствующую запись и обновить значения полей:

|Поле БД|Значение|
|----------|---------|
|check_inn_user_id|значение `user_id` менеджера ответственного за уточнение ИНН|

Уведомить асинхронно средствами CRM менеджера (полученное значение `user_id`) о новой заявке с ИНН, требующим уточнения.

Завершить обработку сообщения `request_manager_validate_inn`

---

При появлении в очереди сообщений `crm_activity` сообщения с темой `reply_ext_info_by_inn`,

если поле сообщения reply_status равно 'not_found', то:
найти в таблице `Заявки клиентов` по ключу `client_request_id` соответствующую запись и обновить значения полей:

|Поле БД|Значение|
|status|'{WAITING, request_manager_validate_inn}'|

В очередь сообщений `crm_activity` поместить сообщение с темой `request_manager_validate_inn` с данными

|Поле сообщения|Данные|
|----------|---------|
|client_request_id|Поле `client_request_id` сообщения|
|inn|Поле `inn` сообщения|

Завершить.

Если поле сообщения reply_status равно 'found_ul', то создать в таблице `Информация о контрагентах` новую запись со значениями:

|Поле БД|Значение|
|----------|---------|
|longName|Поле data.longName сообщения|
|inn|Поле data.inn сообщения|
|kpp|Поле data.kpp сообщения|
|legalAddress|Поле data.legalAddress сообщения|

Получить contractor_id новой записи.

Создать записи в таблице `Учредители` с данными списка объектов `founders` из полученного сообщения.

Если поле сообщения reply_status равно 'found_ip', то создать в таблице `Информация о контрагентах` новую запись со значениями:

|Поле БД|Значение|
|----------|---------|
|longName|Поле data.fio сообщения|
|inn|Поле data.inn сообщения|
|kpp|null|
|legalAddress|Поле data.legalAddress сообщения|

Получить contractor_id новой записи.

Записи в таблице `Учредители` не создавать.

Обновить в таблице `Заявки клиентов` запись по ключу client_request_id = Поле `client_request_id` сообщения:

|Поле БД|Значение|
|contractor_id|contractor_id новой записи в `Информация о контрагентах`|
|status|'{OK, found_by_inn_in_external_service}'|

Завершить обработку сообщения `reply_ext_info_by_inn`

---
**Задание программисту внешних http интгераций**

При поступлении запроса по маршруту "web.crm-labs.ru/client_request/new", в очередь сообщений `crm_activity` поместить новое сообщение с темой `client_request_new` с данными:
|Поле сообщения|Данные|
|----------|---------|
|fio|Поле fio запроса|
|phone|Поле phone запроса|
|inn|Поле inn запроса|
|create_dt|Текущая дата, время|

При появлении в очереди сообщений `crm_activity` сообщения с темой `request_ext_info_by_inn` выполнить запрос web-сервиса 

response = https://focus-api.kontur.ru/api2/ul?key=`key`&inn=`inn`

используя `key` - ключ для сервиса, `inn` - значение поля `inn` из сообщения.


Если в response количество сущностей больше 0, то в очередь сообщений `crm_activity` поместить новое сообщение с темой `reply_ext_info_by_inn` с данными:

|Поле сообщения|Значение|
|----------|---------|
|reply_status|'found_UL'|
|longName|response.longName|
|inn|response.inn|
|kpp|response.kpp|
|legalAddress|дочерние элементы `response.address` собранные в одну строку|
|founders|json данных учредителей из response согласно структуре сущности Учредители в БД|


Если в response количество сущностей = 0, то выполнить запрос web-сервиса 

response = https://focus-api.kontur.ru/api2/ip?key=`key`&inn=`inn`

используя `key` - ключ для сервиса, `inn` - значение поля `inn` из сообщения.

Если в response количество сущностей больше 0, то в очередь сообщений `crm_activity` поместить новое сообщение с темой `reply_ext_info_by_inn` с данными:

|Поле сообщения|Значение|
|----------|---------|
|reply_status|'found_IP'|
|fio|response.fio|
|inn|response.inn|
|legalAddress|дочерние элементы `response.address` собранные в одну строку|

Если в response количество сущностей = 0, то очередь сообщений `crm_activity` поместить новое сообщение с темой `reply_ext_info_by_inn` с данными:

|Поле сообщения|Значение|
|----------|---------|
|reply_status|'not_found'|


**Задание программисту front-end CRM**
---

Разработать визуальную форму "Заявки ожидающие уточнения контрагента" отражающую данные таблицы `Заявки клиентов` по условию:
status = 'request_manager_validate_inn', check_inn_user_id = user_id текущего пользователя системы.

Форма доступна для просмотра пользователям с ролью клиентского менеджера.

Поля формы: ФИО (`fio`), Телефон (`phone`), ИНН (`inn`), Время создания (`create_dt`), Наименование (без значения по умолчанию).

Менеджер может вносить данные в поля `ИНН` и `Наименование`, и имеет опции "Отменить" и "Сохранить".

При вводе данных в поле `Наименование` асинхронно вызывается сервис "Контур-Фокус" автоподсказки, и выпадающий список заполняется найденными значениями по (в этом ТЗ данный процесс деатльно не описан).
При выборе пользователем из списка наименований, заполняется значение в поле "ИНН".

Либо пользователь может вручную внести ИНН в поле "ИНН".

При выборе пользователем "Сохранить":

Обновить в таблице `Заявки клиентов` запись по ключу client_request_id = текущий `client_request_id` записи открытой пользователем

|Поле БД|Значение|
|inn|Значение ИНН введенное менеджером|
|man_inn_update_dt| текущая дата/время| 

Закрыть форму

В очередь сообщений `crm_activity` поместить новое сообщение с темой `request_ext_info_by_inn` с данными:
|Поле сообщения|Данные|
|----------|---------|
|client_request_id|Значение `client_request_id` текущей записи|
|inn|Значение ИНН введенное менеджером|


