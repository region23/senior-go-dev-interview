# Продвинутые паттерны конкурентного программирования в Go

## Введение

В этом руководстве мы рассмотрим продвинутые паттерны конкурентного программирования в Go, которые помогут вам писать эффективный, безопасный и масштабируемый код. Эти паттерны особенно важны для разработки высоконагруженных систем и могут быть полезны на техническом интервью.

## Паттерн "Конвейер" (Pipeline)

Конвейер - это последовательность этапов обработки данных, где выход одного этапа является входом для следующего.

### Базовая реализация конвейера

```go
func generator(nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for _, n := range nums {
            out <- n
        }
    }()
    return out
}

func square(in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            out <- n * n
        }
    }()
    return out
}

func filter(in <-chan int, predicate func(int) bool) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            if predicate(n) {
                out <- n
            }
        }
    }()
    return out
}

// Использование
nums := generator(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
squares := square(nums)
even := filter(squares, func(n int) bool {
    return n%2 == 0
})

for n := range even {
    fmt.Println(n) // 4, 16, 36, 64, 100
}
```

### Конвейер с отменой

```go
func generatorWithCancel(ctx context.Context, nums ...int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for _, n := range nums {
            select {
            case out <- n:
            case <-ctx.Done():
                return
            }
        }
    }()
    return out
}

func squareWithCancel(ctx context.Context, in <-chan int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            select {
            case out <- n * n:
            case <-ctx.Done():
                return
            }
        }
    }()
    return out
}

// Использование
ctx, cancel := context.WithCancel(context.Background())
defer cancel()

nums := generatorWithCancel(ctx, 1, 2, 3, 4, 5)
squares := squareWithCancel(ctx, nums)

// Отмена конвейера после получения первых двух значений
count := 0
for n := range squares {
    fmt.Println(n)
    count++
    if count >= 2 {
        cancel() // Отменяем все этапы конвейера
        break
    }
}
```

## Паттерн "Вентилятор" (Fan-Out, Fan-In)

Этот паттерн позволяет распределить работу между несколькими горутинами (fan-out) и затем собрать результаты (fan-in).

### Fan-Out: распределение работы

```go
func fanOut(in <-chan int, numWorkers int, processor func(int) int) []<-chan int {
    outputs := make([]<-chan int, numWorkers)
    for i := 0; i < numWorkers; i++ {
        outputs[i] = worker(in, processor)
    }
    return outputs
}

func worker(in <-chan int, processor func(int) int) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for n := range in {
            out <- processor(n)
        }
    }()
    return out
}
```

### Fan-In: сбор результатов

```go
func fanIn(channels ...<-chan int) <-chan int {
    var wg sync.WaitGroup
    multiplexed := make(chan int)
    
    // Функция для перенаправления значений из входного канала в выходной
    multiplex := func(c <-chan int) {
        defer wg.Done()
        for i := range c {
            multiplexed <- i
        }
    }
    
    // Запускаем горутину для каждого входного канала
    wg.Add(len(channels))
    for _, c := range channels {
        go multiplex(c)
    }
    
    // Закрываем выходной канал после завершения всех горутин
    go func() {
        wg.Wait()
        close(multiplexed)
    }()
    
    return multiplexed
}
```

### Полный пример Fan-Out, Fan-In

```go
func main() {
    // Создаем входной канал
    input := make(chan int, 100)
    go func() {
        defer close(input)
        for i := 0; i < 100; i++ {
            input <- i
        }
    }()
    
    // Функция обработки (имитация тяжелой работы)
    processor := func(n int) int {
        time.Sleep(50 * time.Millisecond)
        return n * n
    }
    
    // Fan-Out: распределяем работу между 10 горутинами
    workers := fanOut(input, 10, processor)
    
    // Fan-In: собираем результаты
    results := fanIn(workers...)
    
    // Используем результаты
    for result := range results {
        fmt.Println(result)
    }
}
```

## Паттерн "Семафор" (Semaphore)

Семафор ограничивает количество одновременно выполняемых операций.

### Реализация с использованием буферизованного канала

