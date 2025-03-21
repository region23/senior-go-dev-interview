# Многопоточность и конкурентность в Go

## Введение

Одной из сильных сторон Go является встроенная поддержка конкурентного программирования. В этом руководстве мы подробно рассмотрим модель конкурентности Go, горутины, каналы, механизмы синхронизации и управление памятью в многопоточных программах.

## Горутины (Goroutines)

Горутины - это легковесные потоки выполнения, управляемые планировщиком Go. Они позволяют выполнять функции конкурентно.

### Основы горутин

```go
// Запуск функции в горутине
go func() {
    // Код, выполняемый конкурентно
    fmt.Println("Hello from goroutine!")
}()

// Запуск именованной функции в горутине
go sayHello("World")

// Основная горутина продолжает выполнение
fmt.Println("Hello from main goroutine!")
```

### Внутреннее устройство горутин

Горутины отличаются от системных потоков:

1. **Легковесность**: Горутины занимают всего около 2 КБ памяти при создании, в то время как системные потоки могут занимать мегабайты.

2. **Мультиплексирование**: Множество горутин выполняются в рамках небольшого числа системных потоков (обычно по числу ядер процессора).

3. **Планировщик Go**: Управляет выполнением горутин, распределяя их по доступным потокам.

4. **Кооперативная многозадачность**: Горутины добровольно уступают управление в определенных точках (ввод/вывод, блокировки, вызовы функций).

### Жизненный цикл горутины

1. **Создание**: Горутина создается с помощью ключевого слова `go`.
2. **Выполнение**: Планировщик Go распределяет горутины по системным потокам.
3. **Блокировка**: Горутина может быть заблокирована (например, при ожидании ввода/вывода или синхронизации).
4. **Возобновление**: После разблокировки горутина возвращается в очередь на выполнение.
5. **Завершение**: Горутина завершается, когда выполнение ее функции завершается.

### Ожидание завершения горутин

Для ожидания завершения горутин используется пакет `sync` и его тип `WaitGroup`:

```go
var wg sync.WaitGroup

// Добавляем счетчик горутин
wg.Add(2)

go func() {
    defer wg.Done() // Уменьшаем счетчик при завершении
    fmt.Println("Goroutine 1 done")
}()

go func() {
    defer wg.Done() // Уменьшаем счетчик при завершении
    fmt.Println("Goroutine 2 done")
}()

// Ожидаем завершения всех горутин
wg.Wait()
fmt.Println("All goroutines completed")
```

### Утечки горутин

Утечка горутин происходит, когда горутина не завершается и продолжает потреблять ресурсы:

```go
// Пример утечки горутин
func leakyFunction() {
    go func() {
        // Эта горутина никогда не завершится
        for {
            // Бесконечный цикл без условия выхода
        }
    }()
}
```

Способы предотвращения утечек:
1. Использование контекста для отмены горутин
2. Явное завершение горутин с помощью сигнальных каналов
3. Тщательное проектирование жизненного цикла горутин

## Каналы (Channels)

Каналы - это типизированные конвейеры для коммуникации между горутинами.

### Основы каналов

```go
// Создание канала
ch := make(chan int) // Небуферизованный канал для целых чисел

// Отправка значения в канал
go func() {
    ch <- 42 // Блокируется, пока значение не будет получено
}()

// Получение значения из канала
value := <-ch // Блокируется, пока значение не будет отправлено
fmt.Println(value) // 42
```

### Буферизованные каналы

```go
// Создание буферизованного канала емкостью 3
ch := make(chan int, 3)

// Отправка значений (не блокируется, пока буфер не заполнен)
ch <- 1
ch <- 2
ch <- 3
// ch <- 4 // Заблокируется, так как буфер полон

// Получение значений
fmt.Println(<-ch) // 1
fmt.Println(<-ch) // 2
fmt.Println(<-ch) // 3
```

### Направленные каналы

```go
// Канал только для отправки
func send(ch chan<- int) {
    ch <- 42
}

// Канал только для получения
func receive(ch <-chan int) {
    value := <-ch
    fmt.Println(value)
}

// Использование
ch := make(chan int)
go send(ch)
receive(ch)
```

