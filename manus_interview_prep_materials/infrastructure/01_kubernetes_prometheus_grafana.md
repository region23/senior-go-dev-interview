# Основы Kubernetes, Prometheus и Grafana

В этом документе представлены основные концепции Kubernetes, Prometheus и Grafana, которые часто проверяются на технических интервью в компаниях уровня Яндекс и Озон. Материал поможет вам освежить знания в области инфраструктуры и мониторинга.

## 1. Kubernetes

### 1.1 Основные концепции Kubernetes

**Kubernetes (K8s)** - это открытая платформа для автоматизации развертывания, масштабирования и управления контейнеризованными приложениями. Ключевые концепции:

**Кластер Kubernetes** состоит из:
- **Master-узлы** (Control Plane) - управляют кластером
- **Worker-узлы** (Nodes) - запускают приложения

**Компоненты Control Plane**:
- **kube-apiserver**: API-сервер, точка входа для управления кластером
- **etcd**: распределенное хранилище ключ-значение для всех данных кластера
- **kube-scheduler**: выбирает узлы для запуска подов
- **kube-controller-manager**: запускает контроллеры (репликации, сервисов и т.д.)
- **cloud-controller-manager**: взаимодействует с облачным провайдером

**Компоненты Worker-узлов**:
- **kubelet**: агент, запускающий контейнеры в подах
- **kube-proxy**: сетевой прокси, реализующий сервисы Kubernetes
- **Container Runtime**: ПО для запуска контейнеров (Docker, containerd, CRI-O)

### 1.2 Основные объекты Kubernetes

**Pod** - базовая единица развертывания в Kubernetes:
- Группа контейнеров, разделяющих сетевое пространство и хранилище
- Обычно содержит один основной контейнер и, возможно, вспомогательные (sidecar)
- Эфемерен (временный) - может быть удален и пересоздан в любой момент

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
```

**ReplicaSet** - обеспечивает запуск определенного количества реплик подов:
- Поддерживает заданное количество подов
- Автоматически создает новые поды при сбоях
- Обычно не используется напрямую, а через Deployment

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
```

**Deployment** - управляет развертыванием и обновлением приложений:
- Создает и управляет ReplicaSet
- Обеспечивает декларативное обновление (rolling update)
- Поддерживает историю развертываний и возможность отката

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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

**Service** - абстракция, определяющая набор подов и политику доступа к ним:
- Обеспечивает стабильный IP-адрес и DNS-имя
- Балансирует нагрузку между подами
- Типы: ClusterIP, NodePort, LoadBalancer, ExternalName

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
  type: ClusterIP
```

**Ingress** - управляет внешним доступом к сервисам в кластере:
- Предоставляет HTTP/HTTPS маршрутизацию
- Поддерживает виртуальные хосты и TLS
- Требует Ingress-контроллер (Nginx, Traefik, HAProxy и др.)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80
```

**ConfigMap** и **Secret** - хранят конфигурационные данные:
- ConfigMap - для некритичных данных
- Secret - для чувствительных данных (шифруются)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  app.properties: |
    environment=production
    log_level=info
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  db_password: cGFzc3dvcmQ=  # base64-encoded "password"
```

**PersistentVolume (PV)** и **PersistentVolumeClaim (PVC)** - управляют хранилищем:
- PV - абстракция физического хранилища
- PVC - запрос на хранилище от пода

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-storage
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: standard
  hostPath:
    path: /data
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-storage
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: standard
```

**StatefulSet** - для приложений с сохранением состояния:
- Стабильные, уникальные сетевые идентификаторы
- Стабильное, постоянное хранилище
- Упорядоченное развертывание и масштабирование

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: "postgres"
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:13
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 10Gi
```

**DaemonSet** - запускает под на каждом узле кластера:
- Используется для системных сервисов (мониторинг, логирование)
- Автоматически запускается на новых узлах

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      containers:
      - name: node-exporter
        image: prom/node-exporter
        ports:
        - containerPort: 9100
```

