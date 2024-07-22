# ELT Pipeline with Docker and DBT

This project sets up an ELT (Extract, Load, Transform) pipeline using Docker. The pipeline consists of two PostgreSQL databases (source and destination) and a Python script that performs the ELT operations.

### Files

- `docker-compose.yml`: Defines the services for the source and destination PostgreSQL databases and the ELT script.
- `elt/Dockerfile`: Dockerfile for building the image that runs the ELT script.
- `elt/elt_script.py`: Python script that performs the ELT operations.
- `source_db_init/init.sql`: SQL script to initialize the source PostgreSQL database.
- `custom_postgres/`: Contains dbt models, macros, and configurations for the Transform phase.

## Prerequisites

- Docker and Docker Compose installed on your system.
- `dbt` installed (handled via Docker in this setup).

## Setup and Run

1. Clone the repository and navigate to the project directory.
2. Run the following command to build the Docker images and start the services:
   ```sh
   docker-compose up --build
   ```

## Access Destination Database

1. docker exec -it elt-destination_postgres-1 psql -U postgres
2. \c destination_db
3. \dt

Feel free to run queries such as:

- SELECT \* FROM films;

## Access Source Database

1. docker exec -it elt-source_postgres-1 psql -U postgres
2. \c destination_db
3. \dt

## Services

- `source_postgres`: PostgreSQL instance for the source database.
- `destination_postgres`: PostgreSQL instance for the destination database.
- `elt_script`: Custom service that runs the Python ELT script.
- `dbt`: Runs dbt commands to execute the Transform phase.

### ELT Script

The ELT script performs the following steps:

1. **Wait for PostgreSQL**: Ensures the source PostgreSQL database is ready.
2. **Extract**: Dumps data from the source PostgreSQL database.
3. **Load**: Loads the dumped data into the destination PostgreSQL database.

### dbt Transformations

In this project, dbt handles the Transform phase of the ELT pipeline. The key components include:

**Macros**:

- `macros/film_ratings_macro.sql`: Defines a macro `generate_film_ratings` used to categorize film ratings based on user ratings. This macro is applied in the transformation logic to create meaningful rating categories.

**Models**:

- `models/example/actors.sql`: Extracts data from the actors table in the destination database.
- `models/example/film_actors.sql`: Extracts data from the `film_actors` table.
- `models/example/film_ratings.sql`: Combines the ratings and actors data to produce the final result. Uses CTEs (Common Table Expressions) to apply transformations:
  - `films_with_ratings`: Applies the `generate_film_ratings` macro to classify films based on user ratings.
  - `films_with_actors`: Joins film data with actor data to provide a complete view of each film.
- `models/example/films.sql`: Extracts data from the `films` table.
- `models/example/specific_movie.sql`: Uses Jinja to dynamically filter the films model based on a specific movie title

### Environment Variables

- `POSTGRES_DB`: Database name.
- `POSTGRES_USER`: Database user.
- `POSTGRES_PASSWORD`: Database password.

### Network

All services are connected to a custom bridge network (`elt_network`).
