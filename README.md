# Keycloak Sandbox

This project provides a development environment for Keycloak using Docker Compose, with an optional Postgres database. It uses `direnv` to manage environment variables, allowing flexible configuration for local development.

## Project Structure

```
.
├── .envrc                # direnv configuration for environment variables
├── .gitignore            # Git ignore file
├── docker
│   └── docker-compose.yaml  # Docker Compose configuration
└── README.md             # Project documentation
```

## Environment Variables

Environment variables are defined in `.envrc` and managed by `direnv`. They configure the Postgres and Keycloak services. Below are the key variables, their purposes, and default values:

| Variable                  | Description                                                                 | Default Value                         |
|---------------------------|-----------------------------------------------------------------------------|---------------------------------------|
| `POSTGRES_DB`             | Name of the Postgres database.                                            | `keycloak`                            |
| `POSTGRES_USER`           | Postgres user for Keycloak.                                               | `keycloak`                            |
| `POSTGRES_PASSWORD`       | Password for the Postgres user.                                           | `localdev`                            |
| `POSTGRES_EXTERNAL_PORT`  | Host port for the Dockerized Postgres service (to avoid conflicts).        | `35432`                               |
| `KEYCLOAK_DB_URL`         | JDBC URL for Keycloak’s database connection (set to external DB if needed).  | `jdbc:postgresql://postgres/keycloak` |
| `KEYCLOAK_ADMIN`          | Keycloak admin username.                                                    | `admin`                               |
| `KEYCLOAK_ADMIN_PASSWORD` | Keycloak admin password.                                                    | `localdev`                            |
| `KEYCLOAK_EXTERNAL_PORT`  | Host port for Keycloak’s web interface.                                     | `30080`                               |

### Using `direnv`

`direnv` automatically loads environment variables from `.envrc` when you enter the project directory. To set up:

1. Install `direnv`:
   ```bash
   brew install direnv
   ```
   Add `eval "$(direnv hook bash)"` (or equivalent for your shell) to `~/.bashrc` or `~/.zshrc`.

2. Create or edit `.envrc` in the project root. Example:
   ```bash
   export POSTGRES_DB=keycloak
   export POSTGRES_USER=keycloak
   export POSTGRES_PASSWORD=localdev
   export POSTGRES_EXTERNAL_PORT=35432
   export KEYCLOAK_DB_URL=jdbc:postgresql://postgres/keycloak
   export KEYCLOAK_ADMIN=admin
   export KEYCLOAK_ADMIN_PASSWORD=localdev
   export KEYCLOAK_EXTERNAL_PORT=30080
   ```

3. Run `direnv allow` to authorize the `.envrc` file.

Customize `.envrc` to override defaults (e.g., for an external database).

## Starting the Application

You can run Keycloak with either the Dockerized Postgres or an external Postgres database. The `docker-compose.yaml` is located in the `docker/` directory.

### Option 1: Using Dockerized Postgres (Default)

The Dockerized Postgres service is included in the `postgres` profile and starts when explicitly enabled.

1. Ensure `.envrc` uses the default `KEYCLOAK_DB_URL` (pointing to `postgres` service):
   ```bash
   export KEYCLOAK_DB_URL=jdbc:postgresql://postgres/keycloak
   ```

2. Run `direnv allow` to load environment variables.

3. Start both services with the `postgres` profile:
   ```bash
   docker-compose -f docker/docker-compose.yaml --profile postgres up
   ```

    - Postgres runs on `localhost:35432` (or your `POSTGRES_EXTERNAL_PORT`).
    - Keycloak runs on `localhost:30080` (or your `KEYCLOAK_EXTERNAL_PORT`).

4. Access Keycloak at `http://localhost:30080` and log in with the admin credentials (`KEYCLOAK_ADMIN`/`KEYCLOAK_ADMIN_PASSWORD`).

### Option 2: Using an External Postgres Database

To use a local or external Postgres database (e.g., running on `localhost:5432`), configure the connection and skip the `postgres` service.

#### Step 1: Set Up the External Database
Run the following commands to configure a local Postgres instance:

```bash
psql -Atx postgres://postgres@localhost -c "create user keycloak with password 'localdev' CREATEDB;"
psql -Atx postgres://keycloak@localhost/postgres -c "create database keycloak with owner keycloak;"
psql -Atx postgres://keycloak@localhost/keycloak -c "create schema keycloak;"
```

Adjust the username, password, and database name to match your `.envrc` settings if customized.

#### Step 2: Configure `.envrc`
Update `.envrc` to point to the external database. Example for a local Postgres:

```bash
export POSTGRES_PASSWORD=localdev
export KEYCLOAK_DB_URL=jdbc:postgresql://host.docker.internal:5432/keycloak
export KEYCLOAK_ADMIN=admin
export KEYCLOAK_ADMIN_PASSWORD=localdev
export KEYCLOAK_EXTERNAL_PORT=30080
```

- Set `KEYCLOAK_DB_URL` to `jdbc:postgresql://host.docker.internal:5432/keycloak` (use `host.docker.internal` to refer to the host DNS name with Docker desktop, for example).
- Run `direnv allow`.

#### Step 3: Start Keycloak Only
Start only the `keycloak` service to avoid running the Dockerized Postgres:

```bash
docker-compose -f docker/docker-compose.yaml up
```

Keycloak will connect to the external database specified in `KEYCLOAK_DB_URL`.

#### Step 4: Verify
Access Keycloak at `http://localhost:30080` and confirm it connects to the external database.

## Notes

- **Port Conflicts**: Ensure `POSTGRES_EXTERNAL_PORT` (default: `35432`) and `KEYCLOAK_EXTERNAL_PORT` (default: `30080`) don’t conflict with local services. Adjust in `.envrc` if needed.
- **External Database**: Verify the external Postgres is accessible (e.g., `psql -h localhost -U keycloak -d keycloak`) before starting Keycloak.
- **Stopping Services**: Use `docker-compose -f docker/docker-compose.yaml down` to stop and remove containers.

## Troubleshooting

- **Keycloak fails to connect to Postgres**: Check `KEYCLOAK_DB_URL`, `POSTGRES_USER`, and `POSTGRES_PASSWORD`. Ensure the database and schema exist.
- **Port conflicts**: Change `POSTGRES_EXTERNAL_PORT` or `KEYCLOAK_EXTERNAL_PORT` in `.envrc`.
- **direnv not loading**: Run `direnv allow` and verify your shell is configured (e.g., `eval "$(direnv hook bash)"` in `~/.bashrc`).
