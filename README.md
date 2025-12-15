````markdown
# Универсальный CI/CD для микросервисов

Этот документ описывает, как подключить универсальный CI/CD-воркфлоу к вашему сервисному репозиторию.

---

## 1 Создать конфигурационный файл `ci-config.yml`

Файл должен находиться в корне репозитория и быть в формате YAML.

### Пример конфигурации

```yaml
test:
  branches: ["main", "develop"]
  compose_file: "docker-compose.yml"
  test_service: "microservice-c"
  test_command: "pytest tests/"

deploy:
  branches: ["main"]
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

### Объяснение полей

* **`test`** — параметры для тестов:

  * `branches` — ветки, в которых запускаются тесты.
  * `compose_file` — путь к docker-compose для тестов.
  * `test_service` — сервис, в котором выполняются тесты.
  * `test_command` — команда для тестирования.

* **`deploy`** — параметры деплоя:

  * `branches` — ветки для деплоя.
  * `compose_remote_path` — путь на сервере для docker-compose.yml.

* **`docker`** — параметры сборки образов:

  * `registry` — Docker-реестр.
  * `image_prefix` — префикс для имён образов.
  * `services` — список микросервисов для сборки и деплоя:

    * `name` — имя сервиса.
    * `dockerfile` — путь к Dockerfile.
    * `context` — путь к контексту сборки.
    * `container_name` — опциональное имя контейнера.

---

## 2 Создать workflow `.github/workflows/ci-cd.yml`

Пример содержимого:

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

---

## 3 Настроить Secrets в репозитории

| Secret                   | Значение / где взять                                           |
| ------------------------ | -------------------------------------------------------------- |
| `DOCKERHUB_USERNAME`     | Логин Docker Hub (например, `myorg`).                          |
| `DOCKERHUB_TOKEN`        | Токен Docker Hub (не пароль). Создаётся в аккаунте Docker Hub. |
| `SSH_HOST`               | IP или домен сервера для деплоя.                               |
| `SSH_USERNAME`           | Пользователь на сервере.                                       |
| `SSH_PRIVATE_KEY`        | Приватный SSH-ключ в PEM формате (без passphrase).             |
| `SSH_PORT` (опционально) | Порт SSH, если не 22 (например, `2222`).                       |

---

## 4 Структура репозитория

```
.
├── microservice-a/
│   └── Dockerfile
├── microservice-b/
│   └── Dockerfile
├── microservice-c/
│   ├── Dockerfile
│   └── tests/
│       └── ...                  # тесты
├── docker-compose.yml            # для тестов
├── ci-config.yml                 # конфиг CI/CD
└── .github/
    └── workflows/
        └── ci-cd.yml            # вызов универсального workflow
```

---

## 5 Требования к серверу деплоя

* Установлены Docker и Docker Compose.
* Пользователь для деплоя добавлен в группу `docker`:

```bash
sudo usermod -aG docker deploy
```

* SSH-доступ с публичным ключом, соответствующим `SSH_PRIVATE_KEY`.

---

## 6 Как работает пайплайн

1. **Load-config** — считывает `ci-config.yml`, генерирует `ci-config.json` и выставляет outputs.
2. **Test** — запускает тесты в docker-compose сервисах для указанных веток.
3. **Deploy** — собирает Docker-образы, генерирует docker-compose на runner, копирует на сервер и запускает `docker compose up -d`.

> ⚡ Пайплайн адаптивный: можно менять ветки тестов и деплоя, список сервисов, реестр образов и команды тестов в `ci-config.yml`.

---

## 7 Рекомендации

* Поддерживайте `ci-config.yml` актуальным для всех микросервисов.
* Используйте отдельные ветки для тестирования перед деплоем.
* При изменениях docker-compose или сервисов проверяйте тестовый запуск локально.

````