```go
type Semaphore chan struct{}

// Создание семафора с заданным лимитом
func NewSemaphore(limit int) Semaphore {
    return make(Semaphore, limit)
}

// Захват семафора
func (s Semaphore) Acquire() {
    s <- struct{}{}
}

// Освобождение семафора
func (s Semaphore) Release() {
    <-s
}

// Использование
func main() {
    // Создаем семафор с лимитом 3
    sem := NewSemaphore(3)
    
    // Запускаем 10 горутин
    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            // Захватываем семафор
            sem.Acquire()
            defer sem.Release()
            
            // Выполняем работу (не более 3 горутин одновременно)
            fmt.Printf("Worker %d starting\n", id)
            time.Sleep(1 * time.Second)
            fmt.Printf("Worker %d done\n", id)
        }(i)
    }
    
    wg.Wait()
}
```

### Семафор с тайм-аутом

```go
func (s Semaphore) AcquireWithTimeout(timeout time.Duration) bool {
    select {
    case s <- struct{}{}:
        return true
    case <-time.After(timeout):
        return false
    }
}

// Использование
if sem.AcquireWithTimeout(500 * time.Millisecond) {
    defer sem.Release()
    // Выполняем работу
} else {
    // Обрабатываем тайм-аут
    fmt.Println("Timed out waiting for semaphore")
}
```

## Паттерн "Воркер-пул" (Worker Pool)

Воркер-пул - это группа горутин, которые обрабатывают задачи из общей очереди.

### Базовая реализация воркер-пула

```go
type Job struct {
    ID  int
    Data interface{}
}

type Result struct {
    JobID int
    Value interface{}
}

func workerPool(numWorkers int, jobs <-chan Job, results chan<- Result, processor func(Job) Result) {
    var wg sync.WaitGroup
    
    // Запускаем фиксированное количество воркеров
    for i := 0; i < numWorkers; i++ {
        wg.Add(1)
        go func(workerID int) {
            defer wg.Done()
            
            for job := range jobs {
                // Обработка задачи
                result := processor(job)
                results <- result
                fmt.Printf("Worker %d processed job %d\n", workerID, job.ID)
            }
        }(i)
    }
    
    // Ждем завершения всех воркеров и закрываем канал результатов
    go func() {
        wg.Wait()
        close(results)
    }()
}

// Использование
func main() {
    jobs := make(chan Job, 100)
    results := make(chan Result, 100)
    
    // Функция обработки
    processor := func(job Job) Result {
        // Имитация работы
        time.Sleep(100 * time.Millisecond)
        return Result{
            JobID: job.ID,
            Value: job.ID * 2, // Пример обработки
        }
    }
    
    // Запускаем пул из 5 воркеров
    workerPool(5, jobs, results, processor)
    
    // Отправляем задачи
    go func() {
        for i := 0; i < 20; i++ {
            jobs <- Job{ID: i, Data: fmt.Sprintf("Task %d", i)}
        }
        close(jobs)
    }()
    
    // Получаем результаты
    for result := range results {
        fmt.Printf("Result: Job %d = %v\n", result.JobID, result.Value)
    }
}
```

### Воркер-пул с динамическим масштабированием

```go
type DynamicWorkerPool struct {
    jobs           chan Job
    results        chan Result
    processor      func(Job) Result
    maxWorkers     int
    activeWorkers  int32
    workerControl  chan struct{}
    wg             sync.WaitGroup
    mu             sync.Mutex
}

func NewDynamicWorkerPool(maxWorkers int, processor func(Job) Result) *DynamicWorkerPool {
    return &DynamicWorkerPool{
        jobs:          make(chan Job, 100),
        results:       make(chan Result, 100),
        processor:     processor,
        maxWorkers:    maxWorkers,
        workerControl: make(chan struct{}),
    }
}

func (p *DynamicWorkerPool) Start() {
    // Запускаем минимальное количество воркеров
    for i := 0; i < p.maxWorkers/4; i++ {
        p.startWorker()
    }
    
    // Мониторинг нагрузки и масштабирование
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
            case job, ok := <-p.jobs:
                if !ok {
                    return
                }
                
                // Обработка задачи
                result := p.processor(job)
                p.results <- result
                
            case <-p.workerControl:
                // Сигнал для завершения воркера
                return
            }
        }
    }()
}

func (p *DynamicWorkerPool) monitor() {
    ticker := time.NewTicker(100 * time.Millisecond)
    defer ticker.Stop()
    
    for range ticker.C {
        // Оценка нагрузки (упрощенно: по длине очереди задач)
        queueSize := len(p.jobs)
        activeWorkers := atomic.LoadInt32(&p.activeWorkers)
        
        if queueSize > int(activeWorkers) && activeWorkers < int32(p.maxWorkers) {
            // Увеличиваем количество воркеров
            p.startWorker()
        } else if queueSize == 0 && activeWorkers > int32(p.maxWorkers/4) {
            // Уменьшаем количество воркеров
            select {
            case p.workerControl <- struct{}{}:
                // Сигнал для завершения одного воркера
            default:
                // Не блокируемся, если канал заполнен
            }
        }
    }
}

func (p *DynamicWorkerPool) Submit(job Job) {
    p.jobs <- job
}

func (p *DynamicWorkerPool) Results() <-chan Result {
    return p.results
}

func (p *DynamicWorkerPool) Stop() {
    close(p.jobs)
    close(p.workerControl)
    p.wg.Wait()
    close(p.results)
}
```

