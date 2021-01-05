---
title: Activer l’identité managée dans le groupe de conteneurs
description: Découvrez comment utiliser une identité managée pour s’authentifier auprès d’autres services Azure dans Azure Container Instances
ms.topic: article
ms.date: 07/02/2020
ms.openlocfilehash: 67ef17b77a9db92e539dd860a3083760fe1160db
ms.sourcegitcommit: 65a4f2a297639811426a4f27c918ac8b10750d81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96558944"
---
# <a name="how-to-use-managed-identities-with-azure-container-instances"></a>Comment utiliser une identité managée avec Azure Container Instances

Utilisez les [identités managées pour les ressources Azure](../active-directory/managed-identities-azure-resources/overview.md) pour exécuter du code dans Azure Container Instances interagissant avec d’autres services Azure, et ce sans avoir à gérer les codes secrets ou les informations d’identification dans le code. Cette fonctionnalité fournit un déploiement Azure Container Instances avec une identité managée automatiquement dans Azure Active Directory.

Dans cet article, vous en apprendrez davantage sur les identités managées dans Azure Container Instances, et vous découvrirez comment :

> [!div class="checklist"]
> * Activer une identité attribuée par l’utilisateur ou par le système dans un groupe de conteneurs
> * Autoriser l’identité à accéder à un coffre de clés Azure
> * Utiliser l’identité managée pour accéder à un coffre de clés à partir d’un conteneur en cours d’exécution

Adaptez les exemples pour activer et utiliser des identités dans Azure Container Instances pour accéder aux autres services Azure. Ces exemples sont interactifs. Toutefois, en pratique, vos images de conteneur exécuteraient du code pour accéder aux services Azure.
 
