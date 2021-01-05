---
title: FAQ Azure Security Center – Collecte de données et agents
description: Forum aux questions sur la collecte de données, les agents et les espaces de travail pour Azure Security Center, produit qui vous aide à prévenir, détecter et répondre aux menaces
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: be2ab6d5-72a8-411f-878e-98dac21bc5cb
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/15/2020
ms.author: memildin
ms.openlocfilehash: 64fa6c72e3bc37276dd108e3981bbefb5a2021a7
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96444516"
---
# <a name="faq---questions-about-data-collection-agents-and-workspaces"></a>FAQ – Question relatives à la collecte de données, aux agents et aux espaces de travail

Azure Security Center collecte des données à partir de vos machines virtuelles Azure, groupes de machines virtuelles identiques, conteneurs IaaS et ordinateurs autres qu’Azure (y compris les ordinateurs locaux) pour surveiller les menaces et vulnérabilités de sécurité. Les données sont collectées à l’aide de l’agent Log Analytics, qui lit divers journaux d’événements et configurations liées à la sécurité de la machine et copie les données dans votre espace de travail à des fins d’analyse.


## <a name="am-i-billed-for-azure-monitor-logs-on-the-workspaces-created-by-security-center"></a>Est-ce que je suis facturé pour Journaux Azure Monitor sur des espaces de travail créés par Security Center ?

Non. Les espaces de travail créés par Security Center, bien qu’ils soient configurés pour une facturation Journaux Azure Monitor par nœud, n’entraînent pas de frais de journaux Azure Monitor. La facturation Security Center est toujours basée sur la stratégie de sécurité Security Center et les solutions installées sur l’espace de travail :

- **Azure Defender désactivé** : Security Center active la solution « SecurityCenterFree » sur l’espace de travail par défaut. Vous ne serez pas facturé si Azure Defender est désactivé.

- **Azure Defender activé** : Security Center active la solution « Security » sur l’espace de travail par défaut.

Pour plus d’informations sur la tarification, consultez la page de [tarification de Security Center](https://azure.microsoft.com/pricing/details/security-center/).

> [!NOTE]
> Le niveau tarifaire Log Analytics des espaces de travail créés par Security Center n’affecte pas la facturation Security Center.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]


## <a name="what-is-the-log-analytics-agent"></a>Qu’est-ce que l’agent Log Analytics ?

Pour superviser les menaces et les failles de sécurité, Azure Security Center dépend de l’[agent Log Analytics](../azure-monitor/platform/log-analytics-agent.md) ; il s’agit du même agent que celui utilisé par le service Azure Monitor. 

Cet agent est parfois appelé Microsoft Monitoring Agent (ou « MMA »). 

L’agent collecte différents journaux des événements et détails de configuration liés à la sécurité à partir des ordinateurs connectés, puis copie les données dans votre espace de travail Log Analytics pour une analyse approfondie. Il peut s’agir des données suivantes : type et version de système d’exploitation, journaux d’activité de système d’exploitation (journaux d’événements Windows), processus en cours d’exécution, nom de machine, adresses IP et utilisateur connecté.

Vérifiez que vos machines exécutent l’un des systèmes d’exploitation pris en charge pour l’agent, comme décrit dans les pages suivantes :

