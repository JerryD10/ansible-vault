# ansible-vault

Reference: [Ansible-Vault-Docs](https://docs.ansible.com/ansible/latest/user_guide/vault.html)

Ansible Vault is a feature of ansible that allows you to keep sensitive data such as passwords or keys in encrypted files, rather than as plaintext in playbooks or roles.

## Pre-requisites 
* Clone this repository
* In order to get started, we need to set up our two VMs as given in the [CM workshop](https://github.com/CSC-DevOps/CM).

## Sample Playbook
The sample playbook consists of a few roles to install maven and git, create a directory and a file, and copy this from to the remote server. There are three variables files that are used.

To run the plays,

```
sudo ansible-playbook playbook.yml -i inventory
```

You should be able to see all tasks successfully pass.

## Vault
Now if we want to encrypt the variables files so that it is not saved as plain text, we can use `ansible-vault`.

### Encryption:
In order to encrypt the files, we can run the following command:

```
ansible-vault encrypt roles/create_files/vars/main.yml roles/copy_files/vars/main.yml variables.yml
```

You will be prompted to enter a new password and confirm it. This is the password you will use for all the further steps. This command encrypts all three files that we use, with the same password.

You can try opening the file now 

```
sudo vi variables.yml
```

You will see something like this:

```
$ANSIBLE_VAULT;1.1;AES256
32623738626666383866613861373033356331333366646334313161626530363838333164373034
6630653931336538306439356261343065386461303130330a343330626461383065663432653137
33303639633832373662333733356538643063393066386331353861386663623564613235353366
3663333532323262630a366136346637343766363165373364323834663130326633353765303661
66336463376265623963373734306333653263316137643461383762353265646564
```

### Edit:
In order to edit an encrypted file, we can run the following command:

```
ansible-vault edit variables.yml
```

This will prompt us for the password we entered while encrypting the file. Once we type the correct password, we will be allowed to edit the file in plaintext.

### Decryption:
In order to decrypt a file, we can run the following command:

```
ansible-vault decrypt variables.yml
```

This will prompt us for the password we entered while encrypting the file. Once we enter the correct password, it will convert the file back to plaintext.

You can try opening the file now:

```
sudo vi variables.yml
```

You will see something like this:

```
directory_name: 'vault-test'
```

### Running a playbook with an encrypted var file:

If we directly try to run a playbook which requires an encrypted file, we will get an error message:

```
vagrant@ubuntu-bionic:/bakerx$ sudo ansible-playbook playbook.yml -i inventory
ERROR! Attempting to decrypt but no vault secrets found
```

You can run the playbook by:

1. Prompt for an interactive password: 

    ```
    sudo ansible-playbook playbook.yml -i inventory --ask-vault-pass
    ```

    This is prompt you for the password like before.
    
    Once you enter the correct password, it will decrypt all the files that were encrypted and execute the plays.

2. Pass in a password file which contains the password stored as one line in a new file. 
    
    When using this flag, ensure permissions on the file are such that no one else can access your key
    
    First create a new file password_file. 

    ```
    sudo vi ~/password.txt
    ```

    Next add the password entered above as a single line in the file.

    Now run the command,

    ```
    sudo ansible-playbook playbook.yml -i inventory --vault-password-file ~/password.txt
    ```

### Multiple encrypted files with different passwords

Say you need to encrypt all the above files, but now with a different password for each.

First step is to decrypt all the files you encrypted above.

Next, we use the `--vault-id` to assign each file an id and a source to get the password from.

```
ansible-vault encrypt variables.yml --vault-id var@prompt
```

where `var` is any id you can set, and `prompt` can be replace by the path to a file containing a password. In case you choose prompt, you will be prompted for a password to select.

Similarly run this for the other two files too.

```
ansible-vault encrypt roles/create_files/vars/main.yml --vault-id create@prompt
ansible-vault encrypt roles/copy_files/vars/main.yml --vault-id copy@prompt  
```

Now to run the playbook, we run the following command:

```
sudo ansible-playbook playbook.yml -i inventory --vault-id copy@prompt --vault-id create@prompt --vault-id var@prompt
```

This will give us 3 prompts to enter the passwords we selected above.

```
vagrant@ubuntu-bionic:/bakerx$ sudo ansible-playbook playbook.yml -i inventory --vault-id copy@prompt --vault-id create@prompt --vault-id var@prompt
Vault password (copy):
Vault password (create):
Vault password (var):
```
Once we successfully enter the correct password, we will be able to run all the plays.