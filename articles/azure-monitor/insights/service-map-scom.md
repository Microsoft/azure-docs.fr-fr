---
title: Intégrer un mappage Azure Monitor pour machines virtuelles avec Operations Manager | Microsoft Docs
description: Azure Monitor pour machines virtuelles découvre automatiquement les composants d’application sur les systèmes Windows et Linux, et cartographie la communication entre les différents services. Cet article décrit l’utilisation de la fonctionnalité Map pour créer automatiquement des diagrammes d’application distribuée dans Operations Manager.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/12/2019
ms.openlocfilehash: 489b4bfee1eaa2c381b16e88dbcc0bcb7fd8555f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91441504"
---
# <a name="integrate-system-center-operations-manager-with-azure-monitor-for-vms-map-feature"></a>Intégrer System Center Operations Manager avec la fonctionnalité de mappage Azure Monitor pour machines virtuelles

Dans Azure Monitor pour machines virtuelles, vous pouvez voir les composants d’application qui ont été détectés sur les machines virtuelles Windows et Linux exécutées dans Azure ou votre environnement. Avec cette intégration entre la fonctionnalité Map et System Center Operations Manager, vous pouvez créer automatiquement des diagrammes d’application distribuée dans Operations Manager basés sur des cartes de dépendance dynamique dans Azure Monitor pour machines virtuelles. Cet article explique comment configurer votre groupe d’administration System Center Operations Manager pour prendre en charge cette fonctionnalité.

>[!NOTE]
>Si vous avez déjà déployé Service Map, vous pouvez visualiser vos cartes dans Azure Monitor pour machines virtuelles, qui inclut des fonctionnalités supplémentaires pour la supervision de l’intégrité et des performances des machines virtuelles. La fonctionnalité Map d’Azure Monitor pour machines virtuelles est destinée à remplacer la solution autonome Service Map. Pour en savoir plus, consultez [Présentation d’Azure Monitor pour machines virtuelles](vminsights-overview.md).

## <a name="prerequisites"></a>Prérequis

