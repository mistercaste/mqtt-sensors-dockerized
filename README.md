# mqtt-sensors-dockerized
A fully customisable python application for DHT22/DHT11 and BMP085 sensors, running in a docker container to send temperature and humidty sensor values in a JSON format to a MQTT broker.
The container can be found on [Docker Hub](https://hub.docker.com/r/mistercaste/mqtt-sensors-dockerized).

## Initial setup
In terms of hardware, you'll need a DHT11 or DHT22 temperature/humidity sensor, as well as a BMP085 pressure sensor connected to a Raspberry Pi. Examples can be found online depending on your specific models, one I would suggest would be [this one](https://www.instructables.com/Raspberry-Pi-Tutorial-How-to-Use-the-DHT-22/).

Additionally, you'll need to have [docker installed in your Raspberry Pi](https://phoenixnap.com/kb/docker-on-raspberry-pi).

If you already have everything setup, continue to the next section.

## Wiring

![IMG_20231024_190511](https://github.com/mistercaste/mqtt-sensors-dockerized/assets/2144343/3d187917-b785-4af6-a2e5-3596b365190f)
![IMG_20231024_190857](https://github.com/mistercaste/mqtt-sensors-dockerized/assets/2144343/7a9f75aa-9562-406a-8254-5aa3dd5f2693)

## Running docker container

Pre-requisite is to enable the I2C port on the Raspberry PI

```
sudo raspi-config nonint do_i2c 0
```

This can be performed on GUI as well, by entering `raspi-config` &rarr; `Interface Options` &rarr; `I2C`

### As a standalone container
See the available configurations for the command in the `Available variables` section below and run the command:

```
docker run \
    --name dht-mqtt-dockerized \
    --restart=unless-stopped \
    --net=host \
    --privileged \
    -e SENSOR_PIN=14 \
    -e SENSOR_TYPE='dht22' \
    -e SENSOR_CHECK_INTERVAL=60 \
    -e SENSOR_DECIMAL_DIGITS=4 \
    -e MQTT_HOSTNAME=localhost \
    -e MQTT_PORT=1883 \
    -e MQTT_TIMEOUT=60 \
    -e MQTT_TOPIC='sensor/value'
    -e MQTT_CLIENT_ID='dht-mqtt' \
    -e CLIENT_CLEAN_SESSION='False' \
    -e CLIENT_TLS_INSECURE='False' \
    -e MQTT_CLIENT_QOS='4'
    -e MQTT_USERNAME=''
    -e MQTT_PASSWORD=''
    -e LOG_LEVEL='info'
    tomasgrigaliunas/dht-mqtt-dockerized
```

### Available variables

- `SENSOR_PIN` - an integer for the Raspberry Pi pin to which the DHT Data pin is connected to (Default: `4`, refers to `GPIO4`).
- `SENSOR_TYPE` - refers to the DHT sensor type, available options are `DHT22` or `DHT11` (Default `DHT22`).
- `SENSOR_CHECK_INTERVAL` - the frequency in seconds on how often the temperature/humidity is checked by the sensor and sent to the MQTT broker (Default: `30`).
- `SENSOR_DECIMAL_DIGITS` - rounds the temperature/humidity values to the specified number (integer) of decimals (Default `2`).
- `MQTT_HOSTNAME` - the hostname or IP address of the MQTT broker to which the client should connect to, can be left empty if the broker is running on the same raspberry pi host (Default `localhost`).
- `MQTT_PORT` - the network port of the MQTT broker (Default `1883`)
- `MQTT_TIMEOUT` - maximum period in seconds allowed between communications with the broker (Default `60`)
- `MQTT_TOPIC` - the topic to which the temperature/humidity values in JSON should be published to (Default `sensor/value`).
- `MQTT_CLIENT_ID` - the `CLIENT_ID` of the MQTT Client (Default `dht-sensor-mqtt`).
- `CLIENT_CLEAN_SESSION` - defines whether the session is only stored in memory not persisted (Default `False`).
- `CLIENT_TLS_INSECURE` - defines verification of the server hostname in the server certificate.(Default `False`).
- `MQTT_CLIENT_QOS` - the quality of service level to use to publish with (Default `0`).
- `MQTT_USERNAME` - Set a username for broker authentication (Default ``).
- `MQTT_PASSWORD` - Set a password for broker authentication (Default ``).
- `LOG_LEVEL` - Sets the verbosity of the logs publishd to the console, available options are `debug`, `info`, `warn`, `error` (Default `info`).

### Sample
Sample execution running against a Mosquitto broker on `nas.home`, topic `sensor/value`, every 2 seconds (`SENSOR_CHECK_INTERVAL`). Your sensor must be reporting its data to the PIN 14 (check your Raspberry PI docs)

```
docker run --rm --name dht-mqtt-dockerized --net=host --privileged -e SENSOR_PIN=14 -e SENSOR_TYPE='dht11' -e SENSOR_CHECK_INTERVAL=2 -e SENSOR_DECIMAL_DIGITS=4 -e MQTT_HOSTNAME=nas.home -e MQTT_PORT=1883 -e MQTT_TIMEOUT=10 -e MQTT_TOPIC='sensor/value' -e MQTT_CLIENT_ID='dht-mqtt' -e CLIENT_CLEAN_SESSION='False' -e CLIENT_TLS_INSECURE='False' -e MQTT_CLIENT_QOS='4' -e MQTT_USERNAME=<USERNAME> -e MQTT_PASSWORD=<PASSWORD> -e LOG_LEVEL='info' mistercaste/mqtt-sensors-dockerized:latest
```

## License
Eclipse Public License 2.0
