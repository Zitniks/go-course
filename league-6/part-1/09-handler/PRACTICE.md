# Задание

Напиши REST API для магазина с хранением товаров в памяти. Протестируй все эндпоинты через Postman.

---

## Структура проекта

```
cmd/
  main.go
internal/
  domain/product.go      ← модель и in-memory хранилище
  handler/product.go     ← HTTP-хендлеры
```

---

## Шаг 1 — Модель и хранилище

Создай `internal/domain/product.go`.

Опиши структуру товара с JSON-тегами:

```go
type Product struct {
    ID       int     `json:"id"`
    Name     string  `json:"name"`
    Price    float64 `json:"price"`
    Quantity int     `json:"quantity"`
}
```

Создай хранилище на основе map. Для защиты от гонки данных используй `sync.RWMutex` — эта тема пройдена в лекции по горутинам:

```go
type Store struct {
    mu       sync.RWMutex
    products map[int]Product
    nextID   int
}

func NewStore() *Store
func (s *Store) List(search string) []Product
func (s *Store) Get(id int) (Product, bool)
func (s *Store) Create(p Product) Product
func (s *Store) Update(id int, p Product) (Product, bool)
func (s *Store) Delete(id int) bool
```

> `List` фильтрует по подстроке в названии если передан `search`. При пустом `search` возвращает все товары.

---

## Шаг 2 — Хендлеры

Создай `internal/handler/product.go`.

Хендлеры принимают Store через структуру — не используй глобальные переменные:

```go
type ProductHandler struct {
    store *domain.Store
}

func New(store *domain.Store) *ProductHandler
```

Вынеси вспомогательные функции для ответов:

```go
func writeJSON(w http.ResponseWriter, status int, v any) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    json.NewEncoder(w).Encode(v)
}

func writeError(w http.ResponseWriter, status int, msg string) {
    writeJSON(w, status, map[string]string{"error": msg})
}
```

Реализуй пять методов:

| Метод | Эндпоинт | Поведение |
|---|---|---|
| `List` | `GET /products` | Query-параметр `?search=` фильтрует по названию |
| `GetByID` | `GET /products/{id}` | 404 если товар не найден |
| `Create` | `POST /products` | Валидация: имя непустое, цена > 0, количество ≥ 0 |
| `Update` | `PATCH /products/{id}` | Обновляет только переданные поля, 404 если нет товара |
| `Delete` | `DELETE /products/{id}` | 204 No Content при успехе |

---

## Шаг 3 — Роутинг и запуск

В `main.go` собери всё вместе.

Используй стандартный `net/http` с `http.NewServeMux()`. В Go 1.22+ метод и параметры пути указываются прямо в паттерне:

```go
store := domain.NewStore()
h := handler.New(store)

mux := http.NewServeMux()
mux.HandleFunc("GET /products", h.List)
mux.HandleFunc("GET /products/{id}", h.GetByID)
mux.HandleFunc("POST /products", h.Create)
mux.HandleFunc("PATCH /products/{id}", h.Update)
mux.HandleFunc("DELETE /products/{id}", h.Delete)

log.Println("starting server on :8080")
log.Fatal(http.ListenAndServe(":8080", mux))
```

---

## Шаг 4 — Проверь через Postman

Создай коллекцию `Shop API` и добавь в неё запросы:

**Создать товар**
- Метод: `POST`, адрес: `http://localhost:8080/products`
- Body → raw → JSON:
```json
{
  "name": "Ноутбук",
  "price": 79999,
  "quantity": 5
}
```
- Ожидаемый статус: `201 Created`

**Получить список**
- Метод: `GET`, адрес: `http://localhost:8080/products`
- Ожидаемый статус: `200 OK`

**Поиск по названию**
- Метод: `GET`, адрес: `http://localhost:8080/products`
- Params → добавь ключ `search`, значение `ноут`
- Ожидаемый статус: `200 OK`, в ответе только подходящие товары

**Получить один товар**
- Метод: `GET`, адрес: `http://localhost:8080/products/1`
- Ожидаемый статус: `200 OK`
- Попробуй несуществующий id — должен вернуться `404`

**Обновить цену**
- Метод: `PATCH`, адрес: `http://localhost:8080/products/1`
- Body → raw → JSON:
```json
{
  "price": 74999
}
```
- Ожидаемый статус: `200 OK`, остальные поля не изменились

**Удалить товар**
- Метод: `DELETE`, адрес: `http://localhost:8080/products/1`
- Ожидаемый статус: `204 No Content`
- Повтори GET на тот же id — должен вернуться `404`

После выполнения прикрепи скриншоты всех запросов из Postman.

