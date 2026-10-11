# MWS S3 MCP Server & AI CLI

A Go MCP server, CLI and installable agent skill for MWS S3 Object Storage. Use Claude Code, Cursor and Codex to inspect objects, recover backups and work with scoped bucket prefixes. Cloud Platform, storage.mwsapis.ru; ACL is disabled. Read-only MCP access is the default. Independent permission levels control writes; destructive operations use the configured approval policy.

[Русское руководство](README.md) · [Russian S3 MCP server directory](https://stufently.github.io/s3-mcp-ai-cli-kit/en/)

## Features

Streaming object CRUD, copy, multipart and presigned URLs follow the profile. Versions, tags, CORS, policy and Object Lock tools require their capabilities. Independent permissions, bounded analytics, diagnostics and installers preserve the shared safety contract.

## Quick start

This profile targets **MWS Cloud Platform** (`storage.mwsapis.ru`, `ru-central1`). Use a service account HMAC access key/secret through `auth set`. Request presigned URLs for 60–604800 seconds.

Use a platform binary from Releases once published, or build with Go 1.27.2:

```bash
bash scripts/go.sh go build -o mws-s3 ./cmd/mws-s3
./mws-s3 version
./mws-s3 auth set
./mws-s3 auth status
./mws-s3 ls --json
```

auth set reads the key ID and secret without terminal echo; non-TTY input supplies two lines. It atomically creates ~/.config/mws-s3/credentials with mode 0600; broader permissions fail. Do not pass secrets in argv.

Resolution: flags > env > file. Variables: MWS_S3_ACCESS_KEY_ID, MWS_S3_SECRET_ACCESS_KEY, MWS_S3_POOL (ru-central1), MWS_S3_ENDPOINT. HTTP requires MWS_S3_ALLOW_INSECURE=1.

After publication: ghcr.io/stufently/mws-s3-mcp-ai-cli:0.1.0. Build locally with docker build -t mws-s3 .; the scratch image runs MCP as uid 65532 with CA certificates. Supply a private credentials environment/file setup.

## Connect to Claude Code, Cursor, Codex

```bash
mws-s3 mcp install --client cursor --dry-run
mws-s3 mcp install --client cursor
mws-s3 mcp install --client claude-desktop
mws-s3 mcp install --client codex
mws-s3 mcp install --client claude-code --scope project
mws-s3 mcp install --client claude-code --scope user
```

The final command only prints claude mcp add. Executable paths are absolute; secrets stay out of client configuration. Cursor uses ~/.cursor/mcp.json, Claude Code project scope ./.mcp.json, Desktop its OS-specific directory. JSON updates preserve foreign keys, create .bak and are idempotent.

Codex uses ~/.codex/config.toml. Existing tables require --force and must have generated # >>> mws-s3 (managed) / # <<< mws-s3 (managed) markers. Unmanaged tables are refused even with --force; other TOML bytes stay intact.

mws-s3 mcp --allow-write --allow-delete --allow-admin or MWS_S3_ALLOW=write,delete,admin enables independent levels. These flags also work for mcp install. File tools require MWS_S3_FILE_ROOT.

## MCP tools

| Level | Tools |
|---|---|
| read | s3_list_buckets, s3_list_objects, s3_head_object, s3_get_object, s3_preview_object, s3_presign_get, s3_list_object_versions, s3_list_multipart_uploads, s3_list_parts, s3_get_bucket_config, s3_largest_objects, s3_prefix_usage, s3_unfinished_uploads, s3_get_object_tagging |
| read + FILE_ROOT | s3_download_file |
| write | s3_put_object, s3_create_folder, s3_copy_object, s3_presign_put, s3_multipart_create, s3_multipart_presign_part, s3_multipart_complete, s3_put_object_tagging |
| write + FILE_ROOT | s3_upload_file |
| delete | s3_delete_object, s3_delete_objects, s3_multipart_abort |
| admin | s3_create_bucket, s3_put_bucket_versioning, s3_put_bucket_cors, s3_put_bucket_policy, s3_delete_bucket_policy, s3_put_object_lock_configuration |
| admin + delete | s3_delete_bucket |

Resources: `mws-s3://config`, `mws-s3://capabilities`, `mws-s3://pools`. Disabled capabilities remove tools.

Lists return at most 1000 items plus continuation tokens; versions/multipart have secondary markers. Inline get/put is limited to 262144 bytes; preview to 65536. Use presigned URLs or FILE_ROOT transfers for larger objects. Responses include structuredContent and JSON text; text over 256 KiB is summarized. Errors have code/message/retryable; the SDK adaptively retries 429/503.

Analytics scan max_scan entries (default 10000), marking partial results and tokens. Prefix aggregates page via next_result_offset. They are not billing estimates.

## Use case: HMAC access checks and short-lived archive URLs

After saving the service account HMAC credentials, check access and create a short-lived URL for an existing MWS Cloud Platform archive. This example uses 60 seconds, the lower bound documented in the research:

```bash
mws-s3 auth status
mws-s3 doctor --json
mws-s3 presign backups/daily/archive.tar --expires 60
```

Examples assume an existing `backups` bucket and `daily/archive.tar` object; replace them with your own names. Cloud compatibility still requires [live verification](TASKS.md).

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

cp uses ordinary local paths and s3://bucket/key or mws-s3://bucket/key URIs. Large uploads use manager/multipart with configurable part size. --recursive copies directories/prefixes; overwrites require --overwrite and approval. Downloads are atomic. rm/rb/mv without --yes exit 2 on non-TTY input; terminals prompt. --json is global; errors use stderr and nonzero exits.

doctor checks credential source/permissions, DNS/TLS, Date-header clock skew and ListBuckets. Skew >5 minutes warns; >15 minutes fails.

## Agent skill

```bash
mws-s3 skill install
mws-s3 skill install --target ./agent-skills/mws-s3
```

Default: ~/.claude/skills/mws-s3. Idempotently installs SKILL.md and six references: backup/restore, sharing, least privilege, cost hygiene, migration and incidents. Triggers include mws s3, presigned and Russian bucket/upload/backup requests.

## Security

With `conditional_writes:false`, default writes refuse before S3 requests because HEAD alone is not atomic. An approved write requires CLI `--overwrite --yes` or MCP `overwrite:true, confirm:true` under the approval policy. With `conditional_delete:false`, `mv` and `s3_move_object` are disabled.



Read is the MCP default. Missing capabilities/permissions remove tools. MWS_S3_DESTRUCTIVE_POLICY=confirm uses client form elicitation when available, otherwise confirm:true after human approval. elicit requires elicitation and cannot be bypassed with confirm:true. block always refuses; allow proceeds.

Gates precede all S3 requests for deletion, move, signed PUT, policy/Object Lock, versioning suspension and overwrite. With conditional_writes:true, put/copy/move/multipart complete/upload file use overwrite:false with target HEAD and If-None-Match: *; otherwise default writes refuse. overwrite:true uses the same gate.

MWS_S3_ALLOWED_BUCKETS and MWS_S3_ALLOWED_PREFIXES take comma-separated buckets and bucket/prefix/ entries. Prefixes respect segment boundaries and reject '..'. Partial bucket scope denies bucket administration and broad listings even when the bucket is also listed in ALLOWED_BUCKETS. Copy/move checks both sides.

FILE_ROOT MCP tools use os.Root and relative paths only; absolute paths, '..' and escaping symlinks fail. Every tool call emits one JSON stderr audit record; MWS_S3_AUDIT_FILE additionally appends to a 0600 file. Secrets and X-Amz-Signature/Credential/Security-Token are redacted from logs/errors. Full signed URLs are operation results only. Keep underlying S3 key rights minimal too.

s3_download_file publishes downloads without replacing existing local files by default, including files created during the download. To replace a file, pass overwrite:true; the destructive policy applies, using form confirmation or confirm:true after human approval.

## MWS pools and storage classes

See [compatibility](docs/compat.md) for pools, endpoints and capabilities, and [official-source research](docs/research/mws-s3.md) for classes, pricing, limits and errors. Default signing pool: `ru-central1`. The profile remains `verified:false`: an emulator cannot establish live-provider support. Unverified facts are marked **[предположение]**. Profile prices are not a billing calculator.

Lifecycle, ACL, bucket tagging and retention may be cloud features, but this binary does not expose management tools for them; the profile records S3 API compatibility.

## FAQ

**Which MWS product does this MCP server target?** The profile targets **MWS Cloud Platform**, endpoint `storage.mwsapis.ru`, signing region `ru-central1`. A different product with a similar name needs its own profile.

**Which credentials does MWS S3 require?** The Access key/Secret key pair of a service account HMAC key. Enter it with `mws-s3 auth set`; [MWS research](docs/research/mws-s3.md) links the official setup. Never put keys in command arguments.

**Why does the short-lived MWS URL example use 60 seconds?** The research cites a documented 60–604800 second range. Use at least 60; actual behavior and maximum lifetime still require live verification.

**What is an MCP server?** A program exposes agent tools/resources through Model Context Protocol; this server uses stdio.

**How do I connect Claude Code?** Save keys with auth set and use project install or the printed user command.

**Is agent S3 access safe?** Restrict keys, scope and permissions; retain confirm/block and deliberately approve destructive effects.

**How does it differ from aws-cli/rclone?** It combines MCP, gates, audit, analytics, skill and CLI; those tools remain independently useful for transfer/automation.

**Which features are available?** Read capabilities and [compat](docs/compat.md). Registration follows the profile; this version does not configure lifecycle, ACL, bucket tagging or retention.


## Related S3 MCP servers

[Browse the full provider directory](https://stufently.github.io/s3-mcp-ai-cli-kit/en/).

| Provider | Repository | Use case |
|---|---|---|
| Selectel | [selectel-s3-mcp-ai-cli](https://github.com/stufently/selectel-s3-mcp-ai-cli) | Pool selection and backup recovery |
| Yandex Cloud | [yandex-s3-mcp-ai-cli](https://github.com/stufently/yandex-s3-mcp-ai-cli) | Object inventory and backup version inspection |
| VK Cloud | [vkcloud-s3-mcp-ai-cli](https://github.com/stufently/vkcloud-s3-mcp-ai-cli) | Hotbox/Icebox inspection and regional access |
| Cloud.ru | [cloudru-s3-mcp-ai-cli](https://github.com/stufently/cloudru-s3-mcp-ai-cli) | Evolution Object Storage access diagnostics |
| Timeweb Cloud | [timeweb-s3-mcp-ai-cli](https://github.com/stufently/timeweb-s3-mcp-ai-cli) | Archive recovery and bounded prefix usage |
| MWS | [mws-s3-mcp-ai-cli](https://github.com/stufently/mws-s3-mcp-ai-cli) | HMAC access checks and short-lived archive URLs |
| Beget | [beget-s3-mcp-ai-cli](https://github.com/stufently/beget-s3-mcp-ai-cli) | Browse and restore files from an existing bucket |
| Shared Go kit | [s3-mcp-ai-cli-kit](https://github.com/stufently/s3-mcp-ai-cli-kit) | Shared MCP, CLI, safety and CI |

## License

[MIT](LICENSE). [Facts and assumptions](docs/research/mws-s3.md). Docker checks: bash scripts/go.sh go test -race ./..., bash scripts/conformance.sh, bash scripts/mcp_smoke.sh, bash scripts/mutation_gate.sh, bash scripts/check_docs.sh. Conformance uses versitygw v1.8.0 and fake keys.

Shared MCP/CLI behavior, safety and checks come from [s3-mcp-ai-cli-kit v0.1.0](https://github.com/stufently/s3-mcp-ai-cli-kit/tree/v0.1.0); this repository owns its embedded profile, thin main and MWS documentation.
