---
title: Configurer BYOS (Bring Your Own Storage) pour Profiler et le Débogueur de capture instantanée
description: Configurer BYOS (Bring Your Own Storage) pour Profiler et le Débogueur de capture instantanée
ms.topic: conceptual
author: renatosalas
ms.author: regutier
ms.date: 04/14/2020
ms.reviewer: mbullwin
ms.openlocfilehash: 719f0cfa0a1f80568acf3231ce3ffab441e5f6b7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87117387"
---
# <a name="configure-bring-your-own-storage-byos-for-application-insights-profiler-and-snapshot-debugger"></a>Configurer BYOS (Bring Your Own Storage) pour Application Insights Profiler et le Débogueur de capture instantanée

## <a name="what-is-bring-your-own-storage-byos-and-why-might-i-need-it"></a>Qu’est-ce que le Bring Your Own Storage (BYOS) et pourquoi pourrais-je en avoir besoin ? 
Lorsque vous utilisez Application Insights Profiler ou le Débogueur de capture instantanée, les artefacts générés par votre application sont téléchargés dans des comptes de stockage Azure via l’Internet public. Ces comptes sont payés et contrôlés par Microsoft à des fins de traitement et d’analyse. Microsoft contrôle les stratégies de gestion du chiffrement au repos et de la durée de vie de ces artefacts.

Avec Bring Your Own Storage, ces artefacts sont téléchargés dans un compte de stockage que vous contrôlez. Cela signifie que vous contrôlez la stratégie de chiffrement au repos, la stratégie de gestion de la durée de vie et l’accès réseau. Toutefois, vous êtes responsable des coûts associés à ce compte de stockage.

> [!NOTE]
> Si vous activez Azure Private Link, vous devez disposer du BYOS. Pour plus d’informations sur Azure Private Link sur Application Insights, consultez la [documentation](../platform/private-link-security.md).
>
> Si vous activez les Customer-Managed Keys, vous devez disposer du BYOS. Pour plus d’informations sur Customer-Managed Keys pour Application Insights, consultez la [documentation](../platform/customer-managed-keys.md).

## <a name="how-will-my-storage-account-be-accessed"></a>Comment accéder à mon compte de stockage ?
1. Les agents s’exécutant sur vos machines virtuelles ou App Service chargent les artefacts (profils, instantanés et symboles) dans les conteneurs blob de votre compte. Ce processus implique de contacter le service Application Insights Profiler ou Débogueur de capture instantanée pour obtenir un jeton SAS (signature d’accès partagé) à un nouveau blob dans votre compte de stockage.
1. Le service Application Insights Profiler ou Débogueur de capture instantanée analyse le blob entrant et écrit les résultats de l’analyse et les fichiers journaux dans le stockage blob. En fonction de la capacité de calcul disponible, ce processus peut se produire à tout moment après le chargement.
1. Lorsque vous affichez les traces du profileur ou l’analyse du débogueur de capture instantanée, le service récupère les résultats de l’analyse à partir du stockage d’objets blob.

## <a name="prerequisites"></a>Prérequis
* Veillez à créer votre compte de stockage au même emplacement que votre ressource Application Insights. Ex. Si votre ressource Application Insights se trouve dans la zone USA Ouest 2, votre compte de stockage doit également se trouver dans la même zone. 
* Accordez le rôle « Contributeur de données du blob de stockage » à l’application AAD « Accès au stockage approuvé des services de diagnostics » dans votre compte de stockage via l’interface utilisateur Access Control (IAM).
* Si Private Link est activé, configurez le paramètre supplémentaire pour autoriser la connexion à notre service Microsoft approuvé à partir de votre réseau virtuel. 

## <a name="how-to-enable-byos"></a>Activer le BYOS

### <a name="create-storage-account"></a>Créer un compte de stockage
Créez un nouveau compte de stockage (si besoin) au même emplacement que votre ressource Application Insights.
Si votre ressource Application Insights se trouve dans la zone `West US 2`, votre compte de stockage doit également se trouver dans la zone `West US 2`.

### <a name="grant-access-to-diagnostic-services-to-your-storage-account"></a>Accorder l’accès aux Services de diagnostic à votre Compte de stockage
Un compte de stockage BYOS sera lié à une ressource Application Insights. Il peut y avoir un seul compte de stockage par ressource Application Insights et les deux doivent se trouver au même emplacement. Vous pouvez utiliser le même compte de stockage avec plusieurs ressources Application Insights.

