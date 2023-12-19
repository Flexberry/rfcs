- Дата создания: 2018-05-25
- RFC PR: 
- RFC Issue: 
- Flexberry Issue: 

# Генерация индексов для определенных пользователем типов

## Краткое описание

Flexberry Designer при генерации БД для указанных типов полей должен создавать индексы.

## Обоснование

При создании баз геоданных добавляются новые типы Geometry и Geography, для них обязательно нужны индексы, их отсутствие значительно сказывается на производительности ГИС ПО.
Вручную создание этих индексов прописывать не очень эффективно - они шаблонные. Генератор баз данных при создании скриптов генерации должен так же создавать индексы для указанных полей.

## Детальное проектирование

В карту типов для генератора БД добавить поле "Шаблон индекса".
При генерации скриптов при выполнении задач "Привести БД в соответствие с моделью" и "Сгенерировать SQL", если для указанных полей заполнено поле "Шаблон индекса" - добавлять в результирующий SQL файл строки с созданием соответствующего индекса.

Пример шаблона для полей Geometry:

```sql

CREATE INDEX %indexname%
  ON %storagename%
  USING gist
  (%fieldname%);
  
```

Пример результирующего кода для создания таблицы с геометрией для БД PostgreSQL:

```sql

CREATE TABLE geo.forestry_polygon
(
  primarykey uuid NOT NULL DEFAULT uuid_generate_v4(),
  registrynumber character varying(50),
  forestryname character varying(100),
  forestrylocation character varying(200),
  square character varying(20),
  note character varying(200),
  territory character varying(500),
  hierarchy character varying(200),
  dataobjectkey character varying,
  shape geometry(MultiPolygon,3857),
  CONSTRAINT forestry_polygon_pkey PRIMARY KEY (primarykey)
)
WITH (
  OIDS=FALSE
);

CREATE INDEX geo_forestry_polygon_shape
  ON geo.forestry_polygon
  USING gist
  (shape);
```

## Документирование и обучение

В документации надо будет дополнить раздел касающийся "Карты типов" и генераторов БД

## Недостатки

## Альтернативы

## Нерешенные вопросы

* возможно ли изменять сущность "Карта типов" или эти данные должны быть в другом месте?
* нужна ли возможность переопределять шаблон индекса на уровне конкретных сущностей?