## Паттерн "Издатель-Подписчик" (Pub-Sub)

Паттерн "Издатель-Подписчик" позволяет отправлять сообщения множеству получателей без прямой связи между ними.

### Базовая реализация Pub-Sub

```go
type PubSub struct {
    mu          sync.RWMutex
    subscribers map[string][]chan interface{}
}

func NewPubSub() *PubSub {
    return &PubSub{
        subscribers: make(map[string][]chan interface{}),
    }
}

// Подписка на тему
func (ps *PubSub) Subscribe(topic string) <-chan interface{} {
    ps.mu.Lock()
    defer ps.mu.Unlock()
    
    ch := make(chan interface{}, 1)
    ps.subscribers[topic] = append(ps.subscribers[topic], ch)
    return ch
}

// Отписка от темы
func (ps *PubSub) Unsubscribe(topic string, ch <-chan interface{}) {
    ps.mu.Lock()
    defer ps.mu.Unlock()
    
    subscribers := ps.subscribers[topic]
    for i, subscriber := range subscribers {
        if subscriber == ch {
            // Удаляем подписчика
            ps.subscribers[topic] = append(subscribers[:i], subscribers[i+1:]...)
            close(subscriber)
            break
        }
    }
}

// Публикация сообщения
func (ps *PubSub) Publish(topic string, message interface{}) {
    ps.mu.RLock()
    defer ps.mu.RUnlock()
    
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

// Закрытие всех подписок
func (ps *PubSub) Close() {
    ps.mu.Lock()
    defer ps.mu.Unlock()
    
    for _, subscribers := range ps.subscribers {
        for _, ch := range subscribers {
            close(ch)
        }
    }
    
    ps.subscribers = make(map[string][]chan interface{})
}

// Использование
func main() {
    ps := NewPubSub()
    
    // Подписчики
    sub1 := ps.Subscribe("weather")
    sub2 := ps.Subscribe("weather")
    sub3 := ps.Subscribe("sports")
    
    // Горутины для обработки сообщений
    go func() {
        for msg := range sub1 {
            fmt.Println("Subscriber 1 received:", msg)
        }
    }()
    
    go func() {
        for msg := range sub2 {
            fmt.Println("Subscriber 2 received:", msg)
        }
    }()
    
    go func() {
        for msg := range sub3 {
            fmt.Println("Subscriber 3 received:", msg)
        }
    }()
    
    // Публикация сообщений
    ps.Publish("weather", "Sunny, 25°C")
    ps.Publish("sports", "Team A won")
    ps.Publish("weather", "Cloudy, 20°C")
    
    // Отписка
    ps.Unsubscribe("weather", sub1)
    
    // Еще одна публикация
    ps.Publish("weather", "Rainy, 15°C")
    
    // Закрытие всех подписок
    time.Sleep(1 * time.Second)
    ps.Close()
}
```

## Паттерн "Барьер" (Barrier)

Барьер позволяет синхронизировать группу горутин в определенной точке выполнения.

### Реализация с использованием WaitGroup

```go
type Barrier struct {
    wg sync.WaitGroup
}

func NewBarrier(count int) *Barrier {
    b := &Barrier{}
    b.wg.Add(count)
    return b
}

func (b *Barrier) Wait() {
    b.wg.Done()
    b.wg.Wait()
}

// Использование
func main() {
    // Создаем барьер для 5 горутин
    barrier := NewBarrier(5)
    
    for i := 0; i < 5; i++ {
        go func(id int) {
            // Первая фаза работы
            fmt.Printf("Worker %d: Phase 1 complete\n", id)
            
            // Ждем, пока все горутины завершат первую фазу
            barrier.Wait()
            
            // Вторая фаза работы (начинается одновременно для всех горутин)
            fmt.Printf("Worker %d: Phase 2 complete\n", id)
        }(i)
    }
    
    time.Sleep(2 * time.Second)
}
```

