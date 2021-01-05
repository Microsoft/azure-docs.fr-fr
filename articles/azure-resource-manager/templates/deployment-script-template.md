---
title: Utiliser des scripts de déploiement dans des modèles | Microsoft Docs
description: Utilisez des scripts de déploiement dans des modèles Azure Resource Manager.
services: azure-resource-manager
author: mumian
ms.service: azure-resource-manager
ms.topic: conceptual
ms.date: 12/14/2020
ms.author: jgao
ms.openlocfilehash: c6d171717865fe4bdf3dfb30a6d24badd4fe29ca
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97505560"
---
# <a name="use-deployment-scripts-in-arm-templates"></a>Utiliser des scripts de déploiement dans des modèles ARM

Découvrez comment utiliser des scripts de déploiement dans des modèles Azure Resource Manager (modèles ARM). Avec un nouveau type de ressource appelé `Microsoft.Resources/deploymentScripts`, les utilisateurs peuvent exécuter des scripts dans les déploiements de modèle et examiner les résultats de l’exécution. Ces scripts peuvent être utilisés pour effectuer des étapes personnalisées, comme :

- ajouter des utilisateurs à un annuaire ;
- effectuer des opérations de plan de données, par exemple, copier des objets blob ou alimenter une base de données ;
- rechercher et valider une clé de licence ;
- créer un certificat auto-signé ;
- créer un objet dans Azure AD ;
- rechercher des blocs d’adresses IP à partir du système personnalisé ;

Les avantages du script de déploiement :

- Facile à coder, utiliser et déboguer. Vous pouvez développer des scripts de déploiement dans vos environnements de développement préférés. Les scripts peuvent être incorporés aux modèles ou dans des fichiers de script externe.
- Vous pouvez spécifier le langage de script et la plateforme. À l’heure actuelle, les scripts de déploiement Azure PowerShell et Azure CLI dans l’environnement Linux sont pris en charge.
- Permet la transmission des arguments de ligne de commande au script.
- Peut spécifier des sorties de script et les renvoyer au déploiement.

La ressource de script de déploiement n'est disponible que dans les régions où Azure Container Instance est disponible.  Consultez [Disponibilité des ressources pour Azure Container Instances dans les régions Azure](../../container-instances/container-instances-region-availability.md).