> [!IMPORTANT]
> Actuellement, cette fonctionnalité est uniquement disponible en tant que version préliminaire. Les préversions sont à votre disposition, à condition que vous acceptiez les [conditions d’utilisation supplémentaires](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Certains aspects de cette fonctionnalité sont susceptibles d’être modifiés avant la mise à disposition générale. Actuellement, les identités managées sur Azure Container Instances sont uniquement prises en charge avec des conteneurs Linux, mais pas encore avec des conteneurs Windows.

## <a name="why-use-a-managed-identity"></a>Pourquoi utiliser une identité managée ?

Utilisez une identité managée dans un conteneur en cours d’exécution pour vous authentifier sur n’importe quel [service prenant en charge l’authentification Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication), sans avoir à gérer les informations d’identification dans le code de votre conteneur. Pour les services qui ne prennent pas en charge l’authentification AD, vous pouvez stocker des codes secrets dans un coffre de clés Azure et utiliser l’identité managée pour accéder à celui-ci afin de récupérer les informations d’identification. Pour en savoir plus sur l’utilisation des identités managées, consultez la section [Que sont les identités managées pour les ressources Azure ?](../active-directory/managed-identities-azure-resources/overview.md)

### <a name="enable-a-managed-identity"></a>Activer une identité managée

 Lorsque vous créez un groupe de conteneurs, activez une ou plusieurs identités managées en définissant une propriété [ContainerGroupIdentity](/rest/api/container-instances/containergroups/createorupdate#containergroupidentity). Vous pouvez également activer ou mettre à jour des identités managées après l’exécution d’un groupe de conteneurs. Les deux actions entraînent le redémarrage du groupe de conteneurs. Pour définir les identités sur un nouveau groupe de conteneurs ou sur un groupe existant, utilisez Azure CLI, un modèle Resource Manager, un fichier YAML ou un autre outil Azure. 

Azure Container Instances prend en charge les deux types d’identités Azure managées : attribuées par l’utilisateur et attribuées par le système. Sur un groupe de conteneurs, vous pouvez activer une identité attribuée par le système, une ou plusieurs identités attribuées par l’utilisateur, ou les deux types d’identités. Si vous n’êtes pas familiarisé avec les identités managées pour les ressources Azure, consultez la [présentation](../active-directory/managed-identities-azure-resources/overview.md).

### <a name="use-a-managed-identity"></a>Utiliser une identité managée

Pour utiliser une identité managée, l’identité doit être autorisée à accéder à une ou plusieurs ressources de service Azure (par exemple, une application web, un coffre de clés ou un compte de stockage) dans l’abonnement. L’utilisation d’une identité managée dans un conteneur en cours d’exécution est semblable à l’utilisation d’une identité dans une machine virtuelle Azure. Consultez l’aide relative aux machines virtuelles pour en savoir plus sur l’utilisation d’un [jeton](../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md), d’[Azure PowerShell ou Azure CLI](../active-directory/managed-identities-azure-resources/how-to-use-vm-sign-in.md), ou des [kits de développement logiciel Azure](../active-directory/managed-identities-azure-resources/how-to-use-vm-sdk.md).

### <a name="limitations"></a>Limites

* Actuellement, vous ne pouvez pas utiliser d’identité managée dans un groupe de conteneurs déployé sur un réseau virtuel.
* Vous ne pouvez pas utiliser d’identité managée pour extraire une image d’Azure Container Registry lors de la création d’un groupe de conteneurs. L’identité est disponible uniquement dans un conteneur en cours d’exécution.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article demande la version 2.0.49 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-an-azure-key-vault"></a>Créer un coffre de clés Azure

Les exemples présentés dans cet article utilisent une identité managée dans Azure Container Instances pour accéder à un code secret du coffre de clés Azure. 

Commencez par créer un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* à l’aide de la commande [az group create](/cli/azure/group?view=azure-cli-latest#az-group-create) suivante :

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Utilisez la commande [az keyvault create](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-create) pour créer un coffre de clés. Veillez à spécifier un nom de coffre de clés unique. 

```azurecli-interactive
az keyvault create \
  --name mykeyvault \
  --resource-group myResourceGroup \ 
  --location eastus
```

Stockez un exemple de secret dans le coffre de clés à l’aide de la commande [az keyvault secret set](/cli/azure/keyvault/secret?view=azure-cli-latest#az-keyvault-secret-set) :

```azurecli-interactive
az keyvault secret set \
  --name SampleSecret \
  --value "Hello Container Instances" \
  --description ACIsecret --vault-name mykeyvault
```

Utilisez les exemples suivants pour accéder au coffre de clés à l’aide d’une identité managée attribuée par l’utilisateur ou par le système dans Azure Container Instances.

## <a name="example-1-use-a-user-assigned-identity-to-access-azure-key-vault"></a>Exemple 1 : Utiliser une identité attribuée par l’utilisateur pour accéder au coffre de clés Azure

### <a name="create-an-identity"></a>Créer une identité

Créez d’abord une identité dans votre abonnement à l’aide de la commande [az identity create](/cli/azure/identity?view=azure-cli-latest#az-identity-create). Vous pouvez utiliser le groupe de ressources utilisé pour créer le coffre de clés, ou en utiliser un autre.

```azurecli-interactive
az identity create \
  --resource-group myResourceGroup \
  --name myACIId
```

Pour utiliser l’identité dans les étapes suivantes, utilisez la commande [az identity show](/cli/azure/identity?view=azure-cli-latest#az-identity-show) pour stocker l’ID de principal de service et l’ID de ressource de l’identité dans des variables.

```azurecli-interactive
# Get service principal ID of the user-assigned identity
spID=$(az identity show \
  --resource-group myResourceGroup \
  --name myACIId \
  --query principalId --output tsv)

# Get resource ID of the user-assigned identity
resourceID=$(az identity show \
  --resource-group myResourceGroup \
  --name myACIId \
  --query id --output tsv)
```

### <a name="grant-user-assigned-identity-access-to-the-key-vault"></a>Autoriser l’identité attribuée par l’utilisateur à accéder au coffre de clés

Exécutez la commande [az keyvault set-policy](/cli/azure/keyvault?view=azure-cli-latest) suivante pour définir une stratégie d’accès sur le coffre de clés. L’exemple suivant permet à l’identité affectée par l’utilisateur d’obtenir des secrets du coffre de clés :

```azurecli-interactive
 az keyvault set-policy \
    --name mykeyvault \
    --resource-group myResourceGroup \
    --object-id $spID \
    --secret-permissions get
```

### <a name="enable-user-assigned-identity-on-a-container-group"></a>Activer une identité attribuée par l’utilisateur dans un groupe de conteneurs

Exécutez la commande [az container create](/cli/azure/container?view=azure-cli-latest#az-container-create) suivante pour créer une instance de conteneur basée sur une image `azure-cli` de Microsoft. Cet exemple fournit un groupe contenant un seul conteneur que vous pouvez utiliser de manière interactive pour accéder à d’autres services Azure. Dans cette section, seul le système d’exploitation de base est utilisé. Pour obtenir un exemple d’utilisation d’Azure CLI dans le conteneur, consultez [Activer l’identité attribuée par le système sur un groupe de conteneurs](#enable-system-assigned-identity-on-a-container-group). 

Le paramètre `--assign-identity` passe votre identité managée attribuée par l’utilisateur au groupe. Cette commande longue laisse le conteneur s’exécuter. Cet exemple utilise le groupe de ressources utilisé pour créer le coffre de clés, mais vous pouvez en spécifier un autre.

```azurecli-interactive
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image mcr.microsoft.com/azure-cli \
  --assign-identity $resourceID \
  --command-line "tail -f /dev/null"
```

Après quelques secondes, vous devez recevoir une réponse d’Azure CLI indiquant que le déploiement est terminé. Vérifiez son état à l’aide de la commande [az container show](/cli/azure/container?view=azure-cli-latest#az-container-show).

```azurecli-interactive
az container show \
  --resource-group myResourceGroup \
  --name mycontainer
```

La section `identity` de la sortie ressemble à ce qui suit, elle montre la définition de l’identité dans le groupe de conteneurs. La valeur `principalID` sous `userAssignedIdentities` correspond au principal de service de l’identité que vous avez créée dans Azure Active Directory :

```console
[...]
"identity": {
    "principalId": "null",
    "tenantId": "xxxxxxxx-f292-4e60-9122-xxxxxxxxxxxx",
    "type": "UserAssigned",
    "userAssignedIdentities": {
      "/subscriptions/xxxxxxxx-0903-4b79-a55a-xxxxxxxxxxxx/resourcegroups/danlep1018/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myACIId": {
        "clientId": "xxxxxxxx-5523-45fc-9f49-xxxxxxxxxxxx",
        "principalId": "xxxxxxxx-f25b-4895-b828-xxxxxxxxxxxx"
      }
    }
  },
[...]
```

### <a name="use-user-assigned-identity-to-get-secret-from-key-vault"></a>Utiliser une identité attribuée par l’utilisateur pour obtenir un secret du coffre de clés

Vous pouvez désormais utiliser l’identité managée dans l’instance de conteneur en cours d’exécution pour accéder au coffre de clés. Commencez par lancer un shell bash dans le conteneur :

```azurecli-interactive
az container exec \
  --resource-group myResourceGroup \
  --name mycontainer \
  --exec-command "/bin/bash"
```

Exécutez les commandes suivantes dans le shell bash du conteneur. Pour obtenir un jeton d’accès permettant d’utiliser Azure Active Directory pour s’authentifier auprès du coffre de clés, exécutez la commande suivante :

```bash
curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fvault.azure.net' -H Metadata:true -s
```

Sortie :

```bash
{"access_token":"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Imk2bEdrM0ZaenhSY1ViMkMzbkVRN3N5SEpsWSIsImtpZCI6Imk2bEdrM0ZaenhSY1ViMkMzbkVRN3N5SEpsWSJ9......xxxxxxxxxxxxxxxxx","refresh_token":"","expires_in":"28799","expires_on":"1539927532","not_before":"1539898432","resource":"https://vault.azure.net/","token_type":"Bearer"}
```

Pour stocker le jeton d’accès dans une variable qui pourra être utilisée dans les prochaines commandes pour l’authentification, exécutez la commande suivante :

```bash
token=$(curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fvault.azure.net' -H Metadata:true | jq -r '.access_token')

```

Utilisez maintenant le jeton d’accès pour vous authentifier auprès du coffre de clés et lire un secret. Veillez à remplacer le nom de votre coffre de clés dans l’URL (*https:\//mykeyvault.vault.azure.net/...* ) :

```bash
curl https://mykeyvault.vault.azure.net/secrets/SampleSecret/?api-version=2016-10-01 -H "Authorization: Bearer $token"
```

La réponse ressemble à ce qui suit, elle affiche le code secret. Dans votre code, il vous faudrait analyser cette sortie pour obtenir le code secret. Utilisez ensuite le code secret dans une opération ultérieure pour accéder à une autre ressource Azure.

```bash
{"value":"Hello Container Instances","contentType":"ACIsecret","id":"https://mykeyvault.vault.azure.net/secrets/SampleSecret/xxxxxxxxxxxxxxxxxxxx","attributes":{"enabled":true,"created":1539965967,"updated":1539965967,"recoveryLevel":"Purgeable"},"tags":{"file-encoding":"utf-8"}}
```

## <a name="example-2-use-a-system-assigned-identity-to-access-azure-key-vault"></a>Exemple 2 : Utiliser une identité attribuée par le système pour accéder à un coffre de clés Azure

### <a name="enable-system-assigned-identity-on-a-container-group"></a>Activer une identité attribuée par le système dans un groupe de conteneurs

Exécutez la commande [az container create](/cli/azure/container?view=azure-cli-latest#az-container-create) suivante pour créer une instance de conteneur basée sur une image `azure-cli` de Microsoft. Cet exemple fournit un groupe contenant un seul conteneur que vous pouvez utiliser de manière interactive pour accéder à d’autres services Azure. 

Le paramètre `--assign-identity`, sans valeur supplémentaire, active une identité managée attribuée par le système dans le groupe. L’identité est limitée au groupe de ressources du groupe de conteneurs. Cette commande longue laisse le conteneur s’exécuter. Cet exemple utilise le même groupe de ressources que celui utilisé pour créer le coffre de clés, qui se trouve dans l’étendue de l’identité.

```azurecli-interactive
# Get the resource ID of the resource group
rgID=$(az group show --name myResourceGroup --query id --output tsv)

# Create container group with system-managed identity
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image mcr.microsoft.com/azure-cli \
  --assign-identity --scope $rgID \
  --command-line "tail -f /dev/null"
```

Après quelques secondes, vous devez recevoir une réponse d’Azure CLI indiquant que le déploiement est terminé. Vérifiez son état à l’aide de la commande [az container show](/cli/azure/container#az-container-show).

```azurecli-interactive
az container show \
  --resource-group myResourceGroup \
  --name mycontainer
```

La section `identity` de la sortie ressemble à ce qui suit. Elle montre qu’une identité attribuée par le système a été créée dans Azure Active Directory :

```console
[...]
"identity": {
    "principalId": "xxxxxxxx-528d-7083-b74c-xxxxxxxxxxxx",
    "tenantId": "xxxxxxxx-f292-4e60-9122-xxxxxxxxxxxx",
    "type": "SystemAssigned",
    "userAssignedIdentities": null
},
[...]
```

Définissez une variable sur la valeur `principalId` (l’ID du principal de service) de l’identité, pour une utilisation ultérieure.

```azurecli-interactive
spID=$(az container show \
  --resource-group myResourceGroup \
  --name mycontainer \
  --query identity.principalId --out tsv)
```

### <a name="grant-container-group-access-to-the-key-vault"></a>Autoriser le groupe de conteneurs à accéder au coffre de clés

Exécutez la commande [az keyvault set-policy](/cli/azure/keyvault?view=azure-cli-latest) suivante pour définir une stratégie d’accès sur le coffre de clés. L’exemple suivant permet à l’identité managée attribuée par le système d’obtenir des secrets du coffre de clés :

```azurecli-interactive
 az keyvault set-policy \
   --name mykeyvault \
   --resource-group myResourceGroup \
   --object-id $spID \
   --secret-permissions get
```

### <a name="use-container-group-identity-to-get-secret-from-key-vault"></a>Utiliser une identité de groupe de conteneurs pour obtenir un secret du coffre de clés

Maintenant, vous pouvez utiliser l’identité managée pour accéder au coffre de clés dans l’instance de conteneur en cours d’exécution. Commencez par lancer un shell bash dans le conteneur :

```azurecli-interactive
az container exec \
  --resource-group myResourceGroup \
  --name mycontainer \
  --exec-command "/bin/bash"
```

Exécutez les commandes suivantes dans le shell bash du conteneur. Connectez-vous d’abord à Azure CLI à l’aide de l’identité managée :

```bash
az login --identity
```

À partir du conteneur en cours d’exécution, récupérez le secret du coffre de clés :

```bash
az keyvault secret show \
  --name SampleSecret \
  --vault-name mykeyvault --query value
```

La valeur du secret est récupérée :

```bash
"Hello Container Instances"
```

## <a name="enable-managed-identity-using-resource-manager-template"></a>Activer une identité managée à l’aide d’un modèle Resource Manager

Pour activer une identité managée dans un groupe de conteneurs à l’aide d’un [modèle Resource Manager](container-instances-multi-container-group.md), définissez la propriété `identity` de l’objet `Microsoft.ContainerInstance/containerGroups` avec un objet `ContainerGroupIdentity`. Dans les extraits de code suivants, la propriété `identity` est configurée pour différents scénarios. Consultez les [références sur les modèles Resource Manager](/azure/templates/microsoft.containerinstance/containergroups). Spécifiez une version `apiVersion` minimale de `2018-10-01`.

### <a name="user-assigned-identity"></a>Identité attribuée par l’utilisateur

Une identité attribuée par l’utilisateur est un ID de ressource qui se présente sous cette forme :

```
"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{identityName}"
``` 

Vous pouvez activer une ou plusieurs identités attribuées par l’utilisateur.

```json
"identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
        "myResourceID1": {
            }
        }
    }
```

### <a name="system-assigned-identity"></a>Identité attribuée par le système

```json
"identity": {
    "type": "SystemAssigned"
    }
```

### <a name="system--and-user-assigned-identities"></a>Identités attribuées par l’utilisateur et par le système

Sur un groupe de conteneurs, vous pouvez activer à la fois une identité attribuée par le système, et une ou plusieurs identités attribuées par l’utilisateur.

```json
"identity": {
    "type": "System Assigned, UserAssigned",
    "userAssignedIdentities": {
        "myResourceID1": {
            }
        }
    }
...
```

## <a name="enable-managed-identity-using-yaml-file"></a>Activer une identité managée à l’aide du fichier YAML

Pour activer une identité managée dans un groupe de conteneurs déployé à l’aide d’un [fichier YAML](container-instances-multi-container-yaml.md), incluez le code YAML suivant.
Spécifiez une version `apiVersion` minimale de `2018-10-01`.

### <a name="user-assigned-identity"></a>Identité attribuée par l’utilisateur

Une identité attribuée par l’utilisateur est un ID de ressource qui se présente sous la forme 

```
'/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{identityName}'
```

Vous pouvez activer une ou plusieurs identités attribuées par l’utilisateur.

```YAML
identity:
  type: UserAssigned
  userAssignedIdentities:
    {'myResourceID1':{}}
```

### <a name="system-assigned-identity"></a>Identité attribuée par le système

```YAML
identity:
  type: SystemAssigned
```

### <a name="system--and-user-assigned-identities"></a>Identités attribuées par l’utilisateur et par le système

Sur un groupe de conteneurs, vous pouvez activer à la fois une identité attribuée par le système, et une ou plusieurs identités attribuées par l’utilisateur.

```YAML
identity:
  type: SystemAssigned, UserAssigned
  userAssignedIdentities:
   {'myResourceID1':{}}
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous en avez appris davantage sur les identités managées dans Azure Container Instances, et vous avez découvert comment :

> [!div class="checklist"]
> * Activer une identité attribuée par l’utilisateur ou par le système dans un groupe de conteneurs
> * Autoriser l’identité à accéder à un coffre de clés Azure
> * Utiliser l’identité managée pour accéder à un coffre de clés à partir d’un conteneur en cours d’exécution

* En savoir plus sur les [identités managées pour les ressources Azure](../active-directory/managed-identities-azure-resources/index.yml).

* Consultez un [exemple de Kit de développement logiciel (SDK) Azure Go](https://medium.com/@samkreter/c98911206328) dans lequel une identité managée est utilisée pour accéder à un coffre de clés Key Vault à partir d’Azure Container Instances.
