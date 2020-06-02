## Installing Coturn

To enable voice & video call, you need to install Turn Server.

1. Install coturn using apt-get
```
sudo apt-get -y update
sudo apt-get -y install coturn
```
2. Enable coturn by editing `/etc/default/coturn`, remove “#” comment on `TURNSERVER_ENABLED` to enable turnserver.

3. Create your coturn config in `/etc/turnserver.conf`, Some config line in the example is commented out to disable SSL, you can enable it by removing the “#” and editing it to your own need.

4. Run the turnserver from command line
```
turnserver
```

```
sudo turnserver -a -o -v -n  --no-dtls --no-tls -u test:test -r "someRealm"
```


## Firewall Rules

To let coturn work, you need to open these port:
* 80 TCP
* 443 TCP
* 3478 UDP
* 10000–20000 UDP

In some cases, some mobile provider can’t be used to access port 3478 or port 5349, try to replace the port in config with 8000 to check if it solve the problem.


## Trecho Docker Compose

```
 signal-turn:
    build: ./turn
    container_name: signal-turn
    restart: always
    environment:
      - TURN_SECRET=${TURN_SECRET}
      - TURN_REALM=${TURN_REALM}
      - EXTERNAL_IP=${EXTERNAL_IP}
      - TURN_LOW=${TURN_LOW}
      - TURN_HIGH=${TURN_HIGH}
    ports:
      - 3478-3479:3478-3479
      - 3478-3479:3478-3479/udp
      - ${TURN_LOW}-${TURN_HIGH}:${TURN_LOW}-${TURN_HIGH}/udp
```      
### File: .env


Sample **.env** file:
```
POSTGRES_USER=signal
POSTGRES_PASSWORD=thepassword
MINIO_ACCESS_KEY=AKIAIG4ILCORMAJCS37A
MINIO_SECRET_KEY=u8cQx07PvHJS8/zvr7q3IFY+w2toIYIJQ7vm1ETH
HOST=127.0.0.1
EXTERNAL_IP=0.0.0.0
TURN_REALM=turn.domain.ru
TURN_SECRET=test
TURN_LOW=49152
TURN_HIGH=49252
```

### Dockerfile

```
FROM ubuntu:18.04

RUN apt-get update
RUN apt-get install -y build-essential git emacs-nox libssl-dev sqlite3 libsqlite3-dev libevent-dev g++ libboost-dev
RUN git clone https://github.com/coturn/coturn.git
RUN cd coturn && ./configure && make && make install
ENV EXTERNAL_IP 0.0.0.0
ENV TURN_REALM turn
ENV TURN_SECRET yoursecret
ENV TURN_LOW 49152
ENV TURN_HIGH 49252
CMD ["/bin/bash", "-c", "turnserver -a --prod -v -n -f --no-dtls --no-tls -r \"${TURN_REALM}\" --mobility --static-auth-secret ${TURN_SECRET} -X ${EXTERNAL_IP} --min-port ${TURN_LOW} --max-port ${TURN_HIGH}"]
```
