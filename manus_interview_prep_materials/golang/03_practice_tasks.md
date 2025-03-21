# Практические задачи по Golang для подготовки к интервью

В этом документе представлены практические задачи по Golang, которые помогут вам закрепить основные концепции языка и подготовиться к техническому интервью. Задачи разделены по категориям и включают решения с пояснениями.

## Базовые задачи

### Задача 1: Переворот строки

**Задание**: Напишите функцию, которая переворачивает строку.

**Решение**:
```go
func reverseString(s string) string {
    runes := []rune(s) // Преобразуем строку в слайс рун для корректной работы с Unicode
    for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
        runes[i], runes[j] = runes[j], runes[i]
    }
    return string(runes)
}

// Пример использования:
// reverseString("Hello, 世界") -> "界世 ,olleH"
```

**Пояснение**: 
- Мы используем тип `rune` для корректной работы с Unicode символами
- Применяем алгоритм "два указателя" для обмена символов с начала и конца строки
- Временная сложность: O(n), где n - длина строки
- Пространственная сложность: O(n) для хранения слайса рун

### Задача 2: Пересечение двух массивов

**Задание**: Даны два массива: [1, 2, 3, 2, 0] и [5, 1, 2, 7, 3, 2]. Верните их пересечение [1, 2, 2, 3].

**Решение**:
```go
func intersection(nums1, nums2 []int) []int {
    // Создаем карту для подсчета элементов первого массива
    countMap := make(map[int]int)
    for _, num := range nums1 {
        countMap[num]++
    }
    
    // Находим пересечение
    result := []int{}
    for _, num := range nums2 {
        if count, exists := countMap[num]; exists && count > 0 {
            result = append(result, num)
            countMap[num]--
        }
    }
    
    return result
}

// Пример использования:
// intersection([]int{1, 2, 3, 2, 0}, []int{5, 1, 2, 7, 3, 2}) -> [1, 2, 2, 3]
```

**Пояснение**:
- Используем карту для подсчета вхождений каждого элемента в первом массиве
- Проходим по второму массиву и добавляем элементы в результат, если они есть в карте
- Уменьшаем счетчик в карте, чтобы учесть повторяющиеся элементы
- Временная сложность: O(n + m), где n и m - длины массивов
- Пространственная сложность: O(min(n, m)) для хранения карты

### Задача 3: Группировка анаграмм

**Задание**: Дан массив строк ["eat", "tea", "tan", "ate", "nat", "bat"]. Сгруппируйте анаграммы вместе.

**Решение**:
```go
func groupAnagrams(strs []string) [][]string {
    groups := make(map[string][]string)
    
    for _, str := range strs {
        // Сортируем символы строки для получения ключа
        chars := []rune(str)
        sort.Slice(chars, func(i, j int) bool {
            return chars[i] < chars[j]
        })
        key := string(chars)
        
        // Добавляем строку в соответствующую группу
        groups[key] = append(groups[key], str)
    }
    
    // Преобразуем карту в слайс групп
    result := make([][]string, 0, len(groups))
    for _, group := range groups {
        result = append(result, group)
    }
    
    return result
}

// Пример использования:
// groupAnagrams([]string{"eat", "tea", "tan", "ate", "nat", "bat"}) -> 
// [["eat", "tea", "ate"], ["tan", "nat"], ["bat"]]
```

**Пояснение**:
- Используем отсортированную строку как ключ для группировки анаграмм
- Все анаграммы после сортировки дают одинаковую строку
- Временная сложность: O(n * k * log(k)), где n - количество строк, k - максимальная длина строки
- Пространственная сложность: O(n * k)

## Задачи на структуры данных

### Задача 4: Реализация стека

**Задание**: Реализуйте структуру Stack с методами Push, Pop и IsEmpty.

