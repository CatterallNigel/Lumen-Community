# Lumen v3.1.1

- Fixed MongoDB `ConflictingUpdateOperators` when inserting a new session by allowing `$inc` alone to initialise `revision`.
- Fixed `\obt` command detection when Pi sends user text as OpenAI content-part arrays.
- Replaced deprecated FastAPI `on_event("startup")` with a lifespan handler.
- Close the shared MongoDB client during application shutdown.
