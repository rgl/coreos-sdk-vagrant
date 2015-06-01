# -*- mode: ruby -*-
# vi: set ft=ruby :

$root_provision_script = <<'ROOT_PROVISION_SCRIPT'
#!/bin/bash
# abort this script on errors.
set -eux

# prevent apt-get et al from opening stdin.
# NB even with this, you'll still get some warnings that you can ignore:
#     dpkg-preconfigure: unable to re-open stdin: No such file or directory
export DEBIAN_FRONTEND=noninteractive

apt-get -y update
apt-get -y upgrade

# install vim because I like it.
apt-get -y install vim

# install coreos-sdk dependencies
apt-get -y install git
apt-get -y install curl
apt-get -y install xz-utils
ROOT_PROVISION_SCRIPT

$vagrant_provision_script = <<'VAGRANT_PROVISION_SCRIPT'
#!/bin/bash
# abort this script on errors.
set -eux

# setup the shell.
cat<<"EOF" > ~/.bashrc
if [[ $- != *i* ]] ; then
  # bail if the shell is non-interactive.
  return
fi

export EDITOR=vim
export PAGER=less

alias l='ls -lF --color'
alias ll='l -a'
alias h='history 25'
alias j='jobs -l'
EOF

cat<<"EOF" > ~/.inputrc
"\e[A": history-search-backward
"\e[B": history-search-forward
"\eOD": backward-word
"\eOC": forward-word
set show-all-if-ambiguous on
set completion-ignore-case on
EOF

# setup git.
git config --global user.name 'Rui Lopes'
git config --global user.email rgl@ruilopes.com
git config --global push.default simple

# setup vim.
cat<<"EOF" > ~/.vimrc
syntax on
set background=dark
set esckeys
set ruler
set laststatus=2
set nobackup

autocmd BufNewFile,BufRead Vagrantfile set ft=ruby
autocmd BufNewFile,BufRead *.config set ft=xml

" Usefull setting for working with Ruby files.
autocmd FileType ruby set tabstop=2 shiftwidth=2 smarttab expandtab softtabstop=2 autoindent
autocmd FileType ruby set smartindent cinwords=if,elsif,else,for,while,try,rescue,ensure,def,class,module

" Usefull setting for working with Python files.
autocmd FileType python set tabstop=4 shiftwidth=4 smarttab expandtab softtabstop=4 autoindent
" Automatically indent a line that starts with the following words (after we press ENTER).
autocmd FileType python set smartindent cinwords=if,elif,else,for,while,try,except,finally,def,class

" Usefull setting for working with Go files.
autocmd FileType go set tabstop=4 shiftwidth=4 smarttab expandtab softtabstop=4 autoindent
" Automatically indent a line that starts with the following words (after we press ENTER).
autocmd FileType go set smartindent cinwords=if,else,switch,for,func
EOF


# install the repo tool.
mkdir ~/bin
curl -o bin/repo https://storage.googleapis.com/git-repo-downloads/repo
chmod +x ~/bin/repo

cat<<"EOF" >> ~/.bashrc
# put the repo tool on PATH.
export PATH="$PATH:~/bin"
EOF
export PATH="$PATH:~/bin"

# get the sdk.
mkdir coreos && cd coreos
repo init -u https://github.com/coreos/manifest.git
repo sync

cat<<"EOF" >> ~/.bashrc
# put the chromite tools on PATH.
export PATH="$PATH:~/coreos/chromite/bin"
EOF
export PATH="$PATH:~/coreos/chromite/bin"

# create the sdk chroot.
cros_sdk --create

cat<<"EOF"
# ALL DONE
#
# the sdk is now installed inside the VM.
#
# get the VM IP address:
#
#  vagrant ssh -c 'ip -4 addr | grep inet'
#
# enter the VM with vagrant ssh then:
#
# type cros_sdk --enter to enter the chroot
# type cros_sdk --delete to delete the chroot
EOF
VAGRANT_PROVISION_SCRIPT

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu-14.10-amd64"

  config.vm.hostname = "coreos-sdk"

  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.memory = 4096
    vb.cpus = 2
  end

  config.vm.network "private_network", type: "dhcp"

  config.vm.provision "shell", inline: $root_provision_script
  config.vm.provision "shell", inline: $vagrant_provision_script, privileged: false
end