* [Agent Log Analytics pour les systèmes d’exploitation pris en charge par Windows](../azure-monitor/platform/agents-overview.md#supported-operating-systems)

* [Agent Log Analytics pour les systèmes d’exploitation pris en charge par Linux](../azure-monitor/platform/agents-overview.md#supported-operating-systems)

Apprenez-en davantage sur les [données collectées par l’agent Log Analytics](security-center-enable-data-collection.md).




## <a name="what-qualifies-a-vm-for-automatic-provisioning-of-the-log-analytics-agent-installation"></a>Qu’est-ce qui rend une machine virtuelle apte pour le provisionnement automatique de l’installation de l’agent Log Analytics ?

Les machines virtuelles Windows ou Linux IaaS sont retenues dans les cas suivants :

- L’extension de l’agent Log Analytics n’est pas actuellement installée sur la machine virtuelle.
- La machine virtuelle est en cours d’exécution.
- L’[agent de machine virtuelle Azure](../virtual-machines/extensions/agent-windows.md) pour Windows ou Linux est installé.
- La machine virtuelle n’est pas utilisée comme pare-feu d’applications web ou comme pare-feu de nouvelle génération.


## <a name="where-is-the-default-log-analytics-workspace-created"></a>Où est créé l’espace de travail Log Analytics par défaut ?

La localisation de l’espace de travail par défaut dépend de votre région Azure :

- Pour les machines virtuelles aux États-Unis et au Brésil, la localisation de l’espace de travail est les États-Unis
- Pour les machines virtuelles au Canada, la localisation de l’espace de travail est le Canada
- Pour les machines virtuelles en Europe, la localisation de l’espace de travail est l’Europe
- Pour les machines virtuelles au Royaume-Uni, la localisation de l’espace de travail est le Royaume-Uni
- Pour les machines virtuelles dans les régions Asie Est et Asie Sud-Est, la localisation de l’espace de travail est l’Asie
- Pour les machines virtuelles en Corée, la localisation de l’espace de travail est la Corée
- Pour les machines virtuelles en Inde, la localisation de l’espace de travail est l’Inde
- Pour les machines virtuelles au Japon, la localisation de l’espace de travail est le Japon
- Pour les machines virtuelles en Chine, la localisation de l’espace de travail est la Chine
- Pour les machines virtuelles en Australie, la localisation de l’espace de travail est l’Australie


## <a name="what-security-events-are-collected-by-the-log-analytics-agent"></a>Quels sont les événements de sécurité collectés par l'agent Log Analytics ?

Pour obtenir la liste complète des événements de sécurité collectés par l'agent, consultez [Quels types d'événements sont stockés pour les paramètres d'événements de sécurité « Commun » et « Minimal » ?](security-center-enable-data-collection.md#what-event-types-are-stored-for-common-and-minimal).

> [!IMPORTANT]
> Notez que pour certains services, comme Pare-feu Azure, si vous avez activé la journalisation et choisi une ressource bavarde à journaliser (par exemple, en définissant le journal sur *détaillé*), vous pourriez constater des impacts significatifs sur les besoins de stockage de votre espace de travail Log Analytics. 


## <a name="can-i-delete-the-default-workspaces-created-by-security-center"></a>Puis-je supprimer les espaces de travail par défaut créés par Security Center ?

**Il n’est pas recommandé de supprimer l’espace de travail par défaut.** Security Center utilise les espaces de travail par défaut pour stocker les données de sécurité de vos machines virtuelles. Si vous supprimez un espace de travail, Security Center n’est pas en mesure pas collecter ces données, et certaines recommandations de sécurité et alertes ne sont pas disponibles.

Pour effectuer une récupération, supprimez l’agent Log Analytics sur les machines virtuelles connectées à l’espace de travail supprimé. Security Center réinstalle l’agent et crée des nouveaux espaces de travail par défaut.

## <a name="how-can-i-use-my-existing-log-analytics-workspace"></a>Comment puis-je utiliser mon espace de travail Log Analytics existant ?

Vous pouvez sélectionner un espace de travail Log Analytics existant pour stocker les données collectées par Security Center. Pour utiliser votre espace de travail Log Analytics existant :

- L’espace de travail doit être associé à votre abonnement Azure sélectionné.
- Au minimum, vous devez avoir des autorisations de lecture pour accéder à l’espace de travail.

Pour sélectionner un espace de travail Log Analytics existant :

1. Dans le menu de Security Center, sélectionnez **Tarification et paramètres**.
1. Sélectionnez l’abonnement approprié.
1. Ouvrir la page **Approvisionnement automatique**
1. Pour l’agent Log Analytics, sélectionnez **Modifier la configuration**. 

    :::image type="content" source="./media/security-center-enable-data-collection/edit-configuration-auto-deploy-agent.png" alt-text="Configuration de l’agent Log Analytics à utiliser lors de l’utilisation du déploiement automatique" lightbox="./media/security-center-enable-data-collection/edit-configuration-auto-deploy-agent.png":::

1. Sélectionnez **Connect Azure VMs to a different workspace** (Connecter des machines virtuelles Azure à un autre espace de travail) et choisissez votre espace de travail existant.

    :::image type="content" source="./media/security-center-enable-data-collection/choose-workspace.png" alt-text="Sélection d’un espace de travail autre que celui par défaut auquel votre agent Log Analytics doit se rapporter" lightbox="./media/security-center-enable-data-collection/choose-workspace.png":::

    > [!TIP]
    > La liste comprend uniquement les espaces de travail auxquels vous avez accès et ceux qui se trouvent dans votre abonnement Azure.

1. Sélectionnez **Enregistrer**. Vous serez invité à reconfigurer les machines virtuelles surveillées.

    - Sélectionnez **Non** si vous souhaitez que les nouveaux paramètres de l’espace de travail **s’appliquent uniquement aux nouvelles machines virtuelles**. Les nouveaux paramètres de l’espace de travail s’appliquent uniquement aux nouvelles installations d’agent ; machines virtuelles nouvellement détectées sur lesquelles l’agent Log Analytics n’est pas installé.
    - Sélectionnez **Oui** si vous souhaitez que les nouveaux paramètres de l’espace de travail **s’appliquent à toutes les machines virtuelles**. En outre, chaque machine virtuelle connectée à un espace de travail créé par Security Center est reconnectée au nouvel espace de travail cible.

    > [!NOTE]
    > Si vous sélectionnez **Oui**, ne supprimez pas les espaces de travail créés par Security Center tant que toutes les machines virtuelles n’ont pas été reconnectées au nouvel espace de travail cible. Cette opération échoue si un espace de travail est supprimé trop tôt.


## <a name="what-if-the-log-analytics-agent-was-already-installed-as-an-extension-on-the-vm"></a>Que se passe-t-il si l’agent Log Analytics est déjà installé en tant qu’extension sur la machine virtuelle ?<a name="mmaextensioninstalled"></a>

Lorsque l’Agent Monitoring est installé en tant qu’extension, la configuration de l’extension permet de rendre compte à un seul espace de travail. Security Center n’écrase pas les connexions existantes des espaces de travail utilisateur. Security Center stocke les données de sécurité à partir d’une machine virtuelle dans un espace de travail qui est déjà connecté, sous réserve que la solution « Security » ou « SecurityCenterFree » y soit installée. Security Center peut mettre à niveau la version de l'extension vers la dernière version lors de ce processus.

Pour plus d’informations, voir [Approvisionnement automatique en cas d’installation d’un agent préexistant](security-center-enable-data-collection.md#preexisting).



## <a name="what-if-a-log-analytics-agent-is-directly-installed-on-the-machine-but-not-as-an-extension-direct-agent"></a>Que faire si l’agent Log Analytics est installé directement sur la machine, mais pas en tant qu’extension (agent direct) ?<a name="directagentinstalled"></a>

Si l’agent Log Analytics est installé directement sur la machine virtuelle (pas en tant qu’extension Azure), Security Center installe l’extension de l’agent Log Analytics, et peut mettre à niveau l’agent Log Analytics vers la dernière version.

L’agent installé continue de rendre compte à ses espaces de travail déjà configurés et à l’espace de travail configuré dans Security Center (multihébergement pris en charge sur les machines Windows).

Si l’espace de travail configuré est un espace de travail utilisateur (pas un espace de travail par défaut de Security Center), vous devez y installer la solution « Security/SecurityCenterFree » pour que Security Center démarre le traitement des événements à partir des machines virtuelles et ordinateurs rendant compte à cet espace de travail.

Pour les machines Linux, le multihébergement d’agent n’est pas encore pris en charge, par conséquent, si une installation existante de l’agent est détectée, l’approvisionnement automatique n’a pas lieu et la configuration de la machine n’est pas modifiée.

Pour les machines existantes sur les abonnements intégrés à Security Center avant le 17 mars 2019, lorsqu’un agent existant est détecté, l’agent Log Analytics n’est pas installé et la machine n’est pas affectée. Pour ces machines, consultez la recommandation « Résoudre les problèmes d’intégrité de l’agent d’analyse sur vos machines » pour résoudre les problèmes d’installation de l’agent sur ces machines.

Pour plus d’informations, consultez la prochaine section, [Que se passe-t-il si l’agent direct System Center Operations Manager ou OMS est déjà installé sur ma machine virtuelle ?](#scomomsinstalled)

## <a name="what-if-a-system-center-operations-manager-agent-is-already-installed-on-my-vm"></a>Que se passe-t-il si un agent Operations Manager est déjà installé sur ma machine virtuelle ?<a name="scomomsinstalled"></a>

Security Center installe l’extension de l’agent Log Analytics parallèlement à l’agent System Center Operations Manager existant. L’agent existant continue de rendre compte normalement au serveur System Center Operations Manager. Notez que l’agent Operations Manager et l’agent Log Analytics partagent des bibliothèques runtime communes, qui sont mises à jour vers la dernière version lors de ce processus. Remarque : si la version 2012 de l’agent Operations Manager est installée, n’activez pas l’approvisionnement automatique (les fonctionnalités de facilité de gestion peuvent être perdues si la version du serveur Operations Manager date également de 2012).


## <a name="what-is-the-impact-of-removing-these-extensions"></a>Quel est l’impact de la suppression de ces extensions ?

Si vous supprimez l’extension Microsoft Monitoring, Security Center n’est pas en mesure de collecter des données de sécurité sur la machine virtuelle et certaines recommandations de sécurité et les alertes ne sont pas disponibles. Sous 24 heures, Security Center détermine que l’extension est absente de la machine virtuelle et la réinstalle.


## <a name="how-do-i-stop-the-automatic-agent-installation-and-workspace-creation"></a>Comment empêcher l’installation automatique de l’agent et la création de l’espace de travail ?

Vous pouvez désactiver l’approvisionnement automatique pour vos abonnements dans la stratégie de sécurité, mais ce n’est pas recommandé. La désactivation de l’approvisionnement automatique a pour effet de limiter les alertes et recommandations de Security Center. Pour désactiver l’approvisionnement automatique :

1. Dans le menu de Security Center, sélectionnez **Tarification et paramètres**.
1. Sélectionnez l’abonnement approprié.
1. Si votre abonnement a activé Azure Defender, ouvrez **Plans Azure Defender** et sélectionnez **Azure Defender désactivé**.

    :::image type="content" source="./media/security-center-platform-migration-faq/pricing-tier.png" alt-text="Activer ou désactiver Azure Defender":::

1. Dans la page **Approvisionnement automatique**, sélectionnez Pen et désactivez l’approvisionnement automatique dans la page **Stratégie de sécurité – Collecte de données**.

    :::image type="content" source="./media/security-center-enable-data-collection/agent-toggles.png" alt-text="Activer le déploiement automatique pour l’agent Log Analytics":::

1. Sélectionnez **Enregistrer**.


## <a name="should-i-opt-out-of-the-automatic-agent-installation-and-workspace-creation"></a>Dois-je refuser l’installation automatique de l’agent et la création de l’espace de travail ?

> [!NOTE]
> Veillez à consulter les sections [Quelles sont les implications d’un refus ?](#what-are-the-implications-of-opting-out-of-automatic-provisioning) et [Étapes recommandées en cas de refus](#what-are-the-recommended-steps-when-opting-out-of-automatic-provisioning) si vous choisissez de refuser le provisionnement automatique.

Vous pouvez souhaiter refuser le provisionnement automatique si les points suivants s’appliquent à vous :

- L’installation automatique de l’agent par Security Center s’applique à l’ensemble de l’abonnement. Vous ne pouvez pas appliquer l’installation automatique à un sous-ensemble de machines virtuelles. Si des machines virtuelles critiques ne peuvent pas être installées avec l’agent Log Analytics, vous devez refuser le provisionnement automatique.
- L’installation de l’extension de l’agent Log Analytics met à jour la version de l’agent. Cela s’applique à un agent direct et un agent System Center Operations Manager (dans le dernier cas, les bibliothèques Operations Manager et l’agent Log Analytics partagent des bibliothèques runtime communes, qui sont mises à jour dans le processus). Si l’agent Operations Manager installé est en version 2012 et qu’il est mis à niveau, les fonctionnalités de facilité de gestion peuvent être perdues quand le serveur Operations Manager est également en version 2012. Vous pouvez refuser le provisionnement automatique si l’agent Operations Manager installé est en version 2012.
- Si vous disposez d’un espace de travail personnalisé externe à l’abonnement (un espace de travail centralisé), vous devez refuser l’approvisionnement automatique. Vous pouvez installer manuellement l’extension de l’agent Log Analytics et la connecter à votre espace de travail sans que Security Center ne remplace la connexion.
- Si vous souhaitez éviter de créer plusieurs espaces de travail par abonnement et que vous disposez de votre propre espace de travail personnalisé dans l’abonnement, deux options s’offrent à vous :

   1. Vous pouvez refuser le provisionnement automatique. Après la migration, définissez les paramètres d’espace de travail par défaut comme décrit dans [Comment puis-je utiliser mon espace de travail Log Analytics existant ?](#how-can-i-use-my-existing-log-analytics-workspace)

   1. Vous pouvez aussi autoriser l’exécution de la migration, l’installation de l’agent Log Analytics sur les machines virtuelles et la connexion des machines virtuelles à l’espace de travail créé. Sélectionnez ensuite votre propre espace de travail personnalisé en définissant le paramètre d’espace de travail par défaut avec l’activation de la reconfiguration des agents déjà installés. Pour plus d’informations, consultez [Comment puis-je utiliser mon espace de travail Log Analytics existant ?](#how-can-i-use-my-existing-log-analytics-workspace)


## <a name="what-are-the-implications-of-opting-out-of-automatic-provisioning"></a>Quelles sont les implications d’un refus du provisionnement automatique ?

Une fois la migration terminée, Security Center n’est pas en mesure de collecter des données de sécurité sur la machine virtuelle, et certaines recommandations de sécurité et alertes ne sont pas disponibles. En cas de refus, installez manuellement l’agent Log Analytics. Consultez les [étapes recommandées en cas de refus](#what-are-the-recommended-steps-when-opting-out-of-automatic-provisioning).


## <a name="what-are-the-recommended-steps-when-opting-out-of-automatic-provisioning"></a>Quelles sont les étapes recommandées en cas de refus du provisionnement automatique ?

Installez manuellement l’extension de l’agent Log Analytics pour que Security Center puisse collecter des données de sécurité sur vos machines virtuelles et fournir des suggestions et des alertes. Consultez [Installation de l’agent pour les machines virtuelles Windows](../virtual-machines/extensions/oms-windows.md) ou [Installation de l’agent pour les machines virtuelles Linux](../virtual-machines/extensions/oms-linux.md) pour obtenir des instructions sur l’installation.

Vous pouvez connecter l’agent à n’importe quel espace de travail personnalisé existant ou à l’espace de travail créé par Security Center. Si les solutions « Security » ou « SecurityCenterFree » ne sont pas activées pour un espace de travail personnalisé, vous devez appliquer une solution. Pour ce faire, sélectionnez l’abonnement ou l’espace de travail personnalisé, puis appliquez un niveau tarifaire via la page **Stratégie de sécurité – Niveau tarifaire**.

:::image type="content" source="./media/security-center-platform-migration-faq/pricing-tier.png" alt-text="Activer ou désactiver Azure Defender":::

Security Center active la solution appropriée sur l’espace de travail en fonction du niveau tarifaire sélectionné.


## <a name="how-do-i-remove-oms-extensions-installed-by-security-center"></a>Comment supprimer des extensions OMS installées par Security Center ?<a name="remove-oms"></a>

Vous pouvez manuellement supprimer l’agent Log Analytics. Ce n’est pas recommandé car cela limite les recommandations de Security Center et les alertes.

> [!NOTE]
> Si la collecte de données est activée, Security Center réinstalle l’agent après l’avoir supprimé.  Vous devez désactiver la collecte de données avant de supprimer manuellement l’agent. Consultez Comment empêcher l’installation automatique de l’agent et la création de l’espace de travail ? pour obtenir des instructions sur la désactivation de la collecte de données.

Pour supprimer manuellement l’agent :

1.    Dans le portail, ouvrez **Log Analytics**.

1.    Sur la page Log Analytics, sélectionnez un espace de travail :

1.    Sélectionnez les machines virtuelles que vous ne souhaitez pas surveiller, puis choisissez **Déconnecter**.

   ![Supprimer l’agent][3]

> [!NOTE]
> Si une machine virtuelle Linux possède déjà un agent OMS qui n’est pas un agent d’extension, la suppression de l’extension a pour effet de supprimer également l’agent. Vous devrez alors le réinstaller.


## <a name="how-do-i-disable-data-collection"></a>Comment désactiver la collecte des données ?

L’approvisionnement automatique est fortement recommandé si vous souhaitez obtenir des alertes de sécurité et des recommandations sur les mises à jour système, les vulnérabilités du système d’exploitation et la protection du point de terminaison. Par défaut, le provisionnement automatique désactivé.

Si vous l’avez activé, mais que vous souhaitez maintenant le désactiver :

1. Dans le [Portail Azure](https://portal.azure.com), ouvrez **Centre de sécurité** et sélectionnez **Tarification et paramètres**.

1. Sélectionnez l’abonnement pour lequel vous souhaitez désactiver le provisionnement automatique.

1. Sous **Approvisionnement automatique**, désactivez la bascule de l’agent d’analytique des journaux d'activité.


## <a name="how-do-i-enable-data-collection"></a>Comment activer la collecte des données ?

Vous pouvez activer la collecte des données pour votre abonnement Azure dans la stratégie de sécurité. Pour activer la collecte de données. [Connectez-vous au portail Azure](https://portal.azure.com), sélectionnez **Parcourir**, **Centre de sécurité**, puis **Stratégie de sécurité**. Sélectionnez l’abonnement pour lequel vous souhaitez activer l’approvisionnement automatique. Lorsque vous sélectionnez un abonnement, **Stratégie de sécurité - Collecte de données** s’ouvre. Sous **Auto provisioning** (Approvisionnement automatique), sélectionnez **On** (Activé).


## <a name="what-happens-when-data-collection-is-enabled"></a>Que se passe-t-il quand la collecte des données est activée ?

Lorsque l’approvisionnement automatique est activé, Security Center approvisionne l’agent Log Analytics sur toutes les machines virtuelles Azure prises en charge et toutes celles nouvellement créées. L’approvisionnement automatique est recommandé. Toutefois, l’installation manuelle de l’agent est également disponible. [Découvrez comment installer l’extension de l’agent Log Analytics](../azure-monitor/learn/quick-collect-azurevm.md#enable-the-log-analytics-vm-extension). 

L’agent active l’événement de création de processus 4688 et le champ *CommandLine* à l’intérieur de l’événement 4688. Les nouveaux processus créés sur la machine virtuelle sont enregistrés par le journal des événements et analysés par les services de détection Security Center. Pour plus d’informations sur les détails enregistrés pour chaque nouveau processus, voir [Description des champs 4688](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4688#fields). L’agent collecte également les événements 4688 créés sur la machine virtuelle et les stocke dans la recherche.

L’agent permet également d’activer la collecte de données pour les [Contrôles d’application adaptatifs](security-center-adaptive-application.md), Security Center configure une stratégie AppLocker locale en mode Audit pour autoriser toutes les applications. Cette stratégie amène AppLocker à générer des événements qui sont ensuite recueillis et exploités par Security Center. Il est important de noter que cette stratégie ne sera configurée sur aucun ordinateur sur lequel une stratégie AppLocker est déjà configurée. 

Lorsque Security Center détecte une activité suspecte sur la machine virtuelle, le client est averti par e-mail si [des informations de contact de sécurité](security-center-provide-security-contact-details.md) ont été fournies. Une alerte est également visible dans le tableau de bord d’alertes de sécurité de Security Center.


## <a name="will-security-center-work-using-an-oms-gateway"></a>Security Center fonctionne-t-il avec une passerelle OMS ?

Oui. Azure Security Center utilise Azure Monitor pour collecter des données à partir de machines virtuelles et de serveurs Azure, à l’aide de l’agent Log Analytics.
Pour collecter les données, chaque machine virtuelle et chaque serveur doivent se connecter à Internet à l’aide du protocole HTTPS. Il peut s’agir d’une connexion directe, d’une connexion obtenue par un proxy ou bien établie via la [passerelle OMS](../azure-monitor/platform/gateway.md).


## <a name="does-the-monitoring-agent-impact-the-performance-of-my-servers"></a>Est-ce que l’agent d’analyse a un impact sur les performances de mes serveurs ?

L’agent utilise une quantité minime de ressources système et n’a donc qu’un faible impact sur les performances. Pour en savoir plus sur l’impact sur les performances, l’agent et l’extension, consultez le [Guide de planification et de fonctionnement](security-center-planning-and-operations-guide.md#data-collection-and-storage).




<!--Image references-->
[2]: ./media/security-center-platform-migration-faq/data-collection.png
[3]: ./media/security-center-platform-migration-faq/remove-the-agent.png
[4]: ./media/security-center-platform-migration-faq/use-another-workspace.png