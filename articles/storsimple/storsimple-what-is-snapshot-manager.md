---
title: Qu’est-ce que le Gestionnaire d’instantanés StorSimple ? | Microsoft Docs
description: Décrit le Gestionnaire d’instantanés StorSimple, son architecture et ses fonctionnalités.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 6094c31e-e2d9-4592-8a15-76bdcf60a754
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 02/27/2017
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 2214f085c30419cefb3f6f84139d5592873173f7
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017982"
---
# <a name="an-introduction-to-storsimple-snapshot-manager"></a>Présentation de StorSimple Snapshot Manager

## <a name="overview"></a>Vue d’ensemble
Gestionnaire d’instantanés StorSimple est un composant logiciel enfichable Microsoft Management Console (MMC) qui simplifie la protection des données et la gestion des sauvegardes dans l’environnement Microsoft Azure StorSimple. Avec le Gestionnaire d’instantanés StorSimple, vous pouvez gérer les données Microsoft Azure StorSimple présentes dans le centre de données et dans le cloud comme avec une solution de stockage intégrée unique, ce qui simplifie les processus de sauvegarde et contribue à réduire les coûts.

Cette vue d'ensemble présente le Gestionnaire d'instantanés StorSimple, décrit ses caractéristiques et explique son rôle dans Microsoft Azure StorSimple. 

Pour avoir une vue d’ensemble de l’ensemble du système de Microsoft Azure StorSimple, et notamment de l’unité StorSimple, du service StorSimple Manager, du Gestionnaire d’instantanés StorSimple et de l’adaptateur StorSimple pour SharePoint, consultez [StorSimple série 8000 : une solution de stockage de cloud hybride](storsimple-overview.md). 

> [!NOTE]
> * Vous ne pouvez pas utiliser le Gestionnaire d'instantanés StorSimple pour gérer les baies virtuelles Microsoft Azure StorSimple (également appelées appareils virtuels locaux StorSimple).
> * Si vous prévoyez d’installer StorSimple Update 2 sur votre appareil StorSimple, veillez à télécharger la dernière version de StorSimple Snapshot Manager et à l’installer **avant d’installer StorSimple Update 2**. La dernière version de StorSimple Snapshot Manager est à compatibilité descendante et fonctionne avec toutes les versions publiées de Microsoft Azure StorSimple. Si vous utilisez la version précédente de StorSimple Snapshot Manager, vous devez effectuer une mise à jour (il est inutile de désinstaller la version précédente avant d’installer la nouvelle).
> 
> 

## <a name="storsimple-snapshot-manager-purpose-and-architecture"></a>Architecture et objectif du gestionnaire d’instantanés StorSimple
Le gestionnaire d’instantanés StorSimple met à votre disposition une console de gestion centralisée que vous pouvez utiliser pour créer des copies de sauvegarde de données locales et cloud ponctuelles. Par exemple, vous pouvez utiliser la console pour :

* Configurer, sauvegarder et supprimer des volumes.
* Configurer des groupes de volumes pour garantir que les données sauvegardées sont cohérentes pour l’application.
* Gérer les stratégies de sauvegarde afin que les données soient sauvegardées selon un calendrier prédéterminé.
* Créer des instantanés de Cloud et locaux pouvant être stockés dans le cloud et utilisés pour la récupération d’urgence.

StorSimple Snapshot Manager récupère la liste des applications inscrites auprès du fournisseur VSS sur l’hôte. Ensuite, pour créer des sauvegardes cohérentes avec les applications, il vérifie les volumes utilisés par une application et suggère des groupes de volumes à configurer. Le Gestionnaire d’instantanés StorSimple utilise ces groupes de volumes pour générer des copies de sauvegarde cohérentes avec les applications. (On parle de cohérence des applications lorsque tous les fichiers et bases de données associés sont synchronisés et reflètent l’état réel de l’application à un moment précis dans le temps.) 

