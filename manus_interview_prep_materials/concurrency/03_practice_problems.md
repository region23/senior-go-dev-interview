# Практические задачи по многопоточности в Go

В этом документе представлены практические задачи по многопоточности и конкурентности в Go, которые помогут вам закрепить теоретические знания и подготовиться к техническому интервью. Задачи разделены по уровням сложности и включают решения с пояснениями.

## Базовые задачи

### Задача 1: Параллельное вычисление суммы элементов массива

**Задание**: Напишите функцию, которая параллельно вычисляет сумму элементов большого массива, разделяя его на части.

**Решение**:
```go
func parallelSum(numbers []int, numGoroutines int) int {
    // Проверка граничных случаев
    if len(numbers) == 0 {
        return 0
    }
    
    if numGoroutines <= 0 {
        numGoroutines = 1
    }
    
    // Ограничиваем количество горутин размером массива
    if numGoroutines > len(numbers) {
        numGoroutines = len(numbers)
    }
    
    // Размер части для каждой горутины
    chunkSize := len(numbers) / numGoroutines
    
    // Канал для сбора результатов
    resultChan := make(chan int, numGoroutines)
    
    // Запускаем горутины для обработки частей массива
    for i := 0; i < numGoroutines; i++ {
        start := i * chunkSize
        end := start + chunkSize
        
        // Последняя горутина обрабатывает остаток
        if i == numGoroutines-1 {
            end = len(numbers)
        }
        
        go func(start, end int) {
            sum := 0
            for j := start; j < end; j++ {
                sum += numbers[j]
            }
            resultChan <- sum
        }(start, end)
    }
    
    // Собираем результаты
    totalSum := 0
    for i := 0; i < numGoroutines; i++ {
        totalSum += <-resultChan
    }
    
    return totalSum
}

// Пример использования:
// numbers := make([]int, 1000000)
// for i := range numbers {
//     numbers[i] = i + 1
// }
// sum := parallelSum(numbers, 4)
// fmt.Println(sum) // 500000500000
```

**Пояснение**:
- Разделяем массив на равные части для параллельной обработки
- Каждая горутина вычисляет сумму своей части
- Используем канал для сбора результатов
- Складываем частичные суммы для получения общего результата
- Временная сложность: O(n), где n - размер массива
- Пространственная сложность: O(1) (не считая входной массив)

### Задача 2: Реализация конкурентного счетчика

**Задание**: Реализуйте потокобезопасный счетчик, который может быть безопасно использован из нескольких горутин.

**Решение 1**: С использованием мьютекса
```go
type Counter struct {
    mu    sync.Mutex
    value int
}

func NewCounter() *Counter {
    return &Counter{value: 0}
}

func (c *Counter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.value++
}

func (c *Counter) Decrement() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.value--
}

func (c *Counter) Value() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.value
}

// Пример использования:
// counter := NewCounter()
// var wg sync.WaitGroup
// 
// for i := 0; i < 1000; i++ {
//     wg.Add(1)
//     go func() {
//         defer wg.Done()
//         counter.Increment()
//     }()
// }
// 
// wg.Wait()
// fmt.Println(counter.Value()) // 1000
```

**Решение 2**: С использованием атомарных операций
```go
type AtomicCounter struct {
    value int64
}

func NewAtomicCounter() *AtomicCounter {
    return &AtomicCounter{value: 0}
}

func (c *AtomicCounter) Increment() {
    atomic.AddInt64(&c.value, 1)
}

func (c *AtomicCounter) Decrement() {
    atomic.AddInt64(&c.value, -1)
}

func (c *AtomicCounter) Value() int64 {
    return atomic.LoadInt64(&c.value)
}

// Пример использования:
// counter := NewAtomicCounter()
// var wg sync.WaitGroup
// 
// for i := 0; i < 1000; i++ {
//     wg.Add(1)
//     go func() {
//         defer wg.Done()
//         counter.Increment()
//     }()
// }
// 
// wg.Wait()
// fmt.Println(counter.Value()) // 1000
```

**Пояснение**:
- Первое решение использует мьютекс для синхронизации доступа к счетчику
- Второе решение использует атомарные операции, которые более эффективны для простых операций
- Оба решения обеспечивают потокобезопасность
- Атомарные операции обычно быстрее, но применимы только для простых операций

