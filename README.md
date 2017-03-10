# HTML-виджет расчёта стоимости доставки Marschroute

Данный виджет предназначен для выбора типа доставки и расчёта стоимости доставки заказа. Он может быть легко установлен на внешний сайт и изменен под необходимые условия заказчика. В нем присутствует: выбор города, блок показа всех типов доставки с указанием адреса и даты доставки, карта с показом всех пунктов вывоза заказа. 

Далее представлена инструкция по подключению виджета со списком параметров, которые могут быть изменены.


# Подключение осуществляется добавлением, например, следующего скрипта:

```javascript
<script type="text/javascript">
(function (d, w, c) {
    (w[c] = w[c] || []).push(function() {
        try {
            w.marschrouteWidget = w.marschrouteWidget || new Widget({
            public_key: 'a1s23d45fg67hj89kl', // Публичный ключ клиента !!! Необходимо получить у менеджера Маршрута (требуется указать сайты, с которых будет доступ по данному ключу)
            target_id: 'test' // ID элемента куда будет вставлен виджет            
            });

            w.marschrouteWidget.open({
                weight: 1200, // Вес заказа в граммах
                sum: 2500, // Сумма заказа в рублях
                size: [100,250,550] // Габариты упаковки в мм
            });    
        } catch(e) { }
    });

    var n = d.getElementsByTagName("script")[0],
        s = d.createElement("script"),
        f = function () { n.parentNode.insertBefore(s, n); };
    s.type = "text/javascript";
    s.async = true;
    s.src = "https://marschroute.ru/widgets/delivery/js/widget.js";

    if (w.opera == "[object Opera]") {
        d.addEventListener("DOMContentLoaded", f);
    } else { f(); }
})(document, window, "marschroute_queue");
</script>
```

Необходимо заменить соответствующие параметры инициализации на актуальные. 

При инициализации виджета параметры __public_key__ и __target_id__ - обязательны. Также есть дополнительные опции.

Полный список возможных параметров при инициализации следующий: 
* __public_key__ - Публичный ключ клиента (обязательный параметр);
* __target_id__ - ID элемента на странице для размещения виджета (обязательный параметр);
* __width__ - Ширина виджета (по умолчанию 1024);
* __height__ - Высота виджета (по умолчанию 602);
* __slim__ - Мобильный вид виджета (по умолчанию false);
* __currency_symbol__ - Обозначение валюты (по умолчанию "р.");
* __submit_title__ - Текст для кнопки подтверждения выбора (по умолчанию "Подтвердить выбор доставки")
* __city_id__ - КЛАДР города по умолчанию 

В случае, если параметры __width__ и __height__ не указаны, размеры виджета устанавливаются соответственно размерам родительского элемента (элемента с указанным в опциях ID). Однако, в любом случае, размеры не могут быть меньше 768 x 602 пикселей, иначе вычисленные или заданные в настройках размеры игнорируются и устанавливаются минимально возможные (768x602).

Параметр __slim__ может принимать значение false либо true и связан с параметром __width__.  
Если ширина указана, то __slim__ надо указывать явно.  
Если ширина и опция __slim__ не указаны, то виджет ориентируется на ширину слоя, в котором он находится. При ширине слоя меньшей 768 пикселей опция __slim__ автоматически принимает значение true.

Скрипт должен выполняться на момент, когда страница полностью загружена (элемент для вставки виджета существует), 
поэтому можно разместить его либо в конце страницы перед тэгом </body>, либо в обработчике window.onload

Непосредственно открытие окна виджета выполняется функцией __open()__.  
Функция может вызываться как после инициализации объекта виджета, так и быть вызвана в одном из обработчиков элементов страницы, например при нажатии на кнопку.  
Также при открытии окна есть возможность указать параметры веса, суммы и габаритов заказа (указаны в примере выше в квадратных скобках), которые также влияют на стоимость доставки.  
Также для корректной инициализации виджета важно предотвратить многократное создание его экземпляров на странице (как выяснилось, это может привести к проблемам, при интеграции в Bitrix, ранее созданные экземпляры не удалялись из-за специфики выбора службы доставки), поэтому при инициализации рекомендуется следующая конструкция:
```javascript 
w.marschrouteWidget = w.marschrouteWidget || new Widget({options}); 
```

 
# Обработка результата

Для того чтобы обработать результат взаимодействия пользователя с виджетом необходимо при инициализации в опциях задать функцию обработки результата (ключ onSubmit).   
Сделать это можно, например, следующим образом:
```javascript 
w.marschrouteWidget = w.marschrouteWidget || new Widget({
    public_key: 'a1s23d45fg67hj89kl',
    target_id: 'test',
    onSubmit: function (delivery, widget) {
        // что-нибудь делаем
        var info = "";
        for (var k in delivery) {
            info += (k+": "+delivery[k]+", \n");
        }
        alert(info);
        widget.destroy();
    }    
});
```

Функция принимает 2 аргумента: 

1. Объект delivery;
2. Экземпляр созданного виджета.

