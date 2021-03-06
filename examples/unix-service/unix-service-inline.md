
<!-- Inspired from https://medium.com/@benmorel/creating-a-linux-service-with-systemd-611b5c8b91d6 -->

## The program

Let’s create a small server using PHP. I can see your eyebrows rising, but it works surprisingly well. We’ll listen to UDP port 10000, and return any message received with a ROT13 transformation:

```php|content=server.php
<?php
$sock = socket_create(AF_INET, SOCK_DGRAM, SOL_UDP);
socket_bind($sock, '0.0.0.0', 10000);
for (;;) {
    socket_recvfrom($sock, $message, 1024, 0, $ip, $port);
    $reply = str_rot13($message);
    socket_sendto($sock, $reply, strlen($reply), 0, $ip, $port);
}
```

Let’s start it:

```bash|serve
$ php server.php
```

And test it in another terminal:

```bash|expect=1
client$ echo 'Hello, world!' | nc -w 1 -u 127.0.0.1 10000
Uryyb, jbeyq!
```

Cool, it works. Now we want this script to run at all times, be restarted in case of a failure (unexpected exit), and even survive server restarts. That’s where systemd comes into play.

Turning it into a service
Let’s create a file called `/etc/systemd/system/rot13.service`:

```ini|content=/etc/systemd/system/rot13.service
[Unit]
Description=ROT13 demo service
After=network.target
StartLimitIntervalSec=0
[Service]
Type=simple
Restart=always
RestartSec=1
User=centos
ExecStart=/usr/bin/env php /path/to/server.php

[Install]
WantedBy=multi-user.target
```

You’ll need to:

* set your actual username after `User=`
* set the proper path to your script in `ExecStart=`

That’s it. We can now start the service:

```bash|run
$ systemctl start rot13
```

And automatically get it to start on boot:

```bash|run
$ systemctl enable rot13
```