### Задача 3: Ограничение скорости запросов (Rate Limiting)

**Задание**: Реализуйте функцию, которая ограничивает скорость выполнения запросов до заданного количества в секунду.

**Решение**:
```go
type RateLimiter struct {
    ticker *time.Ticker
    stop   chan struct{}
}

func NewRateLimiter(requestsPerSecond int) *RateLimiter {
    interval := time.Second / time.Duration(requestsPerSecond)
    return &RateLimiter{
        ticker: time.NewTicker(interval),
        stop:   make(chan struct{}),
    }
}

func (r *RateLimiter) Wait() {
    <-r.ticker.C
}

func (r *RateLimiter) Stop() {
    r.ticker.Stop()
    close(r.stop)
}

func processRequests(urls []string, requestsPerSecond int) {
    limiter := NewRateLimiter(requestsPerSecond)
    defer limiter.Stop()
    
    for _, url := range urls {
        limiter.Wait() // Ждем разрешения от лимитера
        
        // Выполняем запрос (в отдельной горутине, чтобы не блокировать лимитер)
        go func(url string) {
            fmt.Printf("Processing request to %s at %v\n", url, time.Now())
            // Здесь был бы реальный HTTP-запрос
        }(url)
    }
}

// Пример использования:
// urls := []string{
//     "https://example.com/1",
//     "https://example.com/2",
//     "https://example.com/3",
//     "https://example.com/4",
//     "https://example.com/5",
// }
// processRequests(urls, 2) // 2 запроса в секунду
```

**Пояснение**:
- Используем ticker для равномерного распределения запросов во времени
- Каждый запрос ожидает сигнала от ticker перед выполнением
- Запросы выполняются в отдельных горутинах, чтобы не блокировать основной поток
- Этот подход обеспечивает равномерную нагрузку на сервер

## Задачи среднего уровня

### Задача 4: Реализация конкурентной карты (Concurrent Map)

**Задание**: Реализуйте потокобезопасную карту, которая может быть безопасно использована из нескольких горутин.

**Решение**:
```go
type ConcurrentMap struct {
    mu    sync.RWMutex
    items map[string]interface{}
}

func NewConcurrentMap() *ConcurrentMap {
    return &ConcurrentMap{
        items: make(map[string]interface{}),
    }
}

func (m *ConcurrentMap) Set(key string, value interface{}) {
    m.mu.Lock()
    defer m.mu.Unlock()
    m.items[key] = value
}

func (m *ConcurrentMap) Get(key string) (interface{}, bool) {
    m.mu.RLock()
    defer m.mu.RUnlock()
    value, exists := m.items[key]
    return value, exists
}

func (m *ConcurrentMap) Delete(key string) {
    m.mu.Lock()
    defer m.mu.Unlock()
    delete(m.items, key)
}

func (m *ConcurrentMap) Len() int {
    m.mu.RLock()
    defer m.mu.RUnlock()
    return len(m.items)
}

// Пример использования:
// cm := NewConcurrentMap()
// var wg sync.WaitGroup
// 
// // Запись
// for i := 0; i < 100; i++ {
//     wg.Add(1)
//     go func(i int) {
//         defer wg.Done()
//         key := fmt.Sprintf("key%d", i)
//         cm.Set(key, i)
//     }(i)
// }
// 
// // Чтение
// for i := 0; i < 100; i++ {
//     wg.Add(1)
//     go func(i int) {
//         defer wg.Done()
//         key := fmt.Sprintf("key%d", i)
//         value, _ := cm.Get(key)
//         fmt.Printf("Key: %s, Value: %v\n", key, value)
//     }(i)
// }
// 
// wg.Wait()
// fmt.Println("Map size:", cm.Len())
```

**Пояснение**:
- Используем RWMutex для оптимизации производительности (множество чтений, меньше записей)
- Блокировка на запись (Lock) используется для Set и Delete
- Блокировка на чтение (RLock) используется для Get и Len
- Это обеспечивает безопасный конкурентный доступ к карте
- Для более высокой производительности можно реализовать шардирование (разделение карты на несколько частей)

### Задача 5: Реализация паттерна "Издатель-Подписчик" (Pub-Sub)

**Задание**: Реализуйте простую систему "издатель-подписчик", где клиенты могут подписываться на темы и получать сообщения.

