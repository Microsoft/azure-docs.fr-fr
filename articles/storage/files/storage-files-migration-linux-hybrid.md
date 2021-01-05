---
title: Migration de Linux vers Azure File Sync
description: Découvrez comment migrer des fichiers d’un serveur Linux vers un déploiement de cloud hybride à l’aide d’Azure File Sync et de partages de fichiers Azure.
author: fauhse
ms.service: storage
ms.topic: how-to
ms.date: 03/19/2020
ms.author: fauhse
ms.subservice: files
ms.openlocfilehash: 46bcfd48d8fdfb228670b87df166c1ad8de61e52
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94626348"
---
# <a name="migrate-from-linux-to-a-hybrid-cloud-deployment-with-azure-file-sync"></a>Migrer de Linux vers un déploiement de cloud hybride avec Azure File Sync

Azure File Sync fonctionne sur des instances Windows Servers avec un stockage en attachement direct (DAS). Il ne prend en charge ni la synchronisation vers et depuis des clients Linux, ni un partage SMB (Server Message Block) distant, ni des partages NFS (Network File System).

Par conséquent, la transformation de vos services de fichiers en un déploiement hybride rend nécessaire une migration vers un Windows Server. Cet article vous guide tout au long de la planification et de l’exécution d’une telle migration.

## <a name="migration-goals"></a>Objectifs de la migration

L’objectif est de déplacer les partages que vous avez sur votre serveur Samba Linux vers une instance Windows Server. Utilisez ensuite Azure File Sync pour un déploiement de cloud hybride. Cette migration doit être effectuée de manière à garantir l’intégrité des données de production, ainsi que la disponibilité pendant la migration. Garantir la disponibilité implique de garder les temps d’arrêt à un niveau minimal pour qu’ils respectent les fenêtres de maintenance habituelles ou ne les dépassent que légèrement.

## <a name="migration-overview"></a>Vue d’ensemble de la migration

Comme mentionné dans l’[article de présentation de la migration](storage-files-migration-overview.md) Azure Files, il est important d’utiliser l’outil de copie et l’approche appropriés. Votre serveur Samba Linux expose les partages SMB directement sur votre réseau local. Robocopy, intégré à Windows Server, est la meilleure façon de déplacer vos fichiers dans ce scénario de migration.

