# Playground for [marathon-consul](https://github.com/allegro/marathon-consul)

This repository contains a [Vagrantfile](https://www.vagrantup.com/) which makes it easy to experiment with
the [marathon-consul](https://github.com/allegro/marathon-consul) project.

It creates a virtual machine with [mesos](http://mesos.apache.org/), [marathon](https://mesosphere.github.io/marathon/),
[consul](https://www.consul.io/) and [marathon-consul](https://github.com/allegro/marathon-consul) installed.

## Requirements

* [Vagrant](https://www.vagrantup.com/) (1.7+)
* [VirtualBox](https://www.virtualbox.org/) (4.0.x, 4.1.x, 4.2.x, 4.3.x, 5.0.x, 5.1.x)

## Usage

Download the [Vagrantfile](https://raw.githubusercontent.com/dankraw/marathon-consul-playground/master/Vagrantfile) and run `vagrant up`.

When everything is up and running the services should be available at the following locations:

* Mesos: http://10.10.10.10:5050
* Marathon: http://10.10.10.10:8080
* Consul: http://10.10.10.10:8500

You can access the machine with `vagrant ssh` and stop it with `vagrant halt`.

`Marathon-consul` logs are available at `/var/log/upstart/marathon-consul.log` file.

### Example scenario

Create an application in Marathon by sending a `PUT` request to `10.10.10.10:8080` with some example data:

```js
[{
	"id": "/myapp",
	"cmd": "python -m SimpleHTTPServer $PORT0",
	"instances": 1,
	"cpus": 0.1,
	"mem": 16,
	"disk": 0,
	"executor": "",
	"ports": [0],
	"healthChecks": [{
		"path": "/",
		"protocol": "HTTP",
		"portIndex": 0,
		"gracePeriodSeconds": 300,
		"intervalSeconds": 60,
		"timeoutSeconds": 20,
		"maxConsecutiveFailures": 3,
		"ignoreHttp1xx": false
	}],
	"labels": {
		"consul": "myCustomServiceName"
	}
}]
```

> If you want your registered service name to be the same as Marathon application name, pass empty string as the value of "consul" label

Example using `curl`:
```bash
curl -s -X PUT -H "Content-Type: application/json" http://10.10.10.10:8080/v2/apps -d '[{"id":"/myapp","cmd":"python -m SimpleHTTPServer $PORT0","instances":1,"cpus":0.1,"mem":16,"disk":0,"executor":"","ports":[0],"healthChecks":[{"path":"/","protocol":"HTTP","portIndex":0,"gracePeriodSeconds":300,"intervalSeconds":60,"timeoutSeconds":20,"maxConsecutiveFailures":3,"ignoreHttp1xx":false}],"labels":{"consul":"myCustomServiceName"}}]' | python -m json.tool
```
```js
{
    "deploymentId": "898acddc-9279-4915-84a1-81cbb6bf4b0d",
    "version": "2016-01-13T13:58:58.034Z"
}
```

Now you should be able to see it in the Marathon console at http://10.10.10.10:8080/.

When the healthcheck goes green it will be registered as a service in Consul (http://10.10.10.10:8500/):

```bash
curl -s 10.10.10.10:8500/v1/agent/services | python -m json.tool
```
```js
{
    "consul": {
        "Address": "",
        "CreateIndex": 0,
        "EnableTagOverride": false,
        "ID": "consul",
        "ModifyIndex": 0,
        "Port": 8300,
        "Service": "consul",
        "Tags": []
    },
    "myapp.cc533052-b9f8-11e5-9fc0-080027fcad58": {
        "Address": "10.10.10.10",
        "CreateIndex": 0,
        "EnableTagOverride": false,
        "ID": "myapp.cc533052-b9f8-11e5-9fc0-080027fcad58",
        "ModifyIndex": 0,
        "Port": 31867,
        "Service": "myCustomServiceName",
        "Tags": [
            "marathon"
        ]
    }
}
```
