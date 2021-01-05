---
title: À propos d’Azure Site Recovery
description: Présente le service Azure Site Recovery et récapitule les différents scénarios récupération d’urgence et de déploiement de la migration.
ms.topic: overview
ms.date: 03/17/2020
ms.custom: MVC
ms.openlocfilehash: 19860e64182cd73fe9f9fa1246f440a03109d465
ms.sourcegitcommit: b6f3ccaadf2f7eba4254a402e954adf430a90003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92281885"
---
# <a name="about-site-recovery"></a>À propos de Site Recovery

Bienvenue dans le service Azure Site Recovery. Cet article propose une vue d’ensemble rapide du service.

En tant qu’organisation, vous devez adopter une stratégie de continuité d’activité et de reprise d’activité (BCDR) capable de garantir la sécurité de vos données et de maintenir vos applications et vos charges de travail en ligne, lorsque des interruptions planifiées et non planifiées se produisent.

Azure Recovery Services contribue à votre stratégie BCDR :

- **Service Site Recovery** : Site Recovery permet d’assurer la continuité de l’activité en maintenant l’exécution des charges de travail et applications métier lors des interruptions. Site Recovery réplique les charges de travail s’exécutant sur des machines virtuelles et physiques depuis un site principal vers un emplacement secondaire. En cas d’interruption au niveau de votre site principal, vous basculez vers l’emplacement secondaire, depuis lequel vous pouvez accéder aux applications. Quand l’emplacement principal est de nouveau fonctionnel, vous pouvez effectuer une restauration automatique vers celui-ci.
- **Service Sauvegarde** : Le service [Sauvegarde Azure](../backup/index.yml) préserve la sécurité et la capacité de récupération de vos données.

Site Recovery peut gérer la réplication pour :

- Les machines virtuelles Azure qui répliquent des données entre des régions Azure.
- Machines virtuelles locales, machines virtuelles Azure Stack et serveurs physiques.

## <a name="what-does-site-recovery-provide"></a>À quoi sert Site Recovery ?

**Fonctionnalité** | **Détails**
--- | ---
**Solution BCDR simple** | Grâce à Site Recovery, vous pouvez configurer et gérer la réplication, le basculement et la restauration automatique à partir d’un seul emplacement dans le portail Azure.
**Réplication de machines virtuelles Azure** | Vous pouvez configurer la récupération d’urgence de machines virtuelles Azure d’une région primaire vers une région secondaire.
**Réplication de machines virtuelles locales** | Vous pouvez répliquer des machines virtuelles et des serveurs physiques locaux sur Azure ou vers un centre de données local secondaire. La réplication sur Azure réduit le coût et la complexité qu’implique la maintenance d’un centre de données secondaire.
**Réplication de charges de travail** | Répliquez n’importe quelle charge de travail en cours d’exécution sur des machines virtuelles Azure, des machines virtuelles Hyper-V locales et VMware, et des serveurs physiques Windows/Linux pris en charge.
**Résilience des données** | Site Recovery orchestre la réplication sans intercepter les données d’une application. Lorsque vous effectuez une réplication vers Azure, les données sont stockées dans le stockage Azure, avec la résilience que cela implique. Lors du basculement, les machines virtuelles Azure sont créées à partir des données répliquées.
**Cibles RTO et RPO** | Maintenez les objectifs de délai de récupération (RTO) et les objectifs de point de récupération (RPO) au sein des limites fixées par l’organisation. Site Recovery fournit une réplication continue pour les machines virtuelles Azure et VMware. Les machines virtuelles Hyper-V bénéficient d’une fréquence de réplication de 30 secondes seulement. Vous pouvez réduire davantage les RTO en les intégrant à [Azure Traffic Manager](https://azure.microsoft.com/blog/reduce-rto-by-using-azure-traffic-manager-with-azure-site-recovery/).
**Maintenir la cohérence des applications en cas de basculement** | Vous pouvez effectuer une réplication à l’aide des points de récupération avec des captures instantanées cohérentes au niveau des applications. Ces captures récupèrent les données des disques, l’ensemble des données en mémoire et toutes les transactions en cours de traitement.
**Tests sans interruption** | Vous pouvez facilement exécuter la marche à suivre en cas de récupération d’urgence et ce, sans affecter une réplication en cours.
**Basculements flexibles** | Vous pouvez effectuer des basculements planifiés en cas d’interruptions prévues sans aucune perte de données. Vous pouvez également effectuer des basculements non planifiés avec une perte de données minimale (en fonction de la fréquence de réplication) en cas de sinistres inattendus. Vous pouvez effectuer en toute simplicité une restauration automatique vers votre site principal dès qu’il est à nouveau disponible.
**Plans de récupération personnalisés** | À l’aide de plans de récupération, vous pouvez personnaliser et séquencer le basculement et la récupération des applications multiniveaux s’exécutant sur plusieurs machines virtuelles. Vous regroupez des machines dans un plan de récupération et ajoutez éventuellement des scripts et des actions manuelles. Les plans de récupération peuvent être intégrés à des runbooks Azure Automation.
**Intégration BCDR** | Site Recovery s’intègre à d’autres technologies BCDR. Par exemple, vous pouvez utiliser Site Recovery pour protéger le serveur principal SQL Server de charges de travail d’entreprise, avec la prise en charge native de SQL Server AlwaysOn pour gérer le basculement des groupes de disponibilité.
**Intégration Azure Automation** | La bibliothèque Azure Automation diversifiée fournit des scripts spécifiques à l’application prêts pour la production, qui peuvent être téléchargés et intégrés au service Site Recovery.
**Intégration réseau** | Site Recovery est intégré à Azure pour la gestion du réseau d’applications. Par exemple, pour réserver des adresses IP, configurez des équilibreurs de charge et utilisez Azure Traffic Manager pour des basculements réseau efficaces.

## <a name="what-can-i-replicate"></a>Que puis-je répliquer ?

**Pris en charge** | **Détails**
--- | ---
**Scénarios de réplication** | Répliquez des machines virtuelles Azure d’une région Azure vers une autre.<br/><br/>  Répliquez des machines virtuelles VMware, des machines virtuelles Hyper-V, des serveurs physiques (Windows et Linux) et des machines virtuelles Azure Stack locales sur Azure.<br/><br/> Répliquez des instances Windows AWS sur Azure.<br/><br/> Répliquer des machines virtuelles VMware locales, des machines virtuelles Hyper-V gérées par System Center VMM et des serveurs physiques vers un site secondaire.
**Régions** | Consultez les [régions prises en charge](https://azure.microsoft.com/global-infrastructure/services/?products=site-recovery) pour Site Recovery. |
**Machines répliquées** | Passez en revue la configuration requise pour la réplication de [machines virtuelles Azure](azure-to-azure-support-matrix.md#replicated-machine-operating-systems), de [machines virtuelles locales VMware et de serveurs physiques](vmware-physical-azure-support-matrix.md#replicated-machines),et de [machines virtuelles locales Hyper-V](hyper-v-azure-support-matrix.md#replicated-vms).
**Charges de travail** | Vous pouvez répliquer toutes les charges de travail qui s’exécutent sur une machine prenant en charge la réplication. En outre, l’équipe Site Recovery a effectué des tests spéciaux pour [plusieurs applications](site-recovery-workload.md#workload-summary).

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur la [prise en charge de la charge de travail](site-recovery-workload.md).
- Prise en main de [la réplication d’une machine virtuelle Azure entre des régions](azure-to-azure-quickstart.md).
