# 📦 MyApp Helm Chart
Этот Helm Chart предназначен для деплоя приложения MyApp в Kubernetes кластере. С его помощью можно управлять развертыванием приложения в разных окружениях (development, staging, production), используя единый chart и специфичные настройки для каждого окружения.

### 🛠 Структура проекта
```bash
ui/
├── charts/               # Зависимости (если есть)
├── templates/            # Шаблоны Kubernetes-манифестов
│   ├── deployment.yaml   # Деплоймент
│   ├── _helpers.tpl      # Файл с вспомогательными шаблонами
│   ├── configmap.xml     # XML-файл конфигурации
│   ├── configmap.yaml    # YAML-файл для ConfigMap
│   ├── service.yaml      # Сервис
│   └── hpa.yaml          # YAML-файл для горизонтального автоскейлинга
├── values.yaml           # Общие значения по умолчанию
├── values-dev.yaml       # Параметры для окружения Development
├── values-test.yaml   # Параметры для окружения Staging
├── values-prod.yaml      # Параметры для окружения Production
└── Chart.yaml            # Основной файл описания Helm Chart
```

### 📋 Установка

**1. Установка для конкретного окружения**
Для установки приложения в определённом окружении используйте соответствующий файл `values` с флагом `-f`.

Пример для окружения Development:
```bash
helm install ui-dev ./singlecharts-multienv/ui -f ./singlecharts-multienv/ui/values-dev.yaml --namespace dev --create-namespace
```
Пример для окружения Staging:
```bash
helm install ui-qa ./singlecharts-multienv/ui -f ./singlecharts-multienv/ui/values-qa.yaml --namespace qa --create-namespace
```
Пример для окружения Production:
```bash
helm install ui-prod ./singlecharts-multienv/ui -f ./singlecharts-multienv/ui/values-prod.yaml --namespace prod --create-namespace
```
**Описание параметров**
- `ui-qa / ui-dev / ui-prod` — Имя релиза для каждого окружения.
- `./singlecharts-multienv/ui` — Путь к Helm-чарту.
- `-f ./singlecharts-multienv/ui/values-{environment}.yaml` — Указывает файл конфигурации, соответствующий каждому окружению.
- `--namespace {environment}` — Пространство имен, куда будет установлено приложение (qa, dev, или prod).
- `--create-namespace` — Создает пространство имен, если оно еще не существует.

**Проверка развертывания**
Для проверки статуса каждого релиза используйте:
```bash
helm status ui-qa -n qa
helm status ui-dev -n dev
helm status ui-prod -n prod
```
А для просмотра всех ресурсов в каждом пространстве имен:
```bash
kubectl get all -n qa
kubectl get all -n dev
kubectl get all -n prod
```

**2. Обновление релиза**
Для обновления релиза в определённом окружении используйте команду upgrade с нужным values файлом.

Если релиз уже установлен, используйте helm upgrade вместо helm install для обновления:
```bash
helm upgrade ui-qa ./singlecharts-multienv/ui -f ./singlecharts-multienv/ui/values-qa.yaml --namespace qa
helm upgrade ui-dev ./singlecharts-multienv/ui -f ./singlecharts-multienv/ui/values-dev.yaml --namespace dev
helm upgrade ui-prod ./singlecharts-multienv/ui -f ./singlecharts-multienv/ui/values-prod.yaml --namespace prod
```
Это позволит обновить конфигурации каждого окружения без повторной установки.


**3. Удаление релиза**
Для удаления релиза из кластера выполните команду:
```bash
helm uninstall ui-dev --namespace dev
helm uninstall ui-qa --namespace qa
helm uninstall ui-prod --namespace prod
```
### ⚙️ Конфигурация окружений
Каждый файл `values-<env>.yaml` содержит настройки, специфичные для окружения:
- `values-dev.yaml`: настройки для окружения разработки.
- `values-test.yaml`: настройки для промежуточного окружения.
- `values-prod.yaml`: настройки для окружения продакшн.

