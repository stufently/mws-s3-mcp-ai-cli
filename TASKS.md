# Статус

- Завершено: ресерч официальной документации и консервативный профиль MWS Cloud Platform; README/compat/Registry/skill/Docker/release/CI.
- Завершено: Docker build/vet/race/lint (0 issues), versitygw conformance, MCP smoke (все 8 сочетаний уровней), мутационные ворота 87/87 (100%) и self-tests, check_docs, actionlint, GoReleaser snapshot; образ uid 65532 запущен.
- Доставка завершена: main опубликован, описание/topics настроены; [CI реализации `e6a45df`](https://github.com/stufently/mws-s3-mcp-ai-cli/actions/runs/38066059373) зелёный.
- Нужна живая проверка: эндпоинты/регионы, условные заголовки, классы/цены, presign, API и ошибки на реальном тестовом бакете с ключом владельца.
- Релиз v0.1.0 и публикация OCI/Registry отложены до живой проверки.

- **[предположение]**: атомарные условия записи/удаления, ACL, точные байтовые границы/коды квот, репликация классов и окончательная цена/free tier аккаунта; подробно в [ресерче](docs/research/mws-s3.md).
