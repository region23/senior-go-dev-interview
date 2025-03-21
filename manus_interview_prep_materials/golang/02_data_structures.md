# Структуры данных и ссылочные типы в Go

## Введение

Понимание структур данных и ссылочных типов в Go критически важно для написания эффективного кода и успешного прохождения технического интервью. В этом руководстве мы подробно рассмотрим основные структуры данных в Go, их внутреннюю реализацию и особенности работы с ссылочными типами.

## Слайсы (Slices)

Слайсы - одна из наиболее часто используемых структур данных в Go. Они представляют собой динамические массивы, которые могут изменять свой размер во время выполнения программы.

### Внутренняя структура слайса

Слайс состоит из трех компонентов:
1. Указатель на базовый массив
2. Длина (количество элементов в слайсе)
3. Емкость (размер базового массива от начала слайса)

```go
type slice struct {
    array unsafe.Pointer // указатель на базовый массив
    len   int            // длина
    cap   int            // емкость
}
```

### Создание слайсов

```go
// Создание пустого слайса
var s1 []int

// Создание слайса с помощью литерала
s2 := []int{1, 2, 3, 4, 5}

// Создание слайса с помощью make
s3 := make([]int, 5)      // длина 5, емкость 5
s4 := make([]int, 3, 10)  // длина 3, емкость 10
```

### Операции со слайсами

```go
// Добавление элементов
s := []int{1, 2, 3}
s = append(s, 4)        // [1, 2, 3, 4]
s = append(s, 5, 6, 7)  // [1, 2, 3, 4, 5, 6, 7]

// Создание подслайса
s = []int{1, 2, 3, 4, 5}
sub1 := s[1:3]          // [2, 3]
sub2 := s[:2]           // [1, 2]
sub3 := s[2:]           // [3, 4, 5]
```

### Важные особенности слайсов

1. **Разделение базового массива**:
   ```go
   s := []int{1, 2, 3, 4, 5}
   sub := s[1:3]  // [2, 3]
   
   // Изменение sub влияет на s
   sub[0] = 99
   fmt.Println(s)  // [1, 99, 3, 4, 5]
   ```

2. **Рост емкости**:
   Когда емкость слайса исчерпана, Go создает новый базовый массив с большей емкостью:
   ```go
   s := make([]int, 0, 3)
   fmt.Println(len(s), cap(s))  // 0 3
   
   s = append(s, 1, 2, 3)
   fmt.Println(len(s), cap(s))  // 3 3
   
   s = append(s, 4)  // Требуется увеличение емкости
   fmt.Println(len(s), cap(s))  // 4 6 (емкость удвоилась)
   ```

3. **Копирование слайсов**:
   ```go
   src := []int{1, 2, 3, 4, 5}
   dst := make([]int, len(src))
   copied := copy(dst, src)  // copied = 5
   
   // Теперь dst и src имеют одинаковые значения, но разные базовые массивы
   dst[0] = 99
   fmt.Println(src)  // [1, 2, 3, 4, 5] (не изменился)
   fmt.Println(dst)  // [99, 2, 3, 4, 5]
   ```

## Карты (Maps)

Карты в Go - это реализация хеш-таблиц, которые обеспечивают быстрый поиск, вставку и удаление элементов.

### Внутренняя структура карты

Карта в Go реализована как хеш-таблица с разрешением коллизий через связанные списки. Внутренняя структура включает:
- Массив бакетов (buckets)
- Каждый бакет содержит до 8 пар ключ-значение
- При переполнении бакета создается overflow bucket

### Создание карт

```go
// Объявление карты без инициализации
var m1 map[string]int  // nil map

// Создание карты с помощью make
m2 := make(map[string]int)

// Создание карты с помощью литерала
m3 := map[string]int{
    "one": 1,
    "two": 2,
}
```

### Операции с картами

```go
// Добавление и изменение элементов
m := make(map[string]int)
m["key1"] = 10
m["key2"] = 20
m["key1"] = 30  // Перезаписывает значение

// Получение элемента
value, exists := m["key1"]
if exists {
    fmt.Println(value)  // 30
}

// Удаление элемента
delete(m, "key1")
```

### Важные особенности карт

1. **Nil карты**:
   ```go
   var m map[string]int  // nil map
   // m["key"] = 10  // Паника: assignment to entry in nil map
   
   // Проверка на nil
   if m == nil {
       m = make(map[string]int)
   }
   ```

2. **Итерация по карте**:
   ```go
   m := map[string]int{"a": 1, "b": 2, "c": 3}
   
   // Порядок итерации не гарантирован
   for key, value := range m {
       fmt.Println(key, value)
   }
   ```

3. **Карты не являются потокобезопасными**:
   ```go
   // Для конкурентного доступа используйте sync.Map или мьютексы
   var mu sync.Mutex
   m := make(map[string]int)
   
   mu.Lock()
   m["key"] = 10
   mu.Unlock()
   ```

