---
auto_validation: true
time: 10
tags: [ tutorial>beginner, products>sap-hana-cloud, products>sap-adaptive-server-enterprise, software-product-function>sap-hana-cloud\,-sap-adaptive-server-enterprise]
primary_tag: products>sap-hana-cloud
parser: v2
---

# Migrate SAP ASE Database from On-Premise to SAP HANA Cloud

<!-- description --> Description

## Prerequisites
- You have completed the [previous tutorial](hana-cloud-ase-migration-2) on how to encrypt your SAP ASE database to migrate from on-premise to SAP HANA Cloud.

## You will learn
- How to create an SAP HANA Cloud, SAP ASE database instance
- How to copy the encrypted backup to MS Azure using **`azcopy`**
- How to load the encrypted backup

## Intro

Migrating an SAP ASE database from on-premise to the cloud requires a bit of preparation and a few important steps. You can look at each of them in more detail, but these are the high-level steps required:

1.	Pre-requisites for migration
2.	Pre-Migration - Encrypt the on-premise SAP ASE database
3.	Migration - Create the SAP HANA Cloud, SAP ASE database
4.	Migration - Copy the Encrypted Backup to MS Azure
5.	Migration - Load the Encrypted Backup

This tutorial will cover the steps (3 to 5) involving the migration. You can learn about the [first](hana-cloud-ase-migration-1) and the [second](hana-cloud-ase-migration-2) step by referring to the previous tutorials.

---

### Create the SAP HANA Cloud, SAP ASE database instance

<!--border-->    ![Add unimplemented methods in ABAP Development Tools](add-unimplemented-methods.png)

Now it's time to make sure your SAP ASE database in SAP HANA Cloud is ready to receive the data from your on-premise SAP ASE database. To ensure this, follow these steps:

1.	Provision an SAP HANA Cloud, SAP ASE database instance. The database in SAP HANA Cloud should be at least the same size as the on-premise one, but you might find that you need to increase the size as you go through the migration and performance test. Find more information on our technical documentation.

2.	Configure your SAP ASE database in SAP HANA Cloud. You can find details of the various operational steps to back up the database, SAP ASE configuration or list out the login accounts, roles, and database cache bindings on the SAP HANA Cloud, SAP ASE Migration Guide.

3.	Make sure the SAP HANA Cloud, SAP ASE master database is encrypted with a password and not using an external HSM.

    ```Shell/Bash
    sp_encryption helpkey, sybencrmasterkey
    ```
    If it was not encrypted with a password, then add a password for the master key.

    ```Shell/Bash
alter encryption key master with external key
modify encryption with passwd "<PASSWORD>"
```
4.	Extract the database-encryption-key from the on-premise SAP ASE database.  There is a `dek_mig_gen.py` key migration script to automatically build the create encrypt key command.  Here is the syntax to use with this script in order to get the database-encryption-key:

    ```Shell/Bash
export SYBROOT=/sap/ASE16sp04GA/sap/python_client/testcode/dek_mig_gen.py -U sa -P <PASSWORD> -S <OnPremiseASE> -K <KEY_NAME> -M <PASSWORD>
```
    Optionally, you can manually build the database-encryption-key from the on-premise SAP ASE database using `ddlgen`. Instructions for building out the database-encryption-key using `ddlgen` can be found in the SAP HANA Cloud, SAP ASE Migration Guide.

    ```Shell/Bash
ddlgen -Usa -P<PASSWORD> -SASE16sp04GA -Dmaster -TEK -XOD -N %
```

5.	Create the **encrypted** database in the SAP HANA Cloud, SAP ASE. Execute the command previously created as part of the previous step (step 3) to create the database encryption key in the SAP HANA Cloud, SAP ASE.

6.	Create the database using the migrated DEK.

    > **CAUTION**
    >
    > It is very important to create the database **with the encrypted key**. If you don't, the database will still be created but you won't be able to load the database later.

    ```Shell/Bash
    create database <DATABASE_NAME> on datadev11="1G" log on logdev11="50M" for load encrypt with <KEY_NAME>
    ```


