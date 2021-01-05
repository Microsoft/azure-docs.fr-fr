---
title: Chiffrer les données de déploiement
description: En savoir plus sur le chiffrement des données conservées pour les ressources de vos instances de conteneur et sur la manière de chiffrer les données avec une clé gérée par le client
ms.topic: article
ms.date: 01/17/2020
author: macolso
ms.author: macolso
ms.openlocfilehash: 1b73ce5c994231a1c7b2f26ad702f2ad5880ba44
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94686274"
---
# <a name="encrypt-deployment-data"></a>Chiffrer les données de déploiement

Lors de l’exécution de ressources Azure Container Instances (ACI) dans le cloud, le service ACI collecte et conserve les données associées à vos conteneurs. ACI chiffre automatiquement ces données lors de leur conservation dans le cloud. Ce chiffrement protège vos données et vous aide à répondre aux engagements de votre entreprise en matière de sécurité et de conformité. ACI vous donne également la possibilité de chiffrer ces données avec votre propre clé, ce qui vous permet de mieux contrôler les données relatives à vos déploiements ACI.

## <a name="about-aci-data-encryption"></a>À propos du chiffrement des données ACI 

Dans ACI, les données sont chiffrées et déchiffrées à l’aide du chiffrement AES 256 bits. Il est activé pour tous les déploiements ACI et vous n’avez pas besoin de modifier votre déploiement ou vos conteneurs pour tirer parti de ce chiffrement. Cela comprend les métadonnées sur le déploiement, les variables d’environnement, les clés passées dans vos conteneurs et les journaux conservés après l’arrêt de vos conteneurs, afin que vous puissiez toujours les voir. Le chiffrement n’a pas d’impact sur les performances de votre groupe de conteneurs et n’entraîne aucun coût supplémentaire.

## <a name="encryption-key-management"></a>Gestion des clés de chiffrement

Vous pouvez vous appuyer sur les clés gérées par Microsoft pour le chiffrement de vos données de conteneur, ou vous pouvez gérer le chiffrement avec vos propres clés. Le tableau suivant compare ces options : 

|    |    Clés managées par Microsoft     |     Clés gérées par le client     |
|----|----|----|
|    **Opérations de chiffrement/déchiffrement**    |    Azure    |    Azure    |
|    **Stockage des clés**    |    Magasin de clés Microsoft    |    Azure Key Vault    |
|    **Responsabilité de la permutation des clés**    |    Microsoft    |    Customer    |
|    **Accès aux clés**    |    Microsoft uniquement    |    Microsoft, client    |

Le reste du document décrit les étapes nécessaires pour chiffrer vos données de déploiement ACI avec votre clé (clé gérée par le client). 

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

## <a name="encrypt-data-with-a-customer-managed-key"></a>Chiffrer des données avec une clé gérée par le client

### <a name="create-service-principal-for-aci"></a>Créer un principal de service pour ACI

La première étape consiste à s’assurer que votre[client Azure](../active-directory/develop/quickstart-create-new-tenant.md) dispose d’un principal de service affecté pour accorder des autorisations au service Azure Container Instances. 

> [!IMPORTANT]
> Pour exécuter la commande suivante et créer un principal de service avec succès, vérifiez que vous disposez des autorisations nécessaires pour créer des principaux de service dans votre locataire.
>

La commande CLI suivante permet de configurer le principal de service ACI dans votre environnement Azure :

```azurecli-interactive
az ad sp create --id 6bb8e274-af5d-4df2-98a3-4fd78b4cafd9
```

La sortie de l’exécution de cette commande doit vous indiquer un principal de service qui a été configuré avec « displayName » : « Azure Container Instance Service ».

Si vous n’êtes pas en mesure de créer correctement le principal de service :
* vérifiez que vous disposez des autorisations nécessaires pour le faire dans votre locataire
* vérifiez si un principal de service existe déjà dans votre locataire pour le déploiement dans ACI. Pour ce faire, vous pouvez exécuter `az ad sp show --id 6bb8e274-af5d-4df2-98a3-4fd78b4cafd9` et utiliser ce principal de service à la place.

