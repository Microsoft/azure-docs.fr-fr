---
title: Migration de SAP HANA sur Azure (grandes instances) vers des machines virtuelles Azure | Microsoft Docs
description: Migration de SAP HANA sur Azure (grandes instances) vers des machines virtuelles Azure
services: virtual-machines-linux
documentationcenter: ''
author: bentrin
manager: juergent
editor: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 02/11/2020
ms.author: bentrin
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 25eae9f9ba0e28a5aa069972c8c7d5eb2877545f
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967684"
---
# <a name="sap-hana-on-azure-large-instance-migration-to-azure-virtual-machines"></a>Migration de SAP HANA sur grande instance Azure vers des machines virtuelles Azure
Cet article décrit les scénarios possibles de déploiement d'une grande instance Azure et propose une approche de planification et de migration qui réduit le temps d'arrêt lors de la transition.

## <a name="overview"></a>Vue d’ensemble
Depuis l'annonce du lancement des grandes instances Azure pour SAP HANA (HLI) en septembre 2016, de nombreux clients ont adopté cette offre « Hardware as-a-service » pour leur plateforme de calcul en mémoire.  Ces dernières années, l'extensibilité des machines virtuelles Azure combinée à la prise en charge du déploiement scale-out HANA a permis de dépasser la demande de capacité de base de données ERP de la plupart des entreprises clientes.  Des clients se montrent intéressés pour migrer leur charge de travail SAP HANA de leurs serveurs physiques vers des machines virtuelles Azure.
Ce guide n’est pas un document de configuration pas à pas. Il décrit les modèles de déploiement courants et fournit des conseils de planification et de migration.  Il a pour objectif d'insister sur les éléments à prendre en compte lors de la préparation afin de réduire les temps d’arrêt liés à la transition.

## <a name="assumptions"></a>Hypothèses
Cet article repose sur les hypothèses suivantes :
- L'unique intérêt est de bénéficier d'une migration homogène du service de calcul de base de données HANA d'un serveur HLI (Hana Large Instance) vers une machine virtuelle Azure sans mise à niveau logicielle ou mise à jour corrective significative. Ces mises à jour mineures incluent l'utilisation d'une version plus récente du système d'exploitation ou d'une version prise en charge de HANA, tel qu'explicitement indiqué dans les notes SAP pertinentes. 
- Toutes les activités de mise à jour/mise à niveau doivent être effectuées avant ou après la migration.  Par exemple, conversion de SAP HANA MCOS en déploiement MDC. 
- La réplication du système SAP HANA (HSR) est l'approche de migration qui offrirait le moins de temps d'arrêt. Ce document ne couvre pas les autres méthodes de migration.
- Ce guide s'applique à la fois aux références SKU Rev3 et Rev4 de HLI.
- L'architecture de déploiement HANA reste pour l'essentiel inchangée pendant la migration.  Autrement dit, un système doté d'une seule instance DR restera le même à l'emplacement de destination.
- Les clients ont lu et compris le Contrat de niveau de service (SLA) de l'architecture cible (à venir). 
- Sur le plan commercial, il existe des différences entre les serveurs HLI et les machines virtuelles. Les clients doivent surveiller l’utilisation de leurs machines virtuelles en termes de gestion des coûts.
- Les clients comprennent que HLI est une plateforme de calcul dédiée, tandis que les machines virtuelles s’exécutent sur une infrastructure partagée mais isolée.
- Les clients ont confirmé que les machines virtuelles cibles prennent en charge l'architecture prévue. Pour connaître les références SKU de toutes les machines virtuelles certifiées pour le déploiement de SAP HANA, consultez le [Répertoire de matériel SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure).
- Les clients ont validé le plan de conception et de migration.
- Planifiez la machine virtuelle de récupération d'urgence en même temps que le site principal.  Les clients ne peuvent pas utiliser le nœud de récupération d'urgence HLI pour le site principal exécuté sur les machines virtuelles après la migration.
- Les clients ont copié les fichiers de sauvegarde requis sur les machines virtuelles cibles, conformément aux exigences de récupération et de conformité de l'entreprise. Les sauvegardes accessibles aux machines virtuelles permettent une récupération jusqu’à une date et heure au cours de la période de transition.
- Pour la haute disponibilité de la réplication HSR, les clients doivent installer et configurer l'appareil STONITH conformément aux guides SAP HANA HA relatifs à [SLES](./high-availability-guide-suse-pacemaker.md) et [RHEL](./high-availability-guide-rhel-pacemaker.md).  Elle n'est pas préconfigurée comme dans le cas de HLI.
- Cette approche de migration ne couvre pas les références SKU HLI avec configuration Optane.