### Закрытие каналов

```go
ch := make(chan int, 3)

// Отправка значений
ch <- 1
ch <- 2
ch <- 3

// Закрытие канала
close(ch)

// Получение значений из закрытого канала
for value := range ch {
    fmt.Println(value) // 1, 2, 3
}

// Проверка, закрыт ли канал
value, ok := <-ch
if !ok {
    fmt.Println("Channel is closed")
}
```

### Выбор каналов (Select)

Оператор `select` позволяет горутине ожидать операции на нескольких каналах:

```go
select {
case value := <-ch1:
    // Обработка значения из ch1
    fmt.Println("Received from ch1:", value)
    
case ch2 <- 42:
    // Отправка в ch2 успешна
    fmt.Println("Sent to ch2")
    
case <-time.After(1 * time.Second):
    // Тайм-аут, если ни один из каналов не готов в течение 1 секунды
    fmt.Println("Timeout")
    
default:
    // Выполняется, если ни один из каналов не готов (неблокирующий select)
    fmt.Println("No channel is ready")
}
```

### Паттерны использования каналов

#### 1. Генератор (Fan-Out)

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

// Использование
for n := range generator(1, 2, 3, 4) {
    fmt.Println(n)
}
```

#### 2. Мультиплексирование (Fan-In)

```go
func fanIn(channels ...<-chan int) <-chan int {
    out := make(chan int)
    var wg sync.WaitGroup
    
    // Функция для перенаправления значений из входного канала в выходной
    multiplex := func(ch <-chan int) {
        defer wg.Done()
        for n := range ch {
            out <- n
        }
    }
    
    // Запускаем горутину для каждого входного канала
    wg.Add(len(channels))
    for _, ch := range channels {
        go multiplex(ch)
    }
    
    // Закрываем выходной канал после завершения всех горутин
    go func() {
        wg.Wait()
        close(out)
    }()
    
    return out
}

// Использование
ch1 := generator(1, 2, 3)
ch2 := generator(4, 5, 6)
for n := range fanIn(ch1, ch2) {
    fmt.Println(n)
}
```

#### 3. Конвейер (Pipeline)

```go
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

// Использование
nums := generator(1, 2, 3, 4)
squares := square(nums)
for n := range squares {
    fmt.Println(n) // 1, 4, 9, 16
}
```

#### 4. Отмена с помощью контекста

```go
func worker(ctx context.Context) <-chan int {
    out := make(chan int)
    go func() {
        defer close(out)
        for i := 0; ; i++ {
            select {
            case <-ctx.Done():
                return // Завершаем горутину при отмене контекста
            case out <- i:
                time.Sleep(100 * time.Millisecond)
            }
        }
    }()
    return out
}

// Использование
ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
defer cancel()

for n := range worker(ctx) {
    fmt.Println(n)
}
fmt.Println("Done")
```

## Механизмы синхронизации

### Мьютексы (Mutex)

Мьютексы используются для защиты доступа к общим ресурсам:

```go
var (
    counter int
    mu      sync.Mutex
)

func increment() {
    mu.Lock()
    defer mu.Unlock()
    counter++
}

// Использование
var wg sync.WaitGroup
for i := 0; i < 1000; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        increment()
    }()
}
wg.Wait()
fmt.Println("Counter:", counter) // 1000
```

### RWMutex (Read-Write Mutex)

RWMutex позволяет нескольким горутинам читать данные одновременно, но только одной горутине писать:

```go
var (
    data   map[string]string
    rwmu   sync.RWMutex
)

func init() {
    data = make(map[string]string)
}

// Запись (эксклюзивная блокировка)
func write(key, value string) {
    rwmu.Lock()
    defer rwmu.Unlock()
    data[key] = value
}

// Чтение (разделяемая блокировка)
func read(key string) string {
    rwmu.RLock()
    defer rwmu.RUnlock()
    return data[key]
}
```

### Условные переменные (Cond)

Условные переменные используются для ожидания определенного условия:

```go
var (
    data     []int
    mu       sync.Mutex
    dataReady *sync.Cond
)

