# smart_home_API_guide
Smart home app API guide

### Структура запроса:
- class - название класса
- action - название метода
- param - параметры метода

После авторизации, пользователь получает токен, который в дальнейшем используется в каждом запросе к БД.
При неактивности более определенного времени (в конфигах = 2ч), токен становится недействительным.
Токен лучше передавать в заголовке запроса, но пока временно он передается вместе с параметрами запроса.
При любом запросе происходит проверка введенной информации на наличие инъекций и запрещенных символов, а также
проверка токена. При просроченном токене дальнейшие действия блокируются, требуется повторная авторизация.

Ответ приходит в виде JSON строки:
```js
{"status":"...","msg":"...","data":{...}}
// status может быть или "success" -удачно, или "error" - ошибка
// msg - текстовая строка с расшифровкой ошибки
// data - массив возвращаемых данных, если они должны быть
```
#### Структура запроса на примере AJAX методом POST на JS:

```js
// URL адрес АПИ скрипта
var url = 'https://eibl.ru/api.php';

// обязательные параметры запроса
var class_name = '{Имя класса}';
var action_name = '{Имя метода}';

// массив параметров для вызываемого метода
var params = {
        key1 : param1,
        key2 : param2,
        keyN : paramN,
}

// параметр, требуемый для выполнения любого метода, кроме регистрации/ авторизации
var token = '{токен, полученный при авторизации}';

$.post(
  url,
  {
      class: class_name,
      action: action_name,
      param: { 'token': token, ...params }
  },
  function (data)
  {
    console.log('status: ', data.status, ', msg: ', data.msg );
  }
);
```

____

### Классы
Названия классов соответствуют названиям таблиц в БД
- Access
- Controls
- Icons
- Icon_colors
- Nets
- Nodes
- Node_groups
- Node_types
- Rooms
- Scenario
- Schedule
- States
- Triggers
- Users

### Методы
общие основные методы работы с таблицами
- add
- edit
- del
- getByField

### Примеры запросов с основными методами:

#### add
Добавить запись в таблицу. В параметрах метода указывается массив с названиями полей и их значениями.
Во всех таблицах первое поле является ключевым и заполняется автоматически (AUTO_INCREMENT), поэтому при создании
новой записи его указывать не нужно. При удачном выполнении запрос вернет insert_id - это код ключевого поля новой записи текущей таблицы (в данном примере - id_room).
```js
$.post(
  url,
  {
      class: 'Rooms',
      action: 'add',
      param: { 'token': 'zBpjfMzIxd076EEazvdTe0ADdi9Ro5oB', 'id_net': 1, 'name': 'балкон' }
  },
  function (data) {}
);

// Ответ:
{"status":"success","msg":"record successfully added","insert_id":8}
```
#### edit
Изменить запись в таблице. В параметрах метода указывается массив, начиная с ключевого поля изменяемой записи и далее с названиями изменяемых полей и их значениями
```js
$.post(
  url,
  {
      class: 'Rooms',
      action: 'edit',
      param: { 'token': 'zBpjfMzIxd076EEazvdTe0ADdi9Ro5oB', 'id_room': 8, 'name': 'терраса' }
  },
  function (data) {}
);

// Ответ:
{"status":"success","msg":"record updated successfully"}
```
#### del
Удалить запись в таблице. В параметрах метода указывается ключевое поле удаляемой записи
```js
$.post(
  url,
  {
      class: 'Rooms',
      action: 'del',
      param: { 'token': 'zBpjfMzIxd076EEazvdTe0ADdi9Ro5oB', 'id_room': 8 }
  },
  function (data) {}
);

// Ответ:
{"status":"success","msg":"record deleted successfully"}
```
#### getByField
Получить данные из таблицы по значению одного или нескольких полей.
В параметрах метода указывается ключ-значение нужных полей для фильтра.
```js
$.post(
  url,
  {
      class: 'Nodes',
      action: 'getByField',
      param: { 'token': 'zBpjfMzIxd076EEazvdTe0ADdi9Ro5oB',
        'id_room': 6,
        'key_access': 3
        }
  },
  function (data) {}
);

// Ответ:
{"status":"success","msg":"data loaded successfully",
"data":[{"id_node":"4","id_net":"1","id_room":"6","id_type_device":"1",
"key_access":"3","unicast":"5","where":null,"battery":null,"friend_device":null,
"power":"1000","sensor_address":null,"switcher":null,"direct":null,"double":null}]}
```

____

### Список необходимых запросов для Экранов

