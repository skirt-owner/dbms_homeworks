# HW

![](/assets/overview_small_1.mp4)
![](/assets/overview_small_2.mp4)

## 1. Окружение и запуск

| Параметр | Значение |
|---|---|
| MongoDB | Образ `mongo:8.0.30`, `mongosh` внутри образа |
| Топология | Replica set `hw2rs`, три узла с данными: `m1:27017`, `m2:27017`, `m3:27017` |

1. Подготовка окружения
```bash
docker compose up -d
docker compose run --rm lab reset
docker compose run --rm lab reset index
```
2. Запуск сценариев
```bash
docker compose run --rm lab all
```

## 2. Данные и запросы

> Объясните модель данных в README. Почему позиции встроены в заказ, а история покупателя хранится отдельными документами? Какой факт представляет цена в позиции заказа?

1. **Документ** - отдельная запись, **коллекция** - набор документов. Границы документа выбираем по тому, что читается вместе, что изменяется вместе и как растёт объём данных. При просмотре заказа обычно нужен весь его состав (который *не измениться* после создания). Один документ позволяет получить заказ вместе с товарами, количествами и ценами покупки. Количество заказов же растёт со временем и заранее не ограничено и обычно нужна только часть истории - поэтому храним отдельными документами.
2. "Цена на момент создания заказа" - чтобы возможные изменения в самих продуктах не переписывали историю заказов. 

## 3. Отчёт о продажах

> **Покажите в README расчёт количества и выручки одного товара** по позициям исходных заказов.

```bash
docker compose run --rm lab fixtures
```

