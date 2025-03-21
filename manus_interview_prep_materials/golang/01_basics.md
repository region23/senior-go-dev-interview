# Основы Golang для технического интервью

## Введение

Golang (или Go) - это статически типизированный, компилируемый язык программирования, разработанный в Google. Он сочетает в себе простоту синтаксиса, эффективность компилируемых языков и удобство сборки мусора. В этом руководстве мы рассмотрим основные концепции Go, которые могут быть полезны при подготовке к техническому интервью.

## Типы данных в Go

### Базовые типы

Go предоставляет несколько базовых типов данных:

1. **Целочисленные типы**:
   - `int`, `int8`, `int16`, `int32`, `int64` (со знаком)
   - `uint`, `uint8`, `uint16`, `uint32`, `uint64` (без знака)
   - `uintptr` (беззнаковое целое, достаточное для хранения указателя)

2. **Числа с плавающей точкой**:
   - `float32`, `float64`

3. **Комплексные числа**:
   - `complex64`, `complex128`

4. **Булевы значения**:
   - `bool` (true или false)

5. **Строки**:
   - `string` (неизменяемая последовательность байтов)

6. **Руны**:
   - `rune` (синоним для int32, представляет Unicode код)

### Составные типы

1. **Массивы**:
   ```go
   var a [5]int // Массив из 5 целых чисел
   ```

2. **Слайсы** (динамические массивы):
   ```go
   var s []int // Слайс целых чисел
   s = append(s, 1, 2, 3) // Добавление элементов
   ```

3. **Структуры**:
   ```go
   type Person struct {
       Name string
       Age  int
   }
   ```

4. **Карты** (ассоциативные массивы):
   ```go
   var m map[string]int // Карта со строковыми ключами и целочисленными значениями
   m = make(map[string]int)
   m["key"] = 42
   ```

5. **Каналы**:
   ```go
   ch := make(chan int) // Канал для передачи целых чисел
   ```

6. **Указатели**:
   ```go
   var p *int // Указатель на целое число
   i := 42
   p = &i // p указывает на i
   ```

## Объявление переменных

В Go есть несколько способов объявления переменных:

1. **Полное объявление**:
   ```go
   var name string = "John"
   ```

2. **С выводом типа**:
   ```go
   var name = "John" // Тип выводится как string
   ```

3. **Краткое объявление** (только внутри функций):
   ```go
   name := "John" // Эквивалентно var name = "John"
   ```

4. **Множественное объявление**:
   ```go
   var a, b, c int = 1, 2, 3
   x, y := 10, "hello"
   ```

## Константы

Константы в Go объявляются с помощью ключевого слова `const`:

```go
const Pi = 3.14159
const (
    StatusOK = 200
    StatusNotFound = 404
)
```

Go также поддерживает неявно типизированные константы с помощью `iota`:

```go
const (
    Monday = iota // 0
    Tuesday       // 1
    Wednesday     // 2
    Thursday      // 3
    Friday        // 4
)
```

## Управляющие конструкции

### Условные операторы

```go
if x > 0 {
    // код, если x > 0
} else if x < 0 {
    // код, если x < 0
} else {
    // код, если x == 0
}
```

Go также поддерживает инициализацию переменной в условии:

```go
if err := doSomething(); err != nil {
    // обработка ошибки
}
```

### Циклы

В Go есть только один тип цикла - `for`:

```go
// Стандартный цикл for
for i := 0; i < 10; i++ {
    // тело цикла
}

// Цикл while
for condition {
    // тело цикла
}

// Бесконечный цикл
for {
    // тело цикла
    if shouldBreak {
        break
    }
}
```

Для итерации по коллекциям используется `range`:

```go
// Итерация по слайсу
for index, value := range slice {
    // использование index и value
}

// Итерация по карте
for key, value := range map {
    // использование key и value
}

// Итерация по строке
for index, char := range "hello" {
    // char - это руна (Unicode код)
}
```

### Switch

```go
switch os := runtime.GOOS; os {
case "darwin":
    fmt.Println("OS X")
case "linux":
    fmt.Println("Linux")
default:
    fmt.Printf("%s", os)
}
```

