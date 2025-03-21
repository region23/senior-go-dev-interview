# Шпаргалка по алгоритмам и структурам данных

## Сложность алгоритмов

### Временная сложность (Big O)
- **O(1)** - константное время (хеш-таблицы, доступ к элементу массива)
- **O(log n)** - логарифмическое время (бинарный поиск, сбалансированные деревья)
- **O(n)** - линейное время (перебор массива)
- **O(n log n)** - линеарифмическое время (эффективные алгоритмы сортировки)
- **O(n²)** - квадратичное время (вложенные циклы, простые сортировки)
- **O(2^n)** - экспоненциальное время (рекурсивный перебор)
- **O(n!)** - факториальное время (перестановки)

### Пространственная сложность
- Дополнительная память, используемая алгоритмом
- Рекурсия использует стек вызовов (O(n) для глубины n)

## Базовые структуры данных

### Массивы
- Фиксированный размер, непрерывная память
- Доступ по индексу: O(1)
- Поиск: O(n)
- Вставка/удаление: O(n) из-за сдвига элементов

### Динамические массивы (слайсы в Go)
- Изменяемый размер, автоматическое выделение памяти
- Доступ по индексу: O(1)
- Добавление в конец: O(1) амортизированное, O(n) в худшем случае
- Вставка/удаление в середине: O(n)

### Связные списки
- Односвязный список: каждый узел хранит данные и указатель на следующий узел
- Двусвязный список: каждый узел хранит данные и указатели на следующий и предыдущий узлы
- Доступ к элементу: O(n)
- Вставка/удаление в начале: O(1)
- Вставка/удаление в конце: O(1) для двусвязного, O(n) для односвязного
- Вставка/удаление в середине: O(n) для поиска + O(1) для операции

### Стек
- LIFO (Last In, First Out)
- Push (добавление): O(1)
- Pop (удаление): O(1)
- Peek (просмотр верхнего элемента): O(1)
- Применение: отмена операций, проверка скобок, обход в глубину

```go
// Реализация стека на слайсе
type Stack []interface{}

func (s *Stack) Push(v interface{}) {
    *s = append(*s, v)
}

func (s *Stack) Pop() interface{} {
    if s.IsEmpty() {
        return nil
    }
    index := len(*s) - 1
    element := (*s)[index]
    *s = (*s)[:index]
    return element
}

func (s *Stack) IsEmpty() bool {
    return len(*s) == 0
}
```

### Очередь
- FIFO (First In, First Out)
- Enqueue (добавление): O(1)
- Dequeue (удаление): O(1)
- Peek (просмотр первого элемента): O(1)
- Применение: обработка задач, обход в ширину

```go
// Реализация очереди на слайсе
type Queue []interface{}

func (q *Queue) Enqueue(v interface{}) {
    *q = append(*q, v)
}

func (q *Queue) Dequeue() interface{} {
    if q.IsEmpty() {
        return nil
    }
    element := (*q)[0]
    *q = (*q)[1:]
    return element
}

func (q *Queue) IsEmpty() bool {
    return len(*q) == 0
}
```

### Хеш-таблица (map в Go)
- Хранит пары ключ-значение
- Доступ, вставка, удаление: O(1) в среднем, O(n) в худшем случае
- Основана на хеш-функции
- Коллизии разрешаются цепочками или открытой адресацией

### Множество (set)
- Хранит уникальные элементы
- Проверка наличия, вставка, удаление: O(1) в среднем
- В Go реализуется через map с пустыми значениями

```go
type Set map[interface{}]struct{}

func (s Set) Add(item interface{}) {
    s[item] = struct{}{}
}

func (s Set) Contains(item interface{}) bool {
    _, exists := s[item]
    return exists
}

func (s Set) Remove(item interface{}) {
    delete(s, item)
}
```

## Деревья

### Бинарное дерево
- Каждый узел имеет не более двух потомков
- Высота сбалансированного дерева: O(log n)
- Высота несбалансированного дерева: до O(n)

