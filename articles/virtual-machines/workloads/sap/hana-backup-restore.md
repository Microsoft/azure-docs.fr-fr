---
title: Sauvegarde et restauration HANA sur SAP HANA sur Azure (grandes instances) | Microsoft Docs
description: Sauvegarde et restauration HANA sur SAP HANA sur Azure (grandes instances)
services: virtual-machines-linux
documentationcenter: ''
author: saghorpa
manager: gwallace
editor: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 10/16/2020
ms.author: saghorpa
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 42333a3feae19b6c3c77494f7e843cac1d9bc078
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96006317"
---
# <a name="backup-and-restore-of-sap-hana-on-hana-large-instances"></a>Sauvegarde et restauration de SAP HANA sur de grandes instances HANA

>[!IMPORTANT]
>Cet article n’est en aucun cas un remplacement de la documentation d’administration de SAP HANA ou des Notes SAP. Nous assumons que vous avez une bonne compréhension et une expertise approfondie des opérations et de l’administration SAP HANA, principalement de la sauvegarde, de la restauration, de la haute disponibilité et de la récupération d’urgence. Dans cet article, des captures d’écran de SAP HANA Studio sont affichées. Le contenu, la structure et la nature des écrans des outils d’administration de SAP, ainsi que les outils eux-mêmes, peuvent changer selon la version de SAP HANA.

Il est important de pratiquer les étapes et processus dans votre environnement, avec vos versions de HANA. Certains processus décrits dans cet article sont simplifiés pour une meilleure compréhension générale. Ils ne sont pas destinés à être utilisés comme des étapes détaillées pour d’éventuels manuels d’opération. Si vous souhaitez créer des manuels d’opération pour vos configurations, testez vos processus et documentez-les par rapport à vos configurations spécifiques. 

L’un des aspects les plus importants de l’utilisation des bases de données concerne la protection de celles-ci contre les événements graves. Ces événements peuvent avoir une multitude de causes, des catastrophes naturelles aux simples erreurs des utilisateurs.

La sauvegarde d’une base de données, combinée à une possibilité de restauration à un moment donné quelconque (par exemple, avant la suppression de données critiques), permet de restaurer la base de données à un état aussi proche que possible de celui qu’elle présentait avant l’interruption.

Deux types de sauvegardes doivent être effectués pour permettre la restauration :

- Sauvegardes de base de données : sauvegardes complètes, incrémentielles ou différentielles
- Sauvegardes des journaux de transactions

Vous pouvez également compléter les sauvegardes de base de données complètes exécutées au niveau application par des sauvegardes effectuées à l’aide de captures instantanées de stockage. Les captures instantanées de stockage ne remplacent pas les sauvegardes de fichier journal. Celles-ci restent importantes pour restaurer la base de données à un certain point dans le temps ou pour supprimer des journaux d’activité les transactions déjà validées. Les captures instantanées de stockage peuvent accélérer la récupération en fournissant rapidement une image de restauration par progression de la base de données. 

SAP HANA sur Azure (grandes instances) offre deux options de sauvegarde et de restauration :

- **Utilisation de vos propres méthodes de sauvegarde.** Après avoir vérifié l’espace disque requis, effectuez des sauvegardes complètes de la base de données et des journaux à l’aide de l’une des méthodes de sauvegarde de disque. Vous pouvez sauvegarder soit directement sur des volumes attachés aux unités de grande instance HANA, soit sur des configurations de partage NFS dans une machine virtuelle Azure. Dans le deuxième cas, les clients paramètrent une machine virtuelle Linux dans Azure, attachent le stockage Azure à la machine virtuelle et partagent le stockage dans celle-ci par le biais d’un serveur NFS configuré. Si vous effectuez la sauvegarde sur des volumes directement attachés aux unités de grande instance HANA, copiez les sauvegardes vers un compte de stockage Azure. Faites-le après avoir configuré une machine virtuelle Azure qui exporte les partages NFS basés sur le stockage Azure, ou bien utilisez un coffre de sauvegarde Azure ou un stockage froid Azure. 

   Une autre possibilité consiste à recourir à un outil de protection des données tiers pour stocker les sauvegardes une fois ces dernières copiées dans un compte de stockage Azure. L’utilisation de vos propres méthodes de sauvegarde peut également se révéler nécessaire pour les données qui doivent être stockées sur de plus longues périodes à des fins de conformité et d’audit. Dans tous les cas, les sauvegardes sont copiées dans des partages NFS représentés par le biais d’une machine virtuelle et d’un stockage Azure.

- **Fonctionnalité de sauvegarde et de restauration d’infrastructure.** Vous pouvez également utiliser la fonctionnalité de sauvegarde et de restauration fournie par l’infrastructure sous-jacente de SAP HANA sur Azure (grandes instances). Cette option répond à la nécessité d’effectuer des sauvegardes et des restaurations rapides. Le reste de cette section traite des fonctionnalités de sauvegarde et de restauration offertes avec les grandes instances HANA, Cette section couvre également la relation de la fonctionnalité de sauvegarde et de restauration avec la fonctionnalité de récupération d’urgence fournie par les grandes instances HANA.

> [!NOTE]
>   La technologie de capture instantanée utilisée par l’infrastructure sous-jacente des grandes instances HANA a une dépendance sur les captures instantanées SAP HANA. À ce jour, les captures instantanées SAP HANA ne fonctionnent pas avec plusieurs locataires de conteneurs de bases de données SAP HANA mutualisées. Si seul un locataire est déployé, les captures instantanées de SAP HANA fonctionnent, et cette méthode est utilisable.

## <a name="use-storage-snapshots-of-sap-hana-on-azure-large-instances"></a>Utiliser des captures instantanées de stockage de SAP HANA sur Azure (grandes instances)

L’infrastructure de stockage qui sous-tend SAP HANA sur Azure (grandes instances) prend en charge les captures instantanées de volumes de stockage. La prise en charge de la sauvegarde et de la restauration d’un volume est régie par les règles suivantes :

- Au lieu d’exécuter des sauvegardes de base de données complètes, le système procède à de fréquentes captures instantanées des volumes de stockage.
- Lorsqu’une capture instantanée est déclenchée sur les volumes /hana/data et /hana/shared (/usr/sap compris), la technologie de capture instantanée lance une capture instantanée SAP HANA avant d’exécuter la capture instantanée de stockage. Cette capture instantanée SAP HANA est le point d’installation d’éventuelles restaurations de journaux après récupération de la capture instantanée de stockage. Pour qu’une capture instantanée HANA réussisse, vous avez besoin d’une instance HANA active. Dans un scénario HSR, une capture instantanée de stockage n’est pas prise en charge sur un nœud secondaire actif où une capture instantanée HANA n’est pas possible.
- Une fois la capture instantanée de stockage exécutée, la capture instantanée SAP HANA est supprimée.
- Les sauvegardes de fichier journal sont effectuées fréquemment et stockées dans le volume /hana/logbackups ou dans Azure. Vous pouvez déclencher une capture instantanée séparément pour le volume /hana/logbackups contenant les sauvegardes de fichier journal. Dans ce cas, vous n’avez pas besoin d’exécuter une capture instantanée HANA.
- Si vous devez restaurer une base de données à un point dans le temps, contactez le support technique de Microsoft Azure (pour une interruption de production) ou SAP HANA sur Azure pour solliciter une restauration à partir d’une capture instantanée de stockage donnée (par exemple, une restauration planifiée d’un système bac à sable à son état d’origine).
- La capture instantanée SAP HANA incluse dans la capture instantanée de stockage constitue un point de décalage pour l’application des sauvegardes de fichier journal qui ont été exécutées et stockées après la création de la capture instantanée de stockage.
- Ces sauvegardes de fichier journal sont effectuées pour restaurer la base de données à un point spécifique dans le temps.

Vous avez la possibilité d’effectuer des captures instantanées de stockage ciblant trois classes de volumes différentes :

- Une capture instantanée combinée sur les volumes /hana/data et /hana/shared (/usr/sap compris). Cette capture instantanée nécessite la création d’une capture instantanée de SAP HANA en préparation de la capture instantanée de stockage. La capture instantanée de SAP HANA permet de garantir que la base de données est dans un état cohérent du point de vue du stockage. Pour le processus de restauration, c’est un point de configuration.
- Une capture instantanée distincte sur /hana/logbackups.
- Une partition du système d’exploitation.

Pour vous procurer les derniers scripts de capture instantanée et la documentation associée, consultez [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md). Lorsque vous téléchargez le package de script de capture instantanée à partir de [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/release.md), vous obtenez trois fichiers. Un des fichiers est documenté dans un fichier PDF concernant les fonctionnalités fournies. Après avoir téléchargé l’ensemble d’outils, suivez les instructions de la section « Get the snapshot tools » (Obtenir les outils de capture instantanée).

## <a name="storage-snapshot-considerations"></a>Considérations relatives aux captures instantanées de stockage

>[!NOTE]
>Les captures instantanées de stockage consomment un espace de stockage qui est alloué aux unités de grande instance HANA. Prendre en compte les aspects suivants concernant la planification des captures instantanées de stockage et le nombre de captures instantanées de stockage à conserver. 

Les mécanismes spécifiques des captures instantanées de stockage de SAP HANA sur Azure (grandes instances) incluent :

- Une capture instantanée de stockage spécifique (au moment précis de sa création) consomme peu de stockage.
- Étant donné que les données changent et que le contenu des fichiers de données SAP HANA évolue sur le volume de stockage, la capture instantanée doit stocker le contenu des blocs d’origine et les modifications de données.
- La taille de la capture instantanée de stockage augmente en conséquence. Plus la durée d’existence de la capture instantanée est longue, plus la capture instantanée de stockage devient volumineuse.
- Plus le volume de base de données SAP HANA fait l’objet de modifications pendant la durée de vie d’une capture instantanée de stockage, plus la capture instantanée de stockage consomme d’espace.

