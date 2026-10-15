# ecommerce-by-golang-app

## Architecture from README, 2026.08.30

> 🗣️️ -> :robot_face::
> Прочитай
> - Context map
> - Relationship types
> из README и объясни по-русски архитектурные решения, и почему они были сделаны

:robot_face: Kat Coder 2, Qwen Coder:

## Доменные контексты бэкенда

Бэкенд разбит на независимые доменные контексты:

- productcatalog — продукты, вариации, атрибуты, категории,-stock
- cart — корзины покупок
- checkout — заказы (event-sourced + CQRS)
- auth — пользователи, сессии, роли
- shippinginfo — сохранённые адреса
- search — поиск
- layout — слой презентации (HTMX storefront + admin panel). Контроллеры, вызывающие app-сервисы.

## Структура каждого доменного контекста бэкенда

- domain — сущности, value objects, инварианты, бизнес-правила, генерация событий.
- app — application services / use cases. Границы транзакции.
- adapter — сохранение данных в базу данных (Postgres + in-memory реализация)
- `bounded_context.go` - точка для связи с другими доменными контекстами

## Обоснование архитектуры

Каждый контекст владеет своими данными и имеет чёткую ответственность. Это позволяет разработчикам работать независимо, уменьшает связанность и упрощает поддержку.

Это реализация гексагональной архитектуры (ports & adapters). Доменная логика изолирована от инфраструктуры, что позволяет легко тестировать и заменять реализации.

## Отношения между контекстами: ACL

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

## Отношения между контекстами: Ссылки на Customer-Supplier и Conformist

Проект выбирает `Conformist` там, где у слоя нет доменных правил (layout), и `Customer-Supplier` там, где есть критичные инварианты (резерв стока, оплата заказа).

Посмотрите на import'ы потребителя: чужой domain в подписях портов = conformist (layout); свои/примитивные типы = customer договорился о форме (checkout).

### Conformist

:open_file_folder: `backend/layout/bounded_context.go`:

`type catalogService interface`: `FindVariant()` , `List()`, `CreateAttributeType()`

У presentation-слоя нет собственных инвариантов — его работа «взять и отрендерить». Writing ACL стоил бы сотнями строк мапперов ради защиты типа, который всё равно только печатают. `README.md` так и формулирует: «A presentation layer that just renders is a natural conformist».

Интерфейс объявлен приватно (с мал. буквы) в layout, но форма повторяет поставщика 1-в-1. Это и значит «conformist: принимаешь словарь поставщика как есть, без перевода».

### **Порты `Customer-Supplier` (объявлены потребителем `checkout`):**

