# systemd for Node.js

  Adds support for running node.js as a socket-activated service under systemd.

  *Note:* **This is a fork of [`rubenv/node-systemd`](https://github.com/rubenv/node-systemd)**.

  More info on the how and why: https://rocketeer.be/articles/deploying-node-js-with-systemd/

  For more background on socket activation: http://0pointer.de/blog/projects/socket-activation.html

  Obviously, this will only work on Linux distributions with systemd (such as Fedora).

## Usage

  Consider a simple HTTP server:

```javascript
var http = require('http');

const server = http.createServer(function (req, res) {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('Hello World\n');
});
```

  We want our server to work both stand-alone (`node server.js`) and as a socket-activated systemd service.

```sh
$ npm install @derhuerst/systemd
```

  If the service is started via systemd socket activation, `getListenArgs()` will return an array of arguments to be passed into `server.listen()`. Otherwise, it will return `null` and we'll listen to the port as usual.

```javascript
const {getListenArgs} = require('@derhuerst/systemd');

const port = 1000;
const listenArgs = getListenArgs() || [port];
server.listen(...listenArgs);
```

  Install a systemd socket file (e.g.: /etc/systemd/system/node-hello.socket):

```ini
[Socket]
ListenStream=1337

[Install]
WantedBy=sockets.target
```

  Install a systemd service file (e.g.: /etc/systemd/system/node-hello.service):

```ini
# Adjust according to man 5 systemd.exec

[Service]
ExecStart=/path/to/bin/node /path/to/hello.js
StandardOutput=syslog
User=nobody
Group=nobody
```

  Be sure to substitute the paths to node and your script!

  * &#x26A0; __Run node directly__ or make sure your startup helper scripts
    can hand over the sockets. `npm start` [probably won't work][issue-11].

  [issue-11]: https://github.com/rubenv/node-systemd/issues/11

  Reload the systemd daemon so that it picks up the new unit files:

```sh
$ systemctl --system daemon-reload
```

  Enable and start the socket:

```sh
$ systemctl enable node-hello.socket
$ systemctl start node-hello.socket
```

  Check the status of the socket:

```sh
$ systemctl status node-hello.socket
node-hello.socket
      Loaded: loaded (/etc/systemd/system/node-hello.socket)
      Active: active (listening) since Sat, 15 Oct 2011 20:27:47 +0200; 2s ago
      CGroup: name=systemd:/system/node-hello.socket
```

  Great, it's running!

  Check the status of the service, not running yet:

```sh
$ systemctl status node-hello.service
node-hello.service
      Loaded: loaded (/etc/systemd/system/node-hello.service)
      Active: inactive (dead)
      CGroup: name=systemd:/system/node-hello.service
```

  Do a request to your service:

```sh
$ curl -i http://localhost:1337/
HTTP/1.1 200 OK
Content-Type: text/plain
Connection: keep-alive
Transfer-Encoding: chunked

Hello World
```

  Check again, now it will be running:

```sh
$ systemctl status node-hello.service
node-hello.service
      Loaded: loaded (/etc/systemd/system/node-hello.service)
      Active: active (running) since Sat, 15 Oct 2011 20:32:10 +0200; 38s ago
    Main PID: 1159 (node)
      CGroup: name=systemd:/system/node-hello.service
              └ 1159 /path/to/bin/node /path/to/hello.js
```


## Contributing

  Before submitting a pull request, please check your code.

  Install the dev dependencies:

```sh
$ npm install --dev
```

  Run [ESLint](https://eslint.org):

```sh
$ npm run lint
```


## License

    (The MIT License)

    Copyright (C) 2011-2014 by Ruben Vermeersch <ruben@rocketeer.be>

    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in
    all copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
    THE SOFTWARE.