### Бинарное дерево поиска (BST)
- Для каждого узла: все элементы в левом поддереве меньше, в правом - больше
- Поиск, вставка, удаление: O(log n) для сбалансированного, O(n) для несбалансированного
- Обход: инфиксный (in-order), префиксный (pre-order), постфиксный (post-order)

```go
type TreeNode struct {
    Val   int
    Left  *TreeNode
    Right *TreeNode
}

// Поиск в BST
func search(root *TreeNode, val int) *TreeNode {
    if root == nil || root.Val == val {
        return root
    }
    if val < root.Val {
        return search(root.Left, val)
    }
    return search(root.Right, val)
}

// Вставка в BST
func insert(root *TreeNode, val int) *TreeNode {
    if root == nil {
        return &TreeNode{Val: val}
    }
    if val < root.Val {
        root.Left = insert(root.Left, val)
    } else {
        root.Right = insert(root.Right, val)
    }
    return root
}
```

### Сбалансированные деревья
- **AVL-дерево**: разница высот левого и правого поддеревьев не более 1
- **Красно-черное дерево**: каждый узел красный или черный, с определенными правилами
- **B-дерево**: обобщение BST, используется в базах данных и файловых системах

### Куча (Heap)
- Бинарное дерево, где значение в каждом узле не меньше (max-heap) или не больше (min-heap) значений его потомков
- Извлечение максимума/минимума: O(1)
- Вставка, удаление: O(log n)
- Построение кучи из массива: O(n)
- Применение: сортировка кучей, приоритетная очередь

```go
// Реализация min-heap
type MinHeap []int

func (h MinHeap) Len() int           { return len(h) }
func (h MinHeap) Less(i, j int) bool { return h[i] < h[j] }
func (h MinHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }

func (h *MinHeap) Push(x interface{}) {
    *h = append(*h, x.(int))
}

func (h *MinHeap) Pop() interface{} {
    old := *h
    n := len(old)
    x := old[n-1]
    *h = old[0 : n-1]
    return x
}
```

### Trie (префиксное дерево)
- Используется для хранения строк
- Поиск, вставка, удаление: O(m), где m - длина строки
- Применение: автодополнение, проверка орфографии, поиск по префиксу

```go
type TrieNode struct {
    children map[rune]*TrieNode
    isEnd    bool
}

type Trie struct {
    root *TrieNode
}

func NewTrie() *Trie {
    return &Trie{
        root: &TrieNode{
            children: make(map[rune]*TrieNode),
        },
    }
}

func (t *Trie) Insert(word string) {
    node := t.root
    for _, ch := range word {
        if _, exists := node.children[ch]; !exists {
            node.children[ch] = &TrieNode{
                children: make(map[rune]*TrieNode),
            }
        }
        node = node.children[ch]
    }
    node.isEnd = true
}

func (t *Trie) Search(word string) bool {
    node := t.root
    for _, ch := range word {
        if _, exists := node.children[ch]; !exists {
            return false
        }
        node = node.children[ch]
    }
    return node.isEnd
}

func (t *Trie) StartsWith(prefix string) bool {
    node := t.root
    for _, ch := range prefix {
        if _, exists := node.children[ch]; !exists {
            return false
        }
        node = node.children[ch]
    }
    return true
}
```

## Графы

### Представление графов
- **Матрица смежности**: двумерный массив размера V×V
  - Память: O(V²)
  - Проверка смежности: O(1)
  - Перебор соседей: O(V)
- **Список смежности**: массив списков
  - Память: O(V+E)
  - Проверка смежности: O(degree(v))
  - Перебор соседей: O(degree(v))

```go
// Матрица смежности
type GraphMatrix struct {
    vertices int
    matrix   [][]bool
}

// Список смежности
type GraphList struct {
    vertices int
    adj      [][]int
}
```

### Обход графа
- **Поиск в глубину (DFS)**:
  - Использует стек (явно или через рекурсию)
  - Время: O(V+E)
  - Применение: топологическая сортировка, поиск компонент связности

