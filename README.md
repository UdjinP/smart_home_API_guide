# smart_home_API_guide
Smart home app API guide

### Структура запроса:
- class - название класса
- action - название метода
- param - параметры метода

После авторизации, пользователь получает токен, который в дальнейшем используется в каждом запросе к БД.
При неактивности более определенного времени (в конфигах = 2ч), токен становится недействительным.
Токен лучше передавать в заголовке запроса, но пока временно он передается вместе с параметрами запроса

Ответ приходит в виде JSON строки

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
Добавить запись в таблицу. В параметрах метода указывается массив с названиями полей и их значениями
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
"data": { "user":{...}, "net":{...}, "nodes":{...}, "scenarios":{...}}

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
