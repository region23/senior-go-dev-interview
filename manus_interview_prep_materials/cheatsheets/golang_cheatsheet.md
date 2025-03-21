# Шпаргалка по Golang для технического интервью

## Основные типы данных

### Базовые типы
- **bool**: true или false
- **string**: строка (UTF-8)
- **int, int8, int16, int32, int64**: целые числа
- **uint, uint8, uint16, uint32, uint64**: беззнаковые целые
- **float32, float64**: числа с плавающей точкой
- **complex64, complex128**: комплексные числа
- **byte**: псевдоним для uint8
- **rune**: псевдоним для int32 (Unicode код)

### Составные типы
- **array**: фиксированный размер `[n]T`
- **slice**: динамический массив `[]T`
- **map**: хеш-таблица `map[K]V`
- **struct**: структура с полями
- **interface**: набор методов
- **channel**: канал для коммуникации между горутинами `chan T`
- **pointer**: указатель `*T`

## Объявление переменных

```go
// Явное объявление типа
var name string = "John"

// Вывод типа
var name = "John"

// Краткое объявление (только внутри функций)
name := "John"

// Множественное объявление
var a, b, c int = 1, 2, 3
x, y := 10, "hello"

// Константы
const Pi = 3.14159
```

## Управляющие конструкции

### Условия
```go
if x > 0 {
    // код
} else if x < 0 {
    // код
} else {
    // код
}

// С инициализацией
if err := doSomething(); err != nil {
    // обработка ошибки
}
```

### Циклы
```go
// Только for, нет while/do-while
for i := 0; i < 10; i++ {
    // код
}

// Бесконечный цикл
for {
    // код
    if condition {
        break
    }
}

// Как while
for condition {
    // код
}

// Перебор коллекций
for index, value := range collection {
    // код
}
```

### Switch
```go
switch value {
case 1:
    // код
case 2, 3:
    // код для 2 или 3
default:
    // код по умолчанию
}

// С условиями
switch {
case x > 0:
    // код
case x < 0:
    // код
default:
    // код
}
```

## Функции

```go
// Базовая функция
func add(a, b int) int {
    return a + b
}

// Множественный возврат
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

// Именованные возвращаемые значения
func split(sum int) (x, y int) {
    x = sum * 4 / 9
    y = sum - x
    return // naked return
}

// Variadic функции
func sum(nums ...int) int {
    total := 0
    for _, num := range nums {
        total += num
    }
    return total
}

// Функции как значения
add := func(a, b int) int {
    return a + b
}

// Замыкания
func adder() func(int) int {
    sum := 0
    return func(x int) int {
        sum += x
        return sum
    }
}
```

## Структуры и методы

```go
// Определение структуры
type Person struct {
    Name string
    Age  int
}

// Создание экземпляра
p := Person{Name: "John", Age: 30}
p2 := Person{"John", 30} // без имен полей
p3 := new(Person) // указатель на Person со значениями по умолчанию

// Доступ к полям
fmt.Println(p.Name)
p.Age = 31

// Метод (value receiver)
func (p Person) Greet() string {
    return fmt.Sprintf("Hello, my name is %s", p.Name)
}

// Метод (pointer receiver)
func (p *Person) Birthday() {
    p.Age++
}
```

## Интерфейсы

```go
// Определение интерфейса
type Greeter interface {
    Greet() string
}

// Неявная реализация
func (p Person) Greet() string {
    return fmt.Sprintf("Hello, my name is %s", p.Name)
}

// Использование
var g Greeter = Person{Name: "John"}
fmt.Println(g.Greet())

// Пустой интерфейс
var i interface{} = "hello"

// Type assertion
str, ok := i.(string)
if ok {
    fmt.Println(str)
}

// Type switch
switch v := i.(type) {
case string:
    fmt.Println("string:", v)
case int:
    fmt.Println("int:", v)
default:
    fmt.Println("unknown type")
}
```

## Обработка ошибок

```go
// Возврат ошибки
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

// Проверка ошибки
result, err := divide(10, 0)
if err != nil {
    log.Fatal(err)
}

// Создание пользовательской ошибки
type MyError struct {
    When time.Time
    What string
}

func (e *MyError) Error() string {
    return fmt.Sprintf("%v: %v", e.When, e.What)
}

// Паника и восстановление
func doSomething() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered:", r)
        }
    }()
    
    panic("something went wrong")
}
```