```go
func DFS(graph [][]int, start int) {
    visited := make([]bool, len(graph))
    dfsUtil(graph, start, visited)
}

func dfsUtil(graph [][]int, vertex int, visited []bool) {
    visited[vertex] = true
    fmt.Println(vertex)
    
    for _, neighbor := range graph[vertex] {
        if !visited[neighbor] {
            dfsUtil(graph, neighbor, visited)
        }
    }
}
```

- **Поиск в ширину (BFS)**:
  - Использует очередь
  - Время: O(V+E)
  - Применение: кратчайший путь в невзвешенном графе, проверка двудольности

```go
func BFS(graph [][]int, start int) {
    visited := make([]bool, len(graph))
    queue := []int{start}
    visited[start] = true
    
    for len(queue) > 0 {
        vertex := queue[0]
        queue = queue[1:]
        fmt.Println(vertex)
        
        for _, neighbor := range graph[vertex] {
            if !visited[neighbor] {
                visited[neighbor] = true
                queue = append(queue, neighbor)
            }
        }
    }
}
```

### Алгоритмы для графов
- **Алгоритм Дейкстры**: кратчайший путь от одной вершины до всех остальных во взвешенном графе без отрицательных весов
  - Время: O(E log V) с бинарной кучей
  - Применение: маршрутизация, GPS-навигация

```go
func Dijkstra(graph [][]Edge, start int) []int {
    n := len(graph)
    dist := make([]int, n)
    for i := range dist {
        dist[i] = math.MaxInt32
    }
    dist[start] = 0
    
    pq := &PriorityQueue{}
    heap.Init(pq)
    heap.Push(pq, &Item{value: start, priority: 0})
    
    for pq.Len() > 0 {
        item := heap.Pop(pq).(*Item)
        u := item.value
        
        if item.priority > dist[u] {
            continue
        }
        
        for _, edge := range graph[u] {
            v := edge.to
            weight := edge.weight
            
            if dist[u] + weight < dist[v] {
                dist[v] = dist[u] + weight
                heap.Push(pq, &Item{value: v, priority: dist[v]})
            }
        }
    }
    
    return dist
}
```

- **Алгоритм Беллмана-Форда**: кратчайший путь от одной вершины до всех остальных во взвешенном графе, включая отрицательные веса
  - Время: O(V*E)
  - Обнаруживает отрицательные циклы

- **Алгоритм Флойда-Уоршелла**: кратчайшие пути между всеми парами вершин
  - Время: O(V³)
  - Применение: анализ сетей, транзитивное замыкание

- **Алгоритм Краскала**: построение минимального остовного дерева
  - Время: O(E log E) или O(E log V)
  - Использует структуру данных "Система непересекающихся множеств" (Union-Find)

- **Алгоритм Прима**: построение минимального остовного дерева
  - Время: O(E log V) с бинарной кучей
  - Альтернатива алгоритму Краскала

- **Топологическая сортировка**: линейное упорядочивание вершин ориентированного ациклического графа (DAG)
  - Время: O(V+E)
  - Применение: планирование задач, зависимости в сборке

```go
func TopologicalSort(graph [][]int) []int {
    n := len(graph)
    visited := make([]bool, n)
    temp := make([]bool, n)
    order := []int{}
    
    var dfs func(int) bool
    dfs = func(node int) bool {
        if temp[node] {
            return false // Обнаружен цикл
        }
        if visited[node] {
            return true
        }
        
        temp[node] = true
        
        for _, neighbor := range graph[node] {
            if !dfs(neighbor) {
                return false
            }
        }
        
        temp[node] = false
        visited[node] = true
        order = append([]int{node}, order...) // Добавляем в начало
        
        return true
    }
    
    for i := 0; i < n; i++ {
        if !visited[i] {
            if !dfs(i) {
                return nil // Граф содержит цикл
            }
        }
    }
    
    return order
}
```

## Алгоритмы сортировки

### Простые алгоритмы сортировки
- **Сортировка пузырьком (Bubble Sort)**:
  - Время: O(n²) в среднем и худшем случае
  - Память: O(1)
  - Стабильная

