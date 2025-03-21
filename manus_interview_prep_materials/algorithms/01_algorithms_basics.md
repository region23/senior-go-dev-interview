# Алгоритмы и структуры данных для технического интервью

## Введение

Алгоритмы и структуры данных - одна из ключевых тем на технических интервью в компаниях уровня Яндекс и Озон. В этом руководстве мы рассмотрим основные структуры данных, алгоритмы и подходы к решению алгоритмических задач, которые часто встречаются на собеседованиях.

## Основные структуры данных

### Массивы и слайсы

**Массивы** в Go имеют фиксированную длину, определяемую при создании:

```go
// Объявление массива
var arr [5]int                // Массив из 5 целых чисел, инициализированных нулями
arr2 := [3]int{1, 2, 3}       // Массив с инициализацией
arr3 := [...]int{1, 2, 3, 4}  // Размер определяется компилятором (4)
```

**Слайсы** - это динамические массивы, которые могут изменять размер:

```go
// Объявление слайса
var slice []int               // Пустой слайс (nil)
slice = make([]int, 5)        // Слайс длиной 5, емкостью 5
slice = make([]int, 3, 10)    // Слайс длиной 3, емкостью 10
slice = []int{1, 2, 3}        // Слайс с инициализацией

// Операции со слайсами
slice = append(slice, 4, 5)   // Добавление элементов
slice2 := slice[1:3]          // Создание подслайса (элементы 1 и 2)
```

**Сложность операций**:
- Доступ по индексу: O(1)
- Поиск элемента: O(n)
- Добавление в конец (амортизированно): O(1)
- Добавление в начало/середину: O(n)
- Удаление из начала/середины: O(n)

### Связные списки

**Односвязный список** - каждый узел содержит данные и указатель на следующий узел:

```go
type Node struct {
    Value int
    Next  *Node
}

type LinkedList struct {
    Head *Node
}

// Добавление в начало списка
func (l *LinkedList) Prepend(value int) {
    newNode := &Node{Value: value, Next: l.Head}
    l.Head = newNode
}

// Добавление в конец списка
func (l *LinkedList) Append(value int) {
    newNode := &Node{Value: value}
    
    if l.Head == nil {
        l.Head = newNode
        return
    }
    
    current := l.Head
    for current.Next != nil {
        current = current.Next
    }
    
    current.Next = newNode
}
```

**Двусвязный список** - каждый узел содержит данные и указатели на предыдущий и следующий узлы:

```go
type Node struct {
    Value int
    Prev  *Node
    Next  *Node
}

type DoublyLinkedList struct {
    Head *Node
    Tail *Node
}
```

**Сложность операций**:
- Доступ по индексу: O(n)
- Поиск элемента: O(n)
- Добавление в начало/конец (при наличии указателя на хвост): O(1)
- Добавление в середину (при наличии указателя): O(1)
- Удаление (при наличии указателя): O(1)

### Стек и очередь

**Стек** - структура данных LIFO (Last In, First Out):

```go
type Stack struct {
    items []int
}

func (s *Stack) Push(item int) {
    s.items = append(s.items, item)
}

func (s *Stack) Pop() (int, error) {
    if len(s.items) == 0 {
        return 0, errors.New("stack is empty")
    }
    
    n := len(s.items) - 1
    item := s.items[n]
    s.items = s.items[:n]
    return item, nil
}

func (s *Stack) Peek() (int, error) {
    if len(s.items) == 0 {
        return 0, errors.New("stack is empty")
    }
    
    return s.items[len(s.items)-1], nil
}
```

**Очередь** - структура данных FIFO (First In, First Out):

```go
type Queue struct {
    items []int
}

func (q *Queue) Enqueue(item int) {
    q.items = append(q.items, item)
}

func (q *Queue) Dequeue() (int, error) {
    if len(q.items) == 0 {
        return 0, errors.New("queue is empty")
    }
    
    item := q.items[0]
    q.items = q.items[1:]
    return item, nil
}

func (q *Queue) Peek() (int, error) {
    if len(q.items) == 0 {
        return 0, errors.New("queue is empty")
    }
    
    return q.items[0], nil
}
```

