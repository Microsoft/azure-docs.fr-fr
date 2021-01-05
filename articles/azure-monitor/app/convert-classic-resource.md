---
title: Migrer une ressource Azure Monitor Application Insights classique vers une ressource basée sur un espace de travail | Microsoft Docs
description: En savoir plus sur les étapes requises pour mettre à niveau votre Azure Monitor Application Insights ressource classique vers le nouveau modèle basé sur l’espace de travail.
ms.topic: conceptual
ms.date: 09/23/2020
ms.openlocfilehash: 709cff1326bb6393a14c594ea434a6c16fb80860
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95536521"
---
# <a name="migrate-to-workspace-based-application-insights-resources"></a>Migrer vers des ressources de Application Insights basées sur un espace de travail

Ce guide va vous guider tout au long du processus de migration d’une ressource de Application Insights classique vers une ressource basée sur un espace de travail. Les ressources basées sur un espace de travail prennent en charge l’intégration complète entre Application Insights et Log Analytics. Les ressources basées sur l’espace de travail envoient Application Insights télémétrie à un espace de travail Log Analytics commun, ce qui vous permet d’accéder aux [fonctionnalités les plus récentes de Azure Monitor](#new-capabilities) tout en conservant les journaux des applications, des infrastructures et des plateformes dans un emplacement consolidé unique.

Les ressources basées sur l’espace de travail activent le contrôle d’accès en fonction du rôle Azure (Azure RBAC) commun sur vos ressources, et éliminent le besoin de requêtes inter-applications/espace de travail.

**Les ressources basées sur l’espace de travail sont actuellement disponibles dans toutes les régions commerciales et Azure US Government**

## <a name="new-capabilities"></a>Nouvelles fonctionnalités

Les Application Insights basées sur l’espace de travail vous permettent de tirer profit de toutes les dernières fonctionnalités d’Azure Monitor et Log Analytics, y compris :

* [Les clés gérées par le client (CMK)](../platform/customer-managed-keys.md) fournissent un chiffrement au repos pour vos données avec des clés de chiffrement auxquelles vous seul avez accès.
* [Azure Private Link](../platform/private-link-security.md) vous permet de lier en toute sécurité les services PaaS Azure à votre réseau virtuel à l’aide de points de terminaison privés.
* [BYOS (apportez votre propre stockage) pour Profiler et Débogueur de capture instantanée](./profiler-bring-your-own-storage.md) vous offre un contrôle total sur la stratégie de chiffrement au repos, la stratégie de gestion de la durée de vie et l’accès réseau pour toutes les données associées à Application Insights Profiler et Débogueur de capture instantanée. 
* [Les niveaux de réservation de capacité](../platform/manage-cost-storage.md#pricing-model) vous permettent d’économiser jusqu’à 25% par rapport au tarif du paiement à l’utilisation. 
* Ingestion plus rapide des données via l’ingestion de diffusion en continu Log Analytics.

## <a name="migration-process"></a>Processus de migration

Lorsque vous migrez vers une ressource basée sur un espace de travail, aucune donnée n’est transférée du stockage de votre ressource classique vers le nouveau stockage basé sur l’espace de travail. En revanche, choisir de migrer modifiera l’emplacement où les nouvelles données seront écrites vers un espace de travail Log Analytics tout en préservant l’accès à vos données de ressource classiques. 

Vos données de ressource classiques seront conservées et soumises aux paramètres de rétention de votre ressource Application Insights classique. Toutes les nouvelles données ingérées après migration seront soumises aux [paramètres de rétention](../platform/manage-cost-storage.md#change-the-data-retention-period) de l’espace de travail Log Analytics associé, qui prend également en charge [différents paramètres de rétention par type de données](../platform/manage-cost-storage.md#retention-by-data-type).
Le processus de migration est **permanent et ne peut pas être inversé**. Une fois que vous migrez une ressource vers une Application Insights basée sur un espace de travail, cette dernière reste toujours une ressource basée sur un espace de travail. Toutefois, une fois la migration effectuée, vous pouvez modifier l’espace de travail cible aussi souvent que nécessaire. 

> [!NOTE]
> L’ingestion et la conservation des données pour les ressources Application Insights basées sur un espace de travail sont [facturées dans l’espace de travail Log Analytics](../platform/manage-cost-storage.md) où se trouvent les données. Si vous avez sélectionné une conservation des données supérieure à 90 jours pour les données ingérées dans la ressource Application Insights classique avant la migration, la conservation des données continuera d’être facturée via cette ressource Application Insights. [En savoir plus]( ./pricing.md#workspace-based-application-insights) sur la facturation des ressources Application Insights basées sur un espace de travail.

Si vous n’avez pas besoin de migrer une ressource existante et que vous souhaitez plutôt créer une ressource de Application Insights basée sur un espace de travail, utilisez le [Guide de création de ressources basé sur l’espace de travail](create-workspace-resource.md).

## <a name="pre-requisites"></a>Conditions préalables 

- Un espace de travail Log Analytics avec le mode de contrôle d’accès défini sur le **`use resource or workspace permissions`** paramètre. 

    - Les ressources de Application Insights basées sur l’espace de travail ne sont pas compatibles avec les espaces de travail définis sur le  **paramètre dédié `workspace based permissions`** . Pour en savoir plus sur le contrôle d’accès de l’espace de travail Log Analytics, consultez le [Guide Log Analytics pour configurer le mode de contrôle d’accès](../platform/manage-access.md#configure-access-control-mode)

    - Si vous ne disposez pas déjà d’un espace de travail Log Analytics, [consultez la documentation relative à la création d’un espace de travail Log Analytics](../learn/quick-create-workspace.md).
    
- L’exportation continue n’est pas prise en charge pour les ressources basées sur l’espace de travail et doit être désactivée.
Une fois la migration terminée, vous pouvez utiliser [les paramètres de diagnostic](../platform/diagnostic-settings.md) pour configurer l’archivage des données sur un compte de stockage ou la diffusion en continu vers Azure Event Hub.  

- Vérifiez vos paramètres de rétention actuels sous **Général** >  **Utilisation et coûts estimés** > **Rétention des données** pour votre espace de travail Log Analytics. Ce paramètre aura un impact sur la durée de stockage des nouvelles données ingérées une fois que vous avez migré vos ressources Application Insights. Si vous stockez actuellement des données Application Insights pendant une durée supérieure à la valeur par défaut de 90 jours et que vous souhaitez conserver cette période de rétention supérieure, vous devrez peut-être ajuster les paramètres de rétention de votre espace de travail.

## <a name="migrate-your-resource"></a>Migrer votre ressource

Cette section vous guidera tout au long du processus de migration d’une ressource de Application Insights classique vers le nouveau type de ressource basé sur l’espace de travail.

1. À partir de votre ressource Application Insights, sélectionnez **Propriétés** sous le titre **Configurer** dans la barre de menus de gauche.

    ![Propriétés mises en surbrillance en rouge](./media/convert-classic-resource/properties.png)

2. Sélectionnez **`Migrate to Workspace-based`**.
    
     ![Bouton Migrer la ressource](./media/convert-classic-resource/migrate.png)

3. Choisissez l’espace de travail Log Analytics dans lequel vous souhaitez stocker toutes les données de télémétrie Application Insights ingérées.

     ![Interface utilisateur de l’Assistant Migration avec option permettant de sélectionner l’espace de travail cible](./media/convert-classic-resource/migration.png)
    

Une fois votre ressource migrée, les informations correspondantes de l’espace de travail s’affichent dans le volet **vue d’ensemble du** :

![Nom de l’espace de travail](./media/create-workspace-resource/workspace-name.png)

Le fait de cliquer sur le texte du lien bleu vous permet d’accéder à l’espace de travail Log Analytics associé dans lequel vous pouvez tirer parti du nouvel environnement de requête de l’espace de travail unifié.

## <a name="understanding-log-queries"></a>Comprendre les requêtes de journal

Nous proposons toujours une compatibilité descendante complète pour vos classeurs, alertes basées sur les journaux et requêtes de ressources Application Insights classiques au sein de l’expérience Application Insights. 

Pour écrire des requêtes sur [le nouveau schéma/la nouvelle structure de table basé(e) sur un espace de travail](apm-tables.md), vous devez d’abord accéder à votre espace de travail Log Analytics. 

Lorsque vous interrogez directement à partir de l’interface utilisateur Log Analytics au sein de votre espace de travail, vous ne voyez que les données qui sont ingérées après la migration. Pour voir à la fois vos données de Application Insights classiques et les nouvelles données ingérées après la migration dans une expérience de requête unifiée, utilisez la vue de requête logs (Analytics) à partir de votre ressource Application Insights migrée.

## <a name="programmatic-resource-migration"></a>Migration de ressources par programme

### <a name="azure-cli"></a>Azure CLI

Pour accéder à la préversion des commandes Azure CLI pour Application Insights, vous devez d’abord exécuter :

```azurecli
 az extension add -n application-insights
```

Si vous n’exécutez pas la commande `az extension add`, vous verrez un message d’erreur qui indique : `az : ERROR: az monitor: 'app-insights' is not in the 'az monitor' command group. See 'az monitor --help'.`.

Vous pouvez maintenant exécuter la commande suivante pour créer votre ressource Application Insights :

```azurecli
az monitor app-insights component update --app
                                         --resource-group
                                         [--ingestion-access {Disabled, Enabled}]
                                         [--kind]
                                         [--query-access {Disabled, Enabled}]
                                         [--retention-time]
                                         [--workspace]
```

#### <a name="example"></a>Exemple

```azurecli
az monitor app-insights component update --app your-app-insights-resource-name -g your_resource_group --workspace "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/test1234/providers/microsoft.operationalinsights/workspaces/test1234555"
```

Pour obtenir la documentation Azure CLI complète pour cette commande, consultez la [documentation Azure CLI](/cli/azure/ext/application-insights/monitor/app-insights/component?view=azure-cli-latest#ext-application-insights-az-monitor-app-insights-component-update).

### <a name="azure-powershell"></a>Azure PowerShell

La `Update-AzApplicationInsights` commande PowerShell ne prend actuellement pas en charge la migration d’une ressource Application Insights classique basée sur l’espace de travail. Pour créer une ressource basée sur l’espace de travail avec PowerShell, vous pouvez utiliser les modèles Azure Resource Manager ci-dessous et les déployer à l’aide de PowerShell.

### <a name="azure-resource-manager-templates"></a>Modèles Microsoft Azure Resource Manager

#### <a name="template-file"></a>Fichier de modèle

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "name": {
            "type": "string"
        },
        "type": {
            "type": "string"
        },
        "regionId": {
            "type": "string"
        },
        "tagsArray": {
            "type": "object"
        },
        "requestSource": {
            "type": "string"
        },
        "workspaceResourceId": {
            "type": "string"
        }
    },
    "resources": [
        {
            "name": "[parameters('name')]",
            "type": "microsoft.insights/components",
            "location": "[parameters('regionId')]",
            "tags": "[parameters('tagsArray')]",
            "apiVersion": "2020-02-02-preview",
            "properties": {
                "ApplicationId": "[parameters('name')]",
                "Application_Type": "[parameters('type')]",
                "Flow_Type": "Redfield",
                "Request_Source": "[parameters('requestSource')]",
                "WorkspaceResourceId": "[parameters('workspaceResourceId')]"
            }
        }
    ]
}
```

#### <a name="parameters-file"></a>Fichier de paramètres

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "type": {
            "value": "web"
        },
        "name": {
            "value": "customresourcename"
        },
        "regionId": {
            "value": "eastus"
        },
        "tagsArray": {
            "value": {}
        },
        "requestSource": {
            "value": "Custom"
        },
        "workspaceResourceId": {
            "value": "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/my_resource_group/providers/microsoft.operationalinsights/workspaces/myworkspacename"
        }
    }
}

```

## <a name="modifying-the-associated-workspace"></a>Modification de l’espace de travail associé

Une fois qu’une ressource Application Insights basée sur l’espace de travail a été créée, vous pouvez modifier l’espace de travail Log Analytics associé.

Dans le volet des ressources Application Insights, sélectionnez **Propriétés** > **Modifier l’espace de travail** > **Espaces de travail Log Analytics**.

## <a name="troubleshooting"></a>Dépannage

### <a name="access-mode"></a>Mode d’accès

**Message d’erreur :** *L’espace de travail sélectionné est configuré avec le mode d’accès basé sur l’espace de travail. Certaines fonctionnalités APM peuvent être affectées. Sélectionnez un autre espace de travail ou autorisez l’accès en fonction des ressources dans les paramètres de l’espace de travail. Vous pouvez remplacer cette erreur à l’aide de l’interface CLI.* 

Pour que votre ressource de Application Insights basée sur l’espace de travail fonctionne correctement, vous devez modifier le mode de contrôle d’accès de votre espace de travail Log Analytics cible en définissant le paramètre des  **autorisations de ressource ou d’espace de travail**. Ce paramètre se trouve dans l’interface utilisateur de l’espace de travail Log Analytics sous **Propriétés** > **Mode de contrôle d’accès**. Pour obtenir des instructions détaillées, consultez le [Guide Log Analytics pour configurer le mode de contrôle d’accès](../platform/manage-access.md#configure-access-control-mode). Si votre mode de contrôle d’accès est défini sur le paramètre exclusif **Exiger des autorisations d’espace de travail**, la migration via l’expérience de migration du portail reste bloquée.

Si vous ne pouvez pas modifier le mode de contrôle d’accès pour des raisons de sécurité pour votre espace de travail cible actuel, nous vous recommandons de créer un espace de travail Log Analytics à utiliser pour la migration. 

### <a name="continuous-export"></a>Exportation continue

**Message d’erreur :** *L’exportation continue doit être désactivée avant de continuer. Après la migration, utilisez les Paramètres de diagnostic pour l’exportation.* 

La fonctionnalité héritée d’exportation continue n’est pas prise en charge pour les ressources basées sur l’espace de travail. Avant la migration, vous devez désactiver l’exportation continue.

1. À partir de l’affichage des ressources de votre Application Insights, sous le titre **Configurer**, sélectionnez **Exportation continue**.

    ![Élément de menu d’exportation continue](./media/convert-classic-resource/continuous-export.png)

2. Sélectionnez **Désactiver**.

    ![Bouton de désactivation de l’exportation continue](./media/convert-classic-resource/disable.png)

- Une fois que vous avez sélectionné désactiver, vous pouvez revenir à l’interface utilisateur de migration. Si la page modifier l’exportation continue vous invite à indiquer que vos paramètres ne seront pas enregistrés, vous pouvez sélectionner OK pour cette invite, car elle ne concerne pas la désactivation ou l’activation de l’exportation continue.

- Une fois que vous avez correctement migré votre ressource Application Insights vers l’espace de travail, vous pouvez utiliser les paramètres de diagnostic pour remplacer les fonctionnalités que l’exportation continue a utilisées. Sélectionnez **Paramètres de diagnostic** > **ajoutez des paramètres de diagnostic** à partir de votre ressource Application Insights. Vous pouvez sélectionner toutes les tables ou un sous-ensemble de tables à archiver dans un compte de stockage ou à transmettre en continu à un Event Hub Azure. Pour obtenir des instructions détaillées sur les paramètres de diagnostic, reportez-vous au [Guide des paramètres de diagnostic Azure Monitor](../platform/diagnostic-settings.md).

### <a name="retention-settings"></a>Paramètres de rétention

**Message d’avertissement :** *Vos paramètres de rétention de Application Insights personnalisés ne s’appliquent pas aux données envoyées à l’espace de travail. Vous devez reconfigurer cette opération séparément.*

Vous n’avez pas besoin d’apporter des modifications avant la migration, mais ce message vous avertit que vos paramètres de rétention de Application Insights actuels ne sont pas définis sur la période de rétention par défaut de 90 jours. Ce message d’avertissement signifie que vous pouvez modifier les paramètres de rétention pour votre espace de travail Log Analytics avant la migration et le démarrage de l’ingestion de nouvelles données. 

Vous pouvez vérifier vos paramètres de rétention actuels pour Log Analytics sous **Général** > **Utilisation et coûts estimés** > **Rétention des données**  dans l’interface utilisateur Log Analytics. Ce paramètre aura un impact sur la durée de stockage des nouvelles données ingérées une fois que vous avez migré vos ressources Application Insights.

## <a name="next-steps"></a>Étapes suivantes

* [Exploration des mesures](../platform/metrics-charts.md)
* [Écriture de requêtes Analytics](../log-query/log-query-overview.md)
