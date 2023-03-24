- Дата создания: 2023-03-15
- RFC PR: 
- RFC Issue: 
- Flexberry Issue:

# (Название RFC)

## Краткое описание

Максимально возможный рефактор [Flexberry ORM](https://github.com/Flexberry/NewPlatform.Flexberry.ORM) и [Flexberry ODataService](https://github.com/Flexberry/NewPlatform.Flexberry.ORM.ODataService) для полноценной поддержки Dependency Injection (DI).

## Обоснование

Сейчас существует ряд особенностей ORM и ODataService, которые не позволяют использовать DI в "классическом виде".
1. Обращение к UnityFactory происходит в произвольном методе произвольного класса.
2. Аналогично происходит работа некоторых статических классов, которые фактически являются реализацией внедрения зависимостей (например, [DataServiceProvider.DataService](https://github.com/Flexberry/NewPlatform.Flexberry.ORM/blob/3ec3dc517469e6df519035d750a3da6c44a91bac/ICSSoft.STORMNET.Business/DataServiceProvider.cs#L28)).

Что планируется получить:
1. Внедрение зависимостей в классы происходит максимально возможно в конструкторах.
2. Регистрация зависимостей происходит в единой точке.
3. Из ORM и ODataService убрано максимально возможно произвольное внедрение зависимостей.
4. Классы, которые реализуют некорректную инъекцию, помечаются как Obsolete. Информация об этих классах или ситуациях убраны из документации, прописано, как корректными образом разрешать зависимости.

## Детальное проектирование

Ранее велись работы в этом направлении, которые учитываются при написании данной постановки:
1. [PR Рефактор сервиса данных для полноценной поддержки DI](https://github.com/Flexberry/NewPlatform.Flexberry.ORM/pull/181).
2. [PR Resolve IEventHandlerContainer and IFunctionContainer using dependency injection](https://github.com/Flexberry/NewPlatform.Flexberry.ORM.ODataService/pull/51).
3. [Issue Возможность внедрения зависимостей в обработчики одата-функций и экшенов](https://github.com/Flexberry/NewPlatform.Flexberry.ORM.ODataService/issues/35).
4. [Issue Выбор и реализация оптимального механизма конфигурирования Flexberry-приложений под .net core](https://github.com/Flexberry/NewPlatform.Flexberry.ORM.ODataService/issues/152).
5. [Issue Отказаться от использования UnityFactory в ODataService](https://github.com/Flexberry/NewPlatform.Flexberry.ORM.ODataService/issues/210).
6. [Issue Рефактор SQLDataService и всего обвеса](https://github.com/Flexberry/NewPlatform.Flexberry.ORM/issues/69).
7. [Issue Рефактор настроек IAuditService](https://github.com/Flexberry/NewPlatform.Flexberry.ORM/issues/70).
8. [Issue Реализация CurrentWebHttpUser для netcoreapp](https://github.com/Flexberry/NewPlatform.Flexberry.ORM/issues/141).
9. [Discussion Выбор DI контейнера вместо Unity](https://github.com/orgs/Flexberry/discussions/16)

UnityFactory во Flexberry-библиотеках:
Применение UnityFactory в ORM: 
1. getter IConfigResolver ConfigResolver в SQLDataService.
2. getter ISecurityManager SecurityManager в SQLDataService.
3. getter ISecurityManager SecurityManager в XMLFileDataService.
4. public static View GetPossibleDynamicView(string viewName, Type dataObjectType) в DetailVariableDef.
5. public static bool CheckAccessToAttribute(Type type, string propertyName, out object deniedAccessValue) в Information.
6. public static void Reset() в CurrentUserService.
7. Тесты

Применение UnityFactory в ODataService: 
1. Конструктор DataObjectEdmModel(DataObjectEdmMetadata metadata, IDataObjectEdmModelBuilder edmModelBuilder = null) 
2. Тесты

Что конкретно планируется изменить:
1. Замена использования UnityFactory в ORM и ODataService, инициализация внедряемых зависимостей по возможности переносится в конструктор.
- Инъекция IAuditService и ISecurityManager через конструктор SQLDataService ([PR](https://github.com/Flexberry/NewPlatform.Flexberry.ORM/pull/181)).
- [BREAKING CHANGE] Убрана установка STORMAdvLimit.User в setter'е STORMAdvLimit.Publish ([PR](https://github.com/Flexberry/NewPlatform.Flexberry.ORM/pull/181)).
- Инъекция IConfigResolver через свойство SQLDataService ([PR](https://github.com/Flexberry/NewPlatform.Flexberry.ORM/pull/181)).
- Получение БСов через абстрацию IBusinessServerProvider ([PR](https://github.com/Flexberry/NewPlatform.Flexberry.ORM/pull/181)).
- Удаление получения CommandTimeout через файл конфигурации ([PR](https://github.com/Flexberry/NewPlatform.Flexberry.ORM/pull/181)).
- Удаление статической инстанции AuditService и статической инициализации.
- Преобразование метода ConfigHelper.ConstructProperAuditDataService таким образом, чтобы сервис данных разрешался через Unity по имени container.Resolve<IDataService>(dsName).
2. Использование Unity переориентировано на инициализацию в "точке входа" в приложение (в частности, в .net core 3.1 - это Startup.cs).
3. Удаление сущностей, способствующих некорректной инъекции (возможно, просто отметка как obsolete):
- Удаление DirectoryServices и CurrentWindowsUser ([PR](https://github.com/Flexberry/NewPlatform.Flexberry.ORM/pull/181)).
- Удаление DataServiceWrapper([PR](https://github.com/Flexberry/NewPlatform.Flexberry.ORM/pull/181)).
- Удаление DataServiceProvider.DataService как fallback сервис данных для BusinessServer ([PR](https://github.com/Flexberry/NewPlatform.Flexberry.ORM/pull/181)).
- Удаление AppMode ([PR](https://github.com/Flexberry/NewPlatform.Flexberry.ORM/pull/181)).
4. Реализация возможности внедрения зависимостей в обработчик одата-функции или экшена. Желательно, чтобы внедрение производилось не в параметры метода, чтобы не смешивать зависимости и обычные параметры, а в конструктор ([PR](https://github.com/Flexberry/NewPlatform.Flexberry.ORM.ODataService/pull/51)).

Дополнительно:
1. Подготовить пример работы в новой версии, используя только встроенный DI.
2. Подготовить пример работы в новой версии, используя Unity (однако проект Unity на настоящий момент приостановлен в развитии).
3. Подготовить пример работы в новой версии, используя иной вариант DI (autofac, ninject или иной, у которого реализована поддержка Microsoft...DependencyInjection).

## Документирование и обучение

Информация обо всех классах, что будут помечены как obsolete, должна изыматься из документации и вместо этих статей нужно добавить описание, как реализовать аналогичное поведение.

## Недостатки

1. Возникнут сложности с обратной совместимостью библиотек.
2. Потребуется изменение кода использующих библиотеки проектов для инициализации Unity в единой точке.

## Альтернативы

...

## Нерешенные вопросы

1. Необходимо определиться, один (встроенный DI либо Unity DI) или два (встроенный DI + Unity DI) следует использовать в приложениях на платформе Flexberry под .net core, а так же определиться с реализацией конфигурирования через файлы конфигурации и, соответсвенно, с номенклатурой конфигурационных файлов (один формата json либо xml, один формата json для стандартного конфигурирования .net core приложения + app.config для конфигурирования log4net и Unity DI, либо один формата json + отдельный .config для каждой подсистемы, конфигурируемой через xml.
2. Оценить целесообразность добавления абстракции ICurrentUserAccessor (Получение текущего пользователя ICurrentUser через абстракцию ICurrentUserAccessor ([PR](https://github.com/Flexberry/NewPlatform.Flexberry.ORM/pull/181))).
3. Исследовать обновление внешних зависимостей (npgsql, SqlClient и прочее).
4. Исследовать необходимость реализации функции "CurrentUser" в сервисе данных.