## <a name="deployment-scenarios"></a>Scénarios de déploiement
Les modèles de déploiement courants associés aux clients HLI sont résumés dans le tableau suivant.  La migration vers des machines virtuelles Azure est possible pour tous les scénarios HLI.  Pour tirer parti des services Azure complémentaires disponibles, des modifications architecturales mineures peuvent s'avérer nécessaires.

| ID du scénario | Scénario HLI | Migrer vers une machine virtuelle ? | Remarque |
| --- | --- | --- | --- |
| 1 | [Nœud unique avec un SID](./hana-supported-scenario.md#single-node-with-one-sid) | Oui | - |
| 2 | [Nœud unique avec MCOS](./hana-supported-scenario.md#single-node-mcos) | Oui | - |
| 3 | [Nœud unique avec récupération d'urgence à l'aide de la réplication du stockage](./hana-supported-scenario.md#single-node-with-dr-using-storage-replication) | Non | La réplication du stockage n'est pas disponible avec la plateforme virtuelle Azure. Remplacez la solution de récupération d'urgence actuelle par la réplication HSR ou par la sauvegarde/restauration |
| 4 | [Nœud unique avec récupération d'urgence (polyvalente) à l'aide de la réplication du stockage](./hana-supported-scenario.md#single-node-with-dr-multipurpose-using-storage-replication) | Non | La réplication du stockage n'est pas disponible avec la plateforme virtuelle Azure. Remplacez la solution de récupération d'urgence actuelle par la réplication HSR ou par la sauvegarde/restauration |
| 5 | [Réplication HSR avec STONITH pour une haute disponibilité](./hana-supported-scenario.md#hsr-with-stonith-for-high-availability) | Oui | Aucun SBD préconfiguré pour les machines virtuelles cibles.  Sélectionnez et déployez une solution STONITH.  Options possibles : Agent de délimitation Azure (pris en charge pour [RHEL](./high-availability-guide-rhel-pacemaker.md) et [SLES](./high-availability-guide-suse-pacemaker.md)), SBD |
| 6 | [Haute disponibilité avec réplication HSR, récupération d'urgence avec réplication du stockage](./hana-supported-scenario.md#high-availability-with-hsr-and-dr-with-storage-replication) | Non | Remplacez la réplication du stockage pour les besoins de récupération d'urgence par la réplication HSR ou par la sauvegarde/restauration |
| 7 | [Basculement automatique de l'hôte (1+1)](./hana-supported-scenario.md#host-auto-failover-11) | Oui | Utilisez ANF pour le stockage partagé avec des machines virtuelles Azure |
| 8 | [Scale-out avec nœud de secours](./hana-supported-scenario.md#scale-out-with-standby) | Oui | BW/4HANA avec machines virtuelles M128s, M416s et M416ms utilisant ANF pour le stockage uniquement |
| 9 | [Scale-out sans nœud de secours](./hana-supported-scenario.md#scale-out-without-standby) | Oui | BW/4HANA avec machines virtuelles M128s, M416s, M416ms (avec ou sans utilisation d'ANF pour le stockage) |
| 10 | [Scale-out avec récupération d'urgence à l'aide de la réplication du stockage](./hana-supported-scenario.md#scale-out-with-dr-using-storage-replication) | Non | Remplacez la réplication du stockage pour les besoins de récupération d'urgence par la réplication HSR ou par la sauvegarde/restauration |
| 11 | [Nœud unique avec récupération d'urgence à l'aide de HSR](./hana-supported-scenario.md#single-node-with-dr-using-hsr) | Oui | - |
| 12 | [Réplication HSR à nœud unique vers site de récupération d'urgence (à coût optimisé)](./hana-supported-scenario.md#single-node-hsr-to-dr-cost-optimized) | Oui | - |
| 13 | [Haute disponibilité et récupération d'urgence avec réplication HSR](./hana-supported-scenario.md#high-availability-and-disaster-recovery-with-hsr) | Oui | - |
| 14 | [Haute disponibilité et récupération d'urgence avec réplication HSR (à coût optimisé)](./hana-supported-scenario.md#high-availability-and-disaster-recovery-with-hsr-cost-optimized) | Oui | - |
| 15 | [Scale-out avec récupération d'urgence à l'aide de la réplication HSR](./hana-supported-scenario.md#scale-out-with-dr-using-hsr) | Oui | BW/4HANA avec machines virtuelles M128s, M416s, M416ms (avec ou sans utilisation d'ANF pour le stockage) |


## <a name="source-hli-planning"></a>Planification de la source (HLI)
Lors de l'intégration d'un serveur HLI, Microsoft Service Management et les clients ont planifié les paramètres spécifiques au calcul, au réseau, au stockage et au système d'exploitation pour l'exécution de la base de données SAP HANA.  Une planification similaire doit être effectuée pour la migration vers une machine virtuelle Azure.

### <a name="sap-hana-housekeeping"></a>Tâches de nettoyage SAP HANA 
Il est recommandé de nettoyer le contenu de la base de données pour éviter de migrer des données indésirables/obsolètes et des journaux périmés vers la nouvelle base de données.  Les tâches de nettoyage consistent généralement à supprimer ou à archiver des données anciennes, périmées ou inactives.  Ces opérations liées à l'« hygiène des données » doivent être testées en mode hors production afin de confirmer leur validité avant une utilisation en production.

### <a name="allow-network-connectivity-for-new-vms-and-or-virtual-network"></a>Activer la connectivité réseau pour les nouvelles machines virtuelles et/ou le réseau virtuel 
Dans le cadre du déploiement HLI d'un client, le réseau a été configuré sur la base des informations décrites dans l'article [Architecture réseau de SAP HANA (grandes instances)](./hana-network-architecture.md). En outre, le routage du trafic réseau s'effectue comme décrit à la section « Routage dans Azure ».
- Lors de la configuration d'une nouvelle machine virtuelle en tant que cible de migration, si elle est placée sur le réseau virtuel existant avec des plages d'adresses IP déjà autorisées à se connecter au HLI, aucune mise à jour supplémentaire de la connectivité n'est nécessaire.
- Si la nouvelle machine virtuelle Azure est placée sur un nouveau réseau virtuel Microsoft Azure, qui peut se trouver dans une autre région, et appairée au réseau virtuel existant, la clé de service ExpressRoute et l'ID de ressource de l'approvisionnement HLI d'origine peuvent être utilisés pour autoriser l'accès à la plage d'adresses IP de ce nouveau réseau virtuel.  Coordonnez-vous avec Microsoft Service Management pour activer la connectivité entre le réseau virtuel et HLI.  Remarque : pour réduire le temps de réponse du réseau entre les couches d'application et de base de données, ces deux couches doivent se trouver sur le même réseau virtuel.  

### <a name="existing-app-layer-availability-set-availability-zones-and-proximity-placement-group-ppg"></a>Groupe à haute disponibilité, zones de disponibilité et groupe de placement de proximité de la couche d'application existante (PPG)
Le modèle de déploiement actuel a été conçu pour respecter certains objectifs de niveau de service.  Vous devez donc veiller à ce que l'infrastructure cible atteigne ou dépasse les objectifs fixés.  
Généralement, les serveurs d'application SAP des clients sont placés dans un groupe à haute disponibilité.  Si le niveau de service du déploiement actuel est satisfaisant et 
- Si la machine virtuelle cible utilise le nom d’hôte du nom logique HLI, la mise à jour de la résolution d’adresse DNS pointant vers l’adresse IP de la machine virtuelle fonctionne sans mettre à jour les profils SAP
- Si vous n'utilisez pas groupe de placement de proximité (PPG), veillez à placer tous les serveurs d'applications et de bases de données dans la même zone pour réduire le temps de réponse du réseau.
- Si vous utilisez un PPG, reportez-vous à la section « Planification de la destination, groupe à haute disponibilité, zones de disponibilité et groupe de placement de proximité (PPG) » de ce document.

### <a name="storage-replication-discontinuance-process-if-used"></a>Processus d'interruption de la réplication du stockage (le cas échéant)
Si la réplication du stockage est utilisée en tant que solution de récupération d'urgence, elle doit être interrompue (sans planification) après la fermeture de l'application SAP.  En outre, le dernier catalogue SAP HANA, le dernier fichier journal et les dernières sauvegardes de données sont répliqués sur les volumes de stockage HLI DR distants.  Il s'agit-là d'une précaution en cas de problème lors de la transition du serveur physique vers la machine virtuelle Azure.

### <a name="data-backups-preservation-consideration"></a>Considérations relatives à la conservation des sauvegardes de données
Après le basculement vers SAP HANA sur machine virtuelle Azure, les données basées sur des captures instantanées ou les sauvegardes de fichiers journaux figurant sur le serveur HLI sont difficilement accessibles ou restaurables sur une machine virtuelle, si besoin. Durant la période de transition préliminaire, avant que la sauvegarde basée sur Azure n'établisse un historique suffisant pour répondre aux exigences de récupération jusqu'à une date et heure, nous vous recommandons d'effectuer des sauvegardes au niveau des fichiers en plus des captures instantanées sur le serveur HLI au cours des jours ou semaines précédant le basculement.  Veillez à ce que ces sauvegardes soient copiées sur un compte de stockage Azure accessible à la nouvelle machine virtuelle SAP HANA.
Outre la sauvegarde du contenu du serveur HLI, il est recommandé de veiller à ce que des sauvegardes complètes du paysage SAP soient facilement accessibles au cas où une restauration serait nécessaire.

### <a name="adjusting-system-monitoring"></a>Ajustement de la surveillance du système 
Les clients utilisent de nombreux outils différents afin de surveiller les systèmes de leur paysage SAP et d'envoyer des notifications d'alerte sur ceux-ci.  Il s'agit ici d'attirer l'attention sur l'action qui convient afin d'intégrer des modifications pour la surveillance et de mettre à jour les destinataires des notifications d'alerte, si nécessaire.

### <a name="microsoft-operations-team-involvement"></a>Implication de l'équipe des opérations de Microsoft 
Ouvrez un ticket à partir du portail Azure sur la base de l'instance HLI existante.  Une fois le ticket de support créé, un ingénieur du support vous contactera par e-mail.  

### <a name="engage-microsoft-account-team"></a>Impliquer l'équipe des comptes Microsoft
Planifiez la migration à l'approche de la date anniversaire du renouvellement du contrat HLI afin d'éviter les dépenses inutiles en termes de ressources de calcul. Pour désactiver le panneau HLI, la résiliation du contrat et l'arrêt effectif de l'unité doivent être coordonnés.

## <a name="destination-planning"></a>Planification de la destination
La mise en place d'une nouvelle infrastructure pour remplacer une infrastructure existante nécessite de la réflexion.  Vous trouverez ci-dessous quelques points clés à prendre en considération.

### <a name="resource-availability-in-the-target-region"></a>Disponibilité des ressources dans la région cible 
La région de déploiement actuelle du serveur d'applications SAP se trouve généralement à proximité immédiate des serveurs HLI associés.  Toutefois, les emplacements disponibles pour les serveurs HLI sont moins nombreux que les régions Azure.  La migration de serveurs HLI physiques vers des machines virtuelles Azure est également l'occasion d'affiner la distance de proximité de tous les services associés afin d'optimiser les performances.  Ce faisant, il convient de s'assurer que la région choisie dispose de toutes les ressources requises.  Par exemple, la disponibilité de certaines familles de machines virtuelles ou l'offre de zones Azure pour la configuration de la haute disponibilité.

### <a name="virtual-network"></a>Réseau virtuel 
Les clients doivent choisir d'exécuter la nouvelle base de données HANA dans un réseau virtuel existant ou d'en créer un nouveau.  Ce choix dépendra principalement du schéma actuel de mise en réseau du paysage SAP.  De même, lorsque l’infrastructure passe d’un déploiement à une zone à un déploiement à deux zones et utilise PPG, un changement architectural s'impose. Pour plus d'informations, consultez l'article [Groupes de placement de proximité Azure pour un temps de réponse optimal du réseau avec les applications SAP](./sap-proximity-placement-scenarios.md).   

### <a name="security"></a>Sécurité
Que la nouvelle machine virtuelle SAP HANA se trouve sur un réseau/ sous-réseau virtuel nouveau ou existant, elle représente un nouveau service essentiel à l'entreprise qui doit être protégé.  Un contrôle d'accès conforme à la stratégie de sécurité des informations de l'entreprise doit être évalué et déployé pour cette nouvelle classe de service.

### <a name="vm-sizing-recommendation"></a>Recommandation relative au redimensionnement des machines virtuelles
Cette migration est également l'occasion de redimensionner correctement votre moteur de calcul HANA.  Vous pouvez utiliser les [vues du système](https://help.sap.com/viewer/7c78579ce9b14a669c1f3295b0d8ca16/Cloud/3859e48180bb4cf8a207e15cf25a7e57.html) HANA en conjonction avec HANA Studio pour analyser la consommation des ressources système, ce qui permettra un redimensionnement adéquat et une amélioration de la rentabilité.

### <a name="storage"></a>Stockage 
Les performances de stockage comptent parmi les facteurs qui influencent l'expérience utilisateur des applications SAP.  Sur la base d'une référence SKU de machine virtuelle donnée, une disposition de stockage minimale est publiée : [Configurations du stockage des machines virtuelles SAP HANA Azure](./hana-vm-operations-storage.md). Nous vous recommandons de prendre connaissance de ces spécifications minimales et de les comparer aux statistiques du système HLI existant afin de garantir une capacité et des performances d'E/S adéquates pour la nouvelle machine virtuelle HANA.

Si vous configurez un PPG pour la nouvelle machine virtuelle HANA et ses serveurs, soumettez un ticket de support afin d'inspecter et d'assurer la colocalisation du stockage et de la machine virtuelle. Dans la mesure où votre solution de sauvegarde peut nécessiter des modifications, le coût de stockage doit également être revu afin d'éviter des dépenses opérationnelles imprévues.

### <a name="storage-replication-for-disaster-recovery"></a>Réplication du stockage pour la récupération d'urgence
Avec HLI, la réplication du stockage constituait l'option par défaut en termes de récupération d'urgence. Pour SAP HANA sur machine virtuelle Azure, cette fonctionnalité n'est pas l'option par défaut. Envisagez la réplication HSR, la sauvegarde/restauration ou d'autres solutions prises en charge en fonction des besoins de votre entreprise.

### <a name="availability-sets-availability-zones-and-proximity-placement-groups"></a>Groupes à haute disponibilité, zones de disponibilité et groupes de placement de proximité 
Pour réduire la distance entre la couche d'application et SAP HANA afin de maintenir le temps de réponse du réseau à un niveau minimum, la nouvelle machine virtuelle de base de données et les serveurs d'applications SAP actuels doivent être placés dans un PPG. Consultez [Groupe de placement de proximité](./sap-proximity-placement-scenarios.md) pour en savoir plus sur le fonctionnement du groupe à haute disponibilité et des zones de disponibilité Azure avec un PPG dans le cadre des déploiements SAP.
Si les membres du système HANA cible sont déployés dans plusieurs zones Azure, les clients doivent avoir une vision claire du profil de latence des zones choisies. Le placement des composants du système SAP est optimal en ce qui concerne la distance proximale entre l'application SAP et la base de données.  L'[outil de test de latence de la zone de disponibilité](https://github.com/Azure/SAP-on-Azure-Scripts-and-Utilities/tree/master/AvZone-Latency-Test) du domaine public facilite la mesure.  


### <a name="backup-strategy"></a>Stratégie de sauvegarde
De nombreux clients utilisent déjà des solutions de sauvegarde tierces pour SAP HANA sur HLI.  Dans ce cas, il suffit de configurer une machine virtuelle et une base de données HANA protégées supplémentaires.  Les tâches de sauvegarde HLI en cours peuvent ne plus être planifiées si la machine est désactivée après la migration.
La fonctionnalité Sauvegarde Azure pour SAP HANA sur machine virtuelle est désormais généralement disponible.  Cliquez sur ces liens pour obtenir des informations détaillées sur les éléments suivants : [Sauvegarde](../../../backup/backup-azure-sap-hana-database.md), [Restauration](../../../backup/sap-hana-db-restore.md), [Gestion](../../../backup/sap-hana-db-manage.md) de la sauvegarde SAP HANA dans les machines virtuelles Azure.

### <a name="dr-strategy"></a>Stratégie de récupération d'urgence
Si vos objectifs de niveau de service permettent un temps de récupération plus long, une simple sauvegarde sur Stockage Blob et une restauration sur place ou sur une nouvelle machine virtuelle constituent la stratégie de récupération d'urgence la plus simple et la moins coûteuse.  
Comme sur la plateforme de grande instance où la récupération d'urgence HANA s'effectue généralement à l'aide de la réplication HSR, sur une machine virtuelle Azure, la réplication HSR est également la solution de récupération d'urgence SAP HANA la plus naturelle et la plus native.  Que le déploiement source corresponde à instance unique ou un cluster, un réplica de l’infrastructure source est requis dans la région de récupération d’urgence.
Ce réplica de récupération d'urgence sera configuré au terme de la migration du serveur HLI principal vers la machine virtuelle.  La base de données HANA de récupération d'urgence s'enregistrera auprès de l'instance principale SAP HANA sur machine virtuelle en tant que site de réplication secondaire.  

### <a name="sap-application-server-connectivity-destination-change"></a>Changement de la destination de connectivité du serveur d'applications SAP
La migration HSR se traduit par un nouvel hôte de base de données HANA et donc par un nouveau nom d'hôte de base de données pour la couche d'application. Les profils SAP doivent être modifiés pour refléter le nouveau nom d'hôte.  Si le changement s'effectue par résolution de noms en conservant le nom d'hôte, aucune modification de profil n'est requise.

### <a name="operating-system"></a>Système d’exploitation
Les images de système d'exploitation du serveur HLI et de la machine virtuelle, même si elles présentent le même niveau de version, SLES 12 SP4 par exemple, ne sont pas identiques. Les clients doivent valider les packages, le noyau et les différents correctifs requis sur le serveur HLI afin d'installer les mêmes packages sur la cible.  L’utilisation de HSR à des fins de réplication d'un ancien système d'exploitation vers une machine virtuelle dotée d'une version plus récente du système d'exploitation est prise en charge.  Vérifiez les versions spécifiques prises en charge en consultant [Note SAP 2763388](https://launchpad.support.sap.com/#/notes/2763388).

### <a name="new-sap-license-request"></a>Demande de nouvelle licence SAP
Il s'agit simplement de demander une nouvelle licence SAP pour le nouveau système HANA maintenant qu'il a été migré vers des machines virtuelles.

### <a name="service-level-agreement-sla-differences"></a>Différences de contrats de niveau de service (SLA)
Les auteurs tiennent à rappeler qu'il existe une différence de SLA de disponibilité entre un serveur HLI et une machine virtuelle Azure.  Par exemple, les paires haute disponibilité des serveurs HLI en cluster offrent une disponibilité de 99,99 %. Pour bénéficier du même contrat SLA, vous devez déployer les machines virtuelles dans des zones de disponibilité. Cet [article](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_9/) décrit la disponibilité avec les architectures de déploiement associées afin de permettre aux clients de planifier leur infrastructure cible en conséquence.


## <a name="migration-strategy"></a>Stratégie de migration
Ce document couvre uniquement l'approche de réplication du système HANA pour la migration entre un serveur HLI et une machine virtuelle Azure.  Selon la solution de stockage cible déployée, le processus peut être légèrement différent. Les principales étapes sont décrites ci-dessous.

### <a name="vm-with-premiumultra-disks-for-data"></a>Machine virtuelle avec disques Premium/Ultra pour les données
Pour les machines virtuelles déployées avec des disques Premium ou Ultra, la configuration standard de la réplication du système SAP HANA peut s'appliquer pour configurer la réplication HSR.  L'[article d'aide SAP](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.04/099caa1959ce4b3fa1144562fa09e163.html) donne un aperçu des étapes de configuration de la réplication, de basculement vers un système secondaire, de restauration sur le système principal et de désactivation de la réplication système.  Dans le cadre de la migration, nous aurons uniquement besoin des étapes de configuration, de basculement et de désactivation de la réplication.  

### <a name="vm-with-anf-for-data-and-log-volumes"></a>Machine virtuelle avec ANF pour les volumes de données et de journaux
À un niveau élevé, les dernières captures instantanées de stockage HLI de la totalité des volumes de données et de journaux doivent être copiées dans Stockage Azure où elles sont accessibles et récupérables par la machine virtuelle HANA cible.  La copie peut être effectuée avec n'importe quel outil de copie Linux natif.  

> [!IMPORTANT]
> La copie et le transfert de données peuvent prendre des heures en fonction de la taille de la base de données HANA et de la bande passante réseau disponible.  La majeure partie de la copie doit être effectuée avant le temps d'arrêt de la base de données HANA principale.

### <a name="mcos-to-mdc-conversion"></a>Conversion de MCOS en MDC
Le modèle de déploiement MCOS (Multiple Components in One System) a été utilisé par certains de nos clients HLI.  Leur motivation était de contourner la limitation des captures instantanées de stockage MDC (Multiple Databases Container) des versions antérieures de SAP HANA.  Dans le modèle MCOS, plusieurs instances SAP HANA indépendantes sont empilées sur un même panneau HLI.  L'utilisation de la réplication HSR pour la migration fonctionnait bien, mais impliquait plusieurs machines virtuelles HANA dotées d'une base de données locataire chacune.  Cet état final crée un paysage plus animé que celui auquel un client était peut-être habitué.  Dans la mesure où le déploiement par défaut de SAP HANA 2.0 repose sur le modèle MDC, une alternative viable consiste à [déplacer le locataire HANA](https://launchpad.support.sap.com/#/notes/2472478) après la migration HSR.  Ce processus « regroupe » ces bases de données HANA indépendantes en colocataires au sein d'un même conteneur HANA.  

### <a name="application-layer-consideration"></a>Considérations relatives à la couche d'application
Le serveur de bases de données est considéré comme le centre d'un système SAP.  Tous les serveurs d'applications doivent se trouver à proximité de la base de données SAP HANA.  Dans certains cas, lorsqu'une nouvelle utilisation du PPG est souhaitée, le déplacement des serveurs d'applications existants sur le PPG où se trouve la machine virtuelle HANA peut être requis.  La création de nouveaux serveurs d'applications peut être jugée plus facile si vous disposez déjà de modèles de déploiement.  
Si la localisation des serveurs d'applications existants et de la nouvelle machine virtuelle HANA est optimale, il n'est pas nécessaire de créer de nouveaux serveurs d'applications, sauf si vous avez besoin de capacité supplémentaire.  
Si une nouvelle infrastructure est créée pour améliorer la disponibilité des services, les serveurs d'applications existants peuvent devenir inutiles. Ils doivent alors être arrêtés et supprimés.
Si le nom d'hôte de la machine virtuelle cible a changé et qu'il est différent du nom d'hôte HLI, les profils des serveurs d'applications SAP doivent être modifiés pour pointer vers le nouvel hôte.  Si seule l'adresse IP de la base de données HANA a changé, une mise à jour de l'enregistrement DNS est nécessaire pour diriger les connexions entrantes vers la nouvelle machine virtuelle HANA.

### <a name="acceptance-test"></a>Test d'acceptation
Bien que la migration d'un serveur HLI vers une machine virtuelle n'apporte aucun changement significatif au contenu de la base de données par rapport à une migration hétérogène, nous recommandons toujours de valider les fonctionnalités clés et l'aspect performance de la nouvelle configuration.  

### <a name="cutover-plan"></a>Plan de basculement
Bien que cette migration soit simple, elle implique toutefois la désactivation d'une base de données existante.  Une planification minutieuse visant à préserver le système source, son contenu et les images de sauvegarde est essentielle au cas où une solution de secours serait nécessaire.  Une bonne planification permet une restauration plus rapide.   


## <a name="post-migration"></a>Après la migration
La tâche de migration n'est pas terminée tant que nous n'avons pas procédé à un découplage sécurisé des services ou de la connectivité du serveur HLI pour garantir l'intégrité des données.  Arrêtez également les services inutiles.  Cette section présente quelques points importants.

### <a name="decommissioning-the-hli"></a>Désactivation du serveur HLI
Après une migration réussie de la base de données HANA vers une machine virtuelle Azure, assurez-vous qu'aucune transaction commerciale de production ne s'exécute sur la base de données HLI.  Cependant, il est recommandé de maintenir le serveur HLI opérationnel pendant une période égale à sa fenêtre locale de rétention des sauvegardes pour permettre une récupération plus rapide, si besoin.  Ce n'est qu'alors que le panneau HLI doit être désactivé.  Les clients doivent conclure des engagements contractuels HLI avec Microsoft en contactant leurs représentants Microsoft.

### <a name="remove-any-proxy-ex-iptables-bigip-configured-for-hli"></a>Supprimer n'importe quel proxy (par exemple : Iptables, BIGIP) configuré pour HLI 
Si un service proxy comme IPTables est utilisé pour acheminer le trafic local vers et depuis le serveur HLI, il n'est plus nécessaire après une migration réussie vers une machine virtuelle.  Toutefois, ce service de connectivité doit être conservé aussi longtemps que le panneau HLI reste opérationnel.  N'arrêtez le service qu'une fois le panneau HLI entièrement désactivé.

### <a name="remove-global-reach-for-hli"></a>Supprimer Global Reach pour HLI 
Global Reach est utilisé pour connecter la passerelle ExpressRoute des clients à la passerelle ExpressRoute du serveur HLI.  Cela permet au trafic local des clients d'atteindre directement le locataire HLI, sans recourir à un service proxy.  Cette connexion n’est plus nécessaire en l’absence de l’unité HLI après la migration.  Comme pour le service proxy IPTables, Global Reach doit aussi être conservé jusqu'à ce que le panneau HLI soit entièrement désactivé.

### <a name="operating-system-subscription--movereuse"></a>Abonnement au système d'exploitation - déplacer/réutiliser
Une fois les serveurs des machines virtuelles mis en place et les panneaux HLI désactivés, les abonnements aux systèmes d'exploitation peuvent être remplacés ou réutilisés afin d'éviter de payer des licences en double.



## <a name="next-steps"></a>Étapes suivantes
Reportez-vous aux articles suivants :
- [Configurations et opérations relatives à l'infrastructure SAP HANA sur Azure](./hana-vm-operations.md).
- [Charges de travail SAP sur Azure : liste de contrôle relative à la planification et au déploiement](./sap-deployment-checklist.md)