```go
func bubbleSort(arr []int) {
    n := len(arr)
    for i := 0; i < n; i++ {
        swapped := false
        for j := 0; j < n-i-1; j++ {
            if arr[j] > arr[j+1] {
                arr[j], arr[j+1] = arr[j+1], arr[j]
                swapped = true
            }
        }
        if !swapped {
            break
        }
    }
}
```

- **Сортировка вставками (Insertion Sort)**:
  - Время: O(n²) в среднем и худшем случае, O(n) в лучшем случае
  - Память: O(1)
  - Стабильная
  - Эффективна для почти отсортированных массивов

```go
func insertionSort(arr []int) {
    for i := 1; i < len(arr); i++ {
        key := arr[i]
        j := i - 1
        for j >= 0 && arr[j] > key {
            arr[j+1] = arr[j]
            j--
        }
        arr[j+1] = key
    }
}
```

- **Сортировка выбором (Selection Sort)**:
  - Время: O(n²) во всех случаях
  - Память: O(1)
  - Нестабильная
  - Минимальное количество обменов

```go
func selectionSort(arr []int) {
    n := len(arr)
    for i := 0; i < n-1; i++ {
        minIdx := i
        for j := i + 1; j < n; j++ {
            if arr[j] < arr[minIdx] {
                minIdx = j
            }
        }
        arr[i], arr[minIdx] = arr[minIdx], arr[i]
    }
}
```

### Эффективные алгоритмы сортировки
- **Быстрая сортировка (Quick Sort)**:
  - Время: O(n log n) в среднем, O(n²) в худшем случае
  - Память: O(log n) для стека вызовов
  - Нестабильная
  - Обычно самая быстрая на практике

```go
func quickSort(arr []int, low, high int) {
    if low < high {
        pivot := partition(arr, low, high)
        quickSort(arr, low, pivot-1)
        quickSort(arr, pivot+1, high)
    }
}

func partition(arr []int, low, high int) int {
    pivot := arr[high]
    i := low - 1
    
    for j := low; j < high; j++ {
        if arr[j] <= pivot {
            i++
            arr[i], arr[j] = arr[j], arr[i]
        }
    }
    
    arr[i+1], arr[high] = arr[high], arr[i+1]
    return i + 1
}
```

- **Сортировка слиянием (Merge Sort)**:
  - Время: O(n log n) во всех случаях
  - Память: O(n)
  - Стабильная
  - Хорошо подходит для связных списков

```go
func mergeSort(arr []int) []int {
    if len(arr) <= 1 {
        return arr
    }
    
    mid := len(arr) / 2
    left := mergeSort(arr[:mid])
    right := mergeSort(arr[mid:])
    
    return merge(left, right)
}

func merge(left, right []int) []int {
    result := make([]int, 0, len(left)+len(right))
    i, j := 0, 0
    
    for i < len(left) && j < len(right) {
        if left[i] <= right[j] {
            result = append(result, left[i])
            i++
        } else {
            result = append(result, right[j])
            j++
        }
    }
    
    result = append(result, left[i:]...)
    result = append(result, right[j:]...)
    
    return result
}
```

- **Сортировка кучей (Heap Sort)**:
  - Время: O(n log n) во всех случаях
  - Память: O(1)
  - Нестабильная
  - Гарантированная производительность

```go
func heapSort(arr []int) {
    n := len(arr)
    
    // Построение кучи (перегруппировка массива)
    for i := n/2 - 1; i >= 0; i-- {
        heapify(arr, n, i)
    }
    
    // Извлечение элементов из кучи по одному
    for i := n - 1; i > 0; i-- {
        arr[0], arr[i] = arr[i], arr[0]
        heapify(arr, i, 0)
    }
}

func heapify(arr []int, n, i int) {
    largest := i
    left := 2*i + 1
    right := 2*i + 2
    
    if left < n && arr[left] > arr[largest] {
        largest = left
    }
    
    if right < n && arr[right] > arr[largest] {
        largest = right
    }
    
    if largest != i {
        arr[i], arr[largest] = arr[largest], arr[i]
        heapify(arr, n, largest)
    }
}
```

