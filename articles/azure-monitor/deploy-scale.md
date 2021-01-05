---
title: Déployer les fonctionnalités Azure Monitor à la bonne échelle à l’aide d’Azure Policy
description: Déployez les fonctionnalités Azure Monitor à la bonne échelle à l’aide d’Azure Policy.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 06/08/2020
ms.openlocfilehash: f2f2272363cbc26895b061fe7b6263ed2a29fbab
ms.sourcegitcommit: 83610f637914f09d2a87b98ae7a6ae92122a02f1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91993247"
---
# <a name="deploy-azure-monitor-at-scale-using-azure-policy"></a>Déployer les fonctionnalités Azure Monitor à la bonne échelle à l’aide d’Azure Policy
Même si certaines fonctionnalités Azure Monitor sont configurées une fois pour toutes ou un nombre de fois limité, pour d’autres, l’opération doit être répétée pour chacune des ressources que vous voulez superviser. Cet article décrit des méthodes pour implémenter Azure Monitor à la bonne échelle avec Azure Policy et garantir une configuration cohérente et précise de la supervision pour toutes vos ressources Azure.

Par exemple, vous devez créer un paramètre de diagnostic pour toutes vos ressources Azure existantes et pour chaque nouvelle ressource que vous créez. Vous devez aussi installer et configurer un agent chaque fois que vous créez une machine virtuelle. Vous pouvez effectuer ces opérations avec PowerShell ou l’interface CLI, qui sont des méthodes compatibles avec toutes les fonctionnalités Azure Monitor. Avec Azure Policy, vous pouvez mettre en place une logique capable d’effectuer automatiquement la configuration appropriée chaque fois que vous créez ou modifiez une ressource.


## <a name="azure-policy"></a>Azure Policy
Cette section offre une brève présentation d’[Azure Policy](../governance/policy/overview.md) qui permet d’évaluer et d’appliquer les standards organisationnels à l’échelle d’un abonnement ou d’un groupe d’administration Azure avec un minimum d’effort. Pour des détails complets, consultez la [documentation Azure Policy](../governance/policy/overview.md).

Avec Azure Policy, vous pouvez spécifier des exigences de configuration pour l’ensemble des ressources qui sont créées et soit identifier celles qui ne sont pas conformes, soit empêcher la création des ressources, soit ajouter la configuration exigée. Les appels à créer une nouvelle ressource ou à en modifier une existante sont systématiquement interceptés. Ainsi, une demande peut être refusée si ses propriétés ne correspondent pas aux propriétés attendues dans une définition de stratégie (elle est alors marquée comme étant non conforme) ou une ressource associée peut être déployée. Vous pouvez corriger les ressources existantes avec une définition de stratégie **deployIfNotExists** ou **modify**.