func init() {
    dataReady = sync.NewCond(&mu)
}

// Производитель
func producer() {
    mu.Lock()
    defer mu.Unlock()
    
    // Добавляем данные
    data = append(data, 1, 2, 3)
    
    // Сигнализируем о готовности данных
    dataReady.Broadcast()
}

// Потребитель
func consumer() {
    mu.Lock()
    defer mu.Unlock()
    
    // Ждем, пока данные не будут готовы
    for len(data) == 0 {
        dataReady.Wait()
    }
    
    // Обрабатываем данные
    fmt.Println("Received:", data)
}
```

### Once

`sync.Once` гарантирует, что функция будет выполнена только один раз:

```go
var (
    instance *singleton
    once     sync.Once
)

func getInstance() *singleton {
    once.Do(func() {
        instance = &singleton{}
    })
    return instance
}
```

### Atomic операции

Атомарные операции обеспечивают безопасное изменение значений без использования мьютексов:

```go
var counter int64

// Инкремент
atomic.AddInt64(&counter, 1)

// Загрузка значения
value := atomic.LoadInt64(&counter)

// Сохранение значения
atomic.StoreInt64(&counter, 42)

// Сравнение и замена
oldValue := int64(10)
newValue := int64(20)
swapped := atomic.CompareAndSwapInt64(&counter, oldValue, newValue)
```

### Барьеры памяти (Memory Barriers)

Атомарные операции также создают барьеры памяти, гарантирующие видимость изменений для других горутин:

```go
var (
    ready int32
    data  []int
)

func setup() {
    // Подготовка данных
    data = []int{1, 2, 3}
    
    // Установка флага готовности (с барьером памяти)
    atomic.StoreInt32(&ready, 1)
}

func process() {
    // Проверка флага готовности (с барьером памяти)
    for atomic.LoadInt32(&ready) == 0 {
        // Ждем, пока данные не будут готовы
        time.Sleep(10 * time.Millisecond)
    }
    
    // Теперь данные гарантированно видны
    fmt.Println("Data:", data)
}
```

## Модель памяти Go

### Гарантии модели памяти

Модель памяти Go определяет, когда одна горутина может видеть изменения, сделанные другой горутиной:

1. **Одиночная горутина**: Операции в одной горутине выполняются в порядке, указанном в программе.

2. **Инициализация**: Если пакет p импортирует пакет q, то инициализация q завершается до начала инициализации p.

3. **Создание горутины**: Оператор `go` происходит до начала выполнения новой горутины.

4. **Операции с каналами**: Отправка в канал происходит до соответствующего получения из канала.

5. **Блокировки**: Для любого мьютекса/RWMutex, n-й вызов Unlock() происходит до (n+1)-го вызова Lock().

### Происходит до (Happens Before)

Отношение "происходит до" (happens-before) определяет видимость операций между горутинами:

```go
var a, b int

func setup() {
    a = 1                // Операция A
    b = 2                // Операция B
    fmt.Println(a, b)    // Гарантированно выведет "1 2"
}

func main() {
    go setup()           // Горутина 1
    
    time.Sleep(1 * time.Second)
    
    fmt.Println(a, b)    // Горутина 2: может вывести "0 0", "1 0", "0 2" или "1 2"
}
```

Без синхронизации нет гарантии, что изменения из одной горутины будут видны в другой.

### Синхронизация для обеспечения видимости

```go
var a, b int
var wg sync.WaitGroup

func setup() {
    defer wg.Done()
    a = 1
    b = 2
}

func main() {
    wg.Add(1)
    go setup()           // Горутина 1
    
    wg.Wait()            // Синхронизация
    
    fmt.Println(a, b)    // Гарантированно выведет "1 2"
}
```

## Паттерны конкурентного программирования

### 1. Worker Pool

```go
func workerPool(numWorkers int, jobs <-chan int, results chan<- int) {
    var wg sync.WaitGroup
    
    // Запускаем фиксированное количество воркеров
    for i := 0; i < numWorkers; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            
            for job := range jobs {
                // Обработка задачи
                result := job * 2
                results <- result
                fmt.Printf("Worker %d processed job %d -> %d\n", id, job, result)
            }
        }(i)
    }
    
    // Ждем завершения всех воркеров
    go func() {
        wg.Wait()
        close(results)
    }()
}