Объект __delivery__ имеет следующую структуру:
```javascript 
{
place_id: 1099, // ID выбранной доставки
city_id: "77000000000", // Город (КЛАДР)
city_name: "Москва", // Название города
delivery_cost: 350, // Получившаяся стоимость доставки
weight: 1200, // Вес заказа в граммах (если был передан при инициализации)
sum: 2500, // Сумма заказа (если была передана при инициализации)
size: [100, 250, 550], // Габариты заказа (если были переданы при инициализации),
address: "Кантемировская", // Улица
mkad: 1, // Флаг мкад/за мкад, для доставки по Москве (1 - в пределах МКАД, 2 - за МКАД)
post_index: 115404, // Почтовый индекс
delivery_date: "09.11.2016", // Выбранная дата доставки
delivery_time: 0, // ID выбранного интервала доставки
building_1: 58, // Номер дома
building_2: 1, // Номер строения или корпуса
room: 115, // Номер квартиры
metro_id: 26, // ID выбранной станции метро
comment_user: "Позвоните перед приездом" // Комментарий пользователя
name: "Курьерская служба", // Название типа доставки 
delivery_code: "NASHA55", // Код доставки 
delivery_id: 1, // ID типа доставки 
payment_types: [1,2], // ID доступных способов оплаты 
comment_address: "Проезд: от метро выход - первый вагон из центра, из стеклянных дверей налево...", // Комментарий к адресу ПВЗ 
gps: ["55.78282","37.457259"], // Координаты ПВЗ 
payment_for_terminal: 0, // Признак возможности оплаты с терминала (для ПВЗ) 
fitting_room: 1, // Признак наличия примерочной (для ПВЗ) 
phone: "+7 (985) 618-50-01", // Телефон (для ПВЗ) 
schedule: "пн-пт 10-20, сб-вс 10-18" // График работы (для ПВЗ) 
}
```

Как видно, список полей может меняться в зависимости от выбранного типа доставки. 

Экземпляр виджета может использоваться для вызова метода __destroy()__ - который удаляет созданный виджет.
 
# Изменение выходных данных

На данный момент изменение выходных данных допустимо для двух полей:

1. delivery_date  
При этом оно должно соответствовать следующим условиям:

  * Не может быть меньше, чем delivery_date по умолчанию;  
  * Должно быть в массиве possible_dates, если он не пуст (например, для курьерской службы). В этом случае из массива possible_dates будут удалены все более ранние, по сравнению с переданной, даты.
  
  Если будет передан неправильный delivery_date, то изменения не будут применены и в виджете отобразится дата доставки по умолчанию.

2. delivery_cost  
Возможно изменение цены доставки отображаемой для пользователя в виджете по заранее заданному алгоритму.
Изменение цены задается также в опциях специальной функцией __filter__.    
Например, таким образом можно для всех типов доставки отображать цену на 10% больше:

```javascript 
w.marschrouteWidget = new Widget({
    public_key: 'a1s23d45fg67hj89kl',
    target_id: 'test',
    filter: function (delivery_item) {
        delivery_item.delivery_cost = delivery_item.delivery_cost * 1.1; // + 10%
        return delivery_item; 
    }    
});
```

В функцию передается объект delivery_item с информацией о конкретном способе доставки или пункте самовывоза, который может содержать следующие поля:
```javascript 
{
"name": "Самовывоз Boxberry", // Имя способа доставки,
"delivery_code": "BOXBERRY_M", // Внутренний код доставки
"delivery_id": 2, // ID типа доставки (1 - Курьерская доставка, 2 - Самовывоз, 3 - Почта)
"delivery_cost": 115, // Рассчитанная стоимость доставки
"payment_types": [1,2], // Доступные типы оплаты
"place_id": 10429, // ID пункта самовывоза
"delivery_date": "10.11.2016", // Минимальная дата доставки
"possible_dates": ["10.11.2016", "11.11.2016", "12.11.2016"], // Массив возможных дат доставки (для курьерских доставок)
"comment_address" : "Проезд: метро Сходненская. Последний вагон из центра. Выход с платформы в переход налево.", // Подробное описание пункта самовывоза
"gps": ["55.848685","37.440775"], // Координаты пункта самовывоза
"address": "Сходненская ул, д.52, корпус 1. Метро Сходненская. Boxberry", // Адрес пункта самовывоза
"payment_for_terminal": 0, // Признак возможности оплаты картой
"fitting_room": 0, // Признак наличия примерочной
"phone": "8-499-272-69-44", // Телефон пункта самовывоза
"schedule": "пн-вс: 09.00-20.00", // Режим работы пункта самовывоза
"image": "" // URL картинки с фото пункта самовывоза
}
```
Также при помощи функции filter можно удалять из виджета определенные способы доставки. Для этого необходимо вместо измененного объекта delivery_item возвращать любое другое значение, не являющееся объектом, например, false.  
Например, для удаления всех пунктов самовывоза функция filter может быть такой:
```javascript 
    filter: function (delivery_item) {
        if (delivery_item.delivery_id == 2) {
            return false;
        }
        return delivery_item;
    } 
```
 
# Подключение с помощью JQuery

В случае использования JQuery минимальная инициализация виджета может быть такой:
```javascript 
$(function() {
    var widget = new Widget({
        public_key: 'a1s23d45fg67hj89kl',
        target_id: 'test',
        onSubmit: function (delivery, widget) {
            widget.destroy();
        }
    });

    $('button').click(function() {
        widget.open();
        return false;
    });
});
```

При этом стоит не забывать подключать на странице скрипт с виджетом перед инициализацией
```javascript
<script src="https://marschroute.ru/widgets/delivery/js/widget.js" type="text/javascript"></script>
```
