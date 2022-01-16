# Create a Virtualbox guest VM by Vagrant

## Prerequisites
* A public key in .ssh/id_rsa.pub in your home directory
* Directly download [Virtualbox](https://www.virtualbox.org/) and install, or use homebrew
    ```
    brew install --cask virtualbox
    ```
* Vagrant
    ```
    brew install --cask vagrant
    ```
* [(Optional) Vagrant Manager](http://vagrantmanager.com/)
    ```
    brew install --cask vagrant-manager
    ```

## Provision VM
```
cd create-virtualbox-vm
vagrant up
```

Let's ssh into the Virtualbox guest VM
```
ssh ci@172.16.16.16
```
