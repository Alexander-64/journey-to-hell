# Введение

Всем привет, меня зовут Александр Петрушин, я frontend teamlead в компании Инфосистемы Джет
В конце марта 20-го я устроился в Jet на проект "Офсайт Росреестра". Опыт оказался для меня полезным, о чем я и хочу рассказать. 

## Предмет разработки

[Личный кабинет правообладателя](https://lk.rosreestr.ru) - это набор сервисов который покрывает все операции в отношении объектов недвижимости: запрос сведений, регистрация, уточнение сведений, получение выписки из ЕГРН и множество других операций.

Что нам было нужно сделать:

- осуществить редизайн
- улучшить время загрузки сайта (до 2 секунд при скорости 1Мб\сек)
- доработать все формы подачи услуг (80+ услуг)
- реализовать ряд новых услуг, в том числе совместную подачу заявлений несколькими пользователями

## Предыстория

Я пришел пятым разработчиком в команду, которая уже доделывала часть работ напрямую не связанных с Личным кабинетом. Все писалось с нуля, React, Typescript и пр. Уже был реализован ui-kit, несколько небольших подпроектов.

Ту часть работ, на которую я пришел, предполагалось развивать в рамках контракта. Качаю проект из гитлаба. На дворе апрель, яркое солнце, самое начало пандемии. Скачался, открываю. А там...


# Дебют

> "На него невозможно было смотреть без содрогания. Никакая мумия, возвращенная к жизни, не могла быть ужаснее этого чудовища. <...> когда его суставы и мускулы пришли в движение, получилось нечто более страшное, чем все вымыслы Данте."  
> _Мэри Шелли «Франкенштейн, или Современный Прометей»_

## Обследование доставшейся кодовой базы

Укрупненно, фронт состоял из двух частей:

1. Корневое приложение реализующее авторизацию и клиентской роутинг. В нем же написаны сервисы (например, [Справочная информация по объектам недвижимости в режиме online](https://lk.rosreestr.ru/eservices/real-estate-objects-online)). Стек:

- Js, coffescript
- Angular 1.4
- Gulp
- bower и npm, сразу оба, чего бы и да.
- тесты на karma + protractor

Ладно, думаю. Давайте дальше.

2. самописный фреймворк на котором были реализованы услуги (которых 80+). Внедрялся в корневое приложение.

- чистый ES5
- под капотом backbone (0.9.10)
- сборщика нет, все файлы исходного кода реализуют паттерн модуль и подключается через тег `<script/>`. Как следствие, сталкиваемся с ограничениями по количеству открытых соединений
- кодовой базе на тот момент было лет ~10, судя по комментариям в коде и примененных технологий
- тестов нет

Стоит упомянуть как этот фреймворк работал.

1. Для создания/редактирования услуги открывается редактор (отдельный UI как часть работы со смартформами). Он предполагал создание самих форм услуг. Проще говоря, результатом работы редактора были JSON структуры содержащие информацию о наборе полей, их расположении в панелях, валидаторы, справочники.
2. Итоговую услугу после создания или редактирования сохраняем в базу данных.
3. В клиентском приложении делается запрос к серверу, который достает из базы вьюмодель и отдает на фронт.
4. "Клиентская" часть этого фреймворка подгружает JSON конкретной услуги, движок его парсит и отрисовывает соответсвующую услугу. В Json содержится информация какими контролами представлены поля вью-модели, в каких панелях и прочая визуальная составляющая.
5. Несмотря на то что валидация формы и ее последующая отправка реализована внутри движка, есть возможность вешать кастомные обработчики. В качестве примера, в обход от общего механизма можно внезапно сделать запрос и получить необходимый справочник.

Итого, какие проблемы мы имеем:

- отсутствующую документацию. Никаких схем, никаких тестов.
- При целевой скорости загрузки 1 Mbit/s с чистым кешем сайт грузился порядка 30 секунд, против необходимых 2. Во многом причина была в том, сайт делал 190+ запросов чтобы отрисоваться (при обновлении вкладки на услуге), исходный код не был минифицирован. Однако, сделав попытку собрать код в один бандл я обнаружил что местами была важен порядок подключения скриптов (!). За разумное время задача решена не была, и сложно сказать сколько бы это заняло.
- кодовая база развивалась различными подрядчиками на протяжении ~10 лет. Вследствии этого, попытки привнести ту или иную технологию (с последующими надеждами переписать остальное, видимо) доведены до конца не были. Как итог, имеется "зоопарк" технологий.
- безнадежно устаревшие технологии. Как следствие:
  - неизбежно вызовут лаг по овладению компетенциями командой
  - мотивировать разработчиков на работу с такими старыми технологиями проблематично. Представьте: "Пс, парень! Да-да, ты. Хочешь изучить технологию которая тебе в жини больше не пригодится примерно никогда? я так и думал, держи синюю таблетку".
  - В текущих реалиях рынка найти человека со знаниями технологий чтобы начал приносить пользу сразу, и чтобы его стоимость была ниже уровня senior+ представляется трудноосуществимым и занять может продолжительное время.
- Невозожность оценить количество ресурсов, необходимых на реализацию заявленного функционала

Напомню, доставшаяся мне команда состояла из 2 джунов, 1 мидла, 1 сеньора, которые из фронтенда в принципе владеют только React'ом и библиотеками из его экосистемы.

## Попытки развития имеющейся кодовой базы

> "Бойня и анатомический театр поставляли мне большую часть моих материалов; и я часто содрогался от отвращения, но, подгоняемый всё возрастающим нетерпением, всё же вёл работу к концу."  
> _Мэри Шелли «Франкенштейн, или Современный Прометей»_

К июню команда заканчивала предыдущие активности, и остро встал вопрос о том, что делать дальше. Начали с простого - редизайна, попутно ожидая формализации требований от команды аналитики.
С редизайном было достаточно просто - ангуляровское приложение и приложение фреймворка услуг в качестве контролов использовало компоненты, тем более в нашей команде был свой дизайнер, с которым мы оперативно решали вопросы.

В случае с разработкой новых сервисов и доработкой существующих было сложнее. Ввиду описанных выше сложностей, мы решили переписать на реакт те сервисы и услуги, которые затрагивались для доработок, и реализовать новый функционал на нем же. Внедрить получилось, однако сама идея порождала ряд интересных моментов:

- необходимо было управлять загрузкой React'а, асинхронно грузить при переходе на главную, либо при обновлении страницы реализованной на реакте его грузить. Тут явно требовался отедльный механизм.
- связанный с предыдущим пунктом широкий вопрос доставки клиентского кода:
  - необходимо было прикрутить бандлер к доставшемуся самописному фреймворку работы с услугами чтобы решить проблему ограничения браузером параллельных запросов, минифицировать код
  - разбить на чанки исходный код ангуляровского приложения (собирался на gulp)
  - выстроить процесс сборки всего этого для поставки на CI
- получилось бы нам попасть в заявленные 2 секунды при 1 Mbit/s при развитии кодовой базы? Не уверен. Нужно бы было очень хорошо разобраться в устройстве фреймворка, асинзрнно подгружать необходимые части. С ангуляром было бы проще, но следует понимать, что мы бы все равно грузили большое количество кода (как минимум сразу Angular + Backbone + код самого фреймворка) для отображения только одной услуги при явном переходе к ней по url. 

В общем, если бы Виктор Франкенштейн увидел итоговый результат нашей работы он бы понял, что он просто щенок. К счастью, к лету после встречи с заказчиком стало ясно, что услуги будут переработаны значительно, от нашего редизайна ждут сильно больше чем просто "перекрасить" контролы.

# Миттельшпиль

Как ранее уже было упомянуто, уже была команда со знанием react и реализовано несколько небольших встраиваемых приложений, была библиотека готовых компонентов на которых нужно было реализовать функционала, lerna при помощи которой это все собиралось. Поэтому личный кабинет просто добавлялся новой папкой в имеющуюся инфраструктуру.

С технической точки зрения, был использован актуальный стек: React, redux-saga, react-hook-form. Т.к. контракт с бэкендом уже был, и бэкенд только дорабатывался, то вопросов о взаимодействии с сервером не было - обычный REST, который всем устраивал. Клиентский роутинг во многом предопределил code-splitting. Дополнительно пришлось сделать подгрузку store чтобы уложиться в требования о времени загрузки.

## Работа с услугами

Стоит упомянуть как мы реализовали работу услугами, отказавшись фреймворка.
(тут будет картинка услуги, визуально поясняющая что такое виджет)

На момент проектирования была следующая информация:

- услуги имеют общие части форм
- вариантов подачи было 2400+ ввиду разных заявителей, предметов заявления и пр.
- представление обсуждалось

По уровням абстракции код услуг состоит из трех слоев:

1. В качестве метафоры системы мы ввели понятие "виджет", договорившись с аналитиками и тестированием что это будет набором или даже одним полем формы. Название не самое удачное, однако мы договорились иметь некую единицу смысла, из совокупности которых складывалась конкретная услуга. На этапе постановки требований коллеги из аналитики смогли выдать нам набор виджетов, чем сильно упростили декомпозицию задачи. каждый виджет сам решает когда ему вызвать колбэк onChange передавая в него данные с формы, информацию об успешной валидации данных, и название этого виджета.
   У нас все виджеты реализованы при помощи react hook form, однако новые виджеты можно реализовывать и при помощи других библиотек либо без оных.
2. Виджет менеджер отвечает за визуальное представление виджетов (в нашем случае это панели, но могут быть и "шаги" и пр.), должны ли панели быть свернутыми, отображение корректно заполненного виджета. Этот слой ответственен за представление.
3. Сервис менеджер ответственен за логику:

- создание черновика услуги, сохранение после каждого валидного заполнения виджета
- сохраняет шаги - заполнение, предпросмотр заявление, подписание электронной цифровой подписью
- осуществляет конвертацию данных в формат, принимаемый бэкенд сервисом для подготовки xml заявления в ведомство
- управляет совместной работой над услугой в случае нескольких заявителей

Реализация этого механизма позволила свести работу над услугами к однотипным операциям:

- комбинирование имеющихся виджетов
- разработка виджета с уникальным набором полей для услуги
- написание конвертера для мапинга данных в формат принимаемый сервисом подготовки заявлений

Попутно это позволило подгружать код каждого виджета через lazy loading по мере необходимости. Так же разработку виджетов ввиду проведенных границ можно было проводить в storybook, а значит и отдельно тестировать.

Подводя итог, можно сказать что решение себя оправдало и помогло реализовать 2400+ сценариев силами 5 специалистов за, примерно, 4 месяца. Потом уже исправлялись баги и вносились доработки по актуализированной аналитике.

## Чат-бот

- разработка виджетов на сторибуке

## Code-splitting

Написать про дисклеймер возможной медленной скорости работы из-за сторонних систем не входящих в предмет разработки в данной статье

- подгрузка стора on-demand

## Процесс разработки в команде фронта

В каждой команде процессы отличаются, поэтому стоит коротко рассказать как работали мы.

## Разработка

В качестве тестового фреймворка мы использовали cypress. При помощи него покрывали тестами:

- отдельно виджеты
- библиотека компонентов
- услуги или сервисы целиком
- механизмы по типу сервис-менеджера
- писали обычные юнит тесты там, где это казалось разумным

Как итог, кодовая база оказалась вполне неплохо задокументирована

Тесты запускал Jenkins на каждый MR. Наличие написанных тестов являлось обязательным (не всегда выдерживалось из-за жестких сроков) условием успешного прохождения ревью. Все тесты проходили за 2 - 3 часа, потом мы оптимизировали до 1 часа 40 минут. Многовато, но спокойный сон дороже; тем более локально никто не ограничивал разработчика запускать интересующие его сценарии, да и целиком локально они проходили примерно вдвое-втрое быстрее.

Ревью проводили в основном старшие разработчики. От этого частично ушли, сформировав компетенции по областям у остальных; однако в этой части процесс был не быстрым. Но и особых проблем не было т.к. уже упомянуто было что команда была сработанная, комфортно себя ощущала на выбранном стеке. А что работает, то и "чинить" не хочется =)

Непосредственно разработка проводилась помимо сторибука на мок сервере. Компоненты бэкенда разворачивались параллельно к релизу, поэтому ограничиться проксированием запросов не представлялось возможным; вместо этого реализовывали попутно нехитрую логику на express. Само по себе это и неплохо, т.к. на CI он же и разворачивался для целей прогона тестов. Контракт с бэкендом был известен на старте разработки, поэтому проблемы возникали только на этапе интеграции на dev сервере.

Ежедневный статус на 30 минут достаточно эффективно позволял решать и снимать вопросы. "Проталкивание" историй из серии "я задал вопрос аналитику, а мне не отвечают" снимался созданием микро-чата на заинтересованных. Как правило, время находилось как ответить, так и задать вопрос.

В качестве таск трекера была Jira, на статусе просто шарили канбан-доску с приятным и подобающим количеством столбцов.

## Релизы

Все работы были нами оценены, подготовлен план выпуска релизов. Мы знали какие сервисы и услуги от нас ждут, соответственно и коллеги от тестирования детально готовили тестовые сценарии согласно этого плана (а мы потом аккуратно списывали и вдохновлялись при реализации своих тестов на сервисы и услуги). Релизились каждую неделю - это помогало устранять возможные проблемы как можно раньше

Чем мы гордимся - так это тем, что по графику релизов мы сделали все (...TODO)

# Эндшпиль

## Рост команды до 20 человек

> – Если бы каждый занимался своим делом, – сердито проворчала Герцогиня, – то Земля завертелась бы гораздо быстрее, чем вертится теперь.  
> _«Приключения Алисы в Стране чудес», Льюис Кэрролл_

В сентябре руководство поставило задачу ускориться по работам над Личным кабинетом. В связи с этим был найм пары человек в штат, пары с других проектов, плюс привлечение 10 подрядчиков - итого 20 frontend разработчиков. В связи с этим команда условно была разделена на 3 стрима. За 1,5 месяца я провел около 30-40 собеседований. Спасибо нашим HR за качественную работу!

Подрядчиков выделили в отдельную команду, поставлены задачи по реализации сервисов. Как я упоминал ранее, это были просто страницы с несколькими формами и/или таблицами, графиками. В среднем 1-3 небольших эндпоинта. Т.е. задачи не предполагали глубокого понимания проекта или специфики предметной области.
Поэтому и онбординга как такого не было - человек получал доступы, выдавалась задача, обсуждалась реализация, обсуждался срок.

В новой подкоманде процесс был таким же, за исключением ревью. Я упоминал что досталась сработанная отличная команда ребят из штата; здесь нужно было выстраивать по новой. При этом, учесть что инструменту должны сработать здесь и сейчас - пестовать что-то в ком-то бессмысленно, ввиду того что привлечение на короткий срок.

В очередной раз ситуацию спасли тесты.

МР проверялся по следующему алгоритму:

- проверялось описание типов на соответствие контракту
- соответствие код-стайлу
- наличие тестов, чтобы был реализован смоук
- оставлялись замечания только при грубых нарушениях, в остальных случаях переводилось в техдолг
  Не приходилось даже переключаться на ветку с задачей, за ненадобностью.

Получал МР, в нем пройденные тесты, по алгоритму вопросов нет, можно мержить

## Параграф для избитой цитаты 
> Чувствуешь запах? Это напалм, сынок. Больше ничто в мире не пахнет так.
> Я люблю запах напалма поутру. Однажды мы бомбили одну высоту, двенадцать часов подряд. И когда всё закончилось, я поднялся на неё. Там уже никого не было, даже ни одного вонючего трупа. Но запах! Весь холм был им пропитан. Это был запах… победы!  
> _Билл Килгор, Апокалипсис сегодня_

После трех месяцев напряженной работы усиленной команды, мы достигли результата, реализовав все функциональные требования. Конечно, в итоге мы имели определенный скоуп ошибок, но дальше наша работа уже заключалась исключительно в корректировках связанных с уточнением требований, и вопросы, связанные с опытной эксплуатацией.

В это время мы уже попутно разбирались с техническим долгом, корректировали

# Заключение
> – Ты ошибаешься, милая! Нет ничего на свете, из чего нельзя было бы сделать вывод. Надо только знать, как взяться за дело
> _«Приключения Алисы в Стране чудес», Льюис Кэрролл_


Вся команда (не только фронт) отлично сработала, с высоким уровнем вовлеченности и самоотдачи. Передали заказчику кодовую базу хорошего качества, задокументированную, отлично покрытую тестами которую при должном умении можно развивать. Испытали огромное удовлетворение от хорошо выполненной работы.
Безусловно, все было не столь гладко как описано выше. Однако спустя время из памяти изглаживаются моменты, вызвавшие негативные эмоции, и остается то, с чем хочется ехать дальше:

1. С тестами. Сослужили отличную службу:
* позволили делать стабильные релизы
* упростили разработку
* дали возможность легко проводить рефакторинг  
Однако, оказались не востребованы на уровне команды проекта (могли стать основой для e2e тестов). Не так давно порефлексировали с командой тестирования в рамках митапа на эту тему, решили посмотреть в сторону playwright как более современная альтернатива cypress, подкупает возможность распараллеливания тестов.
2. С людьми. Замотивированный, вовлеченный в проект профессионал, стоит пятерых. Процесс отбора начиная с HR, найм, ввод в проект, общение, поддержание атмосферы - супер важно.  

Надеюсь, статья будет полезна и натолкнет кого-то на светлые мысли.
