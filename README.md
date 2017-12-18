The Very Log lib for golang

Table of Contents
=================

* [Get Logger](#get-logger)
* [Log Message](#log-message)
* [Setting By Code](#setting-by-code)
* [FileAppender Log Rotate](#fileappender-log-rotate)
* [Setting Level By Environ](#setting-level-by-environ)
* [Appenders](#appenders)
* [Rotaters](#rotaters)
* [Transformers](#transformers)

## Add Dependency

Go get:

```sh
go get github.com/hsiafan/vlog
```

If use go dep, add constraint to Gopkg.toml:

```toml
[[constraint]]
  name = "github.com/hsiafan/vlog"
  version = "0.3.0"
```

## Get Logger

Each logger has a name, there is only one logger for same name. You can pass a name, or just using current package name as logger name:

```go
var logger = vlog.GetLogger(loggerName) // specify a logger name
var logger = vlog.CurrentPackageLogger() // using full package name as logger name
```

## Log Message

Logger has six levels: Trace/Debug/Info/Warn/Error/Critical.
Log methods can use format string to format params, if has more params than placeholders, the remain params will be output after formatted string.

```go
logger.Info("start the server")
logger.Info("start the server at {}:{}", host, port)
logger.Info("start the server at", host+":"+strconv.itoa(port))
logger.Error("start server error:", err)
logger.Error("start server {}:{} error:", host, port, err)
```

Loggers also have IsXxxxEnable methods, to avoid unnecessary converting cost:

```go
if logger.IsDebugEnable() {
	logger.Debug("server accept connection:", expensiveConvert(conn))
}
```

## Setting By Code

By default, logger only output message with info level or above, using default message format, to standard output.
To change this, set custom Appender/Level/Transformer to the logger.

```go
var logger = vlog.CurrentPackageLogger()

func init() {
	appender := vlog.NewConsoleAppender()
	// custom log format
	transformer, _ := vlog.NewPatternTransformer("{time} [{Level}] {file}:{line} - {message}\n")
	appender.SetTransformer(transformer)
	// using custom appender
	logger.SetAppenders(appender)
	// set level to debug, will output all message with level equal or higher than Debug
	logger.SetLevel(vlog.Debug)
}
```

## FileAppender Log Rotate

If using FileAppender to write log into file, a log rotater can be set to rotate log file, by log file size or time.

```go
// appender without rotater
appender := vlog.NewFileAppender("path/to/logfile", nil)
// appender with rotater rotate log file every 800m
rotater := vlog.NewSizeRotater(800 * 1024*1024, 6)
appender := vlog.NewFileAppender("path/to/logfile", rotater)
// appender with rotater rotate log file every day
rotater := vlog.NewDailyRotater("20060102")
appender := vlog.NewFileAppender("path/to/logfile", rotater)
```

## Setting Level By Environ

Loggers' level can be set by one environ: VLOG_LEVEL. The level set by environ will override the level set in code.
So you can set logger by code in your final or in your lib in development,
when final routine is deployed, you can set the environ meet your real need.

For example, run program in linux shell, you can set as follows:

```bash
export VLOG_LEVEL="package1=Warn;github.com/user1=Debug"
```

If use package path as logger name, vlog will match the setting by prefix. It means github.com/user1=Debug will take effect
for logger with name github.com/user1/lib.

## Appenders

Appenders supportted now:

| Appender Type | Create by Code |
| :------: | :------: |
| ConsoleAppender | NewConsoleAppender |
| ConsoleAppender | NewConsole2Appender |
| FileAppender | NewFileAppender |
| SyslogAppender | SyslogAppender |
| NopAppender | NewNopAppender |

## Rotaters

| Rotater Type | Create by Code |
| :------: | :------: |
| TimeRotater | NewDailyRotater |
| TimeRotater | NewHourlyRotater |
| TimeRotater | NewTimeRotater |
| SizeRotater | NewSizeRotater |

## Transformers

| Transformer Type | Create by Code |
| :------: | :------: |
| PatternTransformer | NewPatternTransformer |

Below variables can be used in PatternTransformer format string:

* {file} filename
* {package} package name
* {line} line number
* {function} function name
* {time} time
* {logger} the logger name
* {Level}/{level}/{LEVEL} the logger level, with different character case
* {message} the log message

Use {{ to escape  {, use }} to escape }

{time} can set custom format via filter, by {time|2006-01-02 15:04:05.000}


