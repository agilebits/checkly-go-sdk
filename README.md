[![GoDoc](https://godoc.org/github.com/bitfield/checkly?status.png)](http://godoc.org/github.com/bitfield/checkly)[![Go Report Card](https://goreportcard.com/badge/github.com/bitfield/checkly)](https://goreportcard.com/report/github.com/bitfield/checkly)[![CircleCI](https://circleci.com/gh/bitfield/checkly.svg?style=svg)](https://circleci.com/gh/bitfield/checkly)

# checkly

`checkly` is a Go library for the [Checkly](https://checkly.com/) website monitoring service. It allows you to create new checks, get data on existing checks, and delete checks.

## Setting your API key

To use the client library with your Checkly account, you will need an API Key for the account. Go to the [Account Settings: API Keys page](https://app.checklyhq.com/account/api-keys) and click 'Create API Key'.

## Using the Go library

Import the library using:

```go
import "github.com/bitfield/checkly"
```

## Creating a client

Create a new `Client` object by calling `checkly.NewClient()` with your API key:

```go
apiKey := "3a4405dfb5894f4580785b40e48e6e10"
client := checkly.NewClient(apiKey)
```

## Creating a new check

Once you have a client, you can create a check. First, populate a Check struct with the required parameters:

```go
check := checkly.Check{
        Name:      "My Awesome Check",
        Type:      TypeBrowser,
        Activated: true,
        Request:   checkly.Request{
                Method: http.MethodGet,
                URL: "http://example.com",
        }
}
```

Now you can pass it to `client.Create()` to create a check. This returns the ID string of the newly-created check:

```go
ID, err := client.Create(check)
```

## Retrieving a check

`client.Get(ID)` finds an existing check by ID and returns a Check struct containing its details:

```go
check, err := client.Get("87dd7a8d-f6fd-46c0-b73c-b35712f56d72")
fmt.Println(check.Name)
// Output: My Awesome Check

```

## Deleting a check

Use `client.Delete(ID)` to delete a check by ID.

```go
err := client.Delete("73d29ea2-6540-4bb5-967e-e07fa2c9465e")
```

## Debugging

If things aren't working as you expect, you can assign an `io.Writer` to `client.Debug` to receive debug output. If `client.Debug` is non-nil, `MakeAPICall()` will dump the HTTP request and response to it:

```go
client.Debug = os.Stdout
p := checkly.Params{
    "frogurt": "cursed",
}
status, response, err := client.MakeAPICall(http.MethodGet, "monkeyPaw", p)
if err != nil {
    log.Fatal(err)
}
```

outputs:

```
POST /v1/monkeyPaw HTTP/1.1
Host: api.checkly.com
User-Agent: Go-http-client/1.1
Content-Length: 52
Content-Type: application/x-www-form-urlencoded
Accept-Encoding: gzip

api_key=XXX&format=json&frogurt=cursed
...
```

## Managing checks with Terraform

While you can manage your Checkly checks entirely in Go code, using this library, you may prefer to use Terraform. In that case, you can use the Checkly Terraform provider (which in turn uses this library):

https://github.com/bitfield/terraform-provider-checkly

## Bugs and feature requests

If you find a bug in the `checkly` client or library, please [open an issue](https://github.com/bitfield/checkly/issues). Similarly, if you'd like a feature added or improved, let me know via an issue.

Not all the functionality of the Checkly API is implemented yet.

Pull requests welcome!
