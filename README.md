# Police Department Administration Portal

Department-level case, investigation, and digital evidence administration application with a React/TypeScript interface, Node.js REST API, SQLite persistence, and native C++17 data structures and algorithms.

## Requirements

- Node.js 22.13 or newer (Node.js 24 recommended; the API uses the built-in `node:sqlite` module).
- npm.
- A C++17 compiler available as `g++`.
- Windows for the SHA-256 implementation in this project. It uses Windows CryptoAPI with the operating system's SHA-256 provider; it does not implement a custom cryptographic hash.

## Install and run on Windows

Open PowerShell in the project directory:

```powershell
npm install
npm run build:cpp
```

Start the API in one terminal:

```powershell
npm run api
```

Start the frontend in a second terminal:

```powershell
npm run dev
```

Open <http://127.0.0.1:5173>. The API health endpoint is <http://127.0.0.1:8787/api/health>.

The C++ build script uses `g++ -std=c++17` and links the Windows `advapi32` system library for SHA-256. If `g++` is not installed, install MinGW-w64 or MSYS2, add its `bin` folder to `PATH`, then rerun `npm run build:cpp`. The SQLite database is created at `backend/data/evidence.db`; uploaded files are stored under `backend/data/uploads/`. These runtime data files are ignored by Git.

## Project structure

```text
backend/
  build-cpp.mjs       C++17 compiler entry point
  cpp/dsa.cpp         BST, FIFO queue, graph traversals, sorting and SHA-256
  server.mjs          REST API, validation, SQLite schema and demo seed data
src/
  App.tsx             Application pages and interactive workflows
  api.ts              Frontend REST helper and upload encoding
  main.tsx            React entry point
  styles.css          Responsive application styling
```

## Implemented REST endpoints

| Method | Endpoint | Purpose |
| --- | --- | --- |
| GET | `/api/health` | API, database and native-module status |
| GET | `/api/dashboard` | Counts and recent case/activity records |
| GET, POST | `/api/cases` | List and create cases |
| PUT, DELETE | `/api/cases/:id` | Update a case or delete a case without linked evidence |
| GET, POST | `/api/officers` | List and add officers |
| PUT | `/api/officers/:id` | Update officer details or active status |
| GET, POST | `/api/evidence` | List and register evidence (optional file upload as base64) |
| POST | `/api/evidence/:id/verify` | Compare uploaded or stored bytes with registered SHA-256 |
| GET | `/api/evidence/:id/custody` | Read the append-only custody history |
| POST | `/api/evidence/:id/transfer` | Validate and append a custody transfer |
| POST | `/api/evidence/:id/corrections` | Append an administrative correction event |
| GET, POST | `/api/tasks` | List tasks in arrival order and enqueue a task |
| POST | `/api/tasks/process-next` | Process the next FIFO task |
| POST | `/api/dsa/bst` | Insert, search, delete, or traverse demonstration BST keys |
| POST | `/api/dsa/queue` | List the pending queue or inspect its front |
| POST | `/api/dsa/graph` | Run BFS or DFS over the current simplified relationship graph |
| POST | `/api/dsa/sort` | Sort current case or evidence records |
| POST | `/api/demo/reset` | Replace local records with the fictional sample dataset |

The API listens on `127.0.0.1` only. It validates request fields, uses parameterized SQLite statements, enables foreign keys, and uses a transaction for custody transfers and related record changes. The REST service returns JSON error messages with appropriate HTTP status codes.

## DSA presentation notes

- **SHA-256:** The C++ runner calls Windows CryptoAPI to hash actual file bytes in chunks. On registration, uploaded bytes are saved locally and the digest is stored. Verification compares a fresh digest of the submitted or stored bytes. Demo evidence with only a storage reference has no hash to verify.
- **Binary Search Tree:** A C++ BST is rebuilt from the numeric suffixes of current case IDs for each operation. Insert, search, deletion and all three depth-first tree traversals are demonstrated. Visualizer-only keys do not create or delete application cases. An unbalanced BST can take O(n) in the worst case.
- **Queue:** Pending evidence tasks persist in SQLite. The C++ FIFO queue is populated in arrival order for peek/list/dequeue operations; processing removes the front task from SQLite.
- **Graph:** C++ constructs an undirected adjacency list from case–evidence and evidence–collector/current-custodian relationships. The visualizer also allows temporary vertices and edges, then runs BFS or DFS from a selected vertex. These teaching additions are not saved as application records.
- **Sorting:** C++ bubble sort, insertion sort and merge sort order real case/evidence records by ID, title/name or date. The visualizer shows the before/after order and a bounded operation trace; sorting does not modify stored records.

## Demonstration data and limitations

The first run inserts clearly fictional sample officers, cases, evidence metadata, custody events and pending tasks. Use **System Information → Reset demonstration data** to replace the local data with the sample set. Reset removes current SQLite records and locally stored evidence upload files.

Evidence uploads are limited to 10 MB. File names are retained as display metadata; upload storage names are generated by the server, and physical filesystem paths are not returned by the API. Sample records intentionally have no file bytes or SHA-256 digest.

Custody history cannot be edited or deleted through the application. Transfers require the stated previous custodian and responsible officer to match the stored current custodian. Corrections append an event without rewriting prior history. This local academic audit log is **not** a cryptographically linked or tamper-proof chain, and its correction control is not backed by real user authentication or administrator authorization.

SHA-256 can help detect changes to bytes but does not independently prove who collected evidence or whether the evidence was authentic at the time of collection. This project has no authentication/role system, does not implement production forensic controls, and should not be used as certified or operational police software.
