# Polygon Migration Tool - Documentation

## Overview

This is a Django application used to fetch programming problems and test cases from Polygon and migrate them to the database and cloud storage.

**Technologies:** Django, Python, PostgreSQL, Redis, Polygon API and Google Drive.

## Flow 1: User Login

- **User action:** Enters email and password and clicks Login.
- **Data sent by frontend:** `email`, `password`, CSRF token.
- **Backend steps:**
  1. Request goes to `/users/login/`.
  2. `login_view` in `users/views.py` handles the request.
  3. `authenticate()` uses the custom `EmailBackend` to verify credentials.
  4. Checks user, password and `is_staff`.
  5. On success, `login()` creates a session and redirects to `problems:index`.
- **External APIs:** None.
- **Read / written:** Reads the users table and writes session data.
- **What the user sees:** Redirects to the problems page on success; otherwise, displays an error.

## Flow 2: Fetching a Problem from Polygon

- **User action:** Enters a Polygon problem ID and clicks Fetch.
- **Data sent by frontend:** `problem_id`.
- **Backend steps:**
  1. Request goes to `/` and is handled by `index` in `problems/views.py`.
  2. `user_passes_test` checks whether the user is staff.
  3. `PolygonAPI()` initializes the API client.
  4. Fetches problem details, downloads the package and parses the problem HTML.
  5. Gets test cases and stores them in Redis.
  6. Calls `problem.checker` to get checker information.
- **External APIs:** Polygon API.
- **Read / written:** Stores test cases in Redis for 30 minutes. No database changes.
- **What the user sees:** Problem details, test cases and solution.

## Flow 3: Migrating a Problem to the Database

- **User action:** Selects difficulty and tags, then clicks Create/Update Problem in Database.
- **Data sent by frontend:** `problem_id`, `difficulty`, `tags`, `new_tag`, `migrate_to_db`.
- **Backend steps:**
  1. Checks whether difficulty is selected.
  2. Checks for an existing problem using `polygon_id`.
  3. Updates the existing problem or creates a new one.
  4. Adds selected tags using `ProblemTag.get_or_create()`.
  5. Gets sample test cases from Redis and saves them as `SampleTestCase`.
  6. Fetches the main solution from Polygon.
- **External APIs:** Polygon API (`problem.solutions`, `problem.viewSolution`).
- **Read / written:** Reads Redis and writes `Problem`, `ProblemTag` and `SampleTestCase` records to PostgreSQL.
- **What the user sees:** Success message with Database Problem ID and Algopath link.

## Flow 4: Migrating Test Cases to the Database

- **User action:** Clicks Migrate Test Description to Database.
- **Data sent by frontend:** `problem_id`, `migrate_test_cases_to_db`.
- **Backend steps:**
  1. Checks whether the problem exists in the database.
  2. Gets test cases from Redis.
  3. Loads existing `ProblemTestCase` records ordered by `order`.
  4. Reads input, output, sample status and description.
  5. Skips tests with empty input or output.
  6. Updates existing records or creates new records.
- **External APIs:** Polygon API, only if test cases are not cached in Redis.
- **Read / written:** Reads Redis and PostgreSQL, writes `ProblemTestCase` records to PostgreSQL.
- **What the user sees:** Test cases migrated successfully message.

## Flow 5: Uploading Test Cases to Cloud Storage

- **User action:** Clicks Migrate Test Cases to Cloud Storage.
- **Data sent by frontend:** `problem_id`, `migrate_to_azure`.
- **Backend steps:**
  1. Checks whether the problem exists in the database.
  2. Deletes the Redis cache using `delete_problem_test_case_cache`.
  3. Checks for a custom checker.
  4. Initializes storage using `get_storage_backend()`.
  5. Calls `api.migrate_to_storage()`.
  6. Deletes old files and uploads test inputs, outputs and custom checker if available.
- **External APIs:** Polygon API and Google Drive API.
- **Read / written:** Reads Redis and Polygon, uploads files to cloud storage. No database changes.
- **What the user sees:** Success or error message after migration.

## External Integrations

### Polygon API
- **Used for:** Problem details, packages, test cases, checker and solutions.
- **Endpoints:** `problem.info`, `problem.checker`, `problem.solutions`, `problem.viewSolution`.
- **Authentication:** `_generate_api_sig()` generates the signature using `apiKey` and `time`. `_make_request()` sends the request with `apiKey`, `time` and `apiSig`.

### Cloud Storage
- **Provider:** Google Drive.
- **Backend:** `get_storage_backend()` and `api.migrate_to_storage()`.
- **File structure:** `test_cases/<database_problem_id>/`
- **File naming:** Input files use `NN` and output files use `NN.a`.
- Existing files are deleted before uploading new files. Custom checkers are compiled and uploaded if available.