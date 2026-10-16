# Ansible Role: Traefik (Podman Quadlet)

Ansible role to install, configure, and manage [Traefik](https://traefik.io/) v3 as a reverse proxy running in a **Podman Quadlet** container attached to an existing Pod in rootless mode (`user` systemd scope). It supports TLS certificate generation via **Let's Encrypt (ACME)**, global dynamic middlewares (such as HTTP-to-HTTPS redirect, security headers, and IP allowlisting), custom access logging/metrics, SELinux volume relabeling, and systemd user lingering.

## Requirements

* **Ansible Version:** `2.14` or higher
* **OS Platforms:**
  * Fedora (`42`, `43`, `44`)
  * Enterprise Linux / EL (`8`, `9`)
  * Ubuntu (`all`)
* **Podman:** Quadlet requires Podman 4.4+
* **Existing Pod:** The target Pod (`{{ traefik_pod_name }}.pod`) must be created prior to running this role (e.g., via `lxtnow.podman_pod`).

## Dependencies

* `lxtnow.podman_pod` (`git@github.com:laxathom/ansible-role-podman-pod.git`)

## Role Variables

The defaults are defined in `defaults/main.yml`:

| Variable Name | Default Value | Description |
| :--- | :--- | :--- |
| `traefik_user` | `traefik` | System user running the rootless Podman container |
| `traefik_group` | `traefik` | System group for the Traefik user |
| `traefik_shell` | `/bin/bash` | Shell assigned to the Traefik user |
| `traefik_config_dir` | `"/home/{{ traefik_user }}/.config/traefik"` | Host directory for static and dynamic Traefik configurations |
| `traefik_data_dir` | `"/home/{{ traefik_user }}/.local/share/traefik"` | Host directory storing `acme.json` (TLS certificates) |
| `traefik_target_config_dir` | `/etc/traefik` | Container mount path for configuration files |
| `traefik_pod_name` | `traefik` | Target Podman Pod name |
| `traefik_container_name` | `traefik` | Traefik container name |
| `traefik_container_image` | `docker.io/library/traefik:v3.3.4` | Container image URL and tag |
| `traefik_tls_resolver` | `letsencrypt` | Traefik ACME Certificate Resolver name |
| `traefik_tls_admin_email` | `admin@example.com` | Email address registered with Let's Encrypt |
| `traefik_http_challenge` | `httpChallenge: { entryPoint: web }` | ACME challenge type settings (HTTP challenge) |
| `traefik_providers` | `file: { directory: "...", watch: true }` | Traefik providers definition |
| `traefik_log_level` | `INFO` | Traefik binary log level (`DEBUG`, `INFO`, `WARN`, `ERROR`) |
| `traefik_accesslog_format` | `common` | Access log output format (`common` or `json`) |
| `traefik_volumes` | *(See defaults/main.yml)* | List of volume mounts with SELinux flags (`ro,Z`, `ro,z`, `rw,Z`) |
| `traefik_allowed_ips` | `[0.0.0.0/0]` | List of allowed IPs/CIDRs for the IP allowlist middleware |
| `traefik_trusted_ips` | `[127.0.0.1/32]` | List of trusted IP ranges for forwarded headers |

## Configured Templates & Features

* **Quadlet Unit (`traefik.container.j2`):** Deploys a systemd `.container` unit mounting static configs, dynamic configs, and `acme.json` with appropriate SELinux volume options (`:Z`, `:z`).
* **Static Configuration (`traefik.yml.j2`):**
  * Entrypoints for HTTP (`:80`) and HTTPS (`:443`) with automatic HTTP-to-HTTPS redirect.
  * Trusted IP ranges configuration for forwarded headers.
  * ACME / Let's Encrypt certificates resolver configuration.
  * Configurable log levels and access logs.
* **Dynamic Middlewares (`config.yml.j2`):**
  * `redirect-to-https`: Global scheme redirection.
  * `ip-allowlist`: Source range IP filtering.
  * `secure-headers`: HSTS and SSL redirect header policies.

## Example Playbook

```yaml
- name: Deploy Podman Pod and Traefik
  hosts: reverse_proxies
  become: true
  roles:
    - role: laxathom.podman_pod
      vars:
        pod_name: traefik
        pod_ports:
          - "80:80"
          - "443:443"

    - role: laxathom.traefik
      vars:
        traefik_tls_admin_email: "devops@example.com"
        traefik_container_image: "docker.io/library/traefik:v3.3.4"
        traefik_allowed_ips:
          - "192.168.1.0/24"
          - "10.0.0.0/8"
        traefik_trusted_ips:
          - "127.0.0.1/32"
          - "10.0.0.0/8"
```

## License

MIT

## Author Information

- **Author:** laxathom
