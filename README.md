# 🚀 Универсальный CI/CD для микросервисов

[![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)](https://github.com/features/actions)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![YAML](https://img.shields.io/badge/YAML-000000?style=for-the-badge&logo=yaml&logoColor=white)](https://yaml.org/)


## ✨ Возможности

- 🔄 **Автоматическое тестирование** в Docker Compose для указанных веток
- 🐳 **Сборка и пуш образов** в Docker Hub или любой реестр
- 🚀 **Деплой на сервер** через SSH с генерацией docker-compose.yml
- 🎛️ **Гибкая настройка** через `ci-config.yml` — меняйте ветки, сервисы, команды без изменения кода
- 📦 **Поддержка множества микросервисов** в одном репозитории

## 📋 Быстрый старт

### 1. Создайте конфигурационный файл `ci-config.yml`

Поместите его в корень репозитория. Это сердце настройки — укажите ветки, сервисы и параметры деплоя.

#### Пример `ci-config.yml`

```yaml
test:
  branches: ["main", "master", "develop"]
  compose_file: "docker-compose.yml"
  test_service: "microservice-c"
  test_command: "pytest tests/"

deploy:
  branches: ["main", "master"]
  compose_remote_path: "/opt/myproject/docker-compose.yml"

docker:
  registry: "docker.io"
  image_prefix: "myorg/myproject"
  services:
    - name: "microservice-a"
      dockerfile: "microservice-a/Dockerfile"
      context: "microservice-a"
      container_name: "svc-a"
    - name: "microservice-b"
      dockerfile: "microservice-b/Dockerfile"
      context: "microservice-b"
      container_name: "svc-b"
    - name: "microservice-c"
      dockerfile: "microservice-c/Dockerfile"
      context: "microservice-c"
      container_name: "svc-c"
```

#### Описание полей

- **`test`** — настройки тестирования:
  - `branches`: Ветки для запуска тестов
  - `compose_file`: Путь к docker-compose.yml для тестов
  - `test_service`: Сервис, где выполняются тесты
  - `test_command`: Команда тестирования (например, `pytest`)

- **`deploy`** — настройки деплоя:
  - `branches`: Ветки для деплоя
  - `compose_remote_path`: Полный путь на сервере для docker-compose.yml

- **`docker`** — настройки Docker:
  - `registry`: Реестр образов (например, `docker.io`)
  - `image_prefix`: Префикс для тегов образов
  - `services`: Список микросервисов для сборки

### 2. Создайте workflow `.github/workflows/ci-cd.yml`

Вызовите этот reusable workflow в вашем репозитории:

```yaml
name: CI/CD

on:
  push:
  pull_request:

jobs:
  ci_cd:
    uses: Step4ansky/CICD-Reusable-workflow/.github/workflows/universal-ci-cd.yml@main
    with:
      config-path: "ci-config.yml"
    secrets:
      dockerhub-username: ${{ secrets.DOCKERHUB_USERNAME }}
      dockerhub-token: ${{ secrets.DOCKERHUB_TOKEN }}
      ssh-host: ${{ secrets.SSH_HOST }}
      ssh-username: ${{ secrets.SSH_USERNAME }}
      ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
      ssh-port: ${{ secrets.SSH_PORT }}
```

### 3. Настройте Secrets в репозитории

Добавьте эти секреты в **Settings > Secrets and variables > Actions** вашего репозитория:

| Secret              | Описание                                                                 |
|---------------------|--------------------------------------------------------------------------|
| `DOCKERHUB_USERNAME` | Логин Docker Hub (например, `myorg`)                                    |
| `DOCKERHUB_TOKEN`    | Токен доступа Docker Hub (создайте в аккаунте Docker Hub)               |
| `SSH_HOST`           | IP или домен сервера для деплоя                                         |
| `SSH_USERNAME`       | Пользователь на сервере                                                 |
| `SSH_PRIVATE_KEY`    | Приватный SSH-ключ в PEM-формате (без passphrase)                       |
| `SSH_PORT`           | Порт SSH (опционально, по умолчанию 22)                                 |

## 📁 Структура репозитория

```
.
├── microservice-a/
│   └── Dockerfile
├── microservice-b/
│   └── Dockerfile
├── microservice-c/
│   ├── Dockerfile
│   └── tests/
│       └── ...                  # Тесты
├── docker-compose.yml            # Для локальных тестов
├── ci-config.yml                 # Конфиг CI/CD
└── .github/
    └── workflows/
        └── ci-cd.yml             # Вызов универсального workflow
```

## 🛠️ Требования к серверу деплоя

- ✅ Docker и Docker Compose установлены
- ✅ Пользователь деплоя добавлен в группу `docker`:
  ```bash
  sudo usermod -aG docker deploy
  ```
- ✅ SSH-доступ с публичным ключом, соответствующим `SSH_PRIVATE_KEY`

## 🔄 Как работает пайплайн

1. **🔍 Load-config**: Читает `ci-config.yml`, конвертирует в JSON и выставляет outputs
2. **🧪 Test**: Запускает тесты в Docker Compose для указанных веток
3. **🚀 Deploy**: Собирает образы, генерирует docker-compose.yml на runner, копирует на сервер и запускает `docker compose up -d`

> 💡 **Адаптивность**: Меняйте ветки, сервисы и команды в `ci-config.yml` без изменения кода workflow!