В Go не нужен явный `break` после каждого case, и можно использовать несколько значений в одном case:

```go
switch day {
case "Monday", "Tuesday", "Wednesday", "Thursday", "Friday":
    fmt.Println("Weekday")
case "Saturday", "Sunday":
    fmt.Println("Weekend")
}
```

## Функции

### Объявление функций

```go
func add(x int, y int) int {
    return x + y
}

// Сокращенное объявление параметров одного типа
func add(x, y int) int {
    return x + y
}
```

### Множественные возвращаемые значения

```go
func divAndRemainder(x, y int) (int, int) {
    return x / y, x % y
}

quotient, remainder := divAndRemainder(10, 3)
```

### Именованные возвращаемые значения

```go
func divAndRemainder(x, y int) (quotient, remainder int) {
    quotient = x / y
    remainder = x % y
    return // "голый" return возвращает именованные значения
}
```

### Функции как значения и замыкания

```go
// Функция как значение
add := func(x, y int) int {
    return x + y
}
result := add(3, 4)

// Замыкание
func makeAdder(x int) func(int) int {
    return func(y int) int {
        return x + y
    }
}
add5 := makeAdder(5)
result := add5(3) // 8
```

### Вариативные функции

```go
func sum(nums ...int) int {
    total := 0
    for _, num := range nums {
        total += num
    }
    return total
}

result := sum(1, 2, 3, 4) // 10
```

## Структуры и методы

### Определение структур

```go
type Rectangle struct {
    Width  float64
    Height float64
}
```

### Создание экземпляров структур

```go
// Создание с указанием полей
r1 := Rectangle{Width: 10, Height: 5}

// Создание в порядке объявления полей
r2 := Rectangle{10, 5}

// Создание с нулевыми значениями
var r3 Rectangle

// Создание с помощью new (возвращает указатель)
r4 := new(Rectangle)
```

### Методы

В Go нет классов, но есть методы, которые связаны с типами:

```go
// Метод для типа Rectangle
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

// Использование метода
r := Rectangle{Width: 10, Height: 5}
area := r.Area() // 50
```

### Методы с указателями-получателями

```go
// Метод, изменяющий структуру
func (r *Rectangle) Scale(factor float64) {
    r.Width *= factor
    r.Height *= factor
}

// Использование
r := Rectangle{Width: 10, Height: 5}
r.Scale(2)
fmt.Println(r) // {20 10}
```

## Интерфейсы

### Определение интерфейсов

```go
type Shape interface {
    Area() float64
    Perimeter() float64
}
```

### Реализация интерфейсов

В Go нет явного указания на реализацию интерфейса. Тип реализует интерфейс, если он имеет все методы, объявленные в интерфейсе:

```go
// Rectangle реализует интерфейс Shape
func (r Rectangle) Area() float64 {
    return r.Width * r.Height
}

func (r Rectangle) Perimeter() float64 {
    return 2 * (r.Width + r.Height)
}

// Circle также реализует интерфейс Shape
type Circle struct {
    Radius float64
}

func (c Circle) Area() float64 {
    return math.Pi * c.Radius * c.Radius
}

func (c Circle) Perimeter() float64 {
    return 2 * math.Pi * c.Radius
}

// Функция, принимающая интерфейс
func PrintShapeInfo(s Shape) {
    fmt.Printf("Area: %f, Perimeter: %f\n", s.Area(), s.Perimeter())
}

// Использование
r := Rectangle{Width: 10, Height: 5}
c := Circle{Radius: 5}
PrintShapeInfo(r)
PrintShapeInfo(c)
```

### Пустой интерфейс

Пустой интерфейс `interface{}` не имеет методов и может содержать значение любого типа:

```go
var i interface{}
i = 42
i = "hello"
i = struct{ Name string }{"John"}
```

### Утверждение типа (Type Assertion)

```go
var i interface{} = "hello"

// Утверждение типа с проверкой
if s, ok := i.(string); ok {
    fmt.Println(s) // "hello"
}

// Утверждение типа без проверки (паника, если тип не соответствует)
s := i.(string)
```

