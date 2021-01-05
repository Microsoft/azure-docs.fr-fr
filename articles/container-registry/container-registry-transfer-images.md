---
title: Transférer les artefacts
description: Transférez des collections d’images ou d’autres artefacts d’un registre de conteneurs vers un autre registre en créant un pipeline de transfert à l’aide de comptes de stockage Azure
ms.topic: article
ms.date: 10/07/2020
ms.custom: ''
ms.openlocfilehash: fd2cee972ef173853572b871bc80b92b28c505cd
ms.sourcegitcommit: 50802bffd56155f3b01bfb4ed009b70045131750
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91932598"
---
# <a name="transfer-artifacts-to-another-registry"></a>Transférer des artefacts vers un autre registre

Cet article décrit comment transférer des collections d’images ou d’autres artefacts de registre d’un registre de conteneurs Azure vers un autre registre. Les registres source et cible peuvent se trouver dans des abonnements, des locataires Active Directory, des clouds Azure identiques ou différents, ou des clouds physiquement déconnectés. 

Pour transférer des artefacts, vous créez un *pipeline de transfert* qui réplique les artefacts entre deux registres en utilisant un [stockage d’objets blob](../storage/blobs/storage-blobs-introduction.md) :

* Les artefacts d’un registre source sont exportés vers un objet blob dans un compte de stockage source 
* L’objet blob est copié du compte de stockage source vers un compte de stockage cible
* L’objet blob du compte de stockage cible est importé sous forme d’artefacts dans le registre cible. Vous pouvez configurer le pipeline d’importation de sorte qu’il se déclenche dès que l’objet blob d’artefact est mis à jour dans le stockage cible.

Le transfert est idéal pour copier du contenu entre deux registres de conteneurs Azure situés dans des clouds physiquement déconnectés, par le biais de comptes de stockage situés dans chaque cloud. Si, à la place, vous voulez copier des images à partir de registres de conteneurs situés dans des clouds connectés, notamment Docker Hub et d’autres fournisseurs de cloud, il est recommandé d’utiliser [l’importation d’image](container-registry-import-images.md).

Dans cet article, vous utilisez des déploiements de modèle Azure Resource Manager pour créer et exécuter le pipeline de transfert. Azure CLI est utilisé pour approvisionner les ressources associées telles que les secrets de stockage. Azure CLI version 2.2.0 ou supérieure est recommandé. Si vous devez installer ou mettre à niveau l’interface CLI, consultez l’article [Installer Azure CLI 2.0][azure-cli].

Cette fonctionnalité est disponible uniquement au niveau de service **Premium** de registre de conteneurs. Pour plus d’informations sur les niveaux de service et les limites de registre, consultez [Niveaux de service Azure Container Registry](container-registry-skus.md).

> [!IMPORTANT]
> Actuellement, cette fonctionnalité est uniquement disponible en tant que version préliminaire. Les préversions sont à votre disposition, à condition que vous acceptiez les [conditions d’utilisation supplémentaires][terms-of-use]. Certains aspects de cette fonctionnalité sont susceptibles d’être modifiés avant la mise à disposition générale.

## <a name="prerequisites"></a>Prérequis

* **Registres de conteneurs** - Vous devez avoir un registre source existant avec des artefacts à transférer, ainsi qu’un registre cible. Le transfert ACR est destiné au déplacement entre des clouds physiquement déconnectés. Pour le test, les registres source et cible peuvent être dans des abonnements Azure, des locataires Active Directory ou des clouds identiques ou différents. 

   Si vous devez créer un registre, consultez [Démarrage rapide : Créer un registre de conteneurs privé avec Azure CLI](container-registry-get-started-azure-cli.md). 