## Горутины и каналы

```go
// Запуск горутины
go func() {
    // код
}()

// Создание канала
ch := make(chan int)      // небуферизованный
ch := make(chan int, 10)  // буферизованный

// Отправка и получение
ch <- 42      // отправка
value := <-ch // получение

// Закрытие канала
close(ch)

// Проверка закрытия
value, ok := <-ch
if !ok {
    // канал закрыт
}

// Перебор канала
for value := range ch {
    // код
}

// Select
select {
case v := <-ch1:
    // обработка v из ch1
case ch2 <- x:
    // отправка x в ch2
case <-time.After(1 * time.Second):
    // таймаут
default:
    // выполняется, если все каналы блокированы
}
```

## Синхронизация

```go
// Mutex
var mu sync.Mutex
mu.Lock()
// критическая секция
mu.Unlock()

// RWMutex
var mu sync.RWMutex
mu.RLock()  // блокировка для чтения
// чтение
mu.RUnlock()

mu.Lock()    // блокировка для записи
// запись
mu.Unlock()

// WaitGroup
var wg sync.WaitGroup
wg.Add(n)  // добавить n горутин

go func() {
    defer wg.Done()  // уменьшить счетчик
    // код
}()

wg.Wait()  // ждать завершения всех горутин

// Once
var once sync.Once
once.Do(func() {
    // выполнится только один раз
})

// Cond
var mu sync.Mutex
cond := sync.NewCond(&mu)

// Сигнал
cond.Signal()

// Широковещательный сигнал
cond.Broadcast()

// Ожидание
cond.L.Lock()
for !condition {
    cond.Wait()
}
// код
cond.L.Unlock()
```

## Пакеты и импорт

```go
// Объявление пакета
package main

// Импорт
import "fmt"

// Множественный импорт
import (
    "fmt"
    "time"
)

// Импорт с псевдонимом
import (
    f "fmt"
    t "time"
)

// Экспорт (с заглавной буквы)
func PublicFunction() {}
var PublicVar int

// Приватное (с маленькой буквы)
func privateFunction() {}
var privateVar int
```

## Работа с файлами

```go
// Чтение файла
data, err := ioutil.ReadFile("file.txt")
if err != nil {
    log.Fatal(err)
}

// Запись в файл
err := ioutil.WriteFile("file.txt", []byte("Hello"), 0644)
if err != nil {
    log.Fatal(err)
}

// Открытие файла
file, err := os.Open("file.txt")
if err != nil {
    log.Fatal(err)
}
defer file.Close()

// Чтение по частям
buffer := make([]byte, 1024)
for {
    n, err := file.Read(buffer)
    if err == io.EOF {
        break
    }
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(string(buffer[:n]))
}
```

## Тестирование

```go
// math.go
package math

func Add(a, b int) int {
    return a + b
}

// math_test.go
package math

import "testing"

func TestAdd(t *testing.T) {
    got := Add(2, 3)
    want := 5
    if got != want {
        t.Errorf("Add(2, 3) = %d; want %d", got, want)
    }
}

// Запуск тестов
// go test

// Бенчмарк
func BenchmarkAdd(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Add(2, 3)
    }
}

// Запуск бенчмарков
// go test -bench=.
```

## Работа с JSON

```go
// Структура
type Person struct {
    Name string `json:"name"`
    Age  int    `json:"age,omitempty"`
}

// Маршалинг
p := Person{Name: "John", Age: 30}
data, err := json.Marshal(p)
if err != nil {
    log.Fatal(err)
}
fmt.Println(string(data))  // {"name":"John","age":30}

// Анмаршалинг
var p2 Person
err = json.Unmarshal(data, &p2)
if err != nil {
    log.Fatal(err)
}
```

## Контекст

```go
// Создание контекста
ctx := context.Background()
ctx := context.TODO()

// С таймаутом
ctx, cancel := context.WithTimeout(parentCtx, 5*time.Second)
defer cancel()

// С дедлайном
ctx, cancel := context.WithDeadline(parentCtx, time.Now().Add(5*time.Second))
defer cancel()

// С отменой
ctx, cancel := context.WithCancel(parentCtx)
defer cancel()

// С значением
ctx = context.WithValue(ctx, "key", "value")
value := ctx.Value("key").(string)

// Проверка отмены
select {
case <-ctx.Done():
    err := ctx.Err()
    if err == context.Canceled {
        // контекст отменен
    } else if err == context.DeadlineExceeded {
        // таймаут
    }
case <-time.After(1 * time.Second):
    // продолжаем работу
}
```