**Job** и **CronJob** - для выполнения задач:
- Job - одноразовая задача
- CronJob - периодическая задача по расписанию

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job
spec:
  template:
    spec:
      containers:
      - name: batch-job
        image: batch-processor
      restartPolicy: Never
  backoffLimit: 4
```

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-job
spec:
  schedule: "0 0 * * *"  # Ежедневно в полночь
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-tool
          restartPolicy: OnFailure
```

### 1.3 Сетевая модель Kubernetes

**Сетевая модель** Kubernetes обеспечивает:
- Каждый под имеет уникальный IP-адрес
- Поды могут общаться друг с другом без NAT
- Узлы могут общаться с подами без NAT
- IP-адрес пода виден изнутри и снаружи

**Сетевые политики (Network Policies)** - правила, определяющие, как поды могут общаться:
- Ограничение входящего и исходящего трафика
- Фильтрация по IP-адресам, портам, протоколам

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: backend
    ports:
    - protocol: TCP
      port: 5432
```

**Service Mesh** (например, Istio, Linkerd) - дополнительный слой для управления сетевым взаимодействием:
- Шифрование трафика (mTLS)
- Маршрутизация и балансировка нагрузки
- Отказоустойчивость (circuit breaking, retry)
- Наблюдаемость (метрики, трассировка)

### 1.4 Управление ресурсами

**Запросы (Requests)** и **Лимиты (Limits)** ресурсов:
- Requests - минимальные гарантированные ресурсы
- Limits - максимальные доступные ресурсы

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: app
    image: app-image
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

**Namespace** - виртуальный кластер внутри физического:
- Изоляция ресурсов
- Управление доступом
- Квоты ресурсов

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
```

**ResourceQuota** - ограничивает потребление ресурсов в namespace:
- Ограничение CPU, памяти, хранилища
- Ограничение количества объектов

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: development
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
```

**LimitRange** - устанавливает значения по умолчанию для контейнеров:
- Значения по умолчанию для requests и limits
- Минимальные и максимальные значения

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
  namespace: development
spec:
  limits:
  - default:
      cpu: 500m
      memory: 256Mi
    defaultRequest:
      cpu: 200m
      memory: 128Mi
    type: Container
```

### 1.5 Масштабирование в Kubernetes

**Горизонтальное масштабирование подов (HPA)** - автоматическое изменение количества реплик:
- На основе метрик CPU, памяти или пользовательских метрик
- Требует Metrics Server или другой источник метрик

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
        averageUtilization: 70
```

**Вертикальное масштабирование подов (VPA)** - автоматическое изменение запросов и лимитов:
- Анализирует фактическое использование ресурсов
- Рекомендует или автоматически изменяет requests

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-deployment
  updatePolicy:
    updateMode: Auto
```

**Cluster Autoscaler** - автоматическое изменение количества узлов:
- Добавляет узлы при нехватке ресурсов
- Удаляет узлы при избытке ресурсов
- Интегрируется с облачными провайдерами

### 1.6 Безопасность в Kubernetes

