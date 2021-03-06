# Интернет вещей, устройства подключенные к сети. DIY

Движение DIY в электронике началось очень давно, но пик популярности связан с созданием Arduino – платформы, которая объединила программатор вместе с микроконтроллером, свой облегченный язык программирования и простую программу для его компиляции и его загрузки. Тогда как до этого, это все были раздельные вещи.

Но рассматривать эту тему мы будем на основе микроконтроллера ESP8266, нового шага в эволюции Arduino. Революционного продукта китайской промышленности, примерно в 10-15 раз снижающего стоимость подключенности устройства (к WiFi). Приложение, разработанное Arduino-комьюнити теперь умеет компилировать код и под этот микроконтроллер, но до того, как загрузить в наш микроконтроллер первую программу, нужно проделать несколько операций:


### Настройка

0. Установить [Arduino](https://www.arduino.cc/en/Main/Software)

1. Установить [драйвер](http://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers) для ESP8266 (возможно, придется перезагрузить компьютер после установки). Чтобы проверить установился ли драйвер, нужно запустить программу Arduino и попробовать выбрать порт с подключенным устройством. В меню программы: __Tools > Port__

2. Установить дополнительную плату в приложении Arduino:
    - запустить программу Arduino
    - открыть настройки программы __Arduino > Preferences__
    - внизу окна настроек найти поле __'Additional Board Manager URLs'__
    - вставить в это поле ссылку `http://arduino.esp8266.com/stable/package_esp8266com_index.json`
    - нажать OK, сохранить настройки
    - открыть 'Boards Manager' в меню: __Tools > Boards > Boards Manager__
    - ввести в поиск `ESP8266`, нажать на найденное поле и выбрать __Install__
    - выбрать __NodeMCU 1.0__ из списка __Tools > Boards__
    - выбрать порт с подключенным устройством __Tools > Port__
    - открыть код примера __Blink__: __File > Examples > 01.Basics > Blink__
    - загрузить код в микроконтроллер __Sketch > Upload__

3. Установить дополнительную библиотеку PubSubClient через интерфейс приложения. Для этого:
    - в меню выбрать __Sketch > Include Library > Manage Libraries__
    - ввести название библиотеки `PubSubClient` в поиск
    - Нажать на библиотеку в поиске и нажать __Install__

* [Полная инструкция для Windows](https://www.marginallyclever.com/2017/02/setup-nodemcu-drivers-arduino-ide/)
* [Инструкция по установке дополнительной платы](https://learn.sparkfun.com/tutorials/esp8266-thing-hookup-guide/installing-the-esp8266-arduino-addon)


### Режимы работы устройства

- Station - возможность подключения к другм вай-фай сетям
- Soft Access Point - возможность создание собстенной точки доступа
- Station + Soft Access Point - возможность комбинирования обеих вариантов
- Scan - возможность сканирования вай-фай сетей поблизости (их название, сила сигнала и тд)
- Client - возможность делать запросы в интернет
- Server - возможность выступать в роли сервера, то есть принимать и отвечать на http запросы

[Полное описание основных возможностей wifi-библиотеки ESP8266 с картинками и примерами кода](https://arduino-esp8266.readthedocs.io/en/latest/esp8266wifi/readme.html). Ниже мы рассмотрим только две комбинации этих возможностей:


### Работа ESP8266 в качестве точки доступа (Access Point) и сервера

Микроконтроллер можно настроить таким образом, что при подаче питания он будет создавать новую точку доступа с названием, которое вы ему укажете. Устройства, находящиеся рядом, смогут увидеть эту точку доступа в списке WIFI-сетей, и подключится к ней. Кроме того, при запросе к определенному локальному адресу, устройство может отдать обычную html-страницу, и также узнать, что кто-то эту страницу 'открыл'.

- Пустая страница Captive Portal: [http://hackster.io/rayburne/esp8266-captive-portal](https://www.hackster.io/rayburne/esp8266-captive-portal-5798ff).
- Видоизменненный пример с кнопкой, включающей и выключающей светодиод: [./example-esp6266-captive-portal](example-esp6266-captive-portal).


### Работа ESP8266 в качестве Station и MQTT-клиента

В другом случае, микроконтроллер может подключится к существующей сети, и получать или отправлять данные как обычный клиент. Он может делать [запросы к серверам, как обычный браузер](https://github.com/esp8266/Arduino/blob/master/libraries/ESP8266WiFi/examples/WiFiClient/WiFiClient.ino), при этом получая или отправляя данные. Однако в таком режиме сложно передать какую-либо информацию от сервера к устройству, когда устройство просто ждет какого-то действия от сервера. Например, компьютер, проигрывающий видео, может таким образом сообщить о том, что нужно совершить какое-либо действие. Но для начала, нужно написать программу, которая просто подключается к MQTT-брокеру и логирует, присылаемые сообщения.


### MQTT клиент, MQTT брокер

MQTT (Message Queue Telemetry Transport) – это стандарт передачи сообщений для интернета вещей. Спроектирован для этой цели - нестабильного подключения и небольших маломощьных устройств. Построен на моделе подписки и публикации (publish-subscribe). Вся коммуникация происходит через так называемый брокер: сервер, к которому подключаются устройства. Брокеры бывают публичные ([список](http://moxd.io/2015/10/public-mqtt-brokers/)) – их можно использовать бесплатно и свободно, а бывают приватные – их можно установить самостоятельно на свой сервер. На занятии мы будем использовать приватный брокер `mqtt://students.conformity.io` на основе [mosquitto](https://mosquitto.org/download/) – бесплатного брокера с открытым кодом.

Для начала, загрузим код для ESP8266: [example-node-esp8266-video-sync/esp6266-code](example-node-esp8266-video-sync/esp6266-code/esp6266-code.ino). В нем мы подключаемся к локальной WIFI-сети, затем подключаемся к брокеру, и _подписываемся_ на сообщения в канале `/rodchenko/movie/bzz`. При успешном подключении, мы также посылаем сообщение `online` в топик `/rodchenko/movie/status`.

После загрузки кода на устройство, лучше всего открыть _serial monitor_ в ардуино, чтобы проверить подключилось ли устройство. Вы должны будете увидеть что-то похожее:

```
Connecting to WiFi ..... WiFi connected
IP address: XXX.XXX.XXX.XXX
Attempting MQTT connection...failed, rc=-2 try again in 5 seconds
Attempting MQTT connection...connected
```

Интересной особенностью протокола MQTT является поддержка так называемое `Last Will` сообщения (завещения). Это значит, что если устройство вдруг отключится по какой-либо причине от брокера (неполадки в сети, разряженная батарея, поломка), мы узнаем об этом примерно через 20 секунд.

Чтобы получить эти и другие сообщения, отправляемые устройством в node.js, нужно подключится к тому же брокеру и подписаться на сообщения в топике. В данном случае, в коде выше мы задали его как `/rodchenko/movie/status`:

```javascript
var mqtt = require('mqtt')

// подключаемся к MQTT брокеру
var client = mqtt.connect('mqtt://students.conformity.io', {
  // your_unique_client_id нужно поменять на придуманное название вашего устройства
  clientId: 'your_unique_client_id',
  username: 'your_username_for_mqtt_server',
  password: 'your_password_for_mqtt_server'
})

// подписываемся на сообщения в топике '/rodchenko/movie/status'
client.on('connect', function () {
  client.subscribe('/rodchenko/movie/status')
})

// логируем полученные сообщения
client.on('message', function (topic, message) {
  console.log('message', topic, message.toString())
})
```

Чтобы отправлять сообщения, нужно использовать команду `client.publish('topic', 'message')`

```javascript
client.publish('/rodchenko/movie/bzz', '1')
```

Таким образом, можно, например, написать программу, которая в нужные моменты видеофайла отправляет сообщения устройству. Полный пример такой программы: [example-node-video-sync](example-node-video-sync)

Более сложный пример с [входной дверью, подключенной к сети](http://13.conformity.io/), [исходный код](https://github.com/valiafetisov/remote-door).
