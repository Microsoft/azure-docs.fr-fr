---
title: Utiliser l’analyse des changements applicatifs dans Azure Monitor pour rechercher les problèmes liés aux applications web | Microsoft Docs
description: Utilisez l’analyse des changements applicatifs dans Azure Monitor pour résoudre les problèmes liés aux applications sur des sites actifs sur Azure App Service.
ms.topic: conceptual
author: cawams
ms.author: cawa
ms.date: 05/04/2020
ms.openlocfilehash: 36e73ceddaa5e3f9cbbf4a41f76a4ba6d70eed0f
ms.sourcegitcommit: f311f112c9ca711d88a096bed43040fcdad24433
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94979961"
---
# <a name="use-application-change-analysis-preview-in-azure-monitor"></a>Utilise l’analyse des changements applicatifs (préversion) dans Azure Monitor

Lorsqu’un problème ou une panne se produit sur un site actif, il est essentiel de déterminer rapidement la cause racine. Les solutions de contrôle standard peuvent vous signaler un problème. Elles peuvent même indiquer le composant en cause. Mais cette alerte ne vous expliquera pas toujours immédiatement la cause du problème. Vous savez que votre site fonctionnait il y a cinq minutes et que maintenant il ne marche plus. Qu’est-ce qui a changé au cours des cinq dernières minutes ? C’est la question que l’analyse des changements applicatifs est conçue pour répondre dans Azure Monitor.

Reposant sur la puissance [d’Azure Resource Graph](../../governance/resource-graph/overview.md), l’analyse des changements applicatifs fournit des informations sur vos modifications à l’application Azure pour améliorer la capacité d’observation et réduire le temps moyen de réparation.

> [!IMPORTANT]
> L’analyse des changements applicatifs est actuellement en préversion. Cette préversion est fournie sans contrat de niveau de service. Cette version n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="overview"></a>Vue d’ensemble

L’analyse des changements détecte les différents types de modifications, de la couche d’infrastructure au déploiement des applications. C’est un fournisseur de ressources Azure de niveau d’abonnement qui vérifie les modifications des ressources dans l’abonnement. L’analyse des changements fournit les données de divers outils de diagnostic pour aider les utilisateurs à comprendre les modifications susceptibles d’avoir entraîné des problèmes.

Le diagramme suivant illustre l’architecture de l’analyse des changements :

![Diagramme d’architecture de la façon dont l’analyse des changements obtient les données modifiées et les fournit aux outils clients](./media/change-analysis/overview.png)

## <a name="data-sources"></a>Sources de données

L’analyse des changements d’application interroge les propriétés suivies d’Azure Resource Manager, les configurations proxysées et les modifications apportées à une application web dans l’invité. De plus, le service effectue le suivi des modifications sur les dépendances de ressource pour diagnostiquer et superviser une application de bout en bout.

### <a name="azure-resource-manager-tracked-properties-changes"></a>Modifications des propriétés suivies Azure Resource Manager

À l’aide [d’Azure Resource Graph](../../governance/resource-graph/overview.md), l’analyse des changements fournit un historique de la façon dont les ressources Azure qui hébergent votre application ont été modifiées au fil du temps. Les paramètres suivis tels que les identités managées, la mise à niveau du système d’exploitation de la plateforme et les noms d’hôte peuvent être détectés.

### <a name="azure-resource-manager-proxied-setting-changes"></a>Modifications des paramètres de proxy Azure Resource Manager

Les paramètres, tels que la règle Configuration IP, les paramètres TLS et les versions d’extension ne sont pas encore disponibles dans Azure Resource Graph. Ainsi, l’analyse des changements interroge et calcule ces modifications de manière sécurisée pour fournir plus de détails sur ce qui a changé dans l’application.

### <a name="changes-in-web-app-deployment-and-configuration-in-guest-changes"></a>Modifications dans le déploiement et la configuration des applications web (dans l’invité)