4. **Ключи карты должны быть сравнимыми**:
   ```go
   // Допустимые типы ключей: числа, строки, булевы значения, интерфейсы, указатели, каналы, структуры и массивы с сравнимыми элементами
   
   // Недопустимые типы ключей: слайсы, карты, функции
   // var badMap map[[]int]string  // Ошибка компиляции
   ```

## Структуры (Structs)

Структуры в Go позволяют группировать данные разных типов в единую сущность.

### Определение и создание структур

```go
// Определение структуры
type Person struct {
    Name    string
    Age     int
    Address *Address  // Вложенная структура по указателю
}

type Address struct {
    City    string
    Country string
}

// Создание экземпляра структуры
p1 := Person{
    Name: "John",
    Age:  30,
    Address: &Address{
        City:    "New York",
        Country: "USA",
    },
}

// Создание с нулевыми значениями
var p2 Person

// Создание с помощью new (возвращает указатель)
p3 := new(Person)
```

### Встраивание структур (Embedding)

Go поддерживает встраивание структур, что обеспечивает композицию вместо наследования:

```go
type Employee struct {
    Person  // Встроенная структура
    Salary  float64
    Company string
}

// Создание и использование
e := Employee{
    Person: Person{
        Name: "Alice",
        Age:  28,
    },
    Salary:  50000,
    Company: "Acme Inc",
}

// Прямой доступ к полям встроенной структуры
fmt.Println(e.Name)  // "Alice" (эквивалентно e.Person.Name)
```

### Теги структур

Теги структур предоставляют метаданные для полей, которые могут использоваться библиотеками для сериализации, валидации и т.д.:

```go
type User struct {
    ID        int    `json:"id" db:"user_id"`
    Username  string `json:"username" db:"username"`
    Email     string `json:"email,omitempty" db:"email"`
    CreatedAt time.Time `json:"-" db:"created_at"`  // Игнорируется при JSON-сериализации
}

// Получение тегов во время выполнения
t := reflect.TypeOf(User{})
field, _ := t.FieldByName("Email")
fmt.Println(field.Tag.Get("json"))  // "email,omitempty"
```

## Указатели (Pointers)

Указатели в Go хранят адрес памяти переменной и позволяют изменять значения по этому адресу.

### Основы работы с указателями

```go
// Объявление указателя
var p *int

// Получение адреса переменной
x := 42
p = &x

// Разыменование указателя
fmt.Println(*p)  // 42

// Изменение значения через указатель
*p = 100
fmt.Println(x)  // 100
```

### Указатели на структуры

```go
type Person struct {
    Name string
    Age  int
}

// Создание указателя на структуру
p := &Person{Name: "John", Age: 30}

// Доступ к полям через указатель
// В Go не требуется разыменование для доступа к полям
fmt.Println(p.Name)  // "John" (эквивалентно (*p).Name)

// Изменение полей через указатель
p.Age = 31
```

### Указатели и функции

```go
// Передача по значению
func incrementValue(x int) {
    x++  // Изменяет только локальную копию
}

// Передача по указателю
func incrementPointer(x *int) {
    *x++  // Изменяет оригинальное значение
}

x := 10
incrementValue(x)
fmt.Println(x)  // 10 (не изменилось)

incrementPointer(&x)
fmt.Println(x)  // 11 (изменилось)
```

### Нулевые указатели

```go
var p *int  // nil pointer

// Проверка на nil
if p != nil {
    fmt.Println(*p)
} else {
    fmt.Println("Nil pointer")
}

// Разыменование nil указателя вызывает панику
// fmt.Println(*p)  // panic: runtime error: invalid memory address or nil pointer dereference
```

## Ссылочные типы в Go

В Go некоторые типы данных являются ссылочными по своей природе:

1. **Слайсы**
2. **Карты**
3. **Каналы**
4. **Функции**
5. **Интерфейсы**

### Особенности ссылочных типов

1. **Передача в функции**:
   При передаче ссылочных типов в функции, они передаются по ссылке, а не по значению:

   ```go
   func modifySlice(s []int) {
       s[0] = 100  // Изменяет оригинальный слайс
   }
   
   func modifyMap(m map[string]int) {
       m["key"] = 200  // Изменяет оригинальную карту
   }
   
   slice := []int{1, 2, 3}
   modifySlice(slice)
   fmt.Println(slice)  // [100, 2, 3]
   
   myMap := map[string]int{"key": 1}
   modifyMap(myMap)
   fmt.Println(myMap)  // map[key:200]
   ```

2. **Сравнение с nil**:
   Ссылочные типы могут быть nil:

   ```go
   var s []int        // nil slice
   var m map[string]int  // nil map
   var c chan int     // nil channel
   var f func()       // nil function
   var i interface{}  // nil interface
   
   fmt.Println(s == nil)  // true
   fmt.Println(m == nil)  // true
   fmt.Println(c == nil)  // true
   fmt.Println(f == nil)  // true
   fmt.Println(i == nil)  // true
   ```

