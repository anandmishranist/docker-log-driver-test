`config.json` is the plugin config required for registering with docker.

Plugins are containers with a special config

But ultimately it’s just a container.

So the example builds a single static binary.

You would take that and put it into a dir with `config.json` and `rootfs/<binary>`

Then do `docker plugin create` pointing at that dir.

`http.go` is all the http handlers for the plugin interface with docker. Plugins run out of the main Docker daemon process, this means that the Docker daemon needs to find a way to speak with plugins. 
To solve this communication problem, each plugin has to implement an HTTP server which can be discovered by the Docker daemon. This server exposes a set of RPCs issued as HTTP POSTs with JSON payloads. 

`driver.go` has all the business logic for the logger.

`main.go` has all the initialization.




$ docker build -f Dockerfile.build .

`Dockerfile.build` is only building the binary.



LogDriver protocol
Logging plugins must register as a LogDriver during plugin activation. Once activated users can specify the plugin as a log driver.
There are two HTTP endpoints that logging plugins must implement:
						/LogDriver.StartLogging
						/LogDriver.StopLogging
/LogDriver.StartLogging
Signals to the plugin that a container is starting that the plugin should start receiving logs for. Logs will be streamed over the defined file in the request. On Linux this file is a FIFO. Logging plugins are not currently supported on Windows.

/LogDriver.StopLogging
Signals to the plugin to stop collecting logs from the defined file. Once a response is received, the file will be removed by Docker. You must make sure to collect all logs on the stream before responding to this request or risk losing log data.
Requests on this endpoint does not mean that the container has been removed only that it has stopped.

/LogDriver.Capabilities
Defines the capabilities of the log driver. You must implement this endpoint for Docker to be able to take advantage of any of the defined capabilities.

/LogDriver.ReadLogs
Reads back logs to the client. This is used when docker logs <container> is called.
In order for Docker to use this endpoint, the plugin must specify as much when /LogDriver.Capabilities is called.
