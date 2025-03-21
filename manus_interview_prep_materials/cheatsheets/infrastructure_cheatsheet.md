# Шпаргалка по инфраструктуре и мониторингу

## Kubernetes

### Архитектура Kubernetes
- **Control Plane (Master)**:
  - **API Server**: точка входа для всех REST-запросов
  - **etcd**: распределенное хранилище ключ-значение для всех данных кластера
  - **Scheduler**: распределяет поды по нодам
  - **Controller Manager**: управляет контроллерами (Deployment, ReplicaSet и т.д.)
  - **Cloud Controller Manager**: взаимодействие с облачными провайдерами
- **Nodes (Worker)**:
  - **Kubelet**: агент, запускающий контейнеры на ноде
  - **Kube-proxy**: сетевой прокси, реализующий сервисы Kubernetes
  - **Container Runtime**: Docker, containerd, CRI-O

### Основные объекты Kubernetes

#### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.19
    ports:
    - containerPort: 80
    resources:
      limits:
        cpu: "0.5"
        memory: "512Mi"
      requests:
        cpu: "0.2"
        memory: "256Mi"
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 30
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
```

#### Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
```

#### Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP  # ClusterIP, NodePort, LoadBalancer, ExternalName
```

#### ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  app.properties: |
    environment=production
    log_level=info
  database.properties: |
    db_host=postgres
    db_port=5432
```

#### Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  db_password: cGFzc3dvcmQ=  # base64 encoded "password"
  api_key: c2VjcmV0a2V5  # base64 encoded "secretkey"
```

#### Ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
  tls:
  - hosts:
    - app.example.com
    secretName: app-tls-secret
```

#### PersistentVolume и PersistentVolumeClaim
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: app-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: /data/app

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: standard
```

### Сетевая модель Kubernetes
- **Pod Network**: внутренняя сеть для коммуникации между контейнерами в поде
- **Cluster Network**: сеть для коммуникации между подами
- **Service Network**: виртуальная сеть для сервисов
- **CNI (Container Network Interface)**: плагины для сетевого взаимодействия (Calico, Flannel, Weave)

### Управление ресурсами
- **Requests**: минимальные ресурсы, необходимые для запуска
- **Limits**: максимальные ресурсы, которые может использовать контейнер
- **QoS (Quality of Service)**:
  - **Guaranteed**: requests = limits
  - **Burstable**: requests < limits
  - **BestEffort**: requests и limits не указаны
- **ResourceQuota**: ограничение ресурсов на уровне namespace
- **LimitRange**: настройка ограничений по умолчанию

### Масштабирование
- **Horizontal Pod Autoscaler (HPA)**: автоматическое масштабирование подов
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 80
```
- **Vertical Pod Autoscaler (VPA)**: автоматическое изменение requests и limits
- **Cluster Autoscaler**: автоматическое масштабирование нод

### Безопасность
- **RBAC (Role-Based Access Control)**:
  - **Role/ClusterRole**: набор разрешений
  - **RoleBinding/ClusterRoleBinding**: привязка ролей к пользователям
- **NetworkPolicy**: контроль сетевого трафика между подами
- **PodSecurityPolicy**: ограничения безопасности для подов
- **ServiceAccount**: идентификация подов при доступе к API

### Основные команды kubectl
```bash
# Получение информации
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get nodes
kubectl describe pod <pod-name>

# Создание и обновление ресурсов
kubectl apply -f manifest.yaml
kubectl create -f manifest.yaml
kubectl edit deployment <deployment-name>

# Удаление ресурсов
kubectl delete pod <pod-name>
kubectl delete -f manifest.yaml

# Логи и отладка
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- /bin/bash
kubectl port-forward <pod-name> 8080:80

# Масштабирование
kubectl scale deployment <deployment-name> --replicas=5

# Обновление образа
kubectl set image deployment/<deployment-name> <container-name>=<new-image>

# Контекст и namespace
kubectl config get-contexts
kubectl config use-context <context-name>
kubectl config set-context --current --namespace=<namespace>
```

## Prometheus

### Архитектура Prometheus
- **Prometheus Server**: сбор и хранение метрик
- **Pushgateway**: прием метрик от короткоживущих задач
- **Alertmanager**: обработка и маршрутизация алертов
- **Exporters**: сбор метрик с различных систем
- **Client Libraries**: инструментирование приложений

