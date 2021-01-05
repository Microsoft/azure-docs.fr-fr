---
title: Stocker en toute sécurité les informations d’identification d’accès
titleSuffix: Azure Data Science Virtual Machine
description: Découvrez comment stocker des informations d'identification d’accès en toute sécurité sur les machines virtuelles de science des données (DSVM). Vous allez apprendre à utiliser les identités de service géré et Azure Key Vault pour stocker les informations d’identification d’accès.
keywords: formation approfondie, IA, outils de science des données, machine virtuelle de science des données, analyse géospatiale, processus de science des données en équipe
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
author: vijetajo
ms.author: vijetaj
ms.topic: conceptual
ms.date: 05/08/2018
ms.openlocfilehash: d5604e42c2c27463e10c136ccd18c3c21846fc5a
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93309150"
---
# <a name="store-access-credentials-securely-on-an-azure-data-science-virtual-machine"></a>Stocker des informations d’identification d’accès en toute sécurité sur une machine virtuelle de science des données (DSVM) Azure

Il est courant que le code des applications cloud contienne des informations d’identification pour l’authentification auprès des services cloud. La gestion et la sécurisation de ces informations d’identification est un défi bien connu dans la création d’applications cloud. Dans l’idéal, les informations d’identification ne doivent jamais s’afficher sur les stations de travail de développement ou ne sont jamais archivées dans le contrôle de code source.

La fonctionnalité des [identités managées pour les ressources Azure](../../active-directory/managed-identities-azure-resources/overview.md) simplifie la résolution de ce problème en fournissant aux services Azure une identité managée automatiquement dans Azure AD (Azure Active Directory). Vous pouvez utiliser cette identité pour vous authentifier sur n’importe quel service prenant en charge l’authentification Azure AD, sans avoir d’informations d’identification dans votre code.

Un moyen de sécuriser des informations d’identification consiste à utiliser Windows Installer (MSI) en combinaison avec [Azure Key Vault](../../key-vault/index.yml), un service Azure managé permettant de stocker les clés de chiffrement et les secrets en toute sécurité. Vous pouvez accéder à un coffre de clés à l’aide de l’identité managée et y récupérer les secrets et les clés de chiffrement autorisés.

La documentation sur les identités managées pour les ressources Azure et Key Vault contient des informations complètes et détaillées sur ces services. Le reste de cet article décrit l’utilisation de base de MSI et de Key Vault sur Data Science Virtual Machine (DSVM) pour accéder à des ressources Azure. 

## <a name="create-a-managed-identity-on-the-dsvm"></a>Créer une identité gérée sur la machine virtuelle DSVM

```azurecli-interactive
# Prerequisite: You have already created a Data Science VM in the usual way.

# Create an identity principal for the VM.
az vm assign-identity -g <Resource Group Name> -n <Name of the VM>
# Get the principal ID of the DSVM.
az resource list -n <Name of the VM> --query [*].identity.principalId --out tsv
```

## <a name="assign-key-vault-access-permissions-to-a-vm-principal"></a>Assigner des autorisations d’accès à Key Vault à un principal de machine virtuelle

```azurecli-interactive
# Prerequisite: You have already created an empty Key Vault resource on Azure by using the Azure portal or Azure CLI.

# Assign only get and set permissions but not the capability to list the keys.
az keyvault set-policy --object-id <Principal ID of the DSVM from previous step> --name <Key Vault Name> -g <Resource Group of Key Vault>  --secret-permissions get set
```

## <a name="access-a-secret-in-the-key-vault-from-the-dsvm"></a>Accéder à un secret dans le coffre de clés à partir de la machine virtuelle DSVM

```bash
# Get the access token for the VM.
x=`curl http://localhost:50342/oauth2/token --data "resource=https://vault.azure.net" -H Metadata:true`
token=`echo $x | python -c "import sys, json; print(json.load(sys.stdin)['access_token'])"`

# Access the key vault by using the access token.
curl https://<Vault Name>.vault.azure.net/secrets/SQLPasswd?api-version=2016-10-01 -H "Authorization: Bearer $token"
```

## <a name="access-storage-keys-from-the-dsvm"></a>Accéder aux clés de stockage à partir de la machine virtuelle DSVM

```bash
# Prerequisite: You have granted your VMs MSI access to use storage account access keys based on instructions at https://docs.microsoft.com/azure/active-directory/managed-service-identity/tutorial-linux-vm-access-storage. This article describes the process in more detail.

y=`curl http://localhost:50342/oauth2/token --data "resource=https://management.azure.com/" -H Metadata:true`
ytoken=`echo $y | python -c "import sys, json; print(json.load(sys.stdin)['access_token'])"`
curl https://management.azure.com/subscriptions/<SubscriptionID>/resourceGroups/<ResourceGroup of Storage account>/providers/Microsoft.Storage/storageAccounts/<Storage Account Name>/listKeys?api-version=2016-12-01 --request POST -d "" -H "Authorization: Bearer $ytoken"

# Now you can access the data in the storage account from the retrieved storage account keys.
```

## <a name="access-the-key-vault-from-python"></a>Accéder au coffre de clés à partir de Python

```python
from azure.keyvault import KeyVaultClient
from msrestazure.azure_active_directory import MSIAuthentication

"""MSI Authentication example."""

# Get credentials.
credentials = MSIAuthentication(
    resource='https://vault.azure.net'
)

# Create a Key Vault client.
key_vault_client = KeyVaultClient(
    credentials
)

key_vault_uri = "https://<key Vault Name>.vault.azure.net/"

secret = key_vault_client.get_secret(
    key_vault_uri,  # Your key vault URL.
    # The name of your secret that already exists in the key vault.
    "SQLPasswd",
    ""              # The version of the secret; empty string for latest.
)
print("My secret value is {}".format(secret.value))
```

## <a name="access-the-key-vault-from-azure-cli"></a>Accéder au coffre de clés à partir d’Azure CLI

```azurecli-interactive
# With managed identities for Azure resources set up on the DSVM, users on the DSVM can use Azure CLI to perform the authorized functions. The following commands enable access to the key vault from Azure CLI without requiring login to an Azure account.
# Prerequisites: MSI is already set up on the DSVM as indicated earlier. Specific permissions, like accessing storage account keys, reading specific secrets, and writing new secrets, are provided to the MSI.

# Authenticate to Azure CLI without requiring an Azure account. 
az login --msi

# Retrieve a secret from the key vault. 
az keyvault secret show --vault-name <Vault Name> --name SQLPasswd

# Create a new secret in the key vault.
az keyvault secret set --name MySecret --vault-name <Vault Name> --value "Helloworld"

# List access keys for the storage account.
az storage account keys list -g <Storage Account Resource Group> -n <Storage Account Name>
```