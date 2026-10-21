# Laravel Sail Docker Image

Custom Docker images for running [Laravel](https://laravel.com/) applications, designed to be a drop-in replacement for [Laravel Sail](https://laravel.com/docs/sail). Built on Ubuntu 22.04 with the Ondřej Surý PHP PPA for reliable, up-to-date PHP releases.

Images are published to Docker Hub as [`realworldio/laravel-sail`](https://hub.docker.com/r/realworldio/laravel-sail).

---

## Supported PHP Versions

| Tag | PHP Version | Status |
|-----|-------------|--------|
| `8.3` | PHP 8.3 | Active |
| `8.2` | PHP 8.2 | Active |
| `8.1` | PHP 8.1 | Active |
| `8.0` | PHP 8.0 | Active |
| `7.4` | PHP 7.4 | Legacy |

---

## Laravel Compatibility

| Laravel Version | Minimum PHP | Recommended Image Tag |
|-----------------|-------------|----------------------|
| Laravel 12 | PHP 8.2+ | `8.3` |
| Laravel 11 | PHP 8.2+ | `8.3` |
| Laravel 10 | PHP 8.1+ | `8.2` |
| Laravel 9  | PHP 8.0+ | `8.1` |
| Laravel 8  | PHP 7.3+ | `8.0` |

---

## Quick Start

### docker-compose.yml

```yaml
version: '3.7'
services:
    laravel:
        image: realworldio/laravel-sail:8.3
        ports:
            - '${APP_PORT:-80}:80'
        env_file: ./.env
        environment:
            WWWUSER: '${WWWUSER}'
            LARAVEL_SAIL: 1
            XDEBUG_MODE: '${SAIL_XDEBUG_MODE:-off}'
            XDEBUG_CONFIG: '${SAIL_XDEBUG_CONFIG:-client_host=host.docker.internal}'
        volumes:
            - '.:/var/www/html'
        networks:
            - sail

networks:
    sail:
        driver: bridge
```

### Pull an image

```bash
docker pull realworldio/laravel-sail:8.3
```

### Run a container

```bash
docker run -it --rm \
    -v $(pwd):/var/www/html \
    -p 80:80 \
    -e WWWUSER=$(id -u) \
    -e WWWGROUP=$(id -g) \
    realworldio/laravel-sail:8.3
```

---

## Included Extensions

All images ship with the following PHP extensions:

| Category | Extensions |
|----------|-----------|
| **Database** | pgsql, sqlite3, mysql, pdo |
| **Caching** | redis, memcached, igbinary, msgpack |
| **Web** | curl, soap, xml, intl |
| **Encoding** | mbstring, json, imap |
| **Math** | bcmath, gmp (8.2+) |
| **Graphics** | gd |
| **Archive** | zip |
| **Async** | swoole |
| **Debug / Coverage** | xdebug, pcov |
| **Auth** | ldap |
| **Misc** | readline |

### Development Tools

- **Composer** (latest stable)
- **Node.js** 16.x
- **Yarn**
- **MySQL Client**
- **PostgreSQL Client**
- **Git**
- **Supervisor**

---

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `WWWUSER` | `1000` | UID of the `sail` user inside the container |
| `WWWGROUP` | `1000` | GID of the `sail` group inside the container |
| `LARAVEL_SAIL` | `1` | Set by Supervisor when running `php artisan serve` |
| `XDEBUG_MODE` | `off` | Xdebug mode (`off`, `debug`, `coverage`, etc.) |
| `XDEBUG_CONFIG` | _(empty)_ | Additional Xdebug configuration |

---

## PHP Configuration

Custom PHP settings are applied via `/etc/php/<version>/cli/conf.d/99-sail.ini`:

```ini
post_max_size = 100M
upload_max_filesize = 100M
variables_order = EGPCS
```

Override these by mounting your own `.ini` file into the same directory.

---

## Container Behaviour

- **Working directory**: `/var/www/html`
- **Default port**: `8000` (exposed), Supervisor runs `php artisan serve` on port `80`
- **Process manager**: Supervisor
- **Application server**: `php artisan serve --host=0.0.0.0 --port=80`
- **Task scheduler**: Cron runs `php artisan schedule:run` every minute
- **Non-root user**: `sail` (UID 1000); the entrypoint adjusts IDs at runtime when `WWWUSER`/`WWWGROUP` are set

### Overriding the entrypoint

Pass any command as arguments to run it as the `sail` user via `gosu`:

```bash
docker run --rm \
    -v $(pwd):/var/www/html \
    realworldio/laravel-sail:8.3 \
    php artisan migrate
```

---

## Building Locally

```bash
# Build a specific PHP version
docker build -t laravel-sail:8.3 ./docker/php-8.3

# Build with a custom group ID
docker build --build-arg WWWGROUP=$(id -g) -t laravel-sail:8.3 ./docker/php-8.3
```

All Dockerfiles are located under `docker/php-<version>/`:

```
docker/
├── php-7.4/
│   ├── Dockerfile
│   ├── php.ini
│   ├── start-container
│   └── supervisord.conf
├── php-8.0/
├── php-8.1/
├── php-8.2/
└── php-8.3/
```

---

## CI / CD

Images are automatically built and published to Docker Hub via GitHub Actions:

- **On push** to `master` or version tags (`v*.*.*`)
- **On pull request** to `master` or `build` (build only, no push)
- **Scheduled** daily at 00:30 UTC

Tags follow the pattern `<php-version>` (e.g. `8.3`) and `<php-version>-<semver>` for tagged releases.

---

## Maintainer

Oshane Bailey