Tout d’abord, Application Insights Profiler et le service Débogueur de capture instantanée doivent disposer d’un accès au compte de stockage. Pour accorder l’accès, ajoutez le rôle `Storage Blob Data Contributor` à l’application AAD nommée `Diagnostic Services Trusted Storage Access` via la page Access Control (IAM) de votre compte de stockage, comme illustré dans le schéma 1.0. 

Étapes : 
1. Cliquer sur le bouton Ajouter dans la vignette « Ajouter une attribution de rôle » 
1. Sélectionner « Contributeur aux données Blob du stockage » 
1. Sélectionner « Principal de service, groupe ou utilisateur Azure AD » dans la section « Attribuer l’accès à » 
1. Rechercher et sélectionner l’application « Diagnostic Services Trusted Storage Access » 
1. Enregistrer les modifications

_![Schéma 1.0](media/profiler-bring-your-own-storage/figure-10.png)_
_Schéma 1.0_ 

Une fois le rôle ajouté, il apparaît sous la section « Attributions de rôle », comme illustré dans le schéma 1,1 ci-dessous. 
_![Schéma 1.1](media/profiler-bring-your-own-storage/figure-11.png)_
_Schéma 1.1_ 

Si vous utilisez également Azure Private Link, une configuration supplémentaire est nécessaire pour permettre la connexion à notre service Microsoft approuvé depuis votre réseau virtuel. Reportez-vous à la [documentation Sécurité des réseaux de stockage](../../storage/common/storage-network-security.md#trusted-microsoft-services).

### <a name="link-your-storage-account-with-your-application-insights-resource"></a>Reliez votre Compte de stockage à votre ressource Application Insights
Pour configurer le BYOS pour des diagnostics au niveau du code (Profiler/Débogueur), il existe trois options :

* À partir des applets de commande Azure PowerShell
* À partir de l’interface de ligne de commande (CLI) Azure
* Utilisation d’un modèle Azure Resource Manager

#### <a name="configure-using-azure-powershell-cmdlets"></a>Configuration à partir des applets de commande Azure PowerShell

1. Vérifiez que vous avez installé Az PowerShell 4.2.0 ou version supérieure.

    Pour installer Azure PowerShell, reportez-vous à la [documentation Azure PowerShell officielle](/powershell/azure/install-az-ps).

1. Installez l’extension Application Insights PowerShell.
    ```powershell
    Install-Module -Name Az.ApplicationInsights -Force
    ```

1. Connectez-vous à votre compte Azure
    ```powershell
    Connect-AzAccount -Subscription "{subscription_id}"
    ```

    Pour plus d’informations sur la connexion, reportez-vous à la [documentation Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

1. Supprimez le précédent compte de stockage BYOS lié à votre ressource Application Insights.

    Modèle :
    ```powershell
    $appInsights = Get-AzApplicationInsights -ResourceGroupName "{resource_group_name}" -Name "{storage_account_name}"
    Remove-AzApplicationInsightsLinkedStorageAccount -ResourceId $appInsights.Id
    ```

    Exemple :
    ```powershell
    $appInsights = Get-AzApplicationInsights -ResourceGroupName "byos-test" -Name "byos-test-westus2-ai"
    Remove-AzApplicationInsightsLinkedStorageAccount -ResourceId $appInsights.Id
    ```

1. Reliez votre Compte de stockage à votre ressource Application Insights.
    
    Modèle :
    ```powershell
    $storageAccount = Get-AzStorageAccount -ResourceGroupName "{resource_group_name}" -Name "{storage_account_name}"
    $appInsights = Get-AzApplicationInsights -ResourceGroupName "{resource_group_name}" -Name "{application_insights_name}"
    New-AzApplicationInsightsLinkedStorageAccount -ResourceId $appInsights.Id -LinkedStorageAccountResourceId $storageAccount.Id
    ```

    Exemple :
    ```powershell
    $storageAccount = Get-AzStorageAccount -ResourceGroupName "byos-test" -Name "byosteststoragewestus2"
    $appInsights = Get-AzApplicationInsights -ResourceGroupName "byos-test" -Name "byos-test-westus2-ai"
    New-AzApplicationInsightsLinkedStorageAccount -ResourceId $appInsights.Id -LinkedStorageAccountResourceId $storageAccount.Id
    ```

#### <a name="configure-using-azure-cli"></a>Configurer à l’aide d’Azure CLI

1. Assurez-vous que l’interface Azure CLI est installée.

    Pour installer Azure CLI, reportez-vous à la documentation [Azure CLI officielle](/cli/azure/install-azure-cli).

1. Installez l’extension CLI Application Insights.
    ```powershell
    az extension add -n application-insights
    ```

1. Reliez votre Compte de stockage à votre ressource Application Insights.

    Modèle :
    ```powershell
    az monitor app-insights component linked-storage link --resource-group "{resource_group_name}" --app "{application_insights_name}" --storage-account "{storage_account_name}"
    ```
    
    Exemple :
    ```powershell
    az monitor app-insights component linked-storage link --resource-group "byos-test" --app "byos-test-westus2-ai" --storage-account "byosteststoragewestus2"
    ```
    
    Sortie attendue :
    ```powershell
    {
      "id": "/subscriptions/{subscription}/resourcegroups/byos-test/providers/microsoft.insights/components/byos-test-westus2-ai/linkedstorageaccounts/serviceprofiler",
      "linkedStorageAccount": "/subscriptions/{subscription}/resourceGroups/byos-test/providers/Microsoft.Storage/storageAccounts/byosteststoragewestus2",
      "name": "serviceprofiler",
      "resourceGroup": "byos-test",
      "type": "microsoft.insights/components/linkedstorageaccounts"
    }
    ```

    > [!NOTE]
    > Pour effectuer des mises à jour sur les comptes de stockage liés à votre ressource Application Insights, reportez-vous à la documentation [Application Insights CLI](/cli/azure/ext/application-insights/monitor/app-insights/component/linked-storage).

#### <a name="configure-using-azure-resource-manager-template"></a>Activer l’utilisation d’un modèle Azure Resource Manager

1. Créez un fichier de modèle Azure Resource Manager avec le contenu suivant (byos.template.json).
    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "applicationinsights_name": {
          "type": "String"
        },
        "storageaccount_name": {
          "type": "String"
        }
      },
      "variables": {},
      "resources": [
        {
          "name": "[concat(parameters('applicationinsights_name'), '/serviceprofiler')]",
          "type": "Microsoft.Insights/components/linkedStorageAccounts",
          "apiVersion": "2020-03-01-preview",
          "properties": {
            "linkedStorageAccount": "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageaccount_name'))]"
          }
        }
      ],
      "outputs": {}
    }
    ```

1. Exécutez la commande PowerShell suivante pour déployer le modèle précédent (créer un compte de stockage lié).

    Modèle :
    ```powershell
    New-AzResourceGroupDeployment -ResourceGroupName "{your_resource_name}" -TemplateFile "{local_path_to_arm_template}"
    ```

    Exemple :
    ```powershell
    New-AzResourceGroupDeployment -ResourceGroupName "byos-test" -TemplateFile "D:\Docs\byos.template.json"
    ```

1. Fournissez les paramètres suivants lorsque vous y êtes invité dans la console PowerShell :
    
    |           Paramètre           |                                Description                               |
    |-------------------------------|--------------------------------------------------------------------------|
    | application_insights_name     | Nom de la ressource Application Insights pour activer le BYOS.            |
    | storage_account_name          | Nom de la ressource de compte de stockage que vous allez utiliser comme BYOS. |
    
    Sortie attendue :
    ```powershell
    Supply values for the following parameters:
    (Type !? for Help.)
    application_insights_name: byos-test-westus2-ai
    storage_account_name: byosteststoragewestus2
    
    DeploymentName          : byos.template
    ResourceGroupName       : byos-test
    ProvisioningState       : Succeeded
    Timestamp               : 4/16/2020 1:24:57 AM
    Mode                    : Incremental
    TemplateLink            :
    Parameters              :
                              Name                            Type                       Value
                              ==============================  =========================  ==========
                              application_insights_name        String                     byos-test-westus2-ai
                              storage_account_name             String                     byosteststoragewestus2
                              
    Outputs                 :
    DeploymentDebugLogLevel :
    ```

1. Activez les Diagnostics au niveau du code (Profiler/Débogueur) sur la charge de travail qui vous intéresse via le Portail Azure. (App Service > Application Insights) _![Schéma 2.0](media/profiler-bring-your-own-storage/figure-20.png)_
_Schéma 2.0_

## <a name="troubleshooting"></a>Dépannage
### <a name="template-schema-schema_uri-isnt-supported"></a>Le schéma de modèle « {schema_uri} » n’est pas pris en charge.
* Assurez-vous que la propriété `$schema` du modèle est valide. Son modèle doit être le suivant : `https://schema.management.azure.com/schemas/{schema_version}/deploymentTemplate.json#`
* Assurez-vous que la `schema_version` du modèle est comprise dans les valeurs valides : `2014-04-01-preview, 2015-01-01, 2018-05-01, 2019-04-01, 2019-08-01`.
    Message d’erreur :
    ```powershell
    New-AzResourceGroupDeployment : 11:53:49 AM - Error: Code=InvalidTemplate; Message=Deployment template validation failed: 'Template schema
    'https://schema.management.azure.com/schemas/2020-01-01/deploymentTemplate.json#' is not supported. Supported versions are
    '2014-04-01-preview,2015-01-01,2018-05-01,2019-04-01,2019-08-01'. Please see https://aka.ms/arm-template for usage details.'.
    ```