SAP HANA sur Azure (grandes instances) est fourni avec des tailles de volume fixes pour les volumes de données et de journaux SAP HANA. Effectuer des captures instantanées de ces volumes consomme votre espace de volume. Vous devez :

- Planifier les captures instantanées de stockage.
- Surveiller la consommation d’espace des volumes de stockage. 
- Gérer le nombre de captures instantanées stockées. 

Vous pouvez désactiver les captures instantanées de stockage lorsque vous importez des masses de données ou apportez d’autres modifications importantes à la base de données HANA. 


Les sections ci-après fournissent diverses informations concernant l’exécution de ces captures instantanées et des recommandations générales :

- Bien que le matériel puisse prendre en charge 255 captures instantanées par volume, il est recommandé de rester très en deçà de ce seuil. La recommandation est de 250 maximum.
- Avant d’effectuer des captures instantanées du stockage, surveillez l’espace libre.
- Réduisez le nombre de captures instantanées de stockage en fonction de l’espace libre. Vous pouvez réduire le nombre de captures instantanées à conserver ou étendre les volumes. Vous pouvez commander du stockage supplémentaire par unités d’un téraoctet.
- Lorsque vous exécutez des tâches telles que le déplacement de données dans SAP HANA avec des outils de migration de la plateforme SAP (R3load) ou la restauration des bases de données SAP HANA à partir de sauvegardes, désactivez les captures instantanées de stockage sur le volume /hana/data. 
- Dans le cadre de réorganisations plus vastes des tables SAP HANA, évitez les captures instantanées de stockage dans la mesure du possible.
- Les captures instantanées de stockage constituent une condition préalable pour tirer parti des fonctionnalités de récupération d’urgence de SAP HANA sur Azure (grandes instances).

## <a name="prerequisites-for-using-self-service-storage-snapshots"></a>Conditions préalables pour tirer profit des captures instantanées de stockage en libre service

Pour vérifier que le script de capture instantanée s’exécute correctement, vérifiez que Perl est installé sur le système d’exploitation Linux sur le serveur de grandes instances HANA. Perl est préinstallé sur votre unité de grande instance HANA. Pour vérifier la version de Perl, utilisez la commande ci-dessous :

`perl -v`

![L’exécution de cette commande copie la clé publique.](./media/hana-overview-high-availability-disaster-recovery/perl_screen.png)


## <a name="set-up-storage-snapshots"></a>Configuration des captures instantanées de stockage

Les étapes pour configurer des captures instantanées de stockage avec de grandes instances HANA sont les suivantes.
1. Assurez-vous que Perl est installé sur le système d’exploitation Linux sur le serveur de grandes instances HANA.
1. Modifiez /etc/ssh/ssh\_config en ajoutant la ligne _MACs hmac-sha1_.
1. Créez un compte d’utilisateur de sauvegarde SAP HANA sur le nœud principal de chaque instance SAP HANA que vous exécutez, le cas échéant.
1. Installez le client SAP HANA HDB sur tous les serveurs de grandes instances SAP HANA.
1. Sur le premier serveur de grandes instances SAP HANA de chaque région, créez une clé publique pour accéder à l’infrastructure de stockage sous-jacente qui contrôle la création de captures instantanées.
1. Copiez les scripts et le fichier de configuration de [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/release.md) dans l’emplacement de **hdbsql** dans l’installation de SAP HANA.
1. Modifiez le fichier *HANABackupDetails.txt* en fonction des spécifications appropriées du client.

Procurez-vous les derniers scripts d’instantané et la documentation associée à partir de [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/release.md). Pour connaître les étapes répertoriées précédemment, consultez les [outils de capture instantanée Microsoft pour SAP HANA sur Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md).