Si vous n’exécutez pas Samba sur votre serveur Linux et que vous souhaitez plutôt migrer des dossiers vers un déploiement hybride sur Windows Server, vous pouvez utiliser les outils de copie de Linux au lieu de Robocopy. Si vous le faites, tenez compte des capacités de fidélité dans votre outil de copie de fichiers. Consultez la [section sur les bases de la migration](storage-files-migration-overview.md#migration-basics) dans l’article de présentation de la migration pour savoir ce qu’il faut rechercher dans un outil de copie.

## <a name="phase-1-identify-how-many-azure-file-shares-you-need"></a>Phase 1 : Identifier le nombre de partages de fichiers Azure dont vous avez besoin

[!INCLUDE [storage-files-migration-namespace-mapping](../../../includes/storage-files-migration-namespace-mapping.md)]

## <a name="phase-2-provision-a-suitable-windows-server-instance-on-premises"></a>Phase 2 : Approvisionner une instance Windows Server locale appropriée

* Créez une instance Windows Server 2019 comme machine virtuelle ou comme serveur physique. Windows Server 2012 R2 est la configuration minimale requise. Un cluster de basculement Windows Server est également pris en charge.
* Approvisionnez ou ajoutez un stockage en attachement direct (DAS). Le stockage attaché au réseau (NAS) n’est pas pris en charge.

  Si vous utilisez la fonctionnalité de [hiérarchisation cloud](storage-sync-cloud-tiering.md) d’Azure File Sync, la quantité de stockage que vous approvisionnez peut être inférieure à celle que vous utilisez actuellement sur votre serveur Samba Linux. Toutefois, lorsque vous copiez vos fichiers de l’espace plus grand du serveur Samba Linux vers le volume plus petit de Windows Server à une étape ultérieure, vous devrez travailler par lots :

  1. Déplacez un ensemble de fichiers qui tiennent sur le disque.
  2. Autorisez la synchronisation des fichiers et la hiérarchisation cloud.
  3. Lorsque de l’espace libre est créé sur le volume, passez au lot de fichiers suivant. 
    
  Vous pouvez éviter cette approche de traitement par lot en approvisionnant sur l’instance Windows Server un espace équivalent à celui que vos fichiers occupent sur le serveur Samba Linux. Envisagez d’activer la déduplication sur Windows. Si vous ne souhaitez pas valider définitivement cette grande quantité de stockage sur votre instance Windows Server, vous pouvez réduire la taille du volume après la migration et avant d’ajuster les stratégies de hiérarchisation cloud. Cela crée un cache local plus petit de vos partages de fichiers Azure.

La configuration des ressources (calcul et RAM) de l’instance Windows Server que vous déployez dépend principalement du nombre d’éléments (fichiers et dossiers) que vous allez synchroniser. Nous vous recommandons d’utiliser une configuration haute performance si vous rencontrez des problèmes.

[Découvrez comment dimensionner une instance Windows Server en fonction du nombre d’éléments (fichiers et dossiers) que vous devez synchroniser.](storage-sync-files-planning.md#recommended-system-resources)

> [!NOTE]
> L’article précédemment lié présente un tableau avec une plage pour la mémoire du serveur (RAM). Vous pouvez vous orienter vers le plus petit nombre de votre serveur, mais attendez-vous à ce que la synchronisation initiale prenne beaucoup plus de temps.

## <a name="phase-3-deploy-the-azure-file-sync-cloud-resource"></a>Phase 3 : Déployer la ressource cloud Azure File Sync

[!INCLUDE [storage-files-migration-deploy-afs-sss](../../../includes/storage-files-migration-deploy-azure-file-sync-storage-sync-service.md)]

## <a name="phase-4-deploy-azure-storage-resources"></a>Phase 4 : Déployer des ressources de stockage Azure

Au cours de cette phase, consultez la table de mappage de la phase 1 et utilisez-la pour approvisionner le nombre correct de comptes de stockage Azure et de partages de fichiers dans ceux-ci.

[!INCLUDE [storage-files-migration-provision-azfs](../../../includes/storage-files-migration-provision-azure-file-share.md)]

## <a name="phase-5-deploy-the-azure-file-sync-agent"></a>Phase 5 : Déployer l’agent Azure File Sync

[!INCLUDE [storage-files-migration-deploy-afs-agent](../../../includes/storage-files-migration-deploy-azure-file-sync-agent.md)]

## <a name="phase-6-configure-azure-file-sync-on-the-windows-server-deployment"></a>Phase 6 : Configurer Azure File Sync sur le déploiement Windows Server

Votre instance Windows Server locale inscrite doit être prête et connectée à Internet pour ce processus.

[!INCLUDE [storage-files-migration-configure-sync](../../../includes/storage-files-migration-configure-sync.md)]

> [!IMPORTANT]
> La hiérarchisation cloud désigne la fonctionnalité Azure File Sync qui permet au serveur local d’avoir moins de capacité de stockage que ce qui est stocké dans le cloud, tout en ayant l’espace de noms complet disponible. Les données intéressantes localement sont également mises en cache localement pour des performances d’accès rapides. La hiérarchisation cloud est une fonctionnalité facultative pour chaque point de terminaison de serveur Azure File Sync.

> [!WARNING]
> Si vous avez configuré moins de stockage sur vos volumes Windows Server que ce que vos données utilisaient sur le serveur Samba Linux, la hiérarchisation cloud est obligatoire. Si vous n’activez pas la hiérarchisation cloud, votre serveur ne libérera pas d’espace pour stocker tous les fichiers. Définissez temporairement votre stratégie temporaire de hiérarchisation sur 99 pour cent pour la migration. Veillez à revenir à vos paramètres de hiérarchisation cloud une fois la migration terminée, et définissez la stratégie à un niveau plus utile à long terme.

Répétez les étapes de création d’un groupe de synchronisation et d’ajout du dossier serveur correspondant comme point de terminaison de serveur pour tous les partages de fichiers et les emplacements de serveur Azure à configurer pour la synchronisation.

Après la création de tous les points de terminaison de serveur, la synchronisation fonctionne. Vous pouvez créer un fichier de test et le voir se synchroniser de votre serveur au partage de fichiers Azure connecté (comme décrit par le point de terminaison cloud dans le groupe de synchronisation).

Autrement, les deux emplacements, les dossiers du serveur et les partages de fichiers Azure sont vides et attendent des données. À l’étape suivante, vous commencerez à copier des fichiers dans l’instance Windows Server pour Azure File Sync afin de les déplacer dans le cloud. Lorsque vous avez activé la hiérarchisation cloud, le serveur commence à hiérarchiser les fichiers si vous ne disposez pas de suffisamment de capacité sur les volumes locaux.

## <a name="phase-7-robocopy"></a>Phase 7 : Robocopy

L’approche de la migration de base consiste à utiliser Robocopy pour copier des fichiers et Azure File Sync pour effectuer la synchronisation.

Exécutez la première copie locale vers votre dossier Windows Server cible :

1. Identifiez le premier emplacement sur votre serveur Samba Linux.
1. Identifiez le dossier correspondant sur l’instance Windows Server où Azure File Sync est déjà configuré.
1. Démarrez la copie à l’aide de Robocopy.

La commande Robocopy suivante copie les fichiers de votre stockage de votre serveur Samba Linux vers votre dossier cible Windows Server. Le serveur Windows Server va le synchroniser avec les partages de fichiers Azure. 

Si vous avez configuré moins de stockage sur votre instance Windows Server que ce que vos fichiers occupent sur le serveur Samba Linux, cela signifie que vous avez configuré la hiérarchisation cloud. Quand le volume Windows Server local se remplit, la [hiérarchisation cloud](storage-sync-cloud-tiering.md) commence et hiérarchise les fichiers qui ont déjà été correctement synchronisés. La hiérarchisation cloud génère suffisamment d’espace pour poursuivre la copie à partir du serveur Samba Linux. La hiérarchisation cloud effectue une vérification toutes les heures pour déterminer ce qui a été synchronisé et libérer de l’espace disque pour atteindre la stratégie d’un espace de volume libre de 99 %.

Il est possible que Robocopy déplace les fichiers plus rapidement que vous ne pouvez les synchroniser dans le cloud et les hiérarchiser localement, ce qui vous amène à manquer d’espace disque local. Dans ce cas, Robocopy échoue. Nous vous recommandons de traiter les partages dans une séquence qui empêche le problème de survenir. Par exemple, envisagez de ne pas démarrer les travaux Robocopy pour tous les partages en même temps. Vous pouvez également envisager de déplacer les partages pour lesquels l’espace libre est actuellement suffisant sur l’instance Windows Server. Si votre travail Robocopy échoue, vous pouvez toujours réexécuter la commande tant que vous utilisez l’option de mise en miroir/vidage suivante :

```console
Robocopy /MT:32 /UNILOG:<file name> /TEE /B /MIR /COPYALL /DCOPY:DAT <SourcePath> <Dest.Path>
```

Arrière-plan :

:::row:::
   :::column span="1":::
      /MT
   :::column-end:::
   :::column span="1":::
      Permet à Robocopy de s’exécuter en multithread. La valeur par défaut est 8, la valeur maximale est 128.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /UNILOG:\<file name\>
   :::column-end:::
   :::column span="1":::
      Renvoie l’état au fichier journal au format Unicode (remplace le journal existant).
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /TEE
   :::column-end:::
   :::column span="1":::
      Génère les sorties dans une fenêtre de console. Utilisé conjointement avec la sortie dans un fichier journal.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /B
   :::column-end:::
   :::column span="1":::
      Exécute Robocopy dans le même mode qu’une application de sauvegarde. Permet à Robocopy de déplacer des fichiers pour lesquels l’utilisateur actuel n’a pas d’autorisations.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /MIR
   :::column-end:::
   :::column span="1":::
      Permet l’exécution de cette commande Robocopy plusieurs fois, de manière séquentielle, sur la même cible/destination. La commande identifie et omet ce qui a déjà été copié. Seuls les modifications, ajouts et suppressions effectués depuis la dernière exécution sont traités. Si la commande n’a pas encore été exécutée, rien n’est ignoré. L’indicateur **/MIR** est une excellente option pour les emplacements sources qui sont toujours activement utilisés et qui évoluent.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /COPY:copyflag[s]
   :::column-end:::
   :::column span="1":::
      Fidélité de la copie de fichier (la valeur par défaut est /COPY:DAT). Les indicateurs de copie sont : D = données, A = attributs, T = horodatages, S = sécurité = ACL NTFS, O = informations propriétaire, U=informations audit.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /COPYALL
   :::column-end:::
   :::column span="1":::
      Informations de fichier COPY ALL (équivalent de /COPY:DATSOU).
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /DCOPY:copyflag[s]
   :::column-end:::
   :::column span="1":::
      Fidélité de la copie des répertoires (la valeur par défaut est /DCOPY:DA). Les indicateurs de copie sont : D = données, A = attributs, T = horodatages.
   :::column-end:::
:::row-end:::

## <a name="phase-8-user-cut-over"></a>Phase 8 : Transfert de l’utilisateur

Quand vous exécutez la commande Robocopy pour la première fois, les utilisateurs et applications ont toujours accès aux fichiers sur le serveur Samba Linux et les modifient éventuellement. Il est possible que Robocopy traite un répertoire et passe au répertoire suivant, puis qu’un utilisateur à l’emplacement source (Linux) ajoute, modifie ou supprime un fichier qui ne sera pas traité durant cette exécution de Robocopy. Il s’agit du comportement attendu.

La première exécution consiste à déplacer la majeure partie des données vers votre instance Windows Server et dans le cloud à l’aide d’Azure File Sync. Cette première copie peut prendre beaucoup de temps selon les paramètres suivants :

* La bande passante de téléchargement.
* La bande passante de chargement.
* La vitesse du réseau local, ainsi que le nombre de threads Robocopy qui y correspondent de manière optimale.
* Le nombre d’éléments (fichiers et dossiers) qui doivent être traités par Robocopy et Azure File Sync.

Une fois l’exécution initiale terminée, réexécutez la commande.

Elle se termine plus rapidement la deuxième fois. En effet, seuls les éléments modifiés depuis la dernière exécution doivent être déplacés. Au cours de cette deuxième exécution, de nouvelles modifications peuvent toujours venir s’ajouter.

Répétez cette procédure jusqu’à ce que vous considériez que le temps d’arrêt nécessaire à la réalisation d’une opération Robocopy pour un emplacement spécifique est dans une fenêtre acceptable.

Lorsque vous trouvez les temps d’arrêt acceptables et que vous êtes prêt à mettre l’emplacement Linux hors connexion, vous pouvez modifier les listes de contrôle d’accès sur la racine de partage de sorte que les utilisateurs ne puissent plus accéder à l’emplacement. Vous pouvez également prendre toute autre mesure appropriée pour empêcher le contenu de changer dans ce dossier sur votre serveur Linux.

Exécutez une dernière fois la commande Robocopy pour traiter toutes les modifications qui n’ont pas encore été prises en compte. La durée de cette dernière étape dépend de la vitesse d’analyse de Robocopy. Vous pouvez estimer la durée d’exécution (correspondant au temps d’arrêt) en mesurant la durée de l’exécution précédente.

Créez un partage dans le dossier Windows Server et, le cas échéant, ajustez votre déploiement DFS-N pour qu’il pointe vers celui-ci. Veillez à définir les mêmes autorisations au niveau du partage que celles de vos partages SMB sur le serveur Samba Linux. Si vous avez utilisé des utilisateurs locaux sur votre serveur Samba Linux, vous devez les recréer en tant qu’utilisateurs locaux de Windows Server. Vous devez également mapper les SID existants déplacés par Robocopy vers votre instance Windows Server avec les SID de vos nouveaux utilisateurs locaux de Windows Server. Si vous avez utilisé des comptes Active Directory et des ACL, Robocopy les déplace en l’état et aucune action supplémentaire n’est nécessaire.

Vous avez terminé la migration d’un partage ou d’un groupe de partages vers une racine ou un volume commun (selon votre mappage dans la phase 1).

Vous pouvez essayer d’exécuter quelques-unes de ces copies en parallèle. Nous vous recommandons de traiter l’étendue d’un partage de fichiers Azure à la fois.

> [!WARNING]
> Une fois que vous avez déplacé toutes les données de votre serveur Samba Linux vers l’instance Windows Server et que votre migration est terminée, revenez à *tous* les groupes de synchronisation dans le portail Azure. Ajustez le pourcentage d’espace libre pour le volume de hiérarchisation cloud afin d’obtenir une meilleure solution d’utilisation du cache, par exemple 20 pour cent. 

La stratégie de libération d’espace dans le volume de hiérarchisation cloud agit au niveau du volume avec des points de terminaison potentiellement multiples pour la synchronisation à partir de ce dernier. Si vous oubliez de définir l’espace libre sur un point de terminaison de serveur, la synchronisation continue d’appliquer la règle la plus restrictive et tente de conserver 99 % d’espace disque disponible. Le cache local peut alors ne pas fonctionner comme prévu. Le niveau de performance peut être acceptable sauf si votre objectif est de disposer de l’espace de noms pour un volume contenant uniquement des données d’archivage rarement sollicitées et que vous réservez le reste de l’espace de stockage pour un autre scénario.

## <a name="troubleshoot"></a>Dépanner

Le problème le plus courant est un échec de la commande Robocopy de type **Volume plein** côté Windows Server. Toutes les heures, la hiérarchisation cloud retire le contenu du disque Windows Server local qui a été synchronisé. Son objectif est d’atteindre 99 pour cent d’espace libre sur le volume.

Laissez la synchronisation s’effectuer et la hiérarchisation cloud libérer l’espace disque. Vous pouvez observer l’opération dans l’Explorateur de fichiers sur Windows Server.

Quand la capacité disponible de votre instance Windows Server est suffisante, réexécutez la commande pour résoudre le problème. Cette situation n’entraîne aucune dégradation. Vous pouvez poursuivre en toute confiance. La nécessité de réexécuter la commande représente le seul inconvénient.

Pour plus d’informations sur la résolution des problèmes Azure File Sync, suivez le lien de la section suivante.

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez en apprendre davantage sur les partages de fichiers Azure et Azure File Sync. Les articles suivants contiennent des options avancées, les meilleures pratiques et des aides pour la résolution des problèmes. Ces articles sont liés à la [documentation relative aux partages de fichiers Azure](storage-files-introduction.md), le cas échéant.

* [Vue d’ensemble d’Azure File Sync](./storage-sync-files-planning.md)
* [Guide de déploiement d’Azure File Sync](storage-files-deployment-guide.md)
* [Résolution des problèmes Azure File Sync](storage-sync-files-troubleshoot.md)