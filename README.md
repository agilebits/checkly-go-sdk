[![GoDoc](https://godoc.org/github.com/checkly/checkly-go-sdk?status.png)](http://godoc.org/github.com/checkly/checkly-go-sdk)[![Go Report Card](https://goreportcard.com/badge/github.com/checkly/checkly-go-sdk)](https://goreportcard.com/report/github.com/checkly/checkly-go-sdk)[![CircleCI](https://circleci.com/gh/checkly/checkly-go-sdk.svg?style=svg)](https://circleci.com/gh/checkly/checkly-go-sdk)

# checkly

`checkly` is a Go library for the [Checkly](https://checklyhq.com/?utm_source=github&lmref=1374) website monitoring service. It allows you to create new checks, get data on existing checks, and delete checks.

While you can manage your Checkly checks entirely in Go code, using this library, you may prefer to use Terraform. In that case, you can use the Checkly Terraform provider (which in turn uses this library):

https://github.com/checkly/terraform-provider-checkly

## Setting your API key

To use the client library with your Checkly account, you will need an API Key for the account. Go to the [Account Settings: API Keys page](https://app.checklyhq.com/account/api-keys) and click 'Create API Key'.

## Using the Go library

Import the library using:

```go
import checkly "github.com/checkly/checkly-go-sdk"
```

## Creating a client

Create a new `Client` object by calling `checkly.NewClient()` with your API key:

```go
apiKey := "3a4405df05894f4580758b40e48e6e10"
client := checkly.NewClient(apiKey)
```

Or read the key from an environment variable:

```go
client := checkly.NewClient(os.Getenv("CHECKLY_API_KEY"))
```

## Creating a new check

Once you have a client, you can create a check. First, populate a Check struct with the parameters you want:

```go
wantCheck := checkly.Check{
	Name:                 "My API Check",
	Type:                 checkly.TypeAPI,
	Frequency:            5,
	DegradedResponseTime: 5000,
	MaxResponseTime:      15000,
	Activated:            true,
	Muted:                false,
	ShouldFail:           false,
	DoubleCheck:          false,
	SSLCheck:             true,
	LocalSetupScript:     "",
	LocalTearDownScript:  "",
	Locations: []string{
		"eu-west-1",
		"ap-northeast-2",
	},
	Tags: []string{
		"foo",
		"bar",
	},
	AlertSettings:          alertSettings,
	UseGlobalAlertSettings: false,
	Request: checkly.Request{
		Method: http.MethodGet,
		URL:    "http://example.com",
		Headers: []checkly.KeyValue{
			{
				Key:   "X-Test",
				Value: "foo",
			},
		},
		QueryParameters: []checkly.KeyValue{
			{
				Key:   "query",
				Value: "foo",
			},
		},
		Assertions: []checkly.Assertion{
			{
				Source:     checkly.StatusCode,
				Comparison: checkly.Equals,
				Target:     "200",
			},
		},
		Body:     "",
		BodyType: "NONE",
	},
}
```

Now you can pass it to `client.Create()` to create a check. This returns the newly-created Check object, or an error if there was a problem:

```go
check, err := client.Create(wantCheck)
```

For browser checks, the options are slightly different:

```go
wantCheck := checkly.Check{
	Name:          "My Browser Check",
	Type:          checkly.TypeBrowser,
	Frequency:     5,
	Activated:     true,
	Muted:         false,
	ShouldFail:    false,
	DoubleCheck:   false,
	SSLCheck:      true,
	Locations:     []string{"eu-west-1"},
	AlertSettings: alertSettings,
	Script: `const assert = require("chai").assert;
	const puppeteer = require("puppeteer");

	const browser = await puppeteer.launch();
	const page = await browser.newPage();
	await page.goto("https://example.com");
	const title = await page.title();

	assert.equal(title, "Example Site");
	await browser.close();`,
	EnvironmentVariables: []checkly.EnvironmentVariable{
		{
			Key:   "HELLO",
			Value: "Hello world",
		},
	},
	Request: checkly.Request{
		Method: http.MethodGet,
		URL:    "http://example.com",
	},
}
```

## Retrieving a check

`client.Get(ID)` finds an existing check by ID and returns a Check struct containing its details:

```go
check, err := client.Get("87dd7a8d-f6fd-46c0-b73c-b35712f56d72")
fmt.Println(check.Name)
// Output: My Awesome Check

```

## Updating a check

`client.Update(ID, check)` updates an existing check with the specified details. For example, to change the name of a check:

```go
ID := "87dd7a8d-f6fd-46c0-b73c-b35712f56d72"
check, err := client.Get(ID)
check.Name = "My updated check name"
updatedCheck, err = client.Update(ID, check)
```

## Deleting a check

Use `client.Delete(ID)` to delete a check by ID.

```go
err := client.Delete("73d29ea2-6540-4bb5-967e-e07fa2c9465e")
```

## Creating a new check group

Checkly checks can be combined into a group, so that you can configure default values for all the checks within it:

```go
var wantGroup = checkly.Group{
	Name:        "test",
	Activated:   true,
	Muted:       false,
	Tags:        []string{"auto"},
	Locations:   []string{"eu-west-1"},
	Concurrency: 3,
	APICheckDefaults: checkly.APICheckDefaults{
		BaseURL: "example.com/api/test",
		Headers: []checkly.KeyValue{
			{
				Key:   "X-Test",
				Value: "foo",
			},
		},
		QueryParameters: []checkly.KeyValue{
			{
				Key:   "query",
				Value: "foo",
			},
		},
		Assertions: []checkly.Assertion{
			{
				Source:     checkly.StatusCode,
				Comparison: checkly.Equals,
				Target:     "200",
			},
		},
		BasicAuth: checkly.BasicAuth{
			Username: "user",
			Password: "pass",
		},
	},
	EnvironmentVariables: []checkly.EnvironmentVariable{
		{
			Key:   "ENVTEST",
			Value: "Hello world",
		},
	},
	DoubleCheck:            true,
	UseGlobalAlertSettings: false,
	AlertSettings: checkly.AlertSettings{
		EscalationType: checkly.RunBased,
		RunBasedEscalation: checkly.RunBasedEscalation{
			FailedRunThreshold: 1,
		},
		TimeBasedEscalation: checkly.TimeBasedEscalation{
			MinutesFailingThreshold: 5,
		},
		Reminders: checkly.Reminders{
			Amount:   0,
			Interval: 5,
		},
		SSLCertificates: checkly.SSLCertificates{
			Enabled:        true,
			AlertThreshold: 30,
		},
	},
	AlertChannelSubscriptions: []checkly.Subscription{
		{
			Activated: true,
		},
	},
	LocalSetupScript:    "setup-test",
	LocalTearDownScript: "teardown-test",
}
group, err := client.CreateGroup(wantGroup)
```

## A complete example program

You can see an example program which creates a Checkly check in the [examples/demo](examples/demo/main.go) folder.

## Debugging

If things aren't working as you expect, you can assign an `io.Writer` to `client.Debug` to receive debug output. If `client.Debug` is non-nil, then all API requests and responses will be dumped to the specified writer (for example, `os.Stderr`).

Regardless of the debug setting, if a request fails with HTTP status 400 Bad Request), the full response will be dumped (to standard error if no debug writer is set):

```go
client.Debug = os.Stderr
```

Example request and response dump:

```
POST /v1/checks HTTP/1.1
Host: api-test.checklyhq.com
User-Agent: Go-http-client/1.1
Content-Length: 1078
Authorization: Bearer XXX
Content-Type: application/json
Accept-Encoding: gzip

{"id":"","name":"test","checkType":"API","frequency":10,"activated":true,
"muted":false,"shouldFail":false,"locations":["eu-west-1"],
"degradedResponseTime":15000,"maxResponseTime":30000,"script":"foo",
"environmentVariables":[{"key":"ENVTEST","value":"Hello world","locked":false}],
"doubleCheck":true,"tags":["foo","bar"],"sslCheck":true,
"localSetupScript":"setitup","localTearDownScript":"tearitdown","alertSettings":
{"escalationType":"RUN_BASED","runBasedEscalation":{"failedRunThreshold":1},
"timeBasedEscalation":{"minutesFailingThreshold":5},"reminders":{"interval":5},
"sslCertificates":{"enabled":false,"alertThreshold":30}},
"useGlobalAlertSettings":false,"request":{"method":"GET","url":"https://example.
com","followRedirects":false,"body":"","bodyType":"NONE","headers":[
{"key":"X-Test","value":"foo","locked":false}],"queryParameters":[
{"key":"query","value":"foo","locked":false}],"assertions":[
{"edit":false,"order":0,"arrayIndex":0,"arraySelector":0,
"source":"STATUS_CODE","property":"","comparison":"EQUALS",
"target":"200"}],"basicAuth":{"username":"","password":""}}}

HTTP/1.1 201 Created
Transfer-Encoding: chunked
Cache-Control: no-cache
Connection: keep-alive
Content-Type: application/json; charset=utf-8
Date: Thu, 28 May 2020 11:18:31 GMT
Server: Cowboy
Vary: origin,accept-encoding
Via: 1.1 vegur

4ea
{"name":"test","checkType":"API","frequency":10,"activated":true,"muted":false,
"shouldFail":false,"locations":["eu-west-1"],"degradedResponseTime":15000,
"maxResponseTime":30000,"script":"foo","environmentVariables":[{"key":"ENVTEST",
"value":"Hello world","locked":false}],"doubleCheck":true,"tags":["foo","bar"],
"sslCheck":true,"localSetupScript":"setitup","localTearDownScript":"tearitdown",
"alertSettings":{"escalationType":"RUN_BASED","runBasedEscalation":
{"failedRunThreshold":1},"timeBasedEscalation":{"minutesFailingThreshold":5},
"reminders":{"interval":5,"amount":0},"sslCertificates":{"enabled":false,
"alertThreshold":30}},"useGlobalAlertSettings":false,"request":{"method":"GET",
"url":"https://example.com","followRedirects":false,"body":"","bodyType":"NONE",
"headers":[{"key":"X-Test","value":"foo","locked":false}],"queryParameters":[
{"key":"query","value":"foo","locked":false}],"assertions":[
{"source":"STATUS_CODE","property":"","comparison":"EQUALS","target":"200"}],
"basicAuth":{"username":"","password":""}},"setupSnippetId":null,
"tearDownSnippetId":null,"groupId":null,"groupOrder":null,
"alertChannelSubscriptions":[{"activated":true,"alertChannelId":35}],
"created_at":"2020-05-28T11:18:31.280Z",
"id":"29815146-8ab5-492d-a092-9912c1ab8333"}
0
```

## Bugs and feature requests

If you find a bug in the `checkly` client or library, please [open an issue](https://github.com/checkly/checkly-go-sdk/issues). Similarly, if you'd like a feature added or improved, let me know via an issue.

Not all the functionality of the Checkly API is implemented yet.

Pull requests welcome!