**Решение**:
```go
type PubSub struct {
    mu           sync.RWMutex
    subscribers  map[string]map[string]chan string
    nextClientID int
}

func NewPubSub() *PubSub {
    return &PubSub{
        subscribers:  make(map[string]map[string]chan string),
        nextClientID: 0,
    }
}

// Подписка на тему
func (ps *PubSub) Subscribe(topic string) (string, <-chan string) {
    ps.mu.Lock()
    defer ps.mu.Unlock()
    
    // Создаем карту подписчиков для темы, если она не существует
    if _, exists := ps.subscribers[topic]; !exists {
        ps.subscribers[topic] = make(map[string]chan string)
    }
    
    // Создаем канал для подписчика
    clientID := fmt.Sprintf("client-%d", ps.nextClientID)
    ps.nextClientID++
    ch := make(chan string, 10) // Буферизованный канал
    
    // Добавляем подписчика
    ps.subscribers[topic][clientID] = ch
    
    return clientID, ch
}

// Отписка от темы
func (ps *PubSub) Unsubscribe(topic, clientID string) {
    ps.mu.Lock()
    defer ps.mu.Unlock()
    
    // Проверяем существование темы и подписчика
    if _, exists := ps.subscribers[topic]; !exists {
        return
    }
    
    if ch, exists := ps.subscribers[topic][clientID]; exists {
        close(ch)
        delete(ps.subscribers[topic], clientID)
    }
    
    // Удаляем тему, если нет подписчиков
    if len(ps.subscribers[topic]) == 0 {
        delete(ps.subscribers, topic)
    }
}

// Публикация сообщения
func (ps *PubSub) Publish(topic, message string) {
    ps.mu.RLock()
    defer ps.mu.RUnlock()
    
    // Проверяем существование темы
    subscribers, exists := ps.subscribers[topic]
    if !exists {
        return
    }
    
    // Отправляем сообщение всем подписчикам
    for _, ch := range subscribers {
        select {
        case ch <- message:
            // Сообщение отправлено
        default:
            // Канал заполнен, пропускаем
        }
    }
}

// Пример использования:
// ps := NewPubSub()
// 
// // Подписчики
// clientID1, ch1 := ps.Subscribe("weather")
// clientID2, ch2 := ps.Subscribe("weather")
// clientID3, ch3 := ps.Subscribe("sports")
// 
// // Горутины для обработки сообщений
// go func() {
//     for msg := range ch1 {
//         fmt.Println("Client 1 received weather update:", msg)
//     }
// }()
// 
// go func() {
//     for msg := range ch2 {
//         fmt.Println("Client 2 received weather update:", msg)
//     }
// }()
// 
// go func() {
//     for msg := range ch3 {
//         fmt.Println("Client 3 received sports update:", msg)
//     }
// }()
// 
// // Публикация сообщений
// ps.Publish("weather", "Sunny, 25°C")
// ps.Publish("sports", "Team A won")
// 
// // Отписка
// ps.Unsubscribe("weather", clientID1)
// 
// // Еще одна публикация
// ps.Publish("weather", "Rainy, 15°C")
```

**Пояснение**:
- Используем карту карт для хранения подписчиков по темам
- RWMutex обеспечивает безопасный конкурентный доступ
- Буферизованные каналы предотвращают блокировку при публикации
- Неблокирующая отправка (select с default) предотвращает зависание при заполненном канале
- Закрытие каналов при отписке предотвращает утечки горутин

### Задача 6: Параллельный веб-краулер

**Задание**: Реализуйте простой веб-краулер, который параллельно обходит страницы, начиная с заданного URL, с ограничением глубины и количества одновременных запросов.