### Циклический барьер

```go
type CyclicBarrier struct {
    generation int
    count      int
    parties    int
    trigger    func()
    mu         sync.Mutex
    cond       *sync.Cond
}

func NewCyclicBarrier(parties int, trigger func()) *CyclicBarrier {
    cb := &CyclicBarrier{
        parties: parties,
        count:   parties,
        trigger: trigger,
    }
    cb.cond = sync.NewCond(&cb.mu)
    return cb
}

func (cb *CyclicBarrier) Await() {
    cb.mu.Lock()
    defer cb.mu.Unlock()
    
    generation := cb.generation
    
    // Уменьшаем счетчик
    cb.count--
    
    if cb.count == 0 {
        // Последняя горутина достигла барьера
        if cb.trigger != nil {
            cb.trigger()
        }
        
        // Сбрасываем барьер для следующего цикла
        cb.count = cb.parties
        cb.generation++
        
        // Оповещаем все ожидающие горутины
        cb.cond.Broadcast()
    } else {
        // Ждем, пока все горутины достигнут барьера
        for generation == cb.generation {
            cb.cond.Wait()
        }
    }
}

// Использование
func main() {
    // Функция, выполняемая при достижении барьера
    trigger := func() {
        fmt.Println("All workers reached the barrier!")
    }
    
    // Создаем циклический барьер для 3 горутин
    barrier := NewCyclicBarrier(3, trigger)
    
    for i := 0; i < 3; i++ {
        go func(id int) {
            for j := 0; j < 3; j++ {
                // Имитация работы
                time.Sleep(time.Duration(rand.Intn(1000)) * time.Millisecond)
                fmt.Printf("Worker %d completed phase %d\n", id, j+1)
                
                // Ждем на барьере
                barrier.Await()
                
                fmt.Printf("Worker %d starting next phase\n", id)
            }
        }(i)
    }
    
    time.Sleep(5 * time.Second)
}
```

## Паттерн "Будущее" (Future/Promise)

Паттерн "Будущее" представляет результат асинхронной операции, который будет доступен в будущем.

### Реализация Future в Go

```go
type Future struct {
    result interface{}
    err    error
    done   chan struct{}
}

func NewFuture(fn func() (interface{}, error)) *Future {
    f := &Future{
        done: make(chan struct{}),
    }
    
    go func() {
        defer close(f.done)
        f.result, f.err = fn()
    }()
    
    return f
}

// Ожидание результата
func (f *Future) Get() (interface{}, error) {
    <-f.done
    return f.result, f.err
}

// Ожидание результата с тайм-аутом
func (f *Future) GetWithTimeout(timeout time.Duration) (interface{}, error, bool) {
    select {
    case <-f.done:
        return f.result, f.err, true
    case <-time.After(timeout):
        return nil, nil, false
    }
}

// Проверка готовности результата
func (f *Future) IsDone() bool {
    select {
    case <-f.done:
        return true
    default:
        return false
    }
}

// Использование
func main() {
    // Создаем Future для длительной операции
    future := NewFuture(func() (interface{}, error) {
        // Имитация длительной работы
        time.Sleep(2 * time.Second)
        return "Operation completed", nil
    })
    
    // Проверяем готовность (неблокирующий вызов)
    fmt.Println("Is done:", future.IsDone())
    
    // Ожидаем результат с тайм-аутом
    result, err, ok := future.GetWithTimeout(1 * time.Second)
    if !ok {
        fmt.Println("Operation timed out")
    } else {
        fmt.Println("Result:", result, "Error:", err)
    }
    
    // Ожидаем результат (блокирующий вызов)
    result, err = future.Get()
    fmt.Println("Result:", result, "Error:", err)
}
```

### Композиция Future

