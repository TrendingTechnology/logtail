# logtail is a web socket log tailing tool.

## 1. Features
- tailing command output
- support multiple command tailing
- support webhook notice (include dingtalk)
- support websocket tailing
- support log matching filter

## 2. Architecture

![](https://github.com/vogo/vogo.github.io/raw/master/logtail/logtail-architecture.png)

## 3. Usage

### 3.1. install logtail

`go get -u github.com/vogo/logtail/cmd/logtail@master`

### 3.2. start logtail server

usage: `logtail -port=<port> -cmd="<cmd>"`

examples:

```bash
# tailing file logs
logtail -port=54321 -cmd="tailf /home/my/logs/myapp.log"

# tailing kubectl logs
logtail -port=54322 -cmd="kubectl logs --tail 10 -f \$(kubectl get pods --selector=app=myapp -o jsonpath='{.items[*].metadata.name}')"

# using a config file
logtail -file=/Users/wongoo/logtail-config.json
```

config file sample:
```json
{
  "port": 54321,
  "default_format":{
    "prefix": "!!!!-!!-!!"
  },
  "global_routers": [
    {
      "matchers": [],
      "transfers": [
        {"type": "console"}
      ]
    }
  ],
  "default_routers": [
    {
      "matchers": [],
      "transfers": [
        {"type": "console"}
      ]
    }
  ],
  "servers": [
    {
      "id": "app1",
      "command": "tail -f /Users/wongoo/app/app1.log",
      "routers": [
        {
          "matchers": [
            {"match_contains": "ERROR"}
          ],
          "transfers": [
            {
              "type": "ding",
              "ding_url": "https://oapi.dingtalk.com/robot/send?access_token=<token>"
            },
            {
              "type": "webhook",
              "webhook_url": "http://127.0.0.1:9000"
            }
          ]
        }
      ]
    },
    {
      "id": "app2",
      "command": "tail -f /Users/wongoo/app/app2.log"
    }
  ]
}
```

### 3.3. tailing logs

browse `http://<server-ip>:<port>` to list all tailing logs.

## 4. log format

You can config log format globally, or config it for a server.

The config `prefix` of the format is the wildcard of the prefix of a new log record, 
`logtail` will check whether a new line is the start of a new log record, or one of the following lines.

The wildcard does NOT support '*' for none or many chars, it supports the following tag:
- '?' as one byte char;
- '~' as one alphabet char;
- '!' as one number char;
- other chars must exact match.

example:
```bash
{
    "default_format":{
      "prefix": "!!!!-!!-!!"  # matches 2020-12-12
    },
    "servers": [
        {
          "id": "app1",
          "command": "tail -f /Users/wongoo/app/app1.log",
          "format":{
            "prefix": "!!!!-!!-!!"
          }
        }
    ]
}
```

## 5. command examples

```bash
# tail log file
tail -f /usr/local/myapp/myapp.log

# k8s: find and tail logs for the single pod of myapp
kubectl logs --tail 10 -f $(kubectl get pods --selector=app=myapp -o jsonpath='{.items[*].metadata.name}')

# k8s: find and tail logs for the myapp deployment (multiple pods)
kubectl logs --tail 10 -f deployment/$(kubectl get deployments --selector=project-name=myapp -o jsonpath='{.items[*].metadata.name}')

# k8s: find and tail logs for the latest version of the myapp deployment
s=$(kubectl get deployments --selector=project-name=myapp -o jsonpath='{.items[*].metadata.name}');s=${s##* };kubectl logs --tail 10 -f deployment/$s

```