**RBAC (Role-Based Access Control)** - управление доступом на основе ролей:
- Role и ClusterRole - набор разрешений
- RoleBinding и ClusterRoleBinding - привязка ролей к пользователям или группам

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**SecurityContext** - настройки безопасности для подов и контейнеров:
- Запуск от определенного пользователя
- Привилегированный режим
- Возможности Linux (capabilities)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: app
    image: app-image
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop: ["ALL"]
```

**PodSecurityPolicy (PSP)** / **PodSecurityStandard (PSS)** - политики безопасности для подов:
- Ограничение привилегий
- Контроль монтирования томов
- Контроль использования hostNetwork, hostPID, hostIPC

**NetworkPolicy** - управление сетевым трафиком:
- Ограничение входящего и исходящего трафика
- Изоляция подов

### 1.7 Управление конфигурацией

**Helm** - менеджер пакетов для Kubernetes:
- Пакеты (charts) - коллекции ресурсов Kubernetes
- Шаблонизация YAML-файлов
- Управление релизами и откатами

**Kustomize** - инструмент для настройки YAML-манифестов:
- Наложение патчей без шаблонизации
- Встроен в kubectl (kubectl apply -k)

**Оператор (Operator)** - расширение Kubernetes API для управления приложениями:
- Автоматизация операций (установка, обновление, резервное копирование)
- Использует Custom Resource Definitions (CRD)
- Примеры: Prometheus Operator, PostgreSQL Operator

## 2. Prometheus

### 2.1 Основные концепции Prometheus

**Prometheus** - система мониторинга и алертинга с открытым исходным кодом:
- Сбор метрик по модели pull (опрос)
- Хранение временных рядов
- Мощный язык запросов (PromQL)
- Алертинг и визуализация

**Архитектура Prometheus**:
- **Prometheus Server** - сбор и хранение метрик
- **Exporters** - экспортеры метрик для различных систем
- **Pushgateway** - для кратковременных задач
- **Alertmanager** - обработка и маршрутизация алертов
- **Prometheus Web UI** / **Grafana** - визуализация

**Модель данных**:
- **Метрика** - имя и набор пар ключ-значение (лейблы)
- **Временной ряд** - последовательность значений метрики с метками времени
- **Типы метрик**:
  - **Counter** - монотонно возрастающий счетчик (запросы, ошибки)
  - **Gauge** - значение, которое может увеличиваться и уменьшаться (температура, память)
  - **Histogram** - распределение значений по бакетам (время ответа)
  - **Summary** - подобно Histogram, но с квантилями

### 2.2 Сбор метрик

**Конфигурация Prometheus**:
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```

**Service Discovery** - автоматическое обнаружение целей:
- Статическая конфигурация
- DNS-обнаружение
- Kubernetes Service Discovery
- Облачные провайдеры (AWS, GCP, Azure)

**Exporters** - компоненты для экспорта метрик из систем, не поддерживающих Prometheus:
- **Node Exporter** - метрики операционной системы
- **Blackbox Exporter** - проверки доступности (HTTP, ICMP, TCP)
- **MySQL Exporter**, **PostgreSQL Exporter** - метрики баз данных
- **JMX Exporter** - метрики Java-приложений

**Инструментирование приложений** - добавление метрик в код:
- Клиентские библиотеки для разных языков (Go, Java, Python и др.)
- Стандартные метрики (запросы, ошибки, время ответа)
- Пользовательские метрики

Пример инструментирования в Go:
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

### 2.3 PromQL - язык запросов Prometheus

**Базовые запросы**:
- `http_requests_total` - все временные ряды для метрики
- `http_requests_total{method="GET"}` - фильтрация по лейблу
- `http_requests_total{method=~"GET|POST"}` - регулярное выражение
- `http_requests_total{method!="DELETE"}` - отрицание

**Операторы**:
- Арифметические: `+`, `-`, `*`, `/`, `%`, `^`
- Сравнения: `==`, `!=`, `>`, `<`, `>=`, `<=`
- Логические: `and`, `or`, `unless`

**Функции**:
- `rate(http_requests_total[5m])` - скорость изменения за 5 минут
- `sum(rate(http_requests_total[5m]))` - суммирование по всем временным рядам
- `sum by(method) (rate(http_requests_total[5m]))` - суммирование с группировкой
- `topk(5, http_requests_total)` - топ-5 значений

**Агрегация**:
- `sum`, `min`, `max`, `avg`, `stddev`, `count`
- `quantile`

**Примеры запросов**:
- Запросов в секунду по методу:
  ```
  sum by(method) (rate(http_requests_total[5m]))
  ```
- Процент ошибок:
  ```
  sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) * 100
  ```
- 95-й процентиль времени ответа:
  ```
  histogram_quantile(0.95, sum by(le) (rate(http_request_duration_seconds_bucket[5m])))
  ```
- Использование памяти по подам:
  ```
  sum by(pod) (container_memory_usage_bytes{namespace="production"})
  ```

### 2.4 Алертинг

