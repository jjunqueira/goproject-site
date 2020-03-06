---
title: "Basic Template Walkthrough"
date: 2020-02-06T09:14:38-06:00
draft: false
---

The Basic template serves as a starting point for any type of Go project. It will only include the initial project structure, a few helpers (logging, metrics), and a Makefile.

All the steps in this walkthrough assume you have already completed [installation](../../getting-started/installation).

For this walkthrough we will create a simple API with 1 endpoint. This will help you familiarize yourself with the process of creating a project and adding some code.

## Creating a new project
Before boostrapping a new project with goproject you should always update your templates. This can be done with the built in update command.

```
goproject update
```

After updating your templates you are now ready to use the 'new' command. The 'new' command will help you create a fully functioning Go project based on a template. Let's try this out using the 'Basic' template.

```
goproject new basic testproj
```

Let's quickly walk through the parameters to make sure everything is clear:
1. 'new' - this tells goproject we are starting a new project
2. 'basic' - this tells goproject that we want to use the 'basic' template
3. 'testproj' - this is the name of the project we are going to create

If you are using git for your project and want the git url to be part of the module path then you can add an additional flag to the 'new' command.

```
goproject new basic testproj --gitprefix "github.com/jjunqueira"
```


Just to make sure the project built correctly we will go ahead and go into the project directory and attempt to build it.

```
cd testproj
make
.... output ...
rm -rf /Users/jjunqueira/Downloads/testproj/target
go mod tidy
go: finding module for package github.com/prometheus/client_golang/prometheus
go: finding module for package go.uber.org/zap
go: finding module for package github.com/spf13/viper
go: found github.com/spf13/viper in github.com/spf13/viper v1.6.2
go: found go.uber.org/zap in go.uber.org/zap v1.14.0
go: found github.com/prometheus/client_golang/prometheus in github.com/prometheus/client_golang v1.5.0
go mod verify
all modules verified
find . -name \*.go -not -path vendor -not -path target -exec /Users/jjunqueira/go/bin/goimports -w {} \;
golangci-lint run
go test -v ./...
?   	github.com/jjunqueira/testproj/cmd/testproj	[no test files]
?   	github.com/jjunqueira/testproj/pkg/app	[no test files]
?   	github.com/jjunqueira/testproj/pkg/config	[no test files]
?   	github.com/jjunqueira/testproj/pkg/log	[no test files]
?   	github.com/jjunqueira/testproj/pkg/metrics	[no test files]
fatal: No names found, cannot describe anything.
mkdir -p /Users/jjunqueira/Downloads/testproj/target
for COMMAND in /Users/jjunqueira/Downloads/testproj/cmd/* ; do if [[ -z "${GOOS}" ]]; then GOOS=${OSTYPE}; fi; if [[ -d "${COMMAND}" ]]; then go build -trimpath -v -o "/Users/jjunqueira/Downloads/testproj/target/bin/$(basename ${COMMAND})-${GOOS}" -ldflags "-w -s -X main.version= -X main.build=2020-03-06T14:40:27-0600" "${COMMAND}/..."; fi done
github.com/jjunqueira/testproj/pkg/config
github.com/jjunqueira/testproj/pkg/log
github.com/jjunqueira/testproj/pkg/app
github.com/jjunqueira/testproj/cmd/testproj
... end output ...
```

If everything went well you shouldn't see any errors and you should now have a 'target' directory containing a binary for your platform:

```
└── target
    └── bin
        └── testproj-darwin18
```

## Project Structure
Great! goproject did something! But what exactly does it give me? For the basic template goproject will provide you with the following folder layout:

```
├── Jenkinsfile
├── Makefile
├── README.md
├── cmd
│   └── testproj
│       └── main.go
├── deployments
│   └── ansible
│       ├── files
│       │   ├── config.toml
│       │   └── testproj.service
│       ├── meta
│       │   └── main.yml
│       └── tasks
│           └── main.yml
├── go.mod
└── pkg
    ├── app
    │   └── app.go
    ├── config
    │   └── viper.go
    ├── log
    │   └── zap.go
    └── metrics
        └── prometheus.go
```

The project structure follows the fantastic [project-layout](https://github.com/golang-standards/project-layout). Full explanations of the folders and files can be viewed there but for completeness sake lets quickly go over them.

* Jenkinsfile - This provides a basic Jenkins build file that will work with this project
* Makefile - This is a complete makefile for doing various build tasks
* README.md - All good projects need a nice readme :)
* cmd - Your command directory, this is where the main entrypoint into your application lives.
* deployments - This is where your deployment scripts live. The basic template includes a full ansible deployment template.
* go.mod - All goproject templates should use go modules
* pkg - All of your supporting code
  * app - Application bootstrap and dependencies
  * config - fully functional config struct which utilizes the [viper](https://github.com/spf13/viper) library
  * log - logger with some helper functions and default fields. It utilizes the [zap](https://github.com/uber-go/zap) library.
  * metrics - A basic example of a [prometheus](https://github.com/prometheus/client_golang) metric.

## Adding configuration parameters
Let's familiarize ourself with the configuration options. The Viper library supports marshalling a config file into a struct. For this example we will add a new struct that describes a couple HTTP server configuration variables, modify the default config.toml file, and verify that it is working.

Modify pkg/config/viper.go to look like the following (we added the ServerSettings struct):
```
// AppConfig application configs
type AppConfig struct {
	Debug           bool           `mapstructure:"debug" json:"debug"`
	Version         string         `mapstructure:"version" json:"version"`
	Logging         LogSettings    `mapstructure:"logging" json:"logging"`
	Server          ServerSettings `mapstructure:"server" json:"server"`
}

// ServerSettings settings for server
type ServerSettings struct {
	Bindport           int    `mapstructure:"bindport" json:"bindport"`
	BindAddress        string `mapstructure:"bind_address" json:"bind_address"`
	Metricsport        int    `mapstructure:"metricsport" json:"metricsport"`
	MetricsBindAddress string `mapstructure:"metrics_bind_address" json:"metrics_bind_address"`
}
```

We can now modify the included base configuration file to add the new configuration parameters. Open up deployments/ansible/files/config.toml and add the following:

```
[server]
bindport=8080
bind_address="0.0.0.0"
metricsport=9090
metrics_bind_address="0.0.0.0"
```

In order to make sure the changes worked we will compile the project again and run it. If everything worked we will see a log line with a message saying the application bootstrapped successfully.

The default config template assumes you want to log to a file in /var/log. Since we are running locally we should modify the log configuration to just print to stdout. Your logging configuration should look like this:

```
[logging]
level="debug"
encoding="json"
outputpaths=["stdout"]
```

Now we can try running the application:

```
# make
...lots of output
# ./target/bin/testproj-darwin18 -config ./deployments/ansible/files 
{"level":"info","ts":1583527718.401667,"caller":"testproj/main.go:16","msg":"Application bootstrap completed","app":"testproj-darwin18","app_build":"2020-03-06T14:48:11-0600","app_location":"./target/bin/testproj-darwin18","app_version":"","compiler":"go1.14","pid":91812,"server":"JOSHUAs-MacBook-Pro-2.local","config":"{false  {debug [stdout]} {8080 0.0.0.0 9090 0.0.0.0}}"}
```

## Adding a metric

## Adding some code

## Compiling

## Packaging for release