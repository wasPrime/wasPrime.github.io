# wasPrime.github.io

## Install node

Install `node` from <https://nodejs.org>

## Install Hexo

```bash
# Install Hexo
sudo npm install hexo-cli -g
# Install deploy extension
npm install hexo-deployer-git --save
```

More info:

- [Hexo](https://hexo.io/)
- [Hexo documentation](https://hexo.io/docs/)
- [Hexo troubleshooting](https://hexo.io/docs/troubleshooting.html)
- [Hexo GitHub issues](https://github.com/hexojs/hexo/issues)

## Quick Start

### Clone

```bash
git clone git@github.com:wasPrime/wasPrime.github.io.git
```

### Init

```bash
hexo init
```

### Create a new page

```bash
hexo new "page_name" # hexo new
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Clean cache

```bash
hexo clean
```

### Generate static files

```bash
hexo g # hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Run local server

```bash
hexo s # hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Deploy to remote sites

``` bash
hexo deploy -g # hexo deploy
```

More info: [Deployment](https://hexo.io/docs/one-command-deployment.html)

## Theme

### Docs

[Fluid Theme Docs](https://hexo.fluid-dev.com/docs/start)

### Install/Update

```bash
# Install theme fluid
npm install --save hexo-theme-fluid
# Update theme
npm update --save hexo-theme-fluid
```

### Custom Configuration

Adjust to custom setting by changing `_config.fluid.yml`.

## Pages Configuration

Refer to <https://hexo.io/docs/front-matter> if any property need to be added or updated such as `categories` `tags`.