**Правила алертинга** - определяют условия для срабатывания алертов:
```yaml
groups:
- name: example
  rules:
  - alert: HighErrorRate
    expr: sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) > 0.1
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "High error rate detected"
      description: "Error rate is above 10% for 5 minutes (current value: {{ $value }})"
```

**Alertmanager** - обрабатывает и маршрутизирует алерты:
- Группировка похожих алертов
- Подавление дублирующихся алертов
- Маршрутизация по различным каналам (email, Slack, PagerDuty)
- Дежурства и эскалация

```yaml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname', 'job']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 12h
  receiver: 'team-email'
  routes:
  - match:
      severity: critical
    receiver: 'team-pager'

receivers:
- name: 'team-email'
  email_configs:
  - to: 'team@example.com'
- name: 'team-pager'
  pagerduty_configs:
  - service_key: '<key>'
```

### 2.5 Хранение и масштабирование

**Локальное хранилище**:
- Временные ряды хранятся на диске
- Настраиваемое время хранения
- Ограничения по масштабированию

**Удаленное хранилище**:
- Долгосрочное хранение метрик
- Поддержка различных бэкендов (Thanos, Cortex, VictoriaMetrics)
- Федерация для объединения нескольких Prometheus

**Thanos** - система для масштабирования Prometheus:
- Долгосрочное хранение в объектном хранилище (S3, GCS)
- Глобальный обзор нескольких кластеров
- Дедупликация метрик

**Cortex** - горизонтально масштабируемый Prometheus:
- Многотенантность
- Распределенное хранилище
- Высокая доступность

## 3. Grafana

### 3.1 Основные концепции Grafana

**Grafana** - платформа для визуализации и аналитики:
- Поддержка различных источников данных (Prometheus, InfluxDB, Elasticsearch и др.)
- Создание интерактивных дашбордов
- Алертинг и уведомления
- Аннотации и переменные

**Архитектура Grafana**:
- **Веб-сервер** - пользовательский интерфейс
- **База данных** - хранение дашбордов, пользователей, настроек
- **Плагины** - расширения для источников данных, панелей, приложений

### 3.2 Дашборды и панели

**Дашборд** - набор панелей с визуализациями:
- Настраиваемый макет
- Временной диапазон
- Переменные для фильтрации

**Типы панелей**:
- **Graph** - линейные графики, гистограммы
- **Stat** - одиночные значения
- **Gauge** - индикаторы
- **Table** - табличные данные
- **Heatmap** - тепловые карты
- **Alert List** - список алертов

**Создание панели с Prometheus**:
1. Выбор источника данных (Prometheus)
2. Написание PromQL-запроса
3. Настройка визуализации (оси, легенда, цвета)
4. Настройка алертов (опционально)

### 3.3 Переменные и шаблоны

**Переменные** - динамические значения для фильтрации дашбордов:
- **Query** - значения из запроса
- **Interval** - временные интервалы
- **Custom** - пользовательские значения
- **Constant** - константы
- **Data source** - источники данных

**Пример запроса для переменной**:
```
label_values(kube_pod_info{namespace="$namespace"}, pod)
```

**Использование переменных в запросах**:
```
sum by(pod) (rate(http_requests_total{namespace="$namespace", pod=~"$pod"}[5m]))
```

**Шаблоны повторения** - создание панелей для каждого значения переменной:
- Автоматическое создание панелей для каждого пода, сервиса и т.д.
- Единый шаблон для всех панелей

### 3.4 Алертинг в Grafana

**Правила алертинга** - определяют условия для срабатывания алертов:
- Пороговые значения
- Продолжительность
- Уведомления

**Каналы уведомлений**:
- Email
- Slack
- PagerDuty
- Webhook
- Telegram

**Состояния алертов**:
- OK - условие не выполняется
- Pending - условие выполняется, но не достаточно долго
- Alerting - условие выполняется достаточно долго
- No Data - нет данных
- Error - ошибка при выполнении запроса

### 3.5 Организация и управление

**Организации** - изолированные среды с собственными дашбордами и пользователями

**Команды** - группы пользователей с общими разрешениями

