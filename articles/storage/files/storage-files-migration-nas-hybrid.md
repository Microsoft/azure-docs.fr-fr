---
title: Migration NAS locale vers Azure File Sync
description: Découvrez comment migrer des fichiers d’un emplacement NAS (Network Attached Storage) vers un déploiement de cloud hybride à l’aide d’Azure File Sync et de partages de fichiers Azure.
author: fauhse
ms.service: storage
ms.topic: how-to
ms.date: 03/19/2020
ms.author: fauhse
ms.subservice: files
ms.openlocfilehash: 0d8d19256dfca21cc805c2689557099a6785f76b
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94629204"
---
# <a name="migrate-from-network-attached-storage-nas-to-a-hybrid-cloud-deployment-with-azure-file-sync"></a>Migrer de Network Attached Storage (NAS) vers un déploiement de cloud hybride avec Azure File Sync

Azure File Sync fonctionne sur les emplacements DAS (Direct Attached Storage) et ne prend pas en charge la synchronisation avec les emplacements NAS (Network Attached Storage).
En fait, la migration de vos fichiers est nécessaire et cet article vous guide tout au long de la planification et de l’exécution de cette migration.

## <a name="migration-goals"></a>Objectifs de la migration

L’objectif est de déplacer les partages que vous avez sur votre appliance NAS vers un Windows Server. Utilisez ensuite Azure File Sync pour un déploiement de cloud hybride. Cette migration doit être effectuée de manière à garantir l’intégrité des données de production, ainsi que la disponibilité pendant la migration. Garantir la disponibilité implique de garder les temps d’arrêt à un niveau minimal pour qu’ils respectent les fenêtres de maintenance habituelles ou ne les dépassent que légèrement.

## <a name="migration-overview"></a>Vue d’ensemble de la migration

Comme mentionné dans l’[article de présentation de la migration](storage-files-migration-overview.md) Azure Files, il est important d’utiliser l’outil de copie et l’approche appropriés. Votre appliance NAS expose les partages SMB directement sur votre réseau local. RoboCopy, intégré à Windows Server, est la meilleure façon de déplacer vos fichiers dans ce scénario de migration.

