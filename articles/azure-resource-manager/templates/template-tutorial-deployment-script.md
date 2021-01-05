---
title: Utiliser des scripts de déploiement de modèle | Microsoft Docs
description: Découvrez comment utiliser des scripts de déploiement dans des modèles Azure Resource Manager (modèles ARM).
services: azure-resource-manager
documentationcenter: ''
author: mumian
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 12/14/2020
ms.topic: tutorial
ms.author: jgao
ms.openlocfilehash: ec7b951581efd0a25b44d298b1f1bfb997167d88
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97589098"
---
# <a name="tutorial-use-deployment-scripts-to-create-a-self-signed-certificate"></a>Tutoriel : Utiliser des scripts de déploiement pour créer un certificat auto-signé

Découvrez comment utiliser des scripts de déploiement dans des modèles Azure Resource Manager (modèles ARM). Les scripts de déploiement permettent d’effectuer des étapes personnalisées impossibles à réaliser avec des modèles ARM. Créer un certificat auto-signé est un exemple. Dans ce tutoriel, vous allez créer un modèle pour déployer un coffre de clés Azure, puis utiliser une ressource `Microsoft.Resources/deploymentScripts` dans le même modèle pour créer un certificat et enfin ajouter ce certificat au coffre de clés. Pour en savoir plus sur les scripts de déploiement, consultez [Utiliser des scripts de déploiement dans des modèles ARM](./deployment-script-template.md).

