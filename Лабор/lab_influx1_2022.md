**Технології індустрії 4.0. Лабораторний практикум. ** Автор і лектор: Олександр Пупена 

| [<- до лаборних робіт](README.md) | [на основну сторінку курсу](../README.md) |
| --------------------------------- | ----------------------------------------- |
|                                   |                                           |

# Лабораторна робота №8. Influx DB. Збір даних та основи роботи з вибіркою даних.
Завдання лабораторної роботи навчитися писати дані в базу даних часових рядів Influxdb та ознайомитися з основами роботи з інтерфейсом користувача Influxdb (Influxdb UI) з метою налаштування бази даних, побудови запитів та перегляду даних часових рядів.

### 1.Створення безкоштовного облікового запису InfluxDB Cloud
- [ ] Перейдіть на сторінку [реєстрації InfluxDB Cloud](https://cloud2.influxdata.com/signup) та заповніть форму або використайте Google чи Microsoft акаунти.
![](6_1media/1.PNG) 

- [ ] Оберіть будь-якого постачальника хмарних послуг і регіон, близький до вашого місця розташування. 
![](6_1media/2.PNG) 

- [ ] Оберіть план використання InfluxDB Cloud – **free tier**.
![](6_1media/3.PNG) 

### 2.Створення нового бакету

**Бакет** – це іменована локація зберігання даних часових рядів. Усі бакети мають період збереження, тривалість часу, протягом якого кожен елемент  даних зберігається. InfluxDB скидає всі елементи даних з мітками часу, старшими за період зберігання бакету. Бакет належить організації.

- [ ] Відкрийте пункт меню "Load data" –> Buckets.
- [ ] Натисніть **+ Create Bucket** у верхньому правому куті.
- [ ] Введіть назву `firstbucket`. У плані використання InfluxDB Cloud **free tier** максимальний період збереження 30 днів, тому налаштування **Delete Data** залиште за замовчуванням.
- [ ] Натисніть **Create**.



![](6_1media/1.gif) 

### 3. Створення токену повного доступу

**Токени API InfluxDB** забезпечують безпечну взаємодію між користувачами та даними. Токен належить організації та визначає дозволи InfluxDB в організації. Токен – це ключ до даних (бакетів, файлів телеграфу). Influxdb надає два типи токенів: [All-Access token](https://docs.influxdata.com/influxdb/cloud/security/tokens/#all-access-token), [Read/Write token](https://docs.influxdata.com/influxdb/cloud/security/tokens/#readwrite-token). 

All-Access token – надає повний доступ для читання та запису до всіх ресурсів організації.

Read/Write token – надає доступ для читання, запису або і те й інше до певних бакетів в організації.

- [ ] Відкрийте пункт меню "Load data" –> API Tokens.
- [ ] Натисніть **+ Generate API Token**.
- [ ] Виберіть **All Access API Token**.
- [ ] Напишіть опис токену в полі **Description** , наприклад, `all access token`.
- [ ] Скопіюйте згенерований токен (довгий набір символів), натиснувши кнопку **Copy To Clipboard**.
- [ ] **Обов'язково збережіть скопійований токен** для подальшого використання у лабораторній роботі. Токен доступний для копіювання лише після створення.

![](6_1media/2.gif) 

### 4. Налаштування запису даних з OPC UA сервера в influxdb

У цьому пункті ви спробуєте два способи збору та запису даних в InfluxDB. Перший спосіб (4.1) передбачає збір та запис даних з використанням Node-RED та спеціальних вузлів бібліотеки. Другий спосіб (4.2) - використовуючи службу Telegraf. 

#### 4.1. Запис даних за допомогою Node red

- [ ] Запустіть тестовий сервер UaCPPServer, з яким ви працювали в Лабораторній роботі №4 – [Робота з OPC UA](lab_opcua.md) 
- [ ] Запустіть Node-RED. Встановіть модуль `node-red-contrib-stackhero-influxdb-v2`.
- [ ] Можете продовжити працювати в проекті Node-Red з лабораторної роботи №4 [Робота з OPC UA](lab_opcua.md) або створити новий проект. Імпортуйте код нового потоку:
```json
[{"id":"760f23939087cb96","type":"tab","label":"OPC UA to influxdb","disabled":false,"info":"","env":[]},{"id":"ab472f9b460178a2","type":"function","z":"760f23939087cb96","name":"multiplesubscribe","func":"msg.payload=10000;\nfor(i=1;i<11;i++){\n   msg.topic=`ns=3;s=AirConditioner_${i}.Temperature`;\n   node.send(msg);\n    }\n","outputs":1,"noerr":0,"initialize":"","finalize":"","libs":[],"x":290,"y":80,"wires":[["a21fad02e87d5aa8"]]},{"id":"c8770202d16a3012","type":"inject","z":"760f23939087cb96","name":"","props":[{"p":"payload"},{"p":"topic","vt":"str"}],"repeat":"","crontab":"","once":true,"onceDelay":0.1,"topic":"","payloadType":"date","x":110,"y":80,"wires":[["ab472f9b460178a2"]]},{"id":"c79ed1b84a742571","type":"debug","z":"760f23939087cb96","name":"","active":false,"tosidebar":true,"console":false,"tostatus":false,"complete":"true","targetType":"full","statusVal":"","statusType":"auto","x":1090,"y":200,"wires":[]},{"id":"a21fad02e87d5aa8","type":"OpcUa-Client","z":"760f23939087cb96","endpoint":"501a96153c7c8d57","action":"subscribe","deadbandtype":"a","deadbandvalue":1,"time":"10","timeUnit":"s","certificate":"n","localfile":"","localkeyfile":"","securitymode":"None","securitypolicy":"None","folderName4PKI":"","name":"","x":500,"y":80,"wires":[["1ef5bdfa8f62c7a1"]]},{"id":"d3727b6cd9633b8f","type":"Stackhero-InfluxDB-v2-write","z":"760f23939087cb96","server":"","name":"InfluxDBwrite","x":1010,"y":80,"wires":[["c79ed1b84a742571"]]},{"id":"24c0e70e64d3ab41","type":"join","z":"760f23939087cb96","name":"","mode":"custom","build":"array","property":"payload","propertyType":"msg","key":"topic","joiner":"\\n","joinerType":"str","accumulate":false,"timeout":"","count":"10","reduceRight":false,"reduceExp":"","reduceInit":"","reduceInitType":"num","reduceFixup":"","x":735,"y":80,"wires":[["769db09ab15b6d24"]],"l":false},{"id":"1ef5bdfa8f62c7a1","type":"switch","z":"760f23939087cb96","name":"","property":"topic","propertyType":"msg","rules":[{"t":"cont","v":"ns=3;s=AirConditioner","vt":"str"}],"checkall":"true","repair":false,"outputs":1,"x":615,"y":80,"wires":[["53fdf054dc0c9e9f"]],"l":false},{"id":"53fdf054dc0c9e9f","type":"function","z":"760f23939087cb96","name":"","func":"let str=\"\";\nlet obj={};\nstr=msg.topic.replace(\"ns=3;s=\",\"\");\nobj.device=str.replace(\".Temperature\",\"\");\nobj.fields={Temperature:msg.payload};\nobj.timestamp=msg.serverTimestamp;\nmsg={};\nmsg.payload=obj;\nreturn msg;","outputs":1,"noerr":0,"initialize":"","finalize":"","libs":[],"x":675,"y":80,"wires":[["24c0e70e64d3ab41"]],"l":false},{"id":"769db09ab15b6d24","type":"function","z":"760f23939087cb96","name":"msgtoinflux","func":"let datastore=[];\nclass sample {\n  constructor(payload) {\n    this.data={\n        measurement:'Building',\n        tags: {\n            device: payload.device,\n        },\n        fields: payload.fields,\n        timestamp: new Date(payload.timestamp).getTime()\n    }\n  }\n}\nfor (let i=0; i<msg.payload.length;i++){\n    dataobj=new sample(msg.payload[i]);\n    datastore.push(dataobj.data);\n}\nmsg.payload = {\n    bucket:'firstbucket',\n    precision: 'ms',\n    data:datastore,\n};\nreturn msg;","outputs":1,"noerr":0,"initialize":"","finalize":"","libs":[],"x":850,"y":80,"wires":[["d3727b6cd9633b8f"]]},{"id":"501a96153c7c8d57","type":"OpcUa-Endpoint","endpoint":"","secpol":"None","secmode":"None","login":false,"credentials":{}}]
```


- [ ] Налаштуйте вузол `OPC UA Client`, прописавши відповідний URL.

![](6_1media/4.png) 

- [ ] Для вузла InfluxDBwrite необхідно налаштувати джерело даних, в яке будуть записуватися значення температур 10 кондиціонерів кожні 10 секунд. Цим джерелом буде бакет  influxdb "firstbucket", створений в пункті 2. Нижче рисунка наведено опис кожного поля вузла.

![](6_1media/5.png) 

- [ ] Значення властивості `Host` у випадку використання хмарної версії influxdb можна подивитися у загальних налаштуваннях організації в інтерфейсі користувача influxdb cloud. Пункт меню акаунт (email) ->  Organization -> Settings -> Cluster URL. Наприклад, `westeurope-1.azure.cloud2.influxdata.com`, вказувати `https://` **не потрібно**.

![](6_1media/3.gif) 

- [ ] Значення властивості `Port` – прописуємо стандартний https port – `443`.
- [ ] В поле `Token` – вставляємо рядок скопійований та збережений в пункті 3 при створені токену повного доступу. 
- [ ] Значення властивості `Organization` можна подивитися у загальних налаштуваннях організації в інтерфейсі користувача influxdb cloud. Пункт меню акаунт (email) ->  Organization -> Settings -> Organization ID. 

![](6_1media/4.gif) 

- [ ] `Default bucket` – назва бакету "firstbucket", створеного в пункті 2.
- [ ] Зверніть увагу на код вузла **msgtoinflux**. Значенню властивості об'єкта msg.payload bucket повинна відповідати назва бакету, створеного в пункті 2.
![](6_1media/6.png) 
- [ ] Задеплойте новий потік. Якщо всі налаштування були зроблені правильно, то у вікні відладки кожні 10 секунд приходитиме повідомлення з пустим рядком `""`. 


#### 4.2. Запис даних за допомогою Telegraf
Telegraf – серверний агент, який збирає та надсилає метрики з різних джерел даних (баз даних, систем та IoT). Перед тим як збирати дані з OPC UA server за допомогою Telegraf, необхідно його інсталювати на машину, на якій ви запускаєте UaCPPServer. Для цього завантажте Telegraf. Якщо ви працюєте з 64-розрядною операційною системою  Windows, тоді запустіть Windows Power Shell від **імені адміністратора** та виконайте команди: 

```powershell
wget http://dl.influxdata.com/telegraf/releases/telegraf-1.22.1_windows_amd64.zip -UseBasicParsing -OutFile telegraf-1.22.1_windows_amd64.zip
```

```powershell
Expand-Archive .\telegraf-1.22.1_windows_amd64.zip -DestinationPath 'C:\Program Files\InfluxData\telegraf'
```

Тепер Telegraf зберігається в папці `C:\Program Files\InfluxData\telegraf`.

Якщо ви працюєте з іншою операційною системою, тоді слідуйте вказівкам із сторінки [завантажень InfluxDB](https://portal.influxdata.com/downloads/), обравши Telegraf v1.22 та відповідну платформу. 

- [ ] Створіть конфігурацію для Telegraf в інтерфейсі користувача InfluxDB cloud. Пункт меню "Load Data" -> Telegraf -> Create Configuration.


![](6_1media/5.gif) 

- [ ] Оберіть попередньо створений бакет у випадальному списку "firstbucket".

- [ ] В полі пошуку плагінів для телеграфу, напишіть "opcua". Це прискорить пошук вхідного плагіну, який дозволяє налаштувати телеграф для передачі даних із OPC UA серверу в бакет influxdb.
- [ ] Оберіть плагін "OPC UA Client". Натисніть кнопку "Continue Configuring".

![](6_1media/6.gif) 

- [ ] Дайте назву конфігурації, наприклад, `OPC UA server`.
- [ ] Сконфігуруйте плагін, прибираючи потрібні параметри (прибрати "#") та змінюючи їх значення. Зокрема змініть конфігурацію вхідного плагіну (`inputs`) як це показано на рисунку нижче. Враховуйте, що значенню параметра `endpoint` відповідає URL OPC UA сервера. Значення параметру Nodes скопіюйте звідси: 

```json
[
   {name="Humidity", tags=[["device", "AirConditioner_1" ]], identifier="AirConditioner_1.Humidity"},
   {name="Humidity", tags=[["device", "AirConditioner_2" ]], identifier="AirConditioner_2.Humidity"},
   {name="Humidity", tags=[["device", "AirConditioner_3" ]], identifier="AirConditioner_3.Humidity"},
   {name="Humidity", tags=[["device", "AirConditioner_4" ]], identifier="AirConditioner_4.Humidity"},
   {name="Humidity", tags=[["device", "AirConditioner_5" ]], identifier="AirConditioner_5.Humidity"},
   {name="Humidity", tags=[["device", "AirConditioner_6" ]], identifier="AirConditioner_6.Humidity"},
   {name="Humidity", tags=[["device", "AirConditioner_7" ]], identifier="AirConditioner_7.Humidity"},
   {name="Humidity", tags=[["device", "AirConditioner_8" ]], identifier="AirConditioner_8.Humidity"},
   {name="Humidity", tags=[["device", "AirConditioner_9" ]], identifier="AirConditioner_9.Humidity"},
   {name="Humidity", tags=[["device", "AirConditioner_10" ]], identifier="AirConditioner_10.Humidity"},                        
  ]
```

![](6_1media/7.png) 
![](6_1media/8.png) 

- [ ] Натисніть кнопку "Save and Test".


![](6_1media/9.png) 

- [ ] З пункту "Configure your API Token" (вікно Test your configuration) скопіюйте згенерований токен в якийсь текстовий файл для подальшого налаштування доступу до конфігураційного файлу та бакету. Вікно конфігурації також можна буде відкривати пізніше через пункт "Setup Instruction" конкретного налаштування Telegraf.  


![](6_1media/7.gif) 

- [ ] У операційній системі вашого ПК створіть змінну середовища `INFLUX_TOKEN`, яка буде зберігати токен доступу, який ви щойно скопіювали. Якщо ви працюєте на ОС Windows: Панель керування -> Система -> Додаткові параметри системи -> Змінні середовища -> Змінні середовища користувача -> Створити. Ім'я змінної – "INFLUX_TOKEN", значення змінної – скопійований токен.


![](6_1media/10.png) 

- [ ] З пункту "Start Telegraf" (вікно Test your configuration) скопіюйте команду для запуску телеграфа. Атрибут --config вказує на шлях до конфігураційного файлу, який в даному випадку зберігається в influxdb cloud.


![](6_1media/8.gif) 

- [ ] Натисніть **Finish**.

За замовчуванням інтервал запису даних, зазначений у конфігураційному файлі телеграфу, дорівнює 20 секунд. Щоб його змінити, потрібно відкрити конфігураційний файл в інтерфейсі користувача Influxdb: пункт меню "Load data" –> Telegraf, обрати необхідну конфігурацію (в нашому випадку `OPC UA server`). Параметр, який відповідає за інтервал запису, називається "Interval". 
![](6_1media/26.png) 

Змінювати інтервал не потрібно, залиште конфігураційний файл без змін, натиснувши кнопку **Cancel**.

- [ ] Перейдіть в директорію, де зберігається telegraf – `C:\Program Files\InfluxData\telegraf\telegraf-1.22.1`. Запустіть з папки командний рядок, прописавши в шлях провідника "cmd". В командному рядку виконайте команду для запуску телеграфу, скопійовану у попередньому пункті.  Повинно 


![](6_1media/9.gif) 

Серед повідомлень у вас зявиться попередження, на яке не потрібно звертати увагу:

```
W! [inputs.opcua] Failed to load certificate: open /etc/telegraf/cert.pem: The system cannot find the path specified.
```



### 5. Робота з інтерфейсом користувача Influxdb

Інтерфейс користувача InfluxDB UI це інструмент для роботи з даними часових рядів та InfluxDB. Інтерфейс користувача InfluxDB дозволяє:
- створювати запити для візуалізації даних часових рядів. Для вибору доступний широкий спектр типів візуалізації.
- редагувати код [Flux](https://docs.influxdata.com/influxdb/cloud/query-data/get-started/) (функціональна мова, розроблена спеціально для роботи з форматом даних InfluxDB) у редакторі скриптів Flux.
- створювати інформаційні панелі – dashboards, та блокноти – notebooks.
- керувати кількома користувачами організації.
- cтворювати та керувати завданнями – [tasks](https://docs.influxdata.com/influxdb/cloud/process-data/get-started/) (скрипт Flux, який виконується за розкладом; скрипт приймає потік вхідних даних, модифікує або аналізує його якимось чином, потім записує змінені дані назад до InfluxDB або виконує інші дії).
- cтворювати перевірки – checks та сповіщення – notifications. Перевірки та сповіщення – це спеціальні типи завдань (task), які дозволяють створювати аварійні повідослення – alerts.

#### 5.1. Основи роботи з вибіркою даних. Data Explorer
Data Explorer в інтерфейсі користувача InfluxDB дозволяє створювати, виконувати та візуалізувати запити Flux. **Query Builder** (конструктор запитів) дозволяє створювати Flux запити без написання коду на мові Flux, а **Script Editor** (редактор скриптів) – це редактор коду Flux, де можна вручну редагувати запит. При переході на сторінку Data Explorer, за замовчуванням завантажується Конструктор запитів.

- [ ] У інтерфейсі користувача хмарного сервісу InfluxDB відкрийте пункт меню "Data Explorer". За допомогою Конструктора запитів побудуйте запит зчитування значень температур за останні 15 хвилин трьох довільних кондиціонерів:
  1. В розділі From оберіть бакет, в який пишуться дані.
  2. В наступному розділі "Filter" (автоматично обрано тег "\_measurement" ) оберіть вимірювання **Building**.
  3. В наступному "Filter" (автоматично обрано тег "\_field" ) оберіть поле **Temperature**.
  4. В наступному "Filter" (автоматично обрано тег "device" ) оберіть три довільні кондиціонери.
  5. В полі вибору часового діапазону **Time range** оберіть останні 15 хв "Past 15m".
  6. Натисніть кнопку **Submit** для виконання сконфігурованого запиту.
  7. Проаналізуйте лінійний тренд (тип візуалізації Graph за замовчуванням).


![](6_1media/11.png) 


![](6_1media/10.gif)

- [ ] Змініть період та [функцію агрегування](https://docs.influxdata.com/flux/v0.x/function-types/#aggregates) значень: у віконці "Window period" оберіть `Custom` та виставте `30s`;  у віконці "Aggregate functions" оберіть `Custom` та виставте функцію `min`. Після цього натисніть `Submit` щоб подивитися на результати. Проекспериментуйте з різними функціями агрегування.


![](6_1media/12.png) 


- [ ] Перегляньте та проаналізуйте код побудованого запиту на мові Flux, натиснувши кнопку `SCRIPT EDITOR`. Основи мови Flux буде розглянуто в другій частині лабораторної роботи присвяченої InfluxDB.


![](6_1media/13.png) 

- [ ] Перейдіть назад у конструктор запитів, натиснувши кнопку **Query Builder**.
- [ ] Перегляньте дані у табличному вигляді вибравши опцію **View Raw Data**. Після чого поверніться в режим відключено опції. 
- [ ] Data Explorer надає різні [типи візуалізації даних](https://docs.influxdata.com/influxdb/cloud/visualize-data/visualization-types/). Перегляньте наявні візуалізації (Data Explorer –> випадальний список у верхньому лівому куті) та їх налаштування (кнопка **Customize**) .
![](6_1media/27.png) 

#### 5.2. Налаштування дашбоардів (інформаційних панелей)

Дашбоарди - інформаційні пенелі дають можливість надати наперед сконфігуровані панелі перегляду для користувача.   

- [ ] Створіть новий Dashboard з назвою "Building":
  1. У навігаційному меню ліворуч виберіть **Dashboards**.

   ![](6_1media/14.png) 
  
  2. Натисніть **+ Create Dashboard** у верхньому правому куті та виберіть **New Dashboard**.
  3. Введіть назву для своєї інформаційної панелі в поле **Name this dashboard** у верхньому лівому куті.

![](6_1media/11.gif)

- [ ] Додайте нову комірку з лінійним трендом "Humidity" для першого кондиціонера. Для цього натисніть **Add Cell**, щоб відкрити Data Explorer і налаштувати нову комірку для інформаційної панелі.

- [ ] Побудуйте запит на читання значень "Humidity" одного кондиціонера "AirConditioner_1" за останні 15 хвилин, не застосовуючи функцію агрегування. Натисніть `Submit`.


   ![](6_1media/15.png) 

- [ ] Використовуючи кнопку `Customize` налаштуйте відображення тренду : 
  1. Згладжування тренду (опція Interpolation -> Smooth)
  1. Заповнення простору під лінією, (опція "Shade Area Below Lines").
  1. Назва осі Y (Y axis label) `Humidity`.
  1. Одиниці вимірювання `%` на осі Y (опція "Y Axis Suffix").
  1. Статичне відображення легенди (опція "Display Static Legend" - виставити Show).

![](6_1media/12.gif)

- [ ] Дайте назву комірці –  `Humidity`, вписавши її у полі `Name this cell` у лівому верхньому кутку.
- [ ] Збережіть налаштування комірки, натиснувши "Галочку" у верхньому правому куті.
![](6_1media/16.png) 
- [ ] Розтягніть комірку у всю ширину сторінки.

![](6_1media/13.gif)

- [ ] Вручну оновіть Dashboard.


![](6_1media/19.png) 

- [ ] Налаштуйте автоматичне оновлення  Dashboard кожні 30 секунд (опція "Set Auto Refresh"). Після чого натисніть `Confirm` 

![](6_1media/17.png) 

![](6_1media/18.png) 

- [ ] Використовуючи інструкції нижче створіть змінну (Dashboard Variable) з назвою `device`  для зміни конкретних компонентів запитів без необхідності редагування запитів. Ця змінна буде повертати всі значення тегу device, тобто "AirConditioner_1", "AirConditioner_2", "AirConditioner_3" і т.д. 
  1. Відкрийте Data Explorer. 
  1. Перейдіть в Script Editor і в редакторі скриптів вставте наступний запит на мові Flux:

```js
import "influxdata/influxdb/v1"
v1.tagValues(bucket: "firstbucket", tag: "device", predicate: (r) => true, start: -30d)
```

![](6_1media/14.gif)

  3. Збережіть запит (`Save As`) як змінну (`Variable`) з назвою `device` та типом `query`. 

![](6_1media/15.gif)

Створювати зміні можна також в пункті меню "Settings" -> Variables.

- [ ] За допомогою редактора скриптів змініть запит комірки "Humidity" дашбоарда "Building", щоб запит формувався динамічно з урахуваннями обраного кондиціонера. Для цього 

  1. Перейдіть у пункт меню "Dashboards".

  2. Відкрийте dashboard "Building" .

  3. Відкрийте редагування комірки "Humidity" (шестерня у правому верхньому куті -> Configure). 

   ![](6_1media/20.png)

  4. У редакторі скриптів `"AirConditioner_1"` (разом з лапками) замініть на `v.device` і натисніть `Submit`.

  5. Збережіть зміни (галочка).

![](6_1media/16.gif)

- [ ] Активуйте опцію "Show Variables". Якщо в запитах комірок дашбоарда використовуються змінні, то випадальні списки (результат виконання запиту змінної) будуть відображатися над комірками. Це дозволить підставляти у запити різні кондиціонери в залежності від ручного вибору.

![](6_1media/21.png)

- [ ] **Самостійно!** Створіть ще одну аналогічну комірку з назвою "Temperature" для відображення тренду температури кондиціонерів. Можете це зробити з використанням клонування (в налаштуваннях комірки є така команда)

- [ ] Створіть комірку з назвою `Current Humidity` типу "Gauge" для відображення поточного значення вологості. Код запиту:
```flux
from(bucket: "firstbucket")
  
 |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  
 |> filter(fn: (r) => r["_measurement"] == "Building")
  
 |> filter(fn: (r) => r["_field"] == "Humidity")
  
 |> filter(fn: (r) => r["device"] == v.device)
```

- [ ] Налаштуйте комірку наступним чином:


![](6_1media/22.png)

- [ ] **Самостійно!** Створіть ще одну аналогічну комірку з назвою `Current Temperature` для відображення поточного значення температури.
- [ ] Перемкніть Dashboard в повноекранний режим. Вийти з повноекранного режиму – `ESC`.
![](6_1media/23.png)
- [ ] Експортуйте Dashboard (Пункт меню "Dashboards" –> шестерня у правому верхньому куті плитки дашбоарду -> Download Template).
![](6_1media/24.png)
- [ ] Імпортуйте Dashboard (Пункт меню "Dashboards" –> **+ Create Dashboard** у правому верхньому куті -> Import Dashboard).
![](6_1media/25.png) 



# Питання до захисту. 

1. Що таке бакет Influxdb?
1. Де можна знайти конфігураційні файли телеграфу в Influxdb UI?
1. Для чого потрібні змінні дашбоардів?
1. Як створити дашбоард?
1. Як створити змінну дашбоарда?
1. Як екпортувати дашбоард? 

