# live-preview-frp

This project aims to provide a public/live URL of the application under development using [frp](https://github.com/fatedier/frp).

The workflow can facilitate simple front-end verification tests to more complex tests including API and back-end services.

To use it, you must have the project [Dockerized](https://docs.docker.com/) and if you have more than one service, add a [docker-compose](https://docs.docker.com/compose/) file with the communication rules between each service.

As a demo we created a simple [Dockerfile](./Dockerfile) with nginx and a [static html](./static/index.html) page.

To enable the publication of the live URL, the [frp](https://github.com/fatedier/frp) tool was used, which creates a tunnel between the containers running inside the GitHub runners. Therefore, it is necessary to host your own [frp](https://github.com/fatedier/frp) server or use our temporary server `live.arthurbdiniz.com`

## Architecture

![Architecture](images/architecture.png)

## frp server config

Here a simple server configuration:

```bash
# frps.ini
[common]
bind_port = 8000
vhost_http_port = 80
```

> Make sure you have `bind_port` and `vhost_http_port` exposed to the internet.

Download the latest frp version from [Release](https://github.com/fatedier/frp/releases/latest) page according to your operating system and architecture, then run:

```bash
frps -c ./frps.ini
```

> To use different configurations take a look at https://github.com/fatedier/frp#example-usage.


## DNS config

Using your domain, the only thing needed is to create a `CNAME` record pointing to your frp live server URL or to our own demo server `live.arthurbdiniz.com`.

### Record examples:

| Hostname                  | Type  | Target                |
|---------------------------|-------|-----------------------|
| *.test.example.com        | CNAME | live.arthurbdiniz.com |
| branch-1.test.example.com | CNAME | live.arthurbdiniz.com |


## frp client config

Inside your repo, create and commit a file called `frps.ini` with the following configurations.

```bash
# frps.ini
[common]
server_addr = live.arthurbdiniz.com
server_port = 8000

[web]
type = http
local_port = 80
```

In case you are the frp server host take a look at https://github.com/fatedier/frp#example-usage to use different configurations.

## Workflow

```yml
name: Live Preview FRP

on: pull_request

jobs:
  default:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Start services
        run: docker-compose up -d

      - name: Start tunnel
        uses: overhead-actions/live-preview-frp@main
        with:
          domain: ${{ github.head_ref }}.arthurbdiniz.com

      - name: Comment PR
        uses: unsplash/comment-on-pr@v1.3.0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          msg: 'Live preview URL: http://${{ github.head_ref }}.arthurbdiniz.com'
          check_for_duplicate_msg: false

      - name: Wait
        run: sleep 300
```

## Authors

 - Arthur Diniz: <arthurbdiniz@gmail.com>
 - Victor Moura: <victor_cmoura@hotmail.com>

## License

MIT Licensed. See [LICENSE](https://github.com/overhead-actions/live-preview-frp/blob/master/LICENSE) for full details.