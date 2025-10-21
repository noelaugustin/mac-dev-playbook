<img src="https://raw.githubusercontent.com/geerlingguy/mac-dev-playbook/master/files/Mac-Dev-Playbook-Logo.png" width="250" height="156" alt="Mac Dev Playbook Logo" />

# Mac Development Ansible Playbook

[![CI][badge-gh-actions]][link-gh-actions]

This playbook installs and configures most of the software I use on my Mac for web and software development. Some things in macOS are slightly difficult to automate, so I still have a few manual installation steps, but at least it's all documented here.

## Installation

  1. Ensure Apple's command line tools are installed (`xcode-select --install` to launch the installer).
  2. [Install Ansible](https://docs.ansible.com/ansible/latest/installation_guide/index.html):

     1. Run the following command to add Python 3 to your $PATH: `export PATH="$HOME/Library/Python/3.9/bin:/opt/homebrew/bin:$PATH"`
     2. Upgrade Pip: `sudo pip3 install --upgrade pip`
     3. Install Ansible: `pip3 install ansible`

  3. Clone or download this repository to your local drive.
  4. Run `ansible-galaxy install -r requirements.yml` inside this directory to install required Ansible roles.
  5. Run `ansible-playbook main.yml --ask-become-pass` inside this directory. Enter your macOS account password when prompted for the 'BECOME' password.

> Note: If some Homebrew commands fail, you might need to agree to Xcode's license or fix some other Brew issue. Run `brew doctor` to see if this is the case.

### Use with a remote Mac

You can use this playbook to manage other Macs as well; the playbook doesn't even need to be run from a Mac at all! If you want to manage a remote Mac, either another Mac on your network, or a hosted Mac like the ones from [MacStadium](https://www.macstadium.com), you just need to make sure you can connect to it with SSH:

  1. (On the Mac you want to connect to:) Go to System Settings > Sharing.
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

You can filter which part of the provisioning process to run by specifying a set of tags using `ansible-playbook`'s `--tags` flag. The tags available are `dotfiles`, `homebrew`, `mas`, `extra-packages`, `osx`, and `docker-context`.

    ansible-playbook main.yml -K --tags "dotfiles,homebrew"

## Overriding Defaults

Not everyone's development environment and preferred software configuration is the same.

You can override any of the defaults configured in `default.config.yml` by creating a `config.yml` file and setting the overrides in that file. For example, you can customize the installed packages and apps with something like:

```yaml
homebrew_installed_packages:
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

configure_dock: true
dockitems_remove:
  - Launchpad
  - TV
dockitems_persist:
  - name: "Sublime Text"
    path: "/Applications/Sublime Text.app/"
    pos: 5

configure_docker_context: true
docker_contexts:
  - name: "production"
    host: "prod-docker.example.com"
    user: "deploy"
    port: 22
    description: "Production Docker environment"
  - name: "staging"
    host: "staging-docker.example.com"
    user: "deploy"
    description: "Staging Docker environment"
docker_context_default: "production"
```

Any variable can be overridden in `config.yml`; see the supporting roles' documentation for a complete list of available variables.

## Included Applications / Configuration (Default)

Applications (installed with Homebrew Cask):

  - [ChromeDriver](https://sites.google.com/chromium.org/driver/)
  - [Docker](https://www.docker.com/)
  - [Dropbox](https://www.dropbox.com/)
  - [Firefox](https://www.mozilla.org/en-US/firefox/new/)
  - [Google Chrome](https://www.google.com/chrome/)
  - [Handbrake](https://handbrake.fr/)
  - [Homebrew](http://brew.sh/)
  - [LICEcap](http://www.cockos.com/licecap/)
  - [nvALT](http://brettterpstra.com/projects/nvalt/)
  - [Sequel Ace](https://sequel-ace.com) (MySQL client)
  - [Slack](https://slack.com/)
  - [Sublime Text](https://www.sublimetext.com/)
  - [Transmit](https://panic.com/transmit/) (S/FTP client)

Packages (installed with Homebrew):

  - autoconf
  - bash-completion
  - doxygen
  - gettext
  - gifsicle
  - git
  - gh
  - go
  - gpg
  - httpie
  - iperf
  - libevent
  - sqlite
  - nmap
  - node
  - nvm
  - php
  - ssh-copy-id
  - readline
  - openssl
  - pv
  - wget
  - wrk
  - zsh-history-substring-search

My [dotfiles](https://github.com/geerlingguy/dotfiles) are also installed into the current user's home directory, including the `.osx` dotfile for configuring many aspects of macOS for better performance and ease of use. You can disable dotfiles management by setting `configure_dotfiles: no` in your configuration.

Finally, there are a few other preferences and settings added on for various apps and services.

## Docker Context Configuration

This playbook can configure SSH-based Docker contexts to manage remote Docker environments. This is useful for connecting to Docker hosts on remote servers, development environments, or cloud instances.

### Prerequisites

1. **Docker installed** on your local Mac (included in the default homebrew applications)
2. **SSH access** to the remote Docker host
3. **SSH authentication** configured using either:
   - SSH key-based authentication with key files, OR
   - SSH agent (like Bitwarden SSH agent, ssh-agent, etc.)
4. **Docker daemon running** on the remote host

### Configuration

To enable Docker context configuration, set `configure_docker_context: true` in your `config.yml` and define your Docker contexts:

**For traditional SSH key files:**
```yaml
configure_docker_context: true
docker_context_use_ssh_agent: false
docker_context_ssh_key_path: "~/.ssh/id_rsa"  # Path to your SSH private key
docker_context_default: "production"          # Default context to use
docker_context_verify: true                   # Verify connections after setup
docker_contexts:
  - name: "production"
    host: "prod-docker.example.com"
    user: "deploy"
    port: 22
    description: "Production Docker environment"
```

**For Bitwarden SSH agent or other SSH agents:**
```yaml
configure_docker_context: true
docker_context_use_ssh_agent: true           # Use SSH agent instead of key files
docker_context_default: "production"         # Default context to use
docker_context_verify: true                  # Verify connections after setup
docker_contexts:
  - name: "production"
    host: "prod-docker.example.com"
    user: "deploy"
    port: 22
    description: "Production Docker environment"
  - name: "staging"
    host: "staging-docker.example.com" 
    user: "deploy"
    description: "Staging Docker environment"
```

### Usage

After running the playbook with Docker context configuration:

```bash
# Run only Docker context configuration
ansible-playbook main.yml -K --tags "docker-context"

# List available contexts
docker context ls

# Switch between contexts
docker context use production
docker context use staging
docker context use default  # Local Docker

# Verify current context
docker info
```

The playbook will:
- Verify Docker is installed
- Check SSH authentication method (key file or SSH agent)
- Validate SSH agent connectivity (for Bitwarden SSH agent users)
- Create the defined Docker contexts
- Set the default context (if specified)
- Verify connections to remote Docker hosts

**Note for Bitwarden SSH agent users:** Make sure your Bitwarden SSH agent is running and has the necessary SSH keys loaded before running the playbook. The playbook will check `ssh-add -l` to verify SSH agent connectivity.

## Full / From-scratch setup guide

Since I've used this playbook to set up something like 20 different Macs, I decided to write up a full 100% from-scratch install for my own reference (everyone's particular install will be slightly different).

You can see my full from-scratch setup document here: [full-mac-setup.md](full-mac-setup.md).

## Testing the Playbook

Many people have asked me if I often wipe my entire workstation and start from scratch just to test changes to the playbook. Nope! This project is [continuously tested on GitHub Actions' macOS infrastructure](https://github.com/geerlingguy/mac-dev-playbook/actions?query=workflow%3ACI).

You can also run macOS itself inside a VM, for at least some of the required testing (App Store apps and some proprietary software might not install properly). I currently recommend:

  - [UTM](https://mac.getutm.app)
  - [Tart](https://github.com/cirruslabs/tart)

## Ansible for DevOps

Check out [Ansible for DevOps](https://www.ansiblefordevops.com/), which teaches you how to automate almost anything with Ansible.

## Author

This project was created by [Jeff Geerling](https://www.jeffgeerling.com/) (originally inspired by [MWGriffin/ansible-playbooks](https://github.com/MWGriffin/ansible-playbooks)).

[badge-gh-actions]: https://github.com/geerlingguy/mac-dev-playbook/actions/workflows/ci.yml/badge.svg
[link-gh-actions]: https://github.com/geerlingguy/mac-dev-playbook/actions/workflows/ci.yml