## Алгоритмы поиска

### Линейный поиск
- Время: O(n)
- Применение: неотсортированные массивы

```go
func linearSearch(arr []int, target int) int {
    for i, val := range arr {
        if val == target {
            return i
        }
    }
    return -1
}
```

### Бинарный поиск
- Время: O(log n)
- Требует отсортированного массива

```go
func binarySearch(arr []int, target int) int {
    left, right := 0, len(arr)-1
    
    for left <= right {
        mid := left + (right-left)/2
        
        if arr[mid] == target {
            return mid
        }
        
        if arr[mid] < target {
            left = mid + 1
        } else {
            right = mid - 1
        }
    }
    
    return -1
}
```

### Интерполяционный поиск
- Время: O(log log n) в среднем, O(n) в худшем случае
- Для равномерно распределенных данных

```go
func interpolationSearch(arr []int, target int) int {
    low, high := 0, len(arr)-1
    
    for low <= high && target >= arr[low] && target <= arr[high] {
        if low == high {
            if arr[low] == target {
                return low
            }
            return -1
        }
        
        pos := low + ((target-arr[low])*(high-low))/(arr[high]-arr[low])
        
        if arr[pos] == target {
            return pos
        }
        
        if arr[pos] < target {
            low = pos + 1
        } else {
            high = pos - 1
        }
    }
    
    return -1
}
```

## Алгоритмические техники

### Рекурсия
- Функция вызывает сама себя
- Требует базовый случай для завершения
- Использует стек вызовов

```go
func factorial(n int) int {
    if n <= 1 {
        return 1
    }
    return n * factorial(n-1)
}
```

### Динамическое программирование
- Разбиение задачи на подзадачи
- Сохранение результатов подзадач для повторного использования
- Применение: задачи оптимизации с перекрывающимися подзадачами

```go
// Числа Фибоначчи с мемоизацией
func fibonacci(n int, memo map[int]int) int {
    if val, exists := memo[n]; exists {
        return val
    }
    
    if n <= 1 {
        return n
    }
    
    memo[n] = fibonacci(n-1, memo) + fibonacci(n-2, memo)
    return memo[n]
}

// Числа Фибоначчи с табуляцией
func fibonacciDP(n int) int {
    if n <= 1 {
        return n
    }
    
    dp := make([]int, n+1)
    dp[0], dp[1] = 0, 1
    
    for i := 2; i <= n; i++ {
        dp[i] = dp[i-1] + dp[i-2]
    }
    
    return dp[n]
}
```

### Жадные алгоритмы
- Выбор локально оптимального решения на каждом шаге
- Не всегда приводит к глобально оптимальному решению
- Применение: задачи планирования, кодирование Хаффмана

```go
// Задача о выборе активностей
func activitySelection(start, finish []int) []int {
    n := len(start)
    if n == 0 {
        return []int{}
    }
    
    // Сортировка по времени окончания
    activities := make([][2]int, n)
    for i := 0; i < n; i++ {
        activities[i] = [2]int{start[i], finish[i]}
    }
    sort.Slice(activities, func(i, j int) bool {
        return activities[i][1] < activities[j][1]
    })
    
    selected := []int{0} // Выбираем первую активность
    lastFinish := activities[0][1]
    
    for i := 1; i < n; i++ {
        if activities[i][0] >= lastFinish {
            selected = append(selected, i)
            lastFinish = activities[i][1]
        }
    }
    
    return selected
}
```

### Разделяй и властвуй
- Разбиение задачи на подзадачи
- Решение подзадач рекурсивно
- Объединение результатов
- Применение: сортировка слиянием, быстрая сортировка, бинарный поиск

```go
// Поиск максимального подмассива (алгоритм Кадане)
func maxSubArray(nums []int) int {
    maxSoFar := nums[0]
    maxEndingHere := nums[0]
    
    for i := 1; i < len(nums); i++ {
        maxEndingHere = max(nums[i], maxEndingHere+nums[i])
        maxSoFar = max(maxSoFar, maxEndingHere)
    }
    
    return maxSoFar
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

### Поиск с возвратом (Backtracking)
- Перебор с отсечением неперспективных ветвей
- Применение: комбинаторные задачи, головоломки

```go
// Генерация всех перестановок
func permute(nums []int) [][]int {
    result := [][]int{}
    backtrack(nums, 0, &result)
    return result
}

