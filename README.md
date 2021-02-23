<img src="https://raw.githubusercontent.com/Joostvanderlaan/mac-dev-playbook/master/files/Mac-Dev-Playbook-Logo.png" width="250" height="156" alt="Mac Dev Playbook Logo" />

# Mac Development Ansible Playbook

[![CI][badge-gh-actions]][link-gh-actions]

This playbook installs and configures most of the software I use on my Mac for web and software development. Some things in macOS are slightly difficult to automate, so I still have some manual installation steps, but at least it's all documented here.

This is a work in progress, and is mostly a means for me to document my current Mac's setup. I'll be evolving this playbook over time.

*See also*:

  - Forked from: [geerlingguy/mac-dev-playbook](https://github.com/geerlingguy/mac-dev-playbook) 
  - [Boxen](https://github.com/boxen)
  - [Battleschool](http://spencer.gibb.us/blog/2014/02/03/introducing-battleschool)
  - [osxc](https://github.com/osxc)
  - [MWGriffin/ansible-playbooks](https://github.com/MWGriffin/ansible-playbooks) (the original inspiration for this project)

## Installation

  1. Ensure Apple's command line tools are installed (`xcode-select --install` to launch the installer).
  1. Enable Rosetta2 `/usr/sbin/softwareupdate --install-rosetta --agree-to-license`
  2. [Install Ansible](http://docs.ansible.com/intro_installation.html). (pip install)
      ```shell
      # install pip
      curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
      python3 get-pip.py --user
      # install ansible
      python3 -m pip install --user ansible
      ```
       1. In case of cryptography.io complaining about missing Rust: 
          ```bash
          /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
          ```
          Add Homebrew to your **PATH**: (the installer will tell you to)
          ```
          echo 'eval $(/opt/homebrew/bin/brew shellenv)' >> ~/.zprofile
          eval $(/opt/homebrew/bin/brew shellenv)
          ```
          Install Rust:
          ```
          brew install openssl@1.1 rust
          ```
          install cryptography using openssl as MacOS uses LibreSSL:
          ```bash
          env LDFLAGS="-L$(brew --prefix openssl@1.1)/lib" CFLAGS="-I$(brew --prefix openssl@1.1)/include" pip install cryptography
          ```
          Re-run the ansible install above
  3. Clone this repository to your local drive. `git clone https://github.com/Joostvanderlaan/mac-dev-playbook.git`
  4. Run `ansible-galaxy install -r requirements.yml` inside this directory to install required Ansible roles.
  5. Run `ansible-playbook main.yml -i inventory --ask-become-pass` inside this directory. Enter your account password when prompted. Or only the homebrew stuff with `ansible-playbook main.yml -i inventory -K --tags "homebrew" --ask-become-pass`

### Use with a remote Mac

You can use this playbook to manage other Macs as well; the playbook doesn't even need to be run from a Mac at all! ([Install ansible on Ubuntu & others])(https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-ubuntu) If you want to manage a remote Mac, either another Mac on your network, or a hosted Mac like the ones from [MacStadium](https://www.macstadium.com), you just need to make sure you can connect to it with SSH:

  1. (On the Mac you want to connect to:) Go to System Preferences > Sharing.
  2. Enable 'Remote Login'.

> You can also enable remote login on the command line:
>
>     sudo systemsetup -setremotelogin on

Then edit the `inventory` file in this repository and change the line that starts with `127.0.0.1` to:

```
[ip address or hostname of mac]  ansible_user=[mac ssh username]
```

If you need to supply an SSH password (if you don't use SSH keys), make sure to pass the `--ask-pass` parameter to the `ansible-playbook` command.

### Running a specific set of tagged tasks

You can filter which part of the provisioning process to run by specifying a set of tags using `ansible-playbook`'s `--tags` flag. The tags available are `dotfiles`, `homebrew`, `mas`, `extra-packages` and `osx`.

    ansible-playbook main.yml -i inventory -K --tags "dotfiles,homebrew"

## Overriding Defaults

Not everyone's development environment and preferred software configuration is the same.

You can override any of the defaults configured in `default.config.yml` by creating a `config.yml` file and setting the overrides in that file. For example, you can customize the installed packages and apps with something like:

    homebrew_installed_packages:
      - cowsay
      - git
      - go
    
    mas_installed_apps:
      - { id: 443987910, name: "1Password" }
      - { id: 498486288, name: "Quick Resizer" }
      - { id: 557168941, name: "Tweetbot" }
      - { id: 497799835, name: "Xcode" }
    
    composer_packages:
      - name: hirak/prestissimo
      - name: drush/drush
        version: '^8.1'
    
    gem_packages:
      - name: bundler
        state: latest
    
    npm_packages:
      - name: webpack
    
    pip_packages:
      - name: mkdocs

Any variable can be overridden in `config.yml`; see the supporting roles' documentation for a complete list of available variables.

## Included Applications / Configuration (Default)


My [dotfiles](https://github.com/geerlingguy/dotfiles) are also installed into the current user's home directory, including the `.osx` dotfile for configuring many aspects of macOS for better performance and ease of use. You can disable dotfiles management by setting `configure_dotfiles: no` in your configuration.

Finally, there are a few other preferences and settings added on for various apps and services.

## Future additions

### Things that still need to be done manually

It's my hope that I can get the rest of these things wrapped up into Ansible playbooks soon, but for now, these steps need to be completed manually (assuming you already have Xcode and Ansible installed, and have run this playbook).

  1. Turn on **FileVault** under Security & Privacy preferences.
  1. **[Generate SSH key](https://docs.gitlab.com/ee/ssh/#ed25519-ssh-keys)** and add to GitLab & GitHub
  1. setup git details
      ```
      git config --global user.name "Your Name" /
      git config --global user.email you@example.com
      ```
  1. Install [Google Cloud SDK](https://cloud.google.com/sdk/docs/install)
      - Download and extract, `cd ~/Downloads && ./google-cloud-sdk/install.sh`
  1. Install [Cloud SQL Proxy](https://cloud.google.com/sql/docs/mysql/sql-proxy#install)
      ```
      cd ~ && curl -o cloud_sql_proxy https://dl.google.com/cloudsql/cloud_sql_proxy.darwin.amd64 && chmod +x cloud_sql_proxy
      ```
  1. Add Sublime text key
  1. Add GitLab token for glab CLI tool `glab auth login` [GitLab Access Tokens](https://gitlab.com/-/profile/personal_access_tokens)
  1. Clone all git repos. For every gitlab project in a group, use this one liner with curl, jq, tr
     ```
     for repo in $(curl -s --header "PRIVATE-TOKEN: your_private_token" https://<your-host>/api/v4/groups/<group_id> | jq ".projects[].ssh_url_to_repo" | tr -d '"'); do git clone $repo; done;
     ```
     For Gitlab.com use https://gitlab.com/api/v4/groups/<group_id>
  1. `nvm install --lts` (and edit `vi ~/.zshrc` to add nvm or through dotfiles)
  1. Add mail accounts to Thunderbird, addons:
      - [Nederlands woordenboek](https://addons.thunderbird.net/en-US/thunderbird/addon/woordenboek-nederlands/)
      - [DKIM verifier](https://addons.thunderbird.net/en-US/thunderbird/addon/dkim-verifier/?src=search)
      - [Thunderbird Conversations view](https://addons.thunderbird.net/en-US/thunderbird/addon/gmail-conversation-view/) &ndash; makes mailbox more efficient & checks settings like offline storage, global search etc.
      - [Quicktext](https://addons.thunderbird.net/en-US/thunderbird/addon/quicktext/) &ndash; Templates\
  1. Download books and put into Calibre
  2. setup git-sync for git repos that need auto-sync like notes repo
  
  1. Set JJG-Term as the default Terminal theme (it's installed, but not set as default automatically).
  2. Install [Sublime Package Manager](http://sublime.wbond.net/installation).
  3. Install all the apps that aren't yet in this setup (see below).
  4. Remap Caps Lock to Escape (requires macOS Sierra 10.12.1+).
  5. Set trackpad tracking rate.
  6. Set mouse tracking rate.
  7. Configure extra Mail and/or Calendar accounts (e.g. Google, Exchange, etc.).


  ### Workaround Intel vs Apple Silicon Mac
  
  I ran into the issue that `geerlingguy.homebrew` sets `/usr/local` as homebrew dir, which is fine for Intel macs but M1 macs should use `/opt/homebrew`. This is easily fixed by adding the var in default.config.yml, but apps were already installed by then. To remove them:

cd /Applications
rm -r "Authy Desktop.app" calibre.app darktable.app Docker.app Dropbox.app Figma.app Firefox.app Gimp-2.10.app "Google Chrome.app" "Backup and Sync.app" Handbrake.app Inkscape.app LibreOffice.app LICEcap.app Postman.app "Sequel Pro.app" Signal.app Spotify.app "Sublime Text.app" Thunderbird.app "Visual Studio Code.app" VLC.app

You can also set arch on Mac by using `arch -x86_64 brew remove <package>`

Also, could not get brew packages installed quickly. For now used directly instead of ansible script:

```bash
sudo chown -R $(whoami) /opt/homebrew
chmod u+w /opt/homebrew
```

## Manual install

```
brew install autoconf baobab bash-completion erlang gettext gifsicle git gh glab go gpg htop httpie hugo imagemagick iperf libevent pandoc sqlite mcrypt nethogs nmap node nvm p7zip ssh-copy-id cowsay readline terraform tmux thefuck openssl pv webp wget wrk
```

```
brew install authy bitwarden calibre chromedriver darktable docker digikam dropbox figma firefox gimp google-chrome google-drive-file-stream google-photos-backup-and-sync handbrake inkscape libreoffice licecap postman sequel-pro signal spotify sublime-text thunderbird visual-studio-code vlc
```

### Configuration to be added:

  - I have vim configuration in the repo, but I still need to add the actual installation:
    ```
    mkdir -p ~/.vim/autoload
    mkdir -p ~/.vim/bundle
    cd ~/.vim/autoload
    curl https://raw.githubusercontent.com/tpope/vim-pathogen/master/autoload/pathogen.vim > pathogen.vim
    cd ~/.vim/bundle
    git clone git://github.com/scrooloose/nerdtree.git
    ```

## Testing the Playbook

Many people have asked me if I often wipe my entire workstation and start from scratch just to test changes to the playbook. Nope! Instead, I posted instructions for how I build a [Mac OS X VirtualBox VM](https://github.com/geerlingguy/mac-osx-virtualbox-vm), on which I can continually run and re-run this playbook to test changes and make sure things work correctly.

Additionally, this project is [continuously tested on GitHub Actions' macOS infrastructure](https://github.com/geerlingguy/mac-dev-playbook/actions?query=workflow%3ACI).

## Ansible for DevOps

Check out [Ansible for DevOps](https://www.ansiblefordevops.com/), which teaches you how to automate almost anything with Ansible.

## Author

[Jeff Geerling](https://www.jeffgeerling.com/), 2014 (originally inspired by [MWGriffin/ansible-playbooks](https://github.com/MWGriffin/ansible-playbooks)).

[badge-gh-actions]: https://github.com/geerlingguy/mac-dev-playbook/workflows/CI/badge.svg?event=push
[link-gh-actions]: https://github.com/geerlingguy/mac-dev-playbook/actions?query=workflow%3ACI

## Notes

  6. for Apple silicon (non-Intel) mac `brew bundle dump`
  1. Install homebrew Intel as well for the formulae that are intel`arch -x86_64 /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"` (and you can now use `arch -x86_64 brew install <package>`)

> Note: If some Homebrew commands fail, you might need to agree to Xcode's license or fix some other Brew issue. Run `brew doctor` to see if this is the case.
