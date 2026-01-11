## Базовые типы данных

### Event
Структура, представляющая событие (ивент).

| Поле | Тип | Описание | Обязательность |
|------|-----|----------|----------------|
| id | string | Уникальный идентификатор события | Обязательно |
| title | string | Название события | Обязательно |
| description | string | Описание события | Обязательно |
| starts_at | google.protobuf.Timestamp | Дата и время начала события | Обязательно |

## Методы сервиса

### GetEvents
Получение списка всех доступных событий.

**Запрос:** `GetEventsRequest` (пустой запрос)

**Ответ:** `GetEventsResponse`
```protobuf
message GetEventsResponse {
  repeated Event events = 1; // Массив событий
}
```

**Пример вызова:**
```go
response, err := client.GetEvents(ctx, &event.GetEventsRequest{})
```

### GetEvent
Получение информации о конкретном событии по его ID.

**Запрос:** `GetEventRequest`
```protobuf
message GetEventRequest {
  string event_id = 1; // ID запрашиваемого события
}
```

**Ответ:** `GetEventResponse`
```protobuf
message GetEventResponse {
  Event event = 1; // Объект события
}
```

**Пример вызова:**
```go
req := &event.GetEventRequest{EventId: "12345"}
response, err := client.GetEvent(ctx, req)
```

### RegisterUser
Регистрация пользователя на указанное событие.

**Запрос:** `RegisterUserRequest`
```protobuf
message RegisterUserRequest {
  string event_id = 1;   // ID события для регистрации
  int64 chat_id = 2;     // Идентификатор чата пользователя в Telegram
  string username = 3;   // Имя пользователя
}
```

**Ответ:** `RegisterUserResponse`
```protobuf
message RegisterUserResponse {
  bool success = 1; // Флаг успешности регистрации
}
```

**Пример вызова:**
```go
req := &event.RegisterUserRequest{
    EventId: "12345",
    ChatId: 123456789,
    Username: "john_doe",
}
response, err := client.RegisterUser(ctx, req)
```

## Обработка ошибок

Сервис использует стандартные gRPC статусы:
- `OK` (0) - успешное выполнение
- `ALREADY EXISTS` (6) - уже существует
- `INTERNAL` (13) - внутренняя ошибка сервера

## Примеры использования

### Получение всех событий
```go
// GetEvents метод для получения всех событий
func (c *Client) GetEvents(ctx context.Context) ([]*pb.Event, error) {
    // Отправляем запрос на другой микросервис
    response, err := c.client.GetEvents(ctx, &pb.GetEventsRequest{})
    if err != nil {
    c.log.Error("error", err.Error(), slog.String("operation", opGetEvents))
        return nil, fmt.Errorf("%s: %w", opGetEvents, err)
    }
    c.log.Info("getting events successfully", slog.Int("count", len(response.Events)), slog.String("operation", opGetEvents))
    return response.GetEvents(), nil
}
```

### Регистрация пользователя
```go
// RegisterUser метод для регистрации пользователя на конкретное событие
func (c *Client) RegisterUser(ctx context.Context, eventID string, chatID int64, username string) (bool, error) {
    response, err := c.client.RegisterUser(ctx, &pb.RegisterUserRequest{EventId: eventID, ChatId: chatID, Username: username})
    if err != nil {
    c.log.Error("error", err.Error(), slog.String("operation", opRegisterUser))
    return false, fmt.Errorf("%s: %w", opRegisterUser, err)
    }
    c.log.Info("register user on event successfully", slog.String("event_id", eventID), slog.String("username", username), slog.String("operation", opRegisterUser))
    return response.GetSuccess(), nil
}
```

## Особенности реализации

1. **Идентификаторы событий** должны быть уникальными строковыми значениями
2. **ChatID** представляет собой числовой идентификатор чата в Telegram
3. **Username** - имя пользователя в Telegram (без символа @)
4. Временные метки используют формат `google.protobuf.Timestamp`

## Зависимости

- `google/protobuf/timestamp.proto` - для работы с временными метками

## Примечания

1. Сервис предназначен для интеграции с Telegram-ботом
2. Все временные метки должны быть в UTC
3. При регистрации пользователя рекомендуется дополнительно валидировать входные данные