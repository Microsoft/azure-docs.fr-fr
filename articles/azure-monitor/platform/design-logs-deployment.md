---
title: Conception de votre déploiement de journaux Azure Monitor | Microsoft Docs
description: Cet article décrit les considérations et les recommandations pour les clients qui préparent le déploiement d’un espace de travail dans Azure Monitor.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 09/20/2019
ms.openlocfilehash: 034f2b3884d732487a9f7aff4d14740691983885
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95536776"
---
# <a name="designing-your-azure-monitor-logs-deployment"></a>Conception de votre déploiement de journaux Azure Monitor

Azure Monitor stocke des données de [journal](data-platform-logs.md) dans un espace de travail Log Analytics, qui est une ressource Azure et un conteneur où les données sont collectées, agrégées, et qui sert de limite d’administration. Bien que vous puissiez déployer un ou plusieurs espaces de travail dans votre abonnement Azure, vous devez prendre en considération plusieurs éléments pour vous assurer que votre déploiement initial est conforme à nos recommandations pour vous fournir une solution économique, gérable et évolutive qui répond aux besoins de votre organisation.

Les données d’un espace de travail sont organisées en tables, chacune d’entre-elles stockant différents genres de données et possédant son propre ensemble de propriétés, basé sur la ressource qui génère les données. La plupart des sources de données écrivent dans leurs propres tables dans un espace de travail Log Analytics.

![Exemple de modèle de données d’espace de travail](./media/design-logs-deployment/logs-data-model-01.png)

Un espace de travail Log Analytics offre :

