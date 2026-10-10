# MWS S3 compatibility

Generated from the embedded provider profile.

| Feature | Supported |
|---|---|
| acl | false |
| bucket_policy | true |
| bucket_tagging | true |
| conditional_delete | false |
| conditional_writes | false |
| cors | true |
| lifecycle | true |
| multipart | true |
| object_lock_config | true |
| object_retention | true |
| object_tagging | true |
| presign | true |
| versioning | true |

| Pool | Endpoint | Location | Multi-AZ | Classes |
|---|---|---|---|---|
| ru-central1 | https://storage.mwsapis.ru | Россия, MWS Cloud Platform | true | STANDARD, WARM, COLD |

| Class | Name | RUB/GB/month (from) | Verified | Checked |
|---|---|---|---|---|
| STANDARD | Стандартный | 2.44 | false | 2026-10-10 |
| WARM | Тёплый | 9.15 | false | 2026-10-10 |
| COLD | Холодный | 1.26 | false | 2026-10-10 |

Presign maximum: 604800 seconds; verified: false. MWS Cloud Platform: сервис документирует срок от 60 до 604800 секунд; используйте не меньше 60. Условные запись/удаление и ACL не подтверждены, нужна живая проверка.
