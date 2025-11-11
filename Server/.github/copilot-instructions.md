Quick context for AI code agents working on MagicStreamMovies (Server)

- Repo type: Go (Go modules). Entry point: `main.go` at repository root.
- Web framework: Gin (router created in `main.go`, listens on :8080).
- DB: MongoDB via official driver (module uses `go.mongodb.org/mongo-driver/v2`). DB helpers live in `database/database_connection.go`.
- Primary packages: `controllers/`, `models/`, `database/`, `routes/`, `middleware/`.

Essential patterns and examples
- Database: `database.DBInstance()` loads `.env` (via `github.com/joho/godotenv`) and reads `MONGODB_URI`; `database.OpenCollection(name)` reads `DATABASE_NAME` and returns a `*mongo.Collection`.
  - Example usage: `var movieCollection *mongo.Collection = database.OpenCollection("movies")` in `controllers/movie_controller.go`.
  - Important env keys: `MONGODB_URI`, `DATABASE_NAME` (ensure `.env` contains them). The DB package initializes a package-level `Client` with `var Client *mongo.Client = DBInstance()`.
- Controllers: each controller exports a `gin.HandlerFunc` constructor (e.g. `GetMovies() gin.HandlerFunc`) and commonly use a package-level collection variable.
  - Controllers use `context.WithTimeout(..., 100*time.Second)` before DB calls, `bson.M{}` for queries, `cursor.All(ctx, &dest)` to decode.
- Models: `models/movie_model.go` defines the data shapes (Movie, Genre, Ranking) with `bson` + `json` tags and `validate` tags — follow those field names when reading/writing DB documents.

Build / run / debug (what works now)
- Quick run: from repo root run `go run main.go`. Server will bind to `:8080`.
- Build binary: `go build -o bin/server main.go` then `./bin/server`.
- Tests: none discovered. Use `go test ./...` if you add tests.
- Common debug steps:
  - Ensure `.env` exists and contains `MONGODB_URI` and `DATABASE_NAME` before starting, because `database.DBInstance()` calls `log.Fatal` if missing.
  - The DB helper may return a nil client on connection errors; check logs and avoid assuming `Client` is non-nil in new code.

Project-specific conventions / gotchas (be explicit)
- Single global DB client: code uses a package-level `Client` and package-level collection variables in controllers. When modifying DB initialization, update call sites accordingly.
- Controllers return `gin.HandlerFunc` (not direct functions); attach handlers in `main.go` using `router.GET("/movies", controller.GetMovies())`.
- Timeouts: controllers use 100s context timeout for DB operations — keep this consistent unless you have a clear reason.
- Validation: model structs include `validate` tags but no validation helper is present in the codebase; if adding validation, follow existing tags (`validate:"required, min=2"`, etc.).

Integration points and dependencies
- External packages to be aware of:
  - `github.com/joho/godotenv` — `.env` loader used in `database/database_connection.go`.
  - `go.mongodb.org/mongo-driver/v2` — official Mongo driver used for `mongo.Client`, `Collection`, `bson` types.
  - `github.com/gin-gonic/gin` — web framework.
- Module import path used in code: `github.com/Sadik-Sami/MagicStreamMovies/Server` (used in `main.go` and controllers). Keep import paths consistent when moving files.

How to make safe edits (recommendations specific to this codebase)
- Before editing handlers, run the server and exercise `/` and `/movies` to confirm behavior.
- If you change DB init, ensure `Client` is set before any `OpenCollection` calls; consider returning an error from `DBInstance()` or initializing `Client` in `main.go` before controller package-level variables run.
- When adding new collections, follow pattern: add model in `models/`, call `database.OpenCollection("collectionName")` in controller and use `bson` tags matching the model.

Files to reference while coding
- `main.go` — router setup and route registration.
- `database/database_connection.go` — env keys, DB client creation, `OpenCollection` helper.
- `controllers/movie_controller.go` — example controller pattern: context usage, collection usage, decoding cursor.
- `models/movie_model.go` — canonical data shapes and field tags.

If you need anything missing or more examples (e.g., how to add POST/PUT handlers, migrations, or tests), tell me what area to expand and I will add short, targeted examples.

---
Please review this file for accuracy or missing details before I run any follow-up changes.