- `CartReader` в :open_file_folder: `backend/checkout/bounded_context.go` — в корневом файле контекста (дубль в `CartReader interface` в `backend/checkout/app/checkout.go`) с комментарием: _«let New accept the cart service... without importing cart.app directly»_.)
	- Таблица методов не совпадает с `ProductService`: каталог умеет десятки операций, `checkout` попросил ровно две.
	- Поставщик знает о договорённости и её держит: комментарий у `ProductService.Record()`: _«Intended as the StockMovements seam for the checkout context»_. Это и есть _«supplier agrees to honour the contract»_ из статьи [Customer Supplier | ddd-practitioners](https://ddd-practitioners.com/home/glossary/bounded-context/bounded-context-relationship/customer-supplier/) — отдельный метод заведён ради конкретного заказчика.
- `StockReserver` :open_file_folder: `backend/checkout/bounded_context.go` — «implemented by the product catalogue; checkout reserves stock atomically».
- `StockMovements` в :open_file_folder: `backend/checkout/bounded_context.go` — audit-log seam, тоже реализуется каталогом.

## Отношения между контекстами: Open Host Service

Open Host Service — Published Language-пара: контекст-хозяин публикует единый, стабильный, «общий» API (host service), рассчитанный на многих неизвестных потребителей. Ключевое: хозяин не подстраивается под каждого клиента — все клиенты говорят на его языке (словаре), поэтому протокол один, а клиентов может быть сколько угодно.

:open_file_folder: `backend/search/domain/document.go`:

**Язык хозяина — единый value object `Document`:**

```go
// «The search context is an Open Host Service (OHS): it publishes a single value object — Document — together with a Kind discriminator... The search context itself does not know about products, blog posts, or anything else — it only knows about documents».
// «Adding a new producer requires no change here: a producer chooses a new Kind value (any non-empty string), translates its records to Documents and calls Indexer.Index. The set of known kinds is intentionally NOT a closed enum at the type level — it is open by design (that is the OHS pattern)».
// . . . . .
type Document struct { kind, id, title, body, url; tags, meta, updatedAt }
```

**Два публичных порта хозяина** в в :open_file_folder: `backend/search/app/service.go`: `Indexer` и `Querier`. В :open_file_folder: `backend/search/app/service.go` один `*Service` удовлетворяет обоим:

```go
type Indexer interface {
	Index(ctx context.Context, doc domain.Document) error
	Remove(ctx context.Context, kind domain.Kind, id string) error
}

type Querier interface {
	Search(ctx context.Context, q string, opts QueryOptions) ([]Hit, error)
}

// «One struct, two roles — production wires the same instance into both seams».
type Service struct {
	storage Storage
}

func (s *Service) Index(ctx context.Context, doc domain.Document) error {
// . . . . .
func (s *Service) Remove(ctx context.Context, kind domain.Kind, id string) error {
// . . . . .
func (s *Service) Search(ctx context.Context, q string, opts QueryOptions) ([]Hit, error) {
// . . . . .
```

Потребление: :open_file_folder: `backend/productcatalog/app/searchindexer.go`:

```go
type SearchIndexer interface {
	Index(ctx context.Context, doc searchdomain.Document) error
	Remove(ctx context.Context, kind searchdomain.Kind, id string) error
}
```

:open_file_folder: `backend/productcatalog/bounded_context.go`:

```go
func New(db *sql.DB, searchIdx app.SearchIndexer) (application.BoundedContext, app.ProductService) {
	appServ := app.NewProductService(storage).WithSearchIndexer(searchIdx)
```

:open_file_folder: `backend/productcatalog/app/product.go`: (интерфейс `SearchIndexer` доступен без `import`'а, потому что в том же пакете `app`)

```go
func (ps ProductService) WithSearchIndexer(idx SearchIndexer) ProductService {
	ps.searchIdx = idx
// . . . . .

type ProductService struct {
	searchIdx SearchIndexer
```

Отличия от других типов отношений:
Customer-Supplier - 1 заказчик ↔ 1 поставщик, личные договорённости, заказчик пишет интерфейс, нужный ему. В то время как в **OHS** N producers + M consumers, формирует интерфейс до появления клиента;

> 🗣️️: Можно ли заставить компилятор насильно проверить, что структура должна удовлетворить нескольким интерфейсам?

В Go нет ключевого слова `implements`, но есть приём: объявить **пакетную переменную с blank identifier**, значение которой — компилируемый литерал типа, присваиваемый интерфейсу. Можно было бы сделать так:
```go
var _ Indexer = (*Service)(nil)   // "указатель на Service должен быть Indexer'ом"
var _ Querier = (*Service)(nil)   // "и Querier'ом тоже"
```

- `nil`-указатель никуда не разыменовывается, рантайм-стоимости нет;
- `_` — переменная не создаётся для потребителей, мусора в API нет;

Реальный пример из приложения (ищите по `var _`):

:open_file_folder: `backend/payments/adapter/stripe_adapter.go`:

```go
var _ app.Provider = (*StripeProvider)(nil)
```

## Тесты проверяют соблюдение интерфейсов

***Interface churn*** - контракт (интерфейс, схема события, API) постоянно меняется: поля переименовываются, добавляются/уезжают, сигнатуры правятся. ***Churn*** (с англ. «отток, ворóт, трение») — в инженерном сленге означает частые, хаотичные изменения того, что должно быть стабильным.

- :open_file_folder: `backend/integration/orderpaid_test.go`: комментарий _«If the `OrderPaid` integration event's shape changes (fields added/renamed), or a subscriber is added/removed/renamed , or the at-least-once delivery contract from the outbox dispatcher is altered, this test breaks and you'll  notice...»_

- :open_file_folder: `backend/checkout/adapter/outbox_test.go`:
	- `TestExtractIntegrationEvents_JSONShapeMatchesIntegrationOrderPaid()` - JSON round-trip проверка
		- `//` _«If anyone renames a field on `integration.OrderPaid` without updating the producer, this assertion fails loudly.»_
		- `is.NoErr(json.Unmarshal(records[0].Payload, &got))`

- В :open_file_folder: `migrations/000035_checkout_events_payload_version.up.sql`добавляетя поле `checkout_events.payload_version`, чтобы при эволюции контракта старые сообщения можно было отличить.

- :open_file_folder: `backend/internal/eventcatalog/catalog.go`: типы событий `KindIntegration`/ `KindDomain`

## Event bus

Checkout реализован как event-sourced aggregate с отдельной CQRS read side.

Почему: Заказы — сложная область с множеством состояний и аудитом. Event sourcing даёт полный audit trail, возможность воспроизведения состояния и лёгкую интеграцию через события.

:open_file_folder: `backend/internal/eventbus/eventbus.go`

### Outbox — паттерн «Транзакционная почтовая тумбочка»

**Проблема, которую он решает:** допустим, checkout хочет «сохранить заказ в БД» **и** «опубликовать событие в шину». Это две разные системы. Если между ними упадёт процесс — получаем либо заказ без уведомления (корзина не очистится, письмо не уйдёт), либо уведомление о заказе, которого нет в БД. Атомарности между двумя системами «из коробки» не бывает.

Решение: вместо внешней отправки писать событие... в ту же самую БД, отдельной таблицей `outbox_event`, в той же транзакции. Эта таблица и есть «outbox» (тумбочка для исходящей почты):

Дальше
- …каждые `OUTBOX_INTERVAL` (по умолчанию `1s`, берётся из `docker-compose.yaml`
- …отдельный фоновый **диспетчер** `Dispatcher` из :open_file_folder:`backend/internal/outbox/dispatcher.go`
- …читает ещё не отправленные строки `Unsent()` в :open_file_folder: `backend/internal/outbox/postgres.go`
	- Реализация `Unsent()` в `backend/internal/outbox/postgres.go`
- …публикует их в event bus
	- :open_file_folder: `backend/internal/outbox/dispatcher.go`
		- `func dispatchOnce(ctx context.Context)`
			- `d.bus.PublishWithID(ctx, r.ID, event)`
			- 🡤 :open_file_folder: `backend/cmd/web/main.go`
				- `bus := eventbus.New(logger)`
			- :open_file_folder: `backend/internal/eventbus/eventbus.go`
				- `func PublishWithID(ctx context.Context, eventID int64, e Event) {`
- …и помечает `MarkSent()` в :open_file_folder: `backend/internal/outbox/postgres.go`.
	- `UPDATE outbox_event SET sent_at = now()`

## boundedContext, 2026.10.13

🗣️️ `backend/checkout/bounded_context.go`:

```go
func New(db *sql.DB, cart CartReader, outbox adapter.OutboxAppender, payment app.PaymentProcessor, stock StockReserver, movements StockMovements, taxStrategy domain.TaxStrategy, shippingStrategy domain.ShippingStrategy) (application.BoundedContext, app.CheckoutService, query.Service) {
	// . . . . .
	return &boundedContext{}, cmd, queries
}
```

— что за `BoundedContext`?

:robot_face: Qwen:

- :open_file_folder: `backend/cart/bounded_context.go`
	- `New(db *sql.DB, logger logrus.FieldLogger, pc productStorage): (application.BoundedContext, app.CartService) {`
		- return `&boundedContext{ logger: logger, }, srv`

- :open_file_folder: `backend/cmd/web/main.go`
	- `app.AddBoundedContext(cartBD)`
		- :arrow_right_hook: :open_file_folder: `backend/internal/application/appplication.go`
			- `if m, ok := bc.(MuxRegister); ok { 		m.MuxRegister(app.router)	}`


`bc.(MuxRegister)`: _"convert to `m` if `bc` implements `MuxRegister`"_. Т. е. приём «хочешь — реализуй роуты, не хочешь — просто проходим мимо».

:open_file_folder: `backend/internal/application/appplication.go`:

```go
type MuxRegister interface {
	MuxRegister(*mux.Router)
}
```

- :link: :open_file_folder: `backend/auth/bounded_context.go`
	- `func (m boundedContext) MuxRegister(r *mux.Router) {`
		- `r.HandleFunc("/api/v1/auth/login", m.httpHandler.Login).Methods("POST")`

🗣️️ Почему `type boundedContext` объявлен приватным (с  мал. буквы)?

:robot_face: Qwen: Ключевой момент Go, который часто упускают: неэкспортируемый тип можно вернуть из публичной функции — просто снаружи нельзя написать его имя.

- **Сужение API пакета.** Публичная поверхность `checkout` — фактически одна функция `New` (+ порты `CartReader`/`StockReserver` для wiring). Тип-оболочка — деталь реализации; завтра её поля изменятся или её вообще удалят — и это **не будет breaking change** для потребителей, потому что никто не мог на неё сослаться.
- Будь типы публичными, пришлось бы изобретать `CheckoutBoundedContext`, `CartBoundedContext` — и это было бы почти бесполезной номенклатурой.
- Внутри пакета `checkout` префикс избыточен: `checkout.CheckoitBoundedContext` читается как «масло масляное».

Идиома Go: **экспортируй минимум** («expose only what clients need»). Заметьте контраст в том же файле :open_file_folder: `backend/checkout/bounded_context.go`: `CartReader`, `StockReserver` — публичные, они **нужны** `main.go` для правильной сборки, передачи в `New()`.

- Единственное «но» (осознанная жертва): Диспетчер или лог не сможет по имени типа отличить, какой контекст зарегистрировал модуль — но проекту это и не требуется: `main.go` сам знает порядок регистрации (вызовов `app.AddBoundedContext()`), а полиморфизм нужен лишь через `MuxRegister`. Так что приватность здесь ничего не теряет и много даёт.