// Использование
jobs := make(chan int, 100)
results := make(chan int, 100)

// Запускаем пул из 3 воркеров
workerPool(3, jobs, results)

// Отправляем задачи
for i := 0; i < 10; i++ {
    jobs <- i
}
close(jobs)

// Получаем результаты
for result := range results {
    fmt.Println("Result:", result)
}
```

### 2. Ограничение скорости (Rate Limiting)

```go
func rateLimiter(requests <-chan int, limit time.Duration) <-chan int {
    limiter := time.Tick(limit)
    out := make(chan int)
    
    go func() {
        defer close(out)
        for req := range requests {
            <-limiter // Ждем разрешения от лимитера
            out <- req
        }
    }()
    
    return out
}

// Использование
requests := make(chan int, 10)
for i := 0; i < 10; i++ {
    requests <- i
}
close(requests)

// Обрабатываем не более 2 запросов в секунду
for req := range rateLimiter(requests, 500*time.Millisecond) {
    fmt.Println("Processing request:", req)
}
```

### 3. Таймаут (Timeout)

```go
func processWithTimeout(input int, timeout time.Duration) (result int, err error) {
    resultCh := make(chan int, 1)
    
    go func() {
        // Имитация длительной обработки
        time.Sleep(100 * time.Millisecond)
        resultCh <- input * 2
    }()
    
    select {
    case result := <-resultCh:
        return result, nil
    case <-time.After(timeout):
        return 0, errors.New("processing timed out")
    }
}

// Использование
result, err := processWithTimeout(42, 200*time.Millisecond)
if err != nil {
    fmt.Println("Error:", err)
} else {
    fmt.Println("Result:", result)
}
```

### 4. Отмена операций (Cancellation)

```go
func processWithCancellation(ctx context.Context, input int) (result int, err error) {
    resultCh := make(chan int, 1)
    
    go func() {
        // Проверяем отмену перед началом работы
        select {
        case <-ctx.Done():
            return
        default:
            // Продолжаем выполнение
        }
        
        // Имитация длительной обработки с периодической проверкой отмены
        for i := 0; i < 10; i++ {
            select {
            case <-ctx.Done():
                return
            default:
                time.Sleep(10 * time.Millisecond)
            }
        }
        
        resultCh <- input * 2
    }()
    
    select {
    case result := <-resultCh:
        return result, nil
    case <-ctx.Done():
        return 0, ctx.Err()
    }
}

// Использование
ctx, cancel := context.WithTimeout(context.Background(), 50*time.Millisecond)
defer cancel()

result, err := processWithCancellation(ctx, 42)
if err != nil {
    fmt.Println("Error:", err)
} else {
    fmt.Println("Result:", result)
}
```

## Управление памятью в многопоточных программах

### Сборка мусора и горутины

Сборщик мусора Go работает конкурентно с выполнением программы:

1. **Трехцветная маркировка**: Алгоритм сборки мусора использует трехцветную маркировку (белый, серый, черный) для отслеживания объектов.

2. **Параллельная сборка**: Сборщик мусора работает параллельно с выполнением программы, минимизируя паузы.

3. **Барьеры записи**: Обеспечивают корректность сборки мусора при конкурентных изменениях.

### Локальная память горутин (Goroutine-Local Storage)

В Go нет встроенной поддержки локальной памяти потоков (TLS), но можно использовать контекст или замыкания:

```go
func worker(id int) {
    // Локальные переменные горутины
    buffer := make([]byte, 1024)
    counter := 0
    
    // Эти переменные доступны только внутри этой горутины
    for i := 0; i < 10; i++ {
        counter++
        // Использование buffer и counter
    }
}

