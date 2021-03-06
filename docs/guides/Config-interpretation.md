# Baetyl Configuration Interpretation

Supported units:

- Size unit
  - b(byte)
  - k(kilobyte)
  - m(megabyte)
  - g(gigabyte)
- Time unit
  - s(second)
  - m(minute) 
  - h(hour)

Configuration examples can be found in the `example` directory of [baetyl project](https://github.com/baetyl/baetyl).

## Master Configuration

The Master configuration and application configuration are separated. The default configuration file is `etc/baetyl/conf.yml` in the working directory. The configuration is interpreted as follows:

```yaml
mode: The default value is `docker`, running mode of services. **docker** container mode or **native** process mode
grace: The default value is `30s`, the timeout for waiting services to gracefully exit.
server: API Server configuration of Master.
  address: The default value can be read from environment variable `BAETYL_MASTER_API_ADDRESS`, address of API Server.
  timeout: The default value is `30s`, timeout of API Server.
snfile: The serial number (SN) file for master to read as device fingerprint. If set, the content can be read from environment variable `BAETYL_HOST_SN`.
docker:
  api_version: The default value is `1.38`, the api version for client to call Docker Engine server.
logger: Logger configuration
  path: The default is `empty` (none configuration), that is, it does not write to the file. If the path is specified, it writes to the file.
  level: The default value is `info`, log level, support `debug`、`info`、`warn` and `error`.
  format: The default value is `text`, log print format, support `text` and `json`.
  age:
    max: The default value is `15`, means maximum number of days the log file is kept.
  size:
    max: The default value is `50`, log file size limit, default unit is `MB`.
  backup:
    max: The default value is `15`, the maximum number of log files to keep.
```

## Application Configuration

The default configuration file for the application configuration is `var/db/baetyl/application.yml` in the working directory. The configuration is interpreted as follows:

```yaml
version: Application version
services: Service list configuration
  - name: [MUST] Service name, must be unique in the service list
    image: [MUST] Service entry. In the docker container mode, which means the address of image. In the native process mode indicates where the service program package is located.
    replica: The default value is 0, the number of service copies, indicating the number of service instances started. Usually the service only needs to start one. The function runtime service is generally set to 0, not started by the Master, but is dynamically started by the function manager service.
    mounts: Storage volume mapping list
      - name: [MUST] The volume name, corresponding to one of the storage volume lists
        path: [MUST] The path mapped by the volume in the container
        readonly: The default value is false, whether the storage volume is read-only
    ports: Ports exposed in docker container mode, for example
      - 0.0.0.0:1883:1883
      - 0.0.0.0:1884:1884/tcp
      - 8080:8080/tcp
      - 9884:8884
    devices: Device mapping in docker container mode, for example
      - /dev/video0
      - /dev/sda:/dev/xvdc:r
    args: Service instance startup arguments, for example
      - '-c'
      - 'conf/conf.yml'
    env: Environment variables of service instance, for example
      version: v1
    restart: Service restart policy configuration
      retry:
        max: The default is `empty`(none configuration), which means always retry. If not, which means the maximum number of service restarts.
      policy: The default value is `always`, restart policy, support `no`, `always` and `on-failure`. And `no` means none restart, `always` means always restart, `on-failure` means restart the service if it exits abnormally.
      backoff:
        min: The default value is `1s`, minimum interval of restart.
        max: The default value is `5m`, maximum interval of restart.
        factor: The default value is `2`, factor of interval increase.
    resources: Service instance resource limit configuration in docker container mode
      cpu:
        cpus: The percentage of CPU available of the service instance, for example `1.5`, means that `1.5` CPU cores can be used.
        setcpus: The CPU core available for the service instance, for example `0-2`, means that `0` to `2` CPU cores can be used; `0` means that the 0th CPU core can be used; `1`, which means the 1st CPU core can be used.
      memory:
        limit: The available memory of the service, for example `500m`, means that 500 megabytes of memory can be used.
        swap: The swap space available to the service, for example `1g`, means that 1G of memory can be used.
      pids:
        limit: Number of processes the service can create.
volumes: Storage volume list
  - name: [MUST] The volume name, must be unique in the list of storage volumes
    path: [MUST] The path of the storage volume on the host, relative to the working directory of the Master
```

## Module Configuration


The default configuration file for the module configuration is `etc/baetyl/service.yml` in the working directory.

### baetyl-agent

```yaml
remote:
  mqtt: MQTT channel configuration
    clientid: [MUST] The Client ID, must be the id of cloud core device.
    address: [MUST] The endpoint address for client to connect with cloud management suit, must use ssl endpoint.
    username: [MUST] The client username, must be the username of cloud core device.
    ca: [MUST] The CA path for client to connect with cloud management suit.
    key: [MUST] The private key path for client to connect with cloud management suit.
    cert: [MUST] The public key path for client to connect with cloud management suit.
    timeout: The default value is `30s`, means timeout of the client connects to cloud.
    interval: The default value is `1m`, means maximum interval of client reconnection, doubled from 500 microseconds to maximum.
    keepalive: The default value is `10m`, means keep alive time between the client and cloud after connection has been established.
    cleansession: The default value is `false`, , means whether keep session in cloud after client disconnected.
    validatesubs: The default value is `false`, means whether the client checks the subscription result. If it is true, client exits and return errors when subscription is failure.
    buffersize: The default value is `10`, means the size of the memory queue sent by the client to the cloud management suit. If found exception, the client will exit and lose message.
  http: HTTPS channel configuration
    address: This address is automatically inferred based on the address of the MQTT channel. No configuration required
    timeout: The default value is `30s`, connection timeout period
  report: Agent report configuration.
    url: The report URL. No configuration required
    topic: The template of report topic. No configuration required
    interval: The default value is `20s`, interval of reporting.
  desire: Agent desire configuration.
    topic: The template of desire topic. No configuration required
```

### baetyl-hub

```yaml
listen: [MUST] Listening address, for example
  - tcp://0.0.0.0:1883
  - ssl://0.0.0.0:1884
  - ws://:8080/mqtt
  - wss://:8884/mqtt
certificate: SSL/TLS certificate authentication configuration, if `ssl` or `wss` is enabled, it must be configured.
  ca: Server CA certificate path
  key: Server private key path
  cert: Server public key path
principals: ACL configuration. If not configured, client cannot connect to this Hub, support username/password and certificate authentication.
  - username: Username for client non-ssl connection
    password: Password for client connection
    permissions:
      - action: Operation type of permission. `pub` means publish permission, `sub` means subscription permission.
        permit: List of topics allowed by the operation type, support `+` and `#` wildcards.
  - username: Username for client ssl connection
    permissions:
      - action: Operation type of permission. `pub` means publish permission, `sub` means subscription permission.
        permit: List of topics allowed by the operation type, support `+` and `#` wildcards.
subscriptions: Topic routing configuration
  - source:
      topic: subscribe topic
      qos: QoS of topic
    target:
      topic: publish topic
      qos: QoS of topic
message: MQTT message related configuration
  length:
    max: The default value is `32k`, which means maximum message length that can be allowed to be transmitted. The maximum can be set to 268,435,455 Byte(about 256MB).
  ingress: Message receive configuration
    qos0:
      buffer:
        size: The default value is `10000`, means the number of messages that can be cached in memory with QoS0. Increasing the cache can improve the performance of message reception. If the device loses power, it will directly discard the message with QoS0.
    qos1:
      buffer:
        size:  The default value is `100`, means the message cache size of waiting for persistent with QoS1. Increasing the cache can improve the performance of message reception, but the potential risk is that the service will exit abnormally(such as device power failure), it will lose the cached message, and will not reply(puback). The service exits normally and waits for the cached message to be processed without losing data.
      batch:
        max:  The default value is `50`, means the maximum number of messages with QoS1 can be insert into the database (persistence). After the message is persisted, it will reply with confirmation(ack).
      cleanup:
        retention:  The default value is `48h`, means the time that the message with QoS1 can be saved in the database. Messages that exceed this time will be physically deleted during cleanup.
        interval:  The default value is `1m`, means cleanup interval with QoS1.
  egress: Message publish configuration
    qos0:
      buffer:
        size:  The default value is `10000`, means the number of messages to be sent in the in-memory cache wit QoS0. If the device is powered off, the message will be discarded directly. After the buffer is full, the newly pushed message will be discarded directly.
    qos1:
      buffer:
        size:  The default value is `100`, means the size of the message buffer is not confirmed(ack) after the message with QoS1 is sent. After the buffer is full, the new message is no longer read, and the message in the cache is always acknowledged(ack). After the message with QoS1 is sent to the client, it waits for the client to confirm(puback). If the client does not reply within the specified time, the message will be resent until the client replies or the session is closed.
      batch:
        max:  The default value is `50`, means the maximum number of messages read from the database in batches.
      retry:
        interval:  The default value is `20s`, means the re-publish interval of message.
  offset: Message serial number persistence related configuration
    buffer:
      size:  The default value is `10000`, means the size of the cache queue for the serial number of the message that was acknowledged(ack). For example, three messages with QoS1 and serial numbers 1, 2, and 3 are sent to the client in batches. The client confirms the messages of sequence numbers 1 and 3. At this time, sequence number 1 will be queued and persisted. Although sequence number 3 has been confirmed, it still has to wait for the serial number 2 to be confirmed before entering the column. This design can ensure that the message can be recovered from the persistent serial number after the service restarts abnormally, ensuring that the message is not lost, but the message retransmission will occur, and therefore the message with QoS 2 is not supported.
    batch:
      max:  The default value is `100`, means the maximum number of batches of message serial numbers can be insert into the database.
logger: Logger configuration
  path: The default is `empty` (none configuration), that is, it does not write to the file. If the path is specified, it writes to the file.
  level: The default value is `info`, log level, support `debug`、`info`、`warn` and `error`.
  format: The default value is `text`, log print format, support `text` and `json`.
  age:
    max: The default value is `15`, means maximum number of days the log file is kept.
  size:
    max: The default value is `50`, log file size limit, default unit is `MB`.
  backup:
    max: The default value is `15`, the maximum number of log files to keep.
status: Service status configuration
  logging:
    enable: The default value is `false`, means whether to print baetyl status information.
    interval: The default value is `60s`, means interval of printing baetyl status information.
storage: Database storage configuration
  dir: The default value is `var/db/baetyl/data`, means database storage directory.
shutdown: Service exit configuration
  timeout: The default value is `10m`, means timeout of service exit.
```

### baetyl-function-manager Configuration

```yaml
hub:
  clientid: [MUST] The Client ID for the client to connect with the local Hub.
  address: [MUST] The endpoint address for the client to connect with the local Hub.
  username: The username for the client to connect with the local hub.
  password: The password for the client to connect with the local hub.
  ca: The CA path for the client to connect with the local hub.
  key: The private key path for the client to connect with the local hub.
  cert: The public key path for the client to connect with the local hub.
  timeout: The default value is `30s`, means timeout of the client connection with the local hub.
  interval: The default value is `1m`, means maximum interval of client reconnection, doubled from 500 microseconds to maximum.
  keepalive: The default value is `10m`, means keep alive time between the client and the local hub after connection has been established.
  cleansession: The default value is `false`, , means whether keep session in the local Hub after client disconnected.
  validatesubs: The default value is `false`, means whether the client checks the subscription result. If it is true, client exits and return errors when subscription is failure.
  buffersize: The default value is `10`, means the size of the memory queue sent by the client to the local Hub. If found exception, the client will exit and lose messages.
rules: Router rules configuration
  - clientid: [MUST] The Client ID for client to connect with the local Hub
    subscribe:
      topic: [MUST] The message topic subscribed from the local Hub.
      qos: The default value is `0`, the message QoS subscribed from the local Hub.
    function:
      name: [MUST] The name of the function that processes the message.
    publish:
      topic: [MUST] The message topic published to the local Hub.
      qos: The default value is `0`, means the message QoS published to the local Hub.
functions:
  - name: [MUST] The function name, must be unique in the function list.
    service: [MUST] The service name which provides the function runtime instance.
    instance: function instance configuration
      min: The default value is `0`, means the minimum number of function instance. And the minimum configuration allowed to be set is `0`, the maximum configuration allowed to be set is `100`.
      max: The default value is `1`, means the maximum number of function instance. And the minimum configuration allowed to be set is `1`, the maximum configuration allowed to be set is `100`.
      idletime: The default value is `10m`, maximum idle time of function instance.
      evicttime: The default value is `1m`, interval time between two evict operations.
      message:
        length:
          max: The default value is `4m`, means the maximum message length allowed for function instances to be received and publish.
    backoff:
      max: The default value is `1m`, the maximum reconnection interval of the client connection function instance
    timeout: The default value is `30s`, Client connection function instance timeout
```

## baetyl-function-python

```yaml
# the configurations of the two modules(python27 and python36) are the same, so we can follow this sample below
server: GRPC Server configuration; Do not configure if the instances of this service are managed by baetyl-function-manager
  address: GRPC Server address, <host>:<port>
  workers:
    max: The default value is the number of CPU core multiplied by 5, the maximum capacity of the thread pool
  concurrent:
    max: The default value is `empty`, means no limit, the maximum number of concurrent connections
  message:
    length:
      max: The default value is `4m`, the maximum message length allowed for function instances to receive and send
  ca: Server CA certificate path
  key: Server private key path
  cert: Server public key path
functions: function list
  - name: [MUST] The function name, must be unique in the function list.
    handler: [MUST] The function of Python code to handle message, for example, 'sayhi.handler'
    codedir: [MUST] The path of Python code
logger: Logger configuration
  path: The default is `empty` (none configuration), that is, it does not write to the file. If the path is specified, it writes to the file.
  level: The default value is `info`, log level, support `debug`、`info`、`warn` and `error`.
  age:
    max: The default value is `15`, means maximum number of days the log file is kept.
  backup:
    max: The default value is `15`, the maximum number of log files to keep.
```

## baetyl-function-node

```yaml
server: GRPC Server configuration; Do not configure if the instances of this service are managed by baetyl-function-manager
  address: GRPC Server address, <host>:<port>
  message:
    length:
      max: The default value is `4m`, the maximum message length allowed for function instances to receive and send
  ca: Server CA certificate path
  key: Server private key path
  cert: Server public key path
functions: function list
  - name: [MUST] The function name, must be unique in the function list.
    handler: [MUST] The function of Node code to handle message, for example, 'sayjs.handler'
    codedir: [MUST] The path of Node code
logger: Logger configuration
  path: The default is `empty` (none configuration), that is, it does not write to the file. If the path is specified, it writes to the file.
  level: The default value is `info`, log level, support `debug`、`info`、`warn` and `error`.
  backupCount:
    max: The default value is `15`, the maximum number of log files to keep.
```

## baetyl-video-infer

```yaml
hub:
  clientid: [MUST] The Client ID for the client to connect with the local Hub.
  address: [MUST] The endpoint address for the client to connect with the local Hub.
  username: The username for the client to connect with the local hub.
  password: The password for the client to connect with the local hub.
  ca: The CA path for the client to connect with the local hub.
  key: The private key path for the client to connect with the local hub.
  cert: The public key path for the client to connect with the local hub.
  timeout: The default value is `30s`, means timeout of the client connection with the local hub.
  interval: The default value is `1m`, means maximum interval of client reconnection, doubled from 500 microseconds to maximum.
  keepalive: The default value is `10m`, means keep alive time between the client and the local hub after connection has been established.
  cleansession: The default value is `false`, , means whether keep session in the local Hub after client disconnected.
  validatesubs: The default value is `false`, means whether the client checks the subscription result. If it is true, client exits and return errors when subscription is failure.
  buffersize: The default value is `10`, means the size of the memory queue sent by the client to the local Hub. If found exception, the client will exit and lose messages.
video:
  uri: [MUST] The video file path or camera address. 
    # For IP camera, the configuration just like `rtsp://<username>:<password>@<ip>:<port>/Streaming/channels/<stream_number>/`
      # `<username>` and `<password>` are the login authentication element
      # `<ip>` is the IP-address of camera
      # `<port>` is the port number of RTSP protocol, the default value is `554`
      # `<stream_number>` is the channel number, if it is equal to `1`, it indicates that the main stream is being captured; if it is equal to `2`, it indicates that the secondary stream is being captured
    # For USB camera, the configuration just like "0"(represents mapping device `/dev/video0` into container, also should be mounted on video infer service)
    # For video file, the configuration just like `var/db/baetyl/data/test.mp4`(mount the volume(store the video file) on video infer service)
  limit:
    fps: [MUST] The max number of video frames handled by inference per second. If the video fps is N, limit.fps is M, then Ceil(N/M) - 1 frames will be skipped.
infer:
  model: [MUST] The path of model file, more detailed contents please refer to https://docs.opencv.org/4.1.1/d6/d0f/group__dnn.html#ga3b34fe7a29494a6a4295c169a7d32422.
  config: [MUST] The path of model config file, more detailed contents please refer to https://docs.opencv.org/4.1.1/d6/d0f/group__dnn.html#ga3b34fe7a29494a6a4295c169a7d32422.
  backend: [Optional] The network backend which is used to improve inference efficiency. Now support `halide`, `openvino`, `opencv`, `vulkan` and `default`. More detailed contents please refer to https://docs.opencv.org/4.1.1/d6/d0f/group__dnn.html#ga186f7d9bfacac8b0ff2e26e2eab02625.
  device: [Optional] The target device of DNN processing. Now support `cpu`(default), `fp32`, `fp16`, `vpu`, `vulkan` and `fpga`. More detailed contents please refer to https://docs.opencv.org/4.1.1/d6/d0f/group__dnn.html#ga709af7692ba29788182cf573531b0ff5.
process: 
  before: creates 4-dimensional blob from image. Optionally resizes and crops image from center, subtract mean values, scales values by scalefactor, swap Blue and Red channels. More detailed contents please refer to https://docs.opencv.org/4.1.1/d6/d0f/group__dnn.html#ga29f34df9376379a603acd8df581ac8d7.
    scale: multiplier for image values.
    swaprb: flag which indicates that swap first and last channels in 3-channel image is necessary. 
    width: width of spatial size for output image.
    hight: hight of spatial size for output image.
    mean: scalar with mean values which are subtracted from channels. Values are intended to be in (mean-R, mean-G, mean-B) order if image has BGR ordering and swapRB is true.
      v1: blue component of type Scalar(Scalar is a 4-element(v1, v2, v3, v4) vector widely used in OpenCV to pass pixel values).
      v2: green component of type Scalar.
      v3: red component of type Scalar.
      v4: alpha component of type Scalar.
    crop: flag which indicates whether image will be cropped after resize or not.
  after:
    function: 
      name: [MUST] The name of the function that handle the inference result.
functions:
  - name: [MUST] The function name, must be unique in the function list.
    address: The function manager/server address, <host>:<port>, such as `function-manager:50051`
    message:
      length:
        max: The default value is `4m`, means the maximum message length allowed for function instances to be received and publish.
    backoff:
      max: The default value is `1m`, the maximum reconnection interval of the client connection function instance
    timeout: The default value is `30s`, Client connection function instance timeout
logger: Logger configuration
  path: The default is `empty` (none configuration), that is, it does not write to the file. If the path is specified, it writes to the file.
  level: The default value is `info`, log level, support `debug`、`info`、`warn` and `error`.
  format: The default value is `text`, log print format, support `text` and `json`.
  age:
    max: The default value is `15`, means maximum number of days the log file is kept.
  size:
    max: The default value is `50`, log file size limit, default unit is `MB`.
  backup:
    max: The default value is `15`, the maximum number of log files to keep.
```

### baetyl-remote-mqtt

```yaml
hub:
  clientid: [MUST] The Client ID for the client to connect with the local Hub.
  address: [MUST] The endpoint address for the client to connect with the local Hub.
  username: The username for the client to connect with the local hub.
  password: The password for the client to connect with the local hub.
  ca: The CA path for the client to connect with the local hub.
  key: The private key path for the client to connect with the local hub.
  cert: The public key path for the client to connect with the local hub.
  timeout: The default value is `30s`, means timeout of the client connection with the local hub.
  interval: The default value is `1m`, means maximum interval of client reconnection, doubled from 500 microseconds to maximum.
  keepalive: The default value is `10m`, means keep alive time between the client and the local hub after connection has been established.
  cleansession: The default value is `false`, , means whether keep session in the local Hub after client disconnected.
  validatesubs: The default value is `false`, means whether the client checks the subscription result. If it is true, client exits and return errors when subscription is failure.
  buffersize: The default value is `10`, means the size of the memory queue sent by the client to the local Hub. If found exception, the client will exit and lose messages.
rules: Message routing rules configuration
  - hub:
      clientid: The client ID for the client to connect with the local Hub.
      subscriptions: The topics subscribed by client from Hub, for example
        - topic: say
          qos: 1
        - topic: hi
          qos: 0
    remote:
      name: [MUST] The remote name, must be one of the remote list
      clientid: The client ID for the client to connect with remote Hub.
      subscriptions: The topics subscribed by client from remote Hub, for example
        - topic: remote/say
          qos: 0
        - topic: remote/hi
          qos: 0
remotes: The remote list
  - name: [MUST] The remote name, must be unique in this list.
    clientid: The client ID for the client to connect with the remote Hub.
    address: [MUST] The address for the client connect with the remote Hub.
    username: The username for the client connect with the remote Hub.
    password: The password for the client connect with the remote Hub.
    ca: The CA path for the client connect with the remote Hub.
    key: The private key path for the client connect with the remote Hub.
    cert: The public key path for the client connect with the remote Hub.
    timeout: The default value is `30s`, means timeout of the client connect to the remote Hub.
    interval: The default value is `1m`, means maximum interval of client reconnection, doubled from 500 microseconds to maximum.
    keepalive: The default value is `10m`, means keep alive time between the client and the local hub after connection has been established.
    cleansession: The default value is `false`, , means whether keep session in the local Hub after client disconnected.
    validatesubs: The default value is `false`, means whether the client checks the subscription result. If it is true, client exits and return errors when subscription is failure.
    buffersize: The default value is `10`, means the size of the memory queue sent by the client to the local Hub. If found exception, the client will exit and lose messages.
logger: Logger configuration
  path: The default is `empty` (none configuration), that is, it does not write to the file. If the path is specified, it writes to the file.
  level: The default value is `info`, log level, support `debug`、`info`、`warn` and `error`.
  format: The default value is `text`, log print format, support `text` and `json`.
  age:
    max: The default value is `15`, means maximum number of days the log file is kept.
  size:
    max: The default value is `50`, log file size limit, default unit is `MB`.
  backup:
    max: The default value is `15`, the maximum number of log files to keep.
```

### baetyl-timer

```yaml
hub: Hub configuration
  address: The address for the client to connect with the Hub.
  username: The username for the client to connect with the Hub.
  password: The password for the client to connect with the Hub.
  clientid: The client id for the client to connect with the Hub.
timer: timer configuration
  interval: Timing interval
publish:
  topic: The message topic published to the Hub.
  payload: 'The payload content uses golang template, currently supports two types of functions: Time and Rand. Time can be used to get the current date and time, Rand can be used to get random numbers. For example: if set payload with
  "{\"datetime\": {{.Time.Now}},\"timestamp\": {{.Time.NowUnix}},\"timestampNano\": {{.Time.NowUnixNano}},\"random1\": {{.Rand.Int}},\"random2\": {{.Rand.Int63}},\"random3\": {{.Rand.Intn 10}},\"random4\": {{.Rand.Float64}},\"random5\": {{.Rand.Float64n 60}},\"anyString\": \"inputString\"}",
  it will output: {"datetime": 2020-01-03 10:01:40.4381073 +0000 UTC m=+20.018560801,"timestamp": 1578045700,"timestampNano": 1578045700438177900,"random1": 1618091568073368693,"random2": 448858416199438315,"random3": 4,"random4": 0.04151395666829934,"random5": 29.64961133775892,"anyString": "inputString"}'
logger: Logger configuration
  path: The default is `empty` (none configuration), that is, it does not write to the file. If the path is specified, it writes to the file.
  level: The default value is `info`, log level, support `debug`、`info`、`warn` and `error`.
```