**Папки** - организация дашбордов по категориям

**Разрешения**:
- Просмотр
- Редактирование
- Администрирование

**Снимки** - статические копии дашбордов для обмена

**Аннотации** - отметки событий на графиках:
- Развертывания
- Инциденты
- Обслуживание

## 4. Практические примеры

### 4.1 Мониторинг Kubernetes с Prometheus и Grafana

**Установка Prometheus в Kubernetes**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus:v2.30.0
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: config
          mountPath: /etc/prometheus
      volumes:
      - name: config
        configMap:
          name: prometheus-config
---
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  namespace: monitoring
spec:
  selector:
    app: prometheus
  ports:
  - port: 9090
    targetPort: 9090
```

**Конфигурация Prometheus для Kubernetes**:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    
    scrape_configs:
      - job_name: 'kubernetes-apiservers'
        kubernetes_sd_configs:
        - role: endpoints
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        relabel_configs:
        - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
          action: keep
          regex: default;kubernetes;https
      
      - job_name: 'kubernetes-nodes'
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        kubernetes_sd_configs:
        - role: node
        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
      
      - job_name: 'kubernetes-pods'
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
        - action: labelmap
          regex: __meta_kubernetes_pod_label_(.+)
        - source_labels: [__meta_kubernetes_namespace]
          action: replace
          target_label: kubernetes_namespace
        - source_labels: [__meta_kubernetes_pod_name]
          action: replace
          target_label: kubernetes_pod_name
```

**Установка Grafana в Kubernetes**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:8.2.0
        ports:
        - containerPort: 3000
        env:
        - name: GF_SECURITY_ADMIN_PASSWORD
          value: admin
        - name: GF_USERS_ALLOW_SIGN_UP
          value: "false"
---
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: monitoring
spec:
  selector:
    app: grafana
  ports:
  - port: 3000
    targetPort: 3000
```

**Настройка Grafana для Prometheus**:
1. Добавление источника данных Prometheus (URL: http://prometheus:9090)
2. Импорт готовых дашбордов:
   - Kubernetes Cluster Overview (ID: 315)
   - Node Exporter Full (ID: 1860)
   - Kubernetes Pods (ID: 6417)

### 4.2 Мониторинг Go-приложения

**Инструментирование Go-приложения**:
```go
package main

import (
    "net/http"
    "time"
    
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
    
    activeRequests = prometheus.NewGauge(
        prometheus.GaugeOpts{
            Name: "http_active_requests",
            Help: "Number of active HTTP requests",
        },
    )
)

func init() {
    prometheus.MustRegister(httpRequestsTotal)
    prometheus.MustRegister(httpRequestDuration)
    prometheus.MustRegister(activeRequests)
}

func metricsMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        activeRequests.Inc()
        
        // Wrap ResponseWriter to capture status code
        wrapped := newResponseWriter(w)
        
        next.ServeHTTP(wrapped, r)
        
        activeRequests.Dec()
        status := wrapped.statusCode
        
        // Record metrics
        httpRequestsTotal.WithLabelValues(r.Method, r.URL.Path, string(status)).Inc()
        httpRequestDuration.WithLabelValues(r.Method, r.URL.Path).Observe(time.Since(start).Seconds())
    })
}

func main() {
    // Metrics endpoint
    http.Handle("/metrics", promhttp.Handler())
    
    // Application endpoints
    http.Handle("/api/", metricsMiddleware(apiHandler()))
    
    http.ListenAndServe(":8080", nil)
}
```

**Dockerfile для Go-приложения**:
```dockerfile
FROM golang:1.17 AS builder
WORKDIR /app
COPY . .
RUN go mod download
RUN CGO_ENABLED=0 GOOS=linux go build -o app .

FROM alpine:3.14
WORKDIR /app
COPY --from=builder /app/app .
EXPOSE 8080
CMD ["./app"]
```

**Kubernetes Deployment для Go-приложения**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: go-app
  template:
    metadata:
      labels:
        app: go-app
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/metrics"
    spec:
      containers:
      - name: go-app
        image: go-app:latest
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 256Mi
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 20
---
apiVersion: v1
kind: Service
metadata:
  name: go-app
spec:
  selector:
    app: go-app
  ports:
  - port: 80
    targetPort: 8080
```