* Un emplacement géographique pour le stockage des données.
* L’isolation des données en accordant à différents utilisateurs des droits d’accès suivant l’une de nos stratégies de conception recommandées.
* Une étendue pour la configuration des paramètres, comme le [niveau tarifaire](./manage-cost-storage.md#changing-pricing-tier), la [rétention](./manage-cost-storage.md#change-the-data-retention-period) et la [limitation des données](./manage-cost-storage.md#manage-your-maximum-daily-data-volume).

Les espaces de travail sont hébergés sur des clusters physiques. Par défaut, le système crée et gère ces clusters. Les clients qui ingèrent plus de 4 To/jour sont censés créer leurs propres clusters dédiés pour leurs espaces de travail, ce qui leur offre un meilleur contrôle et un taux d’ingestion supérieur.

Cet article fournit une présentation détaillée des considérations relatives à la conception et à la migration, une présentation du contrôle d’accès ainsi que des implémentations de conception recommandées pour votre service informatique.



## <a name="important-considerations-for-an-access-control-strategy"></a>Considérations importantes concernant une stratégie de contrôle d’accès

L’identification du nombre d’espaces de travail dont vous avez besoin dépend d’une ou de plusieurs des exigences suivantes :

* Vous travaillez pour une entreprise internationale et vous avez besoin de stocker vos données de journal dans des régions spécifiques pour des raisons de conformité ou de souveraineté des données.
* Vous utilisez Azure et vous souhaitez éviter les frais liés au transfert de données sortantes en configurant un espace de travail dans la même région que les ressources Azure qu’il gère.
* Vous gérez plusieurs services ou groupes d’entreprise, et vous souhaitez que chacun affiche ses propres données, mais pas les données des autres. De plus, vous n’avez pas besoin d’vue consolidée des services ou groupes d’entreprises.

Aujourd’hui, les services informatiques sont modélisés à l’aide d’un modèle centralisé, décentralisé ou hybride. Par conséquent, les modèles de déploiement d’espace de travail suivants ont été couramment utilisés pour mapper l’une de ces structures organisationnelles :

* **Centralisé** : tous les journaux sont stockés dans un espace de travail central et administrés par une seule équipe, Azure Monitor fournissant un accès différencié par équipe. Dans ce scénario, il est facile de réaliser des opérations de gestion, de recherche dans les ressources et de corrélation entre les journaux. L’espace de travail peut croître considérablement en fonction de la quantité de données collectées à partir de plusieurs ressources de votre abonnement, avec une charge administrative supplémentaire pour maintenir le contrôle d’accès à différents utilisateurs. Ce modèle est appelé « Hub-and-spoke ».
* **Décentralisé** : chaque équipe a son propre espace de travail créé dans un groupe de ressources qu’elle possède et gère, et les données de journalisation sont séparées par ressource. Dans ce scénario, l’espace de travail peut rester sécurisé et le contrôle d’accès est cohérent avec l’accès aux ressources, mais il est difficile d’effectuer des corrélations entre les journaux. Les utilisateurs qui ont besoin d’une vue étendue de nombreuses ressources ne peuvent pas analyser les données de manière explicite.
* **Hybride** : les exigences de conformité de l’audit de sécurité compliquent encore ce scénario, car de nombreuses organisations implémentent les deux modèles de déploiement en parallèle. Cela aboutit généralement à une configuration complexe, coûteuse et difficile à gérer, avec des lacunes dans la couverture des journaux.

Quand vous utilisez les agents Log Analytics pour collecter des données, vous devez comprendre ce qui suit pour planifier le déploiement de votre agent :

* Pour collecter des données à partir d’agents Windows, vous pouvez [configurer chaque agent pour qu’il rend compte à un ou plusieurs espaces de travail](./agent-windows.md), même s’il rend compte à un groupe d’administration System Center Operations Manager. L’agent Windows peut rendre compte à quatre espaces de travail au maximum.
* L’agent Linux ne prend pas en charge l’hébergement multiple et peut uniquement rendre compte à un seul espace de travail.

Si vous utilisez System Center Operations Manager 2012 R2 ou une version ultérieure :

* Chaque groupe d’administration Operations Manager peut être [connecté à un seul espace de travail](./om-agents.md). 
* Les ordinateurs Linux rendant compte à un groupe d’administration doivent être configurés pour rendre compte directement à un espace de travail Log Analytics. Si vos ordinateurs Linux rendent déjà compte directement à un espace de travail et que vous souhaitez les superviser avec Operations Manager, effectuez les étapes suivantes pour [qu’ils rendent compte à un groupe d’administration Operations Manager](agent-manage.md#configure-agent-to-report-to-an-operations-manager-management-group). 
* Vous pouvez installer l’agent Windows Log Analytics sur l’ordinateur Windows et faire en sorte qu’il rende compte à la fois à Operations Manager intégré à un espace de travail et à un autre espace de travail.

## <a name="access-control-overview"></a>Présentation du contrôle d’accès

Avec le contrôle d’accès en fonction du rôle Azure (Azure RBAC), vous pouvez accorder aux utilisateurs et aux groupes uniquement le type d’accès dont ils ont besoin pour travailler avec les données de surveillance dans un espace de travail. Vous pouvez ainsi vous aligner sur le modèle d’exploitation de votre service informatique en utilisant un espace de travail unique pour stocker les données collectées activées sur toutes vos ressources. Par exemple, vous accordez l’accès à votre équipe responsable des services d’infrastructure hébergés sur des machines virtuelles Azure et par conséquent, les membres de l’équipe ont accès uniquement aux journaux générés par les machines virtuelles. Le nouveau modèle de journal Contexte de ressource est le suivant. La base de ce modèle est, pour chaque enregistrement de journal émis par une ressource Azure, qu’il est automatiquement associé à cette ressource. Les journaux sont transférés vers un espace de travail central qui respecte la délimitation et Azure RBAC, en fonction des ressources.

Les données auxquelles un utilisateur a accès sont déterminées par plusieurs facteurs, listés dans le tableau suivant. Chacun est décrit dans les sections ci-dessous.

| Factor | Description |
|:---|:---|
| [Mode d’accès](#access-mode) | Méthode utilisée par l’utilisateur pour accéder à l’espace de travail.  Définit l’étendue des données disponibles et le mode de contrôle d’accès qui est appliqué. |
| [Mode de contrôle d’accès](#access-control-mode) | Paramètre de l’espace de travail qui détermine si des autorisations sont appliquées au niveau de la ressource ou de l’espace de travail. |
| [autorisations](manage-access.md) | Autorisations appliquées à des utilisateurs spécifiques ou à des groupes d’utilisateurs pour l’espace de travail ou la ressource. Définit les données auxquelles l’utilisateur a accès. |
| [Azure RBAC au niveau de la table](manage-access.md#table-level-azure-rbac) | Autorisations granulaires facultatives qui s’appliquent à tous les utilisateurs, quel que soit leur mode d’accès ou mode de contrôle d’accès. Définit les types de données auxquels un utilisateur peut accéder. |

## <a name="access-mode"></a>Mode d’accès

Le *mode d’accès* fait référence à la façon dont un utilisateur accède à un espace de travail Log Analytics et définit l’étendue des données auxquelles il peut accéder. 

Les utilisateurs disposent de deux options pour accéder aux données :

* **Contexte d’espace de travail** : vous pouvez afficher tous les journaux dans l’espace de travail pour lequel vous disposez d’autorisations. Dans ce mode, l’étendue des requêtes englobe toutes les données de toutes les tables dans l’espace de travail. C’est ce mode d’accès qui est utilisé quand vous accédez aux journaux alors que l’étendue définie est l’espace de travail. C’est par exemple le cas quand vous sélectionnez **Journaux** à partir du menu **Azure Monitor** dans le portail Azure.

    ![Contexte Log Analytics de l’espace de travail](./media/design-logs-deployment/query-from-workspace.png)

* **Contexte de ressource** : lorsque vous accédez à l’espace de travail pour une ressource, un groupe de ressources ou un abonnement spécifique, par exemple, lorsque vous sélectionnez **Journaux** à partir d’un menu de ressources dans le portail Azure, vous pouvez voir les journaux pour les ressources dans toutes les tables auxquelles vous avez accès uniquement. Dans ce mode, l’étendue des requêtes englobe uniquement les données associées à cette ressource. Ce mode autorise également le contrôle d’accès en fonction du rôle Azure (Azure RBAC) granulaire.

    ![Contexte Log Analytics de la ressource](./media/design-logs-deployment/query-from-resource.png)

    > [!NOTE]
    > Les journaux sont disponibles pour les requêtes dans le contexte de la ressource uniquement s’ils ont été correctement associés à la ressource appropriée. Les ressources suivantes ont des limitations :
    > - Ordinateurs situés en dehors d’Azure
    > - Service Fabric
    > - Application Insights
    >
    > Vous pouvez tester si les journaux sont correctement associés à leurs ressources en exécutant une requête et en inspectant les enregistrements qui vous intéressent. Si l’ID de ressource correct se trouve dans la propriété [_ResourceId](./log-standard-columns.md#_resourceid), les données sont disponibles pour les requêtes centrées sur la ressource.

Azure Monitor détermine automatiquement le mode approprié en fonction du contexte à partir duquel vous effectuez la recherche dans les journaux. L’étendue est toujours affichée en haut à gauche de Log Analytics.

### <a name="comparing-access-modes"></a>Comparaison des modes d’accès

Le tableau suivant récapitule les modes d’accès :

| Problème | Contexte d’espace de travail | Contexte d’espace de ressource |
|:---|:---|:---|
| À qui chaque modèle s’adresse-t-il ? | Administration centrale. Les administrateurs qui ont besoin de configurer une collecte de données et les utilisateurs qui ont besoin d’accéder à un large éventail de ressources. Également nécessaire pour les utilisateurs qui doivent accéder aux journaux des ressources situées en dehors d’Azure. | Équipes d’application. Administrateurs de ressources Azure en cours de supervision. |
| De quoi un utilisateur a-t-il besoin pour voir les journaux ? | Autorisations sur l’espace de travail. Consultez **Autorisations d’espace de travail** dans [Gérer l’accès à l’aide d’autorisations au niveau de l’espace de travail](manage-access.md#manage-access-using-workspace-permissions). | Accès en lecture à la ressource. Consultez **Autorisations de ressources** dans [Gérer l’accès à l’aide d’autorisations Azure](manage-access.md#manage-access-using-azure-permissions). Les autorisations peuvent être héritées (par exemple du groupe de ressources les contenant) ou directement attribuées à la ressource. L’autorisation sur les journaux pour la ressource est automatiquement attribuée. |
| Quelle est l’étendue des autorisations ? | Espace de travail. Les utilisateurs ayant accès à l’espace de travail peuvent interroger tous les journaux dans cet espace de travail à partir des tables sur lesquelles ils ont des autorisations. Consultez [Contrôle d’accès aux tables](manage-access.md#table-level-azure-rbac). | Ressource Azure. L’utilisateur peut interroger les journaux à propos de ressources, groupes de ressources ou abonnements auxquels il a accès à partir de n’importe quel espace de travail, mais il ne peut pas interroger les journaux associés à d’autres ressources. |
| Comment l’utilisateur peut-il accéder aux journaux ? | <ul><li>Démarrez **Journaux** dans le menu **Azure Monitor**.</li></ul> <ul><li>Démarrez **Journaux** à partir des **Espaces de travail Log Analytics**.</li></ul> <ul><li>À partir des [Workbooks](../visualizations.md#workbooks) Azure Monitor.</li></ul> | <ul><li>Démarrez **Journaux** dans le menu associé à la ressource Azure.</li></ul> <ul><li>Démarrez **Journaux** dans le menu **Azure Monitor**.</li></ul> <ul><li>Démarrez **Journaux** à partir des **Espaces de travail Log Analytics**.</li></ul> <ul><li>À partir des [Workbooks](../visualizations.md#workbooks) Azure Monitor.</li></ul> |

## <a name="access-control-mode"></a>Mode de contrôle d’accès

Le *mode de contrôle d’accès* est un paramètre sur chaque espace de travail qui définit comment les autorisations sont déterminées pour l’espace de travail.

* **Exiger des autorisations d’espace de travail** : Ce mode de contrôle n’autorise pas un contrôle Azure RBAC granulaire. Pour qu’un utilisateur puisse accéder à l’espace de travail, il doit avoir des autorisations sur l’espace de travail ou sur des tables spécifiques.

    Si un utilisateur accède à l’espace de travail dans le contexte de l’espace de travail, il a accès à toutes les données de toutes les tables auxquelles il est autorisé à accéder. Si un utilisateur accède à l’espace de travail dans le contexte de la ressource, il a accès uniquement aux données de cette ressource dans les tables auxquelles il est autorisé à accéder.

    Il s’agit du paramétrage par défaut pour tous les espaces de travail créés avant mars 2019.

* **Utiliser les autorisations de ressource ou d’espace de travail** : Ce mode de contrôle autorise un contrôle Azure RBAC granulaire. Les utilisateurs peuvent être autorisés à accéder uniquement aux données associées aux ressources qu’ils peuvent afficher en attribuant l’autorisation `read` Azure. 

    Quand un utilisateur accède à l’espace de travail dans le contexte de l’espace de travail, les autorisations d’espace de travail s’appliquent. Quand un utilisateur accède à l’espace de travail dans le contexte de la ressource, seules les autorisations de ressource sont vérifiées, les autorisations d’espace de travail étant ignorées. Activez Azure RBAC pour un utilisateur en le supprimant des autorisations d’espace de travail et en permettant à ses autorisations de ressource d’être reconnues.

    Il s’agit du paramétrage par défaut pour tous les espaces de travail créés après mars 2019.

    > [!NOTE]
    > Si un utilisateur dispose uniquement des autorisations de ressource sur l’espace de travail, il peut uniquement accéder à l’espace de travail à l’aide du contexte de ressource, en supposant que le mode d’accès à l’espace de travail est défini sur **Utiliser les autorisations de ressource ou d’espace de travail**.

Pour savoir comment modifier le mode de contrôle d’accès dans le portail, avec PowerShell ou à l’aide d’un modèle Resource Manager, consultez [Configurer le mode de contrôle d’accès](manage-access.md#configure-access-control-mode).

## <a name="scale-and-ingestion-volume-rate-limit"></a>Échelle et débit maximal du volume d’ingestion

Azure Monitor est un service de données à grande échelle servant des milliers de clients envoyant des pétaoctets de données chaque mois à un rythme croissant. Les espaces de travail ne sont pas limités dans leur espace de stockage, et peuvent croître jusqu’à plusieurs pétaoctets de données. Il n’est pas nécessaire de fractionner les espaces de travail en raison d’une mise à l’échelle.

Pour protéger et isoler les clients Azure Monitor et leur infrastructure principale, il existe un débit maximal d’ingestion par défaut conçu pour protéger contre les situations de pics et de saturations. Le débit maximal par défaut, d’environ **6 Go/minute**, est conçu pour permettre une ingestion normale. Pour plus d’informations sur la mesure du volume maximal d’ingestion, consultez [Limites du service Azure Monitor](../service-limits.md#data-ingestion-volume-rate).

Les clients qui ingèrent moins de 4 To/jour ne respectent généralement pas ces limites. Les clients qui ingèrent des volumes plus élevés ou qui sont confrontés à des pics dans le cadre de leurs opérations normales doivent envisager de passer à des [clusters dédiés](../log-query/logs-dedicated-clusters.md) où le débit maximal d’ingestion peut être augmenté.

Lorsque le débit maximal d’ingestion est activé ou atteint 80 % du seuil, un événement est ajouté à la table *Opération* dans votre espace de travail. Il est recommandé de surveiller le débit et de créer une alerte. Pour plus d’informations, consultez [Débit d’ingestion de données](../service-limits.md#data-ingestion-volume-rate).


## <a name="recommendations"></a>Recommandations

![Exemple de conception dans le contexte de la ressource](./media/design-logs-deployment/workspace-design-resource-context-01.png)

Ce scénario couvre une seule conception d’espace de travail dans votre abonnement aux services informatiques, qui n’est pas limitée par la souveraineté des données ni la conformité réglementaire, ou qui doit être mappée aux régions dans lesquelles vos ressources sont déployées. Cela permet aux équipes de sécurité et d’administration informatique de votre organisation de tirer parti de l’intégration améliorée avec la gestion des accès Azure et un contrôle d’accès plus sécurisé.

L’ensemble des ressources, solutions de surveillance et insights tels que Application Insights et Azure Monitor pour machines virtuelles, qui prennent en charge l’infrastructure et les applications gérées par les différentes équipes, sont configurées pour transférer leurs données de journal collectées à l’espace de travail partagé centralisé du service informatique. Les utilisateurs de chaque équipe sont autorisés à accéder aux journaux des ressources auxquelles ils ont accès.

Une fois que vous avez déployé l’architecture de votre espace de travail, vous devez appliquer cette stratégie sur les ressources Azure avec [Azure Policy](../../governance/policy/overview.md). Cela fournit un moyen de définir des stratégies et de garantir la conformité avec vos ressources Azure afin d’envoyer tous les journaux de ressources à un espace de travail particulier. Par exemple, avec des machines virtuelles Azure ou des groupes de machines virtuelles identiques, vous pouvez utiliser des stratégies existantes qui évaluent la conformité de l’espace de travail et rendent compte des résultats, ou personnaliser des stratégies pour apporter des corrections en cas de non-conformité.  

## <a name="workspace-consolidation-migration-strategy"></a>Stratégie de migration pour la consolidation de l’espace de travail

Nous recommandons aux clients qui ont déjà déployé plusieurs espaces de travail et qui sont intéressés par la consolidation du modèle d’accès dans le contexte de la ressource d’adopter une approche incrémentielle pour migrer vers le modèle d’accès recommandé, sans essayer d’y parvenir rapidement ou agressivement. Suivre une approche progressive de la planification, de la migration, de la validation et de la mise hors service à la suite d’une timeline raisonnable permet d’éviter tout incident non planifié ou tout impact inattendu sur vos opérations cloud. Si vous ne disposez pas d’une stratégie de rétention des données pour des raisons professionnelles ou de conformité, vous devez évaluer la durée appropriée pour conserver les données dans l’espace de travail à partir duquel vous effectuez la migration pendant le processus. Lorsque vous reconfigurez des ressources pour qu’elles rendent compte à l’espace de travail partagé, vous pouvez toujours analyser les données dans l’espace de travail d’origine, si nécessaire. Une fois la migration terminée, si vous êtes assujetti à la conservation des données dans l’espace de travail d’origine avant la fin de la période de rétention, ne supprimez pas ces données.

Lors de la planification de votre migration vers ce modèle, prenez en compte les éléments suivants :

* Comprenez les réglementations sectorielles et les stratégies internes relatives à la conservation des données que vous devez respecter.
* Assurez-vous que vos équipes d’applications peuvent travailler dans la fonctionnalité de contexte de ressource existante.
* Identifiez l’accès accordé aux ressources pour vos équipes d’applications et testez-les dans un environnement de développement avant d’effectuer une implémentation en production.
* Configurez l’espace de travail pour activer l’**utilisation des autorisations de ressource ou d’espace de travail**.
* Supprimez l’autorisation des équipes d’application de lire et d’interroger l’espace de travail.
* Activez et configurez des solutions de surveillance, des Insights, tels que Azure Monitor pour conteneurs et/ou Azure Monitor pour machines virtuelles, vos comptes Automation et des solutions de gestion telles que Update Management, Démarrer/arrêter des machines virtuelles, etc., qui ont été déployées dans l’espace de travail d’origine.

## <a name="next-steps"></a>Étapes suivantes

Pour implémenter les contrôles et les autorisations de sécurité recommandés dans ce guide, consultez [Gérer l’accès aux journaux](manage-access.md).