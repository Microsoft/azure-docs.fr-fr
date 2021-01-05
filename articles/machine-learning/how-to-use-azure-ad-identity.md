---
title: Utiliser l’identité Azure AD avec votre service web
titleSuffix: Azure Machine Learning
description: Utilisez l’identité Azure AD avec votre service web dans Azure Kubernetes Service pour accéder à des ressources cloud au cours d’un scoring.
services: machine-learning
ms.author: larryfr
author: BlackMist
ms.reviewer: aashishb
ms.service: machine-learning
ms.subservice: core
ms.date: 11/16/2020
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: bcf5f75cf5cabe42f530a6a179c2cafd43b5520d
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94952537"
---
# <a name="use-azure-ad-identity-with-your-machine-learning-web-service-in-azure-kubernetes-service"></a>Utiliser Azure AD Identity avec votre service web de machine learning dans Azure Kubernetes Service

Dans cette rubrique de guide pratique, vous allez apprendre à attribuer une identité Azure Active Directory (Azure AD) à votre modèle Machine Learning déployé dans Azure Kubernetes Service. Le projet [Azure AD Pod Identity](https://github.com/Azure/aad-pod-identity) permet aux applications d’accéder de manière sécurisée à des ressources cloud avec Azure AD à l’aide d’une [identité managée](../active-directory/managed-identities-azure-resources/overview.md) et de primitives Kubernetes. Votre service web peut ainsi accéder de manière sécurisée à vos ressources Azure sans avoir à incorporer des informations d’identification ou à gérer des jetons directement à l’intérieur de votre script `score.py`. Cet article décrit les étapes nécessaires à la création et à l’installation d’une identité Azure dans votre cluster Azure Kubernetes Service, ainsi qu’à l’attribution de l’identité à votre service web déployé.

## <a name="prerequisites"></a>Prérequis

- L’[extension Azure CLI pour Machine Learning service](reference-azure-machine-learning-cli.md), le [kit SDK Azure Machine Learning pour Python](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py) ou l’[extension Azure Machine Learning pour Visual Studio Code](tutorial-setup-vscode-extension.md).

- Un accès à votre cluster AKS à l’aide de la commande `kubectl`. Pour plus d’informations, consultez [Se connecter au cluster](../aks/kubernetes-walkthrough.md#connect-to-the-cluster).

- Un service web Azure Machine Learning déployé sur votre cluster AKS.

## <a name="create-and-install-an-azure-identity"></a>Créer et installer une identité Azure

1. Pour déterminer si le contrôle d’accès en fonction du rôle (RBAC) Kubernetes est activé dans votre cluster AKS, utilisez la commande suivante :

    ```azurecli-interactive
    az aks show --name <AKS cluster name> --resource-group <resource group name> --subscription <subscription id> --query enableRbac
    ```

    Cette commande retourne la valeur `true` si le contrôle Kubernetes RBAC est activé. Cette valeur détermine la commande à utiliser à l’étape suivante.

1. Installez [Azure AD Pod Identity](https://azure.github.io/aad-pod-identity/docs/getting-started/installation/) dans votre cluster AKS.

1. [Créez une identité sur Azure](https://azure.github.io/aad-pod-identity/docs/demo/standard_walkthrough/#2-create-an-identity-on-azure) en suivant les étapes indiquées dans la page du projet Azure AD Pod Identity.

1. [Déployez AzureIdentity](https://azure.github.io/aad-pod-identity/docs/demo/standard_walkthrough/#3-deploy-azureidentity) en suivant les étapes indiquées dans la page du projet Azure AD Pod Identity.

1. [Déployez AzureIdentityBinding](https://azure.github.io/aad-pod-identity/docs/demo/standard_walkthrough/#5-deploy-azureidentitybinding) en suivant les étapes indiquées dans la page du projet Azure AD Pod Identity.

1. Si l’identité Azure créée à l’étape précédente ne se trouve pas dans le même groupe de ressources de nœud que votre cluster AKS, effectuez la procédure décrite dans [Attribution de rôle](https://azure.github.io/aad-pod-identity/docs/getting-started/role-assignment/#user-assigned-identities-that-are-not-within-the-node-resource-group) en suivant les étapes indiquées dans la page du projet Azure AD Pod Identity.

## <a name="assign-azure-identity-to-web-service"></a>Affecter une identité Azure au service web

Les étapes suivantes utilisent l’identité Azure créée dans la section précédente, puis affectez-la à votre service web AKS par le biais d’une **étiquette de sélecteur**.

Commencez par identifier le nom et l’espace de noms de votre déploiement dans votre cluster AKS auquel vous voulez affecter l’identité Azure. Vous pouvez obtenir ces informations en exécutant la commande suivante. Les espaces de noms doivent être le nom de votre espace de travail Azure Machine Learning et le nom de votre déploiement doit être le nom de votre point de terminaison, tel qu’indiqué dans le portail.

```azurecli-interactive
kubectl get deployment --selector=isazuremlapp=true --all-namespaces --show-labels
```

Ajoutez l’étiquette du sélecteur d’identité Azure à votre déploiement en modifiant la spécification du déploiement. La valeur du sélecteur doit être celle que vous avez définie à l’étape 5 de la section [Déployer AzureIdentityBinding](https://azure.github.io/aad-pod-identity/docs/demo/standard_walkthrough/#5-deploy-azureidentitybinding).

```yaml
apiVersion: "aadpodidentity.k8s.io/v1"
kind: AzureIdentityBinding
metadata:
  name: demo1-azure-identity-binding
spec:
  AzureIdentity: <a-idname>
  Selector: <label value to match>
```

Modifiez le déploiement pour ajouter l’étiquette du sélecteur d’identité Azure. Accédez à la section suivante sous `/spec/template/metadata/labels`. Vous devez normalement voir des valeurs telles que `isazuremlapp: “true”`. Ajoutez l’étiquette aad-pod-identity comme indiqué ci-dessous.

```azurecli-interactive
    kubectl edit deployment/<name of deployment> -n azureml-<name of workspace>
```

```yaml
spec:
  template:
    metadata:
      labels:
       aadpodidbinding: "<value of Selector in AzureIdentityBinding>"
      ...
```

Pour vérifier que l’étiquette a été correctement ajoutée, exécutez la commande suivante. Vous devez également voir les statuts des pods nouvellement créés.

```azurecli-interactive
   kubectl get pod -n azureml-<name of workspace> --show-labels
```

Une fois que les pods sont opérationnels, les services web pour ce déploiement peuvent alors accéder à des ressources Azure par le biais de votre identité Azure sans avoir à incorporer les informations d’identification dans votre code.

## <a name="assign-roles-to-your-azure-identity"></a>Affecter les rôles à votre identité Azure

[Affectez votre identité managée Azure avec les rôles appropriés](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md) pour accéder à d’autres ressources Azure. Vérifiez que les rôles que vous affectez ont les **actions de données** appropriées. Par exemple, le [rôle de lecteur des données Blob du stockage](../role-based-access-control/built-in-roles.md#storage-blob-data-reader) aura des autorisations de lecture sur votre objet blob de stockage, alors que le [rôle de lecteur](../role-based-access-control/built-in-roles.md#reader) générique n’en aura probablement pas.

## <a name="use-azure-identity-with-your-web-service"></a>Utiliser l’identité Azure avec votre service web

Déployez un modèle sur votre cluster AKS. Le script `score.py` peut contenir des opérations pointant vers les ressources Azure auxquelles votre identité Azure a accès. Vérifiez que vous avez installé vos dépendances de bibliothèque de client nécessaires pour la ressource à laquelle vous tentez d’accéder. Voici quelques exemples de la façon dont vous pouvez utiliser votre identité Azure pour accéder à différentes ressources Azure à partir de votre service.

### <a name="access-key-vault-from-your-web-service"></a>Accéder à un coffre de clés à partir de votre service web

Si vous avez donné à votre identité Azure un accès en lecture à un secret à l’intérieur d’un **coffre de clés**, votre `score.py` peut y accéder à l’aide du code suivant.

```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

my_vault_name = "yourkeyvaultname"
my_vault_url = "https://{}.vault.azure.net/".format(my_vault_name)
my_secret_name = "sample-secret"

# This will use your Azure Managed Identity
credential = DefaultAzureCredential()
secret_client = SecretClient(
    vault_url=my_vault_url,
    credential=credential)
secret = secret_client.get_secret(my_secret_name)
```

> [!IMPORTANT]
> Cet exemple utilise DefaultAzureCredential. Pour accorder l’accès à votre identité à l’aide d’une stratégie d’accès spécifique, consultez [Affecter une stratégie d’accès Key Vault à l’aide de l’interface de ligne de commande Azure](../key-vault/general/assign-access-policy-cli.md).

### <a name="access-blob-from-your-web-service"></a>Accéder à un objet blob à partir de votre service web

Si vous avez donné à votre identité Azure un accès en lecture à des données à l’intérieur d’un **objet blob de stockage**, votre `score.py` peut y accéder à l’aide du code suivant.

```python
from azure.identity import DefaultAzureCredential
from azure.storage.blob import BlobServiceClient

my_storage_account_name = "yourstorageaccountname"
my_storage_account_url = "https://{}.blob.core.windows.net/".format(my_storage_account_name)

# This will use your Azure Managed Identity
credential = DefaultAzureCredential()
blob_service_client = BlobServiceClient(
    account_url=my_storage_account_url,
    credential=credential
)
blob_client = blob_service_client.get_blob_client(container="some-container", blob="some_text.txt")
blob_data = blob_client.download_blob()
blob_data.readall()
```

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur l’utilisation de la bibliothèque de client Azure Identity Python, consultez le [dépôt](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/identity/azure-identity#azure-identity-client-library-for-python) sur GitHub.
* Pour obtenir un guide détaillé sur le déploiement de modèles sur des clusters Azure Kubernetes Service, consultez le [guide pratique](how-to-deploy-azure-kubernetes-service.md).