**Пример:** `values-dev.yaml`
```yaml
# Основные параметры приложения
app:
  replicas: 1                 # Количество реплик для разработки
  env: dev                     # Окружение: разработка
  container:
    image: meruyerts/ui        # Docker-образ контейнера

  computerc:                   # Ресурсы для контейнера
    limits:
      cpu: 0.2                 # Лимит на использование CPU - 0.2 ядра
      memory: 500Mi            # Лимит на использование памяти - 500 MiB
    requests:
      cpu: 0.2                 # Запрос на CPU - 0.2 ядра
      memory: 500Mi            # Запрос на память - 500 MiB

# Настройки Horizontal Pod Autoscaler (HPA)
hpa:
  enabled: false               # HPA отключен для окружения разработки

# Конфигурация файла config.yaml
config:
  name: config.yaml            # Название файла конфигурации
  data:
    timeout: 10s               # Тайм-аут для операций в режиме разработки
    logfile: /app/var/log.txt  # Путь к файлу логов
    database:
      ip: 10.20.30.40          # IP-адрес базы данных для разработки
      name: ui-dev             # Название базы данных для разработки

# Дополнительный конфигурационный файл app.xml
configxml:
  name: app.xml                # Название файла конфигурации XML


```
**Пример:** `values-qa.yaml`
```yaml
# Основные параметры приложения
app:
  replicas: 2                 # Количество реплик для QA окружения
  env: qa                      # Окружение: QA
  container:
    image: meruyerts/ui        # Docker-образ контейнера

  computerc:                   # Ресурсы для контейнера
    limits:
      cpu: 1                   # Лимит на использование CPU - 1 ядро
      memory: 2Gi              # Лимит на использование памяти - 2 GiB
    requests:
      cpu: 1                   # Запрос на CPU - 1 ядро
      memory: 2Gi              # Запрос на память - 2 GiB

# Настройки Horizontal Pod Autoscaler (HPA)
hpa:
  enabled: false               # HPA отключен для QA окружения

# Конфигурация файла config.yaml
config:
  name: config.yaml            # Название файла конфигурации
  data:
    timeout: 5s                # Тайм-аут для операций в QA
    logfile: /app/var/log.txt  # Путь к файлу логов
    database:
      ip: 120.20.30.40         # IP-адрес базы данных для QA
      name: ui-qa              # Название базы данных для QA

# Дополнительный конфигурационный файл app.xml
configxml:
  name: app.xml                # Название файла конфигурации XML

```
**Пример:** `values-prod.yaml`
```yaml
# Реплики приложения и переменные окружения
app:
  replicas: 6                # Количество реплик приложения
  env: prod                   # Окружение: продакшн
  container:
    image: meruyerts/ui       # Docker-образ контейнера

  computerc:                  # Ресурсы для контейнера
    limits:
      cpu: 4                  # Лимит на использование CPU - 4 ядра
      memory: 16Gi            # Лимит на использование памяти - 16 GiB
    requests:
      cpu: 4                  # Запрос на CPU - 4 ядра
      memory: 16Gi            # Запрос на память - 16 GiB

# Настройка Horizontal Pod Autoscaler (HPA)
hpa:
  enabled: true               # Включение HPA
  maxReplicas: 10             # Максимальное количество реплик
  minReplicas: 4              # Минимальное количество реплик
  cpuutil: 90                 # Целевое использование CPU для HPA (в процентах)

# Конфигурация файла config.yaml
config:
  name: config.yaml           # Название файла конфигурации
  data:
    timeout: 2s               # Тайм-аут для операций
    logfile: /app/var/log.txt # Путь к файлу логов
    database:
      ip: 120.21.31.10        # IP-адрес базы данных
      name: ui-prod           # Название базы данных

# Дополнительный конфигурационный файл app.xml
configxml:
  name: app.xml               # Название файла конфигурации XML


```

### 📄 Описание параметров
- **app.replicas:** Количество реплик приложения.
- **app.container.image:** Docker-образ приложения.
- **app.computerc:** Минимальные запросы и лимиты для CPU и памяти, чтобы обеспечить стабильную работу.
- **hpa.enabled:** Авто-масштабирование отключено для `dev` и `qa`,  так как разработка требует фиксированной и минимальной конфигурации. Настройка Horizontal Pod Autoscaler, который будет динамически масштабировать поды в зависимости от загрузки CPU.
- **config:** Конфигурационный файл `config.yaml` с параметрами `timeout`, пути к логам `logfile` и настройки для базы данных.
- **configxml:** Имя XML-файла конфигурации app.xml.

### 🔄 Параметры, которые можно изменить через CLI
Некоторые параметры можно изменить без использования отдельных файлов values, используя флаг --set.
```bash
helm install myapp ./myapp-chart --set replicaCount=2 --set image.tag="staging"
```

### 🧩 Примечания
**1. Единый Chart:** Использование одного chart для всех окружений упрощает управление и поддержание консистентности.
**2. Гибкость:** Благодаря файлам values можно адаптировать конфигурации для различных окружений, что удобно для CI/CD.
**3. Параметры через CI/CD:** В CI/CD пайплайнах можно динамически подставлять нужный файл values, соответствующий ветке или окружению.