**Решение**:
```go
type Crawler struct {
    maxDepth      int
    maxGoroutines int
    visited       map[string]bool
    mu            sync.Mutex
    semaphore     chan struct{}
}

func NewCrawler(maxDepth, maxGoroutines int) *Crawler {
    return &Crawler{
        maxDepth:      maxDepth,
        maxGoroutines: maxGoroutines,
        visited:       make(map[string]bool),
        semaphore:     make(chan struct{}, maxGoroutines),
    }
}

func (c *Crawler) Crawl(url string) {
    var wg sync.WaitGroup
    c.crawlRecursive(url, 0, &wg)
    wg.Wait()
}

func (c *Crawler) crawlRecursive(url string, depth int, wg *sync.WaitGroup) {
    // Проверяем глубину
    if depth > c.maxDepth {
        return
    }
    
    // Проверяем, посещали ли мы уже этот URL
    c.mu.Lock()
    if c.visited[url] {
        c.mu.Unlock()
        return
    }
    c.visited[url] = true
    c.mu.Unlock()
    
    // Захватываем семафор (ограничиваем количество горутин)
    c.semaphore <- struct{}{}
    wg.Add(1)
    
    go func() {
        defer func() {
            <-c.semaphore // Освобождаем семафор
            wg.Done()
        }()
        
        // Получаем содержимое страницы
        fmt.Printf("Crawling: %s (depth: %d)\n", url, depth)
        links, err := fetchLinks(url)
        if err != nil {
            fmt.Printf("Error fetching %s: %v\n", url, err)
            return
        }
        
        // Рекурсивно обходим найденные ссылки
        for _, link := range links {
            c.crawlRecursive(link, depth+1, wg)
        }
    }()
}

// Функция для получения ссылок со страницы (упрощенная версия)
func fetchLinks(url string) ([]string, error) {
    // В реальном приложении здесь был бы HTTP-запрос и парсинг HTML
    // Для примера возвращаем фиктивные данные
    time.Sleep(100 * time.Millisecond) // Имитация сетевой задержки
    
    // Генерируем случайные ссылки
    links := make([]string, 0)
    if rand.Intn(10) > 7 { // 30% шанс ошибки
        return nil, fmt.Errorf("failed to fetch %s", url)
    }
    
    numLinks := rand.Intn(5) // 0-4 ссылки
    for i := 0; i < numLinks; i++ {
        links = append(links, fmt.Sprintf("%s/link%d", url, i))
    }
    
    return links, nil
}

// Пример использования:
// crawler := NewCrawler(2, 3) // Максимальная глубина 2, максимум 3 одновременных запроса
// crawler.Crawl("https://example.com")
```

**Пояснение**:
- Используем рекурсивный обход для посещения страниц
- Ограничиваем глубину обхода для предотвращения бесконечного обхода
- Используем карту для отслеживания посещенных URL
- Семафор ограничивает количество одновременных запросов
- WaitGroup обеспечивает ожидание завершения всех горутин
- Мьютекс защищает доступ к карте посещенных URL

## Продвинутые задачи

### Задача 7: Реализация пула воркеров с динамическим масштабированием

**Задание**: Реализуйте пул воркеров, который может динамически масштабироваться в зависимости от нагрузки.

