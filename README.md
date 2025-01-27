# Грокаем по 11 спринт:

## Введение
*Грок (от англ. to grok) — термин из романа Роберта Хайнлайна «Чужак в стране чужой», означающий глубокое понимание, познание до самой сути объекта.*

## Ключевые вызовы спринта
В 11 спринте вам предстоит справиться с двумя основными задачами:
1. Работа с многопоточностью и устранение состояния гонки
2. Реализация запросов для экрана профиля

## Работа с профилем
### Структура сервисов
Вам предстоит реализовать два основных сервиса:
- ProfileService
- ImageService

### Основные компоненты каждого сервиса
1. Свойства для хранения объектов и констант
2. Метод формирования URLRequest
3. Метод получения данных

### Важные аспекты реализации
#### Работа с запросами
- Формирование корректного URLRequest
- Правильная обработка Bearer Token в заголовках
```swift
var request = URLRequest(url: url)
request.httpMethod = "GET"
request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
```

#### Обработка данных
- Корректная работа с многопоточностью
- Обработка result и потенциальных ошибок
- Правильное моделирование данных для парсинга
```swift
func fetchProfile(with token: String, completion: @escaping (Result<Profile, any Error>) -> Void) {
        
        assert(Thread.isMainThread)
        
        if task != nil {
            if lastToken != token {
                task?.cancel()
            } else {
                completion(.failure(AuthServiceError.invalidRequest))
                return
            }
        } else {
            if lastToken == token {
                completion(.failure(AuthServiceError.invalidRequest))
                return
            }
        }
        
        lastToken = token
        
        guard let request = makeProfileResultRequest() else {
            completion(.failure(AuthServiceError.invalidRequest))
            return
        }
        
        let task = urlSession.objectTask(for: request) { [weak self] (result: Result<ProfileResponseResult, Error>) in
            guard let self else { preconditionFailure("self is unavalible") }
            switch result {
            case .success(let profileResponseResult):
                let profile = Profile(from: profileResponseResult)
                self.profile = profile
                completion(.success(profile))
            case .failure(let error):
                print("ProfileService Error - \(error)")
                completion(.failure(error))
            }
            self.task = nil
            self.lastToken = nil
        }
        self.task = task
        task.resume()
    }
```

## Советы по разработке

### Работа с кодом
- Делайте частые коммиты работающего кода
- Создавайте отдельные ветки для экспериментов
- Используйте систему контроля версий для безопасного тестирования гипотез

### Отладка и тестирование
1. Используйте Postman для проверки API-запросов:
   - Тестирование URL
   - Проверка работы с токеном
   - Валидация заголовков

2. Применяйте инструменты отладки:
   - Расставляйте брейкпоинты
   - Проверяйте жизненный цикл данных
   - Отслеживайте значения nil

### Распространенные проблемы
- Неверная конфигурация URLRequest
- Ошибки в Bearer Token
- Некорректная модель данных
- Проблемы с многопоточностью
- Неправильный timing при загрузке данных (например, в viewDidLoad)

## Полезные ресурсы
- **[Официальная документация](https://developer.apple.com/documentation/devicemanagement/implementing-the-simple-authentication-user-enrollment-flow)**.
- Материалы по OAuth 2.0
  **[статья](https://habr.com/ru/companies/vk/articles/115163/)**
   и **[туториал](https://www.kodeco.com/243-oauth-2-0-with-swift-tutorial/page/3?page=3#toc-anchor-014)**.
- Статьи по многопоточности
  **[статья раз](https://habr.com/ru/articles/320152/)**,
  **[статья два](https://habr.com/ru/articles/578752/)**.
- Туториалы по работе с сетью
  **[туториал](https://www.kodeco.com/28540615-grand-central-dispatch-tutorial-for-swift-5-part-1-2)**.
## Заключение
Помните: сложности временны, а полученные знания останутся с вами навсегда. Не стесняйтесь обращаться за помощью к наставникам и коллегам — вместе вы справитесь с любыми задачами.