* Un groupe d’administration System Center Operations Manager (2012 R2 ou version ultérieure).
* Un espace de travail Log Analytics configuré pour prendre en charge Azure Monitor pour machines virtuelles.
* Une ou plusieurs machines virtuelles Windows et Linux, ou des ordinateurs physiques supervisés par Operations Manager qui envoient des données à votre espace de travail Log Analytics. Les serveurs Linux qui envoient des rapports à un groupe d’administration Operations Manager doivent être configurés pour se connecter directement à Azure Monitor. Pour plus d’informations, consultez la vue d’ensemble dans [Collecter des données de journal avec l’agent Log Analytics](../platform/log-analytics-agent.md).
* Un principal de service disposant d’un accès à l’abonnement Azure associé à l’espace de travail Log Analytics. Pour plus d’informations, consultez l’article [Créer un principal du service](#create-a-service-principal).

## <a name="install-the-service-map-management-pack"></a>Installation du pack d’administration de Service Map

Vous activez l’intégration entre Operations Manager et la fonctionnalité Map en important le groupement de packs d’administration Microsoft.SystemCenter.ServiceMap (Microsoft.SystemCenter.ServiceMap.mpb). Vous pouvez télécharger l’offre groupée de packs d’administration à partir du [Centre de téléchargement Microsoft](https://www.microsoft.com/download/details.aspx?id=55763). Le groupe contient les packs d’administration suivants :

* Microsoft Service Map - Vues de l’application
* Microsoft System Center Service Map - Interne
* Microsoft System Center Service Map - Valeurs de remplacement
* Microsoft System Center Service Map

## <a name="configure-integration"></a>Configurer l’intégration

Après avoir installé le pack d’administration Service Map, un nouveau nœud, **Service Map**, s’affiche sous **Operations Management Suite** dans le volet **Administration** de votre console Opérateur d’Operations Manager.

>[!NOTE]
>[Operations Management Suite était une collection de services](../terminology.md#april-2018---retirement-of-operations-management-suite-brand) incluant Log Analytics, qui fait désormais partie intégrante d’[Azure Monitor](../overview.md).

Pour configurer l’intégration de la fonctionnalité MAP d’Azure Monitor pour machines virtuelles, procédez comme suit :

1. Cliquez sur **Ajouter un espace de travail** dans le volet de **vue d’ensemble de Service Map** pour ouvrir l’Assistant Configuration.  

    ![Vue d’ensemble de Service Map](media/service-map-scom/scom-configuration.png)

2. Dans la fenêtre **Configuration des connexions**, entrez l’ID ou le nom du client, l’ID d’application (aussi appelé nom d’utilisateur ou ID client) et le mot de passe du principal de service, puis cliquez sur **Suivant**. Pour plus d’informations, consultez l’article Créer un principal du service.

    ![Fenêtre Configuration des connexions](media/service-map-scom/scom-config-spn.png)

3. Dans la fenêtre **Sélection d’un abonnement**, sélectionnez l’abonnement Azure, le groupe de ressources Azure (celui qui contient l’espace de travail Log Analytics) et l’espace de travail Log Analytics, puis cliquez sur **Suivant**.

    ![Espace de travail de configuration d’Operations Manager](media/service-map-scom/scom-config-workspace.png)

4. Dans la fenêtre **Sélection du groupe d’ordinateurs**, choisissez quels groupes d’ordinateurs Service Map vous voulez synchroniser à Operations Manager. Cliquez sur **Ajouter/supprimer des groupes d’ordinateurs**, choisissez des groupes dans la liste des **Groupes d’ordinateurs disponibles**, puis cliquez sur **Ajouter**.  Lorsque vous avez fini de sélectionner des groupes, cliquez sur **Ok** pour terminer.

    ![Groupes d’ordinateurs de configuration Operations Manager](media/service-map-scom/scom-config-machine-groups.png)

5. Dans la fenêtre **Sélection du serveur**, vous configurez le groupe de serveurs Service Map avec les serveurs que vous voulez synchroniser entre Operations Manager et la fonctionnalité Map. Cliquez sur **Ajouter/Supprimer des serveurs**.

    Pour que l’intégration crée un diagramme d’application distribuée pour un serveur, le serveur doit être :

   * Supervisé par Operations Manager
   * Configuré pour envoyer des rapports à l’espace de travail Log Analytics configuré avec Azure Monitor pour machines virtuelles
   * Répertorié dans le groupe de serveurs Service Map

     ![Groupe de configuration d’Operations Manager](media/service-map-scom/scom-config-group.png)

6. Facultatif : Sélectionnez le pool de ressources de l’ensemble des serveurs d’administration pour communiquer avec Log Analytics, puis cliquez sur **Ajouter un espace de travail**.

    ![Capture de l'écran Pool de serveurs dans Ajouter un espace de travail Microsoft Operations Management Suite, avec l'option Pool de ressources de tous les serveurs de gestion sélectionnée.](media/service-map-scom/scom-config-pool.png)

    La configuration et l’inscription de l’espace de travail Log Analytics peuvent prendre quelques minutes. Une fois configuré, Operations Manager lance la première synchronisation de la fonctionnalité Map.

    ![Capture de l'écran Achèvement dans Ajouter un espace de travail Microsoft Operations Management Suite, avec confirmation que l'espace de travail a été ajouté.](media/service-map-scom/scom-config-success.png)

## <a name="monitor-integration"></a>Superviser l’intégration

Une fois l’espace de travail Log Analytics connecté, un nouveau dossier, Service Map, s’affiche dans le panneau **Supervision** de la console Opérateur d’Operations Manager.

![Panneau Analyse d’Operations Manager](media/service-map-scom/scom-monitoring.png)

Le dossier Service Map possède quatre nœuds :

* **Alertes actives** : répertorie toutes les alertes concernant la communication entre Operations Manager et Azure Monitor.  

  >[!NOTE]
  >Ces alertes ne sont pas des alertes Log Analytics synchronisées avec Operations Manager. Elles sont générées dans le groupe d’administration en fonction des flux de travail définis dans le pack d’administration Service Map.

* **Serveurs** : répertorie les serveurs supervisés configurés pour la synchronisation à partir de la fonctionnalité Map d’Azure Monitor pour machines virtuelles.

    ![Panneau Analyse des serveurs d’Operations Manager](media/service-map-scom/scom-monitoring-servers.png)

* **Vues de dépendance de groupe d’ordinateurs** : répertorie tous les groupes d’ordinateurs synchronisés à partir de la fonctionnalité Map. Vous pouvez cliquer sur n’importe quel groupe pour afficher son diagramme d’application distribuée.

    ![Capture d'écran de Service Map présentant un diagramme avec des images pour chaque groupe de machines et des lignes indiquant les dépendances entre eux.](media/service-map-scom/scom-group-dad.png)

* **Vues de dépendance du serveur** : répertorie tous les serveurs synchronisés à partir de la fonctionnalité Map. Vous pouvez cliquer sur n’importe quel serveur pour afficher son diagramme d’application distribuée.

    ![Capture d'écran de Service Map présentant un diagramme avec des images pour chaque serveur et des lignes indiquant les dépendances entre eux.](media/service-map-scom/scom-dad.png)

## <a name="edit-or-delete-the-workspace"></a>Modification ou suppression de l’espace de travail

Vous pouvez modifier ou supprimer l’espace de travail configuré via le volet de **vue d’ensemble de Service Map** (volet **Administration** > **Operations Management Suite** > **Service Map**).

> [!NOTE]
> [Operations Management Suite était une collection de services](../terminology.md#april-2018---retirement-of-operations-management-suite-brand) incluant Log Analytics, qui fait désormais partie intégrante [d’Azure Monitor](../overview.md).

Vous ne pouvez configurer qu’un seul espace de travail Log Analytics dans la version actuelle.

![Volet de modification de l’espace de travail d’Operations Manager](media/service-map-scom/scom-edit-workspace.png)

## <a name="configure-rules-and-overrides"></a>Configuration des règles et des valeurs de remplacement

Une règle, *Microsoft. SystemCenter. ServiceMapImport. Rule*, extrait périodiquement des informations de la fonctionnalité Map d’Azure Monitor pour machines virtuelles. Pour modifier l’intervalle de synchronisation, vous pouvez remplacer la règle et modifier la valeur du paramètre **IntervalMinutes**.

![Fenêtre des propriétés des valeurs de remplacement Operations Manager](media/service-map-scom/scom-overrides.png)

* **Enabled** : activer ou désactiver les mises à jour automatiques.
* **IntervalMinutes** : spécifie le délai entre les mises à jour. L’intervalle par défaut est égal à une heure. Si vous voulez synchroniser les cartes plus régulièrement, vous pouvez modifier la valeur.
* **TimeoutSeconds** : spécifie le délai avant l’expiration de la demande.
* **TimeWindowMinutes** : spécifie la fenêtre de temps pour l’interrogation des données. La valeur par défaut est de 60 minutes, ce qui correspond à l’intervalle maximal autorisé.

## <a name="known-issues-and-limitations"></a>Problèmes connus et limitations

La conception actuelle présente les problèmes et les limitations suivants :

* Vous ne pouvez vous connecter qu’à un seul espace de travail Log Analytics.
* Bien que vous puissiez ajouter des serveurs au Groupe de serveurs Service Map manuellement via le volet **Création**, les mappages de ces serveurs sont synchronisés immédiatement. Ils sont synchronisés à partir de la fonctionnalité Map d’Azure Monitor pour machines virtuelles lors du cycle de synchronisation suivant.
* Si vous apportez des modifications aux diagrammes d’application distribuée créés par le pack d’administration, ces modifications seront probablement remplacées lors de la prochaine synchronisation avec Azure Monitor pour machines virtuelles.

## <a name="create-a-service-principal"></a>Créer un principal du service

Pour obtenir une documentation Azure officielle sur la création d’un principal de service, consultez :

* [Création d’un principal de service à l’aide de PowerShell](../../active-directory/develop/howto-authenticate-service-principal-powershell.md)
* [Création d’un principal de service à l’aide d’Azure CLI](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)
* [Création d’un principal de service à l’aide du portail Azure](../../active-directory/develop/howto-create-service-principal-portal.md)

### <a name="suggestions"></a>Suggestions

Avez-vous des commentaires à nous adresser concernant l’intégration avec la fonctionnalité Map d’Azure Monitor pour machines virtuelles fonctionnalité Map ou cette documentation ? Consultez notre [page Voix utilisateur](https://feedback.azure.com/forums/267889-log-analytics/category/184492-service-map) qui vous permet de suggérer des fonctionnalités ou de voter pour les suggestions en cours.

