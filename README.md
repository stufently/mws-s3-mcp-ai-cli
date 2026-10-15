# MWS S3 MCP Server & AI CLI

![Status](https://img.shields.io/badge/status-development-blue) ![License](https://img.shields.io/badge/license-MIT-green)

MCP-сервер и CLI для S3 MWS Cloud Platform: подключение объектного хранилища к Claude Code, Cursor и Codex через HMAC-ключ сервисного аккаунта. Бинарь `mws-s3` помогает проверять доступ, находить бэкапы и создавать временные ссылки.

[English guide](README.en.md) · [Каталог S3 MCP-серверов](https://stufently.github.io/s3-mcp-ai-cli-kit/)

## English

A Go MCP server, CLI and agent skill for MWS S3 Object Storage. Connect Claude Code, Cursor and Codex to scoped object storage for inspection and backup recovery. Cloud Platform, storage.mwsapis.ru; ACL is disabled. See the [full English guide](README.en.md).

## Быстрый старт

Профиль предназначен для **MWS Cloud Platform** (`storage.mwsapis.ru`, `ru-central1`). Владелец создаёт HMAC-ключ сервисного аккаунта; его Access key/Secret key вводятся через `auth set`. Для presign используйте срок от 60 до 604800 секунд.

Возьми бинарь своей платформы из Releases, когда версия будет опубликована, либо собери с Go 1.27.2:

```bash
bash scripts/go.sh go build -o mws-s3 ./cmd/mws-s3
./mws-s3 version
./mws-s3 auth set
./mws-s3 auth status
./mws-s3 ls --json
```

auth set спрашивает key ID и secret без эха; без TTY читает две строки stdin. Не передавай секрет в argv. Файл ~/.config/mws-s3/credentials создаётся атомарно с 0600; более широкие права отклоняются.

Приоритет: флаги > env > файл. Переменные: MWS_S3_ACCESS_KEY_ID, MWS_S3_SECRET_ACCESS_KEY, MWS_S3_POOL (ru-central1), MWS_S3_ENDPOINT. HTTP требует MWS_S3_ALLOW_INSECURE=1.

После публикации образ: ghcr.io/stufently/mws-s3-mcp-ai-cli:0.1.0; локальная сборка: docker build -t mws-s3 . Образ запускает MCP от uid 65532, содержит CA-сертификаты. Передай креды через приватное окружение/файл.

## Возможности

- Потоковые object CRUD, copy, multipart и presigned URL в пределах профиля.
- Версии, теги, CORS, политика и Object Lock configuration регистрируются при наличии поддержки.
- Ограниченные сканы, диагностика, JSON, установка MCP и встроенного skill.

## Подключение к Claude Code, Cursor, Codex

[![Add mws-s3 MCP server to Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/en/install-mcp?name=mws-s3&config=eyJjb21tYW5kIjoiZG9ja2VyIiwiYXJncyI6WyJydW4iLCItaSIsIi0tcm0iLCItZSIsIk1XU19TM19BQ0NFU1NfS0VZX0lEIiwiLWUiLCJNV1NfUzNfU0VDUkVUX0FDQ0VTU19LRVkiLCItZSIsIk1XU19TM19QT09MIiwiZ2hjci5pby9zdHVmZW50bHkvbXdzLXMzLW1jcC1haS1jbGk6MC4xLjAiXSwiZW52Ijp7Ik1XU19TM19BQ0NFU1NfS0VZX0lEIjoiJHtlbnY6TVdTX1MzX0FDQ0VTU19LRVlfSUR9IiwiTVdTX1MzX1NFQ1JFVF9BQ0NFU1NfS0VZIjoiJHtlbnY6TVdTX1MzX1NFQ1JFVF9BQ0NFU1NfS0VZfSIsIk1XU19TM19QT09MIjoiJHtlbnY6TVdTX1MzX1BPT0x9In19)

Кнопка ставит MCP из Docker-образа `ghcr.io/stufently/mws-s3-mcp-ai-cli:0.1.0` в режиме только чтения. Ключи в конфиг не пишутся: Cursor подставляет `${env:…}` из окружения, поэтому задай `MWS_S3_ACCESS_KEY_ID` и `MWS_S3_SECRET_ACCESS_KEY` там, откуда запускается Cursor; `MWS_S3_POOL` — если бакет не в пуле по умолчанию (пустое значение = пул по умолчанию). Тот же блок — для Windsurf (`mcp_config.json`) и вручную для `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "mws-s3": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "MWS_S3_ACCESS_KEY_ID",
        "-e",
        "MWS_S3_SECRET_ACCESS_KEY",
        "-e",
        "MWS_S3_POOL",
        "ghcr.io/stufently/mws-s3-mcp-ai-cli:0.1.0"
      ],
      "env": {
        "MWS_S3_ACCESS_KEY_ID": "${env:MWS_S3_ACCESS_KEY_ID}",
        "MWS_S3_SECRET_ACCESS_KEY": "${env:MWS_S3_SECRET_ACCESS_KEY}",
        "MWS_S3_POOL": "${env:MWS_S3_POOL}"
      }
    }
  }
}
```

```bash
mws-s3 mcp install --client cursor --dry-run
mws-s3 mcp install --client cursor
mws-s3 mcp install --client claude-desktop
mws-s3 mcp install --client codex
mws-s3 mcp install --client claude-code --scope project
mws-s3 mcp install --client claude-code --scope user
```

Последняя команда только печатает claude mcp add. Путь бинаря абсолютный; ключи в конфиг не пишутся. Cursor: ~/.cursor/mcp.json, Claude Code project: ./.mcp.json; Desktop — каталог конфигурации своей ОС. JSON сохраняет чужие ключи, .bak и идемпотентность.

Codex: ~/.codex/config.toml. Существующая таблица требует --force; заменяется только блок между # >>> mws-s3 (managed) / # <<< mws-s3 (managed). Таблица без маркеров не заменяется даже с --force; остальные байты TOML целы.

Уровни включаются независимо: mws-s3 mcp --allow-write --allow-delete --allow-admin либо MWS_S3_ALLOW=write,delete,admin. Флаги доступны и в mcp install. FILE_ROOT включается через MWS_S3_FILE_ROOT.

## Инструменты MCP

| Level | Tools |
|---|---|
| read | s3_list_buckets, s3_list_objects, s3_head_object, s3_get_object, s3_preview_object, s3_presign_get, s3_list_object_versions, s3_list_multipart_uploads, s3_list_parts, s3_get_bucket_config, s3_largest_objects, s3_prefix_usage, s3_unfinished_uploads, s3_get_object_tagging |
| read + FILE_ROOT | s3_download_file |
| write | s3_put_object, s3_create_folder, s3_copy_object, s3_presign_put, s3_multipart_create, s3_multipart_presign_part, s3_multipart_complete, s3_put_object_tagging |
| write + FILE_ROOT | s3_upload_file |
| delete | s3_delete_object, s3_delete_objects, s3_multipart_abort |
| admin | s3_create_bucket, s3_put_bucket_versioning, s3_put_bucket_cors, s3_put_bucket_policy, s3_delete_bucket_policy, s3_put_object_lock_configuration |
| admin + delete | s3_delete_bucket |

Ресурсы: `mws-s3://config`, `mws-s3://capabilities`, `mws-s3://pools`. Неподдерживаемые возможности исключают инструменты.

Списки — до 1000 элементов с continuation_token; версии/multipart имеют второй маркер. Inline get/put — 262144 байт, preview — 65536; большие файлы идут через presigned URL или FILE_ROOT. Ответ содержит structuredContent и JSON-текст; текст свыше 256 КиБ заменяется краткой сводкой. Ошибки: code/message/retryable; 429/503 SDK повторяет адаптивно.

Аналитика сканирует максимум max_scan (по умолчанию 10000); partial/token отмечают неполный результат. Prefix usage выдаёт страницы агрегатов через next_result_offset. Это не биллинг.

## Сценарий: Проверка HMAC-доступа и короткие ссылки на архивы

После сохранения HMAC-ключа проверьте доступ сервисного аккаунта и выдайте короткую ссылку на существующий архив MWS Cloud Platform. Пример использует срок 60 секунд, соответствующий нижней границе в ресерче:

```bash
mws-s3 auth status
mws-s3 doctor --json
mws-s3 presign backups/daily/archive.tar --expires 60
```

Примеры используют существующий бакет `backups` и объект `daily/archive.tar`; замените их своими значениями. Профиль и API облака ещё требуют [живой проверки](TASKS.md).

## CLI

```bash
mws-s3 mb backups
mws-s3 cp ./archive.tar s3://backups/daily/archive.tar --part-size 5242880 --overwrite --yes
mws-s3 ls backups/daily/ --max-keys 100 --json
mws-s3 stat backups/daily/archive.tar
mws-s3 cp s3://backups/daily/archive.tar ./restored.tar
mws-s3 cp ./directory s3://backups/tree/ --recursive --overwrite --yes
mws-s3 cp s3://backups/tree/ ./restored-directory --recursive
mws-s3 cp s3://backups/daily/archive.tar s3://backups/copy.tar --overwrite --yes
mws-s3 cat backups/daily/notes.txt
mws-s3 versions backups/daily/ --json
mws-s3 du backups/daily/ --max-scan 10000 --json
mws-s3 presign backups/daily/archive.tar --expires 3600
mws-s3 rm backups/copy.tar --yes
mws-s3 rb empty-bucket --yes
mws-s3 doctor --json
mws-s3 compat --markdown
```

cp принимает обычные локальные пути и S3 URI s3://bucket/key либо mws-s3://bucket/key. Большие загрузки используют manager/multipart, --recursive копирует каталоги/префиксы. Перезапись требует --overwrite и подтверждения; выгрузка атомарна. rm/rb/mv без --yes на не-TTY дают код 2; на TTY спрашивают. --json общий, ошибки — stderr и ненулевой код.

doctor проверяет источник/права кредов, DNS/TLS, часы по Date и ListBuckets: >5 минут — предупреждение, >15 — ошибка.

## Skill для AI-агентов

```bash
mws-s3 skill install
mws-s3 skill install --target ./agent-skills/mws-s3
```

По умолчанию ~/.claude/skills/mws-s3. Идемпотентно устанавливает SKILL.md и шесть references: backup/restore, share, least privilege, cost hygiene, migration, incident response. Триггеры: «mws s3», «бакет MWS», «загрузи в s3», «presigned», «бэкап в s3».

## Безопасность

Если `conditional_writes:false`, запись без перезаписи отказывает до S3-запросов: HEAD не даёт атомарной защиты. Для согласованной записи используй `--overwrite --yes` в CLI либо `overwrite:true, confirm:true` в MCP, сохраняя политику подтверждения. Если `conditional_delete:false`, `mv` и `s3_move_object` отключены.



MCP по умолчанию регистрирует только read; неподдерживаемые возможности и отсутствующие уровни исключают инструмент.

MWS_S3_DESTRUCTIVE_POLICY: confirm использует elicitation при поддержке клиента, иначе нужен confirm:true после согласия человека; elicit требует elicitation и не допускает обхода confirm:true; block всегда запрещает, allow выполняет без гейта. Отказ предшествует любым S3-запросам. Гейт покрывает удаления, move, presigned PUT, политику/Object Lock, suspension versioning и перезаписи.

При conditional_writes:true операции put/copy/move/multipart complete/upload file используют overwrite:false, HEAD цели и If-None-Match: *; при false запись по умолчанию отказывает. overwrite:true требует тот же гейт.

Скоуп: MWS_S3_ALLOWED_BUCKETS=backups и MWS_S3_ALLOWED_PREFIXES=backups/daily/. Префикс учитывает границу сегмента; daily-evil не подходит. Сегменты '..' отклоняются. Частичный скоуп запрещает целый бакет и широкие списки даже при наличии его в ALLOWED_BUCKETS. Copy/move проверяет источник и цель.

Файловые MCP-инструменты используют os.Root: только относительные пути внутри FILE_ROOT; '..', абсолютные пути и симлинки наружу отклоняются. На каждый вызов одна JSON-строка в stderr; MWS_S3_AUDIT_FILE дополнительно пишет в 0600. Секреты и X-Amz-Signature/Credential/Security-Token редактируются в логах/ошибках. Полная подписанная ссылка — только результат операции. Ограничивай также права самого S3-ключа.

s3_download_file по умолчанию публикует скачанный файл без замены существующего, в том числе созданного во время скачивания. Для замены передай overwrite:true: применяется разрушительная политика с подтверждением через форму либо confirm:true после согласия человека.

## Пулы и классы хранения MWS

Пулы, эндпоинты и классы — в [таблице совместимости](docs/compat.md); каждый факт, цены и ошибки с первоисточниками — в [ресерче](docs/research/mws-s3.md). Пул по умолчанию: `ru-central1`; регион SigV4 равен выбранному пулу. В профиле `verified:false`: проверка на versitygw не подтверждает API облака. Неподтверждённые сведения помечены **[предположение]**; цены в профиле не являются калькулятором биллинга.

Lifecycle, ACL, bucket tagging и object retention могут поддерживаться облаком, но эта версия не предоставляет инструментов для их настройки; данные профиля описывают совместимость S3 API.

## FAQ

**Какой продукт MWS поддерживает этот MCP-сервер?** Профиль предназначен для **MWS Cloud Platform**, endpoint `storage.mwsapis.ru`, регион подписи `ru-central1`. Для другого продукта с похожим названием нужен отдельный профиль.

**Какие ключи нужны для MWS S3?** Access key/Secret key HMAC-ключа сервисного аккаунта; введите их через `mws-s3 auth set`. Источник и порядок создания описаны в [ресерче MWS](docs/research/mws-s3.md). Не передавайте ключи в аргументах команд.

**Почему короткая MWS presigned-ссылка использует 60 секунд?** Ресерч ссылается на документированный диапазон 60–604800 секунд. Используйте не меньше 60; фактическое поведение и верхний предел ещё требуют живой проверки.

**Что такое MCP-сервер?** Программа предоставляет агентам инструменты и ресурсы через Model Context Protocol; здесь stdio.

**Как подключить MCP-сервер к Claude Code?** Сохрани ключи auth set, затем используй project install либо напечатанную user-команду.

**Безопасно ли давать агенту доступ к S3?** Ограничь ключи, скоуп и уровни; сохрани confirm/block и одобряй разрушительные эффекты осознанно.

**Чем это отличается от aws-cli и rclone?** Бинарь объединяет MCP, гейт, аудит, аналитику, skill и CLI; aws-cli/rclone остаются самостоятельными средствами передачи/автоматизации.

**Какие возможности поддерживаются?** См. capabilities и [compat](docs/compat.md). Инструменты регистрируются по профилю; lifecycle, ACL, bucket tagging и object retention эта версия не настраивает.


## Другие провайдеры

[Общий каталог: MCP-серверы и CLI для российских S3](https://stufently.github.io/s3-mcp-ai-cli-kit/).

| Провайдер | Репозиторий | Сценарий |
|---|---|---|
| Selectel | [selectel-s3-mcp-ai-cli](https://github.com/stufently/selectel-s3-mcp-ai-cli) | Выбор пула и восстановление резервных копий |
| Yandex Cloud | [yandex-s3-mcp-ai-cli](https://github.com/stufently/yandex-s3-mcp-ai-cli) | Инвентаризация объектов и проверка версий бэкапов |
| VK Cloud | [vkcloud-s3-mcp-ai-cli](https://github.com/stufently/vkcloud-s3-mcp-ai-cli) | Проверка Hotbox/Icebox и доступ к нужному региону |
| Cloud.ru | [cloudru-s3-mcp-ai-cli](https://github.com/stufently/cloudru-s3-mcp-ai-cli) | Диагностика доступа к Evolution Object Storage |
| Timeweb Cloud | [timeweb-s3-mcp-ai-cli](https://github.com/stufently/timeweb-s3-mcp-ai-cli) | Восстановление архивов и учёт объёма префикса |
| MWS | [mws-s3-mcp-ai-cli](https://github.com/stufently/mws-s3-mcp-ai-cli) | Проверка HMAC-доступа и короткие ссылки на архивы |
| Beget | [beget-s3-mcp-ai-cli](https://github.com/stufently/beget-s3-mcp-ai-cli) | Просмотр и восстановление файлов из готового бакета |
| Shared Go kit | [s3-mcp-ai-cli-kit](https://github.com/stufently/s3-mcp-ai-cli-kit) | Общие MCP, CLI, защиты и CI |

## Лицензия

[MIT](LICENSE). [Проверенные факты и предположения](docs/research/mws-s3.md).

Проверки в Docker: bash scripts/go.sh go test -race ./..., bash scripts/conformance.sh, bash scripts/mcp_smoke.sh, bash scripts/mutation_gate.sh, bash scripts/check_docs.sh. Conformance использует versitygw v1.8.0 и тестовые ключи.

Общие MCP/CLI, защиты и проверки поставляет [s3-mcp-ai-cli-kit v0.1.0](https://github.com/stufently/s3-mcp-ai-cli-kit/tree/v0.1.0); в этом репозитории остаются встроенный профиль, тонкий main и документы MWS.