* **Comptes de stockage** - Créez des comptes de stockage source et cible dans un abonnement ou un emplacement de votre choix. À des fins de test, vous pouvez utiliser le ou les même(s) abonnement(s) comme registres source et cible. Pour les scénarios entre différents clouds, vous créez généralement un compte de stockage distinct dans chaque cloud. 

  Si nécessaire, créez les comptes de stockage avec [Azure CLI](../storage/common/storage-account-create.md?tabs=azure-cli) ou d’autres outils. 

  Créez un conteneur d’objets blob pour le transfert d’artefacts dans chaque compte. Par exemple, créez un conteneur nommé *transfert*. Plusieurs pipelines de transfert peuvent partager le même compte de stockage, mais ils doivent utiliser des étendues de conteneur de stockage différentes.
* **Coffres de clés** - Des coffres de clés sont nécessaires pour stocker les secrets de jetons SAS utilisés pour accéder aux comptes de stockage source et cible. Créez les coffres de clés source et cible dans le ou les même(s) abonnement(s) Azure que vos registres source et cible. À des fins de démonstration, les modèles et les commandes utilisés dans cet article supposent également que les coffres de clés source et cible se trouvent dans les mêmes groupes de ressources que les registres source et cible, respectivement. Cette utilisation de groupes de ressources communs n’est pas obligatoire, mais elle simplifie les modèles et les commandes utilisés dans cet article.

   Si nécessaire, créez les coffres de clés avec [Azure CLI](../key-vault/secrets/quick-create-cli.md) ou d’autres outils.

* **Variables d’environnement** - Pour les exemples de commandes dans cet article, définissez les variables d’environnement suivantes pour les environnements source et cible. Tous les exemples sont mis en forme pour l’interpréteur de commandes Bash.
  ```console
  SOURCE_RG="<source-resource-group>"
  TARGET_RG="<target-resource-group>"
  SOURCE_KV="<source-key-vault>"
  TARGET_KV="<target-key-vault>"
  SOURCE_SA="<source-storage-account>"
  TARGET_SA="<target-storage-account>"
  ```

## <a name="scenario-overview"></a>Présentation du scénario

Vous créez les trois ressources de pipeline suivantes pour le transfert d’images entre registres. Elles sont toutes créées à l’aide d’opérations PUT. Ces ressources fonctionnent sur vos comptes de stockage et registres *source* et *cible*. 

L’authentification du stockage utilise des jetons SAS, gérés en tant que secrets dans les coffres de clés. Les pipelines utilisent des identités gérées pour lire les secrets dans les coffres.

