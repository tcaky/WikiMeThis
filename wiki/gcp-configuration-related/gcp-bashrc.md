# My GCP .bashrc
This is added at the end of the default .bashrc that gcp creates
```bash
###### MY CUSTOM .bashrc UPDATES BELOW HERE ######

SECRETS_PROJECT=PROJECT_WHERE_SECRETS_MANAGER_IS_BEING_ACCESSED
export PHAC_ROOT_FOLDER_ID=$(gcloud secrets versions access latest --secret=PHAC_ROOT_FOLDER_ID --project ${SECRETS_PROJECT})
export PHAC_BILLING_ID=$(gcloud secrets versions access latest --secret=PHAC_BILLING_ID --project ${SECRETS_PROJECT})
export PHAC_EXPERIMENT_FOLDER_ID=$(gcloud secrets versions access latest --secret=PHAC_EXPERIMENT_FOLDER_ID --project ${SECRETS_PROJECT})
export ORG_ROOT_ID=$(gcloud secrets versions access latest --secret=ORG_ROOT_ID --project ${SECRETS_PROJECT})
export PATH=$HOME/.local/bin:$PATH

HISTIGNORE='ll:ls:zsh'

######## ENSURING DIRECTORIES AND NON-IMAGE APPS EXIST ############
LOCAL_BIN_DIR="$HOME/.local/bin"
if [ ! -d ${LOCAL_BIN_DIR} ]; then
    echo "'${LOCAL_BIN_DIR}' not found. Creating now."
    mkdir -p ${LOCAL_BIN_DIR}
fi

TMP_DIR="$HOME/tmp"
if [ ! -d ${TMP_DIR} ]; then
    echo "'${TMP_DIR}' not found. Creating now."
    mkdir -p ${TMP_DIR}
fi

NVIM_BIN=${LOCAL_BIN_DIR}/nvim.appimage
if [ ! -f ${NVIM_BIN} ]; then
    # from: https://practical.li/neovim/install/neovim/#suppoting-tools
    wget -O "${NVIM_BIN}" https://github.com/neovim/neovim/releases/download/v0.10.2/nvim.appimage
    chmod +x "${NVIM_BIN}"
    ln -s ${NVIM_BIN} ${LOCAL_BIN_DIR}/nvim
fi

# install the config-connector app
if ! dpkg -l google-cloud-sdk-config-connector >/dev/null 2>&1; then
    echo "google-cloud-sdk-config-connector is not installed. Installing now..."
    sudo apt-get install -y google-cloud-sdk-config-connector
fi

ULID_BIN="${LOCAL_BIN_DIR}/ulid"
if [ ! -f ${ULID_BIN} ]; then
    echo "ulid binary not found.  Installing now"
    wget -O "${ULID_BIN}" https://github.com/PHACDataHub/rust-tools/releases/download/ulid-v1.0/ulid
    chmod +x "${ULID_BIN}"
fi

YQ_PATH="${LOCAL_BIN_DIR}/yq"
if [ ! -f $YQ_PATH ]; then
    echo "yq binary not found.  Installing now"
    YQ_BINARY=yq_linux_amd64
    YQ_VERSION=v4.44.3
    wget https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/${YQ_BINARY}.tar.gz -O - | tar xz && sudo mv ${YQ_BINARY} ${YQ_PATH}
fi

# k9s has a package in homebrew.  This is the simplest way to get it installed.
# https://docs.brew.sh/Installation#alternative-installs
HOMEBREW_DIR="$HOME/homebrew"
HOMEBREW_BIN=${HOMEBREW_DIR}/bin/brew
if [ ! -d ${HOMEBREW_DIR} ]; then
    mkdir ${HOMEBREW_DIR} 
    if [ ! -f ${HOMEBREW_BIN} ]; then
        echo "homebrew binary not found.  Installing now"
        curl -L https://github.com/Homebrew/brew/tarball/master | tar xz --strip-components 1 -C ${HOMEBREW_DIR}
        eval "$(${HOMEBREW_BIN} shellenv)"
        brew update --force --quiet
        chmod -R go-w "$(brew --prefix)/share/zsh"
    fi
else 
    eval "$(${HOMEBREW_BIN} shellenv)"
fi

K9S_BIN=${HOMEBREW_DIR}/bin/k9s
if [ -f "${HOMEBREW_BIN}" ] && [ ! -f "${K9S_BIN}" ]; then
    echo "k9s binary not found.  Installing now"
    ${HOMEBREW_BIN} install derailed/k9s/k9s
fi

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

########## FUNCTIONS #############
function replace_setter_ulid {
    NEW_SHORT_ULID=`$HOME/.local/bin/ulid -sl`
    yq -i ".data.project-unique-id = \"${NEW_SHORT_ULID}\"" $1
}

# function get_ulid(){
#     curl --silent https://ulid.truestamp.com/ | jq -r '.[] | .ulid' | awk '{print tolower($0)}'
# }
function get_ulid(){
    ${ULID_BIN}/ulid
}

function remove_duplicate_bash_history() {
    # Remove all duplicates from .bash_history while keeping the history in order
    HISTFILE=~/.bash_history
    temp_histfile="/tmp/$$.temp_histfile"
    tac ~/.bash_history | awk '!x[$0]++' | tac > $temp_histfile
    mv $temp_histfile $HISTFILE
}


########### ALIASES ##############
# New Project ID
alias npi="/home/admin_keith_young/ulid -sl"
alias kpt_pkg_get_dflt_prj="kpt pkg get https://github.com/PHACDataHub/acm-core/tree/main/templates/default-project"
alias cls="clear"

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

echo ".bashrc COMPLETED EXECUTION"

```