**Решение**:
```go
type Task func() interface{}

type Result struct {
    Value interface{}
}

type DynamicWorkerPool struct {
    tasks           chan Task
    results         chan Result
    minWorkers      int
    maxWorkers      int
    activeWorkers   int32
    workerControl   chan struct{}
    shutdownSignal  chan struct{}
    wg              sync.WaitGroup
    mu              sync.Mutex
}

func NewDynamicWorkerPool(minWorkers, maxWorkers int) *DynamicWorkerPool {
    if minWorkers <= 0 {
        minWorkers = 1
    }
    if maxWorkers < minWorkers {
        maxWorkers = minWorkers
    }
    
    pool := &DynamicWorkerPool{
        tasks:          make(chan Task, 100),
        results:        make(chan Result, 100),
        minWorkers:     minWorkers,
        maxWorkers:     maxWorkers,
        workerControl:  make(chan struct{}),
        shutdownSignal: make(chan struct{}),
    }
    
    return pool
}

func (p *DynamicWorkerPool) Start() {
    // Запускаем минимальное количество воркеров
    for i := 0; i < p.minWorkers; i++ {
        p.startWorker()
    }
    
    // Запускаем мониторинг нагрузки
    go p.monitor()
}

func (p *DynamicWorkerPool) startWorker() {
    p.mu.Lock()
    defer p.mu.Unlock()
    
    if atomic.LoadInt32(&p.activeWorkers) >= int32(p.maxWorkers) {
        return
    }
    
    p.wg.Add(1)
    atomic.AddInt32(&p.activeWorkers, 1)
    
    go func() {
        defer p.wg.Done()
        defer atomic.AddInt32(&p.activeWorkers, -1)
        
        for {
            select {
            case task, ok := <-p.tasks:
                if !ok {
                    return
                }
                
                // Выполняем задачу
                result := task()
                
                // Отправляем результат
                select {
                case p.results <- Result{Value: result}:
                    // Результат отправлен
                case <-p.shutdownSignal:
                    return
                }
                
            case <-p.workerControl:
                // Сигнал для завершения воркера (при уменьшении пула)
                return
                
            case <-p.shutdownSignal:
                // Сигнал для завершения всех воркеров
                return
            }
        }
    }()
}

func (p *DynamicWorkerPool) monitor() {
    ticker := time.NewTicker(100 * time.Millisecond)
    defer ticker.Stop()
    
    for {
        select {
        case <-ticker.C:
            // Оценка нагрузки (упрощенно: по длине очереди задач)
            queueSize := len(p.tasks)
            activeWorkers := atomic.LoadInt32(&p.activeWorkers)
            
            if queueSize > int(activeWorkers) && activeWorkers < int32(p.maxWorkers) {
                // Увеличиваем количество воркеров
                p.startWorker()
                fmt.Printf("Scaling up: %d active workers\n", atomic.LoadInt32(&p.activeWorkers))
            } else if queueSize == 0 && activeWorkers > int32(p.minWorkers) {
                // Уменьшаем количество воркеров
                select {
                case p.workerControl <- struct{}{}:
                    fmt.Printf("Scaling down: %d active workers\n", atomic.LoadInt32(&p.activeWorkers)-1)
                default:
                    // Не блокируемся, если канал заполнен
                }
            }
            
        case <-p.shutdownSignal:
            return
        }
    }
}

func (p *DynamicWorkerPool) Submit(task Task) {
    select {
    case p.tasks <- task:
        // Задача добавлена в очередь
    case <-p.shutdownSignal:
        // Пул завершает работу
    }
}

func (p *DynamicWorkerPool) Results() <-chan Result {
    return p.results
}

func (p *DynamicWorkerPool) Shutdown() {
    close(p.shutdownSignal)
    p.wg.Wait()
    close(p.results)
}

// Пример использования:
// pool := NewDynamicWorkerPool(2, 10) // Минимум 2, максимум 10 воркеров
// pool.Start()
// 
// // Отправляем задачи
// for i := 0; i < 20; i++ {
//     taskID := i
//     pool.Submit(func() interface{} {
//         // Имитация работы разной длительности
//         duration := time.Duration(rand.Intn(500)) * time.Millisecond
//         time.Sleep(duration)
//         return fmt.Sprintf("Task %d completed in %v", taskID, duration)
//     })
// }
// 
// // Получаем результаты
// go func() {
//     for result := range pool.Results() {
//         fmt.Println(result.Value)
//     }
// }()
// 
// // Даем время на выполнение задач
// time.Sleep(5 * time.Second)
// 
// // Завершаем работу пула
// pool.Shutdown()
```

**Пояснение**:
- Пул поддерживает минимальное и максимальное количество воркеров
- Мониторинг нагрузки периодически проверяет длину очереди задач
- Если очередь растет, добавляются новые воркеры (до максимума)
- Если очередь пуста, лишние воркеры завершаются (до минимума)
- Атомарные операции обеспечивают безопасный подсчет активных воркеров
- Каналы используются для управления жизненным циклом воркеров
- Этот подход обеспечивает эффективное использование ресурсов

### Задача 8: Реализация распределенного кэша с временем жизни (TTL)

**Задание**: Реализуйте распределенный кэш с поддержкой TTL (Time To Live) для элементов.

