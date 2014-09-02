---
title: REST API
---

# Marathon REST API

* [Apps](#apps)
  * [POST /v2/apps](#post-/v2/apps): Create and start a new app
  * [GET /v2/apps](#get-/v2/apps): List all running apps
  * [GET /v2/apps?cmd={command}](#get-/v2/apps?cmd={command}): List all running
    apps, filtered by `command`
  * [GET /v2/apps/{appId}](#get-/v2/apps/{appid}): List the app `appId`
  * [GET /v2/apps/{appId}/versions](#get-/v2/apps/{appid}/versions): List the versions of the application with id `appId`.
  * [GET /v2/apps/{appId}/versions/{version}](#get-/v2/apps/{appid}/versions/{version}): List the configuration of the application with id `appId` at version `version`.
  * [PUT /v2/apps/{appId}](#put-/v2/apps/{appid}): Change config of the app
    `appId`
  * [DELETE /v2/apps/{appId}](#delete-/v2/apps/{appid}): Destroy app `appId`
  * [GET /v2/apps/{appId}/tasks](#get-/v2/apps/{appid}/tasks): List running tasks
    for app `appId`
  * [DELETE /v2/apps/{appId}/tasks?host={host}&scale={true|false}](#delete-/v2/apps/{appid}/tasks?host={host}&scale={true|false}):
    kill tasks belonging to app `appId`
  * [DELETE /v2/apps/{appId}/tasks/{taskId}?scale={true|false}](#delete-/v2/apps/{appid}/tasks/{taskid}?scale={true|false}):
    Kill the task `taskId` that belongs to the application `appId`
* [Groups](#groups) <span class="label label-default">v0.7.0</span>
  * [GET /v2/groups](#get-/v2/groups): List all groups
  * [GET /v2/groups/{groupId}](#get-/v2/groups/{groupid}): List the group with the specified ID
  * [POST /v2/groups](#post-/v2/groups): Create and start a new groups
  * [PUT /v2/groups/{groupId}](#put-/v2/groups/{groupid}): Change parameters of a deployed application group
  * [PUT /v2/groups/{groupId}/version/{version}](#put-/v2/groups/{groupid}/version/{$version}): Rollback group to a previous version
  * [DELETE /v2/groups/{groupId}](#delete-/v2/groups/{groupid}): Destroy a group
* [Tasks](#tasks)
  * [GET /v2/tasks](#get-/v2/tasks): List all running tasks
* [Deployments](#deployments) <span class="label label-default">v0.7.0</span>
  * [GET /v2/deployments](#get-/v2/deployments): List running deployments
  * [DELETE /v2/deployments/{deploymentId}](#delete-/v2/deployments/{deploymentid}): Cancel the deployment with `deploymentId`
* [Event Subscriptions](#event-subscriptions)
  * [POST /v2/eventSubscriptions?callbackUrl={url}](#post-/v2/eventsubscriptions?callbackurl={url}): Register a callback URL as an event subscriber
  * [GET /v2/eventSubscriptions](#get-/v2/eventsubscriptions): List all event subscriber callback URLs
  * [DELETE /v2/eventSubscriptions?callbackUrl={url}](#delete-/v2/eventsubscriptions?callbackurl={url}) Unregister a callback URL from the event subscribers list
* [Queue](#queue) <span class="label label-default">v0.7.0</span>
  * [GET /v2/queue](#get-v2queue): List content of the staging queue.
* [Server Info](#server-info) <span class="label label-default">v0.7.0</span>
  * [GET /v2/info](#get-/v2/info): Get info about the Marathon Instance
* [Miscellaneous](#miscellaneous)
  * [GET /ping](#get-/ping)
  * [GET /logging](#get-/logging)
  * [GET /help](#get-/help)
  * [GET /metrics](#get-/status)

### Apps

#### POST `/v2/apps`

Create and start a new application.

The full JSON format of an application resource is as follows:

{% highlight json %}
{
    "id": "/product/service/myApp",
    "cmd": "env && sleep 300",
    "args": ["/bin/sh", "-c", "env && sleep 300"]
    "container": {
        "type": "DOCKER",
        "docker": {
            "image": "group/image"
        },
        "volumes": [
            {
                "containerPath": "/etc/a",
                "hostPath": "/var/data/a",
                "mode": "RO"
            },
            {
                "containerPath": "/etc/b",
                "hostPath": "/var/data/b",
                "mode": "RW"
            }
        ]
    },
    "cpus": 1.5,
    "mem": 256.0,
    "deployments": [
        "2b2eaba6-5c1f-4e8b-b781-bce3208109a7"
    ],
    "env": {
        "LD_LIBRARY_PATH": "/usr/local/lib/myLib"
    },
    "executor": "",
    "constraints": [
        ["attribute", "OPERATOR", "value"]
    ],
    "healthChecks": [
        {
            "protocol": "HTTP",
            "path": "/health",
            "gracePeriodSeconds": 3,
            "intervalSeconds": 10,
            "portIndex": 0,
            "timeoutSeconds": 10,
            "maxConsecutiveFailures": 3
        },
        {
            "protocol": "TCP",
            "gracePeriodSeconds": 3,
            "intervalSeconds": 5,
            "portIndex": 1,
            "timeoutSeconds": 5,
            "maxConsecutiveFailures": 3
        },
        {
            "protocol": "COMMAND",
            "command": { "value": "curl -f -X GET http://$HOST:$PORT0/health" },
            "maxConsecutiveFailures": 3
        }
    ],
    "id": "my-app",
    "instances": 3,
    "mem": 256.0,
    "ports": [
        8080,
        9000
    ],
    "backoffSeconds": 1,
    "backoffFactor": 1.15,
    "tasksRunning": 3, 
    "tasksStaged": 0, 
    "uris": [
        "https://raw.github.com/mesosphere/marathon/master/README.md"
    ],
    "dependencies": ["/product/db/mongo", "/product/db", "../../db"],
    "upgradeStrategy": {
        "minimumHealthCapacity": 0.5
    },
    "version": "2014-03-01T23:29:30.158Z"
}
{% endhighlight %}

##### id

Unique identifier for the app consisting of a series of names separated by slashes.
Each name must be at least 1 character and may
only contain digits (`0-9`), dashes (`-`), dots (`.`), and lowercase letters
(`a-z`). The name may not begin or end with a dash.

The allowable format is represented by the following regular expression  
`^(([a-z0-9]|[a-z0-9][a-z0-9\\-]*[a-z0-9])\\.)*([a-z0-9]|[a-z0-9][a-z0-9\\-]*[a-z0-9])$`

##### args

An array of strings that represents an alternative mode of specifying the command to run. This was motivated by safe usage of containerizer features like a custom Docker ENTRYPOINT. This args field may be used in place of cmd even when using the default command executor. This change mirrors API and semantics changes in the Mesos CommandInfo protobuf message starting with version `0.20.0`.  Either `cmd` or `args` must be supplied. It is invalid to supply both `cmd` and `args` in the same app.

##### backoffSeconds, backoffFactor

Configures exponential backoff behavior when launching potentially sick apps.
This prevents sandboxes associated with consecutively failing tasks from
filling up the hard disk on Mesos slaves. The backoff period is multiplied by
the factor for each consecutive failure.  This applies also to tasks that are
killed due to failing too many health checks.

##### cmd

The command that is executed.  This value is wrapped by Mesos via `/bin/sh -c ${app.cmd}`.  Either `cmd` or `args` must be supplied. It is invalid to supply both `cmd` and `args` in the same app.

##### constraints

Valid constraint operators are one of ["UNIQUE", "CLUSTER",
"GROUP_BY"]. For additional information on using placement constraints see
the [Constraints doc page]({{ site.baseurl }}/docs/constraints.html).

##### container

Additional data passed to the containerizer on application launch.  These consist of a type, zero or more volumes, and additional type-specific options.  Volumes and type are optional (the default type is DOCKER).  In order to make use of the docker containerizer, specify `--containerizers=docker,mesos` to the Mesos slave.

##### dependencies

A list of services upon which this application depends. An order is derived from the dependencies for performing start/stop and upgrade of the application.  For example, an application `/a` relies on the services `/b` which itself relies on `/c`. To start all 3 applications, first `/c` is started than `/b` than `/a`.

##### deployments (read-only)

A list of currently running deployments that affect this application.
If this array is nonempty, then this app is locked for updates.

##### healthChecks

An array of checks to be performed on running tasks to determine if they are
operating as expected. Health checks begin immediately upon task launch. For
design details, refer to the [health checks]({{ site.baseurl}}/docs/health-checks.html)
doc.  By default, health checks are executed by the Marathon scheduler.
It's possible with Mesos `0.20.0` and higher to execute health checks on the hosts where
the tasks are running by supplying the `--executor_health_checks` flag to Marathon.
In this case, the only supported protocol is `COMMAND` and each app is limited to
at most one defined health check.

An HTTP health check is considered passing if (1) its HTTP response code is between
200 and 399, inclusive, and (2) its response is received within the
`timeoutSeconds` period. If a task fails more than `maxConseutiveFailures`
health checks consecutively, that task is killed.

###### HEALTH CHECK OPTIONS

* `command`: Command to run in order to determine the health of a task.
  *Note: only used if `protocol == "COMMAND"`, and only available if Marathon is
  started with the `--executor_health_checks` flag.*
* `gracePeriodSeconds` (Optional. Default: 15): Health check failures are
  ignored within this number of seconds of the task being started or until the
  task becomes healthy for the first time.
* `intervalSeconds` (Optional. Default: 10): Number of seconds to wait between
  health checks.
* `maxConsecutiveFailures`(Optional. Default: 3) : Number of consecutive health
  check failures after which the unhealthy task should be killed.
* `protocol` (Optional. Default: "HTTP"): Protocol of the requests to be
  performed. One of "HTTP", "TCP", or "COMMAND".
* `path` (Optional. Default: "/"): Path to endpoint exposed by the task that
  will provide health  status. Example: "/path/to/health".
  _Note: only used if `protocol == "HTTP"`._
* `portIndex` (Optional. Default: 0): Index in this app's `ports` array to be
  used for health requests. An index is used so the app can use random ports,
  like "[0, 0, 0]" for example, and tasks could be started with port environment
  variables like `$PORT1`.
* `timeoutSeconds` (Optional. Default: 20): Number of seconds after which a
  health check is considered a failure regardless of the response.

##### ports

An array of required port resources on the host. To generate one or more
arbitrary free ports for each application instance, pass zeros as port
values. Each port value is exposed to the instance via environment variables
`$PORT0`, `$PORT1`, etc. Ports assigned to running instances are also available
via the task resource.

##### upgradeStrategy
During an upgrade all instances of an application get replaced by a new version.
The `minimumHealthCapacity` defines the minimum number of healthy nodes, that do not sacrifice overall application purpose. 
It is a number between `0` and `1` which is multiplied with the instance count. 
The default `minimumHealthCapacity` is `1`, which means no old instance can be stopped, before all new instances are deployed. 
A value of `0.5` means that an upgrade can be deployed side by side, by taking half of the instances down in the first step, 
deploy half of the new version and than take the other half down and deploy the rest. 
A value of `0` means take all instances down immediately and replace with the new application.

##### Example

**Request:**


{% highlight http %}
POST /v2/apps HTTP/1.1
Accept: application/json
Accept-Encoding: gzip, deflate
Content-Length: 562
Content-Type: application/json; charset=utf-8
Host: mesos.vm:8080
User-Agent: HTTPie/0.8.0

{
    "cmd": "env && python3 -m http.server $PORT0", 
    "constraints": [
        [
            "hostname", 
            "UNIQUE"
        ]
    ], 
    "container": {
        "docker": {
            "image": "python:3"
        }, 
        "type": "DOCKER"
    }, 
    "cpus": 0.25, 
    "healthChecks": [
        {
            "gracePeriodSeconds": 3, 
            "intervalSeconds": 10, 
            "maxConsecutiveFailures": 3, 
            "path": "/", 
            "portIndex": 0, 
            "protocol": "HTTP", 
            "timeoutSeconds": 5
        }
    ], 
    "id": "my-app", 
    "instances": 2, 
    "mem": 50, 
    "ports": [
        0
    ], 
    "upgradeStrategy": {
        "minimumHealthCapacity": 0.5
    }
}
{% endhighlight json %}

**Response:**


{% highlight http %}
HTTP/1.1 201 Created
Content-Type: application/json
Location: http://mesos.vm:8080/v2/apps/my-app
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "args": null, 
    "backoffFactor": 1.15, 
    "backoffSeconds": 1, 
    "cmd": "env && python3 -m http.server $PORT0", 
    "constraints": [
        [
            "hostname", 
            "UNIQUE"
        ]
    ], 
    "container": {
        "docker": {
            "image": "python:3"
        }, 
        "type": "DOCKER", 
        "volumes": []
    }, 
    "cpus": 0.25, 
    "dependencies": [], 
    "disk": 0.0, 
    "env": {}, 
    "executor": "", 
    "healthChecks": [
        {
            "command": null, 
            "gracePeriodSeconds": 3, 
            "intervalSeconds": 10, 
            "maxConsecutiveFailures": 3, 
            "path": "/", 
            "portIndex": 0, 
            "protocol": "HTTP", 
            "timeoutSeconds": 5
        }
    ], 
    "id": "/my-app", 
    "instances": 2, 
    "mem": 50.0, 
    "ports": [
        0
    ], 
    "requirePorts": false, 
    "storeUrls": [], 
    "upgradeStrategy": {
        "minimumHealthCapacity": 0.5
    }, 
    "uris": [], 
    "user": null, 
    "version": "2014-08-18T22:36:41.451Z"
}
{% endhighlight %}

#### GET `/v2/apps`

List all running applications.

##### Example

**Request:**

{% highlight http %}
GET /v2/apps HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Host: mesos.vm:8080
User-Agent: HTTPie/0.8.0
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "apps": [
        {
            "args": null, 
            "backoffFactor": 1.15, 
            "backoffSeconds": 1, 
            "cmd": "env && python3 -m http.server $PORT0", 
            "constraints": [
                [
                    "hostname", 
                    "UNIQUE"
                ]
            ], 
            "container": {
                "docker": {
                    "image": "python:3"
                }, 
                "type": "DOCKER", 
                "volumes": []
            }, 
            "cpus": 0.25, 
            "dependencies": [],
            "deployments": [
                "3423117e-858b-4af6-8424-b6a2f3866d3a"
            ], 
            "disk": 0.0, 
            "env": {}, 
            "executor": "", 
            "healthChecks": [
                {
                    "command": null, 
                    "gracePeriodSeconds": 3, 
                    "intervalSeconds": 10, 
                    "maxConsecutiveFailures": 3, 
                    "path": "/", 
                    "portIndex": 0, 
                    "protocol": "HTTP", 
                    "timeoutSeconds": 5
                }
            ], 
            "id": "/my-app", 
            "instances": 2, 
            "mem": 50.0, 
            "ports": [
                10000
            ], 
            "requirePorts": false, 
            "storeUrls": [], 
            "tasksRunning": 1, 
            "tasksStaged": 0, 
            "upgradeStrategy": {
                "minimumHealthCapacity": 0.5
            }, 
            "uris": [], 
            "user": null, 
            "version": "2014-08-18T22:36:41.451Z"
        }
    ]
}
{% endhighlight %}

#### GET `/v2/apps?cmd={command}`

List all running applications, filtered by `command`.

##### Example

**Request:**

{% highlight http %}
GET /v2/apps?cmd=env HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Host: mesos.vm:8080
User-Agent: HTTPie/0.8.0
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "apps": [
        {
            "args": null, 
            "backoffFactor": 1.15, 
            "backoffSeconds": 1, 
            "cmd": "env && sleep 300", 
            "constraints": [], 
            "container": {
                "docker": {
                    "image": "busybox"
                }, 
                "type": "DOCKER", 
                "volumes": []
            }, 
            "cpus": 0.5, 
            "dependencies": [], 
            "deployments": [
                "b2a6d112-d709-480b-82c2-0f618924845e"
            ], 
            "disk": 0.0, 
            "env": {}, 
            "executor": "", 
            "healthChecks": [], 
            "id": "/sleepy-docker", 
            "instances": 6, 
            "mem": 64.0, 
            "ports": [
                10001
            ], 
            "requirePorts": false, 
            "storeUrls": [], 
            "tasksRunning": 1, 
            "tasksStaged": 0, 
            "upgradeStrategy": {
                "minimumHealthCapacity": 1.0
            }, 
            "uris": [], 
            "user": null, 
            "version": "2014-08-29T08:00:40.293Z"
        }
    ]
}
{% endhighlight %}

#### GET `/v2/apps?embed=apps.tasks`

List all apps, with their running tasks.

##### Example

**Request:**

{% highlight http %}
GET /v2/apps?embed=apps.tasks HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Host: localhost:8080
User-Agent: HTTPie/0.8.0
{% endhighlight %}

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "apps": [
        {
            "args": null, 
            "backoffFactor": 1.15, 
            "backoffSeconds": 1, 
            "cmd": "python toggle.py $PORT0", 
            "constraints": [], 
            "container": null, 
            "cpus": 0.2, 
            "dependencies": [], 
            "deployments": [
                "d27b3347-8408-4366-8cf1-993ffb258a96"
            ],
            "disk": 0.0, 
            "env": {}, 
            "executor": "", 
            "healthChecks": [], 
            "id": "/test", 
            "instances": 2, 
            "mem": 32.0, 
            "ports": [
                10000
            ], 
            "requirePorts": false, 
            "storeUrls": [], 
            "tasks": [
                {
                    "host": "10.141.141.10",
                    "id": "test.6520607d-2cde-11e4-8852-56847afe9799",
                    "ports": [31369], 
                    "stagedAt": "2014-08-26T05:04:01.895Z", 
                    "startedAt": "2014-08-26T06:59:22.252Z", 
                    "version": "2014-08-26T05:03:26.449Z"
                },
                {
                    "host": "10.141.141.10", 
                    "id": "test.53cf64b8-2cde-11e4-8852-56847afe9799", 
                    "ports": [31124], 
                    "stagedAt": "2014-08-26T05:03:32.844Z", 
                    "startedAt": "2014-08-26T06:59:22.186Z", 
                    "version": "2014-08-26T05:03:26.449Z"
                }
            ],
            "tasksRunning": 2,
            "tasksStaged": 0, 
            "upgradeStrategy": {
                "minimumHealthCapacity": 1.0
            }, 
            "uris": [
                "http://downloads.mesosphere.io/misc/toggle.tgz"
            ], 
            "user": null, 
            "version": "2014-08-26T05:04:26.263Z"
        }
    ]
}
{% endhighlight %}

**Response:**

#### GET `/v2/apps/{appId}`

List the application with id `appId`.

##### Example

**Request:**

{% highlight http %}
GET /v2/apps/sleepy-docker HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Host: mesos.vm:8080
User-Agent: HTTPie/0.8.0
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "app": {
        "args": null, 
        "backoffFactor": 1.15, 
        "backoffSeconds": 1, 
        "cmd": "env && sleep 300", 
        "constraints": [], 
        "container": {
            "docker": {
                "image": "busybox"
            }, 
            "type": "DOCKER", 
            "volumes": []
        }, 
        "cpus": 0.5, 
        "dependencies": [], 
        "deployments": [], 
        "disk": 0.0, 
        "env": {}, 
        "executor": "", 
        "healthChecks": [], 
        "id": "/sleepy-docker", 
        "instances": 6, 
        "mem": 64.0, 
        "ports": [
            10001
        ], 
        "requirePorts": false, 
        "storeUrls": [], 
        "tasks": [
            {
                "host": "10.141.141.10", 
                "id": "sleepy-docker.ae5ececf-2f52-11e4-86eb-56847afe9799", 
                "ports": [
                    31033
                ], 
                "stagedAt": "2014-08-29T08:01:28.724Z", 
                "startedAt": "2014-08-29T08:02:00.953Z", 
                "version": "2014-08-29T08:00:40.293Z"
            }, 
            {
                "host": "10.141.141.10", 
                "id": "sleepy-docker.938184d3-2f52-11e4-86eb-56847afe9799", 
                "ports": [
                    31136
                ], 
                "stagedAt": "2014-08-29T08:00:43.653Z", 
                "startedAt": "2014-08-29T08:02:00.930Z", 
                "version": "2014-08-29T08:00:40.293Z"
            }, 
            {
                "host": "10.141.141.10", 
                "id": "sleepy-docker.c0e0f797-2f52-11e4-86eb-56847afe9799", 
                "ports": [
                    31031
                ], 
                "stagedAt": "2014-08-29T08:01:59.776Z", 
                "startedAt": null, 
                "version": "2014-08-29T08:00:40.293Z"
            }
        ], 
        "tasksRunning": 2, 
        "tasksStaged": 1, 
        "upgradeStrategy": {
            "minimumHealthCapacity": 1.0
        }, 
        "uris": [], 
        "user": null, 
        "version": "2014-08-29T08:00:40.293Z"
    }
}
{% endhighlight %}

#### GET `/v2/apps/{appId}/versions`

List the versions of the application with id `appId`.

##### Example

**Request:**

{% highlight http %}
GET /v2/apps/my-app/versions HTTP/1.1
Accept: application/json
Accept-Encoding: gzip, deflate, compress
Content-Type: application/json; charset=utf-8
Host: localhost:8080
User-Agent: HTTPie/0.7.2
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "versions": [
        "2014-04-04T06:25:31.399Z"
    ]
}
{% endhighlight %}

#### GET `/v2/apps/{appId}/versions/{version}`

List the configuration of the application with id `appId` at version `version`.

##### Example

**Request:**

{% highlight http %}
GET /v2/apps/my-app/versions/2014-03-01T23:17:50.295Z HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate, compress
Host: localhost:8080
User-Agent: HTTPie/0.7.2
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "cmd": "sleep 60", 
    "constraints": [], 
    "container": null, 
    "cpus": 0.1, 
    "env": {}, 
    "executor": "", 
    "id": "my-app", 
    "instances": 4, 
    "mem": 5.0, 
    "ports": [
        18027, 
        13200
    ], 
    "uris": [
        "https://raw.github.com/mesosphere/marathon/master/README.md"
    ], 
    "version": "2014-03-01T23:17:50.295Z"
}
{% endhighlight %}

#### PUT `/v2/apps/{appId}`

Change parameters of a running application.  The new application parameters
apply only to subsequently created tasks.  Currently running tasks are
restarted, while maintaining the `minimumHealthCapacity`

##### Example

**Request:**

{% highlight http %}
PUT /v2/apps/my-app HTTP/1.1
Accept: application/json
Accept-Encoding: gzip, deflate, compress
Content-Length: 126
Content-Type: application/json; charset=utf-8
Host: localhost:8080
User-Agent: HTTPie/0.7.2

{
    "cmd": "sleep 55", 
    "constraints": [
        [
            "hostname", 
            "UNIQUE", 
            ""
        ]
    ], 
    "cpus": "0.3", 
    "instances": "2", 
    "mem": "9", 
    "ports": [
        9000
    ]
}
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "deploymentId": "83b215a6-4e26-4e44-9333-5c385eda6438", 
    "version": "2014-08-26T07:37:50.462Z"
}
{% endhighlight %}

##### Example (version rollback)

If the `version` key is supplied in the JSON body, the rest of the object is ignored.  If the supplied version is known, then the app is updated (a new version is created) with those parameters.  Otherwise, if the supplied version is not known Marathon responds with a 404.

**Request:**

{% highlight http %}
PUT /v2/apps/my-app HTTP/1.1
Accept: application/json
Accept-Encoding: gzip, deflate, compress
Content-Length: 39
Content-Type: application/json; charset=utf-8
Host: localhost:8080
User-Agent: HTTPie/0.7.2

{
    "version": "2014-03-01T23:17:50.295Z"
}
{% endhighlight %}

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "deploymentId": "83b215a6-4e26-4e44-9333-5c385eda6438", 
    "version": "2014-08-26T07:37:50.462Z"
}
{% endhighlight %}

##### Example (update an app that is locked by a running deployment)

If the app is affected by a currently running deployment, then the
update operation fails.  As indicated by the response message, the current
deployment can be overridden by setting the `force` query parameter in a
subsequent request.

**Request:**

{% highlight http %}
PUT /v2/apps/test HTTP/1.1
Accept: application/json
Accept-Encoding: gzip, deflate
Content-Length: 18
Content-Type: application/json; charset=utf-8
Host: localhost:8080
User-Agent: HTTPie/0.8.0

{
    "instances": "2"
}
{% endhighlight %}

{% highlight http %}
HTTP/1.1 409 Conflict
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "deployments": [
        "631061a4-d785-4572-9f49-d9561c47c53b"
    ], 
    "message": "App is locked by one or more deployments. Override with the option '?force=true'. View details at '/v2/deployments/<DEPLOYMENT_ID>'."
}
{% endhighlight %}

#### DELETE `/v2/apps/{appId}`

Destroy an application. All data about that application will be deleted.

##### Example

**Request:**

{% highlight http %}
DELETE /v2/apps/my-app HTTP/1.1
Accept: application/json
Accept-Encoding: gzip, deflate, compress
Content-Length: 0
Content-Type: application/json; charset=utf-8
Host: localhost:8080
User-Agent: HTTPie/0.7.2
{% endhighlight %}


**Response:**

{% highlight http %}
HTTP/1.1 204 No Content
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
{% endhighlight %}


#### GET `/v2/apps/{appId}/tasks`

List all running tasks for application `appId`.

##### Example (as JSON)

**Request:**

{% highlight http %}
GET /v2/apps/my-app/tasks HTTP/1.1
Accept: application/json
Accept-Encoding: gzip, deflate, compress
Content-Type: application/json; charset=utf-8
Host: localhost:8080
User-Agent: HTTPie/0.7.2
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "tasks": [
        {
            "host": "agouti.local", 
            "id": "my-app_1-1396592790353", 
            "ports": [
                31336, 
                31337
            ], 
            "stagedAt": "2014-04-04T06:26:30.355Z", 
            "startedAt": "2014-04-04T06:26:30.860Z", 
            "version": "2014-04-04T06:26:23.051Z"
        }, 
        {
            "host": "agouti.local", 
            "id": "my-app_0-1396592784349", 
            "ports": [
                31382, 
                31383
            ], 
            "stagedAt": "2014-04-04T06:26:24.351Z", 
            "startedAt": "2014-04-04T06:26:24.919Z", 
            "version": "2014-04-04T06:26:23.051Z"
        }
    ]
}
{% endhighlight %}

##### Example (as text)

**Request:**

{% highlight http %}
GET /v2/apps/my-app/tasks HTTP/1.1
Accept: text/plain
Accept-Encoding: gzip, deflate, compress
Host: localhost:8080
User-Agent: HTTPie/0.7.2
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: text/plain
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

my-app  19385 agouti.local:31336  agouti.local:31364  agouti.local:31382  
my-app  11186 agouti.local:31337  agouti.local:31365  agouti.local:31383  
{% endhighlight %}

#### DELETE `/v2/apps/{appId}/tasks?host={host}&scale={true|false}`

Kill tasks that belong to the application `appId`, optionally filtered by the
task's `host`.

The query parameters `host` and `scale` are both optional.  If `host` is
specified, only tasks running on the supplied slave are killed.  If
`scale=true` is specified, then the application is scaled down by the number of
killed tasks.  The `scale` parameter defaults to `false`.

##### Example

**Request:**

{% highlight http %}
DELETE /v2/apps/my-app/tasks?host=mesos.vm&scale=false HTTP/1.1
Accept: application/json
Accept-Encoding: gzip, deflate, compress
Content-Length: 0
Content-Type: application/json; charset=utf-8
Host: localhost:8080
User-Agent: HTTPie/0.7.2
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "tasks": []
}
{% endhighlight %}

#### DELETE `/v2/apps/{appId}/tasks/{taskId}?scale={true|false}`

Kill the task with ID `taskId` that belongs to the application `appId`.

The query parameter `scale` is optional.  If `scale=true` is specified, then
the application is scaled down one if the supplied `taskId` exists.  The
`scale` parameter defaults to `false`.

##### Example

**Request:**

{% highlight http %}
DELETE /v2/apps/my-app/tasks/my-app_3-1389916890411 HTTP/1.1
Accept: application/json
Accept-Encoding: gzip, deflate, compress
Content-Length: 0
Content-Type: application/json; charset=utf-8
Host: localhost:8080
User-Agent: HTTPie/0.7.2
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "task": {
        "host": "mesos.vm",
        "id": "my-app_3-1389916890411",
        "ports": [
            31509,
            31510
        ],
        "stagedAt": "2014-01-17T00:01+0000",
        "startedAt": "2014-01-17T00:01+0000"
    }
}
{% endhighlight %}

### Groups

#### GET `/v2/groups`

List all groups.

**Request:**

{% highlight http %}
GET /v2/groups HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Host: mesos.vm:8080
User-Agent: HTTPie/0.8.0
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "apps": [], 
    "dependencies": [], 
    "groups": [
        {
            "apps": [
                {
                    "args": null, 
                    "backoffFactor": 1.15, 
                    "backoffSeconds": 1, 
                    "cmd": "sleep 30", 
                    "constraints": [], 
                    "container": null, 
                    "cpus": 1.0, 
                    "dependencies": [], 
                    "disk": 0.0, 
                    "env": {}, 
                    "executor": "", 
                    "healthChecks": [], 
                    "id": "/test/app", 
                    "instances": 1, 
                    "mem": 128.0, 
                    "ports": [
                        10000
                    ], 
                    "requirePorts": false, 
                    "storeUrls": [], 
                    "upgradeStrategy": {
                        "minimumHealthCapacity": 1.0
                    }, 
                    "uris": [], 
                    "user": null, 
                    "version": "2014-08-28T01:05:40.586Z"
                }
            ], 
            "dependencies": [], 
            "groups": [], 
            "id": "/test", 
            "version": "2014-08-28T01:09:46.212Z"
        }
    ], 
    "id": "/", 
    "version": "2014-08-28T01:09:46.212Z"
}
{% endhighlight %}

#### GET `/v2/groups/{groupId}`

List the group with the specified ID.

**Request:**

{% highlight http %}
GET /v2/groups/test HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Host: mesos.vm:8080
User-Agent: HTTPie/0.8.0
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "apps": [
        {
            "args": null, 
            "backoffFactor": 1.15, 
            "backoffSeconds": 1, 
            "cmd": "sleep 30", 
            "constraints": [], 
            "container": null, 
            "cpus": 1.0, 
            "dependencies": [], 
            "disk": 0.0, 
            "env": {}, 
            "executor": "", 
            "healthChecks": [], 
            "id": "/test/app", 
            "instances": 1, 
            "mem": 128.0, 
            "ports": [
                10000
            ], 
            "requirePorts": false, 
            "storeUrls": [], 
            "upgradeStrategy": {
                "minimumHealthCapacity": 1.0
            }, 
            "uris": [], 
            "user": null, 
            "version": "2014-08-28T01:05:40.586Z"
        }
    ], 
    "dependencies": [], 
    "groups": [], 
    "id": "/test", 
    "version": "2014-08-28T01:09:46.212Z"
}
{% endhighlight %}

#### POST `/v2/groups`

Create and start a new application group.
Application groups can contain other application groups.
An application group can either hold other groups or applications, but can not be mixed in one.

The JSON format of a group resource is as follows:

```json
{
  "id": "product",
  "groups": [{
    "id": "service",
    "groups": [{
      "id": "us-east",
      "apps": [{
        "id": "app1",
        "cmd": "someExecutable"
      },  
      {
        "id": "app2",
        "cmd": "someOtherExecutable"
      }]
    }],
    "dependencies": ["/product/database", "../backend"]
  }
]
"version": "2014-03-01T23:29:30.158Z"
}
```

Since the deployment of the group can take a considerable amount of time, this endpoint returns immediatly with a version.
The failure or success of the action is signalled via event. There is a group_change_success and group_change_failed with
the given version.

### Example

**Request:**

{% highlight http %}
POST /v2/groups HTTP/1.1
User-Agent: curl/7.35.0
Accept: application/json
Host: localhost:8080
Content-Type: application/json
Content-Length: 273

{
  "id" : "product",
  "apps":[ 
    {
      "id": "myapp",
      "cmd": "ruby app2.rb",
      "instances": 1
    }
  ]
}
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 201 Created
Location: http://localhost:8080/v2/groups/product
Content-Type: application/json
Transfer-Encoding: chunked
Server: Jetty(8.y.z-SNAPSHOT)
{"version":"2014-07-01T10:20:50.196Z"}
{% endhighlight %}

Create and start a new application group.
Application groups can contain other application groups.
An application group can either hold other groups or applications, but can not be mixed in one.

The JSON format of a group resource is as follows:

```json
{
  "id": "product",
  "groups": [{
    "id": "service",
    "groups": [{
      "id": "us-east",
      "apps": [{
        "id": "app1",
        "cmd": "someExecutable"
      },  
      {
        "id": "app2",
        "cmd": "someOtherExecutable"
      }]
    }],
    "dependencies": ["/product/database", "../backend"]
  }
]
"version": "2014-03-01T23:29:30.158Z"
}
```

Since the deployment of the group can take a considerable amount of time, this endpoint returns immediatly with a version.
The failure or success of the action is signalled via event. There is a group_change_success and group_change_failed with
the given version.


### Example

**Request:**

{% highlight http %}
POST /v2/groups HTTP/1.1
User-Agent: curl/7.35.0
Accept: application/json
Host: localhost:8080
Content-Type: application/json
Content-Length: 273

{
  "id" : "product",
  "apps":[ 
    {
      "id": "myapp",
      "cmd": "ruby app2.rb",
      "instances": 1
    }
  ]
}
{% endhighlight %}

**Response:**


{% highlight http %}
HTTP/1.1 201 Created
Location: http://localhost:8080/v2/groups/product
Content-Type: application/json
Transfer-Encoding: chunked
Server: Jetty(8.y.z-SNAPSHOT)
{"version":"2014-07-01T10:20:50.196Z"}
{% endhighlight %}

#### PUT `/v2/groups/{groupId}`

Change parameters of a deployed application group.

* Changes to application parameters will result in a restart of this application.
* A new application added to the group is started.
* An existing application removed from the group gets stopped.

If there are no changes to the application definition, no restart is triggered.
During restart marathon keeps track, that the configured amount of minimal running instances are _always_ available.

A deployment can run forever. This is the case, when the new application has a problem and does not become healthy.
In this case, human interaction is needed with 2 possible choices:

* Rollback to an existing older version (use the rollback endpoint)
* Update with a newer version of the group which does not have the problems of the old one.

If there is an upgrade process already in progress, a new update will be rejected unless the force flag is set.
With the force flag given, a running upgrade is terminated and a new one is started.

Since the deployment of the group can take a considerable amount of time, this endpoint returns immediatly with a version.
The failure or success of the action is signalled via event. There is a group_change_success and group_change_failed with
the given version.

### Example

**Request:**

{% highlight http %}
PUT /v2/groups/test/project HTTP/1.1
Accept: application/json
Accept-Encoding: gzip, deflate
Content-Length: 541
Content-Type: application/json; charset=utf-8
Host: mesos.vm:8080
User-Agent: HTTPie/0.8.0

{
    "apps": [
        {
            "cmd": "ruby app2.rb", 
            "constraints": [], 
            "container": null, 
            "cpus": 0.2, 
            "env": {}, 
            "executor": "//cmd", 
            "healthChecks": [
                {
                    "initialDelaySeconds": 15, 
                    "intervalSeconds": 5, 
                    "path": "/health", 
                    "portIndex": 0, 
                    "protocol": "HTTP", 
                    "timeoutSeconds": 15
                }
            ], 
            "id": "app", 
            "instances": 6, 
            "mem": 128.0, 
            "ports": [
                19970
            ], 
            "taskRateLimit": 1.0, 
            "uris": []
        }
    ]
}
{% endhighlight %}

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "deploymentId": "c0e7434c-df47-4d23-99f1-78bd78662231", 
    "version": "2014-08-28T16:45:41.063Z"
}
{% endhighlight %}

### Example

Scale a group.

The scaling affects apps directly in the group as well as all transitive applications referenced by subgroups of this group.
The scaling factor is applied to each individual instance count of each application.

Since the deployment of the group can take a considerable amount of time, this endpoint returns immediatly with a version.
The failure or success of the action is signalled via event. There is a group_change_success and group_change_failed with
the given version.

**Request:**

{% highlight http %}
PUT /v2/groups/product/service HTTP/1.1
Content-Length: 123
Host: localhost:8080
User-Agent: HTTPie/0.7.2
{ "scaleBy": 1.5 }
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "deploymentId": "c0e7434c-df47-4d23-99f1-78bd78662231", 
    "version": "2014-08-28T16:45:41.063Z"
}
{% endhighlight %}


#### PUT `/v2/groups/{groupId}/version/{version}`

Rollback this group to a previous version.
All changes to a group will create a new version.
With this endpoint it is possible to an older version of this group.

If there is an upgrade process already in progress, this rollback will be rejected unless the force flag is set.
With the force flag given, a running upgrade is terminated and a new one is started.

The rollback to a version is handled as normal update.
All implications of an update will take place.

Since the deployment of the group can take a considerable amount of time, this endpoint returns immediatly with a version.
The failure or success of the action is signalled via event. There is a group_change_success and group_change_failed with
the given version.


### Example

**Request:**

{% highlight http %}
PUT /v2/groups/myproduct/version/2014-03-01T23:29:30.158?force=true HTTP/1.1
Content-Length: 0
Host: localhost:8080
User-Agent: HTTPie/0.7.2
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "deploymentId": "c0e7434c-df47-4d23-99f1-78bd78662231", 
    "version": "2014-08-28T16:45:41.063Z"
}
{% endhighlight %}


#### DELETE `/v2/groups/{groupId}`

Destroy a group. All data about that group and all associated applications will be deleted.

Since the deployment of the group can take a considerable amount of time, this endpoint returns immediatly with a version.
The failure or success of the action is signalled via event. There is a group_change_success and group_change_failed with
the given version.

**Request:**

{% highlight http %}
DELETE /v2/groups/product/service/app HTTP/1.1
Accept: application/json
Accept-Encoding: gzip, deflate, compress
Content-Length: 0
Content-Type: application/json; charset=utf-8
Host: localhost:8080
User-Agent: curl/7.35.0
{% endhighlight %}


**Response:**

{% highlight http %}
HTTP/1.1 200 Ok
Content-Type: application/json
Transfer-Encoding: chunked
Server: Jetty(8.y.z-SNAPSHOT)
{"version":"2014-07-01T10:20:50.196Z"}
{% endhighlight %}

### Tasks

#### GET `/v2/tasks`

List tasks of all applications.

##### Example (as JSON)

**Request:**

{% highlight http %}
GET /v2/tasks HTTP/1.1
Accept: application/json
Accept-Encoding: gzip, deflate, compress
Content-Type: application/json; charset=utf-8
Host: localhost:8080
User-Agent: HTTPie/0.7.2
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "tasks": [
        {
            "appId": "my-app", 
            "host": "agouti.local", 
            "id": "my-app_2-1396592796360", 
            "ports": [
                31364, 
                31365
            ], 
            "stagedAt": "2014-04-04T06:26:36.362Z", 
            "startedAt": "2014-04-04T06:26:37.285Z", 
            "version": "2014-04-04T06:26:23.051Z"
        }, 
        {
            "appId": "my-app", 
            "host": "agouti.local", 
            "id": "my-app_1-1396592790353", 
            "ports": [
                31336, 
                31337
            ], 
            "stagedAt": "2014-04-04T06:26:30.355Z", 
            "startedAt": "2014-04-04T06:26:30.860Z", 
            "version": "2014-04-04T06:26:23.051Z"
        }, 
        {
            "appId": "my-app", 
            "host": "agouti.local", 
            "id": "my-app_0-1396592784349", 
            "ports": [
                31382, 
                31383
            ], 
            "stagedAt": "2014-04-04T06:26:24.351Z", 
            "startedAt": "2014-04-04T06:26:24.919Z", 
            "version": "2014-04-04T06:26:23.051Z"
        }
    ]
}
{% endhighlight %}

#### GET `/v2/tasks?status=[running|staging]`

List running or staging tasks of all applications.

##### Example (as JSON)

**Request:**

{% highlight http %}
GET /v2/tasks HTTP/1.1
Accept: application/json
Accept-Encoding: gzip, deflate, compress
Content-Type: application/json; charset=utf-8
Host: localhost:8080
User-Agent: HTTPie/0.7.2
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "tasks": [
        {
            "appId": "my-app",
            "host": "agouti.local",
            "id": "my-app_0-1396592784349",
            "ports": [
                31382,
                31383
            ],
            "stagedAt": "2014-04-04T06:26:24.351Z",
            "startedAt": "2014-04-04T06:26:24.919Z",
            "version": "2014-04-04T06:26:23.051Z"
        }
    ]
}
{% endhighlight %}

##### Example (as text)

In text/plain only running tasks will be returned.

**Request:**

{% highlight http %}
GET /v2/tasks HTTP/1.1
Accept: text/plain
Accept-Encoding: gzip, deflate, compress
Host: localhost:8080
User-Agent: HTTPie/0.7.2
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: text/plain
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

my-app  19385 agouti.local:31336  agouti.local:31364  agouti.local:31382  
my-app  11186 agouti.local:31337  agouti.local:31365  agouti.local:31383  
{% endhighlight %}

### Deployments

<span class="label label-default">v0.7.0</span>

#### GET /v2/deployments

List running deployments

**Request:**

{% highlight http %}
GET /v2/deployments HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Host: mesos.vm:8080
User-Agent: HTTPie/0.8.0
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

[
    {
        "affectedApps": [
            "/test"
        ],
        "id": "867ed450-f6a8-4d33-9b0e-e11c5513990b",
        "steps": [
            [
                {
                    "action": "ScaleApplication",
                    "app": "/test"
                }
            ]
        ],
        "currentActions": [
          {
            "action": "ScaleApplication",
            "app": "/test"
          }
        ],
        "version": "2014-08-26T08:18:03.595Z",
        "currentStep": 1,
        "totalSteps" 1
    }
]
{% endhighlight %}

#### DELETE /v2/deployments/{deploymentId}

Cancel the deployment with `deploymentId`

**Request:**

{% highlight http %}
DELETE /v2/deployments/867ed450-f6a8-4d33-9b0e-e11c5513990b HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Content-Length: 0
Host: mesos.vm:8080
User-Agent: HTTPie/0.8.0
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "deploymentId": "0b1467fc-d5cd-4bbc-bac2-2805351cee1e", 
    "version": "2014-08-26T08:20:26.171Z"
}
{% endhighlight %}

### Event Subscriptions

#### POST /v2/eventSubscriptions?callbackUrl={url}

Register a callback URL as an event subscriber.

NOTE: To activate this endpoint, you need to startup Marathon with `--event_subscriber http_callback`.

**Request:**

{% highlight http %}
POST /v2/eventSubscriptions?callbackUrl=http://localhost:9292/callback HTTP/1.1
Accept: application/json
Accept-Encoding: gzip, deflate, compress
Content-Length: 0
Content-Type: application/json; charset=utf-8
Host: localhost:8080
User-Agent: HTTPie/0.7.2
{% endhighlight %}


**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "callbackUrl": "http://localhost:9292/callback", 
    "clientIp": "0:0:0:0:0:0:0:1", 
    "eventType": "subscribe_event"
}
{% endhighlight %}

#### GET `/v2/eventSubscriptions`

List all event subscriber callback URLs.

NOTE: To activate this endpoint, you need to startup Marathon with `--event_subscriber http_callback`.

##### Example

**Request:**

{% highlight http %}
GET /v2/eventSubscriptions HTTP/1.1
Accept: application/json
Accept-Encoding: gzip, deflate, compress
Content-Type: application/json; charset=utf-8
Host: localhost:8080
User-Agent: HTTPie/0.7.2
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "callbackUrls": [
        "http://localhost:9292/callback"
    ]
}
{% endhighlight %}

#### DELETE `/v2/eventSubscriptions?callbackUrl={url}`

Unregister a callback URL from the event subscribers list.

NOTE: To activate this endpoint, you need to startup Marathon with `--event_subscriber http_callback`.

##### Example

**Request:**

{% highlight http %}
DELETE /v2/eventSubscriptions?callbackUrl=http://localhost:9292/callback HTTP/1.1
Accept: application/json
Accept-Encoding: gzip, deflate, compress
Content-Length: 0
Content-Type: application/json; charset=utf-8
Host: localhost:8080
User-Agent: HTTPie/0.7.2
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "callbackUrl": "http://localhost:9292/callback", 
    "clientIp": "0:0:0:0:0:0:0:1", 
    "eventType": "unsubscribe_event"
}
{% endhighlight %}

### Queue

<span class="label label-default">v0.7.0</span>

#### GET `/v2/queue`

Show content of the task queue.

##### Example

{% highlight http %}
GET /v2/queue HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Host: localhost:8080
User-Agent: HTTPie/0.8.0
{% endhighlight %}

{% highlight http %}
HTTP/1.1 200 OK
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "queue": [
        {
            "app": {
                "args": null,
                "backoffFactor": 1.15,
                "backoffSeconds": 1,
                "cmd": "python toggle.py $PORT0",
                "constraints": [],
                "container": null,
                "cpus": 0.2,
                "dependencies": [],
                "disk": 0.0,
                "env": {},
                "executor": "",
                "healthChecks": [],
                "id": "/test",
                "instances": 3,
                "mem": 32.0,
                "ports": [10000],
                "requirePorts": false,
                "storeUrls": [],
                "upgradeStrategy": {
                    "minimumHealthCapacity": 1.0
                },
                "uris": [
                    "http://downloads.mesosphere.io/misc/toggle.tgz"
                ],
                "user": null,
                "version": "2014-08-26T05:04:49.766Z"
            },
            "delay": { "overdue": true }
        }
    ]
}
{% endhighlight %}

### Server Info

<span class="label label-default">v0.7.0</span>

#### GET `/v2/info`

Get info about the Marathon Instance

**Request:**

{% highlight http %}
GET /v2/info HTTP/1.1
Accept: application/json
Accept-Encoding: gzip, deflate, compress
Content-Type: application/json; charset=utf-8
Host: localhost:8080
User-Agent: HTTPie/0.7.2
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Content-Length: 872
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)

{
    "frameworkId": "20140730-222531-1863654316-5050-10422-0000", 
    "leader": "127.0.0.1:8080", 
    "marathon_config": {
        "checkpoint": false, 
        "executor": "//cmd", 
        "failover_timeout": 604800, 
        "ha": true, 
        "hostname": "127.0.0.1", 
        "local_port_max": 49151, 
        "local_port_min": 32767, 
        "master": "zk://localhost:2181/mesos", 
        "mesos_role": null, 
        "mesos_user": "root", 
        "reconciliation_initial_delay": 30000, 
        "reconciliation_interval": 30000, 
        "task_launch_timeout": 60000
    }, 
    "name": "marathon", 
    "version": "0.7.0-SNAPSHOT", 
    "zookeeper_config": {
        "zk": "zk://localhost:2181/marathon", 
        "zk_future_timeout": {
            "duration": 10
        }, 
        "zk_hosts": "localhost:2181", 
        "zk_path": "/marathon", 
        "zk_state": "/marathon", 
        "zk_timeout": 10
    }
}
{% endhighlight %}

### Miscellaneous

**Request:**

{% highlight http %}
{% endhighlight %}

**Response:**

{% highlight http %}
{% endhighlight %}

#### GET `/ping`

**Request:**

{% highlight http %}
GET /ping HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Host: mesos.vm:8080
User-Agent: HTTPie/0.8.0
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Access-Control-Allow-Credentials: true
Cache-Control: must-revalidate,no-cache,no-store
Content-Length: 5
Content-Type: text/plain;charset=ISO-8859-1
Server: Jetty(8.y.z-SNAPSHOT)

pong
{% endhighlight %}

#### GET `/logging`

**Request:**

{% highlight http %}
GET /logging HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Host: mesos.vm:8080
User-Agent: HTTPie/0.8.0{% endhighlight %}

**Response:**

_HTML-only endpoint_

#### GET `/help`

**Request:**

{% highlight http %}
GET /help HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Host: mesos.vm:8080
User-Agent: HTTPie/0.8.0
{% endhighlight %}

**Response:**

_HTML-only endpoint_

#### GET `/metrics`

**Request:**

{% highlight http %}
GET /metrics HTTP/1.1
Accept: */*
Accept-Encoding: gzip, deflate
Host: mesos.vm:8080
User-Agent: HTTPie/0.8.0
{% endhighlight %}

**Response:**

{% highlight http %}
HTTP/1.1 200 OK
Cache-Control: must-revalidate,no-cache,no-store
Content-Type: application/json
Server: Jetty(8.y.z-SNAPSHOT)
Transfer-Encoding: chunked

{
    "counters": {
        ...
    }, 
    "gauges": {
        ...
    }, 
    "histograms": {}, 
    "meters": {
        ...
    }, 
    "timers": {
        ...
    }, 
    "version": "3.0.0"
}
{% endhighlight %}
