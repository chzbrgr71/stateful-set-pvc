## Azure Files with StatefulSet

> Reference: Manually create and use a volume with Azure Files share in Azure Kubernetes Service (AKS): https://docs.microsoft.com/en-us/azure/aks/azure-files-volume 

### Set variables
```
export AKS_PERS_STORAGE_ACCOUNT_NAME=briartesting1
export AKS_PERS_RESOURCE_GROUP=sf-testing
export AKS_PERS_LOCATION=eastus
export AKS_PERS_SHARE_NAME=myshare
```

### Create the storage account
```
az storage account create -n $AKS_PERS_STORAGE_ACCOUNT_NAME -g $AKS_PERS_RESOURCE_GROUP -l $AKS_PERS_LOCATION --sku Standard_LRS
```

### Export the connection string as an environment variable, this is used when creating the Azure file share
```
export AZURE_STORAGE_CONNECTION_STRING=`az storage account show-connection-string -n $AKS_PERS_STORAGE_ACCOUNT_NAME -g $AKS_PERS_RESOURCE_GROUP -o tsv`
```

### Create the file share
```
az storage share create -n $AKS_PERS_SHARE_NAME
```

### Get storage account key
```
STORAGE_KEY=$(az storage account keys list --resource-group $AKS_PERS_RESOURCE_GROUP --account-name $AKS_PERS_STORAGE_ACCOUNT_NAME --query "[0].value" -o tsv)
```

### Echo storage account name and key
```
echo Storage account name: $AKS_PERS_STORAGE_ACCOUNT_NAME
echo Storage account key: $STORAGE_KEY

kubectl create secret generic azure-file-secret --from-literal=azurestorageaccountname=$AKS_PERS_STORAGE_ACCOUNT_NAME --from-literal=azurestorageaccountkey=$STORAGE_KEY
```

### Create persistent volume
```
kubectl apply -f pv.yaml
```

### Create PVC
```
kubectl apply -f pvc.yaml
```

### Create StatefulSet
```
kubectl apply -f stateful-set-static.yaml
```