> [!IMPORTANT]
> Un compte de stockage et une instance de conteneur sont nécessaires pour l’exécution et la résolution des problèmes d’un script. Vous avez le choix entre les options pour spécifier un compte de stockage existant ; sinon, le compte de stockage et l’instance de conteneur sont automatiquement créés par le service de script. Les deux ressources créées automatiquement sont généralement supprimées par le service de script lorsque l’exécution du script de déploiement arrive à un état terminal. Vous êtes facturé pour les ressources jusqu’à ce qu’elles soient supprimées. Pour plus d’informations, consultez [Nettoyer les ressources de script de déploiement](#clean-up-deployment-script-resources).

> [!IMPORTANT]
> L’API de ressource deploymentScripts version 2020-10-01 prend en charge [OnBehalfofTokens (OBO)](../../active-directory/develop/v2-oauth2-on-behalf-of-flow.md). À l’aide d’OBO, le service de script de déploiement utilise le jeton du principal de déploiement pour créer les ressources sous-jacentes pour l’exécution de scripts de déploiement, notamment Azure Container Instance, le compte de stockage Azure et les attributions de rôles pour l’identité managée. Dans une ancienne version de l’API, l’identité managée permet de créer ces ressources.
> La logique de nouvelle tentative pour la connexion à Azure est désormais intégrée au script wrapper. Si vous accordez des autorisations dans le même modèle que celui où vous exécutez les scripts de déploiement.  Le service de script de déploiement tente de se connecter pendant 10 minutes avec intervalle de 10 secondes jusqu’à la réplication de l’attribution de rôle d’identité managée.

## <a name="prerequisites"></a>Prérequis

- **(Facultatif) Une identité managée attribuée par l’utilisateur avec les autorisations requises pour effectuer les opérations dans le script**. Pour l’API de script de déploiement version 2020-10-01 ou ultérieure, le principal de déploiement permet de créer des ressources sous-jacentes. Si le script doit s’authentifier auprès d’Azure et effectuer des actions spécifiques à Azure, nous vous recommandons de fournir le script avec une identité managée attribuée à l’utilisateur. L’identité managée doit avoir l’accès requis dans le groupe de ressources cible pour terminer l’opération dans le script. Vous pouvez également vous connecter à Azure dans le script de déploiement. Pour effectuer des opérations en dehors du groupe de ressources, vous devez accorder des autorisations supplémentaires. Par exemple, attribuez l’identité au niveau de l’abonnement si vous souhaitez créer un groupe de ressources. 

  Pour créer une identité, consultez [Créer une identité managée affectée par l’utilisateur à l’aide du Portail Azure](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md), [d’Azure CLI](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md) ou [d’Azure PowerShell](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-powershell.md). Vous avez besoin de l’ID d’identité lorsque vous déployez le modèle. Le format de l’identité est le suivant :

  ```json
  /subscriptions/<SubscriptionID>/resourcegroups/<ResourceGroupName>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<IdentityID>
  ```

  Utilisez le script PowerShell ou CLI suivant pour obtenir l’ID en fournissant le nom du groupe de ressources et le nom de l’identité.

  # <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

  ```azurecli-interactive
  echo "Enter the Resource Group name:" &&
  read resourceGroupName &&
  echo "Enter the managed identity name:" &&
  read idName &&
  az identity show -g $resourceGroupName -n $idName --query id
  ```

  # <a name="powershell"></a>[PowerShell](#tab/PowerShell)

  ```azurepowershell-interactive
  $idGroup = Read-Host -Prompt "Enter the resource group name for the managed identity"
  $idName = Read-Host -Prompt "Enter the name of the managed identity"

  (Get-AzUserAssignedIdentity -resourcegroupname $idGroup -Name $idName).Id
  ```

  ---

- **Azure PowerShell** ou **Azure CLI**. Consultez la liste des versions de [Azure PowerShell prises en charge](https://mcr.microsoft.com/v2/azuredeploymentscripts-powershell/tags/list). Consultez la liste des versions de [Azure CLI prises en charge](https://mcr.microsoft.com/v2/azure-cli/tags/list).

    >[!IMPORTANT]
    > Le script de déploiement utilise les images CLI disponibles à partir de Microsoft Container Registry (MCR). Il faut environ un mois pour certifier une image CLI pour le script de déploiement. N’utilisez pas les versions de l’interface CLI qui ont été publiées il y a moins de 30 jours. Pour trouver les dates de publication des images, consultez les [Notes de publication d’Azure CLI](/cli/azure/release-notes-azure-cli?view=azure-cli-latest&preserve-view=true). Si une version non prise en charge est utilisée, le message d’erreur répertorie les versions prises en charge.

    Vous n’avez pas besoin de ces versions pour déployer des modèles. Par contre, ces versions sont nécessaires pour tester les scripts de déploiement localement. Consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Vous pouvez utiliser une image Docker préconfigurée.  Consultez [Configurer l’environnement de développement](#configure-development-environment).

## <a name="sample-templates"></a>Exemples de modèles

L’extrait json ci-dessous est un exemple.  Le schéma de modèle le plus récent est disponible [ici](/azure/templates/microsoft.resources/deploymentscripts).

```json
{
  "type": "Microsoft.Resources/deploymentScripts",
  "apiVersion": "2020-10-01",
  "name": "runPowerShellInline",
  "location": "[resourceGroup().location]",
  "kind": "AzurePowerShell", // or "AzureCLI"
  "identity": {
    "type": "userAssigned",
    "userAssignedIdentities": {
      "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myID": {}
    }
  },
  "properties": {
    "forceUpdateTag": 1,
    "containerSettings": {
      "containerGroupName": "mycustomaci"
    },
    "storageAccountSettings": {
      "storageAccountName": "myStorageAccount",
      "storageAccountKey": "myKey"
    },
    "azPowerShellVersion": "3.0",  // or "azCliVersion": "2.0.80",
    "arguments": "-name \\\"John Dole\\\"",
    "environmentVariables": [
      {
        "name": "someSecret",
        "secureValue": "if this is really a secret, don't put it here... in plain text..."
      }
    ],
    "scriptContent": "
      param([string] $name)
      $output = 'Hello {0}' -f $name
      Write-Output $output
      $DeploymentScriptOutputs = @{}
      $DeploymentScriptOutputs['text'] = $output
    ", // or "primaryScriptUri": "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-helloworld.ps1",
    "supportingScriptUris":[],
    "timeout": "PT30M",
    "cleanupPreference": "OnSuccess",
    "retentionInterval": "P1D"
  }
}
```

> [!NOTE]
> L’exemple ne sert qu’à des fins de démonstration.  **scriptContent** et **primaryScriptUri** ne peuvent pas coexister dans un modèle.

Détails des valeurs de propriété :

- **Identité** : Pour l’API de script de déploiement version 2020-10-01 ou ultérieure, une identité managée attribuée par l’utilisateur est facultative, sauf si vous devez effectuer des actions spécifiques à Azure dans le script.  Pour la version d’API 2019-10-01-preview, une identité managée est nécessaire, car le service de script de déploiement l’utilise pour exécuter les scripts. Actuellement, seule l’identité managée affectée par l’utilisateur est prise en charge.
- **kind** : spécifie le type de script. Actuellement, les scripts Azure PowerShell et Azure CLI sont pris en charge. Les valeurs sont **AzurePowerShell** et **AzureCLI**.
- **forceUpdateTag** : la modification de cette valeur entre les déploiements de modèle force le script de déploiement à s’exécuter de nouveau. Si vous utilisez la fonction newGuid() ou utcNow(), ces deux fonctions ne peuvent être utilisées que dans la valeur par défaut d’un paramètre. Pour plus d’informations, consultez [Exécuter le script plusieurs fois](#run-script-more-than-once).
- **containerSettings** : Spécifiez les paramètres pour personnaliser l’instance de conteneur Azure.  **containerGroupName** est pour spécifier le nom du groupe de conteneurs.  S’il n’est pas spécifié, le nom du groupe est généré automatiquement.
- **storageAccountSettings**: Spécifiez les paramètres pour utiliser un compte de stockage existant. S’il n’est pas spécifié, un compte de stockage est créé automatiquement. Consultez [Utiliser un compte de stockage existant](#use-existing-storage-account).
- **azPowerShellVersion**/**azCliVersion** : spécifie la version du module à utiliser. Pour obtenir la liste des versions prises en charge de PowerShell et de l’interface CLI, consultez les [conditions préalables](#prerequisites).
- **arguments** : Spécifiez les valeurs de paramètre. Les valeurs sont séparées par des espaces.

    Les scripts de déploiement fractionnent les arguments en un tableau de chaînes en appelant l’appel système [CommandLineToArgvW ](/windows/win32/api/shellapi/nf-shellapi-commandlinetoargvw). Cette étape est nécessaire, car les arguments sont transmis en tant que [propriétés de commande](/rest/api/container-instances/containergroups/createorupdate#containerexec) à Azure Container Instance et la propriété de commande est un tableau de chaîne.

    Si les arguments contiennent des caractères d’échappement, utilisez [JsonEscaper](https://www.jsonescaper.com/) pour double-placer les caractères. Collez votre chaîne d’échappement d’origine dans l’outil, puis sélectionnez **Échappement**.  L’outil génère une chaîne avec deux séquences d’échappement. Par exemple, dans l’exemple de modèle précédent, l’argument est **-nom \\«John Dole\\»** .  La chaîne d’échappement est **\\\\\\«John dole\\\\\\»** .

    Pour passer un paramètre de modèle ARM d’objet type en tant qu’argument, convertissez l’objet en chaîne à l’aide de la fonction [chaîne ()](./template-functions-string.md#string), puis utilisez la fonction [remplacer ()](./template-functions-string.md#replace) pour remplacer tout **\\«** en **\\\\\\»** . Par exemple :

    ```json
    replace(string(parameters('tables')), '\"', '\\\"')
    ```

    Pour afficher un exemple de modèle, sélectionnez [ici](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-jsonEscape.json).

- **environmentVariables** : spécifie les variables d'environnement à transmettre au script. Pour plus d'informations, consultez [Développer des scripts de déploiement](#develop-deployment-scripts).
- **scriptContent** : précise le contenu du script. Pour exécuter un script externe, utilisez plutôt `primaryScriptUri`. Pour obtenir des exemples, consultez [Utiliser un script inclus](#use-inline-scripts) et [Utiliser un script externe](#use-external-scripts).
- **primaryScriptUri** : spécifie une URL accessible publiquement pour le script de déploiement principal avec les extensions de fichier prises en charge.
- **supportingScriptUris** : spécifie un tableau d’URL accessibles publiquement pour les fichiers de prise en charge qui seront appelés dans `ScriptContent` ou `PrimaryScriptUri`.
- **timeout** : précise la durée d’exécution maximale autorisée du script, définie au format [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601). La valeur par défaut est **P1D**.
- **cleanupPreference** : indique la préférence de nettoyage des ressources de déploiement lorsque l’exécution du script arrive à un état terminal. Le paramètre par défaut est **Always**, ce qui signifie que les ressources sont supprimées malgré l’état terminal (Succeeded, Failed, Canceled). Pour plus d’informations, consultez [Nettoyer les ressources de script de déploiement](#clean-up-deployment-script-resources).
- **retentionInterval** : spécifie l’intervalle pendant lequel le service conserve les ressources du script de déploiement une fois que l’exécution du script a atteint un état terminal. Les ressources de script de déploiement sont supprimées à la fin de cette durée. La durée est basée sur le [modèle ISO 8601](https://en.wikipedia.org/wiki/ISO_8601). L’intervalle de rétention est compris entre 1 et 26 heures (PT26H). Cette propriété est utilisée lorsque cleanupPreference a la valeur *OnExpiration*. La propriété *OnExpiration* n’est pas activée actuellement. Pour plus d’informations, consultez [Nettoyer les ressources de script de déploiement](#clean-up-deployment-script-resources).

### <a name="additional-samples"></a>Exemples supplémentaires

- [Exemple 1](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-keyvault.json) : créez un coffre de clés et utiliser le script de déploiement pour affecter un certificat au coffre de clés.
- [Exemple 2](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-keyvault-subscription.json) : créez un groupe de ressources au niveau de l’abonnement, créez un coffre de clés dans le groupe de ressources, puis utilisez le script de déploiement pour affecter un certificat au coffre de clés.
- [Exemple 3](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-keyvault-mi.json) : créez une identité gérée affectée par l’utilisateur, attribuez le rôle collaborateur à l’identité au niveau du groupe de ressources, créez un coffre de clés, puis utilisez le script de déploiement pour affecter un certificat au coffre de clés.

## <a name="use-inline-scripts"></a>Utiliser des scripts inclus

Le modèle suivant dispose d’une ressource définie avec le type `Microsoft.Resources/deploymentScripts`. La partie en surbrillance est le script inclus.

:::code language="json" source="~/resourcemanager-templates/deployment-script/deploymentscript-helloworld.json" range="1-44" highlight="24-30":::

> [!NOTE]
> Étant donné que les scripts de déploiement inclus sont placés entre guillemets doubles, les chaînes contenues dans les scripts de déploiement doivent être placées en échappement à l’aide de **&#92;** ou mises entre guillemets simples. Vous pouvez également envisager d’utiliser la substitution de chaîne, comme montré dans l’exemple JSON précédent.

Le script accepte un paramètre, et génère la valeur du paramètre. **DeploymentScriptOutputs** s’utilise pour stocker les sorties.  À la section outputs, la ligne **value** montre comment accéder aux valeurs stockées. `Write-Output` s’utilise pour le débogage. Pour savoir comment accéder au fichier de sortie, consultez [Analyser et résoudre les problèmes des scripts de déploiement](#monitor-and-troubleshoot-deployment-scripts).  Pour obtenir les descriptions des propriétés, consultez [Exemples de modèles](#sample-templates).

Afin d’exécuter le script, sélectionnez **Try it** (Essayer) pour ouvrir Cloud Shell, puis collez le code suivant dans le volet de l’interpréteur de commandes.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the name of the resource group to be created"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"

New-AzResourceGroup -Name $resourceGroupName -Location $location

New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-helloworld.json"

Write-Host "Press [ENTER] to continue ..."
```

Le résultat se présente ainsi :

![Sortie hello world du script de déploiement du modèle Resource Manager](./media/deployment-script-template/resource-manager-template-deployment-script-helloworld-output.png)

## <a name="use-external-scripts"></a>Utiliser des scripts externes

En plus des scripts inclus, vous pouvez utiliser des fichiers de script externe. Seuls les scripts PowerShell principaux dotés de l’extension de fichier **ps1** sont pris en charge. Pour les scripts CLI, les scripts principaux peuvent avoir n’importe quelle extension (ou n’avoir aucune extension), à condition que les scripts soient des scripts Bash valides. Pour utiliser les fichiers de script externe, remplacez `scriptContent` par `primaryScriptUri`. Par exemple :

```json
"primaryScriptURI": "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-helloworld.ps1",
```

Pour voir un exemple, sélectionnez [ici](https://github.com/Azure/azure-docs-json-samples/blob/master/deployment-script/deploymentscript-helloworld-primaryscripturi.json).

Les fichiers de script externe doivent être accessibles.  Pour sécuriser vos fichiers de script qui sont stockés dans les comptes de stockage Azure, consultez [Déployer un modèle Resource Manager privé avec un jeton SAS](./secure-template-with-sas-token.md).

Vous êtes chargé de garantir l'intégrité des scripts référencés par le script de déploiement, à savoir **PrimaryScriptUri** ou **SupportingScriptUris**.  Référencez uniquement les scripts auxquels vous faites confiance.

## <a name="use-supporting-scripts"></a>Utiliser des scripts de prise en charge

Vous pouvez séparer les logiques complexes en un ou plusieurs fichiers de script de prise en charge. La propriété `supportingScriptUris` vous permet de fournir un tableau d’URI pour les fichiers de script de prise en charge, le cas échéant :

```json
"scriptContent": "
    ...
    ./Create-Cert.ps1
    ...
"

"supportingScriptUris": [
  "https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/create-cert.ps1"
],
```

Les fichiers de script de prise en charge peuvent être appelés à partir de scripts inclus et de fichiers de script principal. Les fichiers de script de prise en charge n’ont aucune restriction quant à l’extension de fichier.

Les fichiers de prise en charge sont copiés dans `azscripts/azscriptinput` au moment de l’exécution. Utilisez un chemin relatif pour référencer les fichiers de prise en charge à partir de scripts inclus et de fichiers de script principal.

## <a name="work-with-outputs-from-powershell-script"></a>Travailler avec les sorties du script PowerShell

Le modèle suivant montre comment passer des valeurs entre deux ressources deploymentScripts :

:::code language="json" source="~/resourcemanager-templates/deployment-script/deploymentscript-basic.json" range="1-68" highlight="30-31,50":::

Dans la première ressource, vous définissez une variable appelée **$DeploymentScriptOutputs** et l’utilisez pour stocker les valeurs de sortie. Pour accéder à la valeur de sortie à partir d’une autre ressource dans le modèle, utilisez :

```json
reference('<ResourceName>').output.text
```

## <a name="work-with-outputs-from-cli-script"></a>Travailler avec les sorties du script CLI

À la différence du script de déploiement PowerShell, la prise en charge de CLI/Bash n’expose pas de variable courante permettant de stocker les sorties de script. Au lieu de cela, il existe une variable d’environnement appelée **AZ_SCRIPTS_OUTPUT_PATH** qui stocke l’emplacement où se trouve le fichier des sorties du script. Si un script de déploiement est exécuté à partir d’un modèle Resource Manager, cette variable d’environnement est automatiquement définie pour vous par l’interpréteur de commandes Bash.

Les sorties de script de déploiement doivent être enregistrées à l’emplacement AZ_SCRIPTS_OUTPUT_PATH et être un objet de chaîne JSON valide. Le contenu du fichier doit être enregistré sous la forme d’une paire clé-valeur. Par exemple, un tableau de chaînes est stocké sous la forme { "MyResult": [ "foo", "bar"] }.  Le stockage des résultats du tableau uniquement, par exemple [ "foo", "bar" ], n’est pas valide.

:::code language="json" source="~/resourcemanager-templates/deployment-script/deploymentscript-basic-cli.json" range="1-44" highlight="32":::

[jq](https://stedolan.github.io/jq/) est utilisé dans l’exemple précédent. Il est fourni avec les images conteneurs. Consultez [Configurer l’environnement de développement](#configure-development-environment).

## <a name="use-existing-storage-account"></a>Utiliser un compte de stockage existant

Un compte de stockage et une instance de conteneur sont nécessaires pour l’exécution et la résolution des problèmes d’un script. Vous avez le choix entre les options pour spécifier un compte de stockage existant ; sinon, le compte de stockage et l’instance de conteneur sont automatiquement créés par le service de script. Exigences requises pour l’utilisation d’un compte de stockage existant :

- Les types de comptes de stockage pris en charge sont les suivants :

    | SKU             | Genre pris en charge     |
    |-----------------|--------------------|
    | Premium_LRS     | FileStorage        |
    | Premium_ZRS     | FileStorage        |
    | Standard_GRS    | Storage, StorageV2 |
    | Standard_GZRS   | StorageV2          |
    | Standard_LRS    | Storage, StorageV2 |
    | Standard_RAGRS  | Storage, StorageV2 |
    | Standard_RAGZRS | StorageV2          |
    | Standard_ZRS    | StorageV2          |

    Ces combinaisons prennent en charge le partage de fichiers.  Pour plus d’informations, consultez[Crée un partage de fichiers Azure](../../storage/files/storage-how-to-create-file-share.md) et [Types de compte de stockage](../../storage/common/storage-account-overview.md).
- Les règles de pare-feu de compte de stockage ne sont pas encore prises en charge. Pour plus d’informations, consultez [Configurer Pare-feu et réseaux virtuels dans Stockage Azure](../../storage/common/storage-network-security.md).
- Le principal de déploiement doit avoir les autorisations nécessaires pour gérer le compte de stockage, y compris la lecture, la création et la suppression des partages de fichiers.

Pour spécifier un compte de stockage existant, ajoutez le code json suivant à l’élément de propriété de `Microsoft.Resources/deploymentScripts` :

```json
"storageAccountSettings": {
  "storageAccountName": "myStorageAccount",
  "storageAccountKey": "myKey"
},
```

- **storageAccountName** : spécifiez le nom du compte de stockage.
- **storageAccountKey "** : spécifiez l’une des clés de compte de stockage. Vous pouvez utiliser la fonction [`listKeys()`](./template-functions-resource.md#listkeys) pour récupérer la clé. Par exemple :

    ```json
    "storageAccountSettings": {
        "storageAccountName": "[variables('storageAccountName')]",
        "storageAccountKey": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName')), '2019-06-01').keys[0].value]"
    }
    ```

Consultez [Exemples de modèles](#sample-templates) pour obtenir un exemple complet de la définition `Microsoft.Resources/deploymentScripts`.

Lorsqu’un compte de stockage existant est utilisé, le service de script crée un partage de fichiers avec un nom unique. Pour savoir comment le service de script nettoie le partage de fichiers, consultez [Nettoyer les ressources de script de déploiement](#clean-up-deployment-script-resources).

## <a name="develop-deployment-scripts"></a>Développer des scripts de déploiement

### <a name="handle-non-terminating-errors"></a>Gérer les erreurs sans fin d’exécution

Vous pouvez contrôler la façon dont PowerShell répond aux erreurs sans fin d’exécution à l’aide de la variable **$ErrorActionPreference** dans votre script de déploiement. Si la variable n’est pas définie dans votre script de déploiement, le service de script utilise la valeur par défaut **Continuer**.

Le service de script définit l’état d’approvisionnement de la ressource sur **Échec** quand le script rencontre une erreur malgré le paramètre $ErrorActionPreference.

### <a name="pass-secured-strings-to-deployment-script"></a>Passer des chaînes sécurisées au script de déploiement

La définition de variables d'environnement (EnvironmentVariable) dans vos instances de conteneur vous permet de fournir une configuration dynamique de l'application ou du script exécuté par le conteneur. Le script de déploiement gère les variables d’environnement sécurisées et non sécurisées de la même manière qu’Azure Container Instance. Pour plus d’informations, consultez [Définir des variables d’environnement dans des instances de conteneur](../../container-instances/container-instances-environment-variables.md#secure-values).

La taille maximale autorisée pour les variables d’environnement est de 64 Ko.

## <a name="monitor-and-troubleshoot-deployment-scripts"></a>Analyser et détecter les problèmes des scripts de déploiement

Le service de script crée un [compte de stockage](../../storage/common/storage-account-overview.md) (sauf si vous spécifiez un compte de stockage existant) et une [instance de conteneur](../../container-instances/container-instances-overview.md) pour l’exécution du script. Si ces ressources sont créées automatiquement par le service de script, les deux ressources ont le suffixe `azscripts` dans les noms des ressources.

![Noms de ressource de script de déploiement d’un modèle Azure Resource Manager](./media/deployment-script-template/resource-manager-template-deployment-script-resources.png)

Le script utilisateur, les résultats de l’exécution et le fichier stdout sont stockés dans les partages de fichiers du compte de stockage. Un dossier est nommé `azscripts`. Dans celui-ci, se trouvent deux dossiers supplémentaires pour les fichiers d’entrée et de sortie : `azscriptinput` et `azscriptoutput`.

Le dossier output contient un fichier **executionresult.json** et le fichier de sortie du script. Vous pouvez voir le message d’erreur de l’exécution du script dans **executionresult.json**. Le fichier de sortie est créé uniquement lorsque le script est exécuté correctement. Le dossier input contient un fichier de script PowerShell système et les fichiers de script de déploiement utilisateur. Vous pouvez remplacer le fichier de script de déploiement utilisateur par un fichier révisé, puis réexécuter le script de déploiement à partir de l’instance de conteneur Azure.

### <a name="use-the-azure-portal"></a>Utilisation du portail Azure

Une fois que vous avez déployé une ressource de script de déploiement, la ressource est listée sous le groupe de ressources dans la Portail Azure. La capture d’écran suivante montre la page vue d’ensemble d’une ressource de script de déploiement :

![Présentation du portail pour un script de déploiement de modèle Resource Manager](./media/deployment-script-template/resource-manager-deployment-script-portal.png)

La page de présentation affiche des informations importantes sur la ressource, telles que l’**État d’approvisionnement**, le **Compte de stockage**, l’**Instance de conteneur** et les **Journaux**.

Dans le menu de gauche, vous pouvez afficher le contenu du script de déploiement, les arguments passés au script et la sortie.  Vous pouvez également exporter un modèle pour le script de déploiement, y compris le script de déploiement.

### <a name="use-powershell"></a>Utiliser PowerShell

À l’aide de Azure PowerShell, vous pouvez gérer les scripts de déploiement au niveau de l’abonnement ou de l’étendue du groupe de ressources :

- [Get-AzDeploymentScript](/powershell/module/az.resources/get-azdeploymentscript) : Obtient ou liste les scripts de déploiement.
- [Get-AzDeploymentScriptLog](/powershell/module/az.resources/get-azdeploymentscriptlog) : Obtient le journal de l’exécution d’un script de déploiement.
- [Remove-AzDeploymentScript](/powershell/module/az.resources/remove-azdeploymentscript) : Supprime un script de déploiement et ses ressources associées.
- [Save-AzDeploymentScriptLog](/powershell/module/az.resources/save-azdeploymentscriptlog) : Enregistre le journal d’exécution d’un script de déploiement sur le disque.

La sortie Get-AzDeploymentScript est similaire à ce qui suit :

```output
Name                : runPowerShellInlineWithOutput
Id                  : /subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0618rg/providers/Microsoft.Resources/deploymentScripts/runPowerShellInlineWithOutput
ResourceGroupName   : myds0618rg
Location            : centralus
SubscriptionId      : 01234567-89AB-CDEF-0123-456789ABCDEF
ProvisioningState   : Succeeded
Identity            : /subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/mydentity1008rg/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myuami
ScriptKind          : AzurePowerShell
AzPowerShellVersion : 3.0
StartTime           : 6/18/2020 7:46:45 PM
EndTime             : 6/18/2020 7:49:45 PM
ExpirationDate      : 6/19/2020 7:49:45 PM
CleanupPreference   : OnSuccess
StorageAccountId    : /subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0618rg/providers/Microsoft.Storage/storageAccounts/ftnlvo6rlrvo2azscripts
ContainerInstanceId : /subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0618rg/providers/Microsoft.ContainerInstance/containerGroups/ftnlvo6rlrvo2azscripts
Outputs             :
                      Key                 Value
                      ==================  ==================
                      text                Hello John Dole

RetentionInterval   : P1D
Timeout             : PT1H
```

### <a name="use-azure-cli"></a>Utiliser l’interface de ligne de commande Microsoft Azure

À l’aide de Azure CLI, vous pouvez gérer les scripts de déploiement au niveau de l’abonnement ou de l’étendue du groupe de ressources :

- [supprimer les scripts de déploiement az](/cli/azure/deployment-scripts?view=azure-cli-latest&preserve-view=true#az-deployment-scripts-delete) : Supprimez un script de déploiement.
- [lister des scripts de déploiement az](/cli/azure/deployment-scripts?view=azure-cli-latest&preserve-view=true#az-deployment-scripts-list) : Listez tous les scripts de déploiement.
- [montrer les scripts de déploiement az](/cli/azure/deployment-scripts?view=azure-cli-latest&preserve-view=true#az-deployment-scripts-show) : Récupérez un script de déploiement.
- [journal des scripts de déploiement az](/cli/azure/deployment-scripts?view=azure-cli-lates&preserve-view=truet#az-deployment-scripts-show-log) : Affichez les journaux de script de déploiement.

La sortie de la commande de liste ressemble à ce qui suit :

```json
[
  {
    "arguments": "-name \\\"John Dole\\\"",
    "azPowerShellVersion": "3.0",
    "cleanupPreference": "OnSuccess",
    "containerSettings": {
      "containerGroupName": null
    },
    "environmentVariables": null,
    "forceUpdateTag": "20200625T025902Z",
    "id": "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0624rg/providers/Microsoft.Resources/deploymentScripts/runPowerShellInlineWithOutput",
    "identity": {
      "tenantId": "01234567-89AB-CDEF-0123-456789ABCDEF",
      "type": "userAssigned",
      "userAssignedIdentities": {
        "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myidentity1008rg/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myuami": {
          "clientId": "01234567-89AB-CDEF-0123-456789ABCDEF",
          "principalId": "01234567-89AB-CDEF-0123-456789ABCDEF"
        }
      }
    },
    "kind": "AzurePowerShell",
    "location": "centralus",
    "name": "runPowerShellInlineWithOutput",
    "outputs": {
      "text": "Hello John Dole"
    },
    "primaryScriptUri": null,
    "provisioningState": "Succeeded",
    "resourceGroup": "myds0624rg",
    "retentionInterval": "1 day, 0:00:00",
    "scriptContent": "\r\n          param([string] $name)\r\n          $output = \"Hello {0}\" -f $name\r\n          Write-Output $output\r\n          $DeploymentScriptOutputs = @{}\r\n          $DeploymentScriptOutputs['text'] = $output\r\n        ",
    "status": {
      "containerInstanceId": "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0624rg/providers/Microsoft.ContainerInstance/containerGroups/64lxews2qfa5uazscripts",
      "endTime": "2020-06-25T03:00:16.796923+00:00",
      "error": null,
      "expirationTime": "2020-06-26T03:00:16.796923+00:00",
      "startTime": "2020-06-25T02:59:07.595140+00:00",
      "storageAccountId": "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0624rg/providers/Microsoft.Storage/storageAccounts/64lxews2qfa5uazscripts"
    },
    "storageAccountSettings": null,
    "supportingScriptUris": null,
    "systemData": {
      "createdAt": "2020-06-25T02:59:04.750195+00:00",
      "createdBy": "someone@contoso.com",
      "createdByType": "User",
      "lastModifiedAt": "2020-06-25T02:59:04.750195+00:00",
      "lastModifiedBy": "someone@contoso.com",
      "lastModifiedByType": "User"
    },
    "tags": null,
    "timeout": "1:00:00",
    "type": "Microsoft.Resources/deploymentScripts"
  }
]
```

### <a name="use-rest-api"></a>Avec l’API REST

Vous pouvez récupérer les informations de déploiement de la ressource de script de déploiement au niveau du groupe de ressources et du niveau de l’abonnement à l’aide de l’API REST :

```rest
/subscriptions/<SubscriptionID>/resourcegroups/<ResourceGroupName>/providers/microsoft.resources/deploymentScripts/<DeploymentScriptResourceName>?api-version=2020-10-01
```

```rest
/subscriptions/<SubscriptionID>/providers/microsoft.resources/deploymentScripts?api-version=2020-10-01
```

L’exemple suivant utilise [ARMClient](https://github.com/projectkudu/ARMClient) :

```azurepowershell
armclient login
armclient get /subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourcegroups/myrg/providers/microsoft.resources/deploymentScripts/myDeployementScript?api-version=2020-10-01
```

Le résultat ressemble à ce qui suit :

```json
{
  "kind": "AzurePowerShell",
  "identity": {
    "type": "userAssigned",
    "tenantId": "01234567-89AB-CDEF-0123-456789ABCDEF",
    "userAssignedIdentities": {
      "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myidentity1008rg/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myuami": {
        "principalId": "01234567-89AB-CDEF-0123-456789ABCDEF",
        "clientId": "01234567-89AB-CDEF-0123-456789ABCDEF"
      }
    }
  },
  "location": "centralus",
  "systemData": {
    "createdBy": "someone@contoso.com",
    "createdByType": "User",
    "createdAt": "2020-06-25T02:59:04.7501955Z",
    "lastModifiedBy": "someone@contoso.com",
    "lastModifiedByType": "User",
    "lastModifiedAt": "2020-06-25T02:59:04.7501955Z"
  },
  "properties": {
    "provisioningState": "Succeeded",
    "forceUpdateTag": "20200625T025902Z",
    "azPowerShellVersion": "3.0",
    "scriptContent": "\r\n          param([string] $name)\r\n          $output = \"Hello {0}\" -f $name\r\n          Write-Output $output\r\n          $DeploymentScriptOutputs = @{}\r\n          $DeploymentScriptOutputs['text'] = $output\r\n        ",
    "arguments": "-name \\\"John Dole\\\"",
    "retentionInterval": "P1D",
    "timeout": "PT1H",
    "containerSettings": {},
    "status": {
      "containerInstanceId": "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0624rg/providers/Microsoft.ContainerInstance/containerGroups/64lxews2qfa5uazscripts",
      "storageAccountId": "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0624rg/providers/Microsoft.Storage/storageAccounts/64lxews2qfa5uazscripts",
      "startTime": "2020-06-25T02:59:07.5951401Z",
      "endTime": "2020-06-25T03:00:16.7969234Z",
      "expirationTime": "2020-06-26T03:00:16.7969234Z"
    },
    "outputs": {
      "text": "Hello John Dole"
    },
    "cleanupPreference": "OnSuccess"
  },
  "id": "/subscriptions/01234567-89AB-CDEF-0123-456789ABCDEF/resourceGroups/myds0624rg/providers/Microsoft.Resources/deploymentScripts/runPowerShellInlineWithOutput",
  "type": "Microsoft.Resources/deploymentScripts",
  "name": "runPowerShellInlineWithOutput"
}

```

L’API REST suivante retourne le journal :

```rest
/subscriptions/<SubscriptionID>/resourcegroups/<ResourceGroupName>/providers/microsoft.resources/deploymentScripts/<DeploymentScriptResourceName>/logs?api-version=2020-10-01
```

Elle fonctionne uniquement avant la suppression des ressources de script de déploiement.

Pour afficher la ressource deploymentScripts dans le portail, sélectionnez **Afficher les types masqués** :

![Portail d’un script de déploiement de modèle Resource Manager avec l’option Afficher les types masqués](./media/deployment-script-template/resource-manager-deployment-script-portal-show-hidden-types.png)

## <a name="clean-up-deployment-script-resources"></a>Nettoyer les ressources de script de déploiement

Un compte de stockage et une instance de conteneur sont nécessaires pour l’exécution et la résolution des problèmes d’un script. Vous avez le choix entre les options pour spécifier un compte de stockage existant ; sinon, un compte de stockage et une instance de conteneur sont automatiquement créés par le service de script. Les deux ressources créées automatiquement sont supprimées par le service de script lorsque l’exécution du script de déploiement arrive à un état terminal. Les ressources vous sont facturées jusqu’à ce qu’elles soient supprimées. Pour plus d’informations sur les prix, consultez [Tarifs Container Instances](https://azure.microsoft.com/pricing/details/container-instances/) et [Tarifs Stockage Azure](https://azure.microsoft.com/pricing/details/storage/).

Le cycle de vie de ces ressources est contrôlé par les propriétés suivantes dans le modèle :

- **cleanupPreference** : nettoie la préférence lorsque l’exécution du script arrive à un état terminal. Les valeurs prises en charge sont les suivantes :

  - **Always** : Supprimez les ressources créées automatiquement une fois que l’exécution du script a atteint un état terminal. Si un compte de stockage existant est utilisé, le service de script supprime le partage de fichiers créé dans le compte de stockage. Étant donné que la ressource deploymentScripts peut encore être présente après le nettoyage des ressources, le service de script conserve les résultats de l’exécution du script (par exemple, stdout, résultats, valeur renvoyée, etc.) avant la suppression des ressources.
  - **OnSuccess** : Supprimez les ressources créées automatiquement uniquement lorsque l’exécution du script est réussie. Si un compte de stockage existant est utilisé, le service de script supprime le partage de fichiers uniquement en cas de réussite de l’exécution du script. Vous pouvez toujours accéder aux ressources pour rechercher les informations de débogage.
  - **OnExpiration** : Supprimez les ressources créées automatiquement uniquement si le paramètre **retentionInterval** a expiré. Si un compte de stockage existant est utilisé, le service de script supprime le partage de fichiers, mais conserve le compte de stockage.

- **retentionInterval** : spécifie l’intervalle de temps pendant lequel une ressource de script est conservée, et après lequel elle expire et est supprimée.

> [!NOTE]
> Il n’est pas recommandé d’utiliser le compte de stockage et l’instance de conteneur qui sont générés par le service de script à d’autres fins. Les deux ressources peuvent être supprimées en fonction du cycle de vie du script.

L’instance de conteneur et le compte de stockage sont supprimés en fonction de **cleanupPreference**. Toutefois, si le script échoue et si **cleanupPreference** n’est pas défini sur **Toujours**, le processus de déploiement maintient automatiquement le conteneur en cours d’exécution pendant une heure. Vous pouvez utiliser cette heure pour résoudre des problèmes de script. Si vous souhaitez laisser le conteneur s’exécuter après des déploiements réussis, ajoutez une étape de mise en veille à votre script. Par exemple, ajoutez [Start-Sleep](https://docs.microsoft.com/powershell/module/microsoft.powershell.utility/start-sleep) à la fin de votre script. Si vous n’ajoutez pas l’étape de mise en veille, le conteneur est défini sur un état terminal et n’est pas accessible, même s’il n’a pas encore été supprimé.

## <a name="run-script-more-than-once"></a>Exécuter le script plusieurs fois

L’exécution d’un script de déploiement est une opération idempotente. Si aucune des propriétés de la ressource deploymentScripts (y compris le script inclus) n’est modifiée, le script ne s’exécute pas lorsque vous redéployez le modèle. Le service de script de déploiement compare les noms des ressource du modèle aux ressources existantes dans le même groupe de ressources. Vous avez deux options à votre disposition si vous souhaitez exécuter le même script de déploiement plusieurs fois :

- Modifiez le nom de votre ressource deploymentScripts. Par exemple, utilisez la fonction de modèle [utcNow](./template-functions-date.md#utcnow) comme nom de ressource, ou comme partie du nom de la ressource. La modification du nom de la ressource crée une nouvelle ressource deploymentScripts. C’est un moyen pratique pour conserver un historique de l’exécution des scripts.

    > [!NOTE]
    > La fonction utcNow ne peut être utilisée que dans la valeur par défaut d’un paramètre.

- Spécifiez une autre valeur dans la propriété de modèle `forceUpdateTag`.  Par exemple, utilisez utcNow comme valeur.

> [!NOTE]
> Écrivez les scripts de déploiement qui sont idempotents. Cela garantit que, s’ils sont exécutés accidentellement, cela n’entraînera de modifications du système. Par exemple, si le script de déploiement est utilisé pour créer une ressource Azure, vérifiez que la ressource n’existe pas avant de la créer, afin que le script aboutisse ou que vous ne recréiez pas la ressource.

## <a name="configure-development-environment"></a>Configurer l’environnement de développement

Vous pouvez utiliser une image conteneur préconfigurée comme environnement de développement de scripts de déploiement. Pour plus d’informations, consultez [Configuration de l’environnement de développement pour les scripts de déploiement dans les modèles](./deployment-script-template-configure-dev.md).

Dès lors que le script a été testé avec succès, vous pouvez l’utiliser en tant que script de déploiement dans vos modèles.

## <a name="deployment-script-error-codes"></a>Codes d’erreur des scripts de déploiement

| Code d'erreur | Description |
|------------|-------------|
| DeploymentScriptInvalidOperation | La définition de la ressource de script de déploiement dans le modèle contient des noms de propriétés non valides. |
| DeploymentScriptResourceConflict | Il est impossible de supprimer une ressource de script de déploiement qui se trouve dans un état non terminal et dont l’exécution n’a pas dépassé une heure. Il est également impossible de réexécuter le même script de déploiement avec le même identificateur de ressource (même abonnement, même nom de groupe de ressources et même nom de ressource), mais un contenu de corps de script différent en même temps. |
| DeploymentScriptOperationFailed | L’opération de script de déploiement a échoué en interne. Contactez le Support Microsoft. |
| DeploymentScriptStorageAccountAccessKeyNotSpecified | La clé d’accès n’a pas été spécifiée pour le compte de stockage existant.|
| DeploymentScriptContainerGroupContainsInvalidContainers | Un groupe de conteneurs créé par le service de script de déploiement a été modifié en externe et des conteneurs non valides ont été ajoutés. |
| DeploymentScriptContainerGroupInNonterminalState | Au moins deux ressources de script de déploiement utilisent le même nom Azure Container Instance dans le même groupe de ressources, et l’une d’entre elles n’a pas encore terminé son exécution. |
| DeploymentScriptStorageAccountInvalidKind | Le compte de stockage existant du type BlobBlobStorage ou BlobStorage ne prend pas en charge les partages de fichiers et ne peut pas être utilisé. |
| DeploymentScriptStorageAccountInvalidKindAndSku | Le compte de stockage existant ne prend pas en charge les partages de fichiers. Pour connaître la liste des types de comptes de stockage pris en charge, consultez [Recours à un compte de stockage existant](#use-existing-storage-account). |
| DeploymentScriptStorageAccountNotFound | Le compte de stockage n’existe pas ou a été supprimé par un processus ou un outil externe. |
| DeploymentScriptStorageAccountWithServiceEndpointEnabled | Le compte de stockage spécifié possède un point de terminaison de service. Les comptes de stockage comportant un point de terminaison de service ne sont pas pris en charge. |
| DeploymentScriptStorageAccountInvalidAccessKey | La clé d’accès spécifiée pour le compte de stockage existant n’est pas valide. |
| DeploymentScriptStorageAccountInvalidAccessKeyFormat | Le format de la clé de compte de stockage n’est pas valide. Consultez [Gérer les clés d’accès au compte de stockage](../../storage/common/storage-account-keys-manage.md). |
| DeploymentScriptExceededMaxAllowedTime | Le temps d’exécution du script de déploiement a dépassé la valeur de délai d’attente spécifiée dans la définition de la ressource de script de déploiement. |
| DeploymentScriptInvalidOutputs | La sortie du script de déploiement n’est pas un objet JSON valide. |
| DeploymentScriptContainerInstancesServiceLoginFailure | L’identité managée attribuée par l’utilisateur n’est pas parvenue à se connecter au bout de 10 tentatives à 1 minute d’intervalle. |
| DeploymentScriptContainerGroupNotFound | Un groupe de conteneurs créé par le service de script de déploiement a été supprimé par un processus ou un outil externe. |
| DeploymentScriptDownloadFailure | Le téléchargement d’un script d’accompagnement a échoué. Consultez [Utilisation d’un script d’accompagnement](#use-supporting-scripts).|
| DeploymentScriptError | Le script utilisateur a généré une erreur. |
| DeploymentScriptBootstrapScriptExecutionFailed | Le script de démarrage a généré une erreur. Le script de démarrage correspond au script système qui orchestre l’exécution du script de déploiement. |
| DeploymentScriptExecutionFailed | Une erreur inconnue s’est produite pendant l’exécution du script de déploiement. |
| DeploymentScriptContainerInstancesServiceUnavailable | Lors de la création de l’instance Azure Container Instances (ACI), ACI a généré une erreur Service indisponible. |
| DeploymentScriptContainerGroupInNonterminalState | Lors de la création d’une instance Azure Container Instances (ACI), un autre script de déploiement utilise le même nom ACI dans la même étendue (même abonnement, même nom de groupe de ressources et même nom de ressource). |
| DeploymentScriptContainerGroupNameInvalid | Le nom de l’instance Azure Container Instances (ACI) spécifié ne répond pas aux exigences ACI. Consultez [Résolution des problèmes courants dans Azure Container Instances](../../container-instances/container-instances-troubleshooting.md#issues-during-container-group-deployment).|

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à utiliser des scripts de déploiement. Pour suivre un tutoriel sur les scripts de déploiement :

> [!div class="nextstepaction"]
> [Tutoriel : Utiliser des scripts de déploiement dans des modèles Azure Resource Manager](./template-tutorial-deployment-script.md)
