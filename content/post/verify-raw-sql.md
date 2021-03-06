+++
date = "2017-03-30T09:01:29+03:00"
title = "Проверка сырых SQL запросов"

+++
### TL;DR
Плагин для Visual Studio, проверка сырых SQL запросов в управляемом C# коде.

### Чуть подробнее
Не поймите меня не правильно, я люблю ORM'ы, разной степени навороченности. Но случаются моменты когда приходится "деградировать" в использовании ORM, к примеру когда нужно использовать функции базы данных которые не поддерживаются в вашей ORM или просто хотите убрать дополнительные накладные расходы в запросах. 

Что делать когда база оптимизирована, где-то денормализирована для достижения лучших показателей скорости выполнения запросов, но данных много и кэшей на всех не хватает, а Entity или Nhibernate дает не достаточную производительность? Можно вновь окунуться в ADO или взять что-то рядом к примеру Dapper.

Использование сырых SQL запросов в управляемом коде, выглядит не так красиво как могло быть, но если гнаться не за красотой а за скоростью исполнения запросов то использование самописных запросов дает полный контроль над ситуацией. Основная проблема с быстро растущей схемой базы данных, это ее изменения, стоит переименовать столбец в базе, как у тебя падает выполнения кода в рантайме если этот кусок еще не был покрыт тестами. Мне прилично надоела подобная ситуация а с Dapper я расставаться не хотел. Порылся в каталоге расширений Visual Studio, взгрустнул и решил запилить свое.

### Из чего будем строить
 Немного подумав и решив, что больше одного вечера тратить на плагин нет жаления, было решено взять несколько компонентов из NuGet, а не вручную писать парсеры\валидаторы c#\sql кода, на стадии прототипа этого более чем достаточно. И так были подключены:

- Microsoft Code Analysis - парсинг .net кода, дабы с меньшим количеством ошибок выдрать строки, так как способов описания множество.
- Dapper - исполнение SQL кода.

Можно понять что логика работы\проверки кода проста как 2 копейки:

- Ловим событие сохранения документа\сборки проекта 
- Парсим документ, выбираем все SQL строки
- Выполняем SQL конструкции через Dapper и в случае ошибки выводим сообщение в окно сообщений об ошибках


### Пишем плагин для Visual Studio
Всем не угодишь, все описывать много, а выбрать небольшую часть будет иметь мало смысла, потому кому нужен код, смотрите на [GitHub](https://github.com/pkochubey/VerifyRawSql), каждый найдет то что ему нужно. 

В работе плагина есть следующие ограничения:

- Проверка осуществляется при сборке проекта или сохранении документа
- Необходимо наличие строки соединения с именем VRS в любом конфигурационном файле проекта. 

В целом поделюсь впечатленим от написания плагина. Делать его оказалось очень даже просто и вполне понятно, но я так полагаю это касается последних версий Visual Studio 2015\2017. Конечно сама студия глюкавая, со своими ограничениями и костылями, с этим приходится мириться при написании расширения.

### Что в итоге
Расширение работает. Оно помогает мне каждый день, тем более в конце рабочего дня, когда концентрация внимания падает. При первом запуске было найдено сразу две ошибки, глубоко в легаси ядре связанным с анализом данных. 
Скачать расширение можно из репозитория или найти его через окно установки расширений Visual Studio, используйте при поиске Verify Raw SQL, это уж точно найдет.