- Phase 1 : [Identifier le nombre de partages de fichiers Azure dont vous avez besoin](#phase-1-identify-how-many-azure-file-shares-you-need)
- Phase 2 : [Approvisionner un Windows Server local approprié](#phase-2-provision-a-suitable-windows-server-on-premises)
- Phase 3 : [Déployer la ressource cloud Azure File Sync](#phase-3-deploy-the-azure-file-sync-cloud-resource)
- Phase 4 : [Déployer des ressources de stockage Azure](#phase-4-deploy-azure-storage-resources)
- Phase 5 : [Déployer l’agent Azure File Sync](#phase-5-deploy-the-azure-file-sync-agent)
- Phase 6 : [Configurer Azure File Sync sur Windows Server](#phase-6-configure-azure-file-sync-on-the-windows-server)
- Phase 7 : [RoboCopy](#phase-7-robocopy)
- Phase 8 : [Transfert de l’utilisateur](#phase-8-user-cut-over)

## <a name="phase-1-identify-how-many-azure-file-shares-you-need"></a>Phase 1 : Identifier le nombre de partages de fichiers Azure dont vous avez besoin

[!INCLUDE [storage-files-migration-namespace-mapping](../../../includes/storage-files-migration-namespace-mapping.md)]

## <a name="phase-2-provision-a-suitable-windows-server-on-premises"></a>Phase 2 : Approvisionner un Windows Server local approprié

* Créez un serveur Windows Server 2019 (ou au minimum 2012R2) comme machine virtuelle ou serveur physique. Un cluster de basculement Windows Server est également pris en charge.
* Provisionnez ou ajoutez un stockage DAS (Direct Attached Storage). Notez que les dispositifs de stockage NAS ne sont pas pris en charge.

    La quantité de stockage que vous approvisionnez peut être inférieure à celle que vous utilisez actuellement sur votre appliance NAS, si vous utilisez la fonctionnalité de [hiérarchisation cloud](storage-sync-cloud-tiering.md) d’Azure File Sync.
    Toutefois, lorsque vous copiez vos fichiers de l’espace NAS plus grand vers le volume plus petit de Windows Server à une étape ultérieure, vous devrez travailler par lots :

    1. Déplacer un ensemble de fichiers qui tiennent sur le disque
    2. Autoriser la synchronisation des fichiers et la hiérarchisation cloud
    3. Lorsque de l’espace libre est créé sur le volume, passez au lot de fichiers suivant. 
    
    Vous pouvez éviter cette approche de traitement par lot en approvisionnant sur le Windows Server un espace équivalent à celui que vos fichiers occupent sur l’appliance NAS. Envisagez la déduplication sur NAS/Windows. Si vous ne souhaitez pas valider définitivement cette grande quantité de stockage sur votre Windows Server, vous pouvez réduire la taille du volume après la migration et avant d’ajuster les stratégies de hiérarchisation cloud. Cela crée un cache local plus petit de vos partages de fichiers Azure.

La configuration des ressources (calcul et RAM) du Windows Server que vous déployez dépend principalement du nombre d’éléments (fichiers et dossiers) que vous allez synchroniser. Nous vous recommandons d’utiliser une configuration haute performance si vous rencontrez des problèmes.

[Découvrez comment dimensionner un serveur Windows Server en fonction du nombre d’éléments (fichiers et dossiers) que vous devez synchroniser.](storage-sync-files-planning.md#recommended-system-resources)

> [!NOTE]
> L’article précédemment lié présente un tableau avec une plage pour la mémoire du serveur (RAM). Vous pouvez vous orienter vers le plus petit nombre de votre serveur, mais attendez-vous à ce que la synchronisation initiale prenne beaucoup plus de temps.

## <a name="phase-3-deploy-the-azure-file-sync-cloud-resource"></a>Phase 3 : Déployer la ressource cloud Azure File Sync

[!INCLUDE [storage-files-migration-deploy-afs-sss](../../../includes/storage-files-migration-deploy-azure-file-sync-storage-sync-service.md)]

## <a name="phase-4-deploy-azure-storage-resources"></a>Phase 4 : Déployer des ressources de stockage Azure

Au cours de cette phase, consultez la table de mappage de la phase 1 et utilisez-la pour approvisionner le nombre correct de comptes de stockage Azure et de partages de fichiers dans ceux-ci.

[!INCLUDE [storage-files-migration-provision-azfs](../../../includes/storage-files-migration-provision-azure-file-share.md)]

## <a name="phase-5-deploy-the-azure-file-sync-agent"></a>Phase 5 : Déployer l’agent Azure File Sync

[!INCLUDE [storage-files-migration-deploy-afs-agent](../../../includes/storage-files-migration-deploy-azure-file-sync-agent.md)]

## <a name="phase-6-configure-azure-file-sync-on-the-windows-server"></a>Phase 6 : Configurer Azure File Sync sur Windows Server

Votre serveur Windows Server local inscrit doit être prêt et connecté à Internet pour ce processus.

[!INCLUDE [storage-files-migration-configure-sync](../../../includes/storage-files-migration-configure-sync.md)]

> [!IMPORTANT]
> La hiérarchisation cloud désigne la fonctionnalité AFS qui permet au serveur local d’avoir moins de capacité de stockage que ce qui est stocké dans le cloud, tout en ayant l’espace de noms complet disponible. Les données intéressantes localement sont également mises en cache localement pour des performances d’accès rapides. La hiérarchisation cloud est une fonctionnalité facultative par « point de terminaison de serveur » Azure File Sync.

> [!WARNING]
> Si vous avez configuré moins de stockage sur vos volumes Windows Server que ce que vos données utilisaient sur l’appliance NAS, la hiérarchisation cloud est obligatoire. Si vous n’activez pas la hiérarchisation cloud, votre serveur ne libérera pas d’espace pour stocker tous les fichiers. Définissez votre stratégie de hiérarchisation pour un espace de volume libre de 99 % temporairement, pour la migration. Veillez à revenir à vos paramètres de hiérarchisation cloud une fois la migration terminée afin de la régler à un niveau utile à plus long terme.

Suivez de nouveau les étapes permettant de créer un groupe de synchronisation et d’ajouter le dossier serveur correspondant comme point de terminaison de serveur pour tous les partages de fichiers/emplacements de serveur Azure à configurer pour la synchronisation.

Après la création de tous les points de terminaison de serveur, la synchronisation fonctionne. Vous pouvez créer un fichier de test et le voir se synchroniser de votre serveur au partage de fichiers Azure connecté (comme décrit par le point de terminaison cloud dans le groupe de synchronisation).

Les deux emplacements, les dossiers du serveur et les partages de fichiers Azure sont autrement vides et attendent des données dans l’un ou l’autre emplacement. À l’étape suivante, vous commencerez à copier des fichiers dans Windows Server pour Azure File Sync afin de les déplacer dans le cloud. Si vous avez activé la hiérarchisation cloud, le serveur commence à hiérarchiser les fichiers, au cas où vous ne disposeriez pas de suffisamment de capacité sur les volumes locaux.

## <a name="phase-7-robocopy"></a>Phase 7 : Robocopy

L’approche de migration de base consiste à effectuer une copie à partir de l’appliance NAS vers le serveur Windows Server à l’aide de RoboCopy et à effectuer une synchronisation avec des partages de fichiers Azure à l’aide d’Azure File Sync.

Exécutez la première copie locale vers votre dossier Windows Server cible :

* Identifiez le premier emplacement sur votre appliance NAS.
* Identifiez le dossier correspondant sur le serveur Windows Server sur lequel Azure File Sync est déjà configuré.
* Démarrer la copie à l’aide de RoboCopy

La commande RoboCopy suivante copie les fichiers de votre stockage NAS vers votre dossier cible Windows Server. Le serveur Windows Server va le synchroniser avec le(s) partage(s) de fichiers Azure. 

Si vous avez configuré moins de stockage sur votre Windows Server que ce que vos fichiers occupent sur l’appliance NAS, cela signifie que vous avez configuré la hiérarchisation cloud. Quand le volume Windows Server local se remplit, la [hiérarchisation cloud](storage-sync-cloud-tiering.md) intervient et hiérarchise les fichiers qui ont déjà été correctement synchronisés. La hiérarchisation cloud génère suffisamment d’espace pour poursuivre la copie à partir de l’appliance NAS. La hiérarchisation cloud effectue une vérification toutes les heures pour déterminer ce qui a été synchronisé et libérer de l’espace disque pour atteindre l’espace de volume libre de 99 %.
Il est possible que RoboCopy déplace les fichiers plus rapidement que vous ne pouvez les synchroniser dans le cloud et le niveau local, ce qui réduit l’espace disque local. RoboCopy échoue. Il est recommandé de traiter les partages dans une séquence qui empêche cela. Par exemple, ne pas démarrer les travaux RoboCopy pour tous les partages en même temps ou déplacer uniquement les partages qui correspondent à la quantité actuelle d’espace libre sur le Windows Server, pour n’en citer que quelques-uns.

```console
Robocopy /MT:32 /UNILOG:<file name> /TEE /B /MIR /COPYALL /DCOPY:DAT <SourcePath> <Dest.Path>
```

Arrière-plan :

:::row:::
   :::column span="1":::
      /MT
   :::column-end:::
   :::column span="1":::
      Permet à RoboCopy de s’exécuter en multithread. La valeur par défaut est 8, le maximum est 128.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /UNILOG:\<file name\>
   :::column-end:::
   :::column span="1":::
      Renvoie l’état au fichier LOG au format UNICODE (remplace le journal existant).
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /TEE
   :::column-end:::
   :::column span="1":::
      Génère les sorties dans la fenêtre de la console. Utilisé conjointement avec la sortie dans un fichier journal.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /B
   :::column-end:::
   :::column span="1":::
      Exécute RoboCopy dans le même mode qu’une application de sauvegarde. Permet à RoboCopy de déplacer des fichiers pour lesquels l’utilisateur actuel n’a pas d’autorisations.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /MIR
   :::column-end:::
   :::column span="1":::
      Permet d’exécuter cette commande RoboCopy plusieurs fois, de manière séquentielle, sur la même cible/destination. La commande identifie et ignore ce qui a déjà été copié. Seuls les modifications, ajouts et « *suppressions* » effectués depuis la dernière exécution sont traités. Si la commande n’a pas encore été exécutée, rien n’est ignoré. L’indicateur */MIR* est une excellente option pour les emplacements sources qui sont toujours activement utilisés et qui évoluent.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /COPY:copyflag[s]
   :::column-end:::
   :::column span="1":::
      Fidélité de la copie de fichier (la valeur par défaut est /COPY:DAT), indicateurs de copie : D=Données, A=Attributs, T=Horodatages, S=Sécurité=ACL NTFS, O=Informations propriétaire, U=Informations audit
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /COPYALL
   :::column-end:::
   :::column span="1":::
      Copie de toutes les informations de fichier (équivalent à /COPY:DATSOU)
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /DCOPY:copyflag[s]
   :::column-end:::
   :::column span="1":::
      Fidélité de la copie des répertoires (la valeur par défaut est /DCOPY:DA), indicateurs de copie : D=Données, A=Attributs, T=Horodatages
   :::column-end:::
:::row-end:::

## <a name="phase-8-user-cut-over"></a>Phase 8 : Transfert de l’utilisateur

Quand vous exécutez la commande RoboCopy pour la première fois, les utilisateurs et applications ont toujours accès aux fichiers sur l’appliance NAS et peuvent éventuellement les modifier. Il est possible que RoboCopy traite un répertoire, passe au répertoire suivant, puis qu’un utilisateur accédant à l’emplacement source (NAS) ajoute, modifie ou supprime un fichier qui ne sera pas traité durant cette exécution de RoboCopy. Il s’agit du comportement attendu.

La première exécution consiste à déplacer la majeure partie des données vers votre Windows Server et dans le cloud à l’aide d’Azure File Sync. Cette première copie peut prendre beaucoup de temps selon les paramètres suivants :

* La bande passante de téléchargement
* La bande passante de chargement
* La vitesse du réseau local, ainsi que le nombre de threads RoboCopy qui y correspondent de manière optimale
* Le nombre d’éléments (fichiers et dossiers) qui doivent être traités par RoboCopy et Azure File Sync

Une fois l’exécution initiale terminée, réexécutez la commande.

Elle s’exécute plus rapidement la deuxième fois. En effet, elle doit déplacer uniquement les éléments modifiés depuis la dernière exécution. Au cours de cette deuxième exécution, de nouvelles modifications peuvent toujours venir s’ajouter.

Répétez cette procédure jusqu’à ce que vous considériez que le temps d’arrêt nécessaire à la réalisation d’une RoboCopy pour un emplacement spécifique est dans une fenêtre acceptable.

Dès lors que vous considérez que le temps d’arrêt est acceptable et que vous êtes prêt à mettre l’emplacement NAS hors connexion : Pour mettre l’accès utilisateur hors connexion, vous avez la possibilité de modifier les listes de contrôle d’accès (ACL, access-control list) sur la racine de partage, de sorte que les utilisateurs ne puissent plus accéder à l’emplacement, ou d’effectuer une autre étape appropriée qui empêche le contenu d’être modifié dans ce dossier sur votre emplacement NAS.

Exécutez une dernière fois la commande RoboCopy pour traiter toutes les modifications qui n’ont pas encore été prises en compte.
La durée de cette dernière étape dépend de la vitesse d’analyse de RoboCopy. Vous pouvez estimer la durée d’exécution (correspondant au temps d’arrêt) en mesurant la durée de l’exécution précédente.

Créez un partage dans le dossier Windows Server et, le cas échéant, ajustez votre déploiement DFS-N pour qu’il pointe vers celui-ci. Veillez à définir les mêmes autorisations au niveau du partage que celles de votre partage SMB NAS. Si vous aviez un NAS joint à un domaine d’entreprise, les SID de l’utilisateur correspondent automatiquement à mesure que la présence des utilisateurs est vérifiée dans Active Directory et que RoboCopy copie fidèlement les fichiers et les métadonnées. Si vous avez utilisé des utilisateurs locaux sur votre emplacement NAS, vous devez recréer ces utilisateurs en tant qu’utilisateurs locaux de Windows Server et mapper les SID existants que RoboCopy a déplacés sur votre Windows Server avec les SID de vos nouveaux utilisateurs locaux de Windows Server.

Vous avez terminé la migration d’un partage/groupe de partages vers une racine ou un volume commun (selon votre mappage à partir de la phase 1)

Vous pouvez essayer d’exécuter quelques-unes de ces copies en parallèle. Nous vous recommandons de traiter l’étendue d’un partage de fichiers Azure à la fois.

> [!WARNING]
> Quand vous avez déplacé toutes les données de votre emplacement NAS vers le serveur Windows Server et que la migration est terminée : revenez à ***tous** les groupes de synchronisation dans le Portail Azure et définissez le pourcentage d’espace de volume libre assuré par la hiérarchisation cloud sur une valeur mieux adaptée à l’utilisation du cache, par exemple 20 %. 

La stratégie de libération d’espace de volume par hiérarchisation cloud agit au niveau du volume avec potentiellement plusieurs points de terminaison pour la synchronisation à partir du volume. Si vous oubliez de définir l’espace libre sur un point de terminaison de serveur, la synchronisation continue d’appliquer la règle la plus restrictive et tente de conserver un espace disque disponible de 99 %. Dans ce cas, vous n’obtiendrez pas les performances attendues du cache local, sauf si vous souhaitez seulement disposer de l’espace de noms pour un volume contenant uniquement des données d’archivage rarement sollicitées et que vous réservez le reste de l’espace de stockage pour un autre scénario.

## <a name="troubleshoot"></a>Dépanner

Le problème que vous êtes le plus susceptible de rencontrer est un échec de la commande RoboCopy de type _« Volume plein » côté Windows Server. Toutes les heures, la hiérarchisation cloud retire le contenu du disque Windows Server local, qui a été synchronisé. Son objectif est d’atteindre 99 % d’espace libre sur le volume.

Laissez la synchronisation s’effectuer et la hiérarchisation cloud libérer l’espace disque. Vous pouvez observer l’opération dans l’Explorateur de fichiers sur votre serveur Windows Server.

Quand la capacité disponible de votre serveur Windows Server est suffisante, réexécutez la commande pour résoudre le problème. Cette situation n’entraîne aucune dégradation. Vous pouvez poursuivre en toute confiance. La nécessité de réexécuter la commande représente le seul inconvénient.

Pour plus d’informations sur la résolution des problèmes Azure File Sync, suivez le lien de la section suivante.

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez en apprendre davantage sur les partages de fichiers Azure et Azure File Sync. Les articles suivants vous aident à comprendre les options avancées et les meilleures pratiques et contiennent également des aides pour la résolution des problèmes. Ces articles sont liés à la [documentation relative aux partages de fichiers Azure](storage-files-introduction.md), le cas échéant.

* [Vue d’ensemble d’AFS](./storage-sync-files-planning.md)
* [Déployer AFS](storage-files-deployment-guide.md)
* [Résoudre les problèmes de synchronisation de fichiers Azure](storage-sync-files-troubleshoot.md)