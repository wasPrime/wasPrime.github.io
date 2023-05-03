---
title: Configure iTerm2 and Zsh
date: 2023-04-09 22:00:00
categories:
- [configuration, iterm2]
- [configuration, zsh]
tags:
- configuration
- iterm2
- zsh
---

{% note success %}
To configure the environments as below:

- `Oh My Zsh`
- `Powerline` font
- `agnoster` theme
- Zsh extensions
  - `zsh-syntax-highlighting`
  - `zsh-autosuggestions`
{% endnote %}

## Install `Oh My Zsh`

```bash
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
# Or
sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```

## Set Theme

Open `~/.zhsrc`, set `ZSH_THEME` as `agnoster`.

![theme](/resources/Configure-iTerm2-zsh/img/theme.jpeg)

{% note info %}

- **Official Website:** <https://ohmyz.sh/>
- **Github:** <https://github.com/ohmyzsh/ohmyzsh>

{% endnote %}

## Set iTerm2 Colors

Open iTerm2 and open its `Preferences...` or press `⌘` + `,`, follow the path `Profiles > Open Profiles > Edit Profiles > Colors` and select a color.

![Colors](/resources/Configure-iTerm2-zsh/img/iTerm2_colors.jpeg)

## Install & Set `Powerline` font

### Install

```bash
git clone https://github.com/powerline/fonts.git --depth=1
cd fonts
./install.sh
```

### Set

From `Profiles > Open Profiles > Edit Profiles > Text` of iTerm2's preferences, set font as `Powerline`.

![iTerm2 font](/resources/Configure-iTerm2-zsh/img/iTerm2_font.jpeg)

## Set Transparency & Background

From `Profiles > Open Profiles > Edit Profiles > Window` of iTerm2's preferences, set transparency and background.

![iTerm2 transparency & background](/resources/Configure-iTerm2-zsh/img/iTerm2_background.jpeg)

## Install Zsh Extensions

1. Install from official website.

    ```bash
    git clone https://github.com/zsh-users/zsh-syntax-highlighting.git $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
    git clone https://github.com/zsh-users/zsh-autosuggestions.git $ZSH_CUSTOM/plugins/zsh-autosuggestions
    ```

    > `$ZSH_CUSTOM` is `~/.oh-my-zsh/custom`.

2. Open `~/.zshrc`, find `plugins` and change it.

    ```vim
    # Which plugins would you like to load?
    # Standard plugins can be found in $ZSH/plugins/
    # Custom plugins may be added to $ZSH_CUSTOM/plugins/
    # Example format: plugins=(rails git textmate ruby lighthouse)
    # Add wisely, as too many plugins slow down shell startup.
    plugins=(git zsh-syntax-highlighting zsh-autosuggestions)
    ```

3. Activate

    ```bash
    source ~/.zshrc
    ```

## References

- <https://mp.weixin.qq.com/s/3Uh3CwZwxko8tiB1yEYj7Q>