Les sauvegardes de Gestionnaire d’instantanés StorSimple prennent la forme d’instantanés incrémentiels, qui capturent uniquement les modifications apportées depuis la dernière sauvegarde. Par conséquent, les sauvegardes nécessitent moins d’espace de stockage et peuvent être créées et restaurées rapidement. Le Gestionnaire d’instantanés StorSimple utilise le service Windows Volume Shadow Copy Service (VSS) garantir que les instantanés contiennent des données cohérentes avec les applications. (Pour plus d’informations, consultez la section Intégration à Windows Volume Shadow Copy Service). Avec le Gestionnaire d’instantanés StorSimple, vous pouvez créer des planifications de sauvegarde ou effectuer des sauvegardes immédiates en fonction des besoins. Si vous avez besoin de restaurer des données à partir d’une sauvegarde par la suite, le gestionnaire d’instantanés StorSimple vous permet de choisir dans le catalogue d’instantanés locaux ou de cloud. StorSimple Azure restaure uniquement les données nécessaires quand elles sont utiles, ce qui permet d’éviter des retards de disponibilité des données pendant les opérations de restauration.

![Architecture du Gestionnaire d’instantanés StorSimple](./media/storsimple-what-is-snapshot-manager/HCS_SSM_Overview.png)

**Architecture du Gestionnaire d’instantanés StorSimple** 

## <a name="support-for-multiple-volume-types"></a>Prise en charge de plusieurs types de volumes
Vous pouvez utiliser le Gestionnaire d’instantanés StorSimple pour configurer et sauvegarder les types de volumes suivants : 

* **Volumes de base** : un volume de base est une partition unique sur un disque de base. 
* **Volumes simples** : un volume simple est un volume dynamique qui contient l’espace d’un seul disque dynamique. Un volume simple se compose d’une seule région d’un disque ou de plusieurs régions liées entre elles sur le même disque. (Vous pouvez créer des volumes simples uniquement sur des disques dynamiques). Les volumes simples ne sont pas tolérants aux pannes.
* **Volumes dynamiques** : un volume dynamique est un volume créé sur un disque dynamique. Les disques dynamiques utilisent une base de données pour effectuer le suivi des informations sur les volumes contenus sur les disques dynamiques d’un ordinateur. 
* **Volumes dynamiques avec mise en miroir** : les volumes dynamiques avec mise en miroir reposent sur l’architecture RAID 1. Avec RAID 1, des données identiques sont écrites sur deux ou plusieurs disques, produisant un ensemble en miroir. Une demande de lecture peut ensuite être traitée par n’importe quel disque contenant les données demandées.
* **Volumes partagés de cluster** : avec les volumes partagés de cluster (CSV), plusieurs nœuds dans un cluster de basculement peuvent lire ou écrire simultanément sur le même disque. Le basculement d’un nœud sur un autre nœud peut se produire rapidement, sans nécessiter la modification de propriété du lecteur ou le montage, démontage et la suppression d’un volume. 

> [!IMPORTANT]
> Ne mélangez pas les volumes partagés de cluster et les volumes non partagés de cluster dans le même instantané. Le mélange de volumes partagés de cluster et de volumes non partagés de cluster dans un instantané n’est pas pris en charge. 
> 
> 

Vous pouvez utiliser le Gestionnaire d’instantanés StorSimple pour restaurer des groupes de volume complets ou cloner et récupérer des fichiers individuels.