func backtrack(nums []int, start int, result *[][]int) {
    if start == len(nums) {
        temp := make([]int, len(nums))
        copy(temp, nums)
        *result = append(*result, temp)
        return
    }
    
    for i := start; i < len(nums); i++ {
        nums[start], nums[i] = nums[i], nums[start]
        backtrack(nums, start+1, result)
        nums[start], nums[i] = nums[i], nums[start] // Возврат
    }
}
```

## Типичные задачи на собеседованиях

### Задачи на массивы и строки
- Поиск подмассива с максимальной суммой
- Поиск двух чисел с заданной суммой
- Поиск дубликатов
- Поиск пропущенного числа
- Поиск наибольшего/наименьшего элемента
- Поиск подстроки
- Проверка анаграмм
- Проверка палиндрома

### Задачи на связные списки
- Обращение списка
- Обнаружение цикла
- Нахождение середины списка
- Слияние отсортированных списков
- Удаление дубликатов
- Удаление n-го элемента с конца

### Задачи на стеки и очереди
- Проверка сбалансированности скобок
- Реализация очереди с помощью стеков
- Реализация стека с минимумом
- Сортировка стека

### Задачи на деревья
- Обход дерева (инфиксный, префиксный, постфиксный)
- Проверка, является ли дерево BST
- Нахождение наименьшего общего предка
- Определение высоты дерева
- Проверка сбалансированности дерева
- Зеркальное отражение дерева

### Задачи на графы
- Поиск пути в лабиринте
- Определение наличия цикла
- Топологическая сортировка
- Поиск кратчайшего пути
- Поиск компонент связности

### Задачи на динамическое программирование
- Числа Фибоначчи
- Задача о рюкзаке
- Наибольшая общая подпоследовательность
- Наибольшая возрастающая подпоследовательность
- Редакционное расстояние (расстояние Левенштейна)
- Задача о монетах (размен)

## Советы по решению алгоритмических задач

1. **Понимание задачи**:
   - Внимательно прочитайте условие
   - Уточните неясные моменты
   - Определите входные и выходные данные
   - Рассмотрите граничные случаи

2. **Разработка алгоритма**:
   - Начните с простого решения (даже если оно неэффективно)
   - Подумайте о возможных оптимизациях
   - Рассмотрите применение известных алгоритмических техник
   - Проговаривайте свои мысли вслух

3. **Реализация**:
   - Пишите чистый, понятный код
   - Используйте осмысленные имена переменных
   - Разбивайте сложную логику на функции
   - Комментируйте неочевидные части

4. **Тестирование**:
   - Проверьте на простых примерах
   - Рассмотрите граничные случаи (пустой массив, один элемент и т.д.)
   - Проверьте на специальных случаях (отрицательные числа, дубликаты и т.д.)
   - Анализируйте ошибки и исправляйте их

5. **Анализ сложности**:
   - Определите временную сложность
   - Определите пространственную сложность
   - Обсудите возможные компромиссы между временем и памятью

## Шаблоны решения типичных задач

### Двухуказательная техника
```go
// Поиск двух чисел с заданной суммой
func twoSum(nums []int, target int) []int {
    // Сортировка массива
    sorted := make([]int, len(nums))
    copy(sorted, nums)
    sort.Ints(sorted)
    
    // Два указателя
    left, right := 0, len(sorted)-1
    
    for left < right {
        sum := sorted[left] + sorted[right]
        
        if sum == target {
            // Найти индексы в исходном массиве
            for i, num := range nums {
                if num == sorted[left] {
                    for j, num := range nums {
                        if j != i && num == sorted[right] {
                            return []int{i, j}
                        }
                    }
                }
            }
        }
        
        if sum < target {
            left++
        } else {
            right--
        }
    }
    
    return []int{}
}
```

### Скользящее окно
```go
// Максимальная сумма подмассива длины k
func maxSumSubarray(nums []int, k int) int {
    n := len(nums)
    if n < k {
        return 0
    }
    
    // Сумма первых k элементов
    windowSum := 0
    for i := 0; i < k; i++ {
        windowSum += nums[i]
    }
    
    maxSum := windowSum
    
    // Скользящее окно
    for i := k; i < n; i++ {
        windowSum = windowSum - nums[i-k] + nums[i]
        maxSum = max(maxSum, windowSum)
    }
    
    return maxSum
}
```

### Быстрое и медленное указатели
```go
// Обнаружение цикла в связном списке
func hasCycle(head *ListNode) bool {
    if head == nil || head.Next == nil {
        return false
    }
    
    slow, fast := head, head
    
    for fast != nil && fast.Next != nil {
        slow = slow.Next
        fast = fast.Next.Next
        
        if slow == fast {
            return true
        }
    }
    
    return false
}
```

### Префиксная сумма
```go
// Подмассив с заданной суммой
func subarraySum(nums []int, k int) int {
    count := 0
    sum := 0
    prefixSums := map[int]int{0: 1}
    
    for _, num := range nums {
        sum += num
        if count, exists := prefixSums[sum-k]; exists {
            count += count
        }
        prefixSums[sum]++
    }
    
    return count
}
```

### Хеш-таблица для O(1) поиска
```go
// Поиск дубликатов
func containsDuplicate(nums []int) bool {
    seen := make(map[int]bool)
    
    for _, num := range nums {
        if seen[num] {
            return true
        }
        seen[num] = true
    }
    
    return false
}
```

### Обход в глубину (DFS) для деревьев
```go
// Максимальная глубина бинарного дерева
func maxDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    
    leftDepth := maxDepth(root.Left)
    rightDepth := maxDepth(root.Right)
    
    return max(leftDepth, rightDepth) + 1
}
```

### Обход в ширину (BFS) для деревьев
```go
// Уровневый обход бинарного дерева
func levelOrder(root *TreeNode) [][]int {
    result := [][]int{}
    if root == nil {
        return result
    }
    
    queue := []*TreeNode{root}
    
    for len(queue) > 0 {
        levelSize := len(queue)
        level := []int{}
        
        for i := 0; i < levelSize; i++ {
            node := queue[0]
            queue = queue[1:]
            
            level = append(level, node.Val)
            
            if node.Left != nil {
                queue = append(queue, node.Left)
            }
            if node.Right != nil {
                queue = append(queue, node.Right)
            }
        }
        
        result = append(result, level)
    }
    
    return result
}
```

### Динамическое программирование с мемоизацией
```go
// Задача о монетах (размен)
func coinChange(coins []int, amount int) int {
    memo := make(map[int]int)
    
    var dp func(int) int
    dp = func(remaining int) int {
        if remaining == 0 {
            return 0
        }
        if remaining < 0 {
            return -1
        }
        
        if count, exists := memo[remaining]; exists {
            return count
        }
        
        minCoins := math.MaxInt32
        
        for _, coin := range coins {
            res := dp(remaining - coin)
            if res != -1 {
                minCoins = min(minCoins, res + 1)
            }
        }
        
        if minCoins == math.MaxInt32 {
            memo[remaining] = -1
            return -1
        }
        
        memo[remaining] = minCoins
        return minCoins
    }
    
    return dp(amount)
}
```

### Динамическое программирование с табуляцией
```go
// Задача о монетах (размен)
func coinChangeDP(coins []int, amount int) int {
    dp := make([]int, amount+1)
    for i := range dp {
        dp[i] = amount + 1
    }
    dp[0] = 0
    
    for i := 1; i <= amount; i++ {
        for _, coin := range coins {
            if coin <= i {
                dp[i] = min(dp[i], dp[i-coin]+1)
            }
        }
    }
    
    if dp[amount] > amount {
        return -1
    }
    return dp[amount]
}
```
