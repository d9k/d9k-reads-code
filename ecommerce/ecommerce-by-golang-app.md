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

Anti-Corruption Layer: доменный контекст `cart/` не импортирует типы доменного контекста `productcatalog/` напрямую. Вместо этого есть слой трансформации `transformProductCatalog` который преобразует `productcatalog.Variant` в собственную модель `domain.Product`.

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


### TODO

 app — application services / use cases. Поподробнее, что такое application services?