<details>
<summary>PRODUCTS</summary>
<code>
[
  {
    "_id": "A",
    "name": "Футболка",
    "category": "clothes",
    "price": {
      "$numberDecimal": "100.00"
    },
    "variants": [
      {
        "size": "L",
        "stock": 0
      },
      {
        "size": "M",
        "stock": 10
      }
    ]
  },
  {
    "_id": "B",
    "name": "Рубашка",
    "category": "clothes",
    "price": {
      "$numberDecimal": "80.00"
    },
    "variants": [
      {
        "size": "L",
        "stock": 5
      },
      {
        "size": "M",
        "stock": 4
      }
    ]
  },
  {
    "_id": "C",
    "name": "Толстовка",
    "category": "clothes",
    "price": {
      "$numberDecimal": "90.00"
    },
    "variants": [
      {
        "size": "L",
        "stock": 3
      },
      {
        "size": "M",
        "stock": 2
      }
    ]
  },
  {
    "_id": "D",
    "name": "Ремень",
    "category": "accessories",
    "price": {
      "$numberDecimal": "120.00"
    },
    "variants": [
      {
        "size": "L",
        "stock": 4
      },
      {
        "size": "M",
        "stock": 6
      }
    ]
  },
  {
    "_id": "E",
    "name": "Сумка",
    "category": "accessories",
    "price": {
      "$numberDecimal": "200.00"
    },
    "variants": [
      {
        "size": "L",
        "stock": 2
      },
      {
        "size": "M",
        "stock": 1
      }
    ]
  },
  {
    "_id": "F",
    "name": "Шарф",
    "category": "accessories",
    "price": {
      "$numberDecimal": "150.00"
    },
    "variants": [
      {
        "size": "L",
        "stock": 1
      },
      {
        "size": "M",
        "stock": 8
      }
    ]
  }
]
</code>
</details>
<details>
<summary>ORDERS</summary>
<code>
[
  {
    "_id": "sale-start",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-01-01T00:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "A",
        "size": "M",
        "qty": 2,
        "price": {
          "$numberDecimal": "100.00"
        }
      },
      {
        "sku": "B",
        "size": "L",
        "qty": 4,
        "price": {
          "$numberDecimal": "80.00"
        }
      }
    ]
  },
  {
    "_id": "sale-middle",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-01-15T12:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "A",
        "size": "L",
        "qty": 1,
        "price": {
          "$numberDecimal": "120.00"
        }
      },
      {
        "sku": "C",
        "size": "M",
        "qty": 3,
        "price": {
          "$numberDecimal": "90.00"
        }
      }
    ]
  },
  {
    "_id": "sale-last",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-01-31T23:59:59.999Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "D",
        "size": "L",
        "qty": 2,
        "price": {
          "$numberDecimal": "120.00"
        }
      },
      {
        "sku": "E",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "200.00"
        }
      },
      {
        "sku": "F",
        "size": "L",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "excluded-end",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-02-01T00:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "A",
        "size": "M",
        "qty": 999,
        "price": {
          "$numberDecimal": "100.00"
        }
      }
    ]
  },
  {
    "_id": "excluded-before",
    "userId": 2500,
    "createdAt": {
      "$date": "2025-12-31T23:59:59.999Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "A",
        "size": "M",
        "qty": 999,
        "price": {
          "$numberDecimal": "100.00"
        }
      }
    ]
  },
  {
    "_id": "excluded-created",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-01-15T00:00:00Z"
    },
    "status": "created",
    "items": [
      {
        "sku": "A",
        "size": "M",
        "qty": 999,
        "price": {
          "$numberDecimal": "100.00"
        }
      }
    ]
  },
  {
    "_id": "excluded-cancelled",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-01-15T00:00:00Z"
    },
    "status": "cancelled",
    "items": [
      {
        "sku": "A",
        "size": "M",
        "qty": 999,
        "price": {
          "$numberDecimal": "100.00"
        }
      }
    ]
  },
  {
    "_id": "H00",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T00:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H01",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T00:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H02",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T01:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H03",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T01:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H04",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T02:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H05",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T02:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H06",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T03:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H07",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T03:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H08",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T04:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H09",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T04:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H10",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T05:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H11",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T05:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H12",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T06:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H13",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T06:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H14",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T07:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H15",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T07:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H16",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T08:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H17",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T08:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H18",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T09:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H19",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T09:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H20",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T10:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H21",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T10:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H22",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T11:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H23",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T11:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H24",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T12:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "H25",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-03-01T12:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "other-user",
    "userId": 2501,
    "createdAt": {
      "$date": "2026-04-01T00:00:00Z"
    },
    "status": "paid",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  },
  {
    "_id": "other-status",
    "userId": 2500,
    "createdAt": {
      "$date": "2026-04-01T00:00:00Z"
    },
    "status": "cancelled",
    "items": [
      {
        "sku": "F",
        "size": "M",
        "qty": 1,
        "price": {
          "$numberDecimal": "150.00"
        }
      }
    ]
  }
]
</code>
</details>

Расммотрим январь 2026 года, исклбчим все без статуса `paid`. И, например, товар `B` встречается всего раз (один заказ разобран на части, в зависимости от наполнения):
```bash
{
    "sku": "B",
    "size": "L",
    "qty": 4,
    "price": {
        "$numberDecimal": "80.00"
    }
}
```
При аггрегации получаем, что для `B` id кол-во проданных едниц - 4, - а выручка `Σ(qty * price)` = 320 (используется цена из заказа, не из коллекции `products`). Название из коллекции `products` "Рубашка". Подобное происходит для каждого предмета в заказах. 

## 4. Индекс и план выполнения

> **Сравните результаты.** Сохраните полные планы и подтвердите неизменность списка `_id` и его порядка. Заполните таблицу в README.

| Вариант | nReturned | totalKeysExamined | totalDocsExamined | Есть отдельная SORT |
|---|---:|---:|---:|---|
| До индекса | 20 | 0 | 10000 | Есть |
| После индекса | 20 | 20 | 20 | Нет |

> **Объясните наблюдение.** Обоснуйте порядок полей индекса, опишите изменение работы запроса и одну издержку индекса. Если применяли `hint`, укажите это явно.

