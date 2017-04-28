# DCOS Universe README
Need to make some updates to DCOS Universe to get up and running quickly? The following steps should help you hit the ground running.

# Build
The Universe server builder requires jq, python3 and jsonschema. To install on OSX:

```
brew install python3 jq
pip3 install jsonschema
```

To package up the Universe server for use:

```
scripts/build.sh
docker/server/build.bash
docker tag mesosphere/universe-server:dev appliedis/universe-server
docker push appliedis/universe-server
```

Go to DCOS and restart the Universe service so your sweet apps show up:
https://dcos.aisohio.net/#/services/%2Funiverse/

# Use
The Universe server must be launched into the cluster. This can be done as follows using the CLI:

```
echo '{
  "id": "/universe",
  "instances": 1,
  "cpus": 0.25,
  "mem": 128,
  "requirePorts": true,
  "container": {
    "type": "DOCKER",
    "docker": {
      "network": "BRIDGE",
      "image": "appliedis/universe-server",
      "portMappings": [
        {
          "containerPort": 80,
          "hostPort": 8085,
          "protocol": "tcp"
        }
      ]
    },
    "volumes": []
  },
  "healthChecks": [
    {
      "gracePeriodSeconds": 120,
      "intervalSeconds": 30,
      "maxConsecutiveFailures": 3,
      "path": "/repo-empty-v3.json",
      "portIndex": 0,
      "protocol": "HTTP",
      "timeoutSeconds": 5
    }
  ],
  "constraints": [
    ["hostname", "UNIQUE"]
  ]
}' > marathon.json
dcos marathon app add marathon.json
dcos package repo add --index=0 dev-universe http://universe.marathon.mesos:8085/repo
```

