## wrapping docker container in systemd

Next, let's consider the creation of a service file on the example of the Nillion node where:
- image in Docker Hub: `nillion/retailtoken-accuser:v1.0.1`
- start command: `docker run -v ./nillion/accuser:/var/tmp nillion/retailtoken-accuser:v1.0.1 accuse --rpc-endpoint "https://testnet-nillion-rpc.lavenderfive.com" --block-start 5624116`

>NOTE
>
>We do not consider the initialisation procedure, only the start/stop/restart of the container

Create a unit configuration file:

```bash
sudo touch /etc/systemd/system/nillion-docker.service
```

Open the `nillion-docker.service` file in text editor and put the content:

```bash
[Unit]
Description=Nillion Service
After=docker.service
Requires=docker.service

[Service]
Restart=always
RestartSec=5
TimeoutStartSec=0
ExecStartPre=-/usr/bin/docker kill nillion
ExecStartPre=-/usr/bin/docker rm nillion
ExecStartPre=/usr/bin/docker pull nillion/retailtoken-accuser:v1.0.1
ExecStart=/usr/bin/docker run -v ./nillion/accuser:/var/tmp nillion/retailtoken-accuser:v1.0.1 accuse --rpc-endpoint "https://testnet-nillion-rpc.lavenderfive.com" --block-start 5624116 --name nillion
ExecStop=/usr/bin/docker stop nillion
StandardOutput=append:/var/log/nillion/output.log
StandardError=append:/var/log/nillion/diagnostic.log

[Install]
WantedBy=multi-user.target
```

Change the file permissions by running the command:

`sudo chmod 644 /etc/systemd/system/nillion-docker.service`

Next, reload the Systemctl Daemon:

`sudo systemctl daemon-reload`

Enable the service on startup when the system boots:

`sudo systemctl enable nillion-docker.service`
