# go-store

A simple file-based JSON database written in Go. Each record is stored as a pretty-printed `.json` file inside a folder named after its collection.

## How it works

```
<db dir>/
└── users/              # collection
    ├── john.json       # resource (one record per file)
    ├── Paul.json
    └── ...
```

- **Collections** are directories, **resources** are `<name>.json` files.
- Writes go to a temporary `.tmp` file first and are then renamed, so a record is never left half-written.
- Each collection has its own mutex, so concurrent writes and deletes to the same collection are safe.
- Logging uses [lumber](https://github.com/jcelliott/lumber) (console logger at `INFO` level by default).

## Requirements

- Go 1.25 or later

## Getting started

```bash
git clone <repo-url>
cd go-store
go mod tidy
go run main.go
```

`main.go` seeds a `users` collection with sample employees, reads them back, and prints them.

## API

```go
db, err := New("./", nil)                    // open or create the database directory
err = db.Write("users", "john", user)        // save a record as users/john.json
err = db.Read("users", "john", &user)        // load one record into a struct
records, err := db.ReadAll("users")          // all records in a collection, as JSON strings
err = db.Delete("users", "john")             // delete one record
err = db.Delete("users", "")                 // delete the whole collection
```

| Method | Description |
|---|---|
| `New(dir string, options *Options) (*Driver, error)` | Creates the database directory if it does not exist. Pass `&Options{Logger: ...}` to use your own logger. |
| `Write(collection, resource string, v interface{}) error` | Marshals `v` to JSON and saves it. |
| `Read(collection, resource string, v interface{}) error` | Reads a record and unmarshals it into `v`. |
| `ReadAll(collection string) ([]string, error)` | Returns the raw JSON of every record in the collection. |
| `Delete(collection, resource string) error` | Removes a single record, or the whole collection when `resource` is empty. |

## Data model

```go
type User struct {
	Name    string
	Age     json.Number
	Contact string
	Company string
	Address Address
}

type Address struct {
	City    string
	State   string
	Country string
	Pincode json.Number
}
```

Example record (`users/john.json`):

```json
{
	"Name": "john",
	"Age": 23,
	"Contact": "2125550143",
	"Company": "Tech Mahindra",
	"Address": {
		"City": "New York",
		"State": "New York",
		"Country": "USA",
		"Pincode": 10001
	}
}
```
