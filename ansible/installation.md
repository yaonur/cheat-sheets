[[ansible]]

make ssh secure first for better experience
---
- Generate key
```bash
	ssh-keygen -t ed25519 -C "ssh key for servers"
```
- Copy key
```bash
	ssh-copy-id -i ~/.ssh/laptop_ssh.pub so2@192.168.2.10
```


Create ansible key
---
- Genereate key for ansible
```bash
   sh-keyget -t ed25519 -C "ssh key for ansible"
```
*creating pass for this key is not recommended*

```bash
ssh -i ~/.ssh/ansible so2@192.168.2.10
```