### <a name="consideration-for-mcod-scenarios"></a>Considération des scénarios MCOD
Si vous exécutez un [scénario MCOD](https://launchpad.support.sap.com/#/notes/1681092) avec plusieurs instances SAP HANA sur une unité de grande instance HANA, vous obtenez des volumes de stockage distincts pour chacune des instances SAP HANA. Pour plus d’informations sur MDC et d’autres considérations, consultez les points importants à retenir concernant les [outils de capture instantanée Microsoft pour SAP HANA sur Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md).
 

### <a name="step-1-install-the-sap-hana-hdb-client"></a>Étape 1 : Installer le client SAP HANA HDB

Le système d’exploitation Linux installé dans SAP HANA sur Azure (grandes instances) inclut les dossiers et scripts nécessaires pour exécuter des captures instantanées de stockage SAP HANA à des fins de sauvegarde et de récupération d’urgence. Vérifiez s’il existe des versions plus récentes dans [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/release.md). 

Il vous incombe d’installer le client SAP HANA HDB sur les unités de grandes instances HANA quand vous installez SAP HANA.

### <a name="step-2-change-the-etcsshssh_config"></a>Étape 2 : Modifier la configuration /etc/ssh/ssh\_

Cette étape est décrite dans la procédure d’activation de la communication avec le stockage concernant les [outils de capture instantanée Microsoft pour SAP HANA sur Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md).


### <a name="step-3-create-a-public-key"></a>Étape 3 : Créer une clé publique

Pour activer l’accès aux interfaces de capture instantanée de stockage de votre locataire de grandes instances HANA, connectez-vous à l’aide d’une clé publique. 

Sur le premier serveur SAP HANA sur Azure (grandes instances) de votre locataire, créez une clé publique pour accéder à l’infrastructure de stockage. Cette clé publique garantit qu’aucun mot de passe n’est requis pour la connexion aux interfaces de capture instantanée de stockage Vous n’avez pas non plus à mettre à jour les informations d’identification de mot de passe avec une clé publique. 

Pour générer une clé publique, consultez la procédure d’activation de la communication avec le stockage concernant les [outils de capture instantanée Microsoft pour SAP HANA sur Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md).


### <a name="step-4-create-an-sap-hana-user-account"></a>Étape 4 : Créer un compte d'utilisateur SAP HANA

Pour lancer la création de captures instantanées SAP HANA, créez dans SAP HANA un compte d’utilisateur que les scripts de capture instantanée de stockage peuvent utiliser. Créez un compte d’utilisateur SAP HANA dans SAP HANA Studio à cette fin. L’utilisateur doit être créé sous le SYSTEMDB et *non* sous la base de données SID pour MDC. Dans l’environnement de conteneur unique, l’utilisateur est créé dans la base de données locataire. Ce compte doit disposer des privilèges **Backup Admin** (administration des sauvegardes) et **Catalog Read** (lecture du catalogue). 

Pour configurer et utiliser un compte d’utilisateur, consultez la procédure d’activation de la communication avec SAP HANA dans [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md).


### <a name="step-5-authorize-the-sap-hana-user-account"></a>Étape 5 : Autoriser le compte d'utilisateur SAP HANA

Dans cette étape, vous autorisez le compte d’utilisateur SAP HANA que vous avez créé, afin que les scripts n’aient pas besoin d’envoyer des mots de passe au moment de l’exécution. La commande SAP HANA `hdbuserstore` permet la création d’une clé utilisateur SAP HANA. La clé est stockée sur un ou plusieurs nœuds SAP HANA. Cette clé utilisateur permet à l’utilisateur d’accéder à SAP HANA sans avoir à gérer les mots de passe à partir du processus d’utilisation de scripts décrit plus loin dans cet article.

>[!IMPORTANT]
>Exécutez ces commandes de configuration avec le même contexte utilisateur que les commandes de capture instantanée. Sinon, les commandes de capture instantanée ne fonctionnent pas correctement.


### <a name="step-6-get-the-snapshot-scripts-configure-the-snapshots-and-test-the-configuration-and-connectivity"></a>Étape 6 : Obtenir les scripts de capture instantanée, configurer les captures instantanées, et tester la configuration et la connectivité

Téléchargez la version la plus récente des scripts depuis [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/release.md). La méthode d’installation des scripts a changé avec la version 4.1 des scripts. Pour plus d’informations, consultez la procédure d’activation de la communication avec SAP HANA concernant les [outils de capture instantanée Microsoft pour SAP HANA sur Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md).

Pour connaître la séquence exacte de commandes, consultez la procédure d’installation facile des outils de capture instantanée (par défaut) pour les [outils de capture instantanée Microsoft pour SAP HANA sur Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md). Nous recommandons l’utilisation de l’installation par défaut. 

Pour effectuer une mise à niveau à partir de la version 3.x vers 4.1, consultez la procédure de mise à niveau d’une installation existante concernant les [outils de capture instantanée Microsoft pour SAP HANA sur Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md). Pour désinstaller l’ensemble d’outils 4.1, consultez la procédure de désinstallation des [outils de capture instantanée Microsoft pour SAP HANA sur Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md).

N’oubliez pas d’exécuter les étapes permettant de terminer l’installation des dans [outils de capture instantanée Microsoft pour SAP HANA sur Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md).

L’objectif des différents scripts et fichiers une fois installés est décrit dans la section « What are these snapshot tools » (Quels sont ces outils de capture instantanée ?) relative aux [outils de capture instantanée Microsoft pour SAP HANA sur Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md).

Avant de configurer les outils de capture instantanée, vérifiez également que vous avez correctement configuré les paramètres et les emplacements de sauvegarde HANA. Pour plus d’informations, consultez la configuration SAP HANA concernant les [outils de capture instantanée Microsoft pour SAP HANA sur Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md).

La configuration de l’ensemble d’outils de capture instantanée est décrite dans la section « Fichier de configuration - HANABackupCustomerDetails.txt » relative aux [outils de capture instantanée Microsoft pour SAP HANA sur Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md).

#### <a name="test-connectivity-with-sap-hana"></a>Tester la connectivité avec SAP HANA

Après avoir placé toutes les données de configuration dans le fichier *HANABackupCustomerDetails.txt*, vérifiez si les configurations des données d’instance HANA sont correctes. Utilisez le script `testHANAConnection`, qui est indépendant d’une configuration Scale-up ou Scale-out SAP HANA.

Pour plus d’informations, consultez la procédure de vérification de la connectivité avec SAP HANA concernant les [outils de capture instantanée Microsoft pour SAP HANA sur Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md).

#### <a name="test-storage-connectivity"></a>Tester la connectivité du stockage

L’étape de test suivante consiste à vérifier la connectivité au stockage en fonction des données que vous avez placées dans le fichier de configuration *HANABackupCustomerDetails.txt*. Exécutez ensuite une capture instantanée de test. Avant d’exécuter la commande `azure_hana_backup`, vous devez exécuter ce test. Pour connaître la séquence de commandes pour ce test, consultez la procédure de vérification de la connectivité avec le stockage - testStorageSnapshotConnection concernant les [outils de capture instantanée Microsoft pour SAP HANA sur Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md).

Une fois que la connexion aux interfaces de machine virtuelle de stockage est établie, le script passe à la phase 2 et crée une capture instantanée de test, comme indiqué ici pour une configuration de SAP HANA à trois nœuds avec scale-out parallèle.

Si la capture instantanée de test est correctement exécutée avec le script, vous pouvez planifier les captures instantanées de stockage réelles. Si elle a échoué, identifiez le problème avant de poursuivre. La capture instantanée de test doit être conservée jusqu’à ce que les premières captures instantanées réelles soient effectuées.


### <a name="step-7-perform-snapshots"></a>Étape 7 : Effectuer des captures instantanées

Une fois toutes les étapes de préparation effectuées, vous pouvez commencer à configurer et à planifier les captures instantanées du stockage réel. Le script à planifier fonctionne avec les configurations de SAP HANA avec montée en puissance et montée en puissance parallèle. Pour une exécution périodique et régulière de la planification du script de sauvegarde, planifiez le script à l’aide de l’utilitaire cron. 

Pour connaître la syntaxe de commande exacte et les fonctionnalités, consultez la procédure de sauvegarde d’une capture instantanée - azure_hana_backup concernant les [outils de capture instantanée Microsoft pour SAP HANA sur Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md). 

Lorsque le script `azure_hana_backup` est exécuté, il crée la capture instantanée de stockage selon les trois phases suivantes :

1. Il crée une capture instantanée SAP HANA.
1. Il crée une capture instantanée de stockage.
1. Il supprime la capture instantanée SAP HANA créée avant l’exécution de la capture instantanée de stockage.

Pour exécuter le script, appelez-le depuis le dossier de l’exécutable HDB dans lequel il a été copié. 

La période de rétention est administrée avec le nombre de captures instantanées soumis en tant que paramètre lorsque vous exécutez le script. La durée couverte par les captures instantanées de stockage dépend de deux facteurs : la période d’exécution et le nombre de captures instantanées envoyées en tant que paramètre lors de l’exécution du script. 

Si le nombre de captures instantanées conservées dépasse le nombre spécifié en tant que paramètre dans l’appel du script, la plus ancienne capture instantanée de stockage portant la même étiquette est supprimée avant l’exécution d’une nouvelle capture instantanée. Le nombre que vous indiquez en tant que dernier paramètre de l’appel correspond au nombre que vous pouvez utiliser pour contrôler le nombre de captures instantanées qui sont conservées. Ce nombre vous permet également de contrôler indirectement l’espace disque utilisé pour les captures instantanées. 


## <a name="snapshot-strategies"></a>Stratégies de capture instantanée
La fréquence des captures instantanées des différents types varie selon que vous utilisez ou non la fonctionnalité de récupération d’urgence des grandes instances HANA. Cette fonctionnalité repose sur les captures instantanées de stockage, qui peuvent avoir des recommandations spéciales pour la fréquence et les périodes d’exécution des captures instantanées de stockage. 

Dans les considérations et recommandations ci-après, nous supposons que vous n’utilisez *pas* la fonctionnalité de récupération d’urgence proposée par les grandes instances HANA. À la place, vous utilisez les captures instantanées de stockage comme un moyen d’obtenir des sauvegardes et de pouvoir fournir une récupération jusqu’à une date et heure couvrant les 30 derniers jours. Étant donné les limites liées au nombre de captures instantanées et à l’espace, tenez compte des exigences suivantes :

- Temps de récupération pour la récupération jusqu’à une date et heure.
- Espace utilisé.
- Objectifs de point et de délai de récupération pour une récupération d’urgence potentielle après un incident.
- Exécution éventuelles de sauvegardes de base de données complètes HANA pour les disques. Chaque fois qu’une sauvegarde de base de données complète est effectuée pour les disques ou pour l’interface **backint**, l’exécution des captures instantanées de stockage échoue. Si vous prévoyez d’exécuter des sauvegardes de base de données complètes reposant sur des captures instantanées de stockage, assurez-vous que l’exécution des captures instantanées de stockage est désactivée pendant cette période.
- Nombre de captures instantanées par volume, qui est limité à 250.

<!-- backint is term for a SAP HANA interface and not a spelling error not spelling errors -->

Si vous n’utilisez pas la fonctionnalité de récupération d’urgence des grandes instances HANA, la période de capture instantanée est moins fréquente. Dans ces cas, effectuez les captures instantanées combinées sur /hana/data et /hana/shared (/usr/sap compris) toutes les 12 ou 24 heures. Conservez les captures instantanées pendant un mois. Il en va de même pour les captures instantanées du volume de sauvegarde de fichier journal. Les sauvegardes de fichier journal SAP HANA sont exécutées toutes les 5 à 15 minutes en fonction du volume de sauvegarde de fichier journal.

Les captures instantanées de stockage planifiées sont les mieux exécutées avec cron. Utilisez le même script pour la totalité des sauvegardes et des besoins en matière de récupération d’urgence. Modifiez les entrées du script en fonction des différentes heures de sauvegarde demandées. Ces captures instantanées sont toutes planifiées différemment dans cron selon leur durée d’exécution : toutes les heures, toutes les 12 heures, une fois par jour ou une fois par semaine. 

Voici un exemple de planification cron dans /etc/crontab :
```
00 1-23 * * * ./azure_hana_backup --type=hana --prefix=hourlyhana --frequency=15min --retention=46
10 00 * * *  ./azure_hana_backup --type=hana --prefix=dailyhana --frequency=15min --retention=28
00,05,10,15,20,25,30,35,40,45,50,55 * * * * ./azure_hana_backup --type=logs --prefix=regularlogback --frequency=3min --retention=28
22 12 * * *  ./azure_hana_backup --type=logs --prefix=dailylogback --frequncy=3min --retention=28
30 00 * * *  ./azure_hana_backup --type=boot --boottype=TypeI --prefix=dailyboot --frequncy=15min --retention=28
```
Dans l’exemple précédent, une capture instantanée horaire combinée couvre les volumes contenant les emplacements /hana/data et /hana/shared/SID (/usr/sap compris). Utilisez ce type de capture instantanée pour une récupération plus rapide pendant les deux derniers jours. En outre, il existe une capture instantanée quotidienne sur ces volumes. Vous avez ainsi deux jours couverts par les captures instantanées horaires, ainsi que quatre semaines couvertes par les captures instantanées quotidiennes. Le volume de sauvegarde de fichier journal est également sauvegardé une fois par jour. Ces sauvegardes sont conservées pendant quatre semaines. 

Comme le montre la troisième ligne de crontab, la sauvegarde du journal des transactions HANA est planifiée pour s’exécuter toutes les 5 minutes. Les heures de début des différents travaux cron qui exécutent des captures instantanées de stockage sont décalées. De cette façon, les captures instantanées ne s’exécutent pas toutes à la fois à un moment donné. 

Dans l’exemple suivant, vous effectuez une capture instantanée combinée qui couvre les volumes contenant les emplacements /hana/data et /hana/shared/SID (/usr/sap compris) toutes les heures. Vous conservez ces captures instantanées pendant deux jours. Les captures instantanées des volumes de sauvegarde de fichier journal s’exécutent toutes les 5 minutes et sont conservées pendant 4 heures. Comme avant, la sauvegarde du fichier journal HANA est planifiée pour s’exécuter toutes les 5 minutes. 

La capture instantanée du volume de sauvegarde de fichier journal s’effectue dans un délai de 2 minutes après le démarrage de la sauvegarde du fichier journal. Normalement, la sauvegarde du fichier journal SAP HANA doit se terminer dans cet intervalle de 2 minutes. Une fois encore, le volume contenant le numéro d’unité logique de démarrage est sauvegardé une fois par jour par une capture instantanée de stockage et est conservé pendant quatre semaines.

```
10 0-23 * * * ./azure_hana_backup --type=hana ==prefix=hourlyhana --frequency=15min --retention=48
0,5,10,15,20,25,30,35,40,45,50,55 * * * * ./azure_hana_backup --type=logs --prefix=regularlogback --frequency=3min --retention=28
2,7,12,17,22,27,32,37,42,47,52,57 * * * *  ./azure_hana_backup --type=logs --prefix=logback --frequency=3min --retention=48
30 00 * * *  ./azure_hana_backup --type=boot --boottype=TypeII --prefix=dailyboot --frequency=15min --retention=28
```

Le graphique suivant illustre les séquences de l’exemple précédent. Le numéro d’unité logique de démarrage est exclu.

![Relation entre les sauvegardes et les captures instantanées](./media/hana-overview-high-availability-disaster-recovery/backup_snapshot_updated0921.PNG)

SAP HANA effectue des écritures régulières sur le volume /hana/log pour documenter les modifications validées dans la base de données. Régulièrement, SAP HANA écrit un point de sauvegarde dans le volume /hana/data. Comme spécifié dans crontab, une sauvegarde de fichier journal SAP HANA s’exécute toutes les 5 minutes. 

Une capture instantanée SAP HANA est également exécutée toutes les heures suite au déclenchement d’une capture instantanée de stockage combinée sur les volumes /hana/data et /hana/shared/SID. Une fois que la capture instantanée HANA a réussi, la capture instantanée de stockage combinée est exécutée. Comme indiqué dans crontab, la capture instantanée de stockage sur le volume /hana/logbackup s’exécute toutes les 5 minutes, environ 2 minutes après la sauvegarde de fichier journal HANA.

> 

>[!IMPORTANT]
> L’utilisation de captures instantanées de stockage pour les sauvegardes SAP HANA n’est précieuse que quand ces captures sont effectuées conjointement avec des sauvegardes de fichier journal SAP HANA. Ces sauvegardes de fichier journal doivent couvrir les périodes entre les captures instantanées de stockage. 

Si vous vous êtes engagé auprès des utilisateurs à assurer une récupération jusqu’à 30 jours, vous devez disposer des éléments suivants :

- Dans des cas extrêmes, accédez à une capture instantanée de stockage combinée sur les volumes /hana/data et /hana/shared/SID remontant à 30 jours. 
- Vérifiez que les sauvegardes de fichier journal contiguës couvrent la période entre des captures instantanées de stockage combinées. La capture instantanée la plus ancienne du volume de sauvegarde de fichier journal doit donc remonter à 30 jours, ce qui n’est pas le cas si vous copiez les sauvegardes de journal dans un autre partage NFS situé sur le Stockage Azure. Auquel cas, vous pourriez récupérer les anciennes sauvegardes de fichier journal, de ce partage NFS.

Pour tirer parti des captures instantanées de stockage et de la réplication de stockage éventuelle des sauvegardes de fichier journal, modifiez l’emplacement où SAP HANA écrit les sauvegardes de fichier journal. Vous pouvez effectuer ce changement dans HANA Studio. 

Bien que SAP HANA sauvegarde automatiquement des segments de journal complets, indiquez un intervalle de sauvegarde de fichier journal de manière à ce qu’il soit déterministe. Cela est particulièrement vrai si vous utilisez l’option de récupération d’urgence, car on souhaite en général exécuter des sauvegardes de fichier journal selon une période déterministe. Dans le cas suivant, 15 minutes sont définies comme intervalle de sauvegarde du fichier journal.

![Planifier les sauvegardes de journaux d’activité SAP HANA dans SAP HANA Studio](./media/hana-overview-high-availability-disaster-recovery/image5-schedule-backup.png)

Vous pouvez également choisir d’effectuer des sauvegardes plus fréquemment que toutes les 15 minutes. La valeur paramétrée est souvent inférieure avec la fonctionnalité de récupération d’urgence des grandes instances HANA. Certains clients effectuent des sauvegardes de fichier journal toutes les 5 minutes.

Si la base de données n’a jamais été sauvegardée, la dernière étape consiste à effectuer une sauvegarde de base de données basée sur un fichier afin de créer une entrée de sauvegarde unique devant figurer dans le catalogue de sauvegarde. Dans le cas contraire, SAP HANA ne peut pas lancer vos sauvegardes de journaux spécifiées.

![Créer une sauvegarde basée sur un fichier pour créer une entrée de sauvegarde unique](./media/hana-overview-high-availability-disaster-recovery/image6-make-backup.png)


Après avoir exécuté correctement vos premières captures instantanées de stockage, supprimez la capture instantanée de test qui a été exécutée à l’étape 6. Pour plus d’informations, consultez la procédure de suppression des captures instantanées de test - removeTestStorageSnapshot concernant les [outils de capture instantanée Microsoft pour SAP HANA sur Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md). 


### <a name="monitor-the-number-and-size-of-snapshots-on-the-disk-volume"></a>Surveiller le nombre et la taille des captures instantanées sur le volume de disque

Sur un volume de stockage, vous pouvez surveiller le nombre de captures instantanées et l’espace de stockage consommé par ces captures. La commande `ls` n’affiche pas le répertoire des captures instantanées ni les fichiers. La commande du système d’exploitation Linux `du` donne des détails sur ces captures instantanées de stockage, car elles sont stockées sur les mêmes volumes. Utilisez la commande avec les options suivantes :

- `du –sh .snapshot`: cette option fournit le nombre total de captures instantanées dans le répertoire de captures instantanées.
- `du –sh --max-depth=1`: cette option répertorie toutes les captures instantanées enregistrées dans le dossier **.snapshot** et la taille de chaque capture instantanée.
- `du –hc`: cette option fournit la taille totale de toutes les captures instantanées.

Utilisez ces commandes pour vous assurer que les captures instantanées créées et stockées ne consomment pas la totalité du stockage sur les volumes.

>[!NOTE]
>Les captures instantanées du numéro d’unité logique de démarrage ne sont pas visibles avec les commandes précédentes.

### <a name="get-details-of-snapshots"></a>Obtenir les détails des captures instantanées
Pour obtenir plus d’informations sur les captures instantanées, utilisez le script `azure_hana_snapshot_details`. Vous pouvez exécuter ce script dans n’importe quel emplacement à la condition qu’un serveur actif se trouve à l’emplacement de la récupération d’urgence. Le script fournit les informations ventilées suivantes pour chaque volume contenant des captures instantanées : 
   * Taille de toutes les captures instantanées dans un volume
   * Chaque capture instantanée dans ce volume inclut les informations suivantes : 
      - Nom de la capture instantanée 
      - Heure de création 
      - Taille de la capture instantanée
      - Fréquence de la capture instantanée
      - ID de sauvegarde HANA associé à cette capture instantanée, le cas échéant

Pour connaître la syntaxe des commandes et des sorties, découvrez comment obtenir la liste des captures instantanées (azure_hana_snapshot_details) concernant les [outils de capture instantanée Microsoft pour SAP HANA sur Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md). 



### <a name="reduce-the-number-of-snapshots-on-a-server"></a>Réduire le nombre de captures instantanées sur un serveur

Comme expliqué précédemment, vous pouvez réduire le nombre de captures instantanées stockées portant une certaine étiquette. Les deux derniers paramètres de la commande permettant de lancer une capture instantanée correspondent à l’étiquette et au nombre de captures instantanées à conserver.

```
./azure_hana_backup --type=hana --prefix=dailyhana --frequency=15min --retention=28
```

Dans l’exemple précédent, l’étiquette de l’instantané est **dailyhana**. Le nombre de captures instantanées portant cette étiquette à conserver est de **28**. Afin de faire face au problème de la consommation d’espace disque, vous pouvez vouloir réduire le nombre de captures instantanées stockées. Un moyen simple de réduire le nombre de captures instantanées à 15, par exemple, consiste à exécuter le script en définissant le dernier paramètre sur la valeur **15** :

```
./azure_hana_backup --type=hana --prefix=dailyhana --frequency=15min --retention=15
```

Si vous exécutez le script défini avec ce paramètre, le nombre total de captures instantanées, y compris la nouvelle capture instantanée de stockage, est 15. Les 15 captures instantanées les plus récentes sont conservées, tandis que les 15 captures instantanées plus anciennes sont supprimées.

 >[!NOTE]
 > Ce script ne réduit le nombre de captures instantanées que s’il existe des captures instantanées datant de plus d’une heure. Le script ne supprime pas les captures instantanées datant de moins d’une heure. Ces restrictions sont associées à la fonctionnalité de récupération d’urgence disponible en option.

Si vous ne souhaitez plus conserver un ensemble de captures instantanées portant le préfixe de sauvegarde **dailyhana** dans les exemples de syntaxe, exécutez le script avec **0** comme valeur de rétention. Toutes les captures instantanées correspondant à cette étiquette sont alors supprimées. La suppression de toutes les captures instantanées a une incidence sur la fonctionnalité de récupération d’urgence des grandes instances HANA.

L’autre possibilité pour supprimer certaines captures instantanées consiste à utiliser le script `azure_hana_snapshot_delete`. Ce script est conçu pour supprimer une capture instantanée ou un ensemble de captures instantanées à l’aide de l’ID de sauvegarde HANA tel que figurant dans HANA Studio ou du nom de la capture instantanée. À l’heure actuelle, l’ID de sauvegarde est uniquement lié aux captures instantanées créées pour le type de capture instantanée **hana**. Les sauvegardes de capture instantanée de type **logs** ou **boot** n’effectuent pas une capture instantanée SAP HANA, et il n’existe donc aucun ID de sauvegarde pour ces captures instantanées. Si le nom de la capture instantanée est entré, le script recherche sur les différents volumes toutes les captures instantanées qui correspondent au nom entré. 

<!-- hana, logs and boot are no spelling errors as Acrolinx indicates, but terms of parameter values -->

Pour plus d’informations sur le script, consultez la procédure de suppression d’un instantané - azure_hana_snapshot_delete concernant les [outils de capture instantanée Microsoft pour SAP HANA sur Azure](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md).

Exécutez le script en tant qu’utilisateur **root**.

>[!IMPORTANT]
>Si des données n’existent que dans la capture instantanée que vous envisagez de supprimer, la suppression de celle-ci entraîne la perte définitive de ces données.


## <a name="file-level-restore-from-a-storage-snapshot"></a>Restauration au niveau du fichier à partir d’une capture instantanée de stockage

<!-- hana, logs and boot are no spelling errors as Acrolinx indicates, but terms of parameter values -->
Vous pouvez accéder aux captures instantanées **hana** et **logs** directement sur les volumes dans le répertoire **.snapshot**. Il existe un sous-répertoire pour chaque capture instantanée. Copiez chaque fichier dans l’état où il se trouvait au moment de la capture instantanée, depuis ce sous-répertoire vers l’arborescence. 

Dans la version actuelle du script, il n’existe *aucun* script de restauration fourni pour la restauration de capture instantanée en libre-service. La restauration de capture instantanée est possible dans le cadre des scripts de récupération d’urgence en libre-service sur le site de récupération d’urgence pendant le basculement. Pour restaurer la capture instantanée de votre choix à partir des captures instantanées disponibles, vous devez contacter l’équipe des opérations Microsoft en ouvrant une demande de service.

>[!NOTE]
>La restauration d’un fichier unique ne fonctionne pas pour les captures instantanées du numéro d’unité logique de démarrage indépendant du type d’unités de grande instance HANA. Le répertoire **.snapshot** n’est pas exposé dans le numéro d’unité logique de démarrage. 
 

## <a name="recover-to-the-most-recent-hana-snapshot"></a>Récupération jusqu’à la capture instantanée HANA la plus récente

En cas d’interruption de la production, vous pouvez lancer le processus de récupération à partir d’une capture instantanée de stockage en signalant un incident client auprès du support technique de Microsoft Azure. Le problème est de nature urgente si des données sont supprimées d’un système de production et si le seul moyen de les récupérer est de restaurer la base de données de production.

Dans un cas différent, une récupération jusqu’à une date et heure peut ne pas avoir de caractère d’urgence et être planifiée plusieurs jours à l’avance. Vous pouvez planifier cette récupération avec SAP HANA sur Azure au lieu de signaler un problème prioritaire. Par exemple, vous pourriez planifier la mise à niveau du logiciel SAP en appliquant un nouveau package d’améliorations, et vouloir ensuite revenir à une capture instantanée correspondant à l’état du système avant sa mise à niveau avec le package d’améliorations.

Avant de soumettre la demande, vous devez effectuer certaines tâches de préparation. L’équipe SAP HANA sur Azure peut alors traiter la demande et fournir les volumes restaurés. Vous pourrez ensuite restaurer la base de données HANA à l’aide des captures instantanées.

Pour connaître les possibilités d’obtention d’une capture instantanée restaurée avec le nouvel ensemble d’outils, consultez la procédure de restauration d’un instantané dans le [guide de récupération manuelle de SAP HANA sur Azure à partir d’une capture instantanée de stockage](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md).

Pour préparer la demande, procédez comme suit.

1. Déterminez la capture instantanée à restaurer. Sauf indication contraire, seul le volume hana/data est restauré. 

1. Arrêtez l’instance HANA.

   ![Arrêter l’instance HANA](./media/hana-overview-high-availability-disaster-recovery/image7-shutdown-hana.png)

1. Démontez les volumes de données sur chaque nœud de base de données HANA. Si les volumes de données sont toujours montés sur le système d’exploitation, la restauration de la capture instantanée échoue.

   ![Démonter les volumes de données sur chaque nœud de base de données HANA](./media/hana-overview-high-availability-disaster-recovery/image8-unmount-data-volumes.png)

1. Ouvrez une demande de support Azure et incluez des instructions sur la restauration d’une capture instantanée spécifique :

   - Pendant la restauration : le service SAP HANA sur Azure peut vous demander de participer à une téléconférence de coordination, de vérification et de confirmation afin de vérifier que la capture instantanée de stockage est restaurée correctement. 

   - Après la restauration : le service SAP HANA sur Azure vous informe lorsque la capture instantanée de stockage est restaurée.

1. Une fois le processus de restauration terminé, remontez tous les volumes de données.

   ![Remonter tous les volumes de données](./media/hana-overview-high-availability-disaster-recovery/image9-remount-data-volumes.png)



Une autre possibilité pour obtenir des fichiers de données SAP HANA récupérés à partir d’une capture instantanée de stockage, par exemple, est décrite à l’étape 7 du [guide de récupération manuelle de SAP HANA sur Azure à partir d’une capture instantanée de stockage](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md).

Pour effectuer une restauration à partir d’une sauvegarde de captures instantanées, consultez le [guide de récupération manuelle de SAP HANA sur Azure à partir d’une capture instantanée de stockage](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md). 

>[!Note]
>Si votre capture instantanée a été restaurée par des opérations Microsoft, vous n’avez pas besoin de l’étape 7.


### <a name="recover-to-another-point-in-time"></a>Récupération jusqu’à un autre point dans le temps
Pour effectuer une restauration à un certain point dans le temps, consultez la procédure de récupération de base de données pour le point suivant dans le temps, dans le [guide de récupération manuelle de SAP HANA sur Azure à partir d’une capture instantanée de stockage](https://github.com/Azure/hana-large-instances-self-service-scripts/blob/master/latest/Microsoft%20Snapshot%20Tools%20for%20SAP%20HANA%20on%20Azure%20Guide.md). 





## <a name="snapcenter-integration-in-sap-hana-large-instances"></a>Intégration de SnapCenter dans de grandes instances SAP HANA

Cette section décrit comment les clients peuvent utiliser le logiciel NetApp SnapCenter pour prendre un instantané, effectuer une sauvegarde et restaurer des bases de données SAP HANA hébergées sur de grandes instances HANA de Microsoft Azure (HLI). 

SnapCenter propose des solutions pour des scénarios tels que la sauvegarde/récupération, la récupération d’urgence avec réplication de stockage asynchrone, la réplication de système et le clonage de système. Avec l’intégration à SAP HANA sur Azure (grandes instances), les clients peuvent désormais utiliser SnapCenter pour les opérations de sauvegarde et de récupération.

Pour d’autres références, consultez NetApp TR-4614 et TR-4646 sur SnapCenter.

- [Sauvegarde/récupération SAP HANA avec SnapCenter (TR-4614)](https://www.netapp.com/us/media/tr-4614.pdf)
- [Récupération d’urgence SAP HANA avec réplication de stockage (TR-4646)](https://www.netapp.com/us/media/tr-4646.pdf)
- [Réplication système HANA (HSR) SAP HANA avec SnapCenter (TR-4719)](https://www.netapp.com/us/media/tr-4719.pdf)
- [Clonage SAP à partir de SnapCenter (TR-4667)](https://www.netapp.com/us/media/tr-4667.pdf)

### <a name="system-requirements-and-prerequisites"></a>Configuration système requise et prérequis

Voici la configuration système requise pour exécuter SnapCenter sur Azure HLI :
* SnapCenter Server sur Azure Windows 2016 ou version ultérieure avec 4 processeurs virtuels, 16 Go de RAM et un minimum de 650 Go de stockage SSD premium managé.
* Système SAP HANA sur Azure (grandes instances) avec 1,5 To à 24 To de RAM. Il est recommandé d’utiliser deux systèmes SAP HANA Grandes instances pour les tests et les opérations de clonage.

La procédure pour intégrer SnapCenter à SAP HANA est la suivante : 

1. Ouvrez une demande de ticket de support pour communiquer la clé publique générée par l’utilisateur à l’équipe Microsoft Ops. Cela est nécessaire pour configurer l’utilisateur SnapCenter de sorte qu’il puisse accéder au système de stockage.
1. Créez une machine virtuelle dans votre réseau virtuel qui a accès à HLI. Elle est utilisée pour SnapCenter. 
1. Téléchargez et installez SnapCenter. 
1. Opérations de sauvegarde et de récupération 

### <a name="create-a-support-ticket-for-user-role-storage-setup"></a>Créer un ticket de support pour la configuration du stockage du rôle d’utilisateur

1. Ouvrez le portail Azure et accédez à la page **Abonnements**. Une fois sur la page « Abonnements », sélectionnez votre abonnement SAP HANA, encadré en rouge dans l’image ci-dessous.

   :::image type="content" source="./media/snapcenter/create-support-case-for-user-role-storage-setup.png" alt-text="Créer une demande de support pour la configuration du stockage utilisateur":::

1. Sur la page de votre abonnement SAP HANA, sélectionnez la sous-page **Groupes de ressources**.

   :::image type="content" source="./media/snapcenter/solution-lab-subscription-resource-groups.png" alt-text="Groupe de ressources Solution lab subscription" lightbox="./media/snapcenter/solution-lab-subscription-resource-groups.png":::

1. Sélectionnez un groupe de ressources approprié dans une région.

   :::image type="content" source="./media/snapcenter/select-appropriate-resource-group-in-region.png" alt-text="Sélectionner un groupe de ressources approprié dans une région" lightbox="./media/snapcenter/select-appropriate-resource-group-in-region.png":::

1. Sélectionnez une référence SKU correspondant au stockage SAP HANA sur Azure.

   :::image type="content" source="./media/snapcenter/select-sku-entry-corresponding-to-sap-hana.png" alt-text="Sélectionner une référence SKU correspondant à SAP HANA" lightbox="./media/snapcenter/select-sku-entry-corresponding-to-sap-hana.png":::

1. Ouvrez une demande de support avec l’option **Nouveau ticket de support**, encadrée en rouge dans l’image ci-dessous.

   :::image type="content" source="./media/snapcenter/open-new-support-ticket-request.png" alt-text="Ouvrir une nouvelle demande de ticket de support":::

1. Sous l’onglet **De base**, fournissez les informations suivantes pour le ticket :

   * **Type de problème :** Techniques
   * **Abonnement :** Votre abonnement
   * **Service :** Grande instance SAP HANA
   * **Ressource :** Votre groupe de ressources
   * **Résumé :** Indiquez la clé publique générée par l’utilisateur
   * **Type de problème :** Configuration et installation
   * **Sous-type de problème :** Configurer SnapCenter pour HLI


1. Dans la **description** du ticket de support, indiquez les informations suivantes sous l’onglet **Détails** : 
   
   * Configurer SnapCenter pour HLI
   * Votre clé publique pour l’utilisateur SnapCenter (snapcenter.pem) : voir l’exemple de création de clé publique ci-dessous

     :::image type="content" source="./media/snapcenter/new-support-request-details.png" alt-text="Nouvelle demande de support - Onglet Détails" lightbox="./media/snapcenter/new-support-request-details.png":::

1. Sélectionnez **Vérifier + créer** pour passer en revue votre ticket de support. 

1. Générez un certificat pour le nom d’utilisateur SnapCenter sur la grande instance HANA ou n’importe quel serveur Linux.

   SnapCenter nécessite un nom d’utilisateur et un mot de passe pour accéder à la machine virtuelle de stockage et pour créer des instantanés de la base de données HANA. Microsoft utilise la clé publique pour vous (le client) permettre de définir le mot de passe afin d’accéder au système de stockage.

   ```bash
   openssl req -x509 -nodes -days 1095 -newkey rsa:2048 -keyout snapcenter.key -out snapcenter.pem -subj "/C=US/ST=WA/L=BEL/O=NetApp/CN=snapcenter"
   Generating a 2048 bit RSA private key
   ................................................................................................................................................+++++
   ...............................+++++
   writing new private key to 'snapcenter.key'
   -----

   sollabsjct31:~ # ls -l cl25*
   -rw-r--r-- 1 root root 1704 Jul 22 09:59 snapcenter.key
   -rw-r--r-- 1 root root 1253 Jul 22 09:59 snapcenter.pem

   ```

1. Attachez le fichier snapcenter.pem au ticket de support, puis sélectionnez **Créer**

   Une fois le certificat de clé publique envoyé, Microsoft configure le nom d’utilisateur SnapCenter pour votre locataire ainsi que l’adresse IP de la machine virtuelle de stockage.   

1. Après avoir reçu cette adresse IP, définissez un mot de passe pour accéder à la machine virtuelle de stockage, que vous contrôlez.

   Voici un exemple de REST CALL (documentation) d’une grande instance HANA ou d’une machine virtuelle dans un réseau virtuel, qui a accès à l’environnement de grande instance HANA et sera utilisé pour définir le mot de passe.

   ```bash
   curl --cert snapcenter.pem --key snapcenter.key -X POST -k "https://10.0.40.11/api/security/authentication/password" -d '{"name":"snapcenter","password":"test1234"}'
   ```

   Vérifiez qu’il n’existe aucune variable de proxy active sur le système de base de données HANA.

   ```bash
   sollabsjct31:/tmp # unset http_proxy
   sollabsjct31:/tmp # unset https_proxy
   ```

### <a name="download-and-install-snapcenter"></a>Télécharger et installer SnapCenter
Le nom d’utilisateur étant désormais configuré pour l’accès de SnapCenter au système de stockage, vous allez utiliser ce nom d’utilisateur SnapCenter pour configurer SnapCenter une fois installé. 

Avant d’installer SnapCenter, passez en revue [Sauvegarde/récupération SAP HANA avec SnapCenter](https://www.netapp.com/us/media/tr-4614.pdf) pour définir votre stratégie de sauvegarde. 

1. Connectez-vous à [NetApp](https://mysupport.netapp.com) pour [télécharger](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fmysupport.netapp.com%2Fsite%2Fproducts%2Fall%2Fdetails%2Fsnapcenter%2Fdownloads-tab&data=02%7C01%7Cmadhukan%40microsoft.com%7Ca53f5e2f245a4e36933008d816efbb54%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637284566603265503&sdata=TOANWNYoAr1q5z1opu70%2FUDPHjluvovqR9AKplYpcpk%3D&reserved=0) la dernière version de SnapCenter.

1. Installez SnapCenter sur la machine virtuelle Windows Azure.

   Le programme d’installation vérifie les prérequis de la machine virtuelle. 

   >[!IMPORTANT]
   >Soyez attentif à la taille de la machine virtuelle, en particulier dans les grands environnements.

1. Configurez les informations d’identification de l’utilisateur pour SnapCenter. Les informations d’identification Windows utilisées pour installer l’application sont indiquées par défaut. 

   :::image type="content" source="media/snapcenter/installation-user-inputs-dialog.png" alt-text="Boîte de dialogue Entrées utilisateur de l’installation"::: 

1. Lorsque vous démarrez la session, enregistrez l’exemption de sécurité et l’interface utilisateur graphique démarre.

1. Connectez-vous à SnapCenter sur la machine virtuelle (https://snapcenter-vm:8146) ) à l’aide des informations d’identification Windows pour configurer l’environnement.


### <a name="set-up-the-storage-system"></a>Configurer le système de stockage

1. Dans SnapCenter, sélectionnez **Système de stockage**, puis **+Nouveau**. 

   :::image type="content" source="./media/snapcenter/snapcenter-storage-connections-window.png" alt-text="Connexions de stockage SnapCenter" lightbox="./media/snapcenter/snapcenter-storage-connections-window.png":::

   La valeur par défaut est une machine virtuelle de stockage par locataire. Si un client possède plusieurs locataires ou grandes instances HANA dans plusieurs régions, il est recommandé de configurer toutes les machines virtuelles de stockage dans SnapCenter

1. Dans Ajouter un système de stockage, indiquez les informations du système de stockage à ajouter, le nom d’utilisateur et le mot de passe SnapCenter, puis sélectionnez **Envoyer**.

   :::image type="content" source="./media/snapcenter/new-storage-connection.png" alt-text="Nouvelle connexion de stockage":::

   >[!NOTE]
   >La valeur par défaut est une machine virtuelle de stockage par locataire.  S’il existe plusieurs locataires, il est recommandé de configurer toutes les machines virtuelles de stockage dans SnapCenter. 

1. Dans SnapCenter, sélectionnez **Hôtes**, puis le bouton **+Ajouter** pour configurer le plug-in HANA et les hôtes de base de données HANA.  La dernière version de SnapCenter détecte automatiquement la base de données HANA sur l’hôte.

   :::image type="content" source="media/snapcenter/managed-hosts-new-host.png" alt-text="Dans SnapCenter, sélectionnez Hôtes, puis Ajouter." lightbox="media/snapcenter/managed-hosts-new-host.png":::

1. Entrez les informations relatives au nouvel hôte :
   1. Sélectionnez le système d’exploitation du type d’hôte.
   1. Entrez le nom d’hôte de la machine virtuelle SnapCenter.
   1. Entrez les informations d’identification que vous souhaitez utiliser.
   1. Sélectionnez les options **Microsoft Windows** et **SAP HANA**, puis **Envoyer**.

   :::image type="content" source="media/snapcenter/add-new-host-operating-system-credentials.png" alt-text="Informations relatives au nouvel hôte":::

   >[!IMPORTANT]
   >Avant de pouvoir installer le premier nœud, SnapCenter autorise un utilisateur non-racine à installer des plug-ins sur la base de données.  Pour savoir comment autoriser un utilisateur non-racine, consultez [Ajout d’un utilisateur non-racine et configuration des privilèges sudo](https://library.netapp.com/ecmdocs/ECMLP2590889/html/GUID-A3EEB5FC-242B-4C2C-B407-510E48A8F131.html).

1. Passez en revue les informations de l’hôte et sélectionnez **Envoyer** pour installer le plug-in sur le serveur SnapCenter.

1. Une fois le plug-in installé, dans SnapCenter, sélectionnez **Hôtes**, puis **+Ajouter** pour ajouter un nœud HANA.

   :::image type="content" source="media/snapcenter/add-hana-node.png" alt-text="Ajouter un nœud HANA" lightbox="media/snapcenter/add-hana-node.png":::

1. Entrez les informations relatives au nœud HANA :
   1. Sélectionnez le système d’exploitation du type d’hôte.
   1. Entrez le nom d’hôte ou l’adresse IP de la base de données HANA.
   1. Sélectionnez **+** pour ajouter les informations d’identification configurées sur le système d’exploitation hôte de la base de données HANA, puis sélectionnez **OK**.
   1. Sélectionnez **SAP HANA**, puis cliquez sur **Envoyer**.

   :::image type="content" source="media/snapcenter/add-hana-node-details.png" alt-text="Informations sur le nœud SAP HANA":::

1. Confirmez l’empreinte digitale et sélectionnez **Confirmer et envoyer**.

   :::image type="content" source="media/snapcenter/confirm-submit-fingerprint.png" alt-text="Confirmer et envoyer l’empreinte digitale":::

1. Sur le nœud HANA, sous la base de données système, sélectionnez **Sécurité** > **Utilisateurs** > **SNAPCENTER** pour créer l’utilisateur SnapCenter.

   :::image type="content" source="media/snapcenter/create-snapcenter-user-hana-system-db.png" alt-text="Créer l’utilisateur SnapCenter dans HANA (base de données système)":::



### <a name="auto-discovery"></a>Détection automatique
SnapCenter 4.3 active la fonction de découverte automatique par défaut.  La découverte automatique n’est pas prise en charge pour les instances HANA sur lesquelles la réplication système HANA (HSR) est configurée. Vous devez ajouter manuellement l’instance au serveur SnapCenter.


### <a name="hana-setup-manual"></a>Configuration de HANA (manuel)
Si vous avez configuré HSR, vous devez configurer le système manuellement.  

1. Dans SnapCenter, sélectionnez **Ressources** et **SAN HANA** (en haut), puis **+Ajouter une base de données SAP HANA** (à droite).

   :::image type="content" source="media/snapcenter/manual-hana-setup.png" alt-text="Configuration manuelle de HANA" lightbox="media/snapcenter/manual-hana-setup.png":::

1. Spécifiez les informations relatives à la ressource de l’utilisateur administrateur HANA configuré sur l’hôte Linux ou sur l’hôte sur lequel les plug-ins sont installés. La sauvegarde sera gérée depuis le plug-in sur le système Linux.

   :::image type="content" source="media/snapcenter/provide-resource-details-sap-hana-database.png" alt-text="Spécifiez les informations de la ressource de l’utilisateur administrateur HANA configuré sur l’hôte Linux.":::

1. Sélectionnez le volume de données pour lequel vous devez prendre des instantanés, sélectionnez **Enregistrer**, puis **Terminer**.

   :::image type="content" source="media/snapcenter/provide-storage-footprint.png" alt-text="Sélectionnez le volume de données pour lequel vous devez prendre des instantanés, sélectionnez Enregistrer, puis Terminer.":::

### <a name="create-a-snapshot-policy"></a>Créer une stratégie de capture instantanée

Avant d’utiliser SnapCenter pour sauvegarder des ressources de base de données SAP HANA, vous devez créer une stratégie de sauvegarde pour la ressource ou le groupe de ressource que vous voulez sauvegarder. Au cours du processus de création d’une stratégie d’instantané, vous avez la possibilité de configurer des commandes pre/post et des clés SSL spéciales. Pour savoir comment créer une stratégie d’instantané, consultez [Création de stratégies de sauvegarde pour des bases de données SAP HANA](http://docs.netapp.com/ocsc-43/index.jsp?topic=%2Fcom.netapp.doc.ocsc-dpg-sap-hana%2FGUID-246C0810-4F0B-4BF7-9A35-B729AD69954A.html).

1. Dans SnapCenter, sélectionnez **Ressources** et sélectionnez une base de données.

   :::image type="content" source="media/snapcenter/select-database-create-policy.png" alt-text="Dans SnapCenter, sélectionnez Ressources et sélectionnez une base de données.":::

1. Suivez le flux de travail de l’assistant de configuration pour configurer le planificateur d’instantanés.

   :::image type="content" source="media/snapcenter/follow-workflow-configuration-wizard.png" alt-text="Suivez le flux de travail de l’assistant de configuration pour configurer le planificateur d’instantanés." lightbox="media/snapcenter/follow-workflow-configuration-wizard.png":::

1. Choisissez les options pour configurer les commandes pre/post et les clés SSL spéciales.  Dans cet exemple, nous n’utilisons aucun paramètre spécial.

   :::image type="content" source="media/snapcenter/configuration-options-pre-post-commands.png" alt-text="Choisissez les options pour configurer les commandes pre/post et les clés SSL spéciales." lightbox="media/snapcenter/configuration-options-pre-post-commands.png":::

1. Sélectionnez **Ajouter** pour créer une stratégie d’instantané, qui peut également être utilisée pour d’autres bases de données HANA. 

   :::image type="content" source="media/snapcenter/select-one-or-more-policies.png" alt-text="Sélectionnez Ajouter pour créer une stratégie d’instantané, qui peut également être utilisée pour d’autres bases de données HANA.":::

1. Entrez un nom et une description pour la stratégie.

   :::image type="content" source="media/snapcenter/new-sap-hana-backup-policy.png" alt-text="Entrez un nom et une description pour la stratégie.":::


1. Sélectionnez le type et la fréquence de la sauvegarde.

   :::image type="content" source="media/snapcenter/new-sap-hana-backup-policy-settings.png" alt-text="Sélectionnez le type et la fréquence de la sauvegarde.":::

1. Configurez les **paramètres de rétention de sauvegarde à la demande**.  Dans notre exemple, nous définissons la rétention sur trois copies d’instantanés à conserver.

   :::image type="content" source="media/snapcenter/new-sap-hana-backup-policy-retention-settings.png" alt-text="Configurez les paramètres de rétention de sauvegarde à la demande.":::

1. Configurez les **paramètres de rétention horaire**. 

   :::image type="content" source="media/snapcenter/new-sap-hana-backup-policy-hourly-retention-settings.png" alt-text="Configurez les paramètres de rétention horaire.":::

1. Si une configuration SnapMirror est configurée, sélectionnez **Mettre à jour SnapMirror après avoir créé une copie locale de SnapShot**.

   :::image type="content" source="media/snapcenter/new-sap-hana-backup-policy-snapmirror.png" alt-text="Si SnapMirror est requis, sélectionnez Mettre à jour SnapMirror après avoir créé une copie locale de SnapShot.":::

1. Sélectionnez **Terminer** pour passer en revue le récapitulatif de la nouvelle stratégie de sauvegarde. 
1. Sous **Configurer une planification**, sélectionnez **Ajouter**.

   :::image type="content" source="media/snapcenter/configure-schedules-for-selected-policies.png" alt-text="Sous Configurer une planification, sélectionnez Ajouter.":::

1. Sélectionnez les paramètres **Date de début**, **Date d’expiration** et la fréquence.

   :::image type="content" source="media/snapcenter/add-schedules-for-policy.png" alt-text="Sélectionnez les paramètres Date de début, Date d’expiration et la fréquence.":::

1. Indiquez les informations relatives aux notifications par e-mail.

   :::image type="content" source="media/snapcenter/backup-policy-notification-settings.png" alt-text="Indiquez les informations relatives aux notifications par e-mail.":::

1.  Sélectionnez **Terminer** pour créer la stratégie de sauvegarde.

### <a name="disable-ems-message-to-netapp-autosupport"></a>Désactiver le message EMS à NetApp Autosupport
Par défaut, la collecte de données EMS est désactivée et s’exécute tous les sept jours après la date d’installation.  Vous pouvez désactiver la collecte de données avec la cmdlet PowerShell `Disable-SmDataCollectionEms`.

1. Dans PowerShell, établissez une session avec SnapCenter.

   ```powershell
   Open-SmConnection
   ```

1. Connectez-vous avec vos informations d’identification.
1. Désactivez la collecte de messages EMS.

   ```powershell
   Disable-SmCollectionEms
   ```

### <a name="restore-database-after-crash"></a>Restaurer la base de données après incident
Vous pouvez utiliser SnapCenter pour restaurer la base de données.  Dans cette section, nous allons aborder les étapes principales. Pour plus d’informations, vous pouvez consulter l’article [Sauvegarde/récupération SAP HANA avec SnapCenter](https://www.netapp.com/us/media/tr-4614.pdf).


1. Arrêtez la base de données et supprimez tous les fichiers de base de données.

   ```
   su - h31adm
   > sapcontrol -nr 00 -function StopSystem
   StopSystem
   OK
   > sapcontrol -nr 00 -function GetProcessList
   OK
   name, description, dispstatus, textstatus, starttime, elapsedtime, pid
   hdbdaemon, HDB Daemon, GRAY, Stopped, , , 35902
 
   ```

1. Démontez le volume de la base de données.

   ```bash
   unmount /hana/data/H31/mnt00001
   ```


1. Restaurez les fichiers de base de données via SnapCenter.  Sélectionnez la base de données, puis cliquez sur **Restaurer**.  

   :::image type="content" source="media/snapcenter/restore-database-via-snapcenter.png" alt-text="Sélectionnez la base de données, puis cliquez sur Restaurer." lightbox="media/snapcenter/restore-database-via-snapcenter.png":::

1. Sélectionnez le type de restauration.  Dans notre exemple, nous allons restaurer la ressource complète. 

   :::image type="content" source="media/snapcenter/restore-database-select-restore-type.png" alt-text="Sélectionnez le type de restauration.":::

   >[!NOTE]
   >Avec une configuration par défaut, vous n’avez pas besoin de spécifier de commandes pour effectuer une restauration locale à partir de l’instantané sur disque. 

   >[!TIP]
   >Si vous voulez restaurer un LUN particulier à l’intérieur du volume, sélectionnez **Niveau de fichier**.

1. Suivez le flux de travail dans l’assistant de configuration.
   
   SnapCenter restaure les données dans l’emplacement d’origine pour que vous puissiez démarrer le processus de restauration dans HANA. De plus, SnapCenter n’étant pas capable de modifier le catalogue de sauvegarde (la base de données est inactive), un avertissement s’affiche.

   :::image type="content" source="media/snapcenter/restore-database-job-details-warning.png" alt-text="SnapCenter n’étant pas capable de modifier le catalogue de sauvegarde, un avertissement s’affiche.":::

1. Tous les fichiers de base de données étant restaurés, démarrez le processus de restauration dans HANA. Dans HANA Studio, sous **Systèmes**, cliquez avec le bouton droit sur la base de données système et sélectionnez **Sauvegarde et récupération** > **Récupérer la base de données système**.

   :::image type="content" source="media/snapcenter/hana-studio-backup-recovery.png" alt-text="Démarrez le processus de restauration dans HANA.":::

1. Sélectionnez un type de récupération.

   :::image type="content" source="media/snapcenter/restore-database-select-recovery-type.png" alt-text="Sélectionnez le type de récupération.":::

1. Sélectionnez l’emplacement du catalogue de sauvegarde.

   :::image type="content" source="media/snapcenter/restore-database-select-location-backup-catalog.png" alt-text="Sélectionnez l’emplacement du catalogue de sauvegarde.":::

1. Sélectionnez une sauvegarde pour récupérer la base de données SAP HANA.

   :::image type="content" source="media/snapcenter/restore-database-select-backup.png" alt-text="Sélectionnez une sauvegarde pour récupérer la base de données SAP HANA.":::

   Une fois la base de données récupérée, un message apparaît avec un tampon **Heure de récupération** et **Emplacement du journal de récupération**.

1. Sous **Systèmes**, cliquez avec le bouton droit sur la base de données système et sélectionnez **Sauvegarde et récupération** > **Récupérer la base de données du locataire**.
1. Suivez le flux de travail de l’assistant pour achever la récupération de la base de données du locataire. 

Pour en savoir plus sur la restauration d’une base de données, consultez [Sauvegarde/récupération SAP HANA avec SnapCenter](https://www.netapp.com/us/media/tr-4614.pdf).


### <a name="non-database-backups"></a>Sauvegardes hors base de données
Vous pouvez restaurer des volumes autres que des données, par exemple un partage de fichiers réseau (/hana/shared) ou une sauvegarde de système d’exploitation.  Pour en savoir plus sur la restauration d’un volume autre que des données, consultez [Sauvegarde/récupération SAP HANA avec SnapCenter](https://www.netapp.com/us/media/tr-4614.pdf).

### <a name="sap-hana-system-cloning"></a>Clonage de système SAP HANA

Pour pouvoir cloner, la version de HANA installée doit être la même que celle de la base de données source. Les SID et ID peuvent être différents. 

:::image type="content" source="media/snapcenter/system-cloning-diagram.png" alt-text="Clonage de système SAP HANA" lightbox="media/snapcenter/system-cloning-diagram.png" border="false":::

1. Créez un magasin d’utilisateurs de base de données HANA pour la base de données H34 à partir de /usr/sap/H34/HDB40.

   ```
   hdbuserstore set H34KEY sollabsjct34:34013 system manager
   ```
 
1. Désactivez le pare-feu.

   ```bash
   systemctl disable SuSEfirewall2
   systemctl stop  SuSEfirewall2
   ```

1. Installez le Kit de développement logiciel (SDK) Java.

   ```bash
   zypper in java-1_8_0-openjdk
   ```

1. Dans SnapCenter, ajoutez l’hôte de destination sur lequel le clone sera monté. Pour en savoir plus, consultez [Ajouter des hôtes et installer des packages de plug-ins sur les hôtes distants](http://docs.netapp.com/ocsc-43/index.jsp?topic=%2Fcom.netapp.doc.ocsc-dpg-sap-hana%2FGUID-246C0810-4F0B-4BF7-9A35-B729AD69954A.html).
   1. Renseignez les informations de connexion « Exécuter en tant que » à ajouter. 
   1. Sélectionnez le système d’exploitation hôte et entrez les informations relatives à l’hôte.
   1. Sous **Plug-ins à installer**, sélectionnez la version, entrez le chemin d’installation, puis sélectionnez **SAP HANA**.
   1. Sélectionnez **Valider** pour exécuter les vérifications de préinstallation.

1. Arrêtez HANA et démontez l’ancien volume de données.  Vous allez monter le clone à partir de SnapCenter.  

   ```bash
   sapcontrol -nr 40 -function StopSystem
   umount /hana/data/H34/mnt00001

   ```
 1. Créez les fichiers de configuration et de script de l’interpréteur de commandes pour la cible.
 
    ```bash
    mkdir /NetApp
    chmod 777 /NetApp
    cd NetApp
    chmod 777 sc-system-refresh-H34.cfg
    chmod 777 sc-system-refresh.sh

    ```

    >[!TIP]
    >Vous pouvez copier les scripts du document [Clonage SAP à partir de SnapCenter](https://www.netapp.com/us/media/tr-4667.pdf).

1. Modifiez le fichier de configuration. 

   ```bash
   vi sc-system-refresh-H34.cfg
   ```

   * HANA_ARCHITECTURE="MDC_single_tenant"
   * KEY="H34KEY"
   * TIME_OUT_START=18
   * TIME_OUT_STOP=18
   * INSTANCENO="40"
   * STORAGE="10.250.101.33"

1. Modifiez le fichier de script de l’interpréteur de commandes.

   ```bash
   vi sc-system-refresh.sh
   ```  

   * VERBOSE=NO
   * MY_NAME="`basename $0`"
   * BASE_SCRIPT_DIR="`dirname $0`"
   * MOUNT_OPTIONS="rw,vers=4,hard,timeo=600,rsize=1048576,wsize=1048576,intr,noatime,nolock"

1. Démarrez un clone à partir d’un processus de sauvegarde. Sélectionnez l’hôte pour créer le clone. 

   >[!NOTE]
   >Pour plus d’informations, consultez [Cloner à partir d’une sauvegarde](https://docs.netapp.com/ocsc-43/index.jsp?topic=%2Fcom.netapp.doc.ocsc-dpg-cpi%2FGUID-F6E7FF73-0183-4B9F-8156-8D7DA17A8555.html).

1. Sous **Scripts**, indiquez les éléments suivants :

   * **Montez la commande :** /NetApp/sc-system-refresh.sh mount H34 %hana_data_h31_mnt00001_t250_vol_Clone
   * **Publiez la commande de clonage :** /NetApp/sc-system-refresh.sh recover H34

1. Désactivez (verrouillez) le montage automatique dans /etc/fstab, le volume de données de la base de données préinstallée n’étant pas nécessaire. 

   ```bash
   vi /etc/fstab
   ```

### <a name="delete-a-clone"></a>Supprimer un clone

Vous pouvez supprimer un clone s’il n’est plus utile. Pour plus d’informations, consultez [Suppression de clones](https://docs.netapp.com/ocsc-43/index.jsp?topic=%2Fcom.netapp.doc.ocsc-dpg-cpi%2FGUID-F6E7FF73-0183-4B9F-8156-8D7DA17A8555.html).

Les commandes à exécuter avant la suppression du clone sont les suivantes :
* **Suppression avant clonage :** /NetApp/sc-system-refresh.sh shut down H34
* **Démontage :** /NetApp/sc-system-refresh.sh umount H34

Ces commandes autorisent SnapCenter à confronter la base de données, démonter le volume et supprimer l’entrée fstab.  Après cela, l’élément FlexClone est supprimé. 

### <a name="cloning-database-logfile"></a>Clonage du fichier journal de la base de données

```   
20190502025323###sollabsjct34###sc-system-refresh.sh: Adding entry in /etc/fstab.
20190502025323###sollabsjct34###sc-system-refresh.sh: 10.250.101.31:/Sc21186309-ee57-41a3-8584-8210297f791d /hana/data/H34/mnt00001 nfs rw,vers=4,hard,timeo=600,rsize=1048576,wsize=1048576,intr,noatime,lock 0 0
20190502025323###sollabsjct34###sc-system-refresh.sh: Mounting data volume.
20190502025323###sollabsjct34###sc-system-refresh.sh: mount /hana/data/H34/mnt00001
20190502025323###sollabsjct34###sc-system-refresh.sh: Data volume mounted successfully.
20190502025323###sollabsjct34###sc-system-refresh.sh: chown -R h34adm:sapsys /hana/data/H34/mnt00001
20190502025333###sollabsjct34###sc-system-refresh.sh: Recover system database.
20190502025333###sollabsjct34###sc-system-refresh.sh: /usr/sap/H34/HDB40/exe/Python/bin/python /usr/sap/H34/HDB40/exe/python_support/recoverSys.py --command "RECOVER DATA USING SNAPSHOT CLEAR LOG"
[140278542735104, 0.005] >> starting recoverSys (at Thu May  2 02:53:33 2019)
[140278542735104, 0.005] args: ()
[140278542735104, 0.005] keys: {'command': 'RECOVER DATA USING SNAPSHOT CLEAR LOG'}
recoverSys started: ============2019-05-02 02:53:33 ============
testing master: sollabsjct34
sollabsjct34 is master
shutdown database, timeout is 120
stop system
stop system: sollabsjct34
stopping system: 2019-05-02 02:53:33
stopped system: 2019-05-02 02:53:33
creating file recoverInstance.sql
restart database
restart master nameserver: 2019-05-02 02:53:38
start system: sollabsjct34
2019-05-02T02:53:59-07:00  P010976      16a77f6c8a2 INFO    RECOVERY state of service: nameserver, sollabsjct34:34001, volume: 1, RecoveryPrepared
recoverSys finished successfully: 2019-05-02 02:54:00
[140278542735104, 26.490] 0
[140278542735104, 26.490] << ending recoverSys, rc = 0 (RC_TEST_OK), after 26.485 secs
20190502025400###sollabsjct34###sc-system-refresh.sh: Wait until SAP HANA database is started ....
20190502025400###sollabsjct34###sc-system-refresh.sh: Status:  YELLOW
20190502025410###sollabsjct34###sc-system-refresh.sh: Status:  YELLOW
20190502025420###sollabsjct34###sc-system-refresh.sh: Status:  YELLOW
20190502025430###sollabsjct34###sc-system-refresh.sh: Status:  YELLOW
20190502025440###sollabsjct34###sc-system-refresh.sh: Status:  YELLOW
20190502025451###sollabsjct34###sc-system-refresh.sh: Status:  GREEN
20190502025451###sollabsjct34###sc-system-refresh.sh: SAP HANA database is started.
20190502025451###sollabsjct34###sc-system-refresh.sh: Recover tenant database H34.
20190502025451###sollabsjct34###sc-system-refresh.sh: /usr/sap/H34/SYS/exe/hdb/hdbsql -U H34KEY RECOVER DATA FOR H34 USING SNAPSHOT CLEAR LOG
0 rows affected (overall time 69.584135 sec; server time 69.582835 sec)
20190502025600###sollabsjct34###sc-system-refresh.sh: Checking availability of Indexserver for tenant H34.
20190502025601###sollabsjct34###sc-system-refresh.sh: Recovery of tenant database H34 succesfully finished.
20190502025601###sollabsjct34###sc-system-refresh.sh: Status: GREEN
Deleting the DB Clone – Logfile
20190502030312###sollabsjct34###sc-system-refresh.sh: Stopping HANA database.
20190502030312###sollabsjct34###sc-system-refresh.sh: sapcontrol -nr 40 -function StopSystem HDB

02.05.2019 03:03:12
StopSystem
OK
20190502030312###sollabsjct34###sc-system-refresh.sh: Wait until SAP HANA database is stopped ....
20190502030312###sollabsjct34###sc-system-refresh.sh: Status:  GREEN
20190502030322###sollabsjct34###sc-system-refresh.sh: Status:  GREEN
20190502030332###sollabsjct34###sc-system-refresh.sh: Status:  GREEN
20190502030342###sollabsjct34###sc-system-refresh.sh: Status:  GRAY
20190502030342###sollabsjct34###sc-system-refresh.sh: SAP HANA database is stopped.
20190502030347###sollabsjct34###sc-system-refresh.sh: Unmounting data volume.
20190502030347###sollabsjct34###sc-system-refresh.sh: Junction path: Sc21186309-ee57-41a3-8584-8210297f791d
20190502030347###sollabsjct34###sc-system-refresh.sh: umount /hana/data/H34/mnt00001
20190502030347###sollabsjct34###sc-system-refresh.sh: Deleting /etc/fstab entry.
20190502030347###sollabsjct34###sc-system-refresh.sh: Data volume unmounted successfully.

```

### <a name="uninstall-snapcenter-plug-ins-package-for-linux"></a>Désinstaller le package de plug-ins SnapCenter pour Linux

Vous pouvez désinstaller le package de plug-ins Linux de la ligne de commandes. Étant donné que le déploiement automatique attend un nouveau système, il est facile de désinstaller le plug-in.  

>[!NOTE]
>Il est possible que vous deviez désinstaller manuellement une version plus ancienne du plug-in. 

Désinstallez les plug-ins.

```bash
cd /opt/NetApp/snapcenter/spl/installation/plugins
./uninstall
```

Vous pouvez désormais installer le dernier plug-in HANA sur le nouveau nœud en sélectionnant **ENVOYER** dans SnapCenter. 




## <a name="next-steps"></a>Étapes suivantes
- Voir [Disaster Recovery principles and preparation](hana-concept-preparation.md) (Principes de récupération d’urgence et préparation).
