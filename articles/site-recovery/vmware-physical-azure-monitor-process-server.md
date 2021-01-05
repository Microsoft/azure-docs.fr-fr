---
title: Surveiller le serveur de processus Azure Site Recovery
description: Cet article explique comment surveiller le serveur de traitement Azure Site Recovery utilisé pour la récupération d’urgence d’un serveur physique/d’une machine virtuelle VMware.
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/14/2019
ms.author: raynew
ms.openlocfilehash: eebaa70cee99380ac67b8f6516a5b08ff2832c86
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86134736"
---
# <a name="monitor-the-process-server"></a>Surveiller le serveur de traitement.

Cet article explique comment surveiller le serveur de processus [Site Recovery](site-recovery-overview.md).

- Le serveur de processus est utilisé pour la configuration d’une récupération d’urgence de machines virtuelles VMware et de serveurs physiques sur Azure.
- Le serveur de processus s’exécute par défaut sur le serveur de configuration. Il est installé par défaut lorsque vous déployez le serveur de configuration.
- Le cas échéant, pour mettre à l’échelle et gérer des nombres plus importants de machines répliquées et des volumes plus conséquents de trafic de réplication, vous pouvez déployer des serveurs de traitement Scale-out supplémentaires.

[Apprenez-en davantage](vmware-physical-azure-config-process-server-overview.md) sur le rôle et le déploiement de serveurs de processus.

## <a name="monitoring-overview"></a>Vue d'ensemble de la surveillance

Étant donné que le serveur de processus a tant de rôles, notamment pour la mise en cache, la compression et le transfert de données répliquées vers Azure, il est important de surveiller en permanence son état d’intégrité.

Il existe un certain nombre de situations qui affectent couramment les performances du serveur de processus. Les problèmes affectant les performances ont un effet en cascade sur l’intégrité des machines virtuelles, qui finit par mettre le serveur de processus et ses machines répliquées dans un état critique. Les situations possibles sont les suivantes :

- Le nombre de machines virtuelles utilisant un serveur de processus approche ou dépasse les limitations recommandées.
- Le taux de variation des machines virtuelles utilisant le serveur de processus est élevé.
- Le débit réseau entre les machines virtuelles et le serveur de processus ne suffit pas pour charger des données de réplication sur ce dernier.
- Le débit réseau entre le serveur de processus et Azure est insuffisant pour charger des données de réplication du serveur vers Azure.

Tous ces problèmes peuvent affecter l’objectif de point de récupération (RPO) des machines virtuelles. 

**Pourquoi ?** Parce que la génération d’un point de récupération pour une machine virtuelle nécessite que tous les disques de la machine virtuelle aient un point commun. Si un disque présente un taux de variation élevé, si la réplication est lente ou si le serveur de processus n’est pas dans un état optimal, cela a une incidence sur l’efficacité de la création de points de récupération.

## <a name="monitor-proactively"></a>Surveiller de manière proactive

Pour éviter des problèmes avec le serveur de processus, il est important de :

