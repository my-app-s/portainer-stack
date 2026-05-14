> [!NOTE]
> # Portainer stack
> ![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
> ![License](https://img.shields.io/badge/license-GNU%20AGPLv3-red.svg)
>
> 📦 **This is a recipe** for deploy container portainer.

### Содержание
1. [Введение](#intro)
2. [Подробный разбор конфигурации (YAML Breakdown)](#engine)
3. [Настройка переменных (Environment Variables)](#env)
4. [Deploy](#deploy)
5. [Disclaimer](#disclaimer)
6. [License](#license)

## <a name="intro"></a>1. Введение
- Рецепт создан для упрощения развертывания portainer.


## <a name="engine"></a>2. 🔍 Подробный разбор конфигурации (YAML Breakdown)
```yaml
services:
    portainer:
        image: portainer/portainer-ce:${VERSION:-latest}
        container_name: ${NAME_CONTAINER:-portainer}
```
- **portainer** задает навзвание сервиса
- **image** задает версию образа, по *${VERSION:-latest}* есть возможность задать переменной `VERSION` нужную версию или значение по умолчанию `latest`
- **container_name** задает имя контейнера, по *${NAME_CONTAINER:-portainer}* есть возможность задать переменной `NAME_CONTAINER` нужное имя контейнера или значение по умолчанию `portainer`
```yaml
        restart: unless-stopped
        command: ${TELEMETRY:--no-analytics}
```
- **restart** задает политику автоматического перезапуска (значение unless-stopped говорит, что контейнер будет перезагружаться при сбоях или перезагрузке хоста, за исключением случаев, когда вы остановили его вручную командой docker stop)
- **command** управляет флагами запуска Portainer
    - Если переменная *TELEMETRY* не задана в окружении, подставляется значение по умолчанию `--no-analytics`
    - Это блокирует отправку данных об использовании на серверы разработчиков сразу при старте контейнера

> [!IMPORTANT]
> Отказ от ответственности (Disclaimer):
> В данном репозитории реализован принцип **Privacy by Default**. По умолчанию сбор анонимной статистики Portainer отключается через флаг --no-analytics.
> 
> Однако, автор репозитория не несет ответственности за изменения в коде стороннего программного обеспечения (Portainer), которые могут повлиять на работу этого флага в будущих версиях. Рекомендуется пользователям самостоятельно проверять исходящий трафик и логи контейнера в высокочувствительных средах.

```yaml
    ports:
        - "${HOST_PORT:-9443}:9443"
    volumes:
        - /var/run/docker.sock:/var/run/docker.sock
        - portainer_data_volume:/data
    environment:
    - TZ=UTC
    networks:
    - bridge_network
```
- **ports** задает параметр проброс портов, по *${HOST_PORT:-9443}* есть возможность задать переменной `HOST_PORT` нужный порт для внешней сети или значение по умолчанию `9443`

> [!TIP]
>
> внутренний порт менять не рекомендуется

- **volumes** задает два параметра проброса первый системный второй для data
- **environment** задает параметр TZ(часовой пояс UTC)
- **networks** задает параметр сети для сервиса
```yaml
    volumes:
    portainer_data_volume:
        name: ${NAME_CONTAINER:-portainer}_data_volume

    networks:
    bridge_network:
        name: portainer_bridge_network
        driver: bridge
```
- **volumes** на уровне сервиса настройка тома, установка статического имени по *${NAME_CONTAINER:-portainer}* есть возможность задать переменной `NAME_CONTAINER` или значение по умолчанию `portainer`
- **networks** на уровне сервиса настройка сети, установка статического имени, драйвера в режим `bridge` для доступа на внешнюю сеть

## <a name="env"></a>3. Настройка переменных (Environment Variables)

> [!NOTE]
> **Подготовка к запуску:** Склонируйте репозиторий и создайте файл `.env` на основе примера. Это позволит вам безопасно хранить настройки вне основного конфига:

```bash
cp .env.example .env
```

|Variable|Info|Variable default|
|-|-|-|
|`VERSION`|Версия образа|`latest`|
|`NAME_CONTAINER`|Имя контейнера и тома для данных|`portainer`|
|`HOST_PORT`|Порт на хост-машине|`9443`|

## <a name="deploy"></a>4. Deploy

> [!IMPORTANT]
> **Первоначальная настройка:** После запуска контейнера необходимо перейти в веб-интерфейс по адресу `http://localhost:${HOST_PORT}` и **обязательно** установить пароль администратора.

```bash
docker compose up -d
```
- команда для развертывания рядом с configuration docker-compose.yml
```bash
docker compose down
```
- command for remove containers and networks creates by command up `docker compose up -d`

> [!TIP]
>
> по команде `sudo ls -lah /var/lib/docker/volumes/` можно проверить тома или через сам docker командами
> `docker volume ls` или `docker volume inspect <имя_тома>`

## <a name="disclaimer"></a>5. ⚠️ Disclaimer / Отказ от ответственности

### English Version
This project is an **independent development** provided on an **"AS IS"** basis.

* **Liability:** In no event shall the author be liable for any errors, bugs, or data loss arising from the use of this software.
* **Status:** This is an experimental tool.

> [!CAUTION]
> Any use (operation) of this code is at your own risk.

---

### Русская версия
Данный проект является **независимой разработкой** и предоставляется «как есть».

* **Ответственность:** Автор не несет ответственности за любые ошибки, баги или потерю данных, возникшие в результате использования данного кода.
* **Статус:** Это экспериментальный инструмент.

> [!CAUTION]
> Любое использование (эксплуатация) данного кода осуществляется на ваш страх и риск.

## <a name="license"></a>6. 📜 License

Проект лицензирован под **GNU Affero General Public License v3.0 (AGPLv3)**.
См. [LICENSE](./LICENSE) для подробностей.
