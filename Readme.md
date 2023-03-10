# Deploy Vaultwarden to Azure Container Apps with Azure Files storage mount


[![Deploy To Azure](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.svg?sanitize=true)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fadamhnat%2Fvaultwarden-azure%2Fmaster%2Fmain.json)
[![Visualize](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/visualizebutton.svg?sanitize=true)](http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2Fadamhnat%2Fvaultwarden-azure%2Fmaster%2Fmain.json)
 
This template provides an easy way to deploy **Vaultwarden** in **Azure Container Apps** with **Azure Files storage mount** as data storage to easily backup and restore **Vaultwarden** data.

## Deployment

1. Click `Deploy To Azure` button above and configure the following options:
   - Resource Group - you can choose an existing one or create a new one, resources created in that group.
   - Storage Account Type - default to `Standard_LRS`, refer to this documentation for the storage type [Type of storage accounts](https://learn.microsoft.com/en-us/azure/storage/common/storage-account-overview#types-of-storage-accounts)
   - AdminAPI Key - used to access the admin page at `https://DOMAIN/admin`, generated automatically. You can also generate your own key using this command `openssl rand -base64 48`.
   - Choose memory and CPU sizing - I recommend starting with 0.25 CPU and 0.5 Memory
The total CPU and memory allocations requested for all the containers in a container app must add up to one of the following combinations.

		| CPU | Memory |
		| --- | --- |
		| 0.25 | 0.5Gi |
		| 0.5 | 1.0Gi |
		| 0.75 | 1.5Gi |
		| 1.0 | 2.0Gi |
		| 1.25 | 2.5Gi |
		| 1.5 | 3.0Gi |
		| 1.75 | 3.5Gi |
		| 2.0 | 4.0Gi |

2. Click Review + create to deploy - wait until your deployment is complete.
3. Go to resource group ⇒ storage account (vwstorage+randomcharacter) ⇒ File shares ⇒ vw-data ⇒ click Upload ⇒ choose `db.sqlite3` file and make sure `Overwrite if files already exist` option is checked ⇒ click Upload.
4. Go to resource group ⇒ vaultwarden ⇒ Revision management ⇒ container name ⇒ Restart
5. Click Overview ⇒ click Application Url to open the application

### Note
> In case you get the following error message on your deployment
> 
> ```
> Resource vaultwarden Microsoft.App/containerApps failed
> ```
> 
> Click **Redeploy** button and use the same configuration as before.
> 
> The error seems to happen when Azure provisions the container and mount the storage share but the storage account creation is not finished yet.


## Updating to a new version

in Azure Portal:

Go to Resource Group ⇒ vaultwarden ⇒ Revision management ⇒ **Create new revision** ⇒ type name/suffix ⇒ check vaultwarden in Container image section ⇒ **Create**

This will update the **Vaultwarden** container app to the most recent version while keeping data in place and no downtime.

## Login to admin panel

1. Go to Resource Group ⇒ vaultwarden ⇒ Containers ⇒ Environment Variables ⇒ copy ADMIN_TOKEN **value**
2. Go to `https://DOMAIN/admin` and paste the ADMIN_TOKEN value
 
## Restoring backup to Azure Container Apps

Upload the backup data to vw-data File share in Azure Storage account

    attachments/
    icon_cache/
    sends/
    tmp/
    config.json
    db.sqlite3
    db.sqlite3-shm
    db.sqlite3-wal
    rsa_key.pem
    rsa_key.pub.pem

In case you got database is locked [issue](https://github.com/adamhnat/vaultwarden-azure/issues/1)

Run the following command before uploading `db.sqlite3`

```sql
sqlite3 db.sqlite3 'PRAGMA journal_mode=wal;'
```