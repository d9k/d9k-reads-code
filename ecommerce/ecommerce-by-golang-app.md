# ecommerce-by-golang-app

## Architecture from README, 2026.08.30

> :speaking_head_in_silhouette: -> :robot_face::
> Прочитай
> - Context map
> - Relationship types
> из README и объясни по-русски архитектурные решения, и почему они были сделаны

:robot_face: Kat Coder 2:

### Домены бэкенда

Бэкенд разбит на независимые домены (или доменные контексты):

- productcatalog — продукты, вариации, атрибуты, категории,-stock
- cart — корзины покупок
- checkout — заказы (event-sourced + CQRS)
- auth — пользователи, сессии, роли
- shippinginfo — сохранённые адреса
- search — поиск
- layout — слой презентации (HTMX storefront + admin panel)

### Структура каждого домена бэкенда

- domain — сущности, value objects, инварианты
- app — application services / use cases
- adapter — сохранение данных в базу данных (Postgres + in-memory реализация)
- `bounded_context.go` - точка для связи с другими доменами

### Обоснование архитектуры

Каждый контекст владеет своими данными и имеет чёткую ответственность. Это позволяет разработчикам работать независимо, уменьшает связанность и упрощает поддержку.

Это реализация гексагональной архитектуры (ports & adapters). Доменная логика изолирована от инфраструктуры, что позволяет легко тестировать и заменять реализации.

### Отношения между контекстами

#### ACL

Anti-Corruption Layer: Cart не импортирует типы productcatalog напрямую. Вместо этого есть слой трансформации ([`transformProductCatalog`](vscode-webview://1d60ov6f6ur3cmu6dhv03tl22jkia4f0a0gi65biva6l6dfao8ec/backend/cart/bounded_context.go)), который преобразует `productcatalog.Variant` в собственную модель `domain.Product`. Это защищает cart от изменений в vocabulary productcatalog.


### TODO

 app — application services / use cases. Поподробнее, что такое application services?