### Типы метрик
- **Counter**: монотонно возрастающий счетчик (запросы, ошибки)
- **Gauge**: значение, которое может увеличиваться и уменьшаться (память, температура)
- **Histogram**: распределение значений по бакетам (время ответа)
- **Summary**: подобно Histogram, но с квантилями

### Инструментирование Go-приложения
```go
package main

import (
    "net/http"
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
    httpRequestsTotal = prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total number of HTTP requests",
        },
        []string{"method", "endpoint", "status"},
    )
    
    httpRequestDuration = prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Name:    "http_request_duration_seconds",
            Help:    "HTTP request duration in seconds",
            Buckets: prometheus.DefBuckets,
        },
        []string{"method", "endpoint"},
    )
)

func init() {
    prometheus.MustRegister(httpRequestsTotal)
    prometheus.MustRegister(httpRequestDuration)
}

func main() {
    http.Handle("/metrics", promhttp.Handler())
    http.ListenAndServe(":8080", nil)
}
```

### Конфигурация Prometheus
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - alertmanager:9093

rule_files:
  - "rules/*.yml"

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']
  
  - job_name: 'app'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
```

### PromQL: язык запросов Prometheus
```
# Базовые запросы
http_requests_total  # Все метрики http_requests_total
http_requests_total{method="GET"}  # Фильтрация по лейблу
http_requests_total{method="GET", status="200"}  # Несколько лейблов
http_requests_total{method=~"GET|POST"}  # Регулярное выражение

# Операторы
http_requests_total > 100  # Больше 100
http_requests_total{method="GET"} / http_requests_total  # Деление

# Функции
rate(http_requests_total[5m])  # Скорость изменения за 5 минут
sum(rate(http_requests_total[5m]))  # Сумма скоростей
sum by(method) (rate(http_requests_total[5m]))  # Группировка по методу
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))  # 95-й процентиль

# Агрегация
sum(http_requests_total)  # Сумма всех метрик
avg(http_requests_total)  # Среднее значение
min(http_requests_total)  # Минимальное значение
max(http_requests_total)  # Максимальное значение
count(http_requests_total)  # Количество метрик

# Временные ряды
http_requests_total offset 1h  # Значение час назад
rate(http_requests_total[5m]) offset 1h  # Скорость час назад
```

### Алертинг
```yaml
groups:
- name: example
  rules:
  - alert: HighRequestLatency
    expr: job:request_latency_seconds:mean5m{job="app"} > 0.5
    for: 10m
    labels:
      severity: page
    annotations:
      summary: High request latency on {{ $labels.instance }}
      description: "Request latency is above 0.5s (current value: {{ $value }}s)"
  
  - alert: InstanceDown
    expr: up == 0
    for: 5m
    labels:
      severity: page
    annotations:
      summary: "Instance {{ $labels.instance }} down"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes."
```

### Конфигурация Alertmanager
```yaml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname', 'job']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 12h
  receiver: 'slack'
  routes:
  - match:
      severity: critical
    receiver: 'pager'

receivers:
- name: 'slack'
  slack_configs:
  - api_url: 'https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX'
    channel: '#alerts'
    text: "{{ range .Alerts }}{{ .Annotations.summary }}\n{{ end }}"

- name: 'pager'
  pagerduty_configs:
  - service_key: '<key>'
```

### Масштабирование Prometheus
- **Федерация**: иерархическая структура серверов Prometheus
- **Remote Storage**: долгосрочное хранение метрик (Thanos, Cortex, VictoriaMetrics)
- **Шардирование**: разделение сбора метрик между несколькими серверами
- **Оптимизация хранения**: настройка retention и scrape_interval

## Grafana

### Основные возможности
- **Дашборды**: визуализация метрик из различных источников
- **Панели**: графики, таблицы, гистограммы, тепловые карты и т.д.
- **Источники данных**: Prometheus, InfluxDB, Elasticsearch, MySQL, PostgreSQL и др.
- **Алертинг**: настройка алертов на основе метрик
- **Аннотации**: отметки событий на графиках
- **Переменные**: динамическая настройка дашбордов

### Настройка источника данных Prometheus
1. Перейти в Configuration -> Data Sources
2. Добавить Prometheus
3. Указать URL (http://prometheus:9090)
4. Настроить интервал обновления (Scrape interval)
5. Сохранить и протестировать

### Создание дашборда
1. Создать новый дашборд (+ -> Dashboard)
2. Добавить панель (Add panel)
3. Выбрать тип визуализации (Graph, Gauge, Table и т.д.)
4. Настроить запрос (PromQL для Prometheus)
5. Настроить отображение (оси, легенда, цвета)
6. Сохранить дашборд

### Переменные в Grafana
```
# Переменная для выбора instance
Name: instance
Label: Instance
Query: label_values(up, instance)
Regex: .*
Multi-value: true
Include All option: true