// Запуск нескольких горутин
for i := 0; i < 5; i++ {
    go worker(i)
}
```

### Избегание гонок данных (Race Conditions)

Гонки данных возникают, когда несколько горутин одновременно обращаются к одним и тем же данным, и хотя бы одна из них выполняет запись:

```go
// Пример гонки данных
var counter int

func increment() {
    counter++ // Операция не атомарна: чтение, инкремент, запись
}

// Запуск горутин
for i := 0; i < 1000; i++ {
    go increment()
}
```

Обнаружение гонок данных:
```bash
go run -race main.go
go test -race ./...
```

### Стратегии избегания гонок данных

1. **Мьютексы**:
   ```go
   var (
       counter int
       mu      sync.Mutex
   )
   
   func increment() {
       mu.Lock()
       defer mu.Unlock()
       counter++
   }
   ```

2. **Атомарные операции**:
   ```go
   var counter int64
   
   func increment() {
       atomic.AddInt64(&counter, 1)
   }
   ```

3. **Каналы**:
   ```go
   var counter int
   updates := make(chan int)
   
   // Горутина, управляющая доступом к counter
   go func() {
       for {
           select {
           case <-updates:
               counter++
           }
       }
   }()
   
   // Инкремент через канал
   func increment() {
       updates <- 1
   }
   ```

4. **Неизменяемые данные**:
   ```go
   type Data struct {
       values []int
   }
   
   // Создание новой структуры вместо изменения существующей
   func addValue(d Data, value int) Data {
       newValues := make([]int, len(d.values)+1)
       copy(newValues, d.values)
       newValues[len(d.values)] = value
       return Data{values: newValues}
   }
   ```

## Профилирование и отладка многопоточных программ

### Профилирование горутин

```go
import _ "net/http/pprof"

func main() {
    // Запуск HTTP-сервера для профилирования
    go func() {
        http.ListenAndServe("localhost:6060", nil)
    }()
    
    // Остальной код программы
    // ...
}
```

Просмотр активных горутин:
```bash
go tool pprof http://localhost:6060/debug/pprof/goroutine
```

### Трассировка выполнения

```go
import "runtime/trace"

func main() {
    // Создание файла для трассировки
    f, err := os.Create("trace.out")
    if err != nil {
        log.Fatal(err)
    }
    defer f.Close()
    
    // Запуск трассировки
    err = trace.Start(f)
    if err != nil {
        log.Fatal(err)
    }
    defer trace.Stop()
    
    // Остальной код программы
    // ...
}
```

Просмотр трассировки:
```bash
go tool trace trace.out
```

### Отладка дедлоков

Дедлок возникает, когда две или более горутины ожидают друг друга, и ни одна не может продолжить выполнение:

```go
func deadlock() {
    var mu1, mu2 sync.Mutex
    
    go func() {
        mu1.Lock()
        time.Sleep(100 * time.Millisecond) // Имитация работы
        mu2.Lock() // Ждет, пока mu2 будет разблокирован
        // ...
        mu2.Unlock()
        mu1.Unlock()
    }()
    
    go func() {
        mu2.Lock()
        time.Sleep(100 * time.Millisecond) // Имитация работы
        mu1.Lock() // Ждет, пока mu1 будет разблокирован
        // ...
        mu1.Unlock()
        mu2.Unlock()
    }()
}
```

Предотвращение дедлоков:
1. Всегда блокировать мьютексы в одном и том же порядке
2. Использовать тайм-ауты для блокировок
3. Использовать `sync.Mutex.TryLock()` (в Go 1.18+)
4. Применять контексты с отменой

## Практические задания

1. Реализуйте конкурентный веб-краулер, который обходит веб-страницы параллельно, но с ограничением на количество одновременных запросов.

2. Создайте пул воркеров для обработки задач из очереди с возможностью отмены всех операций по сигналу.

3. Реализуйте конкурентную карту (concurrent map) с поддержкой безопасного доступа из нескольких горутин.

4. Напишите программу, которая параллельно вычисляет сумму элементов большого массива, разделяя его на части.

5. Реализуйте паттерн "издатель-подписчик" (pub-sub) с использованием горутин и каналов.