* [Volumes et groupes de volumes](#volumes-and-volume-groups) 
* [Types de sauvegarde et stratégies de sauvegarde](#backup-types-and-backup-policies) 

Pour plus d’informations sur les fonctionnalités du Gestionnaire d’instantanés StorSimple et leur utilisation, consultez la section [Interface utilisateur du gestionnaire d’instantanés StorSimple](storsimple-use-snapshot-manager.md).

## <a name="volumes-and-volume-groups"></a>Volumes et groupes de volumes
Avec le Gestionnaire d’instantanés StorSimple, vous créez dans un premier temps des volumes, puis configurez en groupes de volumes. 

Le Gestionnaire d’instantanés StorSimple utilise des groupes de volumes pour créer des copies de sauvegarde cohérentes avec les applications. On parle de cohérence des applications lorsque tous les fichiers connexes et les bases de données sont synchronisés et reflètent l’état réel de l’application à un moment précis dans le temps. Les groupes de volumes (également appelés *groupes de cohérence*) constituent la base d’une tâche de sauvegarde ou de restauration.

Les groupes de volumes ne sont pas équivalents aux conteneurs de volumes. Un conteneur de volume contient un ou plusieurs volumes qui partagent un compte de stockage cloud et d’autres attributs, tels que le chiffrement et la consommation de bande passante. Un conteneur de volumes peut contenir jusqu’à 256 volumes StorSimple alloués de façon dynamique. Pour plus d’informations sur les conteneurs de volumes, consultez la page [Gérer vos conteneurs de volumes](./storsimple-8000-manage-volume-containers.md). Les groupes de volumes sont des ensembles de volumes que vous configurez pour faciliter les opérations de sauvegarde. Si vous sélectionnez deux volumes appartenant à des conteneurs de volumes distincts, placez-les dans un seul groupe de volumes, puis créez une stratégie de sauvegarde pour ce groupe de volumes. Chaque volume sera sauvegardé dans le conteneur de volumes approprié à l’aide du compte de stockage approprié.

> [!NOTE]
> Tous les volumes d’un groupe de volumes doivent provenir d’un fournisseur de services cloud unique.
> 
> 

## <a name="integration-with-windows-volume-shadow-copy-service"></a>Intégration à Windows Volume Shadow Copy Service
Le Gestionnaire d’instantanés StorSimple utilise le service Windows Volume Shadow Copy Service (VSS) pour recueillir des données cohérentes avec les applications. VSS facilite la cohérence des applications en communiquant avec des applications compatibles VSS pour coordonner la création d’instantanés incrémentiels. Le service VSS vérifie que les applications sont provisoirement désactivées ou inactives, quand les instantanés sont créés. 

L’implémentation du gestionnaire d’instantanés StorSimple de VSS fonctionne avec SQL Server et les volumes NTFS génériques. Pour ce faire, procédez comme suit : 

1. Le demandeur, en général une solution de gestion et de protection des données (par exemple Gestionnaire d’instantanés StorSimple) ou une application de sauvegarde, appelle le service VSS et lui demande de recueillir des informations du logiciel enregistreur dans l’application cible.
2. Le service VSS contacte le composant enregistreur pour récupérer une description des données. L’enregistreur retourne la description des données à sauvegarder. 
3. VSS demande à l’enregistreur de préparer l’application de sauvegarde. L’enregistreur prépare les données pour la sauvegarde en terminant des transactions en cours, en mettant à jour des journaux d’activité de transactions et ainsi de suite, puis notifie VSS.
4. VSS ordonne à l’enregistreur d’arrêter provisoirement le stockage de données de l’application et s’assure qu’aucune donnée n’est écrite dans le volume pendant la création du cliché instantané. Cette étape garantit la cohérence des données et ne prend pas plus de 60 secondes.
5. Le service VSS demande au fournisseur de créer le cliché instantané. Les fournisseurs logiciel ou matériel gèrent les modules en cours d’exécution et créent des clichés instantanés d’eux à la demande. Le fournisseur crée le cliché instantané et informe le service VSS à la fin de l’opération.
6. Le service VSS contacte l’enregistreur pour notifier l’application d’entrée/sortie peut reprendre son fonctionnement et confirmer que les entrées/sorties ont été interrompues avec succès pendant la création de clichés instantanés. 
7. Si la copie a réussi, VSS renvoie l’emplacement de la copie au demandeur. 
8. Si les données ont été écrites pendant la création du cliché instantané, la sauvegarde est incohérente. VSS supprime le cliché instantané et informe le demandeur. Le demandeur peut alors répéter automatiquement le processus de sauvegarde automatiquement ou informer l’administrateur pour qu’il renouvelle l’opération ultérieurement.

Consultez l’illustration suivante.

![Processus VSS](./media/storsimple-what-is-snapshot-manager/HCS_SSM_VSS_process.png)

**Processus du service VSS Windows** 

## <a name="backup-types-and-backup-policies"></a>Types de sauvegarde et stratégies de sauvegarde
Avec le Gestionnaire d’instantanés StorSimple, vous pouvez sauvegarder les données et les stocker localement et dans le cloud. Vous pouvez utiliser le Gestionnaire d’instantanés StorSimple pour sauvegarder les données immédiatement, ou vous pouvez utiliser une stratégie de sauvegarde pour créer une planification des sauvegardes et les exécuter automatiquement. Les stratégies de sauvegarde vous permettent également de spécifier le nombre d’instantanés qui sera conservé. 

### <a name="backup-types"></a>Types de sauvegarde
Vous pouvez utiliser le Gestionnaire d’instantanés StorSimple pour créer les types de sauvegardes suivants :

* **Instantanés locaux** : les instantanés locaux sont des copies ponctuelles dans le temps de données de volume stockées sur l’appareil StorSimple. En règle générale, ce type de sauvegarde peut être créé et restauré rapidement. Vous pouvez utiliser un instantané local comme vous utiliseriez une copie de sauvegarde locale.
* **Instantanés cloud** : les instantanés cloud sont des copies ponctuelles de données de volume stockées dans le cloud. Un instantané cloud est équivalent à un instantané répliqué sur un système de stockage hors site distinct. Les instantanés cloud sont particulièrement utiles dans les scénarios de récupération d’urgence.

### <a name="on-demand-and-scheduled-backups"></a>Sauvegardes à la demande et planifiées
Avec le Gestionnaire d’instantanés StorSimple, vous pouvez lancer une sauvegarde ponctuelle à créer immédiatement, ou vous pouvez utiliser une stratégie de sauvegarde pour planifier des opérations de sauvegarde récurrentes.

Une stratégie de sauvegarde est un ensemble de règles automatisées que vous pouvez utiliser pour planifier des sauvegardes régulières. Une stratégie de sauvegarde permet de définir la fréquence et les paramètres de création des instantanés d’un groupe de volumes spécifique. Vous pouvez utiliser des stratégies pour spécifier les dates de début et d’expiration, des heures, les fréquences et les exigences de conservation liées à aux instantanés locaux et aux instantanés du cloud. Une stratégie est appliquée dès que vous l’avez définie. 

Vous pouvez utiliser le Gestionnaire d’instantanés StorSimple pour configurer ou reconfigurer les stratégies de sauvegarde chaque fois que c’est nécessaire. 

Pour chaque stratégie de sauvegarde que vous créez, vous devez définir les informations suivantes :

* **Nom** : nom unique de la stratégie de sauvegarde sélectionnée.
* **Type** : type de stratégie de sauvegarde (instantané local ou instantané cloud).
* **Groupe de volumes** : groupe de volumes auquel la stratégie de sauvegarde sélectionnée est affectée.
* **Rétention** : nombre de copies de sauvegarde à conserver. Si vous cochez la case **Toutes**, toutes les copies de sauvegarde sont conservées jusqu’à ce que le nombre maximal de copies de sauvegarde par volume soit atteint. À ce moment-là, la stratégie échoue et génère un message d’erreur. Vous pouvez également spécifier un nombre de sauvegardes à conserver (entre 1 et 64).
* **Date** : date de création de la stratégie de sauvegarde.

Pour plus d’informations sur la configuration des stratégies de sauvegarde, consultez la section [Utiliser le Gestionnaire d’instantanés StorSimple pour créer et gérer des stratégies de sauvegarde](storsimple-snapshot-manager-manage-backup-policies.md).

### <a name="backup-job-monitoring-and-management"></a>Analyse et gestion des tâches de sauvegarde
Vous pouvez utiliser le Gestionnaire d’instantanés StorSimple pour surveiller et gérer les travaux de sauvegarde à venir, planifiées et terminées. De plus, le Gestionnaire d’instantanés StorSimple offre un catalogue pouvant comporter jusqu’à 64 sauvegardes terminées. Vous pouvez utiliser ce catalogue pour rechercher et restaurer des volumes ou des fichiers individuels. 

Pour plus d’informations sur l’analyse des tâches de sauvegarde, accédez au [Gestionnaire d’instantanés StorSimple pour afficher et gérer des tâches de sauvegarde](storsimple-snapshot-manager-manage-backup-jobs.md).

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur [l’utilisation du Gestionnaire d’instantanés StorSimple pour gérer votre solution StorSimple](storsimple-snapshot-manager-admin.md).
* [Télécharger le Gestionnaire d’instantanés StorSimple](https://www.microsoft.com/download/details.aspx?id=44220).