* **[ExportPipeline](#create-exportpipeline-with-resource-manager)** - Ressource durable contenant des informations de niveau supérieur sur le compte de stockage et le registre *sources*. Ces informations incluent l’URI du conteneur d’objets blob du stockage source et le coffre de clés qui gère le jeton SAS source. 
* **[ImportPipeline](#create-importpipeline-with-resource-manager)** - Ressource durable contenant des informations de niveau supérieur sur le compte de stockage et le registre *cibles*. Ces informations incluent l’URI du conteneur d’objets blob du stockage cible et le coffre de clés qui gère le jeton SAS cible. Un déclencheur d’importation étant activé par défaut, le pipeline s’exécute automatiquement lorsqu’un objet blob d’artefact arrive dans le conteneur de stockage cible. 
* **[PipelineRun](#create-pipelinerun-for-export-with-resource-manager)** - Ressource utilisée pour appeler une ressource ExportPipeline ou ImportPipeline.  
  * Vous exécutez ExportPipeline manuellement en créant une ressource PipelineRun et spécifiez les artefacts à exporter.  
  * Si un déclencheur d’importation est activé, ImportPipeline s’exécute automatiquement. Il peut également être exécuté manuellement en utilisant un PipelineRun. 
  * Actuellement, un maximum de **50 artefacts** peuvent être transférés avec chaque PipelineRun.

### <a name="things-to-know"></a>À savoir
* Les ressources ExportPipeline et ImportPipeline se trouvent généralement dans des locataires Active Directory différents associés aux clouds source et de destination. Ce scénario nécessite des identités gérées et des coffres de clés distincts pour les ressources d’exportation et d’importation. À des fins de test, ces ressources peuvent être placées dans le même cloud et partager les identités.
* Par défaut, les modèles ExportPipeline et ImportPipeline permettent chacun à une identité managée attribuée par le système d’accéder aux secrets du coffre de clés. Les modèles ExportPipeline et ImportPipeline prennent également en charge une identité attribuée par l’utilisateur que vous fournissez. 

## <a name="create-and-store-sas-keys"></a>Créer et stocker des clés SAS

Le transfert utilise des jetons de signature d’accès partagé (SAS) pour accéder aux comptes de stockage dans les environnements source et cible. Générez et stockez les jetons selon les indications fournies dans les sections suivantes.  

### <a name="generate-sas-token-for-export"></a>Générer un jeton SAS pour l’exportation

Exécutez la commande [az storage container generate-sas][az-storage-container-generate-sas] pour générer un jeton SAS pour le conteneur du compte de stockage source, utilisé pour l’exportation d’artefacts.

*Autorisations de jeton recommandées* : Lire, Écrire, Répertorier, Ajouter. 

Dans l’exemple suivant, la sortie de la commande est attribuée à la variable d’environnement EXPORT_SAS, précédée du caractère « ? ». Mettez à jour la valeur `--expiry` de votre environnement :

```azurecli
EXPORT_SAS=?$(az storage container generate-sas \
  --name transfer \
  --account-name $SOURCE_SA \
  --expiry 2021-01-01 \
  --permissions alrw \
  --https-only \
  --output tsv)
```

### <a name="store-sas-token-for-export"></a>Stocker un jeton SAS pour l’exportation

Stockez le jeton SAS dans votre coffre de clés Azure source à l’aide de la commande [az keyvault secret set][az-keyvault-secret-set] :

```azurecli
az keyvault secret set \
  --name acrexportsas \
  --value $EXPORT_SAS \
  --vault-name $SOURCE_KV 
```

### <a name="generate-sas-token-for-import"></a>Générer un jeton SAS pour l’importation

Exécutez la commande [az storage container generate-sas][az-storage-container-generate-sas] pour générer un jeton SAS pour le conteneur du compte de stockage cible, utilisé pour l’importation d’artefacts.

*Autorisations de jeton recommandées* : Lire, Supprimer, Répertorier

Dans l’exemple suivant, la sortie de la commande est attribuée à la variable d’environnement IMPORT_SAS, précédée du caractère « ? ». Mettez à jour la valeur `--expiry` de votre environnement :

```azurecli
IMPORT_SAS=?$(az storage container generate-sas \
  --name transfer \
  --account-name $TARGET_SA \
  --expiry 2021-01-01 \
  --permissions dlr \
  --https-only \
  --output tsv)
```

### <a name="store-sas-token-for-import"></a>Stocker un jeton SAS pour l’importation

Stockez le jeton SAS dans votre coffre de clés Azure cible à l’aide de la commande [az keyvault secret set][az-keyvault-secret-set] :

```azurecli
az keyvault secret set \
  --name acrimportsas \
  --value $IMPORT_SAS \
  --vault-name $TARGET_KV 
```

## <a name="create-exportpipeline-with-resource-manager"></a>Créer un ExportPipeline avec Resource Manager

Créez une ressource ExportPipeline pour votre registre de conteneurs source à l’aide d’un déploiement de modèles Azure Resource Manager.

Copiez les [fichiers de modèle](https://github.com/Azure/acr/tree/master/docs/image-transfer/ExportPipelines) Resource Manager de la ressource ExportPipeline dans un dossier local.

Entrez les valeurs de paramètre suivantes dans le fichier `azuredeploy.parameters.json` :

|Paramètre  |Valeur  |
|---------|---------|
|registryName     | Nom de votre registre de conteneurs source      |
|exportPipelineName     |  Nom choisi pour le pipeline d’exportation       |
|targetUri     |  URI du conteneur de stockage dans votre environnement source (la cible du pipeline d’exportation).<br/>Exemple : `https://sourcestorage.blob.core.windows.net/transfer`       |
|keyVaultName     |  Nom du coffre de clés source  |
|sasTokenSecretName  | Nom du secret de jeton SAS dans le coffre de clés source <br/>Exemple : acrexportsas

### <a name="export-options"></a>Options d'exportation

La propriété `options` pour les pipelines d’exportation prend en charge les valeurs booléennes facultatives. Les valeurs suivantes sont recommandées :

|Paramètre  |Valeur  |
|---------|---------|
|options | OverwriteBlobs - Remplacer les objets blob cibles existants<br/>ContinueOnErrors - Continuer l’exportation des artefacts restants dans le registre source en cas d’échec d’une exportation d’artefacts.

### <a name="create-the-resource"></a>Créer la ressource

Exécutez [az deployment group create][az-deployment-group-create] pour créer une ressource nommée *exportPipeline* comme indiqué dans les exemples suivants. Par défaut, avec la première option, l’exemple de modèle active une identité attribuée par le système dans la ressource ExportPipeline. 

Avec la deuxième option, vous pouvez fournir une identité attribuée par l’utilisateur à la ressource. (La création de l’identité attribuée par l’utilisateur n’est pas illustrée.)

Avec l’une ou l’autre des options, le modèle configure l’identité pour accéder au jeton SAS dans le coffre de clés d’exportation. 

#### <a name="option-1-create-resource-and-enable-system-assigned-identity"></a>Option 1 : Créer une ressource et activer l’identité attribuée par le système

```azurecli
az deployment group create \
  --resource-group $SOURCE_RG \
  --template-file azuredeploy.json \
  --name exportPipeline \
  --parameters azuredeploy.parameters.json
```

#### <a name="option-2-create-resource-and-provide-user-assigned-identity"></a>Option n°2 : Créer une ressource et fournir une identité attribuée par l’utilisateur

Dans cette commande, fournissez l’ID de ressource de l’identité attribuée par l’utilisateur en tant que paramètre supplémentaire.

```azurecli
az deployment group create \
  --resource-group $SOURCE_RG \
  --template-file azuredeploy.json \
  --name exportPipeline \
  --parameters azuredeploy.parameters.json \
  --parameters userAssignedIdentity="/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myUserAssignedIdentity"
```

Dans la sortie de commande, prenez note de l’ID de ressource (`id`) du pipeline. Vous pouvez stocker cette valeur dans une variable d’environnement pour une utilisation ultérieure en exécutant la commande [az deployment group show][az-deployment-group-show]. Par exemple :

```azurecli
EXPORT_RES_ID=$(az deployment group show \
  --resource-group $SOURCE_RG \
  --name exportPipeline \
  --query 'properties.outputResources[1].id' \
  --output tsv)
```

## <a name="create-importpipeline-with-resource-manager"></a>Créer un ImportPipeline avec Resource Manager 

Créez une ressource ImportPipeline dans votre registre de conteneurs cible à l’aide d’un déploiement de modèles Azure Resource Manager. Par défaut, le pipeline est activé pour l’importation automatique lorsque le compte de stockage de l’environnement cible possède un objet blob d’artefact.

Copiez les [fichiers de modèle](https://github.com/Azure/acr/tree/master/docs/image-transfer/ImportPipelines) Resource Manager de la ressource ImportPipeline dans un dossier local.

Entrez les valeurs de paramètre suivantes dans le fichier `azuredeploy.parameters.json` :

Paramètre  |Valeur  |
|---------|---------|
|registryName     | Nom de votre registre de conteneurs cible      |
|importPipelineName     |  Nom choisi pour le pipeline d’importation       |
|sourceUri     |  URI du conteneur de stockage dans votre environnement cible (la source du pipeline d’importation).<br/>Exemple : `https://targetstorage.blob.core.windows.net/transfer/`|
|keyVaultName     |  Nom du coffre de clés cible |
|sasTokenSecretName     |  Nom du secret de jeton SAS dans le coffre de clés cible<br/>Exemple : acr importsas |

### <a name="import-options"></a>Options d’importation

La propriété `options` pour le pipeline d’importation prend en charge les valeurs booléennes facultatives. Les valeurs suivantes sont recommandées :

|Paramètre  |Valeur  |
|---------|---------|
|options | OverwriteBlobs - Remplacer les balises cibles existantes<br/>DeleteSourceBlobOnSuccess - Supprimer l’objet blob de stockage source après une importation réussie dans le registre cible<br/>ContinueOnErrors - Continuer l’importation des artefacts restants dans le registre cible en cas d’échec d’une importation d’artefacts.

### <a name="create-the-resource"></a>Créer la ressource

Exécutez [az deployment group create][az-deployment-group-create] pour créer une ressource nommée *importPipeline* comme indiqué dans les exemples suivants. Par défaut, avec la première option, l’exemple de modèle active une identité attribuée par le système dans la ressource ImportPipeline. 

Avec la deuxième option, vous pouvez fournir une identité attribuée par l’utilisateur à la ressource. (La création de l’identité attribuée par l’utilisateur n’est pas illustrée.)

Avec l’une ou l’autre des options, le modèle configure l’identité pour accéder au jeton SAS dans le coffre de clés d’importation. 

#### <a name="option-1-create-resource-and-enable-system-assigned-identity"></a>Option 1 : Créer une ressource et activer l’identité attribuée par le système

```azurecli
az deployment group create \
  --resource-group $TARGET_RG \
  --template-file azuredeploy.json \
  --name importPipeline \
  --parameters azuredeploy.parameters.json 
```

#### <a name="option-2-create-resource-and-provide-user-assigned-identity"></a>Option n°2 : Créer une ressource et fournir une identité attribuée par l’utilisateur

Dans cette commande, fournissez l’ID de ressource de l’identité attribuée par l’utilisateur en tant que paramètre supplémentaire.

```azurecli
az deployment group create \
  --resource-group $TARGET_RG \
  --template-file azuredeploy.json \
  --name importPipeline \
  --parameters azuredeploy.parameters.json \
  --parameters userAssignedIdentity="/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myUserAssignedIdentity"
```

Si vous envisagez d’exécuter l’importation manuellement, prenez note de l’ID de ressource (`id`) du pipeline. Vous pouvez stocker cette valeur dans une variable d’environnement pour une utilisation ultérieure en exécutant la commande [az deployment group show][az-deployment-group-show]. Par exemple :

```azurecli
IMPORT_RES_ID=$(az deployment group show \
  --resource-group $TARGET_RG \
  --name importPipeline \
  --query 'properties.outputResources[1].id' \
  --output tsv)
```

## <a name="create-pipelinerun-for-export-with-resource-manager"></a>Créer un PipelineRun pour l’exportation avec Resource Manager 

Créez une ressource PipelineRun pour votre registre de conteneurs source à l’aide d’un déploiement de modèles Azure Resource Manager. Cette ressource exécute la ressource ExportPipeline que vous avez créée précédemment et exporte les artefacts spécifiés de votre registre de conteneurs en tant qu’objet blob vers votre compte de stockage source.

Copiez les [fichiers de modèle](https://github.com/Azure/acr/tree/master/docs/image-transfer/PipelineRun/PipelineRun-Export) Resource Manager de la ressource PipelineRun dans un dossier local.

Entrez les valeurs de paramètre suivantes dans le fichier `azuredeploy.parameters.json` :

|Paramètre  |Valeur  |
|---------|---------|
|registryName     | Nom de votre registre de conteneurs source      |
|pipelineRunName     |  Nom choisi pour l’exécution       |
|pipelineResourceId     |  ID de ressource du pipeline d’exportation.<br/>Exemple : `/subscriptions/<subscriptionID>/resourceGroups/<resourceGroupName>/providers/Microsoft.ContainerRegistry/registries/<sourceRegistryName>/exportPipelines/myExportPipeline`|
|targetName     |  Nom choisi pour l’objet blob d’artefacts exporté vers votre compte de stockage source, par exemple *myblob*
|artifacts | Tableau d’artefacts source à transférer, en tant que balises ou synthèses de manifestes<br/>Exemple : `[samples/hello-world:v1", "samples/nginx:v1" , "myrepository@sha256:0a2e01852872..."]` |

Si vous redéployez une ressource PipelineRun avec des propriétés identiques, vous devez également utiliser la propriété [forceUpdateTag](#redeploy-pipelinerun-resource).

Exécutez [az deployment group create][az-deployment-group-create] pour créer la ressource PipelineRun. L’exemple suivant nomme le déploiement *exportPipelineRun*.

```azurecli
az deployment group create \
  --resource-group $SOURCE_RG \
  --template-file azuredeploy.json \
  --name exportPipelineRun \
  --parameters azuredeploy.parameters.json
```

Pour une utilisation ultérieure, stockez l’ID de ressource de l’exécution du pipeline dans une variable d’environnement :

```azurecli
EXPORT_RUN_RES_ID=$(az deployment group show \
  --resource-group $SOURCE_RG \
  --name exportPipelineRun \
  --query 'properties.outputResources[0].id' \
  --output tsv)
```

L’exportation des artefacts peut prendre plusieurs minutes. Une fois le déploiement terminé, vérifiez l’exportation des artefacts en dressant la liste des objets blob exportés dans le conteneur *transfert* du compte de stockage source. Par exemple, exécutez la commande [az storage blob list][az-storage-blob-list] :

```azurecli
az storage blob list \
  --account-name $SOURCE_SA \
  --container transfer \
  --output table
```

## <a name="transfer-blob-optional"></a>Transférer un objet blob (facultatif) 

Utilisez l’outil AzCopy ou une autre méthode pour [transférer des données d’objet blob](../storage/common/storage-use-azcopy-blobs.md#copy-blobs-between-storage-accounts) du compte de stockage source vers le compte de stockage cible.

Par exemple, la commande [`azcopy copy`](../storage/common/storage-ref-azcopy-copy.md) suivante copie myblob du conteneur *transfert* du compte source vers le conteneur *transfert* du compte cible. Si l’objet blob existe dans le compte cible, il est remplacé. L’authentification utilise des jetons SAS avec les autorisations appropriées pour les conteneurs source et cible. (Les étapes de création des jetons ne sont pas affichées.)

```console
azcopy copy \
  'https://<source-storage-account-name>.blob.core.windows.net/transfer/myblob'$SOURCE_SAS \
  'https://<destination-storage-account-name>.blob.core.windows.net/transfer/myblob'$TARGET_SAS \
  --overwrite true
```

## <a name="trigger-importpipeline-resource"></a>Déclencher la ressource ImportPipeline

Si vous avez activé le paramètre `sourceTriggerStatus` de ImportPipeline (valeur par défaut), le pipeline est déclenché une fois l’objet blob copié dans le compte de stockage cible. L’importation des artefacts peut prendre plusieurs minutes. Une fois l’importation terminée, vérifiez l’importation des artefacts en dressant la liste des dépôts dans le registre de conteneurs cible. Par exemple, exécutez [az acr repository list][az-acr-repository-list]:

```azurecli
az acr repository list --name <target-registry-name>
```

Si vous n’avez pas activé le paramètre `sourceTriggerStatus` du pipeline d’importation, exécutez la ressource ImportPipeline manuellement, comme indiqué dans la section suivante. 

## <a name="create-pipelinerun-for-import-with-resource-manager-optional"></a>Créer un PipelineRun pour l’importation avec Resource Manager (facultatif) 
 
Vous pouvez également utiliser une ressource PipelineRun pour déclencher un ImportPipeline pour l’importation d’artefacts vers votre registre de conteneurs cible.

Copiez les [fichiers de modèle](https://github.com/Azure/acr/tree/master/docs/image-transfer/PipelineRun/PipelineRun-Import) Resource Manager de la ressource PipelineRun dans un dossier local.

Entrez les valeurs de paramètre suivantes dans le fichier `azuredeploy.parameters.json` :

|Paramètre  |Valeur  |
|---------|---------|
|registryName     | Nom de votre registre de conteneurs cible      |
|pipelineRunName     |  Nom choisi pour l’exécution       |
|pipelineResourceId     |  ID de ressource du pipeline d’importation.<br/>Exemple : `/subscriptions/<subscriptionID>/resourceGroups/<resourceGroupName>/providers/Microsoft.ContainerRegistry/registries/<sourceRegistryName>/importPipelines/myImportPipeline`       |
|sourceName     |  Nom de l’objet blob existant pour les artefacts exportés dans votre compte de stockage, par exemple *myblob*

Si vous redéployez une ressource PipelineRun avec des propriétés identiques, vous devez également utiliser la propriété [forceUpdateTag](#redeploy-pipelinerun-resource).

Exécutez [az deployment group create][az-deployment-group-create] pour exécuter la ressource.

```azurecli
az deployment group create \
  --resource-group $TARGET_RG \
  --name importPipelineRun \
  --template-file azuredeploy.json \
  --parameters azuredeploy.parameters.json
```

Pour une utilisation ultérieure, stockez l’ID de ressource de l’exécution du pipeline dans une variable d’environnement :

```azurecli
IMPORT_RUN_RES_ID=$(az deployment group show \
  --resource-group $TARGET_RG \
  --name importPipelineRun \
  --query 'properties.outputResources[0].id' \
  --output tsv)

When deployment completes successfully, verify artifact import by listing the repositories in the target container registry. For example, run [az acr repository list][az-acr-repository-list]:

```azurecli
az acr repository list --name <target-registry-name>
```

## <a name="redeploy-pipelinerun-resource"></a>Redéployer la ressource PipelineRun

Si vous redéployez une ressource PipelineRun avec des *propriétés identiques*, vous devez tirer parti de la propriété **forceUpdateTag**. Cette propriété indique que la ressource PipelineRun doit être recréée même si la configuration n’a pas changé. Vérifiez que forceUpdateTag est différente chaque fois que vous redéployez la ressource PipelineRun. L’exemple ci-dessous recrée une ressource PipelineRun pour l’exportation. La valeur DateHeure actuelle est utilisée pour définir forceUpdateTag, garantissant ainsi que cette propriété est toujours unique.

```console
CURRENT_DATETIME=`date +"%Y-%m-%d:%T"`
```

```azurecli
az deployment group create \
  --resource-group $SOURCE_RG \
  --template-file azuredeploy.json \
  --name exportPipelineRun \
  --parameters azuredeploy.parameters.json \
  --parameters forceUpdateTag=$CURRENT_DATETIME
```

## <a name="delete-pipeline-resources"></a>Supprimer les ressources de pipeline

Les exemples de commandes suivants utilisent [az resource delete][az-resource-delete] pour supprimer les ressources du pipeline créées dans cet article. Les ID de ressource ont été précédemment stockés dans des variables d’environnement.

```
# Delete export resources
az resource delete \
--resource-group $SOURCE_RG \
--ids $EXPORT_RES_ID $EXPORT_RUN_RES_ID \
--api-version 2019-12-01-preview

# Delete import resources
az resource delete \
--resource-group $TARGET_RG \
--ids $IMPORT_RES_ID $IMPORT_RUN_RES_ID \
--api-version 2019-12-01-preview
```

## <a name="troubleshooting"></a>Dépannage

* **Échecs ou erreurs de déploiement de modèle**
  * En cas d’échec de l’exécution d’un pipeline, observez la propriété `pipelineRunErrorMessage` de la ressource exécutée.
  * Pour les erreurs de déploiement de modèle courantes, consultez [Résoudre les problèmes des déploiements de modèles ARM](../azure-resource-manager/templates/template-tutorial-troubleshoot.md)
* **Problèmes liés à l’exportation ou l’importation d’objets blob de stockage**
  * Il est possible que le jeton SAS soit arrivé à expiration ou qu’il ne dispose pas d’autorisations suffisantes pour l’exécution de l’exportation ou de l’importation spécifiée
  * Il est possible que l’objet blob de stockage du compte de stockage source ne soit pas remplacé lors de plusieurs exécutions d’exportation. Vérifiez que l’option OverwriteBlob est définie dans l’exécution de l’exportation et que le jeton SAS dispose d’autorisations suffisantes.
  * Il est possible que l’objet blob de stockage du compte de stockage cible ne soit pas supprimé après une exécution d’importation réussie. Vérifiez que l’option DeleteBlobOnSuccess est définie dans l’exécution de l’importation et que le jeton SAS dispose d’autorisations suffisantes.
  * L’objet blob de stockage n’a pas été créé ou supprimé. Vérifiez que le conteneur spécifié dans l’exécution de l’exportation ou de l’importation existe ou que l’objet blob de stockage spécifié existe pour une exécution d’importation manuelle. 
* **Problèmes liés à AzCopy**
  * Voir [Résoudre les problèmes liés à AzCopy](../storage/common/storage-use-azcopy-configure.md#troubleshoot-issues).  
* **Problèmes liés au transfert d’artefacts**
  * Tous les artefacts n’ont pas été transférés, voire aucun ne l’a été. Vérifiez l’orthographe des artefacts dans l’exécution d’exportation ainsi que le nom de l’objet blob dans les exécutions d’exportation et d’importation. Vérifiez que le nombre d’artefacts transférés ne dépasse pas 50.
  * Il est possible que l’exécution du pipeline ne soit pas terminée. Une exécution d’exportation ou d’importation peut prendre du temps. 
  * Pour les autres problèmes de pipeline, transmettez l’[ID de corrélation](../azure-resource-manager/templates/deployment-history.md) du déploiement de l’exécution d’exportation ou d’importation à l’équipe Azure Container Registry.


## <a name="next-steps"></a>Étapes suivantes

Pour importer des images d’un seul conteneur dans un registre de conteneurs Azure à partir d’un registre public ou d’un autre registre privé, consultez la documentation de référence de la commande [az acr import][az-acr-import].

<!-- LINKS - External -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/



<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-login]: /cli/azure/reference-index#az-login
[az-keyvault-secret-set]: /cli/azure/keyvault/secret#az-keyvault-secret-set
[az-keyvault-secret-show]: /cli/azure/keyvault/secret#az-keyvault-secret-show
[az-keyvault-set-policy]: /cli/azure/keyvault#az-keyvault-set-policy
[az-storage-container-generate-sas]: /cli/azure/storage/container#az-storage-container-generate-sas
[az-storage-blob-list]: /cli/azure/storage/blob#az-storage-blob-list
[az-deployment-group-create]: /cli/azure/deployment/group#az-deployment-group-create
[az-deployment-group-delete]: /cli/azure/deployment/group#az-deployment-group-delete
[az-deployment-group-show]: /cli/azure/deployment/group#az-deployment-group-show
[az-acr-repository-list]: /cli/azure/acr/repository#az-acr-repository-list
[az-acr-import]: /cli/azure/acr#az-acr-import
[az-resource-delete]: /cli/azure/resource#az-resource-delete