### Переключение типов (Type Switch)

```go
func describe(i interface{}) {
    switch v := i.(type) {
    case int:
        fmt.Printf("Integer: %d\n", v)
    case string:
        fmt.Printf("String: %s\n", v)
    default:
        fmt.Printf("Unknown type\n")
    }
}

describe(42)      // "Integer: 42"
describe("hello") // "String: hello"
```

## Обработка ошибок

В Go ошибки представлены интерфейсом `error`:

```go
type error interface {
    Error() string
}
```

### Возвращение ошибок

```go
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

// Использование
result, err := divide(10, 0)
if err != nil {
    fmt.Println("Error:", err)
} else {
    fmt.Println("Result:", result)
}
```

### Создание пользовательских ошибок

```go
// Определение пользовательского типа ошибки
type MyError struct {
    When time.Time
    What string
}

// Реализация метода Error() для интерфейса error
func (e MyError) Error() string {
    return fmt.Sprintf("%v: %v", e.When, e.What)
}

// Функция, возвращающая пользовательскую ошибку
func doSomething() error {
    return MyError{
        When: time.Now(),
        What: "something went wrong",
    }
}
```

## Указатели и ссылочные типы

### Указатели

Указатель хранит адрес памяти переменной:

```go
var x int = 42
var p *int = &x // p указывает на x

fmt.Println(*p) // 42 (разыменование указателя)

*p = 21 // изменение значения x через указатель
fmt.Println(x) // 21
```

### Ссылочные типы

В Go некоторые типы являются ссылочными по своей природе:
- Слайсы
- Карты
- Каналы
- Функции

Это означает, что при передаче этих типов в функции, они передаются по ссылке, а не по значению:

```go
func modifySlice(s []int) {
    s[0] = 100
}

slice := []int{1, 2, 3}
modifySlice(slice)
fmt.Println(slice) // [100, 2, 3]
```

### Слайсы как ссылочные типы

Слайс состоит из трех компонентов:
- Указатель на базовый массив
- Длина
- Емкость

```go
// Создание слайса
s := make([]int, 5, 10) // длина 5, емкость 10

// Добавление элементов
s = append(s, 1, 2, 3)

// Создание подслайса (ссылается на тот же базовый массив)
sub := s[2:4]
sub[0] = 99 // изменяет также s[2]
```

### Карты как ссылочные типы

```go
func modifyMap(m map[string]int) {
    m["key"] = 100
}

myMap := make(map[string]int)
myMap["key"] = 42
modifyMap(myMap)
fmt.Println(myMap["key"]) // 100
```

## Пакеты и импорты

### Структура пакета

```go
// файл: mypackage/myfile.go
package mypackage

// Экспортируемая функция (начинается с заглавной буквы)
func ExportedFunction() {
    // ...
}

// Неэкспортируемая функция (начинается с маленькой буквы)
func privateFunction() {
    // ...
}
```

### Импорт пакетов

```go
import (
    "fmt"
    "math/rand"
    
    // Импорт с псевдонимом
    m "math"
    
    // Импорт только для побочных эффектов (инициализации)
    _ "image/png"
)
```

### Инициализация пакета

```go
package mypackage

// Переменные уровня пакета
var (
    privateVar = "private"
    PublicVar  = "public"
)

// Функция init вызывается при импорте пакета
func init() {
    // Код инициализации
}
```

## Заключение

Это базовый обзор основных концепций Go, которые могут быть полезны при подготовке к техническому интервью. Для более глубокого понимания рекомендуется изучить официальную документацию и практиковаться в написании кода на Go.

## Практические задания

1. Напишите функцию, которая переворачивает строку.
2. Реализуйте структуру Stack с методами Push, Pop и IsEmpty.
3. Напишите функцию, которая находит пересечение двух слайсов.
4. Создайте интерфейс Sorter и реализуйте его для разных типов данных.
5. Напишите функцию, которая безопасно обрабатывает панику с помощью defer и recover.