**Решение**:
```go
type CacheItem struct {
    Value      interface{}
    Expiration int64
}

type ShardedCache struct {
    shards     []*cacheShard
    shardCount int
    hash       func(string) uint32
}

type cacheShard struct {
    items map[string]CacheItem
    mu    sync.RWMutex
}

func NewShardedCache(shardCount int) *ShardedCache {
    if shardCount <= 0 {
        shardCount = 16 // Значение по умолчанию
    }
    
    cache := &ShardedCache{
        shards:     make([]*cacheShard, shardCount),
        shardCount: shardCount,
        hash:       fnv32Hash,
    }
    
    for i := 0; i < shardCount; i++ {
        cache.shards[i] = &cacheShard{
            items: make(map[string]CacheItem),
        }
    }
    
    // Запускаем очистку устаревших элементов
    go cache.cleanupRoutine()
    
    return cache
}

// Хеш-функция FNV-1a
func fnv32Hash(key string) uint32 {
    hash := uint32(2166136261)
    const prime32 = uint32(16777619)
    for i := 0; i < len(key); i++ {
        hash ^= uint32(key[i])
        hash *= prime32
    }
    return hash
}

// Получение шарда для ключа
func (c *ShardedCache) getShard(key string) *cacheShard {
    return c.shards[c.hash(key)%uint32(c.shardCount)]
}

// Установка значения с TTL
func (c *ShardedCache) Set(key string, value interface{}, ttl time.Duration) {
    shard := c.getShard(key)
    shard.mu.Lock()
    defer shard.mu.Unlock()
    
    var expiration int64
    if ttl > 0 {
        expiration = time.Now().Add(ttl).UnixNano()
    }
    
    shard.items[key] = CacheItem{
        Value:      value,
        Expiration: expiration,
    }
}

// Получение значения
func (c *ShardedCache) Get(key string) (interface{}, bool) {
    shard := c.getShard(key)
    shard.mu.RLock()
    defer shard.mu.RUnlock()
    
    item, found := shard.items[key]
    if !found {
        return nil, false
    }
    
    // Проверяем срок действия
    if item.Expiration > 0 && time.Now().UnixNano() > item.Expiration {
        return nil, false
    }
    
    return item.Value, true
}

// Удаление значения
func (c *ShardedCache) Delete(key string) {
    shard := c.getShard(key)
    shard.mu.Lock()
    defer shard.mu.Unlock()
    
    delete(shard.items, key)
}

// Очистка устаревших элементов
func (c *ShardedCache) cleanupRoutine() {
    ticker := time.NewTicker(5 * time.Minute)
    defer ticker.Stop()
    
    for range ticker.C {
        now := time.Now().UnixNano()
        
        for _, shard := range c.shards {
            shard.mu.Lock()
            
            for key, item := range shard.items {
                if item.Expiration > 0 && now > item.Expiration {
                    delete(shard.items, key)
                }
            }
            
            shard.mu.Unlock()
        }
    }
}

// Пример использования:
// cache := NewShardedCache(8) // 8 шардов
// 
// // Установка значений с разным TTL
// cache.Set("key1", "value1", 1*time.Second)
// cache.Set("key2", "value2", 1*time.Minute)
// cache.Set("key3", "value3", 0) // Без TTL
// 
// // Получение значений
// if value, found := cache.Get("key1"); found {
//     fmt.Println("Key1:", value)
// }
// 
// // Ждем истечения TTL
// time.Sleep(2 * time.Second)
// 
// // Проверяем истекшее значение
// if _, found := cache.Get("key1"); !found {
//     fmt.Println("Key1 expired")
// }
// 
// // Проверяем неистекшее значение
// if value, found := cache.Get("key2"); found {
//     fmt.Println("Key2:", value)
// }
```

**Пояснение**:
- Используем шардирование для повышения производительности при конкурентном доступе
- Каждый шард имеет свой мьютекс, что уменьшает конкуренцию за блокировки
- Хеш-функция распределяет ключи по шардам
- TTL реализовано через хранение времени истечения для каждого элемента
- Фоновая горутина периодически очищает устаревшие элементы
- Этот подход обеспечивает высокую производительность и масштабируемость

### Задача 9: Реализация конкурентного пайплайна с отменой

**Задание**: Реализуйте конкурентный пайплайн для обработки данных с возможностью отмены всех операций.

