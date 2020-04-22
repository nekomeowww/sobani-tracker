# sobani-tracker

## Introduction

Aim to set up a tracker server for sobani service and clients

## Setup with Docker

Clone to your instance

```
git clone https://github.com/nekomeowww/sobani-tracker.git
```

```
cd sobani-tracker

# default port is 3000
# if you want another port
# please edit in deploy-docker.sh
# if you are using this with docker composer
# please change config.json and Dockerfile

# setup by script
chmod +x ./deploy-docker.sh
./deploy-docker.sh

# or setup manually
docker build -t sobani-tracker .
docker run -p 3000:3000/udp \
  --restart=always \
  -v ./config:/usr/src/sobani-tracker/config \
  sobani-tracker

# test
# default tracker set to 'localhost:3000'
node test/test-client-only.js
```

## Setup with Docker Composer

Clone to your instance

```
git clone https://github.com/nekomeowww/sobani-tracker.git
```

```
cd sobani-tracker

# default port is 3000
# if you want to use another port
#   please edit in docker-compose.yml
# docker-compose.yml will map `./config` in host to
#   `/usr/src/sobani-tracker/config` in container by default

chmod +x ./deploy-docker-compose.sh
./deploy-docker-compose.sh

# test
# default tracker set to 'localhost:3000'
node test/test-client-only.js
```

## Setup

Clone to your instance

```
git clone https://github.com/nekomeowww/sobani-tracker.git
```

Install dependencies

```
cd sobani-tracker
yarn install
```

Start server

```
yarn run test
```

## API Documentation

### Actions

#### announce

Announce action relates to the first connection from peer to server, within this action, a client send three parameters to tracker server in order to record and make the share action possible.

If sending is successful, an [announceReceived](#announceReceived) object will be returned.

| Field  | Type   | Required | Description             |
| ------ | ------ | -------- | ----------------------- |
| action | string | yes      | Fixed value, "announce" |

Expected response from server:

```json
{
  "action": "announceReceived",
  "data": {
    "shareId": "1AbhoECj"
  }
}
```

#### pulse

Pulse action meant to make maintaining the connection and the sharing session valid and possible by rapidly sending a packet to server to report the `Alive` state of the client. By doing this, server will store the session and shareId while the clients are alive, if a client stopped to send a packet, server will delete the related shareId and multiaddr it announced before. (**Default expiration time will be 5 minutes**)

If sending is successful, a [pulseReceived](#pulseReceived) object is returned.

| Field     | Type    | Required | Description                                                  |
| --------- | ------- | -------- | ------------------------------------------------------------ |
| action    | string  | yes      | Fixed value, "pulse"                                         |
| override  | boolean | optional | If set to `true`, the following data in this body object will be  updated to server, default is `false` |
| ip        | string  | optional | Based on override param, server will update the ip address according to the given one |
| port      | string  | optional | Based on override param, server will update the port mapping according to the given one |
| multiaddr | string  | optional | Based on override param, server will update the port mapping according to the given one |
| shareId   | boolean | optional | If set to `true`, server will regenerate a shareId to client |

Expected response from server:

```json
{
  "action": "pulseReceived",
  "data": {
    "status": "0",
    "overriden": "false"
  }
}
```

#### push

Push action tells the server the unique shareId of the target client the peers shared. Tracker server respond with the `IP:Port` back to the requesting client, so on, the client can try to establish the connection between. 

If sending is successful, a [pushReceived](#pushReceived) is returned.

| Field   | Type   | Required | Description                                                  |
| ------- | ------ | -------- | ------------------------------------------------------------ |
| action  | string | yes      | Fixed value, "push"                                          |
| shareId | string | yes      | The unique shareId from other client, this shareId can  be get through [announce](#announce) method or [pulse](#pulse) method with override set to `true` |

Expected response from server:

```json
{
  "action": "pushReceived",
  "data": {
    "peeraddr": "1.2.3.4:23333"
  }
}
```

### Types

#### announceReceived

| Field  | Type   | Required | Description                     |
| ------ | ------ | -------- | ------------------------------- |
| action | string | yes      | Fixed value, "announceReceived" |
| data   | dict   | yes      |                                 |

##### announceReceived Data

| Field   | Type   | Required | Description                      |
| ------- | ------ | -------- | -------------------------------- |
| shareId | string | yes      | shareId generated by server side |

#### pulseReceived

| Field  | Type   | Required | Description                  |
| ------ | ------ | -------- | ---------------------------- |
| action | string | yes      | Fixed value, "pulseReceived" |
| data   | dict   | yes      |                              |

##### pulseReceived Data

| Field     | Type   | Required | Description                                   |
| --------- | ------ | -------- | --------------------------------------------- |
| status    | string | yes      | 0 if pulse sent successful                    |
| overriden | string | Yes      | `true` if overriden data was successfully set |

#### pushReceived

| Field  | Type   | Required | Description                 |
| ------ | ------ | -------- | --------------------------- |
| action | string | yes      | Fixed value, "pushReceived" |
| data   | dict   | yes      |                             |

##### pushReceived Data

| Field    | Type   | Required | Description |
| -------- | ------ | -------- | ----------- |
| peeraddr | string | yes      | `IP:Port`   |