Azure Policy comporte les objets présents dans le tableau suivant. Pour obtenir une description plus détaillée de chacun d’eux, consultez [Objets Azure Policy](../governance/policy/overview.md#azure-policy-objects).

| Élément | Description |
|:---|:---|
| Définition de stratégie | Décrit les conditions de conformité des ressources et l’effet à produire si une condition est remplie. Elle peut s’appliquer à toutes les ressources d’un type particulier ou seulement à celles qui ont certaines propriétés. L’effet peut consister simplement à marquer la ressource pour conformité ou à déployer une ressource associée. Les définitions de stratégie sont écrites en JSON, comme indiqué dans [Structure de définition Azure Policy](../governance/policy/concepts/definition-structure.md). Les effets sont décrits dans [Comprendre les effets d’Azure Policy](../governance/policy/concepts/effects.md).
| Initiative de stratégie | Groupe de définitions de stratégie qui doivent être appliquées ensemble. Par exemple, vous pouvez disposer d’une définition de stratégie pour l’envoi des journaux de ressources à un espace de travail Log Analytics et une autre pour l’envoi des journaux de ressources à des hubs d’événements. Créez une initiative qui regroupe les deux définitions de stratégie, puis appliquez l’initiative aux ressources au lieu des définitions de stratégie individuelles. Les initiatives sont écrites en JSON, comme indiqué dans [Structure de définition Azure Policy](../governance/policy/concepts/initiative-definition-structure.md). |
| Affectation | Une définition ou une initiative de stratégie ne prend pas effet tant qu’elle n’est pas affectée à une étendue. Par exemple, affectez une stratégie à un groupe de ressources pour l’appliquer à toutes les ressources créées dans cette ressource, ou appliquez-la à un abonnement pour l’appliquer à toutes les ressources de cet abonnement.  Pour plus d’informations, consultez [Structure d’attribution Azure Policy](../governance/policy/concepts/assignment-structure.md). |

## <a name="built-in-policy-definitions-for-azure-monitor"></a>Définitions de stratégie intégrées pour Azure Monitor
Azure Policy comprend plusieurs définitions prédéfinies en rapport avec Azure Monitor. Vous pouvez affecter ces définitions de stratégie à votre abonnement existant ou vous en servir de base pour créer vos propres définitions personnalisées. Pour obtenir la liste complète des politiques intégrées de la catégorie **Supervision**, consultez [Définitions de stratégie intégrées d’Azure Policy pour Azure Monitor](./samples/policy-reference.md).

Pour consulter les définitions de stratégie intégrées en rapport avec la supervision, procédez comme suit :

1. Accédez à **Azure Policy** sur le portail Azure.
2. Sélectionnez **Définitions**.
3. Pour **Type**, sélectionnez *Intégré* pour **Catégorie**, sélectionnez *Supervision*.

  ![Capture d’écran de la page de définitions d’Azure Policy dans le Portail Azure montrant une liste de définitions de stratégie pour la catégorie Supervision et le type Intégré.](media/deploy-scale/builtin-policies.png)


## <a name="diagnostic-settings"></a>Paramètres de diagnostic
Les [paramètres de diagnostic](platform/diagnostic-settings.md) recueillent les journaux et les métriques des ressources Azure à plusieurs emplacements, généralement dans un espace de travail Log Analytics, ce qui vous permet d’analyser les données avec des [requêtes de journal](log-query/log-query-overview.md) et des [alertes de journal](platform/alerts-log.md). Utilisez Policy pour créer automatiquement un paramètre de diagnostic chaque fois que vous créez une ressource.

Chaque type de ressource Azure est constitué d’un ensemble unique de catégories qui doivent être listées dans le paramètre de diagnostic. Pour cette raison, chaque type de ressource a besoin d’une définition de stratégie distincte. Certains types de ressources comportent des définitions de stratégie intégrées que vous pouvez affecter sans modification. Pour les autres types de ressources, vous devez créer une définition personnalisée.

### <a name="built-in-policy-definitions-for-azure-monitor"></a>Définitions de stratégie intégrées pour Azure Monitor
Il existe deux définitions de stratégie intégrées pour chaque type de ressource, l’une à envoyer à un espace de travail Log Analytics et l’autre à Event Hub. Si vous n’avez besoin que d’un seul emplacement, affectez cette stratégie pour le type de ressource. Si vous avez besoin des deux, affectez les deux définitions de stratégie pour la ressource.

Par exemple, l’image suivante montre les définitions de stratégie de paramètres de diagnostic intégrées pour Data Lake Analytics.

  ![Capture d’écran partielle de la page de définitions d’Azure Policy montrant deux définitions de stratégie de paramètres de diagnostic intégrées pour Data Lake Analytics.](media/deploy-scale/builtin-diagnostic-settings.png)

### <a name="custom-policy-definitions"></a>Définitions de stratégie personnalisées
Pour les types de ressource qui n’ont pas de stratégie intégrée, vous devez créer une définition de stratégie personnalisée. Vous pouvez le faire manuellement sur le portail Azure en copiant une stratégie intégrée existante et en la modifiant pour votre type de ressource. Il est néanmoins plus efficace de créer la stratégie par programmation en utilisant un script de PowerShell Gallery.

Le script [Create-AzDiagPolicy](https://www.powershellgallery.com/packages/Create-AzDiagPolicy) crée des fichiers de stratégie pour un type de ressource déterminé que vous pouvez installer via PowerShell ou l’interface CLI. Pour créer une définition de stratégie personnalisée pour des paramètres de diagnostic, utilisez la procédure suivante.


1. Vérifiez qu’[Azure PowerShell](/powershell/azure/install-az-ps) est installé.
2. Installez le script avec la commande suivante :
  
    ```azurepowershell
    Install-Script -Name Create-AzDiagPolicy
    ```

3. Exécutez le script en spécifiant la destination des journaux à l’aide des paramètres. Vous êtes ensuite invité à spécifier un abonnement et un type de ressource. Par exemple, pour créer une définition de stratégie qui envoie vers un espace de travail Log Analytics et Event Hub, utilisez la commande suivante.

   ```azurepowershell
   Create-AzDiagPolicy.ps1 -ExportLA -ExportEH -ExportDir ".\PolicyFiles"  
   ```

4. Vous pouvez aussi spécifier un abonnement et un type de ressource dans la commande. Ainsi, pour créer une définition de stratégie qui envoie vers un espace de travail Log Analytics et Event Hub pour des bases de données Azure SQL Server, utilisez la commande suivante.

   ```azurepowershell
   Create-AzDiagPolicy.ps1 -SubscriptionID xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -ResourceType Microsoft.Sql/servers/databases  -ExportLA -ExportEH -ExportDir ".\PolicyFiles"  
   ```

5. Le script crée des dossiers distincts pour chaque définition de stratégie, chacun contenant trois fichiers nommés azurepolicy.json, azurepolicy.rules.json et azurepolicy.parameters.json. Si vous voulez créer la stratégie manuellement sur le portail Azure, vous pouvez copier et coller le contenu du fichier azurepolicy.json, car il contient la définition de stratégie complète. Utilisez les deux autres fichiers avec PowerShell ou l’interface CLI pour créer la définition de stratégie à partir d’une ligne de commande.

    Les exemples suivants montrent comment installer la définition de stratégie à partir de PowerShell et de l’interface CLI. Chacun comprend des métadonnées pour spécifier une catégorie de **supervision** pour regrouper la nouvelle définition de stratégie avec les définitions de stratégie intégrées.

      ```azurepowershell
      New-AzPolicyDefinition -name "Deploy Diagnostic Settings for SQL Server database to Log Analytics workspace" -policy .\Apply-Diag-Settings-LA-Microsoft.Sql-servers-databases\azurepolicy.rules.json -parameter .\Apply-Diag-Settings-LA-Microsoft.Sql-servers-databases\azurepolicy.parameters.json -mode All -Metadata '{"category":"Monitoring"}'
      ```

      ```azurecli
      az policy definition create --name 'deploy-diag-setting-sql-database--workspace' --display-name 'Deploy Diagnostic Settings for SQL Server database to Log Analytics workspace'  --rules 'Apply-Diag-Settings-LA-Microsoft.Sql-servers-databases\azurepolicy.rules.json' --params 'Apply-Diag-Settings-LA-Microsoft.Sql-servers-databases\azurepolicy.parameters.json' --subscription 'AzureMonitor_Docs' --mode All
      ```

### <a name="initiative"></a>Initiative
Au lieu de créer une affectation pour chaque définition de stratégie, une stratégie courante consiste à créer une initiative incluant les définitions de stratégie pour créer des paramètres de diagnostic pour chaque service Azure. Créez une affectation entre l’initiative et un groupe d’administration, un abonnement ou un groupe de ressources en fonction de la façon dont vous gérez votre environnement. Cette stratégie offre les avantages suivants :

- Créez une seule affectation pour l’initiative au lieu de plusieurs pour chaque type de ressource. Utilisez la même initiative pour plusieurs groupes de supervision, abonnements ou groupes de ressources.
- Modifiez l’initiative quand vous devez ajouter un nouveau type de ressource ou une nouvelle destination. Par exemple, vos exigences initiales peuvent être d’envoyer les données uniquement vers un espace de travail Log Analytics et d’ajouter par la suite Event Hub. Au lieu de créer des affectations, modifiez l’initiative.

Pour plus d’informations sur la création d’une initiative, consultez [Créer et attribuer une définition d’initiative](../governance/policy/tutorials/create-and-manage.md#create-and-assign-an-initiative-definition). Tenez compte des recommandations suivantes :

- Définissez la **Catégorie**sur **Supervision** pour la regrouper avec les définitions de stratégie intégrées et personnalisées associées.
- Au lieu de spécifier les détails de l’espace de travail Log Analytics et du hub d’événements pour la définition de stratégie incluse dans l’initiative, utilisez un paramètre d’initiative commun. Vous pourrez ainsi spécifier facilement une valeur commune pour toutes les définitions de stratégie et changer cette valeur si nécessaire.

![Définition d’initiative](media/deploy-scale/initiative-definition.png)

### <a name="assignment"></a>Affectation 
Affectez l’initiative à un groupe d’administration, un abonnement ou un groupe de ressources Azure en fonction de l’étendue de vos ressources à superviser. Un [groupe d’administration ](../governance/management-groups/overview.md) est particulièrement utile pour définir l’étendue de la stratégie, surtout si votre organisation possède plusieurs abonnements.

![Capture d’écran des paramètres de l’onglet De base dans la section Affecter une initiative des paramètres de diagnostic pour l’espace de travail Log Analytics dans le Portail Azure.](media/deploy-scale/initiative-assignment.png)

Les paramètres initiative vous permettent de spécifier l’espace de travail ou d’autres détails une seule fois pour toutes les définitions de stratégie de l’initiative. 

![Paramètres d’initiative](media/deploy-scale/initiative-parameters.png)

### <a name="remediation"></a>Correction
L’initiative s’applique à chaque machine virtuelle à mesure qu’elle est créée. Une [tâche de correction](../governance/policy/how-to/remediate-resources.md) déploie les définitions de stratégie de l’initiative dans les ressources existantes, ce qui vous permet de créer des paramètres de diagnostic pour toutes les ressources qui ont déjà été créées. Quand vous créez l’affectation à partir du portail Azure, vous avez la possibilité de créer une tâche de correction en même temps. Pour plus d’informations sur la correction, consultez [Corriger les ressources non conformes avec Azure Policy](../governance/policy/how-to/remediate-resources.md).

![Correction d’initiative](media/deploy-scale/initiative-remediation.png)


## <a name="azure-monitor-for-vms"></a>Azure Monitor pour machines virtuelles
[Azure Monitor pour machines virtuelles](insights/vminsights-overview.md) est le principal outil de supervision des machines virtuelles d’Azure Monitor. L’activation d’Azure Monitor pour machines virtuelles a pour effet d’installer l’agent Log Analytics et Dependency Agent. Au lieu d’effectuer ces tâches manuellement, utilisez Azure Policy pour faire en sorte que chaque machine virtuelle soit configurée à mesure que vous la créez.

> [!NOTE]
> Azure Monitor pour machines virtuelles inclut une fonctionnalité appelée **Couverture de stratégie Azure Monitor pour machines virtuelles**, qui permet de découvrir et de corriger les machines virtuelles non conformes dans votre environnement. Vous pouvez utiliser cette fonctionnalité plutôt que de travailler directement avec Azure Policy pour les machines virtuelles Azure et les machines virtuelles hybrides connectées à Azure Arc. Pour les groupes de machines virtuelles identiques Azure, vous devez créer l’affectation à l’aide d’Azure Policy.
 

Azure Monitor pour machines virtuelles comprend les initiatives intégrées suivantes ,qui installent les deux agents pour activer la surveillance complète. 

|Nom |Description |
|:---|:---|
|Activer Azure Monitor pour machines virtuelles | Installe l’agent Log Analytics et l’agent Dependency sur les machines virtuelles Azure et les machines virtuelles hybrides connectées à Azure Arc. |
|Activer Azure Monitor pour les groupes de machines virtuelles identiques | Ceci permet d’installer l'agent Log Analytics et l’agent Dependency sur des groupes de machines virtuelles identiques Azure. |


### <a name="virtual-machines"></a>Machines virtuelles
Au lieu de créer des affectations pour ces initiatives à l’aide de l’interface Azure Policy, Azure Monitor pour machines virtuelles est doté d’une fonctionnalité qui vous permet d’inspecter le nombre de machines virtuelles dans chaque étendue pour déterminer si l’initiative a été appliquée. Vous pouvez ensuite configurer l’espace de travail et créer les affectations nécessaires à l’aide de cette interface.

Pour obtenir des détails sur ce processus, consultez [Activer Azure Monitor pour machines virtuelles VM à l’aide d’Azure Policy](./insights/vminsights-enable-policy.md).

![Stratégie Azure Monitor pour machines virtuelles](media/deploy-scale/vminsights-policy.png)

### <a name="virtual-machine-scale-sets"></a>Groupes identiques de machines virtuelles
Pour utiliser Azure Policy dans le but d’activer l’analyse de groupes de machines virtuelles identiques, attribuez l’initiative **Activer Azure Monitor pour les groupes de machines virtuelles identiques** à un groupe d’administration, un abonnement ou un groupe de ressources Azure, en fonction de l’étendue de vos ressources à surveiller. Un [groupe d’administration ](../governance/management-groups/overview.md) est particulièrement utile pour définir l’étendue de la stratégie, surtout si votre organisation possède plusieurs abonnements.

![Capture d’écran de la page Affecter une initiative dans le Portail Azure. La définition d’initiative est définie sur Activer Azure Monitor pour les groupes de machines virtuelles identiques.](media/deploy-scale/virtual-machine-scale-set-assign-initiative.png)

Sélectionnez l’espace de travail auquel les données seront envoyées. La solution *VMInsights* doit avoir été installée dans cet espace de travail comme décrit dans []().

![Sélectionner un espace de travail](media/deploy-scale/virtual-machine-scale-set-workspace.png)

Créez une tâche de correction si vous devez affecter cette stratégie à un groupe de machines virtuelles identiques existant.

![Tâche de correction](media/deploy-scale/virtual-machine-scale-set-remediation.png)

### <a name="log-analytics-agent"></a>Agent Log Analytics
Vous pouvez avoir des scénarios dans lesquels vous souhaitez installer l’agent Log Analytics, mais pas l’agent Dependency. Il n’existe pas d’initiative intégrée pour le seul agent, mais vous pouvez créer votre propre initiative sur la base des définitions de stratégie intégrées fournies par Azure Monitor pour machines virtuelles.

> [!NOTE]
> Il n’y a aucune raison de déployer l’agent Dependency seul, car il requiert que l’agent Log Analytics fournisse ses données à Azure Monitor.


|Nom |Description |
|-----|------------|
|Auditer le déploiement de l’agent Log Analytics - Image de machine virtuelle (système d’exploitation) non listée |Présente les machines virtuelles comme non conformes si l’image de machine virtuelle (système d’exploitation) n’est pas définie dans la liste et si l’agent n’est pas installé. |
|Déployer l’agent Log Analytics pour les machines virtuelles Linux |Déployez l’agent Log Analytics pour les machines virtuelles Linux si l’image de machine virtuelle (système d’exploitation) est définie dans la liste et si l’agent n’est pas installé. |
|Déployer l’agent Log Analytics pour les machines virtuelles Windows |Déployez l’agent Log Analytics pour les machines virtuelles Windows si l’image de machine virtuelle (système d’exploitation) est définie dans la liste et si l’agent n’est pas installé. |
| [Préversion] : L’agent Log Analytics doit être installé sur vos machines Azure Arc Linux |Signale les machines Azure Arc hybrides comme non conformes pour les machines virtuelles Linux si l’image de machine virtuelle (système d’exploitation) est définie dans la liste et que l’agent n’est pas installé. |
| [Préversion] : L’agent Log Analytics doit être installé sur vos machines Azure Arc Windows |Signale les machines Azure Arc hybrides comme non conformes pour les machines virtuelles Windows si l’image de machine virtuelle (système d’exploitation) est définie dans la liste et que l’agent n’est pas installé. |
| [Préversion] : Déployer l’agent Log Analytics sur des machines Azure Arc Linux |Déployez l’agent Log Analytics pour les machines Azure Arc hybrides Linux si l’image de machine virtuelle (système d’exploitation) est définie dans la liste et que l’agent n’est pas installé. |
| [Préversion] : Déployer l’agent Log Analytics sur des machines Azure Arc Windows |Déployez l’agent Log Analytics pour les machines Azure Arc hybrides Windows si l’image de machine virtuelle (système d’exploitation) est définie dans la liste et que l’agent n’est pas installé. |
|Auditer le déploiement de l’agent de dépendances dans des groupes de machines virtuelles identiques - Image de machine virtuelle (système d’exploitation) non listée |Présente le groupe de machines virtuelles identiques comme non conforme si l’image de machine virtuelle (système d’exploitation) n’est pas définie dans la liste et si l’agent n’est pas installé. |
|Auditer le déploiement de l’agent Log Analytics dans des groupes de machines virtuelles identiques - Image de machine virtuelle (système d’exploitation) non listée |Présente le groupe de machines virtuelles identiques comme non conforme si l’image de machine virtuelle (système d’exploitation) n’est pas définie dans la liste et si l’agent n’est pas installé. |
|Déployer l’agent Log Analytics pour les groupes de machines virtuelles identiques Linux |Déployez l’agent Log Analytics pour les groupes de machines virtuelles identiques Linux si l’image de machine virtuelle (système d’exploitation) est définie dans la liste et si l’agent n’est pas installé. |
|Déployer l’agent Log Analytics pour les groupes de machines virtuelles identiques Windows |Déployez l’agent Log Analytics pour les groupes de machines virtuelles identiques Windows si l’image de machine virtuelle (système d’exploitation) est définie dans la liste et si l’agent n’est pas installé. |


## <a name="next-steps"></a>Étapes suivantes

- Découvrir plus en détail [Azure Policy](../governance/policy/overview.md).
- Découvrir plus en détail les [paramètres de diagnostic](platform/diagnostic-settings.md).