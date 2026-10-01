# halloy-irc

My setup for [Halloy](https://halloy.chat), a free, open-source IRC client, after moving over from LimeChat. It includes a sample config, two Solarized themes with readable IRC colors, and tips for connecting through a ZNC bouncer.

Tested with Halloy 2026.8 on macOS.

## What's here

- `config.example.toml`: my config with the server details and highlight words swapped for placeholders.
- `themes/solarized-dark.toml` and `themes/solarized-light.toml`: [Solarized](https://ethanschoonover.com/solarized/) themes that fix IRC colors that are hard to read on each background.

## Install

1. Install Halloy: `brew install --cask halloy`.
2. Open the config folder. On macOS it's `~/Library/Application Support/halloy/`.
3. Copy `themes/*.toml` into its `themes/` folder.
4. Copy `config.example.toml` to `config.toml` and fill in your server, nickname, and password command.
5. Reload the config from the sidebar menu or the command bar. Font changes need a restart.

## Tips

### Keep your password out of the config file

`password_command` runs a shell command and uses its output as the password. With the [1Password CLI](https://developer.1password.com/docs/cli/):

```toml
password_command = "op item get 'ZNC' --fields password --reveal"
```

Halloy can also read the [system keychain](https://halloy.chat/guides/keyring) or a [password file](https://halloy.chat/guides/password-file).

### Make bot colors readable

Bots color their messages with the 16 mIRC colors. Several of the defaults are nearly invisible on a Solarized background. A theme's `[formatting]` table [overrides them](https://halloy.chat/guides/custom-themes).

The themes here override the colors below 3:1 contrast, using values from my LimeChat theme, [Colloquial-Lance](https://github.com/lancewillett/colloquial-lance). The rest stay as comments, ready to turn on.

They also tone down the bright white-on-green deploy lines. `green` becomes a muted olive and `white` matches the page, so those lines read as a quiet chip while green text stays readable.

Two things to know:

- Halloy uses one value per color for both text and background. A color has to read on the page and behind other text.
- Leave `black` alone. Status bots often put yellow or red text on a black background, and overriding black breaks those labels.

To see which colors your channels actually use, count them in Halloy's history files:

```sh
cd ~/Library/Application\ Support/halloy/history
gzcat *.json.gz | grep -oE '"(fg|bg)":"[A-Za-z]+"' | sort | uniq -c | sort -rn
```

### See your text selection

In my starting Solarized theme, `selection` matched the background, so selected text was invisible. These themes set it to the next shade: `#073642` (dark) and `#EEE8D5` (light).

### Pick a font Halloy can load

Use a monospaced font that is not variable-weight; Halloy [can't load variable fonts](https://halloy.chat/configuration/font). Menlo ships with macOS and works. Restart Halloy after changing fonts.

### Hide old connect and disconnect lines

Bouncer reconnects leave a trail of status lines. This hides any older than five minutes ([reduce noise guide](https://halloy.chat/guides/reduce-noise)):

```toml
[buffer.internal_messages]
success.smart = 300
error.smart = 300
```

The same guide covers join, part, and quit filters.

### Show link previews only where you want them

```toml
[preview.card]
exclude = "*"
include = { channels = ["#your-channel"] }
```

### Stop ZNC from kicking you off

If you see `You are being disconnected because another user just authenticated as you`, two clients share one ZNC login. With ZNC's `MultiClients` setting off, each new login knocks the others off, and two auto-reconnecting clients loop forever.

Fix it one of two ways:

- Quit your old client.
- Turn on **Allow multiple clients** in ZNC's web settings, or run `/msg *controlpanel Set MultiClients $me true`.

The ZNC [`clientnotify`](https://wiki.znc.in/Clientnotify) module tells you when another client connects. Halloy skips duplicates when ZNC replays missed messages.

### Know the highlight quirk

As of 2026.8, a `[[highlights.match]]` word that is also the nickname of someone in the channel doesn't trigger. Halloy turns nicknames into user links before it checks highlight words, and it only checks plain text ([source](https://github.com/squidowl/halloy/blob/2026.8/data/src/message.rs#L1935-L1947)). The word does trigger once that person leaves.

## Credits

- [Halloy](https://github.com/squidowl/halloy) by squidowl.
- [Solarized](https://ethanschoonover.com/solarized/) by Ethan Schoonover.
- Color values from [Colloquial-Lance](https://github.com/lancewillett/colloquial-lance), based on [Colloquial](http://julianstahnke.com/read/a_theme_for_limechat_colloquial/) by Julian Stahnke.
