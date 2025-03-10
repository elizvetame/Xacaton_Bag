# Android + Callibri + ECG using KotlinCompose

## Структура проекта

Проект состоит из трех основных файлов:
1. файл интерфейса [MainScreen.kt](https://gitlab.com/neurosdk2/cybergarden2024/-/blob/main/school/AndroidSample/app/src/main/java/com/example/callibriecgdemo/MainScreen.kt?ref_type=heads). В нем ничего интересного
2. контроллер интерфейса [MainScreenViewModel.kt](https://gitlab.com/neurosdk2/cybergarden2024/-/blob/main/school/AndroidSample/app/src/main/java/com/example/callibriecgdemo/MainScreenViewModel.kt?ref_type=heads). В нем же находится методы связи интерфейса и адаптера
3. самый важный файл [CallibriController.kt](https://gitlab.com/neurosdk2/cybergarden2024/-/blob/main/school/AndroidSample/app/src/main/java/com/example/callibriecgdemo/CallibriController.kt?ref_type=heads)

В проекте используется библиотека `Hilt` для более простого внедрения механизма внедрения зависимостей. Она необязательна для использования, но очень удобна.

**CallibriAdapter** - это основной класс, на который нужно обратить внимание. В нем происходит все взаимодействие с девайсом Callibri, а так же получение ЧСС из дополнительной библиотеки. Каждый метод ожидает одним из аргументов мак-адрес устройства. Это означает, что объект **CallibriAdapter** хранит в себе все подключенные пользователем устройства и различает их по мак-адресу. Все оповещения помимо основной информации так же отправляют мак-адрес устройства, от которого пришло это оповещение. Этот класс должен быть синглтоном, поэтому он помечен как синглтом аннотацией `@Singleton`

#### Поиск устройства

Поиск представлен методом **startSearchWithResult** с аргументами:

1. время поиска. Указывается в секундах. Тип данных **int**
2. список мак-адресов устройств для поиска. Если передать пустой список - найдутся все девайся типа Callibri. Тип данных - **List\<String\>**.

Эта функция синхронна, но не блокирует интерфейс. Возвращает список найденных устройств.

Как использовать:

```kt
val founded = adapter.startSearchWithResult(10, emptyList())
``` 

В примере кода показан поиск любого девайса в течении 10 сек.

Список найденных девайсов приходит в списке типа **List\<CallibriInfo\>**. **CallibriInfo** содержит два значимых поля:
 1. Имя девайса. Тип данных **String**
 2. Мак-адрес девайса. Тип данных **String**


#### Подключение к устройству

Для подключения используется метод **connectTo**. Метод синхронный, но ничего не возвращает, поэтому о статусе подключения можно узнать от соответствующего колбека. Метод принимает следующие аргументы:

 1. Информацию об устройстве. Тип **CallibriInfo**
 2. Нужно ли устройство переподключать при отключении. Тип **bool**

Если вторым аргументом передано true:
 1. при незапланированном отключении девайса он подключится обратно
 2. если во время отключения было запущено сопротивление - его нужно будет включать отдельно
 3. если во время отключения был запущен сигнал - он включится самостоятельно, никаких действий предпринимать не нужно

Состояние уже подключенного девайса можно так же получить с помощью колбека **connectionStateChanged**.

1. подписаться на колбек:
    ```kt
    val adapter: CallibriAdapter // сохраненный адаптер девайса
    var _selectedDevice = "" // здесь нужно ввести адрес выбранного каллибри

    adapter.connectionStateChanged = {addr, state ->
        if(addr == _selectedDevice) { }
    }
    ``` 

2. подключиться к девайсу:

    ```kt
    val selectedCallibri =// получить CallibriInfo из списка найденных устройств
    adapter.connectTo(selectedCallibri, true)
    ```

#### Получение ЧСС

ЧСС является числом типа **Double**. Это значение приходит не сразу, а после набора определеннгого количества данных. По окончанию набора данных библиотека оповестит колбеком **signalQuality**.

Чтобы получить ЧСС нужно:

1. подписаться на колбеки

    ```kt
    var _selectedDevice = "" // здесь нужно ввести адрес выбранного каллибри

    adapter.signalQuality = {addr, hasRRPeacks ->
        if(addr == _selectedDevice) { }
    }
    adapter.hrValue = {addr, hr ->
        if(addr == _selectedDevice) { }
    }
    ```

2. запустить вычисления

    ```kt
    var _selectedDevice = "" // здесь нужно ввести адрес выбранного каллибри

    adapter.startCalculations(_selectedDevice)
    ```
3. по завершению работы нужно остановить вычисления

    ```kt
    var _selectedDevice = "" // здесь нужно ввести адрес выбранного каллибри

    adapter.stopCalculations(_selectedDevice)
    ```