L’analyse des changements capture l’état de déploiement et la configuration d’une application toutes les 4 heures. Elle peut détecter, par exemple, les modifications dans les variables d’environnement de l’application. L’outil calcule les différences et présente ce qui a changé. Contrairement aux modifications de Resource Manager, les informations de modification de déploiement de code peuvent ne pas être disponibles immédiatement dans l’outil. Pour afficher les toutes dernières modifications dans l’analyse des changements, sélectionnez **Actualiser**.

![Capture d’écran du bouton « Analyser les changements »](./media/change-analysis/scan-changes.png)

### <a name="dependency-changes"></a>Changements de dépendances

Les modifications apportées aux dépendances de ressources peuvent également entraîner des problèmes dans une application web. Par exemple, si une application web appelle un cache Redis, la référence SKU du cache Redis peut affecter les performances de l’application web. Pour détecter les modifications dans les dépendances, l’analyse des changements vérifie les enregistrements DNS de l’application web. De cette façon, elle identifie les modifications dans tous les composants d’application qui peuvent entraîner des problèmes.
Actuellement, les dépendances suivantes sont prises en charge :
- Web Apps
- Stockage Azure
- Azure SQL

## <a name="application-change-analysis-service"></a>Service d’analyse des changements d’application

Le service d’analyse des changements d’application calcule et agrège les données modifiées à partir des sources de données mentionnées ci-dessus. Il fournit un ensemble d’analytiques permettant aux utilisateurs de parcourir facilement toutes les modifications apportées aux ressources, et d’identifier quelle modification est pertinente dans le contexte de dépannage ou de supervision.
Le fournisseur de ressources « Microsoft.ChangeAnalysis » doit être inscrit avec un abonnement pour que les données sur les modifications des propriétés suivies et des paramètres de proxy Azure Resource Manager soient disponibles. Quand vous entrez dans l’outil Diagnostiquer et résoudre les problèmes de l’application web ou que vous affichez l’onglet autonome Analyse des changements, ce fournisseur de ressources est automatiquement inscrit. Il est sans incidence sur votre abonnement pour ce qui est des performances ou des coûts. Quand vous activez Analyse des changements pour les applications web (ou que vous activez l’outil Diagnostiquer et résoudre les problèmes), l’opération a un impact négligeable sur les performances de l’application web et n’entraîne aucun coût de facturation.
Pour les modifications d’une application web dans l’invité, une activation distincte est requise pour l’analyse des fichiers de code au sein de l’application web. Pour plus d’informations, consultez la section [Analyse des changements dans l’outil Diagnostiquer et résoudre les problèmes](#application-change-analysis-in-the-diagnose-and-solve-problems-tool) plus loin dans cet article.

## <a name="visualizations-for-application-change-analysis"></a>Visualisations de l’analyse des changements d’application

### <a name="standalone-ui"></a>Interface utilisateur autonome

Il existe dans Azure Monitor un volet autonome pour l’Analyse des changements qui permet d’afficher toutes les modifications avec des insights dans les ressources et les dépendances de l’application.

Recherchez Analyse des changements dans la barre de recherche du portail Azure pour lancer l’expérience.

![Capture d’écran de la recherche de l’Analyse des changements sur le Portail Azure](./media/change-analysis/search-change-analysis.png)

Toutes les ressources sous un abonnement sélectionné s’affichent avec les modifications effectuées au cours des dernières 24 heures. Pour optimiser les performances de chargement de la page, le service affiche 10 ressources à la fois. Cliquez sur les pages suivantes pour afficher plus de ressources. Nous travaillons actuellement à la suppression de cette limitation.

![Capture d’écran du panneau Analyse des changements sur le Portail Azure](./media/change-analysis/change-analysis-standalone-blade.png)

Cliquez sur une ressource pour afficher toutes ses modifications. Au besoin, explorez une modification pour afficher les détails et les insights de cette modification au format json.

![Capture d’écran des détails de la modification](./media/change-analysis/change-details.png)

Pour tout commentaire, utilisez le bouton Envoyer des commentaires dans le panneau ou envoyez un e-mail à l’adresse changeanalysisteam@microsoft.com.

![Capture d’écran du bouton Commentaires dans le panneau Analyse des changements](./media/change-analysis/change-analysis-feedback.png)

### <a name="web-app-diagnose-and-solve-problems"></a>Diagnostiquer et résoudre les problèmes d’application web

Dans Azure Monitor, l’Analyse des changements est également intégrée à l’expérience en libre-service **Diagnostiquer et résoudre les problèmes**. Accédez à cette expérience à partir de la page **Vue d’ensemble** de votre application App Service.

![Capture d’écran du bouton « Vue d’ensemble » et du bouton « Diagnostiquer et résoudre les problèmes »](./media/change-analysis/change-analysis.png)

### <a name="application-change-analysis-in-the-diagnose-and-solve-problems-tool"></a>Analyse des changements d’application dans l’outil Diagnostiquer et résoudre les problèmes

L’analyse des changements d’application est un détecteur autonome dans les outils de diagnostic et de résolution des problèmes d’application web. Il est également agrégé dans **Blocages de l’application** et les **détecteurs d’application web arrêtée**. À mesure que vous entrez dans l’outil Diagnostiquer et résoudre les problèmes, le fournisseur de ressources **Microsoft.ChangeAnalysis** est automatiquement enregistré. Suivez ces instructions pour activer le suivi des modifications d’une application web dans l’invité.

1. Sélectionnez **Disponibilité et performances**.

    ![Capture d’écran des options de résolution des problèmes « Disponibilité et performances »](./media/change-analysis/availability-and-performance.png)

2. Cliquez sur **Changements d’application**. Cette fonctionnalité est également disponible dans **Blocages de l’application**.

   ![Capture d’écran du bouton « Blocage de l’application »](./media/change-analysis/application-changes.png)

3. Le lien dirige vers l’interface utilisateur Analyse des changements d’application à l’échelle de l’application web. Si le suivi des modifications dans l’invité de l’application web n’est pas activé, suivez la bannière pour accéder aux modifications apportées aux paramètres de fichier et d’application.

   ![Capture d’écran des options de « Blocage de l’application »](./media/change-analysis/enable-changeanalysis.png)

4. Activez **Analyse des changements** et sélectionnez **Enregistrer**. L’outil affiche toutes les applications web dans le cadre d’un plan App Service. Vous pouvez utiliser le commutateur de niveau de plan afin d’activer l’analyse des modifications pour toutes les applications web dans le cadre d’un plan.

    ![Capture d’écran de l’interface utilisateur « Activer l’analyse des changements »](./media/change-analysis/change-analysis-on.png)

5. Les données modifiées sont également disponibles dans les détecteurs d’**application web arrêtée** et **d’incident de l’application**. Vous verrez un graphique qui résume les types de modifications au fil du temps ainsi que des détails sur ces modifications. Par défaut, les modifications effectuées au cours des dernières 24 heures s’affichent pour vous aider à résoudre des problèmes immédiats.

     ![Capture d’écran de la vue de comparaison des modifications](./media/change-analysis/change-view.png)



### <a name="virtual-machine-diagnose-and-solve-problems"></a>Diagnostiquer et résoudre les problèmes d’une machine virtuelle

Accédez à l’outil Diagnostiquer et résoudre les problèmes d’une machine virtuelle.  Accédez à **Outils de résolution des problèmes**, parcourez la page et sélectionnez **Analyser les changements récents** pour afficher les changements sur la machine virtuelle.

![Capture d’écran de Diagnostiquer et résoudre les problèmes d’une machine virtuelle](./media/change-analysis/vm-dnsp-troubleshootingtools.png)

![Analyseur des changements dans les outils de dépannage](./media/change-analysis/analyze-recent-changes.png)

### <a name="activity-log-change-history"></a>Journal d’activité - Historique des modifications
La fonctionnalité [Affichage de l’historique des modifications](../platform/activity-log.md#view-change-history) dans le Journal d’activité demande au serveur principal du service Analyse des changements d’application d’obtenir les modifications associées à une opération. L’**historique des modifications** appelait directement [Azure Resource Graph](../../governance/resource-graph/overview.md), mais demandait au serveur principal d’appeler Analyse des changements d’application de sorte que les modifications retournées incluent les modifications au niveau des ressources d’[Azure Resource Graph](../../governance/resource-graph/overview.md), les propriétés de ressource d’[Azure Resource Manager](../../azure-resource-manager/management/overview.md) et les modifications dans l’invité de services PaaS tels que l’application web App Services. Pour que le service Analyse des changements d’application puisse analyser les modifications apportées aux abonnements des utilisateurs, un fournisseur de ressources doit être inscrit. La première fois que vous accédez à l’onglet **Historique des modifications**, l’outil commence automatiquement à inscrire le fournisseur de ressources **Microsoft.ChangeAnalysis**. Une fois inscrit, les modifications d’**Azure Resource Graph** sont immédiatement disponibles et couvrent les 14 derniers jours. Les modifications d’autres sources sont disponibles 4 heures environ après l’intégration de l’abonnement.

![Journal d’activité : Intégration de l’historique des modifications](./media/change-analysis/activity-log-change-history.png)

### <a name="vm-insights-integration"></a>Intégration de VM Insights
Les utilisateurs disposant de [VM Insights](../insights/vminsights-overview.md) activé peuvent voir ce qui a changé dans leurs machines virtuelles, ce qui peut entraîner des pics dans un graphique de métriques, comme le processeur ou la mémoire, et se demander ce qui l’a provoqué. Les données modifiées sont intégrées à la barre de navigation côté VM Insights. L’utilisateur peut voir si des modifications sont apportées à la machine virtuelle, puis cliquer sur **Examiner les modifications** pour afficher les détails des modifications dans l’interface utilisateur autonome d’Analyse des changements d’application.

[![Intégration de VM Insights](./media/change-analysis/vm-insights.png)](./media/change-analysis/vm-insights.png#lightbox)



## <a name="enable-change-analysis-at-scale"></a>Activez l’analyse des changements à l’échelle

Si votre abonnement inclut de nombreuses applications web, l’activation du service pour chaque application web serait inefficace. Exécutez le script suivant pour activer toutes les applications web dans votre abonnement.

Conditions préalables :

- Module PowerShell Az. Suivez les instructions dans la rubrique[Installer le module Azure PowerShell](/powershell/azure/install-az-ps)

Exécutez le script suivant :

```PowerShell
# Log in to your Azure subscription
Connect-AzAccount

# Get subscription Id
$SubscriptionId = Read-Host -Prompt 'Input your subscription Id'

# Make Feature Flag visible to the subscription
Set-AzContext -SubscriptionId $SubscriptionId

# Register resource provider
Register-AzResourceProvider -ProviderNamespace "Microsoft.ChangeAnalysis"

# Enable each web app
$webapp_list = Get-AzWebApp | Where-Object {$_.kind -eq 'app'}
foreach ($webapp in $webapp_list)
{
    $tags = $webapp.Tags
    $tags[“hidden-related:diagnostics/changeAnalysisScanEnabled”]=$true
    Set-AzResource -ResourceId $webapp.Id -Tag $tags -Force
}

```

## <a name="troubleshoot"></a>Dépanner

### <a name="having-trouble-registering-microsoftchange-analysis-resource-provider-from-change-history-tab"></a>Problèmes d’inscription du fournisseur de ressources Microsoft.Change Analysis à partir de l’onglet Historique des modifications
Si vous affichez l’historique des modifications pour la première fois après son intégration à Analyse des changements d’application, vous verrez qu’il inscrit automatiquement un fournisseur de ressources **Microsoft.ChangeAnalysis**. Dans de rares cas, cela peut échouer pour les raisons suivantes :

- **Vous ne disposez pas d’autorisations suffisantes pour inscrire le fournisseur de ressources Microsoft.ChangeAnalysis**. Ce message d’erreur signifie que votre rôle dans l’abonnement actuel n’est pas associé à l’étendue **Microsoft.Support/register/action**. Cela peut se produire si vous n’êtes pas le propriétaire d’un abonnement et que vous disposez d’autorisations d’accès partagé par le biais d’un collègue. Par exemple, accès en lecture à un groupe de ressources. Pour résoudre ce problème, vous pouvez contacter le propriétaire de votre abonnement pour inscrire le fournisseur de ressources **Microsoft.ChangeAnalysis**. Vous pouvez le faire dans le portail Azure via **Abonnements | Fournisseurs de ressources**, en recherchant ```Microsoft.ChangeAnalysis``` et en l’inscrivant dans l’interface utilisateur ou via Azure PowerShell ou Azure CLI.

    Inscrire le fournisseur de ressources par le biais de PowerShell : 
    ```PowerShell
    # Register resource provider
    Register-AzResourceProvider -ProviderNamespace "Microsoft.ChangeAnalysis"
    ```

- **Échec de l’inscription du fournisseur de ressources Microsoft.ChangeAnalysis**. Ce message signifie que l’opération a échoué immédiatement quand l’interface utilisateur a envoyé la demande d’inscription du fournisseur de ressources, et que ce problème n’est pas lié à un problème d’autorisation. Il peut s’agir d’un problème de connectivité à Internet temporaire. Essayez d’actualiser la page et de vérifier votre connexion Internet. Si l’erreur persiste, contactez changeanalysishelp@microsoft.com
- **Échec de l’interrogation du fournisseur de ressources Microsoft.ChangeAnalysis** avec le message *L’abonnement à Azure Lighthouse n’est pas pris en charge, les modifications ne sont disponibles que dans le locataire d’accueil de l’abonnement*. Il existe actuellement une limite pour le fournisseur de ressources Analyse des changements pour s’inscrire par le biais d’un abonnement à Azure Lighthouse pour les utilisateurs qui ne sont pas dans le locataire d’accueil. Nous pensons que cette limite sera résolue dans un avenir proche. Si ce problème vous bloque, il existe une solution de contournement qui implique la création d’un principal de service et l’attribution explicite du rôle pour autoriser l’accès.  Pour en savoir plus, contactez changeanalysishelp@microsoft.com.

- **Cela prend plus de temps que prévu.** Ce message signifie que l’inscription prend plus de 2 minutes. Cela est inhabituel, mais cela ne signifie pas nécessairement qu’une erreur s’est produite. Vous pouvez accéder à **Abonnements | Fournisseur de ressources** pour vérifier l’état d’inscription du fournisseur de ressources **Microsoft.ChangeAnalysis**. Vous pouvez essayer d’utiliser l’interface utilisateur pour annuler l’inscription, procéder à une nouvelle inscription ou actualiser pour voir si cela vous aide. Si le problème persiste, contactez changeanalysishelp@microsoft.com pour obtenir de l’aide.
    ![Résolution des problèmes d’inscription de RP trop longue](./media/change-analysis/troubleshoot-registration-taking-too-long.png)

![Capture d’écran de l’outil Diagnostiquer et résoudre les problèmes d’une machine virtuelle avec Outils de dépannage sélectionné.](./media/change-analysis/vm-dnsp-troubleshootingtools.png)

![Capture d’écran de la vignette de l’outil de dépannage Analyser les changements récents d’une machine virtuelle.](./media/change-analysis/analyze-recent-changes.png)

## <a name="next-steps"></a>Étapes suivantes

- Activez application Insights pour les [applications Azure App Services](azure-web-apps.md).
- Activez Application Insights pour les [applications hébergées par IIS sur une machine virtuelle Azure et un groupe de machines virtuelles identiques Azure](azure-vm-vmss-apps.md).
- Apprenez-en davantage sur [Azure Resource Graph](../../governance/resource-graph/overview.md), qui vous aide à exécuter l’analyse des changements.