## Особенности Go

### Указатели
```go
var x int = 10
var p *int = &x  // p указывает на x
fmt.Println(*p)  // разыменование: 10
*p = 20          // изменение x через p
```

### Слайсы
```go
// Создание
s := []int{1, 2, 3}
s := make([]int, 5)      // длина и емкость 5
s := make([]int, 3, 5)   // длина 3, емкость 5

// Добавление
s = append(s, 4, 5, 6)

// Срезы
s2 := s[1:4]  // элементы с индексами 1, 2, 3

// Копирование
dst := make([]int, len(src))
copy(dst, src)
```

### Мапы
```go
// Создание
m := make(map[string]int)
m := map[string]int{"one": 1, "two": 2}

// Доступ
value, exists := m["key"]
if exists {
    // ключ существует
}

// Удаление
delete(m, "key")

// Перебор
for key, value := range m {
    fmt.Println(key, value)
}
```

### Defer
```go
// Выполняется при выходе из функции
func example() {
    defer fmt.Println("last")
    fmt.Println("first")
}

// Множественные defer выполняются в обратном порядке (LIFO)
func example() {
    defer fmt.Println("3")
    defer fmt.Println("2")
    defer fmt.Println("1")
}
// Вывод: 1, 2, 3
```

## Идиомы Go

### Обработка ошибок
```go
// Проверка ошибок
if err != nil {
    return err  // или обработка
}

// Множественный возврат для ошибок
result, err := someFunc()
if err != nil {
    return nil, err
}
```

### Пустой идентификатор
```go
// Игнорирование значений
_, err := someFunc()

// Игнорирование индекса в range
for _, value := range collection {
    // код
}
```

### Композиция вместо наследования
```go
type Engine struct {
    Power int
}

func (e *Engine) Start() {
    fmt.Println("Engine started")
}

type Car struct {
    Engine  // встраивание
    Model string
}

// Car получает все методы Engine
car := Car{Engine: Engine{Power: 100}, Model: "Tesla"}
car.Start()  // "Engine started"
```

### Функциональные опции
```go
type Server struct {
    addr     string
    port     int
    timeout  time.Duration
}

type Option func(*Server)

func WithPort(port int) Option {
    return func(s *Server) {
        s.port = port
    }
}

func WithTimeout(timeout time.Duration) Option {
    return func(s *Server) {
        s.timeout = timeout
    }
}

func NewServer(addr string, options ...Option) *Server {
    s := &Server{
        addr:    addr,
        port:    8080,      // значение по умолчанию
        timeout: 30 * time.Second,  // значение по умолчанию
    }
    
    for _, option := range options {
        option(s)
    }
    
    return s
}

// Использование
server := NewServer("localhost", WithPort(9000), WithTimeout(1*time.Minute))
```

## Производительность и оптимизация

### Escape-анализ
```go
// Переменная не "убегает" - размещается на стеке
func Sum(a, b int) int {
    result := a + b
    return result
}

// Переменная "убегает" - размещается в куче
func NewPerson() *Person {
    p := Person{Name: "John"}
    return &p  // возврат адреса локальной переменной
}
```

### Пулы объектов
```go
var bufferPool = sync.Pool{
    New: func() interface{} {
        return new(bytes.Buffer)
    },
}

func process() {
    buf := bufferPool.Get().(*bytes.Buffer)
    buf.Reset()
    defer bufferPool.Put(buf)
    
    // использование buf
}
```

### Предварительное выделение памяти
```go
// Плохо: многократное выделение памяти
s := []int{}
for i := 0; i < 10000; i++ {
    s = append(s, i)
}

// Хорошо: однократное выделение
s := make([]int, 0, 10000)
for i := 0; i < 10000; i++ {
    s = append(s, i)
}
```

## Внутреннее устройство

### Горутины
- Легковесные потоки (2-4 КБ стека изначально)
- Управляются планировщиком Go, не ОС
- Мультиплексируются на OS-потоки (M:N scheduling)

### Сборка мусора
- Параллельная, трехцветная маркировка
- Малые паузы (< 1ms)
- Работает конкурентно с программой

### Модель памяти
- Слабая консистентность
- Необходима явная синхронизация для гарантии порядка операций
- Каналы обеспечивают синхронизацию (happens-before)