# Использование переменной в запросе
up{instance=~"$instance"}
```

### Алертинг в Grafana
1. Создать правило алерта на панели
2. Настроить условие (например, значение > 80 в течение 5 минут)
3. Настроить уведомления (email, Slack, PagerDuty и т.д.)
4. Добавить сообщение и теги

### Экспорт и импорт дашбордов
- **Экспорт**: Share -> Export -> Save to file
- **Импорт**: + -> Import -> Upload .json file

### Примеры дашбордов
- **Node Exporter**: системные метрики (CPU, память, диск, сеть)
- **Kubernetes**: метрики кластера, нод, подов
- **Go Applications**: метрики Go-приложений (GC, горутины, память)
- **PostgreSQL**: метрики базы данных (соединения, транзакции, блокировки)

## Мониторинг Go-приложений

### Основные метрики для мониторинга
- **Базовые метрики**:
  - Количество запросов (total, по методам, по статусам)
  - Время ответа (среднее, процентили)
  - Ошибки (количество, процент)
  - Активные соединения
- **Метрики Go runtime**:
  - Количество горутин
  - Использование памяти (heap, stack)
  - Сборка мусора (частота, продолжительность)
  - Количество потоков ОС

### Инструментирование с помощью Prometheus
```go
package main

import (
    "net/http"
    "time"
    
    "github.com/prometheus/client_golang/prometheus"
    "github.com/prometheus/client_golang/prometheus/promauto"
    "github.com/prometheus/client_golang/prometheus/promhttp"
)

var (
    httpRequestsTotal = promauto.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total number of HTTP requests",
        },
        []string{"method", "endpoint", "status"},
    )
    
    httpRequestDuration = promauto.NewHistogramVec(
        prometheus.HistogramOpts{
            Name:    "http_request_duration_seconds",
            Help:    "HTTP request duration in seconds",
            Buckets: prometheus.DefBuckets,
        },
        []string{"method", "endpoint"},
    )
    
    activeRequests = promauto.NewGauge(
        prometheus.GaugeOpts{
            Name: "http_active_requests",
            Help: "Number of active HTTP requests",
        },
    )
)

func instrumentHandler(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        activeRequests.Inc()
        
        // Обертка для перехвата статуса ответа
        wrapped := newResponseWriter(w)
        
        defer func() {
            duration := time.Since(start).Seconds()
            httpRequestDuration.WithLabelValues(r.Method, r.URL.Path).Observe(duration)
            httpRequestsTotal.WithLabelValues(r.Method, r.URL.Path, wrapped.status).Inc()
            activeRequests.Dec()
        }()
        
        next.ServeHTTP(wrapped, r)
    })
}

func main() {
    // Регистрация метрик Go runtime
    prometheus.MustRegister(prometheus.NewBuildInfoCollector())
    prometheus.MustRegister(prometheus.NewGoCollector())
    
    // Обработчики
    http.Handle("/metrics", promhttp.Handler())
    
    api := http.NewServeMux()
    api.HandleFunc("/api/users", handleUsers)
    
    // Применение инструментирования
    http.Handle("/api/", instrumentHandler(api))
    
    http.ListenAndServe(":8080", nil)
}
```

### Мониторинг с помощью pprof
```go
import (
    "net/http"
    _ "net/http/pprof"  // Регистрирует обработчики pprof
)

func main() {
    // Доступно по /debug/pprof/
    go http.ListenAndServe(":6060", nil)
    
    // Основной код приложения
}
```

Доступные профили:
- `/debug/pprof/heap`: профиль кучи
- `/debug/pprof/goroutine`: профиль горутин
- `/debug/pprof/block`: профиль блокировок
- `/debug/pprof/mutex`: профиль мьютексов
- `/debug/pprof/threadcreate`: профиль создания потоков
- `/debug/pprof/profile`: профиль CPU (30 секунд по умолчанию)

Использование pprof:
```bash
# Анализ профиля CPU
go tool pprof http://localhost:6060/debug/pprof/profile