```go
// Выполнение нескольких Future параллельно
func WhenAll(futures ...*Future) *Future {
    return NewFuture(func() (interface{}, error) {
        results := make([]interface{}, len(futures))
        
        for i, future := range futures {
            result, err := future.Get()
            if err != nil {
                return nil, err
            }
            results[i] = result
        }
        
        return results, nil
    })
}

// Выполнение Future последовательно с передачей результата
func Chain(first *Future, next func(interface{}) *Future) *Future {
    return NewFuture(func() (interface{}, error) {
        result, err := first.Get()
        if err != nil {
            return nil, err
        }
        
        nextFuture := next(result)
        return nextFuture.Get()
    })
}

// Использование
func main() {
    // Создаем несколько Future
    f1 := NewFuture(func() (interface{}, error) {
        time.Sleep(1 * time.Second)
        return 10, nil
    })
    
    f2 := NewFuture(func() (interface{}, error) {
        time.Sleep(2 * time.Second)
        return 20, nil
    })
    
    // Ожидаем выполнения всех Future
    allFuture := WhenAll(f1, f2)
    results, err := allFuture.Get()
    fmt.Println("All results:", results, "Error:", err)
    
    // Цепочка Future
    chainedFuture := Chain(f1, func(result interface{}) *Future {
        value := result.(int)
        return NewFuture(func() (interface{}, error) {
            time.Sleep(500 * time.Millisecond)
            return value * 2, nil
        })
    })
    
    result, err := chainedFuture.Get()
    fmt.Println("Chained result:", result, "Error:", err)
}
```

## Паттерн "Контекст" (Context)

Контекст в Go используется для передачи сигналов отмены, дедлайнов и значений между горутинами.

### Использование контекста для отмены

```go
func worker(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            // Получен сигнал отмены
            fmt.Println("Worker: Received cancellation signal")
            return
        default:
            // Продолжаем работу
            fmt.Println("Worker: Working...")
            time.Sleep(500 * time.Millisecond)
        }
    }
}

func main() {
    // Создаем контекст с возможностью отмены
    ctx, cancel := context.WithCancel(context.Background())
    
    // Запускаем воркер
    go worker(ctx)
    
    // Работаем 2 секунды, затем отменяем
    time.Sleep(2 * time.Second)
    cancel()
    
    // Даем время воркеру завершиться
    time.Sleep(1 * time.Second)
    fmt.Println("Main: Done")
}
```

### Контекст с тайм-аутом

```go
func workerWithTimeout(ctx context.Context) {
    for {
        select {
        case <-ctx.Done():
            // Проверяем причину завершения
            if deadline, ok := ctx.Deadline(); ok {
                fmt.Printf("Worker: Deadline reached at %v\n", deadline)
            } else {
                fmt.Println("Worker: Cancelled")
            }
            return
        default:
            // Продолжаем работу
            fmt.Println("Worker: Working...")
            time.Sleep(500 * time.Millisecond)
        }
    }
}

func main() {
    // Создаем контекст с тайм-аутом
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel()
    
    // Запускаем воркер
    go workerWithTimeout(ctx)
    
    // Ждем завершения
    <-ctx.Done()
    fmt.Println("Main: Context done with error:", ctx.Err())
}
```

### Передача значений через контекст

```go
type key string

const (
    userIDKey key = "userID"
    authTokenKey key = "authToken"
)

func processRequest(ctx context.Context) {
    // Извлекаем значения из контекста
    userID, ok := ctx.Value(userIDKey).(string)
    if !ok {
        fmt.Println("User ID not found in context")
        return
    }
    
    authToken, ok := ctx.Value(authTokenKey).(string)
    if !ok {
        fmt.Println("Auth token not found in context")
        return
    }
    
    fmt.Printf("Processing request for user %s with token %s\n", userID, authToken)
    
    // Передаем контекст дальше
    processUserData(ctx)
}

func processUserData(ctx context.Context) {
    // Извлекаем userID из контекста
    userID, _ := ctx.Value(userIDKey).(string)
    fmt.Printf("Processing data for user %s\n", userID)
}

func main() {
    // Создаем базовый контекст
    ctx := context.Background()
    
    // Добавляем значения
    ctx = context.WithValue(ctx, userIDKey, "user123")
    ctx = context.WithValue(ctx, authTokenKey, "token456")
    
    // Обрабатываем запрос
    processRequest(ctx)
}
```

## Паттерн "Пул объектов" (Object Pool)

Пул объектов позволяет повторно использовать дорогостоящие в создании объекты.

### Реализация пула объектов