### Step 2 - Copy the encrypted backup to MS Azure using azcopy

![Add unimplemented methods in ABAP Development Tools](add-unimplemented-methods.png)

You can see a migration demo video here, and then check out the detailed steps:

<iframe width="560" height="315" src="https://urldefense.com/v3/__https://www.youtube.com/embed/zNAfk9Wt0Qo__;!!GF_29dbcQIUBPA!lv2WWH1SnniEy-6iRkEYgH4p1xWzLVRHZNEAaJHWGhRvASDV51lH3hBc648fq8crMnioQw$ [youtube[.]com]" frameborder="0" allowfullscreen></iframe>

&nbsp;

1.	If you don't already have an Azure account, you can go to [this website](https://azure.microsoft.com/en-us/free/) and create one.  Once you have logged into the Azure portal, proceed to create an Azure Storage Account of type `blogstorage`.

2.	Install the `azcopy` tool using the help of [this link](https://urldefense.com/v3/__https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=dnf__;!!GF_29dbcQIUBPA!lv2WWH1SnniEy-6iRkEYgH4p1xWzLVRHZNEAaJHWGhRvASDV51lH3hBc648fq8dXIXBgWw$ [docs[.]microsoft[.]com]).

    Then run this command:
    ```Shell/Bash
rpm --import https://urldefense.com/v3/__https://packages.microsoft.com/keys/microsoft.asc__;!!GF_29dbcQIUBPA!lv2WWH1SnniEy-6iRkEYgH4p1xWzLVRHZNEAaJHWGhRvASDV51lH3hBc648fq8dluBbZyQ$ [packages[.]microsoft[.]com]
```

3.	Once you have `azcopy` loaded on your system, you will need to log in into your azure account with `azcopy`. Below is the command that you use to do this:

    ```Shell/Bash
/docker/toolkit/microsoft/azcopy_linux_amd64_10.9.0/azcopy login --tenant-id "XXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"
```
    Replace the `tenant-id` above with the `tenant-id` of your MS Azure account.  After you have executed the `azcopy login` command above, you will be asked to open a browser and type in a code that was provided to allow the access.

4.	Next, create a container using the `azcopy` tool.

    ```Shell/Bash
/docker/toolkit/microsoft/azcopy_linux_amd64_10.9.0/azcopy make "https://urldefense.com/v3/__https://salsestorage01.blob.core.windows.net/__;!!GF_29dbcQIUBPA!lv2WWH1SnniEy-6iRkEYgH4p1xWzLVRHZNEAaJHWGhRvASDV51lH3hBc648fq8c9Rpd6Wg$ [salsestorage01[.]blob[.]core[.]windows[.]net]<DATABASE_NAME>container"
```
5.	Using the MS Azure portal, generate a shared access signature (SAS) and assign to signing key 1 (access key).

6.	Finally, upload the on-premise encrypted backup to the azure storage account using `azcopy`. Be sure to select the database directory name as the root of the directory tree that you want uploaded.

    ```Shell/Bash
/docker/toolkit/microsoft/azcopy_linux_amd64_10.9.0/azcopy copy '<DATABASE_NAME>_dumpdir/<DATABASE_NAME>' 'https://urldefense.com/v3/__https://salsestorage01.blob.core.windows.net/__;!!GF_29dbcQIUBPA!lv2WWH1SnniEy-6iRkEYgH4p1xWzLVRHZNEAaJHWGhRvASDV51lH3hBc648fq8c9Rpd6Wg$ [salsestorage01[.]blob[.]core[.]windows[.]net]<DATABASE_NAME>container?sp=racwdl&st=2021-02-28T14:19:34Z&se=2022-03-01T22:19:34Z&spr=https&sv=2020-02-10&sr=c&sig=yKCnItSYnfkIb%2BqFzTu7iiP3Saso2zPqY%2F1QtgiEpH8%3D' --recursive
```

Replace the full https parameter in the `azcopy` command above with your shared access signature (SAS) created in step 5.


---