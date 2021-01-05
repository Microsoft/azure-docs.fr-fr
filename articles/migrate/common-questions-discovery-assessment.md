---
title: Questions sur la découverte, l’évaluation et l’analyse des dépendances dans Azure Migrate
description: Obtenez des réponses aux questions courantes sur la découverte, l’évaluation et l’analyse des dépendances dans Azure Migrate.
author: vineetvikram
ms.author: vivikram
ms.manager: abhemraj
ms.topic: conceptual
ms.date: 06/09/2020
ms.openlocfilehash: 4531d68c2fbd0698c33d70a75bb82ac9c7f52f49
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96752241"
---
# <a name="discovery-assessment-and-dependency-analysis---common-questions"></a>Découverte, évaluation et analyse des dépendances - Questions courantes

Cet article donne des réponses aux questions courantes sur la découverte, l’évaluation et l’analyse des dépendances dans Azure Migrate. Si vous avez d’autres questions, consultez les ressources suivantes :

- [Questions générales](resources-faq.md) sur Azure Migrate.
- Questions sur l’[appliance Azure Migrate](common-questions-appliance.md)
- Questions sur la [migration de serveur](common-questions-server-migration.md)
- Obtenez des réponses à vos questions sur le [forum Azure Migrate](https://aka.ms/AzureMigrateForum)


## <a name="what-geographies-are-supported-for-discovery-and-assessment-with-azure-migrate"></a>Quelles sont les zones géographiques prises en charge pour la détection et l’évaluation avec Azure Migrate ?

Passez en revue les zones géographiques prises en charge pour les clouds [publics](migrate-support-matrix.md#supported-geographies-public-cloud) et [gouvernementaux](migrate-support-matrix.md#supported-geographies-azure-government).


## <a name="how-many-vms-can-i-discover-with-an-appliance"></a>Combien de machines virtuelles peut-on détecter avec une appliance ?

Vous pouvez découvrir jusqu’à 10 000 machines virtuelles VMware, jusqu’à 5 000 machines virtuelles Hyper-V et jusqu’à 1 000 serveurs physiques avec une seule appliance. Si vous avez davantage de machines, découvrez comment [mettre à l’échelle une évaluation Hyper-V](scale-hyper-v-assessment.md), [mettre à l’échelle une évaluation VMware](scale-vmware-assessment.md) ou [mettre à l’échelle une évaluation de serveur physique](scale-physical-assessment.md).

## <a name="how-do-i-choose-the-assessment-type"></a>Comment choisir le type d’évaluation ?

- Utilisez les **évaluations des machines virtuelles Azure** quand vous voulez évaluer vos [machines virtuelles VMware](how-to-set-up-appliance-vmware.md) et [Hyper-V](how-to-set-up-appliance-hyper-v.md) ainsi que des [serveurs physiques](how-to-set-up-appliance-physical.md) en vue de leur migration vers des machines virtuelles Azure. [En savoir plus](concepts-assessment-calculation.md)

- Utilisez les évaluations **Azure VMware Solution (AVS)** quand vous voulez évaluer vos [machines virtuelles VMware](how-to-set-up-appliance-vmware.md) locales pour la migration vers [Azure VMware Solution (AVS)](../azure-vmware/introduction.md) avec ce type d’évaluation. [En savoir plus](concepts-azure-vmware-solution-assessment-calculation.md)

- Vous pouvez utiliser un groupe commun avec seulement des machines VMware pour effectuer les deux types d’évaluations. Notez que si vous effectuez des évaluations AVS dans Azure Migrate pour la première fois, il est recommandé de créer un groupe de machines VMware.
 

## <a name="why-is-performance-data-missing-for-someall-vms-in-my-assessment-report"></a>Pourquoi les données de performances sont-elles manquantes pour une partie ou la totalité des machines virtuelles dans mon rapport d’évaluation ?

Pour l’évaluation « En fonction des performances », le rapport d’évaluation signale « PercentageOfCoresUtilizedMissing » ou « PercentageOfMemoryUtilizedMissing » quand l’appliance Azure Migrate ne peut pas collecter les données de performances pour les machines virtuelles locales. Vérifiez :

- Si les machines virtuelles sont sous tension pendant que vous créez l’évaluation.
- S’il manque uniquement les compteurs de mémoire et si vous tentez d’évaluer des machines virtuelles Hyper-V, vérifiez si la mémoire dynamique est activée sur ces dernières. Il existe un problème connu qui empêche l’appliance Azure Migrate de collecter les données d’utilisation de la mémoire pour les machines virtuelles de ce type.
- Si tous les compteurs de performances sont manquants, assurez-vous que les connexions sortantes sur le port 443 (HTTPS) sont autorisées.

Remarque : Si l’un des compteurs de performances est manquant, Azure Migrate : Server Assessment se base à la place sur les cœurs et la mémoire alloués localement pour recommander une taille de machine virtuelle.

## <a name="why-is-the-confidence-rating-of-my-assessment-low"></a>Pourquoi la note de confiance de mon évaluation est faible ?

La note de confiance pour les évaluations « En fonction des performances » dépend du pourcentage de [points de données disponibles](./concepts-assessment-calculation.md#ratings) nécessaires pour calculer l’évaluation. Voici les raisons pour lesquelles une évaluation pourrait obtenir une note de confiance faible :

- Vous n’avez pas profilé votre environnement pour la durée pour laquelle vous créez l’évaluation. Par exemple, si vous créez une évaluation avec une durée des performances définie sur une semaine, vous devez attendre au moins une semaine après le démarrage de la découverte pour que tous les points de données soient recueillis. Si vous ne pouvez pas attendre pendant cette durée, définissez la durée des performances sur une période plus courte et « recalculez » l’évaluation.
 
- Server Assessment ne peut pas collecter les données de performances pour une partie ou la totalité des machines virtuelles pendant la période d’évaluation. Vérifiez que les machines virtuelles étaient sous tension pendant la durée de l’évaluation et que les connexions sortantes sur les ports 443 sont autorisées. Pour les machines virtuelles Hyper-V, si la mémoire dynamique est activée, des compteurs de mémoire font défaut, aboutissant à une note de confiance faible. « Recalculez » l’évaluation pour qu’elle reflète l’évolution récente de la note de confiance. 

- Peu de machines virtuelles ont été créées après que la découverte dans Évaluation de serveur a commencé. Par exemple, si vous créez une évaluation de l’historique des performances du mois dernier, mais si la création de quelques machines virtuelles dans l’environnement ne remonte qu’à une semaine. Dans ce cas, les données de performances pour les nouvelles machines virtuelles ne seront pas disponibles pour toute la durée et la note de confiance sera faible.

[Apprenez-en davantage](./concepts-assessment-calculation.md#confidence-ratings-performance-based) sur les notes de confiance.

## <a name="i-cant-see-some-groups-when-i-am-creating-an-azure-vmware-solution-avs-assessment"></a>Je ne peux pas voir certains groupes quand je crée une évaluation Azure VMware Solution (AVS)

- L’évaluation AVS peut être effectuée sur des groupes qui ne contiennent que des machines VMware. Supprimez du groupe les machines non-VMware si vous prévoyez d’effectuer une évaluation AVS.
- Si vous effectuez des évaluations AVS dans Azure Migrate pour la première fois, il est recommandé de créer un groupe de machines VMware.

## <a name="i-cant-see-some-vm-types-in-azure-government"></a>Je ne vois pas certains types de machines virtuelles dans Azure Government.

Les types de machines virtuelles pris en charge pour l’évaluation et la migration dépendent de la disponibilité dans l’emplacement Azure Government. Vous pouvez [examiner et comparer](https://azure.microsoft.com/global-infrastructure/services/?regions=usgov-non-regional,us-dod-central,us-dod-east,usgov-arizona,usgov-iowa,usgov-texas,usgov-virginia&products=virtual-machines) les types de machines virtuelles dans Azure Government.

## <a name="the-size-of-my-vm-changed-can-i-run-an-assessment-again"></a>La taille de ma machine virtuelle a changé. Puis-je réexécuter une évaluation ?

L’appliance Azure Migrate collecte en permanence des informations sur l’environnement local.  Une évaluation est une capture instantanée à une date et heure de machines virtuelles locales. Si vous changez les paramètres d’une machine virtuelle que vous voulez évaluer, utilisez l’option de recalcul pour mettre à jour l’évaluation avec les derniers changements.

## <a name="how-do-i-discover-vms-in-a-multitenant-environment"></a>Comment faire pour découvrir les machines virtuelles dans un environnement multilocataire ?

- **VMware** : Si un environnement est partagé entre des locataires et que vous ne voulez pas découvrir les machines virtuelles d’un locataire se trouvant dans l’abonnement d’un autre locataire, créez des informations d’identification de serveur VMware vCenter permettant d’accéder uniquement aux machines virtuelles que vous voulez découvrir. Utilisez ensuite ces informations d’identification quand vous démarrez la détection dans l’appliance Azure Migrate.
- **Hyper-V** : La découverte utilise les informations d’identification de l’hôte Hyper-V. Si les machines virtuelles partagent le même hôte Hyper-V, aucun moyen ne permet de séparer la détection.  

## <a name="do-i-need-vcenter-server"></a>Ai-je besoin de vCenter Server ?

Oui, Azure Migrate nécessite vCenter Server dans un environnement VMware pour effectuer la découverte. Azure Migrate ne prend pas en charge la découverte des hôtes ESXi qui ne sont pas gérés par vCenter Server.

## <a name="what-are-the-sizing-options-in-an-azure-vm-assessment"></a>Quelles sont les options de dimensionnement dans une évaluation de machine virtuelle Azure ?

Avec un dimensionnement local, Azure Migrate ne tient pas compte des données de performances des machines virtuelles pour l’évaluation. Azure Migrate évalue les tailles des machines virtuelles en fonction de la configuration locale. Avec un dimensionnement basé sur les performances, le dimensionnement est basé sur les données d’utilisation.

Par exemple, si une machine virtuelle locale a quatre cœurs et 8 Go de mémoire et présente 50 % d’utilisation du processeur et 50 % d’utilisation de la mémoire :
- Le dimensionnement local recommandera une référence (SKU) de machine virtuelle Azure dotée de quatre cœurs et de 8 Go de mémoire.
- Le dimensionnement basé sur les performances recommandera une référence SKU de machine virtuelle de deux cœurs et de 4 Go de mémoire, car le pourcentage d’utilisation est pris en compte.

De même, le dimensionnement de disque dépend du critère de dimensionnement et du type de stockage :
- Si le critère de dimensionnement est basé sur les performances et que le type de stockage est automatique, Azure Migrate prend en compte les valeurs de débit et d’IOPS du disque quand il identifie le type de disque cible (Standard ou Premium).
- Si le critère de dimensionnement est basé sur les performances et que le type de stockage est Premium, Azure Migrate recommande une référence SKU de disque Premium en fonction de la taille du disque local. La même logique est appliquée au dimensionnement du disque quand le dimensionnement est local et que le type de stockage est Standard ou Premium.

## <a name="does-performance-history-and-utilization-affect-sizing-in-an-azure-vm-assessment"></a>Est-ce que l’historique des performances et le niveau d’utilisation affectent l’évaluation d’une machine virtuelle Azure ?

Oui, l’historique des performances et le niveau d’utilisation affectent l’évaluation d’une machine virtuelle Azure.

### <a name="performance-history"></a>Historique des performances

Pour le dimensionnement basé sur les performances uniquement, Azure Migrate collecte l’historique des performances des machines locales, puis l’utilise pour recommander la taille de la machine virtuelle et le type de disque dans Azure :

1. L’appliance profile en continu l’environnement local pour collecter les données d’utilisation en temps réel toutes les 20 secondes.
2. L’appliance cumule les échantillons de 20 secondes collectés et les utilise pour créer un point de données unique toutes les 15 minutes.
3. Pour créer le point de données, l’appliance sélectionne la valeur maximale dans les échantillons de 20 secondes.
4. L’appliance envoie le point de données à Azure.

### <a name="utilization"></a>Utilisation

Quand vous créez une évaluation dans Azure (en fonction de la durée des performances et de la valeur de centile de l’historique des performances définie), Azure Migrate calcule la valeur d’utilisation effective, puis l’utilise pour le dimensionnement.

Par exemple, si vous définissez la durée des performances à un jour et la valeur de centile au 95e centile, Azure Migrate trie les points d’échantillonnage de 15 minutes envoyés par le collecteur pour le dernier jour dans l’ordre croissant. Il choisit la valeur du 95e centile comme utilisation effective.

L’utilisation de la valeur du 95e centile garantit que les valeurs hors norme sont ignorées. Des valeurs hors norme peuvent être incluses si votre appliance Azure Migrate utilise le 99e centile. Pour choisir l’utilisation maximale de la période sans manquer les valeurs hors norme, définissez Azure Migrate pour qu’il utilise le 99e centile.


## <a name="how-are-import-based-assessments-different-from-assessments-with-discovery-source-as-appliance"></a>En quoi les évaluations basées sur l’importation diffèrent-elles des évaluations avec une source de découverte sous forme d’appliance ?

Les évaluations basées sur l’importation sont des évaluations de machine virtuelle Azure créées avec des machines importées dans Azure Migrate à l’aide d’un fichier CSV. Seuls quatre champs sont obligatoires pour l’importation : nom du serveur, cœurs, mémoire et système d’exploitation. Voici quelques points à prendre en considération : 
 - Les critères de préparation sont moins stricts pour les évaluations basées sur l’importation vis-à-vis du paramètre de type de démarrage. Si le type de démarrage n’est pas fourni, la machine est supposée avoir le type de démarrage BIOS et ne pas être marquée comme **Préparée sous condition**. Pour les évaluations avec une source de découverte sous forme d’appliance, la préparation est marquée comme **Préparée sous condition** si le type de démarrage est manquant. Cette différence dans le calcul de la préparation est due au fait que les utilisateurs ne disposent peut-être pas de toutes les informations sur les machines au début de la planification de la migration, quand les évaluations basées sur l’importation sont effectuées. 
 - Les évaluations d’importation basées sur les performances utilisent la valeur d’utilisation fournie par l’utilisateur pour les calculs du dimensionnement correct. Étant donné que la valeur d’utilisation est fournie par l’utilisateur, les options **Historique des performances** et **Utilisation en centile** sont désactivées dans les propriétés d’évaluation. Pour les évaluations avec une source de découverte sous forme d’appliance, la valeur de centile est choisie parmi les données de performances collectées par l’appliance.

## <a name="why-is-the-suggested-migration-tool-in-import-based-avs-assessment-marked-as-unknown"></a>Pourquoi l’outil de migration suggéré dans l’évaluation AVS basé sur l’importation est-il marqué comme inconnu ?

Pour les machines importées via un fichier CSV, l’outil de migration par défaut dans l’évaluation AVS est inconnu. Pour les machines VMware, il est toutefois recommandé d’utiliser la solution VMware Hybrid Cloud Extension (HCX). [En savoir plus](../azure-vmware/tutorial-deploy-vmware-hcx.md)


## <a name="what-is-dependency-visualization"></a>Qu’est-ce que la visualisation des dépendances ?

La visualisation des dépendances permet d’évaluer les groupes de machines virtuelles à migrer avec une meilleure fiabilité. La visualisation des dépendances vérifie les dépendances croisées des machines avant que vous exécutiez une évaluation. Elle vous permet de vérifier que rien n’est oublié et ainsi d’éviter des interruptions inattendues quand vous migrez vers Azure. Azure Migrate utilise la solution Service Map dans Azure Monitor pour activer la visualisation des dépendances. [Plus d’informations](concepts-dependency-visualization.md)

> [!NOTE]
> L’analyse des dépendances basée sur les agents n’est pas disponible dans Azure Government. Vous pouvez utiliser l’analyse des dépendances sans agent.

## <a name="whats-the-difference-between-agent-based-and-agentless"></a>Quelle est la différence entre la visualisation basée sur les agents et la visualisation sans agent ?

Les différences entre la visualisation sans agent et la visualisation basée sur les agents sont résumées dans le tableau.

**Prérequis** | **Sans agent** | **Basé sur un agent**
--- | --- | ---
Support | Cette option est en préversion et est disponible uniquement pour les machines virtuelles VMware. [Vérifiez](migrate-support-matrix-vmware.md#dependency-analysis-requirements-agentless) quels sont les systèmes d’exploitation pris en charge. | En disponibilité générale.
Agent | Inutile d’installer des agents sur les machines qui doivent faire l’objet d’une vérification croisée. | Agents à installer sur chaque machine locale à analyser : [Microsoft Monitoring Agent (MMA)](../azure-monitor/platform/agent-windows.md) et l’[Agent de dépendances](../azure-monitor/platform/agents-overview.md#dependency-agent). 
Prérequis | [Passez en revue](concepts-dependency-visualization.md#agentless-analysis) les prérequis et la configuration requise pour le déploiement. | [Passez en revue](concepts-dependency-visualization.md#agent-based-analysis) les prérequis et la configuration requise pour le déploiement.
Log Analytics | Non requis. | Azure Migrate utilise la solution [Service Map](../azure-monitor/insights/service-map.md) dans [Journaux d’activité Azure Monitor](../azure-monitor/log-query/log-query-overview.md) pour la visualisation des dépendances. [Plus d’informations](concepts-dependency-visualization.md#agent-based-analysis)
Fonctionnement | Capture les données de connexion TCP sur les ordinateurs activés pour la visualisation des dépendances. Après la découverte, il collecte des données toutes les cinq minutes. | Les agents Service Map installés sur un ordinateur collectent des données sur les processus TCP et sur les connexions entrantes/sortantes pour chaque processus.
Données | Nom du serveur de l’ordinateur source, processus, nom de l’application.<br/><br/> Nom du serveur de l’ordinateur de destination, processus, nom de l’application et port. | Nom du serveur de l’ordinateur source, processus, nom de l’application.<br/><br/> Nom du serveur de l’ordinateur de destination, processus, nom de l’application et port.<br/><br/> Le nombre de connexions, la latence et les informations de transfert de données sont collectés et mis à la disposition des requêtes Log Analytics. 
Visualisation | La carte des dépendances d’un serveur unique peut être affichée sur une durée allant d’une heure à 30 jours. | Carte des dépendances d’un serveur unique.<br/><br/> La carte peut être affichée pendant une heure uniquement.<br/><br/> Carte des dépendances d’un groupe de serveurs.<br/><br/> Ajoutez et supprimez des serveurs dans un groupe à partir de la vue cartographique.
Exportation de données | Les données des 30 derniers jours peuvent être téléchargées dans un format CSV. | Les données peuvent être interrogées avec Log Analytics.


## <a name="do-i-need-to-deploy-the-appliance-for-agentless-dependency-analysis"></a>Est-ce que je dois déployer l’appliance pour l’analyse des dépendances sans agent ?

Oui, l’[appliance Azure Migrate](migrate-appliance.md) doit être déployée.

## <a name="do-i-pay-for-dependency-visualization"></a>Dois-je payer pour la visualisation des dépendances ?

Non. Apprenez-en davantage sur la [tarification Azure Migrate](https://azure.microsoft.com/pricing/details/azure-migrate/).

## <a name="what-do-i-install-for-agent-based-dependency-visualization"></a>Que dois-je installer pour la visualisation des dépendances basée sur les agents ?

Pour utiliser la visualisation des dépendances basée sur les agents, téléchargez et installez des agents sur chaque machine locale que vous souhaitez évaluer :

- [Microsoft Monitoring Agent (MMA)](../azure-monitor/platform/agent-windows.md)
- [Agent de dépendances](../azure-monitor/platform/agents-overview.md#dependency-agent)
- Si certaines de vos machines sont dépourvues de connexion Internet, téléchargez et installez la passerelle Log Analytics sur ces machines.

Vous avez besoin de ces agents uniquement si vous utilisez la visualisation des dépendances basée sur les agents.

## <a name="can-i-use-an-existing-workspace"></a>Puis-je utiliser un espace de travail existant ?

Oui, pour la visualisation des dépendances basée sur les agents, vous pouvez attacher un espace de travail existant au projet de migration et l’utiliser pour la visualisation des dépendances. 

## <a name="can-i-export-the-dependency-visualization-report"></a>Puis-je exporter le rapport de visualisation des dépendances ?

Non, le rapport de visualisation des dépendances dans la visualisation basée sur les agents ne peut pas être exporté. Toutefois, Azure Migrate utilise Service Map, et vous pouvez utiliser les [API REST Service Map](/rest/api/servicemap/machines/listconnections) pour récupérer les dépendances au format JSON.

## <a name="can-i-automate-agent-installation"></a>Puis-je automatiser l’installation de l’agent ?

Pour la visualisation des dépendances basée sur les agents :

- Utilisez un [script pour installer l’agent de dépendances](../azure-monitor/insights/vminsights-enable-hybrid.md#dependency-agent).
- Pour MMA, [utilisez la ligne de commande ou l’automatisation](../azure-monitor/platform/log-analytics-agent.md#installation-options), ou utilisez un [script](https://gallery.technet.microsoft.com/scriptcenter/Install-OMS-Agent-with-2c9c99ab).
- En plus des scripts, vous pouvez utiliser des outils de déploiement tels que Microsoft Endpoint Configuration Manager et [Intigua](https://www.intigua.com/intigua-for-azure-migration) pour déployer les agents.

## <a name="what-operating-systems-does-mma-support"></a>Quels sont les systèmes d’exploitation pris en charge par MMA ?

- Consultez la liste des [systèmes d’exploitation Windows pris en charge par MMA](../azure-monitor/platform/log-analytics-agent.md#installation-options).
- Consultez la liste des [systèmes d’exploitation Linux pris en charge par MMA](../azure-monitor/platform/log-analytics-agent.md#installation-options).

## <a name="can-i-visualize-dependencies-for-more-than-one-hour"></a>Puis-je visualiser des dépendances durant plus d’une heure ?

Pour la visualisation basée sur les agents, vous pouvez visualiser les dépendances durant une heure au maximum. Vous pouvez revenir à une date donnée de l’historique jusqu’à un mois, mais la durée maximale de visualisation est d’une heure. Par exemple, vous pouvez utiliser la durée mentionnée dans la carte des dépendances pour voir les dépendances d’hier, mais seulement pour une fenêtre d’une heure. Vous pouvez cependant utiliser les journaux Azure Monitor pour [interroger les données de dépendance](./how-to-create-group-machine-dependencies.md) sur une durée plus longue.

Pour la visualisation sans agent, vous pouvez voir la carte des dépendances d’un serveur unique sur une durée allant d’une heure à 30 jours.

## <a name="can-i-visualize-dependencies-for-groups-of-more-than-10-vms"></a>Puis-je visualiser les dépendances pour des groupes de plus de 10 machines virtuelles ?

Vous pouvez [visualiser les dépendances](./how-to-create-a-group.md#refine-a-group-with-dependency-mapping) de groupes ayant 10 machines virtuelles au maximum. En présence d’un groupe de plus de 10 machines virtuelles, nous vous recommandons de diviser le groupe en groupes plus petits, puis de visualiser les dépendances.

## <a name="next-steps"></a>Étapes suivantes

Consultez la [vue d’ensemble d’Azure Migrate](migrate-services-overview.md).