3. **Сравнение между собой**:
   Не все ссылочные типы можно сравнивать между собой:

   ```go
   // Можно сравнивать с nil
   var s1, s2 []int
   // fmt.Println(s1 == s2)  // Ошибка компиляции: слайсы нельзя сравнивать
   
   // Каналы и функции можно сравнивать
   var c1, c2 chan int
   fmt.Println(c1 == c2)  // true (оба nil)
   
   // Интерфейсы можно сравнивать
   var i1, i2 interface{}
   fmt.Println(i1 == i2)  // true (оба nil)
   ```

## Интерфейсы (Interfaces)

Интерфейсы в Go определяют набор методов, которые должен реализовать тип.

### Внутренняя структура интерфейса

Интерфейс в Go состоит из двух указателей:
1. Указатель на таблицу методов (тип)
2. Указатель на конкретное значение

```go
type iface struct {
    tab  *itab          // указатель на таблицу методов
    data unsafe.Pointer // указатель на данные
}
```

### Определение и реализация интерфейсов

```go
// Определение интерфейса
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

// Композиция интерфейсов
type ReadWriter interface {
    Reader
    Writer
}

// Реализация интерфейса
type File struct {
    // ...
}

func (f *File) Read(p []byte) (n int, err error) {
    // Реализация метода Read
    return len(p), nil
}

func (f *File) Write(p []byte) (n int, err error) {
    // Реализация метода Write
    return len(p), nil
}

// Использование
var rw ReadWriter = &File{}
```

### Пустой интерфейс

Пустой интерфейс `interface{}` (или `any` в Go 1.18+) не имеет методов и может содержать значение любого типа:

```go
var i interface{}

i = 42
i = "hello"
i = []int{1, 2, 3}
i = struct{ Name string }{"John"}
```

### Утверждение типа (Type Assertion)

```go
var i interface{} = "hello"

// Утверждение типа с проверкой
if s, ok := i.(string); ok {
    fmt.Println(s)  // "hello"
} else {
    fmt.Println("not a string")
}

// Утверждение типа без проверки
s := i.(string)  // Паника, если i не строка
```

### Переключение типов (Type Switch)

```go
func describe(i interface{}) {
    switch v := i.(type) {
    case int:
        fmt.Printf("Integer: %d\n", v)
    case string:
        fmt.Printf("String: %s\n", v)
    case []int:
        fmt.Printf("Slice of ints with %d elements\n", len(v))
    default:
        fmt.Printf("Unknown type: %T\n", v)
    }
}

describe(42)        // "Integer: 42"
describe("hello")   // "String: hello"
describe([]int{1, 2, 3})  // "Slice of ints with 3 elements"
```

## Управление памятью в Go

### Стек и куча

В Go переменные могут быть размещены на стеке или в куче:

1. **Стек**:
   - Быстрое выделение и освобождение памяти
   - Ограниченный размер (обычно несколько МБ)
   - Автоматическое управление временем жизни (при выходе из функции)

2. **Куча**:
   - Более медленное выделение и освобождение памяти
   - Практически неограниченный размер
   - Управляется сборщиком мусора

### Escape Analysis

Go компилятор выполняет escape analysis, чтобы определить, где должна быть размещена переменная:

```go
func createOnStack() int {
    x := 42
    return x  // x копируется, может остаться на стеке
}

func createOnHeap() *int {
    x := 42
    return &x  // x должна "убежать" на кучу, так как её адрес возвращается
}
```

### Сборка мусора

Go использует параллельный сборщик мусора с тремя фазами:
1. **Mark**: Поиск и маркировка достижимых объектов
2. **Sweep**: Освобождение памяти недостижимых объектов
3. **Compact**: (в некоторых версиях) Уплотнение памяти для уменьшения фрагментации

### Оптимизация использования памяти

1. **Повторное использование слайсов**:
   ```go
   // Вместо создания нового слайса при каждом вызове
   func process() []int {
       result := make([]int, 0, 10)
       // Заполнение result
       return result
   }
   
   // Можно переиспользовать существующий слайс
   func process(result []int) []int {
       result = result[:0]  // Очистка слайса без выделения новой памяти
       // Заполнение result
       return result
   }
   ```

2. **Предварительное выделение памяти**:
   ```go
   // Если известен примерный размер
   s := make([]int, 0, expectedSize)
   for i := 0; i < n; i++ {
       s = append(s, i)
   }
   ```

3. **Использование sync.Pool для временных объектов**:
   ```go
   var bufferPool = sync.Pool{
       New: func() interface{} {
           return new(bytes.Buffer)
       },
   }
   
   func process() {
       buf := bufferPool.Get().(*bytes.Buffer)
       defer bufferPool.Put(buf)
       
       buf.Reset()  // Очистка буфера перед использованием
       // Использование buf
   }
   ```

## Практические задания

1. Напишите функцию, которая объединяет два слайса без дублирования элементов.
2. Реализуйте структуру Cache с методами Get, Set и Delete, используя карту и мьютекс для потокобезопасности.
3. Создайте интерфейс Storage с методами Save и Load, и реализуйте его для разных типов хранилищ (память, файл).
4. Напишите функцию, которая принимает интерфейс и возвращает глубокую копию содержащегося в нем значения.
5. Реализуйте пул объектов с помощью sync.Pool для повторного использования буферов.