> [!IMPORTANT]
> Deux ressources de script de déploiement, un compte de stockage et une instance de conteneur, sont créées dans le même groupe de ressources pour l’exécution du script et la résolution de problèmes. Ces ressources sont généralement supprimées par le service de script lorsque l’exécution du script atteint un état terminal. Les ressources vous sont facturées jusqu’à leur suppression. Pour plus d’informations, consultez [Nettoyer les ressources de script de déploiement](./deployment-script-template.md#clean-up-deployment-script-resources).

Ce tutoriel décrit les tâches suivantes :

> [!div class="checklist"]
> * Ouvrir un modèle de démarrage rapide
> * Modifier le modèle
> * Déployer le modèle
> * Déboguer le script qui a échoué
> * Nettoyer les ressources

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce qui est décrit dans cet article, vous avez besoin des éléments suivants :

* **[Visual Studio Code](https://code.visualstudio.com/) avec l’extension Outils Resource Manager**. Consultez [Démarrage rapide : Créer des modèles ARM avec Visual Studio Code](./quickstart-create-templates-use-visual-studio-code.md).

* **Une identité managée affectée par l’utilisateur avec le rôle de contributeur au niveau de l’abonnement**. Cette identité est utilisée pour exécuter les scripts de déploiement. Pour en créer une, consultez [Identité managée affectée par l’utilisateur](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md). Vous avez besoin de l’ID d’identité lorsque vous déployez le modèle. Le format de l’identité est le suivant :

  ```json
  /subscriptions/<SubscriptionID>/resourcegroups/<ResourceGroupName>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<IdentityID>
  ```

  Utilisez le script d’interface de ligne de commande suivant pour obtenir l’ID en fournissant le nom du groupe de ressources et le nom de l’identité.

  ```azurecli-interactive
  echo "Enter the Resource Group name:" &&
  read resourceGroupName &&
  az identity list -g $resourceGroupName
  ```

## <a name="open-a-quickstart-template"></a>Ouvrir un modèle de démarrage rapide

Au lieu de créer un modèle à partir de zéro, ouvrez un modèle à partir de [Modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/). Le dépôt Modèles de démarrage rapide Azure contient les modèles ARM.

Le modèle utilisé dans ce guide de démarrage rapide est appelé [Créer un coffre de clés et un secret Azure](https://azure.microsoft.com/resources/templates/101-key-vault-create/). Le modèle crée un coffre de clés, puis y ajoute un secret.

1. À partir de Visual Studio Code, sélectionnez **Fichier** > **Ouvrir un fichier**.
2. Collez l’URL suivante dans **Nom de fichier** :

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-key-vault-create/azuredeploy.json
    ```

3. Sélectionnez **Ouvrir** pour ouvrir le fichier.
4. Sélectionnez **Fichier** > **Enregistrer sous** pour enregistrer le fichier sous le nom _azuredeploy.json_ sur votre ordinateur local.

## <a name="edit-the-template"></a>Modifier le modèle

Apportez les modifications suivantes au modèle :

### <a name="clean-up-the-template-optional"></a>Nettoyer le modèle (facultatif)

Le modèle d’origine ajoute un secret au coffre de clés. Pour simplifier ce tutoriel, supprimez la ressource suivante :

* `Microsoft.KeyVault/vaults/secrets`

Supprimez les deux définitions de paramètre suivantes :

* `secretName`
* `secretValue`

Si vous choisissez de ne pas supprimer ces définitions, vous devez spécifier les valeurs de paramètre pendant le déploiement.

### <a name="configure-the-key-vault-access-policies"></a>Configurer la stratégie d’accès au coffre de clés

Le script de déploiement ajoute un certificat au coffre de clés. Configurez les stratégies d’accès au coffre de clés pour accorder l’autorisation à l’identité managée :

1. Ajoutez un paramètre pour obtenir l’ID de l’identité managée :

    ```json
    "identityId": {
      "type": "string",
      "metadata": {
        "description": "Specifies the ID of the user-assigned managed identity."
      }
    },
    ```

    > [!NOTE]
    > L’extension du modèle Resource Manager de Visual Studio Code ne peut pas encore mettre en forme des scripts de déploiement. N’utilisez pas les touches Maj+Alt+F pour mettre en forme les ressources `deploymentScripts`, comme la suivante.

1. Ajoutez un paramètre pour configurer les stratégies d’accès au coffre de clés afin que l’identité managée puisse ajouter des certificats à ce coffre de clés :

    ```json
    "certificatesPermissions": {
      "type": "array",
      "defaultValue": [
        "get",
        "list",
        "update",
        "create"
      ],
      "metadata": {
      "description": "Specifies the permissions to certificates in the vault. Valid values are: all, get, list, update, create, import, delete, recover, backup, restore, manage contacts, manage certificate authorities, get certificate authorities, list certificate authorities, set certificate authorities, delete certificate authorities."
      }
    }
    ```

1. Mettez à jour les stratégies d’accès au coffre de clés existantes comme suit :

    ```json
    "accessPolicies": [
      {
        "objectId": "[parameters('objectId')]",
        "tenantId": "[parameters('tenantId')]",
        "permissions": {
          "keys": "[parameters('keysPermissions')]",
          "secrets": "[parameters('secretsPermissions')]",
          "certificates": "[parameters('certificatesPermissions')]"
        }
      },
      {
        "objectId": "[reference(parameters('identityId'), '2018-11-30').principalId]",
        "tenantId": "[parameters('tenantId')]",
        "permissions": {
          "keys": "[parameters('keysPermissions')]",
          "secrets": "[parameters('secretsPermissions')]",
          "certificates": "[parameters('certificatesPermissions')]"
        }
      }
    ],
    ```

    Deux stratégies sont définies : une pour l’utilisateur connecté et l’autre pour l’identité managée. L’utilisateur connecté n’a besoin que de l’autorisation *Liste* pour vérifier le déploiement. Pour simplifier ce tutoriel, le même certificat est affecté à l’identité managée et aux utilisateurs connectés.

### <a name="add-the-deployment-script"></a>Ajouter le script de déploiement

1. Ajoutez trois paramètres utilisés par le script de déploiement :

    ```json
    "certificateName": {
      "type": "string",
      "defaultValue": "DeploymentScripts2019"
    },
    "subjectName": {
      "type": "string",
      "defaultValue": "CN=contoso.com"
    },
    "utcValue": {
      "type": "string",
      "defaultValue": "[utcNow()]"
    }
    ```

1. Ajoutez une ressource `deploymentScripts` :

    > [!NOTE]
    > Étant donné que les scripts de déploiement inclus sont placés entre guillemets doubles, les chaînes contenues dans les scripts de déploiement doivent être mises entre guillemets simples. Le [caractère d’échappement PowerShell](/powershell/module/microsoft.powershell.core/about/about_quoting_rules#single-and-double-quoted-strings) est l’accent grave (`` ` ``).

    ```json
    {
      "type": "Microsoft.Resources/deploymentScripts",
      "apiVersion": "2020-10-01",
      "name": "createAddCertificate",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.KeyVault/vaults', parameters('keyVaultName'))]"
      ],
      "identity": {
        "type": "UserAssigned",
        "userAssignedIdentities": {
          "[parameters('identityId')]": {
          }
        }
      },
      "kind": "AzurePowerShell",
      "properties": {
        "forceUpdateTag": "[parameters('utcValue')]",
        "azPowerShellVersion": "3.0",
        "timeout": "PT30M",
        "arguments": "[format(' -vaultName {0} -certificateName {1} -subjectName {2}', parameters('keyVaultName'), parameters('certificateName'), parameters('subjectName'))]", // can pass an argument string, double quotes must be escaped
        "scriptContent": "
          param(
            [string] [Parameter(Mandatory=$true)] $vaultName,
            [string] [Parameter(Mandatory=$true)] $certificateName,
            [string] [Parameter(Mandatory=$true)] $subjectName
          )

          $ErrorActionPreference = 'Stop'
          $DeploymentScriptOutputs = @{}

          $existingCert = Get-AzKeyVaultCertificate -VaultName $vaultName -Name $certificateName

          if ($existingCert -and $existingCert.Certificate.Subject -eq $subjectName) {

            Write-Host 'Certificate $certificateName in vault $vaultName is already present.'

            $DeploymentScriptOutputs['certThumbprint'] = $existingCert.Thumbprint
            $existingCert | Out-String
          }
          else {
            $policy = New-AzKeyVaultCertificatePolicy -SubjectName $subjectName -IssuerName Self -ValidityInMonths 12 -Verbose

            # private key is added as a secret that can be retrieved in the Resource Manager template
            Add-AzKeyVaultCertificate -VaultName $vaultName -Name $certificateName -CertificatePolicy $policy -Verbose

            $newCert = Get-AzKeyVaultCertificate -VaultName $vaultName -Name $certificateName

            # it takes a few seconds for KeyVault to finish
            $tries = 0
            do {
              Write-Host 'Waiting for certificate creation completion...'
              Start-Sleep -Seconds 10
              $operation = Get-AzKeyVaultCertificateOperation -VaultName $vaultName -Name $certificateName
              $tries++

              if ($operation.Status -eq 'failed')
              {
                throw 'Creating certificate $certificateName in vault $vaultName failed with error $($operation.ErrorMessage)'
              }

              if ($tries -gt 120)
              {
                throw 'Timed out waiting for creation of certificate $certificateName in vault $vaultName'
              }
            } while ($operation.Status -ne 'completed')

            $DeploymentScriptOutputs['certThumbprint'] = $newCert.Thumbprint
            $newCert | Out-String
          }
        ",
        "cleanupPreference": "OnSuccess",
        "retentionInterval": "P1D"
      }
    }
    ```

    La ressource `deploymentScripts` dépend de la ressource du coffre de clés et de la ressource d’attribution de rôle. Ses propriétés sont les suivantes :

    * `identity` : Le script de déploiement utilise une identité managée affectée par l’utilisateur pour exécuter les scripts.
    * `kind` : Spécifiez le type de script. Seuls les scripts PowerShell sont actuellement pris en charge.
    * `forceUpdateTag` : Déterminez si le script de déploiement doit être exécuté même si la source du script n’a pas changé. Il peut s’agir de l’horodatage actuel ou d’un GUID. Pour plus d’informations, consultez [Exécuter un script plusieurs fois](./deployment-script-template.md#run-script-more-than-once).
    * `azPowerShellVersion` : Spécifie la version du module Azure PowerShell à utiliser. Actuellement, le script de déploiement prend en charge la version 2.7.0, 2.8.0 et 3.0.0.
    * `timeout` : précise la durée d’exécution maximale autorisée du script, définie au format [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601). La valeur par défaut est **P1D**.
    * `arguments` : Spécifiez les valeurs de paramètre. Les valeurs sont séparées par des espaces.
    * `scriptContent` : Spécifiez le contenu du script. Pour exécuter un script externe, utilisez plutôt `primaryScriptURI`. Pour plus d’informations, consultez [Utiliser un script externe](./deployment-script-template.md#use-external-scripts).
        La déclaration de `$DeploymentScriptOutputs` est uniquement nécessaire lors du test du script sur une machine locale. La déclaration de la variable permet d’exécuter le script sur une machine locale et dans une ressource `deploymentScript` sans avoir à apporter de modifications. La valeur affectée à `$DeploymentScriptOutputs` est disponible en tant que sortie dans les déploiements. Pour plus d’informations, consultez [Utiliser les sorties des scripts de déploiement PowerShell](./deployment-script-template.md#work-with-outputs-from-powershell-script) ou [Utiliser les sorties des scripts de déploiement CLI](./deployment-script-template.md#work-with-outputs-from-cli-script).
    * `cleanupPreference` : Spécifiez votre préférence quant à la suppression des ressources de script de déploiement. La valeur par défaut est **Toujours**, ce qui signifie que les ressources de script de déploiement sont supprimées quel que soit l’état terminal (réussite, échec, annulation). Dans ce tutoriel, **OnSuccess** est utilisé pour vous permettre de voir les résultats de l’exécution du script.
    * `retentionInterval` : Spécifiez l’intervalle pendant lequel le service conserve les ressources de script une fois qu’il a atteint un état terminal. Les ressources sont supprimées à l’issue de cet interval. La durée s’appuie sur le modèle ISO 8601. Ce tutoriel utilise **P1D**, ce qui correspond à une journée. Cette propriété est utilisée quand `cleanupPreference` a la valeur **OnExpiration**. Cette propriété n’est pas activée pour le moment.

    Le script de déploiement prend trois paramètres : `keyVaultName`, `certificateName` et `subjectName`. Il crée un certificat, puis l’ajoute au coffre de clés.

    `$DeploymentScriptOutputs` est utilisé pour stocker la valeur de sortie. Pour en savoir plus, consultez [Utiliser les sorties des scripts de déploiement PowerShell](./deployment-script-template.md#work-with-outputs-from-powershell-script) ou [Utiliser les sorties des scripts de déploiement CLI](./deployment-script-template.md#work-with-outputs-from-cli-script).

    Le modèle complet est disponible [ici](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/deployment-script/deploymentscript-keyvault.json).

1. Pour voir le processus de débogage, placez une erreur dans le code en ajoutant la ligne suivante au script de déploiement :

    ```powershell
    Write-Output1 $keyVaultName
    ```

    La commande correcte est `Write-Output` plutôt que `Write-Output1`.

1. Sélectionnez **Fichier** > **Enregistrer** pour enregistrer le fichier.

## <a name="deploy-the-template"></a>Déployer le modèle

1. Se connecter à [Azure Cloud Shell](https://shell.azure.com)

1. Choisissez votre environnement préféré en sélectionnant **PowerShell** ou **Bash** (pour CLI) en haut à gauche. Il est nécessaire de redémarrer l’interpréteur de commandes lors d’un tel changement.

    ![Fichier de chargement du Cloud Shell du portail Azure](./media/template-tutorial-use-template-reference/azure-portal-cloud-shell-upload-file.png)

1. Sélectionnez **Charger/Télécharger des fichiers**, puis **Charger**. Consultez la capture d’écran précédente.  Sélectionnez le fichier que vous avez enregistré dans la section précédente. Après avoir chargé le fichier, vous pouvez utiliser la commande `ls` et la commande `cat` pour vérifier que le chargement a été correctement effectué.

1. Exécutez le script PowerShell suivant pour déployer le modèle.

    ```azurepowershell-interactive
    $projectName = Read-Host -Prompt "Enter a project name that is used to generate resource names"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $upn = Read-Host -Prompt "Enter your email address used to sign in to Azure"
    $identityId = Read-Host -Prompt "Enter the user-assigned managed identity ID"

    $adUserId = (Get-AzADUser -UserPrincipalName $upn).Id
    $resourceGroupName = "${projectName}rg"
    $keyVaultName = "${projectName}kv"

    New-AzResourceGroup -Name $resourceGroupName -Location $location

    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateFile "$HOME/azuredeploy.json" -identityId $identityId -keyVaultName $keyVaultName -objectId $adUserId

    Write-Host "Press [ENTER] to continue ..."
    ```

    Le service de script de déploiement a besoin de créer des ressources de script de déploiement supplémentaires pour l’exécution du script. La préparation et le processus de nettoyage peuvent prendre jusqu’à une minute, en plus de la durée réelle d’exécution du script.

    Le déploiement a échoué en raison de la commande non valide, `Write-Output1`, utilisée dans le script. Vous recevez une erreur indiquant :

    ```error
    The term 'Write-Output1' is not recognized as the name of a cmdlet, function, script file, or operable
    program. Check the spelling of the name, or if a path was included, verify that the path is correct and try again.
    ```

    Le résultat de l’exécution du script de déploiement est stocké dans les ressources de script de déploiement à des fins de résolution des problèmes.

## <a name="debug-the-failed-script"></a>Déboguer le script qui a échoué

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Ouvrez le groupe de ressources. Il s’agit du nom du projet auquel **rg** est ajouté. Vous devez voir deux ressources supplémentaires dans le groupe de ressources. Ces ressources sont appelées *ressources de script de déploiement*.

    ![Ressources de script de déploiement du modèle Resource Manager](./media/template-tutorial-deployment-script/resource-manager-template-deployment-script-resources.png)

    Les deux fichiers portent le suffixe **azscripts**. L’un est un compte de stockage et l’autre est une instance de conteneur.

    Sélectionnez **Afficher les types masqués** pour lister la ressource `deploymentScripts`.

1. Sélectionnez le compte de stockage portant le suffixe **azscripts**.
1. Sélectionnez la vignette **Partages de fichiers**. Vous devez voir un dossier **azscripts**. Ce dossier contient les fichiers d’exécution du script de déploiement.
1. Sélectionnez **azscripts**. Vous devez voir deux dossiers : **azscriptinput** et **azscriptoutput**. Le dossier input contient un fichier de script PowerShell système et les fichiers de script de déploiement utilisateur. Le dossier output contient un fichier _executionresult.json_ et le fichier de sortie du script. Vous pouvez voir le message d’erreur dans _executionresult.json_. Le fichier de sortie n’est pas là en raison de l’échec de l’exécution.

Supprimez la ligne `Write-Output1` et redéployez le modèle.

Quand la seconde exécution du déploiement réussit, les ressources de script de déploiement doivent être supprimées par le service de script, car la propriété `cleanupPreference` a la valeur **OnSuccess**.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, nettoyez les ressources Azure que vous avez déployées en supprimant le groupe de ressources.

1. Dans le portail Azure, sélectionnez **Groupe de ressources** dans le menu de gauche.
2. Entrez le nom du groupe de ressources dans le champ **Filtrer par nom**.
3. Sélectionnez le nom du groupe de ressources.  Vous verrez six ressources au total dans le groupe de ressources.
4. Sélectionnez **Supprimer le groupe de ressources** dans le menu supérieur.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à utiliser un script de déploiement dans des modèles ARM. Pour savoir comment déployer des ressources Azure en fonction des conditions, consultez :

> [!div class="nextstepaction"]
> [Utiliser des conditions](./template-tutorial-use-conditions.md)