**Сложность операций**:
- Стек: Push - O(1), Pop - O(1), Peek - O(1)
- Очередь (на основе слайса): Enqueue - O(1), Dequeue - O(n)
- Очередь (на основе связного списка): Enqueue - O(1), Dequeue - O(1)

### Хеш-таблицы (карты)

В Go хеш-таблицы реализованы как встроенный тип `map`:

```go
// Объявление карты
var m map[string]int          // Пустая карта (nil)
m = make(map[string]int)      // Инициализация карты
m = map[string]int{           // Карта с инициализацией
    "one": 1,
    "two": 2,
}

// Операции с картой
m["three"] = 3                // Добавление/изменение элемента
value, exists := m["two"]     // Проверка наличия ключа
delete(m, "one")              // Удаление элемента
```

**Сложность операций** (в среднем):
- Доступ/вставка/удаление: O(1)
- В худшем случае: O(n)

### Деревья

**Бинарное дерево поиска** - каждый узел имеет не более двух потомков, при этом все узлы в левом поддереве меньше корня, а все узлы в правом поддереве больше корня:

```go
type TreeNode struct {
    Value int
    Left  *TreeNode
    Right *TreeNode
}

type BinarySearchTree struct {
    Root *TreeNode
}

// Вставка элемента
func (bst *BinarySearchTree) Insert(value int) {
    if bst.Root == nil {
        bst.Root = &TreeNode{Value: value}
        return
    }
    
    insertNode(bst.Root, value)
}

func insertNode(node *TreeNode, value int) {
    if value < node.Value {
        if node.Left == nil {
            node.Left = &TreeNode{Value: value}
        } else {
            insertNode(node.Left, value)
        }
    } else {
        if node.Right == nil {
            node.Right = &TreeNode{Value: value}
        } else {
            insertNode(node.Right, value)
        }
    }
}

// Поиск элемента
func (bst *BinarySearchTree) Search(value int) bool {
    return searchNode(bst.Root, value)
}

func searchNode(node *TreeNode, value int) bool {
    if node == nil {
        return false
    }
    
    if value == node.Value {
        return true
    }
    
    if value < node.Value {
        return searchNode(node.Left, value)
    }
    
    return searchNode(node.Right, value)
}
```

**Сложность операций** (для сбалансированного дерева):
- Поиск/вставка/удаление: O(log n)
- В худшем случае (несбалансированное дерево): O(n)

**Сбалансированные деревья** (AVL, красно-черные) поддерживают баланс, обеспечивая логарифмическую сложность операций.

### Кучи (Heap)

**Куча** - это бинарное дерево, в котором значение каждого узла не меньше (max-heap) или не больше (min-heap) значений его потомков:

```go
// В Go можно использовать пакет container/heap
import "container/heap"

// Реализация min-heap
type IntHeap []int

func (h IntHeap) Len() int           { return len(h) }
func (h IntHeap) Less(i, j int) bool { return h[i] < h[j] }
func (h IntHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }

func (h *IntHeap) Push(x interface{}) {
    *h = append(*h, x.(int))
}

func (h *IntHeap) Pop() interface{} {
    old := *h
    n := len(old)
    x := old[n-1]
    *h = old[0 : n-1]
    return x
}

// Использование
h := &IntHeap{2, 1, 5}
heap.Init(h)
heap.Push(h, 3)
fmt.Printf("minimum: %d\n", (*h)[0])
for h.Len() > 0 {
    fmt.Printf("%d ", heap.Pop(h))
}
```

**Сложность операций**:
- Вставка: O(log n)
- Извлечение минимума/максимума: O(log n)
- Просмотр минимума/максимума: O(1)
- Построение кучи из массива: O(n)

### Графы

**Граф** - это набор вершин и ребер, соединяющих эти вершины:

```go
// Представление графа с помощью списка смежности
type Graph struct {
    Vertices int
    AdjList  map[int][]int
}

func NewGraph(vertices int) *Graph {
    return &Graph{
        Vertices: vertices,
        AdjList:  make(map[int][]int),
    }
}

// Добавление ребра
func (g *Graph) AddEdge(src, dest int) {
    g.AdjList[src] = append(g.AdjList[src], dest)
    g.AdjList[dest] = append(g.AdjList[dest], src) // Для неориентированного графа
}

// Поиск в ширину (BFS)
func (g *Graph) BFS(start int) []int {
    visited := make(map[int]bool)
    queue := []int{start}
    visited[start] = true
    result := []int{}
    
    for len(queue) > 0 {
        vertex := queue[0]
        queue = queue[1:]
        result = append(result, vertex)
        
        for _, neighbor := range g.AdjList[vertex] {
            if !visited[neighbor] {
                visited[neighbor] = true
                queue = append(queue, neighbor)
            }
        }
    }
    
    return result
}

// Поиск в глубину (DFS)
func (g *Graph) DFS(start int) []int {
    visited := make(map[int]bool)
    result := []int{}
    
    var dfsUtil func(vertex int)
    dfsUtil = func(vertex int) {
        visited[vertex] = true
        result = append(result, vertex)
        
        for _, neighbor := range g.AdjList[vertex] {
            if !visited[neighbor] {
                dfsUtil(neighbor)
            }
        }
    }
    
    dfsUtil(start)
    return result
}
```

**Сложность операций**:
- BFS/DFS: O(V + E), где V - количество вершин, E - количество ребер

## Основные алгоритмы

### Алгоритмы сортировки

#### Быстрая сортировка (Quick Sort)

```go
func quickSort(arr []int) []int {
    if len(arr) <= 1 {
        return arr
    }
    
    pivot := arr[len(arr)/2]
    var left, middle, right []int
    
    for _, x := range arr {
        if x < pivot {
            left = append(left, x)
        } else if x == pivot {
            middle = append(middle, x)
        } else {
            right = append(right, x)
        }
    }
    
    left = quickSort(left)
    right = quickSort(right)
    
    return append(append(left, middle...), right...)
}
```

**Сложность**:
- В среднем: O(n log n)
- В худшем случае: O(n²)
- Пространственная сложность: O(log n) - O(n)

#### Сортировка слиянием (Merge Sort)

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

**Сложность**:
- Временная сложность: O(n log n)
- Пространственная сложность: O(n)

#### Сортировка вставками (Insertion Sort)

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

**Сложность**:
- Временная сложность: O(n²)
- Пространственная сложность: O(1)

### Алгоритмы поиска

#### Бинарный поиск

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
    
    return -1 // Элемент не найден
}
```

**Сложность**:
- Временная сложность: O(log n)
- Пространственная сложность: O(1)

### Алгоритмы на графах

#### Алгоритм Дейкстры (поиск кратчайшего пути)

```go
type Edge struct {
    To     int
    Weight int
}

type Graph struct {
    Vertices int
    AdjList  map[int][]Edge
}

func NewGraph(vertices int) *Graph {
    return &Graph{
        Vertices: vertices,
        AdjList:  make(map[int][]Edge),
    }
}

func (g *Graph) AddEdge(from, to, weight int) {
    g.AdjList[from] = append(g.AdjList[from], Edge{To: to, Weight: weight})
}