**Grafana Dashboard для Go-приложения**:
- Запросов в секунду:
  ```
  sum by(endpoint) (rate(http_requests_total[5m]))
  ```
- Ошибки в секунду:
  ```
  sum by(endpoint) (rate(http_requests_total{status=~"5.."}[5m]))
  ```
- Процент ошибок:
  ```
  sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) * 100
  ```
- Время ответа (95-й процентиль):
  ```
  histogram_quantile(0.95, sum by(endpoint, le) (rate(http_request_duration_seconds_bucket[5m])))
  ```
- Активные запросы:
  ```
  sum(http_active_requests)
  ```

### 4.3 Алерты для типичных проблем

**Высокая загрузка CPU**:
```yaml
- alert: HighCPULoad
  expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "High CPU load (instance {{ $labels.instance }})"
    description: "CPU load is above 80% for 5 minutes (current value: {{ $value }}%)"
```

**Высокое использование памяти**:
```yaml
- alert: HighMemoryUsage
  expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 90
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "High memory usage (instance {{ $labels.instance }})"
    description: "Memory usage is above 90% for 5 minutes (current value: {{ $value }}%)"
```

**Высокая загрузка диска**:
```yaml
- alert: HighDiskUsage
  expr: 100 - ((node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100) > 85
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "High disk usage (instance {{ $labels.instance }})"
    description: "Disk usage is above 85% for 5 minutes (current value: {{ $value }}%)"
```

**Недоступность сервиса**:
```yaml
- alert: ServiceDown
  expr: up == 0
  for: 1m
  labels:
    severity: critical
  annotations:
    summary: "Service down (instance {{ $labels.instance }})"
    description: "Service is down for more than 1 minute"
```

**Высокая латентность**:
```yaml
- alert: HighLatency
  expr: histogram_quantile(0.95, sum by(le) (rate(http_request_duration_seconds_bucket[5m]))) > 1
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "High latency"
    description: "95th percentile of request duration is above 1 second for 5 minutes (current value: {{ $value }}s)"
```

**Высокий уровень ошибок**:
```yaml
- alert: HighErrorRate
  expr: sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) > 0.05
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "High error rate"
    description: "Error rate is above 5% for 5 minutes (current value: {{ $value | humanizePercentage }})"
```

**Недостаток ресурсов в Kubernetes**:
```yaml
- alert: KubernetesPodCrashLooping
  expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
  for: 15m
  labels:
    severity: warning
  annotations:
    summary: "Pod is crash looping ({{ $labels.namespace }}/{{ $labels.pod }})"
    description: "Pod {{ $labels.namespace }}/{{ $labels.pod }} is restarting {{ $value }} times / 15 minutes"
```

## 5. Заключение

В этом документе мы рассмотрели основные концепции Kubernetes, Prometheus и Grafana, которые часто проверяются на технических интервью в компаниях уровня Яндекс и Озон:

1. **Kubernetes**:
   - Архитектура и компоненты
   - Основные объекты (Pod, Deployment, Service и др.)
   - Сетевая модель
   - Управление ресурсами
   - Масштабирование
   - Безопасность
   - Управление конфигурацией

2. **Prometheus**:
   - Архитектура и компоненты
   - Модель данных
   - Сбор метрик
   - PromQL
   - Алертинг
   - Хранение и масштабирование

3. **Grafana**:
   - Дашборды и панели
   - Переменные и шаблоны
   - Алертинг
   - Организация и управление

4. **Практические примеры**:
   - Мониторинг Kubernetes
   - Мониторинг Go-приложения
   - Алерты для типичных проблем

Эти знания помогут вам успешно пройти техническое интервью в компаниях уровня Яндекс и Озон, особенно в части, касающейся инфраструктуры и мониторинга.