- Comprendre les exigences spécifiques des serveurs de processus en suivant les [recommandations de capacité et de dimensionnement ](site-recovery-plan-capacity-vmware.md#capacity-considerations), et veiller à ce que les serveurs de processus soient déployés et fonctionnent conformément aux recommandations.
- Surveiller les alertes et résoudre les problèmes à mesure qu’ils se produisent, de façon à ce que les serveurs de processus fonctionnent efficacement.


## <a name="process-server-alerts"></a>Alertes du serveur de processus

Le serveur de processus génère un certain nombre d’alertes d’intégrité, résumées dans le tableau suivant.

**Type d’alerte** | **Détails**
--- | ---
![Healthy][green] | Le serveur de processus est connecté et sain.
![Avertissement][yellow] | L’utilisation du processeur a été supérieure à 80 % durant les 15 dernières minutes.
![Avertissement][yellow] | L’utilisation de la mémoire a été supérieure à 80 % durant les 15 dernières minutes.
![Avertissement][yellow] | Espace libre du dossier cache < 30 % durant les 15 dernières minutes.
![Avertissement][yellow] | Site Recovery supervise les données en attente/sortantes toutes les cinq minutes et estime que les données présentes dans le cache de serveur de processus ne peuvent pas être chargées dans Azure dans les 30 minutes.
![Avertissement][yellow] | Les services du serveur de processus n’ont pas fonctionné durant les 15 dernières minutes.
![Critique][red] | L’utilisation du processeur a été supérieure à 95 % durant les 15 dernières minutes.
![Critique][red] | L’utilisation de la mémoire a été supérieure à 95 % durant les 15 dernières minutes.
![Critique][red] | L’espace libre du dossier de cache a été supérieur à 25 % durant les 15 dernières minutes.
![Critique][red] | Site Recovery supervise les données en attente/sortantes toutes les cinq minutes et estime que les données présentes dans le cache de serveur de processus ne peuvent pas être chargées dans Azure dans les 45 minutes.
![Critique][red] | Le serveur de processus n’a émis aucune pulsation pendant 15 minutes.

![Clé de table](./media/vmware-physical-azure-monitor-process-server/table-key.png)

> [!NOTE]
> L’état d’intégrité global du serveur de processus est basé sur la pire alerte générée.



## <a name="monitor-process-server-health"></a>Surveiller l’intégrité du serveur de processus

Vous pouvez surveiller l’état d’intégrité de vos serveurs de processus comme suit : 

1. Pour surveiller l’intégrité et l’état de la réplication d’une machine répliquée et de son serveur de processus, dans le coffre, accédez à **Éléments répliqués**, puis cliquez sur la machine à surveiller.
2. Dans **Intégrité de la réplication**, vous pouvez surveiller l’état d’intégrité de machine virtuelle. Cliquez sur l’état pour consulter les détails de l’erreur.

    ![Intégrité du serveur de processus dans le tableau de bord de machine virtuelle](./media/vmware-physical-azure-monitor-process-server/vm-ps-health.png)

4. Dans **Intégrité du serveur de processus**, vous pouvez surveiller l’état du serveur de processus. Explorez les détails.

    ![Détails du serveur de processus dans le tableau de bord de machine virtuelle](./media/vmware-physical-azure-monitor-process-server/ps-summary.png)

5. Vous pouvez également surveiller l’intégrité à l’aide de la représentation graphique sur la page de la machine virtuelle.
    - Un serveur de processus Scale-out est mis en surbrillance orange si des avertissements y sont associés, et rouge s’il présente des problèmes critiques. 
    - Si le serveur de processus s’exécute dans le déploiement par défaut sur le serveur de configuration, celui-ci est mis en surbrillance en conséquence.
    - Pour explorer, cliquez sur le serveur de configuration ou le serveur de processus. Notez les problèmes et les suggestions de correction.

Vous pouvez également surveiller des serveurs de processus dans le coffre sous **Infrastructure Site Recovery**. Dans **Gérer votre infrastructure Site Recovery**, cliquez sur **Serveurs de configuration**. Sélectionnez le serveur de configuration associé au serveur de processus, puis explorez-en les détails.


## <a name="next-steps"></a>Étapes suivantes

- Si vous rencontrez des problèmes avec des serveurs de processus, suivez nos [instructions de dépannage](vmware-physical-azure-troubleshoot-process-server.md).
- Si vous avez besoin d’aide supplémentaire, publiez votre question sur la [page de questions Microsoft Q&R sur Azure Site Recovery](/answers/topics/azure-site-recovery.html). 

[green]: ./media/vmware-physical-azure-monitor-process-server/green.png
[yellow]: ./media/vmware-physical-azure-monitor-process-server/yellow.png
[red]: ./media/vmware-physical-azure-monitor-process-server/red.png
