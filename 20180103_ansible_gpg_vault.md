## Ansible Vault and GPG usage

These pages will explain how gpg and ansible 2.4+ can be used to have various degrees of secrets within a single repository.
Personal, shared, team, company secrets and even cookies can be had.


* [Basic Ansible Vault usage](#vault)  
* [Basic GPG usage (and keychain)](#gpg)  
* [Multiple vaults in one repository, what is new in ansible 2.4](#vaults)  
* [Ultra-Mega-GPG-Vault-Id-Label-Combo!](#combo)  


<a name=vault></a>
### Basic Ansible Vault usage
Ansible comes with ansible-vault, a program that will let you encrypt files (also separate strings, but this is outside the scope of this documentation).

**Easy usage:**  
> ansible-vault encrypt ~/file/to/encrypt.yml  

Ansible Vault will ask you for a password, encrypt the whole file.
Whenever ansible will need to read the file (be it in playbooks, roles or in plain task mode) it will ask you for that password decrypt it in memory and use what it needs from it.


**Edit the file:**  
> ansible-vault edit ~/file/to/encrypt.yml  

When you quit it will re-encrypt the file, even if nothing changed, your version control will notice it.

**Viewing can be done with:**  
> ansible-vault view ~/file/to/encrypt.yml

This will not change the file, so version control stays happy.


**Password files**  
Ansible Vault passwords can be put in a file, when using the flag --vault-password-file or the Environment Variable ANSIBLE_VAULT_PASSWORD_FILE you don't need to enter a password.

examples:  
> ansible-vault encrypt --vault-password-file ~/.ansible_secret_password ~/file/to/encrypt.yml
> ansible-vault edit --vault-password-file ~/.ansible_secret_password ~/file/to/encrypt.yml
> ansible-vault view --vault-password-file ~/.ansible_secret_password ~/file/to/encrypt.yml  

<!-- -->
> export ANSIBLE_VAULT_PASSWORD_FILE="~/.ansible_secret_password"
> ansible-vault encrypt ~/file/to/encrypt.yml
> ansible-vault edit ~/file/to/encrypt.yml
> ansible-vault view ~/file/to/encrypt.yml

So far the basic usage of Ansible Vault.

*More reading material:*  
\+ https://gist.github.com/tristanfisher/e5a306144a637dc739e7  
\+ https://docs.ansible.com/ansible/2.4/vault.html



<br><br><br><a name=gpg></a>
### Basic GPG usage (and keychain)

**What is GPG?**  
Those unfamiliar with GPG are strongly encouraged to read :  
\+ http://curtiswallen.com/pgp/  
\+ https://en.wikipedia.org/wiki/GNU_Privacy_Guard

**Basic usage:**  
Before generating your key it might be a good idea to add some entropy to your system, installing rng-tools will vastly improve this.  
We will be using gpg2 for these pages, you might want to do so too :)  

**Generate your key:** 
> gpg2 --full-gen-key  

Required items:  
- type of key (default)  
- keysize
- key validity  
- name  
- email  
- Passphrase  

Optional:  
- Comment (when used in a  this might be the name of the computer you are using this key on)


**Listing the keys on your system:**  
> gpg2 --list-keys  
> gpg2 --list-secret-keys  

**Generating a revocation certificate:**  
> gpg2 --output revoke.asc --gen-revoke (email|key-id|name)  

If you forget your passphrase or if your private key is compromised or lost, this revocation certificate may be published to notify others that the public key should no longer be used.

**Exporting a public key (will be needed later on)**
> gpg2 --armor --export (email|key-id|name) > keyname_gpg.pub

This file may be shared with everyone that may encrypt a message or a file towards you.

**Importing public keys:**  
> gpg2 --import keyname_gpg.pub  

**Encrypt a document:**  
> gpg2 --output document.gpg --encrypt --recipient (email|key-id|name) document  

The recipient option is repeated for every public key of the users you wish to be able to decrypt the message.  

**Decrypt a document:**  
> gpg2 --decrypt document.gpg --output document


That's all there is to it! ...  
*"We like security, but it has to be easy!"*

**That's where agents like keychain comes in!**  

Keychain is a daemon that will load ssh and gpg keys for your user and keep them "unlocked" as long as your computer is running (or the timeout you specify).  
Add the following to your ~/.bashrc (or similar configuration file):  
> eval \`keychain --gpg2 --eval --agents gpg (email|key-id|name) --nogui\`  

Depending on your setup you might need to remove --gpg2 from the command, depending on your preferences even the --nogui can go.  
If you would like keychain to load your ssh keys you may add ssh to the agents like this: --agents gpg,ssh  

**beware:** *for full gpg2 compatibility keychain version 2.8.4 or higher is required !*

More (gui?) keychain like tools are available, feel free to find the tool that suits your needs:  
\+ https://gnupg.org/software/frontends.html  

*More reading materials:*  
\+ https://www.gnupg.org/gph/en/manual/c14.html  
\+ http://blog.ghostinthemachines.com/2015/03/01/how-to-use-gpg-command-line/  
\+ https://sanctum.geek.nz/arabesque/gnu-linux-crypto-agents/  
\+ https://www.funtoo.org/Keychain  


<br><br><br><a name=vaults></a>
### Multiple vaults in one repository, what is new in ansible 2.4

Multiple vaults can be used in ansible, until 2.4 one could encrypt a file with one password, and encrypt another file with a second password.  
Unless both files were needed at the same time this would work well enough.  

One of the Proof of Concepts done around this can be found here: https://github.com/brianmor/ansible-poc  
In the case of the poc above one would launch the Vault view command like this:

> user ~ansible/ > ansible-vault view  --vault-password-file vault_pass_production.txt inventories/production/group_vars/all/vault  
> user ~ansible/ > ansible-vault view  --vault-password-file vault_pass_testing.txt inventories/testing/group_vars/all/vault  


**multiple passwords and Vault Ids:**
With Ansible 2.4 some additions have been made to the vaults part: https://docs.ansible.com/ansible/2.4/vault.html  
Multiple vault-id options can be used when editing encrypted files or vars, and the addition of labels makes it easier to see what vault-id was used to encrypt the values.  

These changes made it easier to use different vault ids (passwords) for different parts of ones environment (rnd, dev, test, uat, prod, or even others) in a repository.  
It goes without saying that added configuration and environment variables were added to ansible to support this new way of working (https://docs.ansible.com/ansible/2.4/config.html#default-vault-id-match ).  
> DEFAULT_VAULT_ID_MATCH / ANSIBLE_VAULT_ID_MATCH
> DEFAULT_VAULT_IDENTITY / ANSIBLE_VAULT_IDENTITY
> DEFAULT_VAULT_IDENTITY_LIST / ANSIBLE_VAULT_IDENTITY_LIST

in the previous example with Ansible 2.4 you could now launch  
> user ~ansible/ > ansible-vault view  --vault-id vault_pass_production.txt --vault-id vault_pass_testing.txt  inventories/production/group_vars/all/vault inventories/testing/group_vars/all/vault  

and you would see both items decrypted.

**Labeled vault-ids:**  
Also new in ansible 2.4 is the addition of labeled vault-ids.  
You can add (and in shared repositories should) a label by just using it in front of the vault-id.  

If we were to re-encrypt the testing var file of the previous example this would result in:
> test /ansible > head -n 1 inventories/testing/group_vars/all/vault
> $ANSIBLE_VAULT;1.1;AES256

> test /ansible > ansible-vault rekey --vault-id vault_pass_testing.txt   --new-vault-id=test@vault_pass_testing.txt inventories/testing/group_vars/all/vault
> Rekey successful

> test /ansible > head -n 1 inventories/testing/group_vars/all/vault
> $ANSIBLE_VAULT;1.2;AES256;test

> prod /ansible > ansible-vault rekey --vault-id vault_pass_production.txt   --new-vault-id=prod@vault_pass_production.txt inventories/production/group_vars/all/vault
> Rekey successful

As you can see with the test label, the label is clear(text)ly is put into the encrypted file, this will greatly add visibility to whom could/should be responsible for the file.


<br><br><br><a name=combo></a>
### Ultra-Mega-GPG-Vault-Id-Label-Combo!
At this point we have encrypted secrets, our passwords and vault-ids are still in plaintext, but now we can use our gpg keys to encrypt these.  
If we were to use the example of Multiple vaults in one repository, what is new in ansible 2.4 (https://github.com/brianmor/ansible-poc )  
The test user should only have access to his own group vars and not the production values, and the production user should be able to access the testing variabels as base to prepare the needed production variables.  
This means that the vault_pass_testing.txt should be gpg encrypted with both test as prod recipients, and the vault_pass_production.txt only with prod as recipient.  

Both users have exported their public gpg keys in the ansible folder as username_pub.gpg

As a prod user we would first import all needed gpg keys and encrypt our secret vault-id and then encrypt the test vault-id for both users:  
> prod ~/ansible > gpg --import /ansible/*gpg
> gpg: key 4D9F6B40B36C0F8B: "Bruce Wayne (I'm Batman) <prod@gpg.com>" not changed
> gpg: key 56DB3809C2147E28: public key "John Wayne (shoots bugs) <test@gpg.com>" imported
> gpg: Total number processed: 2
> gpg:               imported: 1
> gpg:              unchanged: 1
> prod ~/ansible > gpg --encrypt --recipient prod@gpg.com vault_pass_production.txt
> prod ~/ansible > gpg --encrypt --recipient prod@gpg.com --recipient test@gpg.com vault_pass_testing.txt
> prod ~/ansible > ls vault_pass_* -1
> vault_pass_production.txt
> vault_pass_production.txt.gpg
> vault_pass_testing.txt
> vault_pass_testing.txt.gpg

**We create the following vault decryption scripts:**  
*vault_prod.sh:*  
> \#!/bin/bash
> gpg --batch --use-agent --decrypt vault_pass_production.txt.gpg 2>/dev/null

*vault_test.sh:*  
> \#!/bin/bash
> gpg --batch --use-agent --decrypt vault_pass_testing.txt.gpg 2>/dev/null

The prod user can set an environment variable pointing to the vault ids and decrypt all files  
> prod ~/ansible > export ANSIBLE_VAULT_IDENTITY_LIST="prod@vault_prod.sh,test@vault_test.sh"
> prod ~/ansible > ansible-vault view inventories/production/group_vars/all/vault  
> \-\-\-
> 
> vault_www_db_usr : brian-prd-u
> vault_www_db_psw : qwe123
> 
> prod ~/ansible > ansible-vault view inventories/testing/group_vars/all/vault  
> \-\-\-
> 
> vault_www_db_usr : brian-tst-u
> vault_www_db_psw : 12345

remember we encrypted the testing vault file with the password in vault_pass_testing.txt, not the production one.

as a test user we can only decrypt the production gpg file, so even when we export the same list mileage will vary:

> test ~/ansible > export ANSIBLE_VAULT_IDENTITY_LIST="prod@vault_prod.sh,test@vault_test.sh"
> test ~/ansible > ansible-vault view inventories/production/group_vars/all/vault
> [WARNING]: Error in vault password file loading (prod): Vault password script ~/ansible/vault_prod.sh returned non-zero (2): None
> 
> ERROR! Vault password script ~/ansible/vault_prod.sh returned non-zero (2): None
> 
> test ~/ansible > export ANSIBLE_VAULT_IDENTITY_LIST="test@vault_test.sh"
> test ~/ansible > ansible-vault view inventories/production/group_vars/all/vault
> ERROR! Decryption failed (no vault secrets would found that could decrypt) for inventories/production/group_vars/all/vault
> test ~/ansible > ansible-vault view inventories/testing/group_vars/all/vault
> \-\-\-
> 
> vault_www_db_usr : brian-tst-u
> vault_www_db_psw : 12345

As you can see the production has access to both encrypted files, while the test user only has access to his own environment.

This can be coupled with various users, groups and offcourse the use of scripts will only help making this easily maintainable

*More reading materials:*  
\+ https://disjoint.ca/til/2016/12/14/encrypting-the-ansible-vault-passphrase-using-gpg/
\+ https://ahlers.me/blog/using-vault-to-unlock-gpg-keys/
\+ https://benincosa.com/?p=3235








