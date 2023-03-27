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
hexo n "page_name" # hexo new
hexo clean
hexo g # hexo generate
hexo s # hexo server
hexo deploy -g # hexo deploy
```