**Решение**:
```go
type Stack struct {
    items []interface{}
}

// Создание нового стека
func NewStack() *Stack {
    return &Stack{
        items: make([]interface{}, 0),
    }
}

// Добавление элемента в стек
func (s *Stack) Push(item interface{}) {
    s.items = append(s.items, item)
}

// Извлечение элемента из стека
func (s *Stack) Pop() (interface{}, error) {
    if s.IsEmpty() {
        return nil, errors.New("stack is empty")
    }
    
    index := len(s.items) - 1
    item := s.items[index]
    s.items = s.items[:index]
    return item, nil
}

// Проверка, пуст ли стек
func (s *Stack) IsEmpty() bool {
    return len(s.items) == 0
}

// Просмотр верхнего элемента без извлечения
func (s *Stack) Peek() (interface{}, error) {
    if s.IsEmpty() {
        return nil, errors.New("stack is empty")
    }
    
    return s.items[len(s.items)-1], nil
}

// Пример использования:
// stack := NewStack()
// stack.Push(1)
// stack.Push(2)
// val, _ := stack.Pop() // val = 2
// isEmpty := stack.IsEmpty() // false
```

**Пояснение**:
- Используем слайс для хранения элементов стека
- Push добавляет элемент в конец слайса
- Pop удаляет и возвращает последний элемент
- IsEmpty проверяет, пуст ли стек
- Временная сложность операций: Push - O(1) амортизированно, Pop - O(1), IsEmpty - O(1)

### Задача 5: Реализация очереди

**Задание**: Реализуйте структуру Queue с методами Enqueue, Dequeue и IsEmpty.

**Решение**:
```go
type Queue struct {
    items []interface{}
}

// Создание новой очереди
func NewQueue() *Queue {
    return &Queue{
        items: make([]interface{}, 0),
    }
}

// Добавление элемента в очередь
func (q *Queue) Enqueue(item interface{}) {
    q.items = append(q.items, item)
}

// Извлечение элемента из очереди
func (q *Queue) Dequeue() (interface{}, error) {
    if q.IsEmpty() {
        return nil, errors.New("queue is empty")
    }
    
    item := q.items[0]
    q.items = q.items[1:]
    return item, nil
}

// Проверка, пуста ли очередь
func (q *Queue) IsEmpty() bool {
    return len(q.items) == 0
}

// Просмотр первого элемента без извлечения
func (q *Queue) Peek() (interface{}, error) {
    if q.IsEmpty() {
        return nil, errors.New("queue is empty")
    }
    
    return q.items[0], nil
}

// Пример использования:
// queue := NewQueue()
// queue.Enqueue(1)
// queue.Enqueue(2)
// val, _ := queue.Dequeue() // val = 1
// isEmpty := queue.IsEmpty() // false
```

**Пояснение**:
- Используем слайс для хранения элементов очереди
- Enqueue добавляет элемент в конец слайса
- Dequeue удаляет и возвращает первый элемент
- IsEmpty проверяет, пуста ли очередь
- Временная сложность: Enqueue - O(1) амортизированно, Dequeue - O(n), IsEmpty - O(1)
- Для более эффективной реализации можно использовать кольцевой буфер или связный список

## Задачи на интерфейсы и ООП

### Задача 6: Реализация интерфейса Shape

**Задание**: Создайте интерфейс Shape с методами Area() и Perimeter(), и реализуйте его для типов Rectangle и Circle.

**Решение**:
```go
import "math"

// Интерфейс Shape
type Shape interface {
    Area() float64
    Perimeter() float64
}

// Структура Rectangle
type Rectangle struct {
    Width  float64
    Height float64
}

// Реализация методов интерфейса Shape для Rectangle
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}

// Структура Circle
type Circle struct {
    Radius float64
}

// Реализация методов интерфейса Shape для Circle
func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

func (c Circle) Perimeter() float64 {
    return 2 * math.Pi * c.Radius
}

// Функция для работы с любой фигурой
func PrintShapeInfo(s Shape) {
    fmt.Printf("Area: %.2f, Perimeter: %.2f\n", s.Area(), s.Perimeter())
}

// Пример использования:
// rect := Rectangle{Width: 5, Height: 3}
// circle := Circle{Radius: 2}
// PrintShapeInfo(rect)   // "Area: 15.00, Perimeter: 16.00"
// PrintShapeInfo(circle) // "Area: 12.57, Perimeter: 12.57"
```