Составной индекс содержит поля `userId`, `status`, `createdAt`, `_id`. Первые два поля используются для точного равенства - они выделят участок индекса с нужным покупателем и статусом. Поля же сортировки `createdAt` и `_id` - позволят уже хранить данные внутри в отсортированном виде.

До создания индекса MongoDB использовала просмотр коллекции: проверила 10000 документов и выполнила отдельную сортировку. Ключи индекса при этом не просматривались. 

После создания индекса запрос просмотрел 20 ключей и 20 документов, отдельная стадия сортировки исчезла. Индекс позволил получать подходящие заказы в нужном порядке и завершить чтение после достижения ограничения в `limit`.

Издержка индекса - дополнительная работа при записи.

## 5. Оформление заказа

> **Объясните в README** различие атомарности и защиты от повторного бизнес-запроса, настройки транзакции и то, как ваш код приводит к commit или abort.

### Атомарность и защита от повтора

Атомарность означает, что списание остатков двух товаров и создание заказа фиксируются вместе. Если часть из шагов не выполнима, то все откатывается целиком.

Защита от повторого безнес-запроса решает проблему того, что покупка совершается лишь раз. Поэтому передается `operationId`, как id заказа. И уже в коде идет обработка того, есть ли такой уже заказ и что в таком случае сделать.

### Настройки транзакции

| Настройка | Значение | Назначение |
|---|---|---|
| `readPreference` | `primary` | Чтения выполняются на primary |
| `readConcern` | `snapshot` | Чтения транзакции используют общий снимок данных |
| `writeConcern.w` | `majority` | Commit требует подтверждения большинством |
| `writeConcern.j` | `true` | Требуется дождаться журналирования на необходимых для подтверждения узлах |
| `writeConcern.wtimeout` | `10000` | Ограничивается ожидание подтверждения репликации |

### Путь к commit или abort

Например, если продукта, который хотят приобрести, нет, то выбрасывается ошибка с кодом `ITEM_UNAVAILABLE`. На этом этапе, никакая транзакция еще не была отменена, только выброс из функции. 

Теперь код в lab.js ловит исключение, вызывает `session.abortTransaction()` и передает ошибку дальше. И теперь транзакция отменена. 

При успехе же, мы возвращаем созданный документ, затем код сверху вызывает `session.commitTransaction()` и закрывает транзакцию. 

## 6. Смена primary

> **Объясните в README**, как клиент находит новый primary и почему majority-подтверждение не гарантирует немедленно актуальное чтение с любой secondary.

Клиент общается с участниками топологии через драйвер, который поддерживает о ней информацию и отслеживает роли внутри. После того, как необходимо было выбирать нового primary - драйвер получил сообщение, что теперь этот узел стал им и направлять запросы клиента на него (чтения `readPreference`).

Наши настройки `w: "majority", j: true` говорят о том, чтобы дождать подтверждения записи в журналы от 2 из 3 узлов. Но запись в журналы не равна приминению этих самых изменений. Именно поэтому нет гарантии на актуальное чтение с любой secondary даже после majority-подтверждения. 

## 7. Шардирование

> Коллекция `orders` распределена по ключу `{userId: 1}`; карта неизменна, на обоих шардах есть документы.

| Диапазон userId | Шард |
|---|---|
| Меньше 2000 | A |
| От 2000 включительно до 5000 исключительно | B |
| От 5000 включительно | A |

> 1. Какие шарды опросит `mongos` для истории `{userId: 2500}`, а какие — для общего отчёта по `{status: "paid"}`? Обоснуйте оба ответа.

Будет опрошен шард В, так как значение ключа шардирование позволит определить нужный маршрут. 

Для общего отчета по всем двум шардам, так как подходящие данные могут быть на обоих шардах и без подсказки в виде ключа шардирования, нужно проверить все.

> 2. Изменится ли число участников второго запроса после создания индекса `{status: 1}` внутри шардов? Какую работу этот индекс способен изменить?

Нет, не измениться. Индекс может сократить работу поиска внутри каждого шарда только.
