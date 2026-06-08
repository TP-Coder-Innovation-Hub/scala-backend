# Database Migrations ``

Databases evolve. New tables, new columns, changed constraints. Migrations track these changes as versioned, repeatable scripts. Never modify a database schema manually. Always use migrations.

## Flyway — The Standard

Flyway is the most widely used migration tool in the JVM ecosystem. It works with any JVM language including Scala.

### Setup

```scala
libraryDependencies += "org.flywaydb" % "flyway-core" % "10.13.1"
```

### Migration Files

Place SQL files in `src/main/resources/db/migration/`. Name them with the pattern `V{version}__{description}.sql`:

```
src/main/resources/db/migration/
  V1__create_users.sql
  V2__add_orders.sql
  V3__add_email_index.sql
```

`V1__create_users.sql`:

```sql
CREATE TABLE users (
  id    BIGSERIAL PRIMARY KEY,
  name  VARCHAR(255) NOT NULL,
  email VARCHAR(255) NOT NULL UNIQUE
);
```

`V2__add_orders.sql`:

```sql
CREATE TABLE orders (
  id       BIGSERIAL PRIMARY KEY,
  user_id  BIGINT NOT NULL REFERENCES users(id),
  total    DECIMAL(10,2) NOT NULL,
  status   VARCHAR(50) NOT NULL DEFAULT 'pending',
  created_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_orders_user_id ON orders(user_id);
```

`V3__add_email_index.sql`:

```sql
CREATE INDEX idx_users_email ON users(email);
```

### Running Migrations

```scala
import org.flywaydb.core.Flyway

val flyway = Flyway.configure()
  .dataSource("jdbc:postgresql://localhost:5432/mydb", "postgres", "password")
  .load()

flyway.migrate()  // Runs all pending migrations in order
```

Flyway tracks applied migrations in a `flyway_schema_history` table. It runs only new migrations. Already-applied migrations are never re-run.

### Integration with Cats Effect

```scala
import cats.effect.*
import org.flywaydb.core.Flyway

def runMigrations(url: String, user: String, pass: String): IO[Unit] =
  IO.blocking {
    val flyway = Flyway.configure()
      .dataSource(url, user, pass)
      .load()
    flyway.migrate()
  }.void

// Run migrations before starting your server
object Main extends IOApp.Simple:
  val run =
    runMigrations("jdbc:postgresql://localhost:5432/mydb", "postgres", "password") *>
    startServer
```

### sbt Integration

Run migrations from sbt during development:

```scala
// project/plugins.sbt
addSbtPlugin("org.flywaydb" % "flyway-sbt" % "4.2.0")
```

Configure in `build.sbt`:

```scala
flywayUrl := "jdbc:postgresql://localhost:5432/mydb"
flywayUser := "postgres"
flywayPassword := "password"
flywayLocations := Seq("filesystem:src/main/resources/db/migration")
```

Run:

```bash
sbt flywayMigrate     # Apply pending migrations
sbt flywayInfo        # Show migration status
sbt flywayClean       # Drop all objects (DEV ONLY)
```

## Migration Best Practices

1. **Never modify applied migrations.** Once a migration runs, it's immutable. Create a new migration to change something.

2. **Make migrations reversible in dev.** Write `V{version}__{desc}.sql` and optionally `U{version}__{desc}.sql` for rollback.

3. **Keep migrations small.** One concern per migration. A new table. A new column. An index. Not all three in one file.

4. **Test migrations against a real database.** Use Docker or Testcontainers:

```scala
// With testcontainers
import com.dimafeng.testcontainers.PostgreSQLContainer

val container = PostgreSQLContainer()
container.start()
runMigrations(container.jdbcUrl, container.username, container.password)
```

5. **Run migrations on startup.** The application should run migrations before accepting requests. This ensures the schema always matches the code.

## Schema Evolution Pattern

```
V1__create_users.sql        -- Initial schema
V2__add_orders.sql          -- New feature: orders
V3__add_email_index.sql     -- Performance: index
V4__add_user_status.sql     -- New column
V5__rename_status_field.sql -- Rename (requires care)
```

Each migration builds on the last. The migration history tells the story of your schema. Any developer can read the migration files and understand how the database evolved.

Next: [Concurrency](../05-concurrency/01-fibers.md)
