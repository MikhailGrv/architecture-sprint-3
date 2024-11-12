# Спринт 3

## Грязнов М

## Задание 1

# Задание 1. Анализ и планирование

## 1. Изучите функциональность монолитного приложения
### Функции:
1. Управление отоплением
2. Мониторинг температуры
Реализованы в одном контроллере
Нет авторизации/защиты
Одна БД с 2мя таблицами
Нет ограничений на количество подключений
Сервисы stateless - возможно горизонтальное масштабирование



## 2. Проанализируйте архитектуру монолитного приложения:
Язык программирования: Java
База данных: PostgreSQL
Архитектура: Монолитная, все компоненты системы (обработка запросов, бизнес-логика, работа с данными) находятся в рамках одного приложения.
Взаимодействие: Синхронное, запросы обрабатываются последовательно.
Масштабируемость: Ограничена, так как монолит сложно масштабировать по частям.
Развёртывание: Требует остановки всего приложения.

## 3. Определите домены и границы контекстов.
Исходя из текущего решения
Домены:
1. Взаимодействия с датчиками(сенсоры)/устройствами(системы) или просто Устройства
Входит как передача команд, так и получение данных с Устройств
2. Управление устройствами или просто Управление
Входит регистрация устройств, удаление устройств, текущее стостояние
Для целевого сценария доемнов больше

## 4. Проведите анализ архитектуры монолитного приложения.
### Плюсы:
1. Простота реализации - все функции в одном файле


### Возможные проблемы:
1. Надежность: Проблемы с сервисом приводят к сбою всей системы
2. Отказоустойчивость: Зависимость от одной СУБД прои нагрузке могут дать замедление/отказ всей системы
3. Простои в обслуживании: происходят с остановкой сервисов (лечится в данном случае просто - балансир + второй инстанс, с БД сложнее, но тоже реализуемо)
4. Безопасность: нет авторизации/аутентификации, работа ведется по ID датчика

## 5.Визуализируйте контекст системы



# Базовая настройка

## Запуск minikube

[Инструкция по установке](https://minikube.sigs.k8s.io/docs/start/)

```bash
minikube start
```

## Добавление токена авторизации GitHub

[Получение токена](https://github.com/settings/tokens/new)

```bash
kubectl create secret docker-registry ghcr --docker-server=https://ghcr.io --docker-username=<github_username> --docker-password=<github_token> -n default
```

## Установка API GW kusk

[Install Kusk CLI](https://docs.kusk.io/getting-started/install-kusk-cli)

```bash
kusk cluster install
```

## Смена адреса образа в helm chart

После того как вы сделали форк репозитория и у вас в репозитории отработал GitHub Action. Вам нужно получить адрес образа <https://github.com/><github_username>/architecture-sprint-3/pkgs/container/architecture-sprint-3

Он выглядит таким образом
```ghcr.io/<github_username>/architecture-sprint-3:latest```

Замените адрес образа в файле `helm/smart-home-monolith/values.yaml` на полученный файл:

```yaml
image:
  repository: ghcr.io/<github_username>/architecture-sprint-3
  tag: latest
```

## Настройка terraform

[Установите Terraform](https://yandex.cloud/ru/docs/tutorials/infrastructure-management/terraform-quickstart#install-terraform)

Создайте файл ~/.terraformrc

```hcl
provider_installation {
  network_mirror {
    url = "https://terraform-mirror.yandexcloud.net/"
    include = ["registry.terraform.io/*/*"]
  }
  direct {
    exclude = ["registry.terraform.io/*/*"]
  }
}
```

## Применяем terraform конфигурацию

```bash
cd terraform
terraform init
terraform apply
```

## Настройка API GW

```bash
kusk deploy -i api.yaml
```

## Проверяем работоспособность

```bash
kubectl port-forward svc/kusk-gateway-envoy-fleet -n kusk-system 8080:80
curl localhost:8080/hello
```

## Delete minikube

```bash
minikube delete
```