# Анализ профиля памяти
go tool pprof http://localhost:6060/debug/pprof/heap

# Сохранение профиля в файл
curl -o cpu.prof http://localhost:6060/debug/pprof/profile

# Анализ сохраненного профиля
go tool pprof cpu.prof
```

## Практические примеры

### Мониторинг Kubernetes с помощью Prometheus и Grafana

#### Установка с помощью Helm
```bash
# Добавление репозитория Helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Установка Prometheus Stack (Prometheus, Alertmanager, Grafana, Node Exporter)
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace
```

#### Конфигурация ServiceMonitor для приложения
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: app-monitor
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: my-app
  endpoints:
  - port: metrics
    interval: 15s
    path: /metrics
  namespaceSelector:
    matchNames:
    - default
```

#### Настройка PrometheusRule для алертов
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: app-alerts
  namespace: monitoring
spec:
  groups:
  - name: app
    rules:
    - alert: HighErrorRate
      expr: sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) > 0.05
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: High error rate
        description: Error rate is above 5% ({{ $value | humanizePercentage }})
```

### Мониторинг Go-приложения в Kubernetes

#### Dockerfile для Go-приложения с метриками
```dockerfile
FROM golang:1.19-alpine AS builder
WORKDIR /app
COPY . .
RUN go mod download
RUN CGO_ENABLED=0 GOOS=linux go build -o app .

FROM alpine:3.16
WORKDIR /app
COPY --from=builder /app/app .
EXPOSE 8080
CMD ["./app"]
```

#### Kubernetes Deployment с аннотациями для Prometheus
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/metrics"
    spec:
      containers:
      - name: my-app
        image: my-app:latest
        ports:
        - containerPort: 8080
          name: http
        - containerPort: 8080
          name: metrics
        resources:
          limits:
            cpu: "0.5"
            memory: "512Mi"
          requests:
            cpu: "0.2"
            memory: "256Mi"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
```

#### Service для приложения
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 8080
    name: http
  - port: 8080
    targetPort: 8080
    name: metrics
```

### Настройка алертов в Prometheus

#### Правила алертов для Go-приложения
```yaml
groups:
- name: go-app
  rules:
  - alert: HighErrorRate
    expr: sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) > 0.05
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: High error rate
      description: Error rate is above 5% ({{ $value | humanizePercentage }})
  
  - alert: SlowResponses
    expr: histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le)) > 1
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: Slow responses
      description: 95th percentile of response time is above 1s ({{ $value }}s)
  
  - alert: HighMemoryUsage
    expr: go_memstats_alloc_bytes / go_memstats_sys_bytes > 0.8
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: High memory usage
      description: Memory usage is above 80% ({{ $value | humanizePercentage }})
  
  - alert: TooManyGoroutines
    expr: go_goroutines > 10000
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: Too many goroutines
      description: Number of goroutines is above 10000 ({{ $value }})
