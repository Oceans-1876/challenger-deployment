> Tested with docker v1.25, docker-compose v20.10, postgres v14.5, postgis v3.4.


## Set up
1. On your server, create a folder for the project, e.g. `oceans1876` and put the included `docker-compose.yml` file in it. This file has configuration for a PostgreSQL service. If you are using your own instance of postgres, you can remove this block.
    > For the following steps, we assume you are working in the project folder you created in this step.
    > You can adjust the values in `docker-compose.yml` as you want. The current config, exposes two ports: the API on port `8000` and the web app on port `8080`.
2. Create a folder named `volumes`. This is used to store all mounted volumes in the docker containers.
3. Clone the data repo in `volumes/challenger-data`: `git clone https://github.com/Oceans-1876/challenger-data.git`.
4. Clone the fonts repo in `volumes/maplibre-glyphs`: `git clone https://github.com/Oceans-1876/maplibre-glyphs.git`.
5. Create a `.env` file in the project folder using the provided `dotenv_example` file. You need to adjust the following values in the file:
    - `SERVER_HOST`: API url
    - `BACKEND_CORS_ORIGINS`: list of urls that are allowed to call the API.
    - `SECRET_KEY`: a random string used for cryptographic signing.
    - `FIRST_SUPERUSER`: email for the API admin.
    - `FIRST_SUPERUSER_PASSWORD`: password for the API admin.
    - `POSTGRES_USER`: postgres instance user.
      > If you are using the docker service for postgres, this value will be set and used as the postgres user in the container.
    - `POSTGRES_PASSWORD`: postgres instance password.
      > If you are using the docker service for postgres, this value will be set and used as the password for postgres user in the container.
    - `POSTGRES_DB`: name of the database.
      > If you are using the docker service for postgres, a database with the provided name will be created the first time the service runs.
    - Email settings: adjust accordingly.
6. Start the docker services: `docker-compose up`.

# First run
After the first run, you need to configure and populate the database.

1. Connect to the configured postgres database with the credentials set in `.env`. Then run the followings:
    > If you are using the docker service for postgres, you can connect to the database with this command: `docker-compose exec -u [POSTGRES_USER] db psql -d [POSTGRES_DB]` (replace `[POSTGRES_USER]` and `[POSTGRES_DB]` with the values in `.env`).
    - Make sure postgis is installed: `SELECT postgis_version();`.
      - If postgis is not installed, install it with `CREATE EXTENSION postgis;`
    - Install `pg_trgm` extension: `CREATE EXTENSION pg_trgm;`.
    - You can now disconnect from the database by entering `\q`.
2. Connect to the API docker service: `docker-compose exec api bash`. Then in the opened shell, run the following commands:
    - `./scripts/migrations_forward.sh`: creates the necessary tables in the database.
    - `./scripts/import_data.sh`: Imports data from the data repo cloned in step 3 of **Set Up** section.
    - You can now close the API shell session: `exit`.

At this point, you should have data in the database, and the API should be accessible locally.

Test the API with `curl localhost:8000:api/v1/data_source/`. It should return a JSON response, containing list of data sources from the database.

Test the web app by calling `curl localhost:8080`, which returns the landing page of the app.