func (g *Graph) Dijkstra(start int) map[int]int {
    // Инициализация расстояний
    dist := make(map[int]int)
    for i := 0; i < g.Vertices; i++ {
        dist[i] = math.MaxInt32
    }
    dist[start] = 0
    
    // Очередь с приоритетом (для простоты используем обычный слайс)
    queue := make([]int, 0)
    queue = append(queue, start)
    
    for len(queue) > 0 {
        // Находим вершину с минимальным расстоянием
        minIdx := 0
        for i := 1; i < len(queue); i++ {
            if dist[queue[i]] < dist[queue[minIdx]] {
                minIdx = i
            }
        }
        
        u := queue[minIdx]
        queue = append(queue[:minIdx], queue[minIdx+1:]...)
        
        // Обновляем расстояния до соседей
        for _, edge := range g.AdjList[u] {
            v := edge.To
            weight := edge.Weight
            
            if dist[u]+weight < dist[v] {
                dist[v] = dist[u] + weight
                queue = append(queue, v)
            }
        }
    }
    
    return dist
}
```

**Сложность** (с бинарной кучей):
- Временная сложность: O((V + E) log V)
- Пространственная сложность: O(V + E)

#### Алгоритм Флойда-Уоршелла (все пары кратчайших путей)

```go
func floydWarshall(graph [][]int) [][]int {
    n := len(graph)
    dist := make([][]int, n)
    
    // Инициализация матрицы расстояний
    for i := 0; i < n; i++ {
        dist[i] = make([]int, n)
        for j := 0; j < n; j++ {
            dist[i][j] = graph[i][j]
        }
    }
    
    // Алгоритм Флойда-Уоршелла
    for k := 0; k < n; k++ {
        for i := 0; i < n; i++ {
            for j := 0; j < n; j++ {
                if dist[i][k] != math.MaxInt32 && dist[k][j] != math.MaxInt32 &&
                   dist[i][k]+dist[k][j] < dist[i][j] {
                    dist[i][j] = dist[i][k] + dist[k][j]
                }
            }
        }
    }
    
    return dist
}
```

**Сложность**:
- Временная сложность: O(V³)
- Пространственная сложность: O(V²)

## Подходы к решению алгоритмических задач

### Рекурсия

Рекурсия - это метод решения задач, при котором функция вызывает сама себя:

```go
// Факториал
func factorial(n int) int {
    if n <= 1 {
        return 1
    }
    return n * factorial(n-1)
}

// Числа Фибоначчи
func fibonacci(n int) int {
    if n <= 1 {
        return n
    }
    return fibonacci(n-1) + fibonacci(n-2)
}
```

**Ключевые моменты**:
1. Базовый случай (условие выхода)
2. Рекурсивный случай (вызов функции с измененными параметрами)
3. Приближение к базовому случаю

### Динамическое программирование

Динамическое программирование - это метод решения задач путем разбиения их на подзадачи и сохранения результатов подзадач для избежания повторных вычислений:

```go
// Числа Фибоначчи с мемоизацией
func fibonacciDP(n int, memo map[int]int) int {
    if val, ok := memo[n]; ok {
        return val
    }
    
    if n <= 1 {
        return n
    }
    
    memo[n] = fibonacciDP(n-1, memo) + fibonacciDP(n-2, memo)
    return memo[n]
}

// Числа Фибоначчи с табуляцией
func fibonacciTabulation(n int) int {
    if n <= 1 {
        return n
    }
    
    dp := make([]int, n+1)
    dp[0] = 0
    dp[1] = 1
    
    for i := 2; i <= n; i++ {
        dp[i] = dp[i-1] + dp[i-2]
    }
    
    return dp[n]
}
```

**Ключевые моменты**:
1. Определение подзадач
2. Формулировка рекуррентного соотношения
3. Мемоизация (сверху вниз) или табуляция (снизу вверх)

### Жадные алгоритмы

Жадные алгоритмы принимают локально оптимальные решения на каждом шаге:

```go
// Задача о размене монет
func coinChange(coins []int, amount int) int {
    sort.Sort(sort.Reverse(sort.IntSlice(coins)))
    count := 0
    
    for _, coin := range coins {
        for amount >= coin {
            amount -= coin
            count++
        }
    }
    
    if amount == 0 {
        return count
    }
    
    return -1 // Невозможно разменять
}
```

**Ключевые моменты**:
1. Выбор локально оптимального решения на каждом шаге
2. Не всегда приводит к глобально оптимальному решению

### Разделяй и властвуй

Разделяй и властвуй - это метод решения задач путем разбиения их на подзадачи, решения подзадач и объединения результатов:

```go
// Быстрая сортировка (пример выше)
// Сортировка слиянием (пример выше)