**Пояснение**:
- Определяем интерфейс Shape с двумя методами
- Реализуем этот интерфейс для двух разных типов: Rectangle и Circle
- Создаем функцию, которая принимает любую фигуру через интерфейс
- Это демонстрирует полиморфизм в Go через интерфейсы

### Задача 7: Реализация кэша с помощью интерфейса

**Задание**: Создайте интерфейс Cache с методами Get, Set и Delete, и реализуйте его для хранения в памяти с поддержкой TTL (Time To Live).

**Решение**:
```go
import (
    "sync"
    "time"
)

// Интерфейс Cache
type Cache interface {
    Get(key string) (interface{}, bool)
    Set(key string, value interface{}, ttl time.Duration)
    Delete(key string)
}

// Структура для хранения значения с временем истечения
type cacheItem struct {
    value      interface{}
    expiration time.Time
}

// Реализация кэша в памяти
type MemoryCache struct {
    items map[string]cacheItem
    mu    sync.RWMutex
}

// Создание нового кэша
func NewMemoryCache() *MemoryCache {
    cache := &MemoryCache{
        items: make(map[string]cacheItem),
    }
    
    // Запускаем горутину для очистки устаревших элементов
    go cache.cleanupRoutine()
    
    return cache
}

// Получение значения из кэша
func (c *MemoryCache) Get(key string) (interface{}, bool) {
    c.mu.RLock()
    defer c.mu.RUnlock()
    
    item, exists := c.items[key]
    if !exists {
        return nil, false
    }
    
    // Проверяем, не истек ли срок действия
    if item.expiration.Before(time.Now()) {
        return nil, false
    }
    
    return item.value, true
}

// Установка значения в кэш
func (c *MemoryCache) Set(key string, value interface{}, ttl time.Duration) {
    c.mu.Lock()
    defer c.mu.Unlock()
    
    expiration := time.Now().Add(ttl)
    c.items[key] = cacheItem{
        value:      value,
        expiration: expiration,
    }
}

// Удаление значения из кэша
func (c *MemoryCache) Delete(key string) {
    c.mu.Lock()
    defer c.mu.Unlock()
    
    delete(c.items, key)
}

// Периодическая очистка устаревших элементов
func (c *MemoryCache) cleanupRoutine() {
    ticker := time.NewTicker(5 * time.Minute)
    defer ticker.Stop()
    
    for {
        <-ticker.C
        c.cleanup()
    }
}

// Очистка устаревших элементов
func (c *MemoryCache) cleanup() {
    c.mu.Lock()
    defer c.mu.Unlock()
    
    now := time.Now()
    for key, item := range c.items {
        if item.expiration.Before(now) {
            delete(c.items, key)
        }
    }
}

// Пример использования:
// cache := NewMemoryCache()
// cache.Set("key1", "value1", 1*time.Minute)
// value, exists := cache.Get("key1")
// cache.Delete("key1")
```

**Пояснение**:
- Определяем интерфейс Cache с тремя методами
- Реализуем кэш в памяти с поддержкой TTL
- Используем мьютекс для потокобезопасности
- Запускаем фоновую горутину для очистки устаревших элементов
- Демонстрируем использование интерфейсов, горутин и мьютексов

## Задачи на обработку ошибок

### Задача 8: Безопасная обработка паники

**Задание**: Напишите функцию, которая безопасно обрабатывает панику с помощью defer и recover.

**Решение**:
```go
func safeOperation(operation func() error) (err error) {
    defer func() {
        if r := recover(); r != nil {
            // Преобразуем панику в ошибку
            switch x := r.(type) {
            case string:
                err = errors.New(x)
            case error:
                err = x
            default:
                err = fmt.Errorf("unknown panic: %v", r)
            }
        }
    }()
    
    // Выполняем операцию
    err = operation()
    return
}

// Пример использования:
// err := safeOperation(func() error {
//     // Операция, которая может вызвать панику
//     a := []int{1, 2, 3}
//     fmt.Println(a[10]) // Вызовет панику
//     return nil
// })
// fmt.Println("Error:", err) // Выведет ошибку вместо паники
```

