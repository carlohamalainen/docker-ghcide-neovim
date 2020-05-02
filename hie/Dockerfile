FROM debian:10
MAINTAINER Carlo Hamalainen <carlo@carlo-hamalainen.net>

ENV         PATH    /root/.cabal/bin:/opt:/opt/ghcide_bin:$PATH

################################ build requirements ########################################
RUN         apt-get -qq update
RUN         apt-get -qqy install git build-essential                                \
                                 zlib1g-dev libicu-dev libncurses-dev libgmp-dev    \
                                 npm nodejs wget curl                               \
                                 ghc cabal-install 

################################# freshen up cabal-install #########################
RUN         cabal update -v
RUN         cabal install cabal-install

######################################## ghcide #################################
#
# Tweak the constraint on ghc-check, a known issue:
#
#       https://github.com/digital-asset/ghcide/issues/536
#
RUN         git clone https://github.com/digital-asset/ghcide.git /opt/ghcide
WORKDIR     /opt/ghcide
RUN         git checkout 78d4031f7cf119bf2a0d1fae6157d52d1fbd2063
RUN         sed -i 's@ghc-check >= 0.1.0.3,@ghc-check == 0.1.0.3,@g' ghcide.cabal
RUN         cabal install --installdir=/opt/ghcide_bin

################################## haskell-ide-engine #################################

RUN         git clone https://github.com/haskell/haskell-ide-engine --recurse-submodules /opt/haskell-ide-engine
WORKDIR     /opt/haskell-ide-engine

# Many "Starting LS languageserver.haskell" notifications:
# https://github.com/haskell/haskell-ide-engine/issues/1749
RUN         git checkout a5d8c1968c7bf1bcbec6160235a3fa024ad7c55f
RUN         git submodule update --init --recursive
RUN         cabal v2-run ./install.hs --project-file install/shake.project hie

############################### neovim, plugged, coc ##########################

RUN         wget https://github.com/neovim/neovim/releases/download/v0.4.3/nvim.appimage -O /opt/nvim
RUN         chmod +x /opt/nvim
ENV         APPIMAGE_EXTRACT_AND_RUN 1
RUN         mkdir -p /root/.config/nvim/plugged
RUN         curl -fLo ~/.local/share/nvim/site/autoload/plug.vim \
                 --create-dirs \
                 https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
ADD         init.vim /root/.config/nvim/

# Note the language server sample on http://marco-lopes.com/articles/Vim-and-Haskell-in-2019/
# doesn't work anymore; see the updated one taken from https://github.com/haskell/haskell-ide-engine/#installation
ADD         coc-settings.json /root/.config/nvim/coc-settings.json

# Running PlugInstall causes some drama (xterm colour codes get dropped out to the
# shell that calls docker!) so I'll just clone directly and grab particular versions.

WORKDIR     /root/.config/nvim/plugged
RUN         git clone --branch release https://github.com/neoclide/coc.nvim.git
WORKDIR     /root/.config/nvim/plugged/coc.nvim
RUN         git checkout b1b8d50b8b62cbd50c520726b371da96b20f63a3

WORKDIR     /root/.config/nvim/plugged
RUN         git clone https://github.com/neovimhaskell/haskell-vim.git
WORKDIR     /root/.config/nvim/plugged/haskell-vim
RUN         git checkout b1ac46807835423c4a4dd063df6d5b613d89c731

WORKDIR     /root/.config/nvim/plugged
RUN         git clone https://github.com/alx741/vim-hindent.git
WORKDIR     /root/.config/nvim/plugged/vim-hindent
RUN         git checkout f8e84c199fd00a3ccaf5bbbc97786bde9a4faa13

# Note that Foo/hie.yaml manually specifies the project, otherwise
# we hit this error about Cabal versions:
#
#       https://github.com/haskell/haskell-ide-engine/issues/1734
#
ADD         Foo /opt/Foo

WORKDIR     /opt/Foo

RUN         hie-wrapper

RUN         ghcide || echo "non-zero return code from ghcide; ignoring"

CMD /bin/bash