```

### Дашборд Grafana для Go-приложения

#### Основные панели
1. **Запросы**:
   - График: `sum(rate(http_requests_total[5m])) by (method, endpoint)`
   - Счетчик: `sum(rate(http_requests_total[5m]))`

2. **Ошибки**:
   - График: `sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))`
   - Счетчик: `sum(rate(http_requests_total{status=~"5.."}[5m]))`

3. **Время ответа**:
   - График: `histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))`
   - Таблица: `histogram_quantile(0.5, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, endpoint))` (медиана по эндпоинтам)

4. **Горутины**:
   - График: `go_goroutines`
   - Счетчик: `go_goroutines`

5. **Память**:
   - График: `go_memstats_alloc_bytes`
   - График: `rate(go_memstats_alloc_bytes_total[5m])`
   - График: `go_memstats_heap_objects`

6. **Сборка мусора**:
   - График: `rate(go_gc_duration_seconds_sum[5m]) / rate(go_gc_duration_seconds_count[5m])`
   - График: `rate(go_gc_duration_seconds_count[5m])`

## Советы для собеседования

### Ключевые темы для подготовки
1. **Kubernetes**:
   - Архитектура и компоненты
   - Основные объекты (Pod, Deployment, Service)
   - Сетевая модель
   - Управление ресурсами и масштабирование
   - Безопасность

2. **Prometheus**:
   - Архитектура и компоненты
   - Типы метрик
   - PromQL
   - Алертинг
   - Масштабирование

3. **Grafana**:
   - Дашборды и визуализация
   - Источники данных
   - Переменные
   - Алертинг

4. **Мониторинг Go-приложений**:
   - Инструментирование с Prometheus
   - Профилирование с pprof
   - Ключевые метрики для мониторинга

### Типичные вопросы на собеседовании

1. **Kubernetes**:
   - **Что такое Pod и почему это базовая единица в Kubernetes?**
     - Pod - группа контейнеров, разделяющих сетевое пространство и хранилище
     - Контейнеры в поде всегда запускаются на одной ноде
     - Поды эфемерны и могут быть перезапущены в любой момент

   - **Чем отличаются Deployment, StatefulSet и DaemonSet?**
     - Deployment: для stateless приложений, произвольный порядок создания/удаления
     - StatefulSet: для stateful приложений, гарантированный порядок, стабильные имена
     - DaemonSet: запуск по одному поду на каждой ноде

   - **Как работает Service в Kubernetes?**
     - Абстракция для доступа к подам
     - Балансировка нагрузки между подами
     - Стабильный IP-адрес и DNS-имя
     - Типы: ClusterIP, NodePort, LoadBalancer, ExternalName

   - **Что такое Ingress и зачем он нужен?**
     - Управление входящим HTTP-трафиком
     - Маршрутизация по хостам и путям
     - Терминация TLS
     - Реализуется через Ingress Controller (Nginx, Traefik, HAProxy)

2. **Prometheus**:
   - **Как работает сбор метрик в Prometheus?**
     - Pull-модель: Prometheus опрашивает экспортеры
     - HTTP-запросы к эндпоинту /metrics
     - Метрики в текстовом формате
     - Настраиваемый интервал опроса

   - **Какие типы метрик существуют в Prometheus?**
     - Counter: монотонно возрастающий счетчик
     - Gauge: значение, которое может увеличиваться и уменьшаться
     - Histogram: распределение значений по бакетам
     - Summary: подобно Histogram, но с квантилями

   - **Как написать запрос для получения скорости изменения метрики?**
     - `rate(http_requests_total[5m])` - скорость за 5 минут
     - `irate(http_requests_total[5m])` - мгновенная скорость

   - **Как организовать долгосрочное хранение метрик?**
     - Remote Write/Read API
     - Thanos, Cortex, VictoriaMetrics
     - Федерация Prometheus

3. **Мониторинг**:
   - **Какие метрики важно мониторить в Go-приложении?**
     - Количество запросов и ошибок
     - Время ответа (латентность)
     - Количество горутин
     - Использование памяти
     - Сборка мусора
     - Количество открытых файлов и соединений

   - **Как инструментировать Go-приложение для Prometheus?**
     - Использование библиотеки prometheus/client_golang
     - Регистрация метрик (Counter, Gauge, Histogram)
     - Экспорт метрик через HTTP-эндпоинт
     - Добавление метрик Go runtime

   - **Что такое RED и USE методологии мониторинга?**
     - RED: Rate (запросы в секунду), Errors (ошибки), Duration (время ответа)
     - USE: Utilization (использование), Saturation (насыщение), Errors (ошибки)

   - **Как настроить алерты на основе метрик?**
     - Определение пороговых значений
     - Настройка правил в Prometheus
     - Настройка Alertmanager для отправки уведомлений
     - Группировка и маршрутизация алертов

### Практические советы
1. **Объясняйте свой ход мыслей**: проговаривайте, как вы подходите к решению задачи
2. **Начинайте с простого решения**: сначала предложите работающее решение, затем улучшайте его
3. **Обсуждайте компромиссы**: каждое решение имеет плюсы и минусы
4. **Задавайте уточняющие вопросы**: убедитесь, что вы правильно поняли задачу
5. **Используйте правильную терминологию**: демонстрируйте знание предметной области
6. **Приводите примеры из опыта**: рассказывайте о реальных ситуациях и решениях
