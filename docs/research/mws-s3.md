# MWS Cloud Platform S3 — официальный ресерч

Проверено 2026-10-10 по официальным MWS Docs. Профиль предназначен для **нынешней MWS Cloud Platform**, endpoint `storage.mwsapis.ru`; прежний S3 MTS Cloud требует отдельного контракта. Живой аккаунт не использовался, `verified:false`; **[предположение]** — непроверенные сведения/консервативные пределы. [Quickstart](https://mws.ru/docs/cloud-platform/storage/general/quickstart-aws-cli.html).

## Подключение и адресация

Endpoint `https://storage.mwsapis.ru`, регион SigV4 `ru-central1`. Для доступа нужен **HMAC-ключ сервисного аккаунта**, его Access key/Secret key; токен веб-консоли не является S3-ключом. [Quickstart](https://mws.ru/docs/cloud-platform/storage/general/quickstart-aws-cli.html), [AWS CLI](https://mws.ru/docs/cloud-platform/storage/general/instruments-aws-cli.html).

Поддерживаются path-style `storage.mwsapis.ru/bucket/key` и virtual-host `bucket.storage.mwsapis.ru/key`; профиль использует path-style. [Объект/URL](https://mws.ru/docs/cloud-platform/storage/general/object-overview.html), [S3cmd](https://mws.ru/docs/cloud-platform/storage/general/instruments-s3cmd.html).

Доступность заявлена не ниже 99,95% в год; сервис сохраняет доступность при отказе одной зоны. Это основание `multi_az:true`; количество реплик и схема распределения конкретного класса не установлены — **[предположение]** для живой проверки. [Обзор сервиса](https://mws.ru/docs/cloud-platform/storage/general/whatis-object-storage.html).

## Классы и цены

Действующая документация объектов, обновлённая 30.09.2026, перечисляет `STANDARD`, `WARM`, `COLD`, выбор при загрузке и default-класс бакета. WARM оптимизирован для скорости, COLD — для редко используемых архивов. Устаревшее утверждение в блоге про один класс не используется. [Классы](https://mws.ru/docs/cloud-platform/storage/general/object-overview.html).

Таблицы тарификации: STANDARD — 2,44 ₽/ГБ·месяц, WARM — 9,15, COLD — 1,26, с НДС 22%. Исходящий трафик — 1,3212 ₽/ГБ для всех классов. Биллинговый ГБ = 2^30 байт. PUT/POST/PATCH/LIST оплачиваются пакетами по 10, GET/HEAD/OPTIONS — по 100, DELETE бесплатны. [Object Storage pricing](https://mws.ru/docs/cloud-platform/storage/general/pricing.html), [Общий прайс](https://mws.ru/docs/cloud-platform/about/general/mws-cloud-platform-pricing.html).

В статье pricing расчёт-пример использует старые ставки операций (отличающиеся от таблицы); числа примера не перенесены. **[предположение]** скидки, free tier, оплата пустых бакетов и окончательная цена аккаунта требуют проверки: старая страница payment противоречиво описывает остановку платы и плату за пустые бакеты. Это не мешает построить API-профиль; `verified:false` сохраняется. [Pricing](https://mws.ru/docs/cloud-platform/storage/general/pricing.html), [Payment](https://mws.ru/docs/cloud-platform/storage/general/payment.html).

## Условия и API

| Возможность | Документированный контракт/решение | Источник |
|---|---|---|
| If-None-Match / If-Match | **[предположение]** атомарные условия PUT/Copy/Complete/Delete не установлены в инструкциях; `conditional_writes:false`, `conditional_delete:false` | [Upload](https://mws.ru/docs/cloud-platform/storage/general/object-upload.html), [Copy](https://mws.ru/docs/cloud-platform/storage/general/object-copy.html), [Delete](https://mws.ru/docs/cloud-platform/storage/general/object-delete.html) |
| Object tags | Get/Put/DeleteObjectTagging | [Теги объекта](https://mws.ru/docs/cloud-platform/storage/general/object-tags.html) |
| Bucket tags | Get/Put/DeleteBucketTagging | [Теги бакета](https://mws.ru/docs/cloud-platform/storage/general/bucket-tags.html) |
| Lifecycle | Get/Put/DeleteBucketLifecycleConfiguration | [Настройка lifecycle](https://mws.ru/docs/cloud-platform/storage/general/lifecycle-work-with-configuration.html) |
| ACL | **[предположение]** S3 ACL не подтверждён, `acl:false`; документированы IAM/политики доступа | [Управление доступом](https://mws.ru/docs/cloud-platform/storage/general/access-overview.html) |
| Policy | Get/Put/DeleteBucketPolicy | [Политики](https://mws.ru/docs/cloud-platform/storage/general/access-bucket-policy-operations.html) |
| Versioning | Enable/Suspend, операции с версиями и маркерами | [Версии](https://mws.ru/docs/cloud-platform/storage/general/versioning-managing-versions.html) |
| Object Lock | Get/PutObjectLockConfiguration; Get/PutRetention и legal hold, governance/compliance | [Lock API](https://mws.ru/docs/cloud-platform/storage/general/object-lock-settings.html), [Lock обзор](https://mws.ru/docs/cloud-platform/storage/general/object-lock-overview.html) |
| CORS | Get/Put/DeleteBucketCors | [CORS](https://mws.ru/docs/cloud-platform/storage/general/cors-work-with-configuration.html) |
| Multipart | Create/UploadPart/ListParts/Complete/Abort | [Загрузка](https://mws.ru/docs/cloud-platform/storage/general/object-upload.html) |

Запись по умолчанию отказывает до S3-запросов; согласованная перезапись требует `overwrite:true` и общего гейта. Move отсутствует. Lifecycle, ACL, bucket tags и retention в этой версии kit не имеют инструментов настройки.

## Presign, квоты и ошибки

GET и PUT presign доступны; документированный срок **60–604800 секунд (1 минута–7 дней)**. Профиль задаёт потолок 7 дней; используйте срок не меньше 60 секунд, поскольку общий клиент может подписать более короткий URL, принятие которого MWS не обещает. [Presign](https://mws.ru/docs/cloud-platform/storage/general/access-presigned-url.html).

На проект по умолчанию: 25 бакетов, 1 ТБ суммарного хранения. RPS всех пользователей проекта по группам: LIST 500, OPTIONS 1500, GET/HEAD по 2500, DELETE 750, DELETE-MULTIPLE 50, PUT 1500. Превышение возвращает **HTTP 429, код SlowDown, сообщение Rate limit exceeded**. [Квоты/лимиты](https://mws.ru/docs/cloud-platform/storage/general/quotas-limits.html).

PUT/веб-загрузка до 5 ГБ, multipart-объект до 5 ТБ; части 5 МБ–5 ГБ, 10000 частей; метаданные 2 КБ. **[предположение]** `max_put_bytes:5000000000` — консервативная десятичная граница; точное число байт и HTTP/XML-коды превышения объёма/числа бакетов/размера объекта не перечислены и требуют живой проверки, не подменяются AWS-кодами. [Лимиты](https://mws.ru/docs/cloud-platform/storage/general/quotas-limits.html).

## Нужна живая проверка

HMAC-ключ владельца; endpoint/регион, условные запросы, ACL, все классы/цены, presign включая нижнюю/верхнюю границу, bucket API и квоты/точные XML-ошибки. Docker/versitygw проверяет общий код, а не MWS Cloud Platform.
