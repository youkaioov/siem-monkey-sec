# SiemMonkey - плагин к браузеру Google Chrome для упрощения некоторых действий в MP SIEM

[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-2.1-4baaaa.svg)](https://github.com/Security-Experts-Community/.github/blob/main/CODE_OF_CONDUCT.md)

## Установка
Описание процесса установки описано в файле [INSTALL.md](INSTALL.md)

## Репозитории проекта
Основной:
- GitHub: https://github.com/Security-Experts-Community/siem-monkey

Зеркала:
- Codeberg: https://codeberg.org/Security-Experts-Community/siem-monkey
- GitFlic: https://gitflic.ru/project/security-experts-community/siem-monkey

# Основные функции
- [Popup](#popup):  
    * получение дополнительной информации из событий MP SIEM и/других систем на основе полей события в правой панели
    * быстрый переход к определенным событиям или другим системам (PT NAD, PT Sandbox) в отдельной вкладке  
- [UI SIEM](#ui-siem):
    * выполнение некоторых действий по клику на определенные поля события в правой панели 
    * возможность расширения правой панели с информацией о событии на достаточную ширину 

**Поддерживаемые версии:**
- MP SIEM R25, R25.1, R26, R26.1
- PT NAD 11  

# Подробное описание
# Popup
При открытии всплывающего окна плагина происходит чтение значений всех полей события, отображающегося в правой панели
 (текущее событие).  
Пользователю предлагается ряд действий по поиску информации в MP SIEM или переходу к внешним системам на основе 
значений некоторых полей текущего события.  
**ВАЖНО:** Значения из "свернутых" групп полей не получаются и их использование в действиях невозможно.  
## Процессы
<img src="https://user-images.githubusercontent.com/51186173/222954645-c9a7f551-efdd-4b0c-b8cf-1dbf94327d7f.png" width="400">

На вкладке "Процессы" доступны функции, позволяющие отобразить информацию о событиях запуска процессов
 в виде иерархического дерева.  
Ограничение количества запрашиваемых событий о запусках процессов задается в поле "Максимальное число событий".  
По умолчанию при построении дерева всех процессов текущей сессии будет запрашивать не более 1000 событий.  
Ограничение времени, за которое осуществляется поиск событий при построении дерева процессов задается в полях "С" и "По".  
По умолчанию эти поля заполняются временем, отстоящим на одни сутки в меньшую и в большую сторону от времени текущего события,
информации о котором отображается в правой панели интерфейса MP SIEM.  
### Найти в SIEM и проверить на VT хеш файла 
Производится поиск события запуска процесса sysmon 1 или инвентаризации файла антивирусом Касперского для имени файла, 
содержащегося в поле `object.name` и `object.process.name` (если они различны). Из найденного события извлекается хеш 
файла и проверяется на VirusTotal. Результаты проверки выводятся в окно плагина в виде отношения числа "плохих" вердиктов 
к общему числу, а так же список известных имен файла. При клике на количество вердиктов происходи открытие VirusTotal 
с информацией о файле в новой вкладке браузера.  
### Проверить на VirusTotal по хешу  
Из текущего события извлекается хеш файла и проверяется на VirusTotal. Результаты проверки выводятся в окно плагина 
в виде отношения числа "плохих" вердиктов к общему числу, а также список известных имен файла. 
При клике на количество вердиктов происходи открытие VirusTotal с информацией о файле в новой вкладке браузера.  
### Показать сессии и число процессов в них на узле
Из текущего события запуска процесса (4688 или sysmon 1) извлекается имя хоста (`event_src.host`). Для этого хоста запрашивается информация 
о числе запущенных процессов в каждой сессии и учетной записи, от имени которой выполнялись процессы в данной сессии.
Полученная информация выводится в окно плагина. При этом темно-красным цветом помечаются сессии, запущенные от имени 
встроенных учетных записей (SYSTEM, LOCAL SERVICE, NETWORK SERVICE и др. у которых номер сессии меньше 1000).
Серым цветом помечаются сессии, запущенные от имени учетных записей с именами, начинающимися на `dwm-` и `umfd-`.  
Для получения информации о процессах определенной сессии в виде дерева необходимо кликнуть на соответствующую строку 
в списке сессий.  
### Найти процессы в сессии на узле
Из текущего события запуска процесса (4688 или sysmon 1) извлекается имя хоста (`event_src.host`) и номер сессии 
(`object.account.session_id`). Далее из MP SIEM запрашивается информация обо всех событиях запуска процессов в указанной сессии 
(с учетом ограничений по количеству и времени выборки). Полученные сведения отображаются в окне плагина в виде дерева.
В каждом узле дерева выводится информация о времени запуска процесса (в UTC), PID процесса и параметры командной строки.
При клике на определенную строку с информацией о процессе происходит открытие новой вкладки MP SIEM, в которой отображаются 
все события, связанные с выбранным процессом на основе значения guid процесса (только для дерева, построенного на основе событий sysmon 1).  
### Родители процесса
Из текущего события запуска процесса (4688 или sysmon 1) извлекается идентификатор события (`uuid`).
Далее из MP SIEM последовательными запросами получается информация о событиях запуска родительских процессов.
Полученные сведения отображаются в окне плагина в виде дерева.
В каждом узле дерева выводится информация о времени запуска процесса (в UTC), PID процесса и параметры командной строки.
При клике на определенную строку с информацией о процессе происходит открытие новой вкладки MP SIEM, в которой отображаются 
все события, связанные с выбранным процессом на основе значения guid процесса (только для дерева, построенного на основе событий sysmon 1).  

### Все потомки процесса
Из текущего события запуска процесса (4688 или sysmon 1) извлекается идентификатор события (`uuid`).  
Далее из MP SIEM последовательными запросами получается информация о событиях запуска всех потомков процессов текущего процесса.
Полученные сведения отображаются в окне плагина в виде дерева.
В каждом узле дерева выводится информация о времени запуска процесса (в UTC), PID процесса и параметры командной строки.
При клике на определенную строку с информацией о процессе происходит открытие новой вкладки MP SIEM, в которой отображаются 
все события, связанные с выбранным процессом на основе значения guid процесса (только для дерева, построенного на основе событий sysmon 1).

## Полезные фильтры

<img src="https://user-images.githubusercontent.com/51186173/222954665-9ee6f019-6a6e-4f4e-b404-0658a37d1b76.png" width="400">
На вкладке "Полезные фильтры" доступны фильтры, позволяющие открыть в новой вкладке окно MP SIEM по запросу, построенному на 
основе значений полей текущего события.
Шаблоны названий и сами фильтры описываются в файле customfilters.json и могут быть самостоятельно дополнены пользователем плагина.  
Пример:
`"description":"Все события запуска процессов для учетной записи ${subject.account.name}", "filter":"msgid in [1,4688] and subject.account.name = '${subject.account.name}'"}`

Поле `description` содержит описание фильтра, выводимое в окне плагина.  
Поле `filter` содержит фильтр для MP SIEM на языке PDQL.
Конструкция `${subject.account.name}` позволяет подставить в описание и фильтр значение указанного поля текущего события.

**ВАЖНО:** В окно плагина при его открытии выводятся только те фильтры, где удалось произвести подстановку всех шаблонов 
(не выводятся фильтры, где подразумевалась подстановка полей исходного события, содержащих значение null).  

## Внешние запросы
В случае, если MP SIEM интегрирован с PT NAD и/или PT Sandbox оператору на вкладке "Внешние запросы"
предлагаются удобные способы переходить от текущего события в MP SIEM к изучению связанных с ним подробностей в этих продуктах.
Доступны следующие возможности:
- открыть в новой вкладке окно PT Sandbox с информацией о задаче обработке файла (для текущего события, полученного из PT Sandbox).
  Идентификатор задачи извлекается из поля `chain_id` текущего события.
- открыть в новой вкладке окно PT NAD с информацией о трафике по фильтру на основе IP-адресов и номеров портов источника и получателя
(строится на основе значений полей `src.ip`, `src.port`, `dst.ip` и `dst.port` текущего события)

# UI SIEM
При загрузке страницы MP SIEM в код страницы внедряется код плагина, добавляющий ряд дополнительных функций

## Обработчики кликов 
Поля события в правой панели, для которых доступны обработчики клика выделяются красным или зеленым цветом.
- `object.hash`, `object.process.hash`, `subject.process.hash` - переход на VirusTotal для просмотра вердиктов по хешу в новой вкладке  
   В параметрах плагина можно добавить дополнительные сервисы для проверки хешей, указав название, которое должно выводиться в UI,
   а также шаблон ссылки, по которой доступна проверка конкретного хеша (в качестве шаблона подстановки следует использовать `${hash}`.
   Проверка при помощи VirusTotal доступна всегда, независимо от дополнительных добавленных сервисов.  
   <img src="https://github.com/Security-Experts-Community/siem-monkey/assets/51186173/23641a71-732f-4ef0-bc73-6be92841a071" width="400">  
   <img src="https://github.com/Security-Experts-Community/siem-monkey/assets/51186173/de1f638b-d413-4ac6-8b57-18a2fa280478" width="400">  


- `external_link` - переход по ссылке, содержащейся в значении поля, в новой вкладке  
- `id` - переход к PTKB для поиска правила нормализации, соответствующего текущему событию, в новой вкладке
- `task_id` - переход к соответствующей задаче сбора событий SIEM в новой вкладке
- `src.ip`, `dst.ip` - открытие дополнительно меню со ссылками, позволяющими открыть в новой вкладке внешние системы с информацией по 
адресу, содержащемуся в названии поля. Список сервисов и шаблоны ссылкок, по которым доступна проверка конкретного адреса необходимо предварительно задать в параметрах расширения (в качестве шаблона для подстановки IP-адреса следует использовать `${ip}`). Для локальных адресов выводятся только сервисы, явно отмеченные специальным чекбоксом в настройках. Для адресов, проверка которых во внешних сервисах бессмысленна (см. https://en.wikipedia.org/wiki/Reserved_IP_addresses), ссылки не выводятся.
        
    <img src="https://github.com/Security-Experts-Community/siem-monkey/assets/51186173/2ee7a904-95c6-4d5e-b586-db207d1241ce" width="400">    
    
    <img src="https://github.com/Security-Experts-Community/siem-monkey/assets/51186173/22285065-f072-44b5-b7b3-f18aee91b984" width="400">  

## Автодополнение названий полей события и операторов PDQL при вводе фильтра событий  
При наборе текста в окне ввода фильтра событий появляется меню, предлагающее подходящие названия полей событий или операторов языка PDQL (по совпадению текущего слова с подстрокой в названии поля или оператора).
Для выбора определенного значения можно воспользоваться мышкой или кпопками ВНИЗ/ВВЕРХ клавиатуры. Клавиша TAB или ENTER позволяет вставить выбранное значение в окно ввода фильтра.  
<img src="https://user-images.githubusercontent.com/51186173/227527237-27b80ca5-4982-49ba-a74f-ea1c214d6475.png" width="400">  

## Построение дерева процессов
Для событий запуска процессов в правой панели справа от значения поля `object` отображается три пиктограммы 🦍
🦧
🐒  
Клик по ним позволяет получить во всплывающем окне дерево всех процессов текущей сессии, дерево всех предков, дерево 
всех потомков, аналогично тому, как это происходит при построении деревьев в окне плагина.

Построение дерева процессов возможно так же для любых событий (в том числе, корреляционных), если у них заполнено поле object.process.guid или subject.process.guid.
В этом случае плагин сначала самостоятельно находит событие запуска процеса, после чего строит на его основе дерево аналогичным образом, как для событий запуска процессов.  
<img src="https://github.com/Security-Experts-Community/siem-monkey/assets/51186173/bc3bf37b-db90-4bd7-986f-0bd6bfc21ec4" width="400">    


## Сохранение нормализованного события в виде JSON
<img src="https://user-images.githubusercontent.com/51186173/231389068-a8a96575-02d4-4286-ac6a-ad73dc23a7ff.png" width="400">  

В заголовке правой панели добавлены кнопки, позволяющие сохранить нормализованное событие в виде JSON (без незаполненных полей):  
- 📋 - сохраняет текущее событие в виде JSON в буфер обмена  
- 💾 - сохраняет текущее событие в текстовый файл в виде JSON, в качестве имени используется uuid события  
- 🖫 - сохраняет в текстовый файл в виде JSON массив исходных событий для текущего корреляционного события, в качестве имени используется uuid текущего события с суффиксом 'subevents'  

Появление этих кнопок можно отключить в параметрах плагина.

<img src="https://user-images.githubusercontent.com/51186173/231386611-4656632b-f485-48aa-8687-086d56dc35f7.png" width="400"> 

## Получение ссылки на текущее событие  
<img src="https://user-images.githubusercontent.com/51186173/234027736-014daeec-bbd1-4242-886b-6d7fdec5e203.png" width="400">  

В заголовке правой панели добавлена кнопка, позволяющая сохранить в буфер обмена ссылку на текущее событие.  

## Отключение нового способа сортировки для запросов событий с группировками и агрегациями  

<img src="https://user-images.githubusercontent.com/51186173/234699797-84de4a43-b5f5-4a4d-86af-f122ee9604b4.png" width="400">  

В версии R25.1 появился дополнитеьный параметр в запросах с группировками и агрегациами, позволяющий отсортировать результат по одному из столбцов, используемых в групировке.
Такое поведение не всегда нужно, и иногда необходимо дать пользователю возможность выполнять такие запросы по старому.
В расширение добавлен параметр ```Отключить новое поведение сортировки при аггрегации (R25.1 и выше)```, при использовании которого происходит модернизация запросов к бекенду, убирающая параметр ```groupByOrder```, влияющий на сортровку.  
Изменения поведения вступают в силу после полной перезагрузки страницы.  


## Пользовательские названия для полей при их отображении в панели событий  
<img src="https://github.com/Security-Experts-Community/siem-monkey/assets/51186173/8764523a-eae5-4bfe-b91b-b7331dc2d9fb" width="400">  

Можно настроить отображение собственных имен полей событий. Пример файла конфигурации приведен в файле fieldaliases.example.json. 
Требуемое сопоставление необходимо указать в файле fieldaliases.json. Поддерживается настройка маппинга названий полей как для всех событий (секция default), 
так и индивидуально по значению поля id или correlation_name события.  

Пример конфигурационного файла fieldaliases.json  
```
{
    "LSASS_memory_access_SubRule": {
        "datafield9": "стек вызовов"
    },
    "LSASS_Memory_Dump": {
        "datafield9": "стек вызовов"
    },
    "Suspicious_Connection_System_Process": {
        "datafield19": "продвинутая цепочка"
    },
    "PT_UNIX_like_auditd_syslog_structured_syscall_process_start": {
        "datafield3": "Binary File Access Mode"
    },
    "default": {
        "subject.process.cmdline": "командная строка процесса",
        "subject.process.parent.cmdline": "командная строка родителя"
    }
}
```  
 


# Bonus \#1
Плагин так же позволяет переходить из карточки сессии или карточки атаки в интерфейсе PT NAD к окну MP SIEM в новой вкладке
для поиска по фильтру событий на основе адресов и номеров портов отправителя и получателя.
Для этого нужно находясь в окне просмотра карточки сессии или атаки PT NAD открыть окно плагина и на вкладке "полезные фильтры"
выбрать один из подходящих пунктов:  
- События подключения к 4.8.15.16:2342 в SIEM (для поиска из NAD)  
- События подключения 4.8.15.16:2342 ⇄ 3.14.15.92:6535 в SIEM (для поиска из NAD)  
Первый пункт может помочь найти в MP SIEM все события подключения к указанному серверу по определенному порту.
Второй фильтр удобен для поиска информации о конкретной сессии.  
При переходе к MP SIEM диапазон времени задается на основе времени начала сессии ±15 минут. 



# Bonus \#2
Убрано ограничение на максимальную ширину правой панели (панели события).

Иконка  
[Monkey icons created by Darius Dan - Flaticon](https://www.flaticon.com/free-icons/monkey)  
Анимация загрузки  
[loading.io css spinner]( https://loading.io/css/ ) 
