# Google Cloud Shell

Google Cloud Shell Editor is based on the codeoss project (the base of vscode).

## Which region is my cloud shell running in?

```bash
# from: https://stackoverflow.com/a/51201608
curl -H "Metadata-Flavor: Google" metadata/computeMetadata/v1/instance/zone
# from: https://cloud.google.com/shell/docs/how-cloud-shell-works#zone_selection
curl metadata/computeMetadata/v1/instance/zone
```
## Understanding where my persisted cloud shell data is stored
```bash
curl metadata/computeMetadata/v1/instance/disks/
# disk 0 should be 'boot', disk 1 should be 'home'

```

## Customize .bashrc

This reflects customizations to adjust the editor and terminal that are part of the editor.  This is not the complete .bashrc with everything custom in it. See [[gcp-my_bashrc]] for my full bashrc customizations.

```bash
FONT_NAME="FiraMono"
FONT_URL="https://github.com/ryanoasis/nerd-fonts/releases/download/v3.2.1/FiraMono.zip"
FONT_DIR="$HOME/.local/share/fonts"
if ! fc-list : family | grep -q "$FONT_NAME"; then
    echo "$FONT_NAME font is not installed. Downloading and installing..."
  
    if [ ! -d "$FONT_DIR" ]; then
        mkdir -p "$FONT_DIR"
    fi
  
    wget -O "$FONT_DIR/FiraMono.zip" "$FONT_URL"
    unzip "$FONT_DIR/FiraMono.zip" -d "$FONT_DIR"
    rm "$FONT_DIR/FiraMono.zip"
  
    # Refresh the font cache
    fc-cache -fv
  
    echo "$FONT_NAME font installed successfully."
fi

OHMYPOSH_BIN=${LOCAL_BIN_DIR}/oh-my-posh
if [ ! -f ${OHMYPOSH_BIN} ]; then
    # install oh-my-posh
    echo "oh-my-posh binary not found.  Installing now"
    curl -s https://ohmyposh.dev/install.sh | bash -s -- -d ${LOCAL_BIN_DIR}
fi


##### ADD THIS TO THE END OF BASHRC TO INIT THE PROMPT #####
# install oh-my-posh and add eval line to .bashrc
# curl -s https://ohmyposh.dev/install.sh | bash -s -- -d ~/.local/bin
# export current theme to customize it:
# oh-my-posh config export --output ~/.mytheme.omp.yaml
# insert the below block in the theme between the session and path segments
#   - type: gcp
#     style: powerline
#     powerline_symbol: 
#     foreground: "#ffffff"
#     background: "#47888d"
#     template: "  {{.Project}} "

if [ -f ${OHMYPOSH_BIN} ]; then
    eval "$(oh-my-posh init bash --config ~/.mytheme.omp.yaml)"
    # now exec bash to reload
fi
```

## Extensions

The following extensions are what I use at the moment. I am listing them here because they may be referenced in the `settings.json` section after this.

* [ESLint](https://open-vsx.org/extension/dbaeumer/vscode-eslint)
* [Javascript and TypeScript Nightly](https://open-vsx.org/extension/ms-vscode/vscode-typescript-next)
* [Linter](https://open-vsx.org/extension/fnando/linter)
* [Markdown Editor](https://open-vsx.org/extension/zaaack/markdown-editor)
* [OpenSumi Default Themes](https://open-vsx.org/extension/opensumi/opensumi-default-themes)
  * (I use the dark theme)
* [Prettier - Code Formatter](https://open-vsx.org/extension/esbenp/prettier-vscode)
* [Pretty TypeScript Errors](https://open-vsx.org/extension/yoavbls/pretty-ts-errors)

## Customize settings.json

Because of how GCP is based on codeoss, the standard `~/.vscode/settings.json` path doesn't seem to work. Using `ctrl + ,` in the IDE will open the settings file in UI mode.  You can then switch to the actual settings file by clicking the `Open Settings (JSON)` icon:

![](assets/20241029_080043_image.png)

The `settings.json` file I am using is as follows:

```json
{
    "http.proxySupport": "off",
    "workbench.startupEditor": "welcomePageInEmptyWorkbench",
    "redhat.telemetry.enabled": false,
    "window.commandCenter": false,
    "terminal.integrated.defaultProfile.linux": "Google Cloud Shell",
    "workbench.layoutControl.enabled": false,
    "workbench.colorTheme": "opensumi-dark",
    "[typescript]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "git.confirmSync": false,
    "git.autofetch": true,
    "editor.fontFamily": "FiraMono Nerd Font, Consolas, 'Courier New', monospace",
    "editor.fontSize": 14,
    "editor.fontLigatures": true,  // Enable font ligatures if your font supports them
    "terminal.integrated.fontFamily": "FiraMono Nerd Font, Consolas, 'Courier New', monospace",
    "terminal.integrated.fontSize": 14,
    "terminal.integrated.localEchoStyle": "dim",
    "cloudcode.project": "GCP_CLOUD_PROJECT_HERE",
    "googlecloudcode.gemini.enabled": false
    // KY2
}
```

For those that are curious, the `// KY2` comment is deliberate for me to find out where the settings are actually being saved when the editor is active.  I don't understand everythign about why it is doing things the way it does, but it appears to be random caching being generated.  We can find the files where `// KY2` is being saved to using the following:

```bash
> grep -r '// KY2' ~/.codeoss/data/User/*
/home/USERNAME/.codeoss/data/User/History/-422a496d/7752.json:    // KY2
```

I have tried a grep against `~/` and I only get results from the `~/.codeoss/data/User/History` directory.  I am unsure of how it stores the permanent referenced version.  Is it all just based on cached versions?
