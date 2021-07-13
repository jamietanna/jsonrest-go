# jsonrest-go

[![CircleCI](https://img.shields.io/circleci/build/github/deliveroo/jsonrest-go)](https://circleci.com/gh/deliveroo/jsonrest-go/tree/master)
[![Go Report Card](https://goreportcard.com/badge/github.com/deliveroo/jsonrest-go)](https://goreportcard.com/report/github.com/deliveroo/jsonrest-go)
[![GoDoc](https://godoc.org/net/http?status.svg)](https://godoc.org/github.com/deliveroo/jsonrest-go)
[![go.dev](https://img.shields.io/badge/go.dev-pkg-007d9c.svg?style=flat)](https://pkg.go.dev/github.com/deliveroo/jsonrest-go)

Package jsonrest implements a microframework for writing RESTful web
applications.

Endpoints are defined as:

```go
func(ctx context.Context, req *jsonrest.Request) (interface{}, error)
```

If an endpoint returns a value along with a nil error, the value will be
rendered to the client as JSON.

If an error is returned, it will be sanitized and returned to the client as
json. Errors generated by a call to `jsonrest.Error(status, code, message)`
will be rendered in the following form:
```
{
    "error": {
        "message": message,
        "code": code
    }
}
```
along with the given HTTP status code.
If the error returned supports `StatusCode() int` method, it will be marshaled as-is to the client, using standard `json.Marshal()` with the status code provided as well.
Any other errors will be obfuscated to the caller (unless `router.DumpError` is
enabled).

Example:

```go
func main() {
    r := jsonrest.NewRouter()
    r.Use(logging)
    r.Get("/", hello)
}

func hello(ctx context.Context, req *jsonrest.Request) (interface{}, error) {
    return jsonrest.M{"message": "Hello, world"}, nil
}

func logging(next jsonrest.Endpoint) jsonrest.Endpoint {
    return func(ctx context.Context, req *jsonrest.Request) (interface{}, error) {
        start := time.Now()
        defer func() {
            log.Printf("%s (%v)\n", req.URL().Path, time.Since(start))
        }()
        return next(ctx, req)
    }
}
```

## Contributing

Review the [contributing guidelines](./CONTRIBUTING.md).