### Экран Main
#### getMainInfo
Получить данные для экрана по коду авторизованного пользователя
```js
$.post(
  url,
  {
      class: 'Main',
      action: 'getMainInfo',
      param: { 'token': 'zBpjfMzIxd076EEazvdTe0ADdi9Ro5oB' }
      // пользователь и код активной сети определяется по токену
  },
  function (data) {}
);

// Ответ:
{"status":"success","msg":"data loaded successfully",
"data": { "user":{...}, "net":{...}, "nodes":{...}, "scenarios":{...}}}

// если net не выбрана, придет только user
// массивы nodes и scenarios отсортированы по популярности (count_click)

```

### Экран Enter
#### registration
Зарегистрировать пользователя (пример: имя=Sergey, email=testov@mail.ru, тел=123456789, установить пароль=test)
```js
$.post(
  url,
  {
      class: 'Users',
      action: 'registration',
      param: { 'email': 'testov@mail.ru', 'password': 'test', 'name': 'Sergey', 'phone' : '123456789' }
  },
  function (data) {}
);

// Ответ:
{"status":"success","msg":"user registered successfully",
"data":{"email":"testov@mail.ru","password":"test","name":"Sergey","phone":"123456789"}}
```
#### log_in
Авторизовать зарегистрированного пользователя (пример: по email=testov@mail.ru, с паролем=test)
```js
$.post(
  url,
  {
      class: 'Users',
      action: 'log_in',
      param: { 'email': 'testov@mail.ru', 'password': 'test' }
  },
  function (data) {}
);

// Ответ:
{"status":"success","token":"zBpjfMzIxd076EEazvdTe0ADdi9Ro5oB"}
```
#### change_password
Сброс и смена пароля. В параметрах метода указываются старый и новый пароль
```js
$.post(
  url,
  {
      class: 'Users',
      action: 'change_password',
      param: { 'token': 'zBpjfMzIxd076EEazvdTe0ADdi9Ro5oB',
                'old_password': 'test', 'new_password': 'admin' }
                // пользователь определяется по токену
  },
  function (data) {}
);

// Ответ:
{"status":"success","msg":"user password changed successfully"}
// если старый пароль неправильный
{"status":"error","msg":"wrong old password"}
```

### Экран Edit Scenario
#### getInfo
Получить данные о сценарии
```js
$.post(
  url,
  {
      class: 'Scenario',
      action: 'getInfo',
      param: { 'token': 'zBpjfMzIxd076EEazvdTe0ADdi9Ro5oB', 'id_scenario': 12 }
      // пользователь и код активной сети определяется по токену
  },
  function (data) {}
);

// Ответ:
{"status":"success","msg":"data loaded successfully",
"data":[{name: "Все ушли", id_icon: "7", color: "#FBB050", count_click: "2"}]}
```

### Экран Scenario List
#### getMainInfo
Список сценариев для активной сети юзера
```js
// Данные можно взять из полученного массива data.scenarios, метода getMainInfo экрана Main
```
#### delScenario
Удалить сценарий и связанные с ним записи в таблицах Controls и Triggers
```js
$.post(
  url,
  {
      class: 'Scenario',
      action: 'delScenario',
      param: { 'token': 'zBpjfMzIxd076EEazvdTe0ADdi9Ro5oB', 'id_scenario': 12 }
  },
  function (data) {}
);

// Ответ:
{"status":"success","msg":"record deleted successfully"}
```

### Экран New Scenario
#### addScenario
Сохранить новый сценарий
```js
$.post(
  url,
  {
      class: 'Scenario',
      action: 'addScenario',
      param: { 'token': 'zBpjfMzIxd076EEazvdTe0ADdi9Ro5oB', 'name': 'My new scenario', 'id_icon': 2 }
      // пользователь и код активной сети определяется по токену
  },
  function (data) {}
);

// Ответ:
{"status":"success","msg":"record successfully added","insert_id":25}
// id_scenario = 25
```
### Таблицы
Описание таблиц
#### Access - Таблица доступа.
Активность пользователя в сети net с указанием прав
- id_access (AUTO_INCREMENT)
- id_user
- id_net
- id_type_user (флаг админ)
- active (флаг активности)

#### Controls - Контролируемые ноды.
Если требуется без сценария контролировать ноду (одну или несколько, но одного типа), необходимо предварительно создать новую группу в таблице Node_groups, с указанием кода родительской группы (если 0 - корень). После этого добавить нужным однотипным нодам в таблице Nodes код созданной группы id_group, а также указать его в данной
таблице в поле id_group.
При необходимости в сценарии управлять несколькими нодами одного типа - опустить id_node и заполнить id_group - код группы однотипных нодов.
- id_ctrl (AUTO_INCREMENT)
- id_node (код управляющего устройства, если нет сценария / управляемого, если есть сценарий)
- id_group (код группы нодов)
- id_scenario (код сценария)
- id_state (код состояния)
- id_sign (код знака условия для установки состояния)
- id_schedule (код расписания)
- value (значение состояния)
- name