### <a name="no-registered-resource-provider-found-for-location-location"></a>Aucun fournisseur de ressources inscrit pour l’emplacement « {location} » n’a été trouvé.
* Assurez-vous que la `apiVersion` de la ressource `microsoft.insights/components` est `2015-05-01`.
* Assurez-vous que la `apiVersion` de la ressource `linkedStorageAccount` est `2020-03-01-preview`.
    Message d’erreur :
    ```powershell
    New-AzResourceGroupDeployment : 6:18:03 PM - Resource microsoft.insights/components 'byos-test-westus2-ai' failed with message '{
      "error": {
        "code": "NoRegisteredProviderFound",
        "message": "No registered resource provider found for location 'westus2' and API version '2020-03-01-preview' for type 'components'. The supported api-versions are '2014-04-01,
    2014-08-01, 2014-12-01-preview, 2015-05-01, 2018-05-01-preview'. The supported locations are ', eastus, southcentralus, northeurope, westeurope, southeastasia, westus2, uksouth,
    canadacentral, centralindia, japaneast, australiaeast, koreacentral, francecentral, centralus, eastus2, eastasia, westus, southafricanorth, northcentralus, brazilsouth, switzerlandnorth,
    australiasoutheast'."
      }
    }'
    ```
### <a name="storage-account-location-should-match-ai-component-location"></a>L’emplacement du compte de stockage doit correspondre à l’emplacement du composant AI.
* Assurez-vous que l’emplacement de la ressource Application Insights est le même que celui du Compte de stockage.
    Message d’erreur :
    ```powershell
    New-AzResourceGroupDeployment : 1:01:12 PM - Resource microsoft.insights/components/linkedStorageAccounts 'byos-test-centralus-ai/serviceprofiler' failed with message '{
      "error": {
        "code": "BadRequest",
        "message": "Storage account location should match AI component location",
        "innererror": {
          "trace": [
            "System.ArgumentException"
          ]
        }
      }
    }'
    ```