```go
type Pool struct {
    factory func() interface{}
    pool    chan interface{}
}

func NewPool(factory func() interface{}, size int) *Pool {
    return &Pool{
        factory: factory,
        pool:    make(chan interface{}, size),
    }
}

// Получение объекта из пула
func (p *Pool) Get() interface{} {
    select {
    case obj := <-p.pool:
        // Получаем существующий объект из пула
        return obj
    default:
        // Пул пуст, создаем новый объект
        return p.factory()
    }
}

// Возврат объекта в пул
func (p *Pool) Put(obj interface{}) {
    select {
    case p.pool <- obj:
        // Объект возвращен в пул
    default:
        // Пул заполнен, объект отбрасывается
    }
}

// Использование
func main() {
    // Создаем пул буферов
    bufferPool := NewPool(func() interface{} {
        fmt.Println("Creating new buffer")
        return new(bytes.Buffer)
    }, 10)
    
    // Получаем буфер из пула
    buf1 := bufferPool.Get().(*bytes.Buffer)
    buf1.WriteString("Hello")
    fmt.Println("Buffer 1:", buf1.String())
    
    // Возвращаем буфер в пул
    buf1.Reset() // Очищаем перед возвратом
    bufferPool.Put(buf1)
    
    // Получаем буфер из пула (будет использован buf1)
    buf2 := bufferPool.Get().(*bytes.Buffer)
    buf2.WriteString("World")
    fmt.Println("Buffer 2:", buf2.String())
}
```

### Пул объектов с очисткой

```go
type PoolWithCleanup struct {
    factory func() interface{}
    cleanup func(interface{})
    pool    chan interface{}
}

func NewPoolWithCleanup(factory func() interface{}, cleanup func(interface{}), size int) *PoolWithCleanup {
    return &PoolWithCleanup{
        factory: factory,
        cleanup: cleanup,
        pool:    make(chan interface{}, size),
    }
}

// Получение объекта из пула
func (p *PoolWithCleanup) Get() interface{} {
    select {
    case obj := <-p.pool:
        return obj
    default:
        return p.factory()
    }
}

// Возврат объекта в пул
func (p *PoolWithCleanup) Put(obj interface{}) {
    // Очищаем объект перед возвратом
    p.cleanup(obj)
    
    select {
    case p.pool <- obj:
        // Объект возвращен в пул
    default:
        // Пул заполнен, объект отбрасывается
    }
}

// Использование
func main() {
    // Создаем пул соединений с базой данных
    connPool := NewPoolWithCleanup(
        func() interface{} {
            fmt.Println("Opening database connection")
            return &fakeDBConn{id: rand.Intn(1000)}
        },
        func(obj interface{}) {
            conn := obj.(*fakeDBConn)
            fmt.Printf("Resetting connection %d\n", conn.id)
            conn.inUse = false
        },
        5,
    )
    
    // Получаем соединение
    conn1 := connPool.Get().(*fakeDBConn)
    conn1.inUse = true
    fmt.Printf("Using connection %d\n", conn1.id)
    
    // Возвращаем соединение в пул
    connPool.Put(conn1)
    
    // Получаем соединение из пула
    conn2 := connPool.Get().(*fakeDBConn)
    fmt.Printf("Reusing connection %d, in use: %v\n", conn2.id, conn2.inUse)
}

// Имитация соединения с базой данных
type fakeDBConn struct {
    id    int
    inUse bool
}
```

## Заключение

В этом руководстве мы рассмотрели продвинутые паттерны конкурентного программирования в Go, которые помогут вам писать эффективный, безопасный и масштабируемый код. Эти паттерны особенно важны для разработки высоконагруженных систем и могут быть полезны на техническом интервью.

Ключевые моменты:
1. Конвейеры (Pipelines) позволяют организовать последовательную обработку данных.
2. Паттерн "Вентилятор" (Fan-Out, Fan-In) помогает распараллелить работу и собрать результаты.
3. Семафоры ограничивают количество одновременно выполняемых операций.
4. Воркер-пулы эффективно распределяют задачи между горутинами.
5. Паттерн "Издатель-Подписчик" обеспечивает слабую связанность компонентов.
6. Барьеры синхронизируют группы горутин в определенных точках выполнения.
7. Паттерн "Будущее" представляет результаты асинхронных операций.
8. Контексты управляют жизненным циклом горутин и передают значения.
9. Пулы объектов повышают производительность за счет повторного использования ресурсов.

Эти паттерны являются мощными инструментами в арсенале Go-разработчика и помогут вам создавать высокопроизводительные конкурентные приложения.