// Поиск максимального подмассива
func maxSubArray(nums []int) int {
    return maxSubArrayDivideConquer(nums, 0, len(nums)-1)
}

func maxSubArrayDivideConquer(nums []int, left, right int) int {
    if left == right {
        return nums[left]
    }
    
    mid := left + (right-left)/2
    
    leftMax := maxSubArrayDivideConquer(nums, left, mid)
    rightMax := maxSubArrayDivideConquer(nums, mid+1, right)
    crossMax := maxCrossingSum(nums, left, mid, right)
    
    return max(max(leftMax, rightMax), crossMax)
}

func maxCrossingSum(nums []int, left, mid, right int) int {
    leftSum := math.MinInt32
    sum := 0
    
    for i := mid; i >= left; i-- {
        sum += nums[i]
        if sum > leftSum {
            leftSum = sum
        }
    }
    
    rightSum := math.MinInt32
    sum = 0
    
    for i := mid + 1; i <= right; i++ {
        sum += nums[i]
        if sum > rightSum {
            rightSum = sum
        }
    }
    
    return leftSum + rightSum
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}
```

**Ключевые моменты**:
1. Разделение задачи на подзадачи
2. Решение подзадач рекурсивно
3. Объединение результатов

### Поиск с возвратом (Backtracking)

Поиск с возвратом - это метод решения задач путем перебора всех возможных решений и отсечения неподходящих вариантов:

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
        // Меняем местами элементы
        nums[start], nums[i] = nums[i], nums[start]
        
        // Рекурсивно генерируем перестановки для оставшихся элементов
        backtrack(nums, start+1, result)
        
        // Возвращаем исходный порядок (backtrack)
        nums[start], nums[i] = nums[i], nums[start]
    }
}
```

**Ключевые моменты**:
1. Построение решения пошагово
2. Проверка условий на каждом шаге
3. Возврат (backtrack) при нарушении условий

## Анализ сложности алгоритмов

### Временная сложность

Временная сложность описывает, как время выполнения алгоритма растет с увеличением размера входных данных:

- O(1) - константное время
- O(log n) - логарифмическое время
- O(n) - линейное время
- O(n log n) - линеарифмическое время
- O(n²) - квадратичное время
- O(2^n) - экспоненциальное время

### Пространственная сложность

Пространственная сложность описывает, как объем памяти, используемый алгоритмом, растет с увеличением размера входных данных.

### Амортизированная сложность

Амортизированная сложность учитывает среднюю стоимость операции за длительный период времени, а не в худшем случае.

## Типичные задачи на собеседованиях

### Задачи на массивы и строки

#### 1. Поиск двух чисел, дающих в сумме заданное число

```go
func twoSum(nums []int, target int) []int {
    numMap := make(map[int]int)
    
    for i, num := range nums {
        complement := target - num
        if idx, found := numMap[complement]; found {
            return []int{idx, i}
        }
        numMap[num] = i
    }
    
    return nil
}
```

#### 2. Поиск наибольшей подстроки без повторяющихся символов

```go
func lengthOfLongestSubstring(s string) int {
    charMap := make(map[byte]int)
    maxLength := 0
    start := 0
    
    for i := 0; i < len(s); i++ {
        if idx, found := charMap[s[i]]; found && idx >= start {
            start = idx + 1
        }
        
        charMap[s[i]] = i
        currentLength := i - start + 1
        
        if currentLength > maxLength {
            maxLength = currentLength
        }
    }
    
    return maxLength
}
```

#### 3. Поиск медианы двух отсортированных массивов