### <a name="create-a-key-vault-resource"></a>Créez une ressource Key Vault

Créez un coffre de clés Azure à l’aide du [portail Azure](../key-vault/general/quick-create-portal.md), d’[Azure CLI](../key-vault/general/quick-create-cli.md) ou d’[Azure PowerShell](../key-vault/general/quick-create-powershell.md).

Pour les propriétés de votre coffre de clés, respectez les recommandations suivantes : 
* Nom : un nom unique est obligatoire. 
* Abonnement : Choisissez un abonnement.
* Sous Groupe de ressources, choisissez un groupe de ressources existant ou créez et entrez un nom de groupe de ressources.
* Dans le menu déroulant Emplacement, choisissez un emplacement.
* Vous pouvez conserver les valeurs par défaut ou choisir d’autres options en fonction de spécifications supplémentaires.

> [!IMPORTANT]
> Lorsque vous utilisez des clés gérées par le client pour chiffrer un modèle de déploiement ACI, il est recommandé de définir les deux propriétés suivantes sur le coffre de clés : Suppression réversible et Ne pas vider. Ces propriétés ne sont pas activées par défaut, mais peuvent être activées à l’aide de PowerShell ou d’Azure CLI sur un coffre de clés nouveau ou existant.

### <a name="generate-a-new-key"></a>Générer une nouvelle clé 

Une fois votre coffre de clés créé, accédez à la ressource dans le Portail Azure. Dans le menu de navigation de gauche du panneau des ressources, sous Paramètres, cliquez sur **Clés**. Dans l’affichage « Clés », cliquez sur « Générer/importer » pour générer une nouvelle clé. Utilisez un nom unique pour cette clé et toute autre préférence en fonction de vos besoins. 

![Générer une nouvelle clé](./media/container-instances-encrypt-data/generate-key.png)

### <a name="set-access-policy"></a>Définir une stratégie d’accès

Créez une nouvelle stratégie d’accès pour autoriser le service ACI à accéder à votre clé.

* Une fois votre clé générée, revenez dans le panneau de vos ressources du coffre de clés et, sous Paramètres, cliquez sur **Stratégies d’accès**.
* Dans la page « Stratégies d’accès » de votre coffre de clés, cliquez sur **Ajouter une stratégie d’accès**.
* Définissez les *Autorisations de clé* pour inclure **Get** et **Unwrap Key** ![Définir des autorisations de clé](./media/container-instances-encrypt-data/set-key-permissions.png)
* Pour *Sélectionner un principal*, sélectionnez **Service Azure Container Instances**
* En bas, cliquez sur **Ajouter**. 

La stratégie d’accès doit maintenant apparaître dans les stratégies d’accès de votre coffre de clés.

![Nouvelle stratégie d’accès](./media/container-instances-encrypt-data/access-policy.png)

### <a name="modify-your-json-deployment-template"></a>Modifier votre modèle de déploiement JSON

> [!IMPORTANT]
> Le chiffrement des données de déploiement avec une clé gérée par le client est disponible dans la dernière version de l’API (2019-12-01) qui est en cours de déploiement. Spécifiez cette version d’API dans votre modèle de déploiement. Si vous rencontrez des problèmes, contactez le support Azure.

Une fois la clé du coffre de clés et la stratégie d’accès définies, ajoutez les propriétés suivantes à votre modèle de déploiement ACI. Apprenez-en davantage sur le déploiement de ressources ACI à l’aide d’un modèle dans le [Tutoriel : Déployer un groupe de plusieurs conteneurs avec un modèle Resource Manager](./container-instances-multi-container-group.md). 
* Sous `resources`, définissez `apiVersion` sur `2019-12-01`.
* Dans la section Propriétés du groupe de conteneurs du modèle de déploiement, ajoutez une propriété `encryptionProperties` contenant les valeurs suivantes :
  * `vaultBaseUrl` : nom DNS de votre coffre de clés. Vous le trouverez dans le panneau de vue d’ensemble de la ressource Coffre de clés dans le portail
  * `keyName` : nom de la clé générée précédemment
  * `keyVersion` : Version actuelle de la clé Vous la trouverez en cliquant sur la clé elle-même (sous « Clés » dans la section Paramètres de votre ressource Coffre de clés)
