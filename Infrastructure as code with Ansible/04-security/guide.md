# Ansible Lab 4 - Security

**Security should be the first or one of the first few things you consider. Not in step(Lab) #4.**

In this section you will:

- Create a secret in an existing Azure Key Vault.
- Modify your playbook so that you reference the secret in your playbook.

## Before you start

From Lab 4 onwards, you will work with multiple files. 

Running playbooks in Cloud Shell presents a couple of limitations:

- you can only upload file to Cloud Shell one by one
- after uploading to Cloud Shell, you need to perform an additional step to move files uploaded to the right location.

It can become cumbersome to continue running your playbooks in Cloud Shell.

The VS Code Ansible extension provides better integration experience when you need to work with multiple files so we recommend running Ansible in the remote host via SSH. 

When running your playbook remotely via SSH, you will be prompted if you want to always save your workspace in the remote host. 

You can also hit `F1` to explicitly copy files to remote host:

1. Hit `F1`
1. Type/select **Ansible: copy folder to Remote Host**
1. Follow the prompt to first provide the source directory
1. If this is the first time:
    -  set up a new host by selecting **Add new host**. Provide a name. e.g., the Ansible DNS Name in `Environment Details` tab of this workshop
    - For SSH port, enter 22
    - Provide **ansible Admin Username** as the SSH username and ansible Admin Password as SSH password 
1. If you have already set up the remote host, you will find the SSH host in the list for selection.
1. Next, specify the target folder. You can keep it the same as your local directory.

Run your playbook by right clicking your .yml file and select **Run playbook remotely via SSH**.

## Add a secret to Azure Key Vault

Azure Key Vault is a tool for securely storing and accessing secrets. We have pre-provisioned an Azure Key Vault for you in the first resource group, i.e., `IoC-01-XXXXXX`. The name should be `keyvaultXXXXXX` where `XXXXXX` is the last 6 digits of your first resource group.
1. Create two verible one for keyvault_name and second for secret_name.
1. Use [azure_rm_keyvault](https://docs.ansible.com/ansible/latest/modules/azure_rm_keyvault_module.html) to create an Azure Key Vault. For the purpose of this lab, we have pre-provisioned an Azure Key Vault for each lab participant.
1. Use [azure_rm_keyvaultsecret](https://docs.ansible.com/ansible/latest/modules/azure_rm_keyvaultsecret_module.html) to add a secret

#### Cheat Sheet: create secret
<details>
<summary>
Expand to see how you can create a key vault secret
</summary>

```yml
  - name: create a Key Vault secret
    azure_rm_keyvaultsecret:
      keyvault_uri: "https://{{ keyvault_name }}.vault.azure.net"
      secret_name: "{{ secret_name }}"
      secret_value: "Password1234!"
```

</details>

## Replace hard coded password in previous lab

Next, instead of hard coding the password and exposing a major security risk when provisioning the VM, you can now:
- In the Azure Portal select the Properties tab for the Azure KeyVault
- Find the resourceId in the Properties window, and click the copy icon
- Create a varible "admin_password" paste the resourceId as a value.

### Cheat Sheet: pass secret retrieved from Key Vault to the next task
<details>
<summary>
Expand to see how you can pass secret retrieved from Key Vault to the next task
</summary>

```yml
  - name: Get latest version of a secret
    azure_rm_keyvaultsecret_info:
      vault_uri: "https://{{ keyvault_name }}.vault.azure.net"
      name: "{{ secret_name }}"
    register: output

  - name: Create a virtual machines
    azure_rm_virtualmachine:
      resource_group: "{{ myResource_group }}"
      name: "{{ myVM }}"
      admin_username: "testadmin"
      admin_password: " {{ adminPassword }}"
      vm_size: Standard_B1ms
      network_interfaces: "{{ myNIC }}"
      image:
        offer: UbuntuServer
        publisher: Canonical
        sku: 16.04-LTS
        version: latest
```

</details>

> **CODE:** To view all of the completed codes, go to [lab4.yml](Code/lab4.yml).  

## Further readings

This [sample playbook](https://github.com/Azure-Samples/ansible-playbooks/blob/master/keyvault_create.yml) for a good walk-through of creating an Azure Key Vault.