```go
func findMedianSortedArrays(nums1 []int, nums2 []int) float64 {
    // Обеспечиваем, что nums1 - более короткий массив
    if len(nums1) > len(nums2) {
        nums1, nums2 = nums2, nums1
    }
    
    x, y := len(nums1), len(nums2)
    low, high := 0, x
    
    for low <= high {
        partitionX := (low + high) / 2
        partitionY := (x + y + 1) / 2 - partitionX
        
        maxX := math.MinInt32
        if partitionX > 0 {
            maxX = nums1[partitionX-1]
        }
        
        minX := math.MaxInt32
        if partitionX < x {
            minX = nums1[partitionX]
        }
        
        maxY := math.MinInt32
        if partitionY > 0 {
            maxY = nums2[partitionY-1]
        }
        
        minY := math.MaxInt32
        if partitionY < y {
            minY = nums2[partitionY]
        }
        
        if maxX <= minY && maxY <= minX {
            // Нашли правильное разделение
            if (x+y)%2 == 0 {
                return float64(max(maxX, maxY)+min(minX, minY)) / 2.0
            } else {
                return float64(max(maxX, maxY))
            }
        } else if maxX > minY {
            high = partitionX - 1
        } else {
            low = partitionX + 1
        }
    }
    
    return 0.0
}

func min(a, b int) int {
    if a < b {
        return a
    }
    return b
}
```

### Задачи на динамическое программирование

#### 1. Наибольшая общая подпоследовательность

```go
func longestCommonSubsequence(text1, text2 string) int {
    m, n := len(text1), len(text2)
    dp := make([][]int, m+1)
    
    for i := range dp {
        dp[i] = make([]int, n+1)
    }
    
    for i := 1; i <= m; i++ {
        for j := 1; j <= n; j++ {
            if text1[i-1] == text2[j-1] {
                dp[i][j] = dp[i-1][j-1] + 1
            } else {
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
            }
        }
    }
    
    return dp[m][n]
}
```

#### 2. Задача о рюкзаке

```go
func knapsack(values []int, weights []int, capacity int) int {
    n := len(values)
    dp := make([][]int, n+1)
    
    for i := range dp {
        dp[i] = make([]int, capacity+1)
    }
    
    for i := 1; i <= n; i++ {
        for w := 0; w <= capacity; w++ {
            if weights[i-1] <= w {
                dp[i][w] = max(dp[i-1][w], dp[i-1][w-weights[i-1]]+values[i-1])
            } else {
                dp[i][w] = dp[i-1][w]
            }
        }
    }
    
    return dp[n][capacity]
}
```

#### 3. Максимальная сумма подмассива

```go
func maxSubArray(nums []int) int {
    if len(nums) == 0 {
        return 0
    }
    
    maxSoFar := nums[0]
    maxEndingHere := nums[0]
    
    for i := 1; i < len(nums); i++ {
        maxEndingHere = max(nums[i], maxEndingHere+nums[i])
        maxSoFar = max(maxSoFar, maxEndingHere)
    }
    
    return maxSoFar
}
```

### Задачи на графы

#### 1. Поиск кратчайшего пути в графе

```go
// Алгоритм Дейкстры (пример выше)
```

#### 2. Обнаружение цикла в графе

```go
func hasCycle(graph *Graph) bool {
    visited := make(map[int]bool)
    recStack := make(map[int]bool)
    
    for i := 0; i < graph.Vertices; i++ {
        if !visited[i] {
            if hasCycleUtil(graph, i, visited, recStack) {
                return true
            }
        }
    }
    
    return false
}

func hasCycleUtil(graph *Graph, vertex int, visited, recStack map[int]bool) bool {
    visited[vertex] = true
    recStack[vertex] = true
    
    for _, neighbor := range graph.AdjList[vertex] {
        if !visited[neighbor] {
            if hasCycleUtil(graph, neighbor, visited, recStack) {
                return true
            }
        } else if recStack[neighbor] {
            return true
        }
    }
    
    recStack[vertex] = false
    return false
}
```

#### 3. Топологическая сортировка

