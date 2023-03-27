# wasPrime.github.io

## Install node

Install `node` from <https://www.nodejs.org>

## Install Hexo

```bash
# Install Hexo
sudo npm install hexo-cli -g
# Install deploy extension
npm install hexo-deployer-git --save
```

## Run

```bash
git clone git@github.com:wasPrime/wasPrime.github.io.git

hexo init
hexo new "page_name" # hexo new
hexo clean
hexo g # hexo generate
hexo s # hexo server
hexo deploy -g # hexo deploy
```

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

Refer to <https://hexo.io/docs/front-matter> if adding tags and so on.