Pour obtenir des informations générales sur la résolution des problèmes du Profiler, reportez-vous à la [documentation Dépannage du Profiler](profiler-troubleshooting.md).

Pour obtenir des informations générales sur la résolution des problèmes du Débogueur de capture instantanée, consultez la [documentation Dépannage du Débogueur de capture instantanée](snapshot-debugger-troubleshoot.md). 

## <a name="faqs"></a>FAQ
* Si j’ai activé le Profiler ou l’Instantané, puis que j’ai activé le BYOS, mes données seront-elles migrées dans mon compte de stockage ?
    _Non, elles ne le seront pas._

* Le BYOS utilisera-t-il un chiffrement au repos et une clé gérée par le client ?
    _Oui, pour être précis, le BYOS est requis pour activer le Profilr/Débogueur avec les clés du gestionnaire de clients._

* Le BYOS fonctionne-t-il dans un environnement isolé d’Internet ?
    _Oui. En fait, le BYOS est requis pour les scénarios de réseau isolé._

* Le BYOS fonctionnera-t-il lorsque les clés gérées par le client et Azure Private Link seront tous deux activés ? 
    _Oui, cela est possible._

* Si j’ai activé le BYOS, puis-je revenir en arrière en utilisant les comptes de stockage des services de diagnostic pour stocker mes données collectées ? 
    _Oui, vous le pouvez, mais à ce stade, nous ne prenons pas en charge la migration des données à partir de votre BYOS._

* Après avoir activé le BYOS, vais-je prendre en charge tous les coûts liés à l’informatique, à savoir le stockage et la mise en réseau ? 
    _Oui_