**Решение**:
```go
// Этап пайплайна
type PipelineStage func(ctx context.Context, in <-chan interface{}) <-chan interface{}

// Функция для создания пайплайна из нескольких этапов
func Pipeline(ctx context.Context, source <-chan interface{}, stages ...PipelineStage) <-chan interface{} {
    var current <-chan interface{} = source
    
    // Последовательно применяем каждый этап
    for _, stage := range stages {
        current = stage(ctx, current)
    }
    
    return current
}

// Пример этапа: фильтрация
func Filter(predicate func(interface{}) bool) PipelineStage {
    return func(ctx context.Context, in <-chan interface{}) <-chan interface{} {
        out := make(chan interface{})
        
        go func() {
            defer close(out)
            
            for item := range in {
                select {
                case <-ctx.Done():
                    return // Отмена операции
                default:
                    if predicate(item) {
                        select {
                        case out <- item:
                            // Элемент отправлен
                        case <-ctx.Done():
                            return // Отмена операции
                        }
                    }
                }
            }
        }()
        
        return out
    }
}

// Пример этапа: преобразование
func Map(transform func(interface{}) interface{}) PipelineStage {
    return func(ctx context.Context, in <-chan interface{}) <-chan interface{} {
        out := make(chan interface{})
        
        go func() {
            defer close(out)
            
            for item := range in {
                select {
                case <-ctx.Done():
                    return // Отмена операции
                default:
                    result := transform(item)
                    select {
                    case out <- result:
                        // Результат отправлен
                    case <-ctx.Done():
                        return // Отмена операции
                    }
                }
            }
        }()
        
        return out
    }
}

// Пример этапа: параллельная обработка
func Parallel(worker func(interface{}) interface{}, numWorkers int) PipelineStage {
    return func(ctx context.Context, in <-chan interface{}) <-chan interface{} {
        out := make(chan interface{})
        
        // Создаем WaitGroup для ожидания завершения всех воркеров
        var wg sync.WaitGroup
        
        // Функция для воркера
        workerFunc := func() {
            defer wg.Done()
            
            for item := range in {
                select {
                case <-ctx.Done():
                    return // Отмена операции
                default:
                    result := worker(item)
                    select {
                    case out <- result:
                        // Результат отправлен
                    case <-ctx.Done():
                        return // Отмена операции
                    }
                }
            }
        }
        
        // Запускаем воркеры
        wg.Add(numWorkers)
        for i := 0; i < numWorkers; i++ {
            go workerFunc()
        }
        
        // Закрываем выходной канал после завершения всех воркеров
        go func() {
            wg.Wait()
            close(out)
        }()
        
        return out
    }
}

// Пример использования:
// // Создаем контекст с возможностью отмены
// ctx, cancel := context.WithCancel(context.Background())
// defer cancel()
// 
// // Создаем источник данных
// source := make(chan interface{})
// go func() {
//     defer close(source)
//     for i := 1; i <= 100; i++ {
//         select {
//         case source <- i:
//             // Элемент отправлен
//         case <-ctx.Done():
//             return // Отмена операции
//         }
//     }
// }()
// 
// // Создаем пайплайн
// result := Pipeline(
//     ctx,
//     source,
//     Filter(func(item interface{}) bool {
//         // Фильтруем только четные числа
//         return item.(int)%2 == 0
//     }),
//     Map(func(item interface{}) interface{} {
//         // Возводим в квадрат
//         return item.(int) * item.(int)
//     }),
//     Parallel(func(item interface{}) interface{} {
//         // Имитация тяжелой работы
//         time.Sleep(100 * time.Millisecond)
//         return fmt.Sprintf("Result: %d", item.(int))
//     }, 5), // 5 параллельных воркеров
// )
// 
// // Получаем результаты
// count := 0
// for item := range result {
//     fmt.Println(item)
//     count++
//     
//     // Отменяем пайплайн после получения 10 результатов
//     if count >= 10 {
//         cancel()
//         break
//     }
// }
// 
// fmt.Println("Pipeline cancelled after", count, "results")
```

**Пояснение**:
- Пайплайн состоит из последовательных этапов обработки данных
- Каждый этап принимает входной канал и возвращает выходной канал
- Контекст используется для отмены всех операций
- Этап Parallel распараллеливает обработку между несколькими воркерами
- Select с ctx.Done() обеспечивает быструю отмену на всех этапах
- Этот подход обеспечивает эффективную обработку данных с возможностью отмены

## Заключение

В этом документе мы рассмотрели практические задачи по многопоточности и конкурентности в Go разного уровня сложности. Эти задачи охватывают основные паттерны конкурентного программирования и помогут вам подготовиться к техническому интервью.

Ключевые моменты:
1. Горутины и каналы - основа конкурентности в Go
2. Синхронизация доступа к общим ресурсам критически важна
3. Паттерны конкурентного программирования помогают решать типовые задачи
4. Контекст обеспечивает управление жизненным циклом горутин
5. Шардирование повышает производительность при конкурентном доступе

Для дальнейшей практики рекомендуется решать задачи на платформах LeetCode или HackerRank, фокусируясь на задачах, связанных с конкурентностью и многопоточностью.
