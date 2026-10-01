# ansible-role-vaultwarden

Deploys [Vaultwarden](https://github.com/dani-garcia/vaultwarden) (an unofficial
Bitwarden-compatible server) as a Docker container, fronted optionally by Caddy
for automatic TLS.

## Requirements

- Target host reachable via SSH with a user capable of `become`
- Docker Engine on the target (this role can install it for you, see
  `vaultwarden_manage_docker`)
- Internet access on the target to pull the `vaultwarden/server` image

## Role Variables

See `defaults/main.yml` for the full list with defaults. Key variables:

| Variable | Default | Description |
|---|---|---|
| `vaultwarden_manage_docker` | `true` | Install Docker via `geerlingguy.docker` if not already present |
| `vaultwarden_image` | `vaultwarden/server` | Docker image name |
| `vaultwarden_image_tag` | `latest` | Docker image tag |
| `vaultwarden_data_dir` | `/opt/vaultwarden/data` | Host path for persistent data (SQLite DB, attachments, RSA keys) |
| `vaultwarden_container_name` | `vaultwarden` | Docker container name |
| `vaultwarden_http_port` | `8080` | Host port mapped to the container's internal web port |
| `vaultwarden_domain` | `https://vault.example.com` | Public URL used to build invite/notification links |
| `vaultwarden_signups_allowed` | `false` | Allow new user signups |
| `vaultwarden_invitations_allowed` | `true` | Allow admin-issued invitations |
| `vaultwarden_admin_token` | `""` | Admin panel token (bcrypt hash or plain, see Vaultwarden docs). **Set this via vault** — leave blank to disable the admin panel |
| `vaultwarden_websocket_enabled` | `true` | Enable the WebSocket notifications port |
| `vaultwarden_websocket_port` | `3012` | Host port mapped to the container's WebSocket port |
| `vaultwarden_smtp_*` | see defaults | Outbound mail settings for invites/2FA |
| `vaultwarden_backup_enabled` | `true` | Install a daily cron job that tars up the data dir |
| `vaultwarden_backup_dir` | `/opt/vaultwarden/backups` | Where backups are written |
| `vaultwarden_backup_retention_days` | `14` | How long to keep backups |
| `vaultwarden_manage_caddy` | `false` | Install and configure Caddy as a reverse proxy with automatic HTTPS |
| `vaultwarden_extra_env` | `{}` | Free-form dict merged into the container's environment, for any setting not modeled explicitly |

Sensitive variables (`vaultwarden_admin_token`, `vaultwarden_smtp_password`, ...)
should be provided via `ansible-vault`, not committed in plaintext.

## Example Playbook

```yaml
- hosts: vaultwarden
  become: true
  roles:
    - role: ansible-role-vaultwarden
      vars:
        vaultwarden_domain: "https://vault.example.com"
        vaultwarden_admin_token: "{{ vault_vaultwarden_admin_token }}"
        vaultwarden_signups_allowed: false
        vaultwarden_manage_caddy: true
        vaultwarden_smtp_host: "smtp.example.com"
        vaultwarden_smtp_from: "vaultwarden@example.com"
        vaultwarden_smtp_username: "vaultwarden@example.com"
        vaultwarden_smtp_password: "{{ vault_vaultwarden_smtp_password }}"
```

## Backups

When `vaultwarden_backup_enabled` is true, a cron job runs daily
(`vaultwarden_backup_hour`/`vaultwarden_backup_minute`) and tars
`vaultwarden_data_dir` into `vaultwarden_backup_dir`, pruning archives older
than `vaultwarden_backup_retention_days`. This is a filesystem-level backup;
stop or snapshot-quiesce the container if you need strict consistency, or rely
on SQLite's WAL durability for routine backups.

## License

MIT

## Author

Jason Parks
