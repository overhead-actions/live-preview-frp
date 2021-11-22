# live-preview-frp

This project aims to provide a public/live URL of the application under development using [ngrok](https://ngrok.com/).

The workflow can facilitate simple front-end verification tests to more complex tests including API and back-end services.

To use it, you must have the project [Dockerized](https://docs.docker.com/) and if you have more than one service, add a [docker-compose](https://docs.docker.com/compose/) file with the communication rules between each service.

As a demo we created a simple [Dockerfile](./Dockerfile) with nginx with a [static html](./static/index.html) page.

To enable the publication of the live URL, the [ngrok](https://ngrok.com/) tool was used, which creates a tunnel between the containers running inside the GitHub runners. Therefore, it is necessary to [login](https://dashboard.ngrok.com/signup) to the ngrok website and generate an auth token for it's use, adding as secret into GitHub repository.

## Secrets

- NGROK_AUTH_TOKEN

## Authors

 - Arthur Diniz: <arthurbdiniz@gmail.com>
 - Victor Moura: <victor_cmoura@hotmail.com>


## Workflow

```yml
name: Live Preview

on: pull_request

jobs:
  live-preview-frp:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Start services
        run: docker-compose up -d

      - name: Start tunnel
        uses: overhead-actions/live-preview-frp@main
        with:
          protocol: http
          port: 4000
          ngrok_auth_token: ${{ secrets.NGROK_AUTH_TOKEN }}

      - name: Get URL
        id: vars
        run: echo "::set-output name=url::$(curl -s localhost:4040/api/tunnels | jq -r .tunnels[0].public_url)"

      - name: Comment PR
        uses: unsplash/comment-on-pr@v1.3.0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          msg: 'Live preview URL: ${{ steps.vars.outputs.url }}'
          check_for_duplicate_msg: false

      - name: Wait
        run: sleep 300
```

## frps.ini

```bash
[common]
server_addr = live.arthurbdiniz.com
server_port = 8000

[web]
type = http
local_port = 80
custom_domains = nginx.arthurbdiniz.com
```


## License

MIT Licensed. See [LICENSE](https://github.com/overhead-actions/live-preview-frp/blob/master/LICENSE) for full details.