**Пояснение**:
- Используем defer и recover для перехвата паники
- Преобразуем панику в ошибку в зависимости от типа
- Возвращаем ошибку вместо паники
- Это позволяет безопасно обрабатывать неожиданные ситуации

### Задача 9: Пользовательские типы ошибок

**Задание**: Создайте пользовательский тип ошибки для обработки различных ситуаций в приложении.

**Решение**:
```go
// Типы ошибок
const (
    ErrNotFound      = "not_found"
    ErrUnauthorized  = "unauthorized"
    ErrInvalidInput  = "invalid_input"
)

// Пользовательский тип ошибки
type AppError struct {
    Type    string
    Message string
    Err     error
}

// Реализация интерфейса error
func (e AppError) Error() string {
    if e.Err != nil {
        return fmt.Sprintf("%s: %s (%v)", e.Type, e.Message, e.Err)
    }
    return fmt.Sprintf("%s: %s", e.Type, e.Message)
}

// Функция-обертка для создания ошибки "не найдено"
func NewNotFoundError(message string, err error) AppError {
    return AppError{
        Type:    ErrNotFound,
        Message: message,
        Err:     err,
    }
}

// Функция-обертка для создания ошибки "неавторизован"
func NewUnauthorizedError(message string, err error) AppError {
    return AppError{
        Type:    ErrUnauthorized,
        Message: message,
        Err:     err,
    }
}

// Функция-обертка для создания ошибки "неверный ввод"
func NewInvalidInputError(message string, err error) AppError {
    return AppError{
        Type:    ErrInvalidInput,
        Message: message,
        Err:     err,
    }
}

// Пример использования:
// func GetUser(id string) (User, error) {
//     if id == "" {
//         return User{}, NewInvalidInputError("user ID cannot be empty", nil)
//     }
//     
//     user, exists := userDB[id]
//     if !exists {
//         return User{}, NewNotFoundError("user not found", nil)
//     }
//     
//     return user, nil
// }
//
// func handleError(err error) {
//     if appErr, ok := err.(AppError); ok {
//         switch appErr.Type {
//         case ErrNotFound:
//             // Обработка ошибки "не найдено"
//         case ErrUnauthorized:
//             // Обработка ошибки "неавторизован"
//         case ErrInvalidInput:
//             // Обработка ошибки "неверный ввод"
//         }
//     } else {
//         // Обработка других ошибок
//     }
// }
```

**Пояснение**:
- Создаем пользовательский тип ошибки с дополнительной информацией
- Реализуем интерфейс error для нашего типа
- Создаем функции-обертки для создания типовых ошибок
- Демонстрируем обработку ошибок с помощью type assertion
- Это позволяет более гибко обрабатывать различные ситуации в приложении

## Задачи на многопоточность

### Задача 10: Параллельная обработка слайса

**Задание**: Напишите функцию, которая параллельно обрабатывает элементы слайса с ограничением на количество одновременно выполняемых горутин.

**Решение**:
```go
func parallelProcess(items []int, processFunc func(int) int, maxGoroutines int) []int {
    // Создаем канал для результатов
    resultChan := make(chan struct {
        index int
        value int
    }, len(items))
    
    // Создаем канал-семафор для ограничения количества горутин
    semaphore := make(chan struct{}, maxGoroutines)
    
    // Запускаем обработку в горутинах
    var wg sync.WaitGroup
    for i, item := range items {
        wg.Add(1)
        
        go func(index int, value int) {
            defer wg.Done()
            
            // Захватываем семафор
            semaphore <- struct{}{}
            defer func() { <-semaphore }()
            
            // Обрабатываем элемент
            result := processFunc(value)
            
            // Отправляем результат
            resultChan <- struct {
                index int
                value int
            }{index, result}
        }(i, item)
    }
    
    // Закрываем канал результатов после завершения всех горутин
    go func() {
        wg.Wait()
        close(resultChan)
    }()
    
    // Собираем результаты в правильном порядке
    results := make([]int, len(items))
    for res := range resultChan {
        results[res.index] = res.value
    }
    
    return results
}

// Пример использования:
// items := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
// results := parallelProcess(items, func(x int) int {
//     time.Sleep(100 * time.Millisecond) // Имитация тяжелой работы
//     return x * x
// }, 3)
// fmt.Println(results) // [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
```

