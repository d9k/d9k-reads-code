# ecommerce-by-golang-app

## Architecture from README, 2026.08.30

> :speaking_head_in_silhouette: -> :robot_face::
> Прочитай
> - Context map
> - Relationship types
> из README и объясни по-русски архитектурные решения, и почему они были сделаны

:robot_face: Kat Coder 2:

### Доменные контексты бэкенда

Бэкенд разбит на независимые доменные контексты:

- productcatalog — продукты, вариации, атрибуты, категории,-stock
- cart — корзины покупок
- checkout — заказы (event-sourced + CQRS)
- auth — пользователи, сессии, роли
- shippinginfo — сохранённые адреса
- search — поиск
- layout — слой презентации (HTMX storefront + admin panel)

### Структура каждого доменного контекста бэкенда

- domain — сущности, value objects, инварианты
- app — application services / use cases
- adapter — сохранение данных в базу данных (Postgres + in-memory реализация)
- `bounded_context.go` - точка для связи с другими доменами

### Обоснование архитектуры

Каждый контекст владеет своими данными и имеет чёткую ответственность. Это позволяет разработчикам работать независимо, уменьшает связанность и упрощает поддержку.

Это реализация гексагональной архитектуры (ports & adapters). Доменная логика изолирована от инфраструктуры, что позволяет легко тестировать и заменять реализации.

### Отношения между контекстами

#### ACL

Anti-Corruption Layer: доменный контекст `cart/` не импортирует типы доменного контекста `productcatalog/` напрямую. Вместо этого есть слой трансформации: адаптер `transformProductCatalog` который преобразует `productcatalog.Variant` в собственную модель `domain.Product`.

:open_file_folder: `backend/cmd/web/main.go`

```go
import (
	"github.com/bkielbasa/go-ecommerce/backend/cart"
)

func main() {
	// . . . . .
	cartBD, cartSrv := cart.New( // C: usage
		db,
		logger,
		catalogService
	)
	// . . . . .
}
```

:open_file_folder: `backend/cart/app/cart.go`

Порт потребителя - ProductCatalog - относится к архитектурному паттерну «Гексагональная архитектура» (она же Ports and Adapters). **Порт потребителя** — это **интерфейс уровня приложения**, который определяет, какие действия внешние клиенты (потребители) могут совершать с нашей бизнес-логикой.

```go
type CartService struct {
	storage        CartStorage
	productCatalog ProductCatalog // B: usage
}

type ProductCatalog interface { // B: declaration
	Find(ctx context.Context, variantID string) (domain.Product, error)
}

func (c CartService) AddToCart(ctx context.Context, sessID string, variantID string, qty int) error {
	p, err := c.productCatalog.Find(ctx, variantID) // 1: usage
}

func NewCartService( // A: declaration
	storage CartStorage, pc ProductCatalog
) CartService {
	return CartService{storage: storage, productCatalog: pc}
}
```

:open_file_folder: `./backend/cart/bounded_context.go`:

```go
package cart

import ( pcdomain "github.com/bkielbasa/go-ecommerce/backend/productcatalog/domain" )

/** Только domain импортируется из productCatalog  */
import(pcdomain "github.com/bkielbasa/go-ecommerce/backend/productcatalog/domain")

func New(db *sql.DB, logger logrus.FieldLogger, pc productStorage) (application.BoundedContext, app.CartService) { // C: declaration
	// . . . . .
	srv := app.NewCartService( // A: usage
		storage,
		transformProductCatalog{pc} // 2: usage -> B: fulfilling interface
	)
	// . . . . .
}

func (
	tpc transformProductCatalog, // 2: usage
) Find( // 1: declaration
	ctx context.Context,
	variantID string,
) (domain.Product, error) {
	p, v, err := tpc.pc.FindVariant(ctx, variantID) // 4: usage
	/** Берутся ошибки из другого домена, заменяются на ошибки из своего домена, иногда с таким же названием и текстом. Пример: domain.ErrProductNotFound <- pcdomain.ErrProductNotFound */
	// . . . . .
	domain.NewProduct(v.ID(), name, v.Price().Amount(), cur), nil
}

type transformProductCatalog struct { // 2: definition
	pc productStorage // 3: usage
}

type productStorage interface { // 3: definition
	FindVariant(ctx context.Context, variantID string) (pcdomain.Product, pcdomain.Variant, error) // 4: declaration
}
```

\< Граница доменные контекстов: \>

```
^        |
|        |   product catalog
cart     v
```

:open_file_folder: `backend/productcatalog/app/product.go`:

```go
func (ps ProductService) FindVariant(ctx context.Context, variantID string) (domain.Product, domain.Variant, error) { // 4: definition
	return ps.storage.FindVariant(ctx, variantID) // 5: usage
}
```

:open_file_folder: `backend/productcatalog/adapter/postgres.go`:

```go
func (db postgres) FindVariant(ctx context.Context, variantID string) (domain.Product, domain.Variant, error) { // 5: definition
	err := db.db.QueryRowContext(ctx, `SELECT product_id FROM productcatalog_variant WHERE id = $1`, variantID).Scan(&productID)
	// . . . . .
}
```

Это защищает `cart` от изменений в vocabulary productcatalog.

> 🗣️️ Как-то многовато слоёв. Это самая короткая имплементация из возможных или возможно короче?

🤖 Qwen Flash:

Пункт README «Anti-Corruption Layer» (`README.md`) говорит, что трансформация «**re-checking stock and currency**» — то есть повторная проверка остатка и валюты — это и есть назначение этих строк. ACL не просто конвертирует типы из формата A в формат B — он **валидирует данные чужого контекста по своим инвариантам** перед тем, как впустить их в свой домен. Убрав эти четыре блока, вы превратите `transformProductCatalog` в тупой type-cast, и инварианты cart перестанут защищаться от изменений словари `productcatalog` (завтра каталог начнёт возвращать `Currency("EU")` или снятый с продажи вариант — и это молча протечёт в корзину).

в рамках выбора «ACL» это минимальная имплементация. «Многовато слоёв» — это не раздутость кода, а фиксированная налоговая ставка паттерна: 1 порт + 1 адаптер + 1 локальный интерфейс. Всё остальное в вашей цепочке (`New`, `NewCartService`, делегация в `ProductService`) — это обычная композиция гексагональной архитектуры, которая была бы и без ACL.

> 🗣️️ Короче, DDD похож на выделение микросервисов из монолита

🤖 Qwen Flash: Аналогия верна наполовину — и именно в этом суть
Сходство (почему вы так решили): границы bounded contexts действительно выглядят как границы микросервисов: свои модели (у cart свой Product, у productcatalog свой), свои таблицы в миграциях с префиксом контекста (`cart_*`, `checkout_*`, `productcatalog_*`), свои интеграционные паттерны (ACL, Published Language, Outbox/Inbox) — это ровно тот же инструментарий, что применяют при проектировании сервисов. Есть даже поговорка: «bounded context — это граница, по которой микросервис можно вырезать».

#### Ссылки на Customer-Supplier и Conformist

Проект выбирает `Conformist` там, где у слоя нет доменных правил (layout), и `Customer-Supplier` там, где есть критичные инварианты (резерв стока, оплата заказа).

Посмотрите на import'ы потребителя: чужой domain в подписях портов = conformist (layout); свои/примитивные типы = customer договорился о форме (checkout).

##### Conformist

:open_file_folder: `backend/layout/bounded_context.go`:

`type catalogService interface`: `FindVariant()` , `List()`, `CreateAttributeType()`

У presentation-слоя нет собственных инвариантов — его работа «взять и отрендерить». Writing ACL стоил бы сотнями строк мапперов ради защиты типа, который всё равно только печатают. `README.md` так и формулирует: «A presentation layer that just renders is a natural conformist».

Интерфейс объявлен приватно (с мал. буквы) в layout, но форма повторяет поставщика 1-в-1. Это и значит «conformist: принимаешь словарь поставщика как есть, без перевода».

##### **Порты `Customer-Supplier` (объявлены потребителем `checkout`):**

- `CartReader` в :open_file_folder: `backend/checkout/bounded_context.go` — в корневом файле контекста (дубль в `CartReader interface` в `backend/checkout/app/checkout.go`) с комментарием: _«let New accept the cart service... without importing cart.app directly»_.)
	- Таблица методов не совпадает с `ProductService`: каталог умеет десятки операций, `checkout` попросил ровно две.
	- Поставщик знает о договорённости и её держит: комментарий у `ProductService.Record()`: _«Intended as the StockMovements seam for the checkout context»_. Это и есть _«supplier agrees to honour the contract»_ из статьи [Customer Supplier | ddd-practitioners](https://ddd-practitioners.com/home/glossary/bounded-context/bounded-context-relationship/customer-supplier/) — отдельный метод заведён ради конкретного заказчика.
- `StockReserver` :open_file_folder: `backend/checkout/bounded_context.go` — «implemented by the product catalogue; checkout reserves stock atomically».
- `StockMovements` в :open_file_folder: `backend/checkout/bounded_context.go` — audit-log seam, тоже реализуется каталогом.

### Тесты проверяют соблюдение интерфейсов

:open_file_folder: `backend/integration/orderpaid_test.go`: комментарий _«If the `OrderPaid` integration event's shape changes (fields added/renamed), or a subscriber is added/removed/renamed , or the at-least-once delivery contract from the outbox dispatcher is altered, this test breaks and you'll  notice...»_

### TODO

 app — application services / use cases. Поподробнее, что такое application services?

## Event bus

:open_file_folder: `backend/internal/eventbus/eventbus.go`

- [ ] TODO как работает?

