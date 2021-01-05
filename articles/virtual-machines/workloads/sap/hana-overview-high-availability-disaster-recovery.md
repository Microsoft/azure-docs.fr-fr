---
title: Haute disponibilité et récupération d’urgence de SAP HANA sur Azure (grandes instances) | Microsoft Docs
description: Établir la haute disponibilité et planifier la récupération d’urgence de SAP HANA sur Azure (grandes instances)
services: virtual-machines-linux
documentationcenter: ''
author: saghorpa
manager: juergent
editor: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/10/2018
ms.author: saghorpa
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 56e2e2a048e44dcad626208b059e258d55ba7057
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967597"
---
# <a name="sap-hana-large-instances-high-availability-and-disaster-recovery-on-azure"></a>Haute disponibilité et récupération d’urgence des grandes instances SAP HANA sur Azure 

>[!IMPORTANT]
>Cette documentation n’est en aucun cas un remplacement de la documentation d’administration de SAP HANA ou des Notes SAP. Le lecteur doit avoir une bonne compréhension et une expertise approfondie des opérations et de l’administration SAP HANA, principalement sur la sauvegarde, la restauration, la haute disponibilité et la récupération d’urgence.

Il est important de pratiquer les étapes et processus dans votre environnement, avec vos versions de HANA. Certains processus décrits dans cette documentation sont simplifiés pour une meilleure compréhension générale et ne sont pas destinés à être utilisés comme des étapes détaillées des manuels d’opération éventuels. Si vous souhaitez créer des manuels d’opération pour vos configurations, vous devez tester vos processus et les documenter en lien avec vos configurations spécifiques. 


La haute disponibilité et la récupération d’urgence constituent des aspects fondamentaux de l’exécution de vos serveurs stratégiques SAP HANA sur Azure (grandes instances). Il est important que vous collaboriez avec SAP, votre intégrateur de systèmes ou Microsoft pour concevoir et implémenter efficacement des stratégies de haute disponibilité et récupération d’urgence. Il est également important de tenir compte des objectifs de point de récupération (RPO) et de délai de récupération (RTO), qui sont propres à votre environnement.

Microsoft prend en charge certaines fonctionnalités de haute disponibilité de SAP HANA avec les grandes instances HANA. Ces fonctionnalités sont les suivantes :

- **Réplication du stockage** : capacité du système de stockage à répliquer toutes les données à un autre tampon de grande instance HANA dans une autre région Azure. SAP HANA opère indépendamment de cette méthode. Cette fonctionnalité est le mécanisme de récupération d’urgence par défaut pour les grandes instances HANA.
- **Réplication du système HANA** : La [réplication de toutes les données de SAP HANA](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/b74e16a9e09541749a745f41246a065e.html) pour un système SAP HANA à part. L’objectif de délai de récupération est réduit grâce à une réplication des données à intervalles réguliers. SAP HANA prend en charge les modes asynchrone, synchrone en mémoire et synchrone. Le mode synchrone n’est utilisé que pour les systèmes SAP HANA situés dans le même centre de données ou à moins de 100 km de distance. Avec la conception actuelle des horodatages de grande instance HANA, la réplication de système HANA ne peut garantir la haute disponibilité que dans une région. La réplication de système HANA nécessite un composant de routage ou proxy inverse tiers pour les récupérations d’urgence dans une autre région Azure. 
- **Basculement automatique de l’hôte** : solution de récupération après incident locale pour SAP HANA à utiliser comme alternative à la réplication de système HANA. Si le nœud principal n’est plus disponible, configurez un ou plusieurs nœuds SAP HANA de secours en mode montée en puissance parallèle, et SAP HANA bascule automatiquement vers un nœud de secours.

SAP HANA sur Azure (grandes instances) est disponible dans deux régions Azure qui couvrent quatre zones géopolitiques (États-Unis, Australie, Europe et Japon). Deux régions d’une zone géopolitique qui hébergent des horodatages de grande instance HANA sont connectées à des circuits réseau dédiés distincts. Ceux-ci servent à répliquer des captures instantanées de stockage pour fournir plusieurs méthodes de récupération après sinistre. La réplication n’est pas établie par défaut, mais configurée pour les client qui commandent la fonctionnalité de récupération d’urgence. La réplication de stockage dépend de l’utilisation des captures instantanées de stockage pour les grandes instances HANA. Il est impossible de choisir comme région de récupération d’urgence, une région Azure située dans une autre zone géopolitique. 

