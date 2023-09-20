Create Fedora CoreOS VM
=========

Create Fedora CoreOS VM.

```shell
ansible-playbook -i YOUR_HOST -u YOUR_SSH_USER --extra-vars "vm_name=$VM_NAME vm_vcpus=4 vm_memory_mb=16384 vm_disk
_size=100G ssh_user_password_hash=YOUR_PASSWORD_HASH ssh_authorized_key=YOUR_SSH_PUBLIC_KEY" create-fedora-coreos-vm.yaml
```