```go
func topologicalSort(graph *Graph) []int {
    visited := make(map[int]bool)
    stack := []int{}
    
    for i := 0; i < graph.Vertices; i++ {
        if !visited[i] {
            topologicalSortUtil(graph, i, visited, &stack)
        }
    }
    
    // Переворачиваем стек
    for i, j := 0, len(stack)-1; i < j; i, j = i+1, j-1 {
        stack[i], stack[j] = stack[j], stack[i]
    }
    
    return stack
}

func topologicalSortUtil(graph *Graph, vertex int, visited map[int]bool, stack *[]int) {
    visited[vertex] = true
    
    for _, neighbor := range graph.AdjList[vertex] {
        if !visited[neighbor] {
            topologicalSortUtil(graph, neighbor, visited, stack)
        }
    }
    
    *stack = append(*stack, vertex)
}
```

### Задачи на деревья

#### 1. Обход дерева (in-order, pre-order, post-order)

```go
// In-order (левый -> корень -> правый)
func inOrderTraversal(root *TreeNode) []int {
    result := []int{}
    inOrder(root, &result)
    return result
}

func inOrder(node *TreeNode, result *[]int) {
    if node == nil {
        return
    }
    
    inOrder(node.Left, result)
    *result = append(*result, node.Value)
    inOrder(node.Right, result)
}

// Pre-order (корень -> левый -> правый)
func preOrderTraversal(root *TreeNode) []int {
    result := []int{}
    preOrder(root, &result)
    return result
}

func preOrder(node *TreeNode, result *[]int) {
    if node == nil {
        return
    }
    
    *result = append(*result, node.Value)
    preOrder(node.Left, result)
    preOrder(node.Right, result)
}

// Post-order (левый -> правый -> корень)
func postOrderTraversal(root *TreeNode) []int {
    result := []int{}
    postOrder(root, &result)
    return result
}

func postOrder(node *TreeNode, result *[]int) {
    if node == nil {
        return
    }
    
    postOrder(node.Left, result)
    postOrder(node.Right, result)
    *result = append(*result, node.Value)
}
```

#### 2. Проверка сбалансированности дерева

```go
func isBalanced(root *TreeNode) bool {
    if root == nil {
        return true
    }
    
    leftHeight := height(root.Left)
    rightHeight := height(root.Right)
    
    if abs(leftHeight-rightHeight) <= 1 && isBalanced(root.Left) && isBalanced(root.Right) {
        return true
    }
    
    return false
}

func height(node *TreeNode) int {
    if node == nil {
        return 0
    }
    
    return 1 + max(height(node.Left), height(node.Right))
}

func abs(x int) int {
    if x < 0 {
        return -x
    }
    return x
}
```

#### 3. Наименьший общий предок

```go
func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
    if root == nil {
        return nil
    }
    
    if root.Value == p.Value || root.Value == q.Value {
        return root
    }
    
    leftLCA := lowestCommonAncestor(root.Left, p, q)
    rightLCA := lowestCommonAncestor(root.Right, p, q)
    
    if leftLCA != nil && rightLCA != nil {
        return root
    }
    
    if leftLCA != nil {
        return leftLCA
    }
    
    return rightLCA
}
```

## Советы по решению задач на собеседовании

1. **Понимание задачи**:
   - Внимательно прочитайте условие
   - Задавайте уточняющие вопросы
   - Проговорите вслух свое понимание задачи

2. **Разработка подхода**:
   - Начните с простых примеров
   - Подумайте о граничных случаях
   - Рассмотрите различные подходы (перебор, жадный алгоритм, ДП и т.д.)
   - Обсудите временную и пространственную сложность

3. **Реализация**:
   - Пишите чистый, понятный код
   - Используйте осмысленные имена переменных
   - Комментируйте сложные части

4. **Тестирование**:
   - Проверьте решение на простых примерах
   - Рассмотрите граничные случаи
   - Проверьте на наличие ошибок

5. **Оптимизация**:
   - Подумайте, можно ли улучшить решение
   - Обсудите компромиссы между временем и памятью

## Заключение

Алгоритмы и структуры данных - фундаментальная часть технического интервью. Понимание основных структур данных, алгоритмов и подходов к решению задач поможет вам успешно пройти собеседование в компаниях уровня Яндекс и Озон.

Регулярная практика на платформах вроде LeetCode, HackerRank или Codeforces поможет закрепить знания и развить навыки решения алгоритмических задач.