Le tableau suivant indique les combinaisons et méthodes de haute disponibilité et de récupération d’urgence actuellement prises en charge :

| Scénario pris en charge dans les grandes instances HANA | Option de haute disponibilité | Option de récupération d’urgence | Commentaires |
| --- | --- | --- | --- |
| Nœud unique | Non disponible. | Configuration de récupération d’urgence dédiée.<br /> Configuration de récupération d’urgence polyvalente. | |
| Basculement automatique avec hôte : Scale-out (avec ou sans unité de secours)<br /> y compris 1+1 | Possible avec nœud de secours en rôle actif.<br /> Contrôle par HANA de la permutation des rôles. | Configuration de récupération d’urgence dédiée.<br /> Configuration de récupération d’urgence polyvalente.<br /> Synchronisation de la récupération d’urgence à l’aide de la réplication du stockage. | Des jeux de volumes HANA sont attachés à tous les nœuds.<br /> Le site de récupération d’urgence doit avoir le même nombre de nœuds. |
| Réplication de système HANA | Possible avec configuration de réplica principal ou secondaire.<br /> Le réplica secondaire prend le rôle principal en cas de basculement.<br /> Réplication de système HANA et basculement du contrôle du système d’exploitation. | Configuration de récupération d’urgence dédiée.<br /> Configuration de récupération d’urgence polyvalente.<br /> Synchronisation de la récupération d’urgence à l’aide de la réplication du stockage.<br /> La récupération d’urgence à l’aide de la réplication de système HANA n’est pas possible sans composants tiers. | Des jeux distincts de volumes de disque sont attachés à chaque nœud.<br /> Seuls les volumes de disque de réplica secondaire sur le site de production sont répliqués à l’emplacement de la récupération d’urgence.<br /> Un jeu de volumes est requis sur le site de récupération d’urgence. | 

L’expression « configuration de récupération d’urgence dédiée » désigne une configuration où l’unité de grande instance HANA sur le site de récupération d’urgence n’est pas utilisée pour exécuter d’autres charges de travail ou systèmes de non-production. L’unité est passive et est déployée uniquement si un basculement d’urgence est exécuté. Cependant, cette configuration n’est le choix préféré de nombreux clients.

Consultez [Scénarios HLI pris en charge](hana-supported-scenario.md) pour connaître la disposition de stockage et les détails Ethernet de votre architecture.

