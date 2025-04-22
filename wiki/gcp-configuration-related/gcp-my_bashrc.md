```bash
# BEGIN bashrc block.  Add to end of .bashrc uncomment the code to include my customizations.
###### MY CUSTOM .bashrc UPDATES BELOW HERE ######
# if [ -f ~/.my_bashrc ]; then
#     . ~/.my_bashrc
# fi
# echo ".bashrc COMPLETED EXECUTION"
# END bashrc block.

# everything below here goes into .my_bashrc


SECRETS_PROJECT=php-configconnector
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
if [ ! -f ${YQ_PATH} ]; then
    echo "yq binary not found.  Installing now"
    YQ_BINARY=yq_linux_amd64
    YQ_VERSION=v4.44.3
    wget https://github.com/mikefarah/yq/releases/download/${YQ_VERSION}/${YQ_BINARY}.tar.gz -O - | tar xz && sudo mv ${YQ_BINARY} ${YQ_PATH}
fi

CURLRC=$HOME/.curlrc
if [ ! -f "${CURLRC}" ] || ! grep -Fxq '-w "\n"' "${CURLRC}"; then
    echo '-w "\n"' >> "${CURLRC}"
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
if ! fc-list : family | grep -q "${FONT_NAME}"; then
    echo "${FONT_NAME} font is not installed. Downloading and installing..."
    
    if [ ! -d "${FONT_DIR}" ]; then
        mkdir -p "${FONT_DIR}"
    fi
    
    wget -O "${FONT_DIR}/FiraMono.zip" "${FONT_URL}"
    unzip "${FONT_DIR}/FiraMono.zip" -d "${FONT_DIR}"
    rm "${FONT_DIR}/FiraMono.zip"
    
    # Refresh the font cache
    fc-cache -fv
    
    echo "${FONT_NAME} font installed successfully."
fi

OHMYPOSH_BIN=${LOCAL_BIN_DIR}/oh-my-posh
if [ ! -f ${OHMYPOSH_BIN} ]; then
    # install oh-my-posh
    echo "oh-my-posh binary not found.  Installing now"
    curl -s https://ohmyposh.dev/install.sh | bash -s -- -d ${LOCAL_BIN_DIR}
fi

########## FUNCTIONS #############
replace_setter_ulid() {
    NEW_SHORT_ULID=`$HOME/.local/bin/ulid -sl`
    yq -i ".data.project-unique-id = \"${NEW_SHORT_ULID}\"" $1
}

# function get_ulid(){
#     curl --silent https://ulid.truestamp.com/ | jq -r '.[] | .ulid' | awk '{print tolower($0)}'
# }
get_ulid(){
    ${ULID_BIN}/ulid
}

remove_duplicate_bash_history() {
    # Remove all duplicates from .bash_history while keeping the history in order
    HISTFILE=~/.bash_history
    temp_histfile="/tmp/$$.temp_histfile"
    tac ~/.bash_history | awk '!x[$0]++' | tac > $temp_histfile
    mv $temp_histfile $HISTFILE
}

gcp_filter_project() {
    local search_term="$1"
    gcloud projects list --filter="name~$search_term OR project_id~$search_term"
}

gcp_switch_project() {
    local search_term="$1"
    local project_id
    local project_count

    # Store the output of the gcp_filter_project to avoid running it multiple times
    local project_output
    project_output=$(gcloud projects list --filter="name~$search_term OR project_id~$search_term")

    # Count the occurrences of "PROJECT_ID:" in the output to ensure only one project is returned
    project_count=$(echo "$project_output" | grep -c '^PROJECT_ID: ')
    if [ "$project_count" -eq 1 ]; then
        project_id=$(echo "$project_output" | sed -n 's/^PROJECT_ID: //p')
        gcloud config set project "$project_id"
    elif [ "$project_count" -gt 1 ]; then
        echo "Multiple projects found for search term: $search_term. Please refine your search."
        echo "Found projects:"
        echo "$project_output" | grep -E 'PROJECT_ID:|NAME:'
    else
        echo "No project ID found for search term: $search_term."
    fi
}




########### ALIASES ##############
# New Project ID
alias npi="/home/admin_keith_young/ulid -sl"
alias kpt_pkg_get_dflt_prj="kpt pkg get https://github.com/PHACDataHub/acm-core/tree/main/templates/default-project"
alias cls="clear"
alias gcsp="gcloud config set project"

# install oh-my-posh and add eval line to .bashrc
# curl -s https://ohmyposh.dev/install.sh | bash -s -- -d ~/.local/bin
# export current theme to customize it:
# mkdir ~/.config/ohmyposh
# oh-my-posh config export --output ~/.config/ohmyposh/.mytheme.omp.yaml
# insert the below block in the theme between the session and path segments
#   - type: gcp
#     style: powerline
#     powerline_symbol: 
#     foreground: "#ffffff"
#     background: "#47888d"
#     template: "  {{.Project}} "

if [ -f ${OHMYPOSH_BIN} ]; then
    eval "$(oh-my-posh init bash --config ~/.config/ohmyposh/.mytheme.omp.yaml)"
    # now exec bash to reload
fi

```