**Пояснение**:
- Используем горутины для параллельной обработки элементов
- Ограничиваем количество одновременно выполняемых горутин с помощью семафора
- Используем канал для сбора результатов
- Используем WaitGroup для ожидания завершения всех горутин
- Сохраняем порядок элементов в результате

### Задача 11: Реализация воркер-пула

**Задание**: Реализуйте воркер-пул для параллельного выполнения задач.

**Решение**:
```go
// Задача для выполнения
type Task struct {
    ID  int
    Job func() interface{}
}

// Результат выполнения задачи
type Result struct {
    TaskID int
    Value  interface{}
}

// Воркер-пул
type WorkerPool struct {
    numWorkers int
    tasks      chan Task
    results    chan Result
    done       chan struct{}
}

// Создание нового воркер-пула
func NewWorkerPool(numWorkers int) *WorkerPool {
    return &WorkerPool{
        numWorkers: numWorkers,
        tasks:      make(chan Task),
        results:    make(chan Result),
        done:       make(chan struct{}),
    }
}

// Запуск воркер-пула
func (wp *WorkerPool) Start() {
    // Запускаем воркеры
    for i := 0; i < wp.numWorkers; i++ {
        go wp.worker(i)
    }
}

// Остановка воркер-пула
func (wp *WorkerPool) Stop() {
    close(wp.done)
}

// Функция воркера
func (wp *WorkerPool) worker(id int) {
    for {
        select {
        case task := <-wp.tasks:
            // Выполняем задачу
            result := Result{
                TaskID: task.ID,
                Value:  task.Job(),
            }
            
            // Отправляем результат
            wp.results <- result
            
        case <-wp.done:
            // Получен сигнал завершения
            return
        }
    }
}

// Добавление задачи в пул
func (wp *WorkerPool) AddTask(task Task) {
    wp.tasks <- task
}

// Получение результатов
func (wp *WorkerPool) Results() <-chan Result {
    return wp.results
}

// Пример использования:
// pool := NewWorkerPool(3)
// pool.Start()
// defer pool.Stop()
// 
// // Добавляем задачи
// for i := 0; i < 10; i++ {
//     id := i
//     pool.AddTask(Task{
//         ID: id,
//         Job: func() interface{} {
//             time.Sleep(100 * time.Millisecond) // Имитация работы
//             return id * id
//         },
//     })
// }
// 
// // Получаем результаты
// for i := 0; i < 10; i++ {
//     result := <-pool.Results()
//     fmt.Printf("Task %d result: %v\n", result.TaskID, result.Value)
// }
```

**Пояснение**:
- Создаем структуру для воркер-пула с каналами для задач и результатов
- Запускаем заданное количество воркеров в отдельных горутинах
- Каждый воркер берет задачи из канала и отправляет результаты в другой канал
- Используем select для обработки задач и сигнала завершения
- Это позволяет эффективно распределять нагрузку между несколькими горутинами

## Заключение

Эти практические задачи охватывают основные концепции Golang, которые могут быть полезны при подготовке к техническому интервью. Рекомендуется не только изучить решения, но и самостоятельно реализовать их, чтобы лучше понять принципы работы языка.

Для дальнейшей практики рекомендуется решать задачи на платформах LeetCode, HackerRank или CodeWars, фокусируясь на задачах, которые можно решить с использованием особенностей Go, таких как горутины, каналы и интерфейсы.