#### Icons - Цвета иконок.
Id_icon совпадает с именем файла иконки, лежащей на сервере в каталоге /icons,
с расширением .svg (пример: /icons/10.svg)
- id_icon (AUTO_INCREMENT)
- name (название иконки)
- color_default (цвет иконки по умолчанию) #FFFFFF - белый

#### Icon_colors - Таблица переназначения цветов иконок для пользователя.
Все иконки имеют цвет по умолчанию, который указан в таблице Icons. Здесь ставим приоритетный цвет в HEX формате для конкретной иконки для конкретного пользователя
- id_icon_color (AUTO_INCREMENT)
- id_user (код пользователя)
- id_icon (код иконки, цвет которой нужно изменить)
- color (код нужного цвета в HEX)

#### Nets - Сети mash.
Таблица пользовательских сетей
- id_net (AUTO_INCREMENT)
- name (название сети)
- key_net

#### Nodes - Ноды.
Устройства в сети mash
- id_node (AUTO_INCREMENT)
- id_group (код группы к которой принадлежит нода)
- id_net (код сети mash)
- id_room (название локации)
- id_node_type (тип устройства)
- id_schedule (код расписания)
- name (название устройства)
- key_access
- unicast (unicast устройства)
- place (вместо этого поля лучше использовать название)
- battery
- unicast_friend (unicast прикрепленного устройства с питанием от батарейки)
- power
- sensor_address
- switcher
- direct
- double
- work_on_schedule
- on_after_start
- count_click (количество кликов на устройстве, чтобы определить популярность)

#### Node_groups - Группы нодов.
Для группировки нодов, имеющих один тип id_node_type
- id_group (AUTO_INCREMENT)
- id_parent (код родительской группы нодов)
- name

#### Node_types - Типы нодов.
Таблица типов устройств. У каждого типа одинаковые параметры
- id_node_type (AUTO_INCREMENT)
- id_icon (код иконки)
- id_icon_on (код иконки при включенном состоянии устройства)
- sensor (флаг сенсор)
- name (название типа устройства)
- name_on (название действия во включенном состоянии)

#### Rooms - Комнаты.
Таблица мест расположения устройств в сети
- id_room (AUTO_INCREMENT)
- id_net (код сети)
- name (название комнаты)

#### Scenario - Сценарии.
Таблица сценариев для управления одним или группой нодов по расписанию или без.
Если создаем график работы ноды без сценария - ставим флаг timetable и работаем так же как и со сценарием
- id_scenario (AUTO_INCREMENT)
- id_net (код сети mash)
- id_icon (код иконки)
- timetable (флаг - это график работы)
- name (название сценария)
- count_click (количество кликов на сценарии для расчета популярности)

#### Schedule - Расписание.
Таблица определяющая временной интервал или график работы устройства. Установка временных интервалов аналогична Сrontab. Код расписания устанавливается для записей в таблице Controls (при работе со сценарием) и Nodes (при работе без сценария)
- id_schedule (AUTO_INCREMENT)
- date_begin (дата начала периода действия расписания)
- date_end (дата окончания периода действия расписания)
- minute (минута запуска сценария)
- hour (час запуска сценария)
- day (день запуска сценария)
- month (месяц запуска сценария)
- dayofweek (дни недели через запятую)
- name

### States - Состояния.
Таблица возможных состояний для нодов разных типов с указанием типа данных
- id_state (AUTO_INCREMENT)
- id_node_type (код типа устройства)
- data_type (тип данных: 0 - bool, 1 - int)
- name (название состояния)
- write (флаг запись разрешена)

#### Triggers - Условия для срабатывания контролируемой ноды.
Таблица нодов, имеющих определенное значение состояния, при условии выполнения которого срабатывает контролируемый нод или группа нодов. При указании сценария, условие для нода относится именно к этому сценарию.
- id_trigger (AUTO_INCREMENT)
- id_node (код ноды)
- id_scenario (код сценария)
- id_state (состояние ноды)
- id_sign (знак условия)
- value (значение состояния устройства)
- name (название условия)

#### Users - Пользователи.
Таблица данных пользователей. Стандартными методами добавить пользователя нельзя. Нужно использовать метод регистрации registration. Также нельзя получить или изменить пароль пользователя и его email, т.к. это логин в системе, потребуется метод смены пароля change_password или новая регистрация в случае со сменой email.
- id_user (AUTO_INCREMENT)
- name
- surname
- email
- phone
