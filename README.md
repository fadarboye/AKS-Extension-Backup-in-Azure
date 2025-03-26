<a href="https://www.linkedin.com/in/adeboye-famurewa-700b9426/"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white"></a> 

![](https://img.shields.io/badge/Follow%20%ad-1.4k-blue?logo=linkedin&style=social "Follow in/Adeboye on LinkedIn") 


![Screenshot 2025-03-25 021635](https://github.com/user-attachments/assets/f06aa678-8b8e-46d2-8cff-cdfc1e355da9)


# AKS BACK-UP IN AZURE

This is a quick guides focus on a command-line/portal experience to easily and quickly run all the required steps. for backing up your AKS cluster in Azure.


## PREREQUISITES

- Before installing Backup Extension in the AKS cluster, ensure that the `CSI drivers` and `Snapshots` are Enabled for your cluster. If disabled, please enable them.


- Azure Backup for AKS supports clusters using either a `system-assigned managed identity` or a `user-assigned managed identity` for backup operations. It does not service principal.


- The Backup Extension during installation fetches Container Images stored in Microsoft Container Registry (MCR). If you enable a `firewall` on the AKS cluster, the extension installation process might fail due to access issues on the Registry.  Allow MCR access from the firewall.


- For cluster using a Private Virtual Network and Firewall, apply the FQDN/application rules


- If you have any previous installation of `Velero` in the AKS cluster, you need to delete it before installing Backup Extension.


- If you are using Azure policies in your AKS cluster, ensure that the extension namespace `dataprotection-microsoft` is excluded from these policies to allow backup and restore operations to run successfully.


- Finally, If you are using Azure network security group (NSG) to filter network traffic between Azure resources in an Azure virtual network then set an `inbound rule` to allow service tags `azurebackup` and `azurecloud`.

<br/>

#### CREATE A BACKUP VAULT

- To create the Backup vault, run the following command:

```sh
az dataprotection backup-vault create --resource-group $backupvaultresourcegroup --vault-name $backupvault --location $region --type SystemAssigned --storage-settings datastore-type="VaultStore" type="GeoRedundant“
```
<br/>

#### CREATE BACKUP POLICY

- Once the vault creation is complete, create a backup policy to protect AKS clusters

```sh
az dataprotection backup-policy get-default-policy-template --datasource-type AzureKubernetesService > akspolicy.json
```
<br/>

#### CREATE STORAGE ACCOUNT

- Extension is mandatory to be installed in the AKS cluster to perform any backup and restore operations. The Extension requires the storage account as inputs for installation.

```sh
az storage account create --name $storageaccount --resource-group $storageaccountresourcegroup --location $region --sku Standard_LRS
```


#### CREATE BLOB CONTAINER

```sh
az storage container create --name $blobcontainer --account-name $storageaccount --auth-mode login
```



<img width="573" alt="image" src="https://github.com/user-attachments/assets/af4f29eb-bf53-4d7c-94d2-5d01d704ae5b" />


<br/>

#### INSTALL BACKUP EXTENSION

```sh
az k8s-extension create --name azure-aks-backup --extension-type microsoft.dataprotection.kubernetes --scope cluster --cluster-type managedClusters --cluster-name $akscluster --resource-group $aksclusterresourcegroup --release-train stable --configuration-settings blobContainer=$blobcontainer storageAccount=$storageaccount storageAccountResourceGroup=$storageaccountresourcegroup storageAccountSubscriptionId=$subscriptionId
```

<br/>

⚠️Note : The backup vault and the AKS cluster needs to be in the same region and subscription.

<br/>


#### CREATE Storage Blob Data Contributor role 

```sh
az role assignment create --assignee-object-id $(az k8s-extension show --name azure-aks-backup --cluster-name $akscluster --resource-group $aksclusterresourcegroup --cluster-type managedClusters --query aksAssignedIdentity.principalId --output tsv) --role 'Storage Blob Data Contributor' --scope /subscriptions/$subscriptionId/resourceGroups/$storageaccountresourcegroup/providers/Microsoft.Storage/storageAccounts/$storageaccount
```

![image](https://github.com/user-attachments/assets/90394ec2-c82c-4f10-acc9-30ab93707df1)