* Sous les propriétés du groupe de conteneurs, ajoutez une propriété `sku` avec la valeur `Standard`. La propriété `sku` est requise dans la version 2019-12-01 de l’API.

L’extrait de modèle suivant montre ces propriétés supplémentaires pour chiffrer les données de déploiement :

```json
[...]
"resources": [
    {
        "name": "[parameters('containerGroupName')]",
        "type": "Microsoft.ContainerInstance/containerGroups",
        "apiVersion": "2019-12-01",
        "location": "[resourceGroup().location]",    
        "properties": {
            "encryptionProperties": {
                "vaultBaseUrl": "https://example.vault.azure.net",
                "keyName": "acikey",
                "keyVersion": "xxxxxxxxxxxxxxxx"
            },
            "sku": "Standard",
            "containers": {
                [...]
            }
        }
    }
]
```

Voici un modèle complet, adapté à partir du modèle proposé dans [Tutoriel : Déployer un groupe de plusieurs conteneurs avec un modèle Resource Manager](./container-instances-multi-container-group.md). 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "containerGroupName": {
      "type": "string",
      "defaultValue": "myContainerGroup",
      "metadata": {
        "description": "Container Group name."
      }
    }
  },
  "variables": {
    "container1name": "aci-tutorial-app",
    "container1image": "mcr.microsoft.com/azuredocs/aci-helloworld:latest",
    "container2name": "aci-tutorial-sidecar",
    "container2image": "mcr.microsoft.com/azuredocs/aci-tutorial-sidecar"
  },
  "resources": [
    {
      "name": "[parameters('containerGroupName')]",
      "type": "Microsoft.ContainerInstance/containerGroups",
      "apiVersion": "2019-12-01",
      "location": "[resourceGroup().location]",
      "properties": {
        "encryptionProperties": {
            "vaultBaseUrl": "https://example.vault.azure.net",
            "keyName": "acikey",
            "keyVersion": "xxxxxxxxxxxxxxxx"
        },
        "sku": "Standard",  
        "containers": [
          {
            "name": "[variables('container1name')]",
            "properties": {
              "image": "[variables('container1image')]",
              "resources": {
                "requests": {
                  "cpu": 1,
                  "memoryInGb": 1.5
                }
              },
              "ports": [
                {
                  "port": 80
                },
                {
                  "port": 8080
                }
              ]
            }
          },
          {
            "name": "[variables('container2name')]",
            "properties": {
              "image": "[variables('container2image')]",
              "resources": {
                "requests": {
                  "cpu": 1,
                  "memoryInGb": 1.5
                }
              }
            }
          }
        ],
        "osType": "Linux",
        "ipAddress": {
          "type": "Public",
          "ports": [
            {
              "protocol": "tcp",
              "port": "80"
            },
            {
                "protocol": "tcp",
                "port": "8080"
            }
          ]
        }
      }
    }
  ],
  "outputs": {
    "containerIPv4Address": {
      "type": "string",
      "value": "[reference(resourceId('Microsoft.ContainerInstance/containerGroups/', parameters('containerGroupName'))).ipAddress.ip]"
    }
  }
}
```

### <a name="deploy-your-resources"></a>Déployer vos ressources

Si vous avez créé et modifié le fichier de modèle sur votre Bureau, vous pouvez le télécharger dans votre répertoire Cloud Shell en y faisant glisser le fichier. 

Créez un groupe de ressources avec la commande [az group create][az-group-create].

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Déployez ensuite le modèle avec la commande [az deployment group create][az-deployment-group-create].

```azurecli-interactive
az deployment group create --resource-group myResourceGroup --template-file deployment-template.json
```

Après quelques secondes, vous devriez recevoir une réponse initiale d’Azure. Une fois le déploiement terminé, toutes les données qui y sont associées et qui sont conservées par le service ACI sont chiffrées à l’aide de la clé que vous avez fournie.

<!-- LINKS - Internal -->
[az-group-create]: /cli/azure/group#az-group-create
[az-deployment-group-create]: /cli/azure/deployment/group/#az-deployment-group-create