> [!NOTE]
> [Les déploiements SAP HANA MCOD](https://launchpad.support.sap.com/#/notes/1681092) (plusieurs Instances HANA sur une unité) comme des scénarios de superpositions fonctionnent avec les méthodes de haute disponibilité et de récupération d’urgence répertoriées dans le tableau. L’utilisation de la réplication de système HANA avec un cluster de basculement automatique basé sur Pacemaker est une exception. Ce cas prend uniquement en charge une seule instance HANA par unité. Pour les déploiements [SAP HANA MDC](https://launchpad.support.sap.com/#/notes/2096000), seules les méthodes de haute disponibilité et de récupération d’urgence sans stockage fonctionnent si plusieurs locataires sont déployés. Si un seule locataire est déployé, toutes les méthodes indiquées sont valides.  

L’expression « configuration de récupération d’urgence polyvalente » désigne une configuration où l’unité de grande instance HANA sur le site de récupération d’urgence exécute une charge de travail de non-production. En cas de panne, arrêtez le système de non-production, montez les jeux de volumes (supplémentaires) répliqués par le stockage, puis démarrez l’instance HANA de production. La plupart des clients qui utilisent la fonctionnalité de récupération d’urgence des grandes instances HANA utilisent cette configuration. 


Vous trouverez plus d’informations sur la haute disponibilité de SAP HANA dans les articles SAP suivants : 

- [SAP HANA High Availability Whitepaper (Livre blanc sur la haute disponibilité de SAP HANA)](https://go.sap.com/documents/2016/05/f8e5eeba-737c-0010-82c7-eda71af511fa.html)
- [SAP HANA Administration Guide (Guide d’administration de SAP HANA)](https://help.sap.com/hana/SAP_HANA_Administration_Guide_en.pdf)
- [SAP Academy Video on SAP HANA System Replication (Vidéo SAP Academy sur la réplication du système SAP HANA)](https://scn.sap.com/community/hana-in-memory/blog/2015/05/19/sap-hana-system-replication)
- [SAP Support Note #1999880 – FAQ on SAP HANA System Replication (Note de support SAP n°1999880 – FAQ sur la réplication du système SAP HANA)](https://apps.support.sap.com/sap/support/knowledge/preview/en/1999880)
- [SAP Support Note #2165547 – SAP HANA Back up and Restore within SAP HANA System Replication Environment (Note de support SAP n°2165547 – Sauvegarde et restauration SAP HANA dans l’environnement de réplication du système SAP HANA)](https://websmp230.sap-ag.de/sap(bD1lbiZjPTAwMQ==)/bc/bsp/sno/ui_entry/entry.htm?param=69765F6D6F64653D3030312669765F7361706E6F7465735F6E756D6265723D3231363535343726)
- [SAP Support Note #1984882 – Using SAP HANA System Replication for Hardware Exchange with Minimum/Zero Downtime (Note de support SAP n°1984882 – Utilisation de la réplication du système SAP HANA pour l’échange de matériel avec un temps d’arrêt minime ou nul)](https://websmp230.sap-ag.de/sap(bD1lbiZjPTAwMQ==)/bc/bsp/sno/ui_entry/entry.htm?param=69765F6D6F64653D3030312669765F7361706E6F7465735F6E756D6265723D3139383438383226)

## <a name="network-considerations-for-disaster-recovery-with-hana-large-instances"></a>Considérations sur le réseau pour la récupération d’urgence avec de grandes instances HANA

Pour tirer parti de la fonctionnalité de récupération d’urgence des grandes instances HANA, vous devez concevoir la connectivité réseau des deux régions Azure. Vous avez besoin d’un circuit Azure ExpressRoute assurant la connexion depuis l’emplacement local dans votre région Azure principale, et d’un autre circuit pour la connexion entre l’emplacement local et votre région de récupération d’urgence. Cette mesure s’applique lorsqu’une région Azure, incluant l’emplacement MSEE (Microsoft Enterprise Edge Router), connaît un problème.

En guise de seconde mesure, vous pouvez connecter tous les réseaux virtuels Azure reliés à SAP HANA sur Azure (grandes instances) dans une région, au circuit ExpressRoute relié aux grandes instances HANA dans l’autre région. Avec cette *connexion croisée*, les services en cours d’exécution sur un réseau virtuel Azure de la région 1 peuvent se connecter aux unités de grande instance HANA dans la région 2, et inversement. Cette mesure s’applique lorsqu’un seul des emplacements MSEE qui connecte votre emplacement local à Azure se déconnecte.

Le graphique suivant illustre une configuration résistante aux cas de récupération d’urgence :

![Configuration optimale pour la récupération d’urgence](./media/hana-overview-high-availability-disaster-recovery/image1-optimal-configuration.png)



## <a name="other-requirements-with-hana-large-instances-storage-replication-for-disaster-recovery"></a>Autres exigences pour la réplication de stockage avec de grandes instances HANA dans le cadre d’une récupération d’urgence

Outre les exigences précédentes pour configurer une récupération d’urgence avec de grandes instances HANA, vous devez :

- Commander des références SKU SAP HANA sur Azure (grandes instances) de la même taille que vos références SKU de production et les déployer dans la région de récupération d’urgence. Dans les déploiements actuels des clients, ces instances sont utilisées pour exécuter des instances HANA de non-production. Ces configurations sont appelées *configurations de récupération d’urgence multi-usage*.   
- Commander du stockage supplémentaire sur le site de récupération d’urgence pour chacune de vos références SKU SAP HANA sur Azure (grandes instances) que vous souhaitez récupérer sur le site de récupération d’urgence. L’achat de stockage supplémentaire vous permet d’allouer les volumes de stockage. Vous pouvez allouer les volumes de stockage qui constituent la cible de la réplication de stockage entre votre région Azure de production et la région Azure de récupération d’urgence.
- Dans ce cas, si l’installation de HSR a été réalisée sur le réplica principal et que vous configurez la réplication basée sur le stockage sur le site de récupération d’urgence, vous devez acheter du stockage supplémentaire au niveau du site de récupération d’urgence, pour que les données des nœuds principaux et secondaires soient répliquées sur le site de récupération d’urgence.

  **Étapes suivantes**
- Voir [Sauvegarde et restauration](hana-backup-restore.md).













