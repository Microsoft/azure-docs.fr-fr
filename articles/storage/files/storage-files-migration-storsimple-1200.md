---
title: Migration à partir de StorSimple 1200 vers Azure File Sync
description: Découvrez comment effectuer une migration d’une appliance virtuelle StorSimple série 1200 vers Azure File Sync.
author: fauhse
ms.service: storage
ms.topic: how-to
ms.date: 03/09/2020
ms.author: fauhse
ms.subservice: files
ms.openlocfilehash: 78c7953ef6432d37542a7a8b06f226a07f2b701f
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94630479"
---
# <a name="storsimple-1200-migration-to-azure-file-sync"></a>Migration à partir de StorSimple 1200 vers Azure File Sync

StorSimple série 1200 est une appliance virtuelle exécutée dans un centre de données local. Il est possible d’effectuer une migration des données de cette appliance vers un environnement Azure File Sync. Azure File Sync est le service Azure à long terme stratégique et par défaut vers lequel les appliances StorSimple peuvent être migrées.

StorSimple série 1200 atteindra sa [fin de vie](https://support.microsoft.com/en-us/lifecycle/search?alpha=StorSimple%201200%20Series) en décembre 2022.  Il est important de commencer à planifier votre migration dès que possible. Cet article décrit les étapes à suivre pour effectuer correctement une migration vers Azure File Sync et fournit les connaissances générales nécessaires. 

## <a name="azure-file-sync"></a>Azure File Sync

> [!IMPORTANT]
> Microsoft s’engage à aider les clients lors de la migration. Envoyez un e-mail à l’adresse AzureFilesMigration@microsoft.com pour obtenir un plan de migration personnalisé ainsi qu’une assistance pendant la migration.

Azure File Sync est un service cloud Microsoft basé sur deux composants principaux :

* Synchronisation des fichiers et hiérarchisation cloud.
* Partages de fichiers comme stockage natif dans Azure, accessibles par le biais de différents protocoles comme SMB et File REST. Un partage de fichiers Azure est comparable à un partage de fichiers sur un serveur Windows Server, que vous pouvez monter comme lecteur réseau de façon native. Il prend en charge des aspects clés de la fidélité des fichiers comme les attributs, les autorisations et les horodatages. Aucun service/application n’est nécessaire pour interpréter les fichiers et dossiers stockés dans le cloud (comme c’est le cas avec StorSimple). Il s’agit de l’approche idéale et la plus flexible pour stocker des données de serveur de fichiers à usage général et certaines données d’application dans le cloud.

Cet article est consacré aux étapes de migration. Si vous souhaitez en savoir plus sur Azure File Sync avant d’effectuer la migration, consultez les articles suivants :

* [Azure File Sync - Vue d’ensemble](./storage-sync-files-planning.md "Vue d’ensemble")
* [Azure File Sync - Guide de déploiement](storage-sync-files-deployment-guide.md)

## <a name="migration-goals"></a>Objectifs de la migration

L’objectif consiste à garantir l’intégrité des données de production et la disponibilité. Garantir la disponibilité implique de garder les temps d’arrêt à un niveau minimal pour qu’ils respectent les fenêtres de maintenance habituelles ou ne les dépassent que légèrement.

## <a name="storsimple-1200-migration-path-to-azure-file-sync"></a>Chemin de migration à partir de StorSimple 1200 vers Azure File Sync

L’exécution d’un agent Azure File Sync nécessite un serveur Windows Server local. Il doit s’agir au minimum d’un serveur 2012R2, la version idéale étant Windows Server 2019.

Il existe de nombreux chemins de migration possibles, et il serait trop long de tous les documenter dans un article et d’expliquer clairement les risques et inconvénients qu’ils présentent par rapport à celui que nous recommandons de privilégier dans cet article.

:::image type="content" source="media/storage-files-migration-storsimple-shared/storsimple-1200-migration-overview.png" alt-text="Chemin de migration - vue d’ensemble des étapes décrites dans cet article.":::

L’image précédente illustre les étapes correspondant aux sections de cet article.

### <a name="step-1-provision-your-on-premises-windows-server-and-storage"></a>Étape 1 : Provisionner le serveur Windows Server local et le stockage

1. Créez un serveur Windows Server 2019 (ou au minimum 2012R2) comme machine virtuelle ou serveur physique. Un cluster de basculement Windows Server est également pris en charge.
2. Provisionnez ou ajoutez un stockage DAS (Direct Attached Storage). Notez que les dispositifs de stockage NAS ne sont pas pris en charge. La taille du stockage Windows Server doit correspondre au minimum à la capacité disponible de l’appliance virtuelle StorSimple 1200.

### <a name="step-2-configure-your-windows-server-storage"></a>Étape 2 : Configurer le stockage Windows Server

Dans le cadre de cette étape, vous allez mapper votre structure de stockage StorSimple (volumes et partages) à votre structure de stockage Windows Server.
Si vous envisagez d’apporter des modifications à votre structure de stockage (nombre de volumes, association de dossiers de données aux volumes ou structure de sous-dossiers au-dessus ou en dessous de vos partages SMB/NFS actuels), vous devez y penser dès à présent.
La modification de la structure de fichiers et de dossiers après la configuration d’Azure File Sync est particulièrement fastidieuses et doit être évitée.
Cet article suppose que vous effectuez un mappage 1:1. Vous devez donc prendre en considération vos modifications de mappage à mesure que vous effectuez les étapes de cet article.

* Aucune de vos données de production ne doit se retrouver sur le volume système Windows Server. La hiérarchisation cloud n’est pas prise en charge sur les volumes système. Toutefois, cette fonctionnalité est nécessaire pour la migration et pour la continuité des opérations dans le cadre d’un remplacement de StorSimple.
* Provisionnez sur votre serveur Windows Server le même nombre de volumes que celui de votre appliance virtuelle StorSimple 1200.
* Configurez les rôles, fonctionnalités et paramètres Windows Server dont vous avez besoin. Nous vous recommandons d’accepter les mises à jour Windows Server pour garantir la sécurité de votre système d’exploitation et vous assurer qu’il est systématiquement à jour. De même, nous vous recommandons de vous abonner à Microsoft Update pour assurer la mise à jour des applications Microsoft et notamment de l’agent Azure File Sync.
* Ne configurez aucun dossier ou partage avant de prendre connaissance des étapes suivantes.

### <a name="step-3-deploy-the-first-azure-file-sync-cloud-resource"></a>Étape 3 : Déployer la première ressource cloud Azure File Sync

[!INCLUDE [storage-files-migration-deploy-afs-sss](../../../includes/storage-files-migration-deploy-azure-file-sync-storage-sync-service.md)]

### <a name="step-4-match-your-local-volume-and-folder-structure-to-azure-file-sync-and-azure-file-share-resources"></a>Étape 4 : Mettre en correspondance votre structure de dossiers et de volumes locale avec Azure File Sync et les ressources de partage de fichiers Azure

[!INCLUDE [storage-files-migration-namespace-mapping](../../../includes/storage-files-migration-namespace-mapping.md)]

### <a name="step-5-provision-azure-file-shares"></a>Étape 5 : Provisionner les partages de fichiers Azure

[!INCLUDE [storage-files-migration-provision-azfs](../../../includes/storage-files-migration-provision-azure-file-share.md)]

### <a name="step-6-configure-windows-server-target-folders"></a>Étape 6 : Configurer les dossiers Windows Server cibles

Dans le cadre des étapes précédentes, vous avez pris en compte tous les aspects qui détermineront les composants de vos topologies de synchronisation. À présent, vous devez préparer le serveur pour qu’il reçoive les fichiers à charger.

Créez **tous** les dossiers qui seront synchronisés avec leur propre partage de fichiers Azure.
Il est important de suivre la structure de dossiers que vous avez documentée précédemment. Par exemple, si vous avez décidé de synchroniser plusieurs partages SMB locaux avec un même partage de fichiers Azure, vous devez les placer dans un dossier racine commun sur le volume. Créez ce dossier racine cible sur le volume dès à présent.

Le nombre de partages de fichiers Azure que vous avez provisionnés doit correspondre au nombre de dossiers que vous avez créés au cours de cette étape plus le nombre de volumes que vous allez synchroniser au niveau racine.

### <a name="step-7-deploy-the-azure-file-sync-agent"></a>Étape 7 : Déployer l’agent Azure File Sync

[!INCLUDE [storage-files-migration-deploy-afs-agent](../../../includes/storage-files-migration-deploy-azure-file-sync-agent.md)]

### <a name="step-8-configure-sync"></a>Étape 8 : Configurer la synchronisation

[!INCLUDE [storage-files-migration-configure-sync](../../../includes/storage-files-migration-configure-sync.md)]

> [!WARNING]
> **Veillez à activer la hiérarchisation cloud.** Cette activation est nécessaire si votre serveur local ne dispose pas de l’espace suffisant pour stocker l’ensemble des données de votre stockage cloud StorSimple. Définissez votre stratégie de hiérarchisation pour un espace de volume libre de 99 % temporairement, pour la migration.

Suivez de nouveau les étapes permettant de créer un groupe de synchronisation et d’ajouter le dossier serveur correspondant comme point de terminaison de serveur pour tous les partages de fichiers/emplacements de serveur Azure à configurer pour la synchronisation.

### <a name="step-9-copy-your-files"></a>Étape 9 : Copier vos fichiers

L’approche de migration de base consiste à effectuer une copie à partir de l’appliance virtuelle StorSimple vers le serveur Windows Server à l’aide de RoboCopy et à effectuer une synchronisation avec des partages de fichiers Azure à l’aide d’Azure File Sync.

Exécutez la première copie locale vers votre dossier Windows Server cible :

* Identifiez le premier emplacement sur votre appliance virtuelle StorSimple.
* Identifiez le dossier correspondant sur le serveur Windows Server sur lequel Azure File Sync est déjà configuré.
* Démarrer la copie à l’aide de RoboCopy

La commande RoboCopy suivante rappelle les fichiers de votre stockage Azure StorSimple sur votre stockage StorSimple local, puis les déplace vers le dossier Windows Server cible. Le serveur Windows Server va le synchroniser avec le(s) partage(s) de fichiers Azure. Quand le volume Windows Server local se remplit, la hiérarchisation cloud intervient et hiérarchise les fichiers qui ont déjà été correctement synchronisés. La hiérarchisation cloud génère suffisamment d’espace pour poursuivre la copie à partir de l’appliance virtuelle StorSimple. La hiérarchisation cloud effectue une vérification toutes les heures pour déterminer ce qui a été synchronisé et libérer de l’espace disque pour atteindre l’espace de volume libre de 99 %.

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
      /UNILOG:<file name>
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
      Permet d’exécuter cette commande RoboCopy plusieurs fois, de manière séquentielle, sur la même cible/destination. La commande identifie et ignore ce qui a déjà été copié. Seuls les modifications, ajouts et « *suppressions* » effectués depuis la dernière exécution sont traités. Si la commande n’a pas encore été exécutée, rien n’est ignoré. Il s’agit d’une excellente option pour les emplacements sources qui sont toujours activement utilisés et qui évoluent.
   :::column-end:::
:::row-end:::
:::row:::
   :::column span="1":::
      /COPY:indicateur[s]
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

Quand vous exécutez la commande RoboCopy pour la première fois, les utilisateurs et applications ont toujours accès aux fichiers et dossiers StorSimple et peuvent éventuellement les modifier. Il est possible que RoboCopy traite un répertoire, passe au répertoire suivant, puis qu’un utilisateur accédant à l’emplacement source (StorSimple) ajoute, modifie ou supprime un fichier qui ne sera pas traité durant cette exécution de RoboCopy. Bien sûr.

La première exécution consiste à déplacer la majeure partie des données vers l’emplacement local sur votre serveur Windows Server et à effectuer une sauvegarde dans le cloud à l’aide d’Azure File Sync. Ceci peut prendre beaucoup de temps selon les paramètres suivants :

* La bande passante de téléchargement
* La vitesse de rappel du service cloud StorSimple
* La bande passante de chargement
* Le nombre d’éléments (fichiers et dossiers) qui doivent être traités par l’un ou l’autre des services

Une fois l’exécution initiale terminée, réexécutez la commande.

Elle s’exécute plus rapidement la deuxième fois. En effet, elle doit déplacer uniquement les éléments modifiés depuis la dernière exécution. Les modifications sont récentes et sont donc généralement déjà effectuées localement sur StorSimple. Ceci réduit encore la durée d’exécution, car la nécessité de rappeler des fichiers à partir du cloud est moindre. Au cours de cette deuxième exécution, de nouvelles modifications peuvent toujours venir s’ajouter.

Répétez cette procédure jusqu’à ce que vous considériez que le temps d’arrêt impliqué par l’exécution est acceptable.

Dès lors que vous considérez que le temps d’arrêt est acceptable et que vous êtes prêt à mettre l’emplacement StorSimple hors connexion, effectuez l’opération : par exemple, supprimez le partage SMB pour qu’aucun utilisateur ne puisse accéder au dossier ou prenez toute autre mesure appropriée pour empêcher toute modification du contenu de ce dossier sur StorSimple.

Exécutez une dernière fois la commande RoboCopy pour traiter toutes les modifications qui n’ont pas encore été prises en compte.
La durée de cette dernière étape dépend de la vitesse d’analyse de RoboCopy. Vous pouvez estimer la durée d’exécution (correspondant au temps d’arrêt) en mesurant la durée de l’exécution précédente.

Créez un partage dans le dossier Windows Server et, le cas échéant, ajustez votre déploiement DFS-N pour qu’il pointe vers celui-ci. Veillez à définir les mêmes autorisations au niveau du partage que celles de votre partage SMB StorSimple.

Vous avez terminé la migration d’un partage/groupe de partages vers une racine ou un volume commun (selon ce que vous avez mappé et décidé d’envoyer vers le même partage de fichiers Azure).

Vous pouvez essayer d’exécuter quelques-unes de ces copies en parallèle. Nous vous recommandons de traiter l’étendue d’un partage de fichiers Azure à la fois.

> [!WARNING]
> Quand vous avez déplacé toutes les données de votre stockage StorSimple vers le serveur Windows Server et que la migration est terminée : revenez à ***tous** les groupes de synchronisation dans le Portail Azure et définissez le pourcentage d’espace de volume libre assuré par la hiérarchisation cloud sur une valeur mieux adaptée à l’utilisation du cache, par exemple 20 %. 

La stratégie de libération d’espace de volume par hiérarchisation cloud agit au niveau du volume avec potentiellement plusieurs points de terminaison pour la synchronisation à partir du volume. Si vous oubliez de définir l’espace libre sur un point de terminaison de serveur, la synchronisation continue d’appliquer la règle la plus restrictive et tente de conserver un espace disque disponible de 99 %. Dans ce cas, vous n’obtiendrez pas les performances attendues du cache local, sauf si vous souhaitez seulement disposer de l’espace de noms pour un volume contenant uniquement des données d’archivage auxquelles il est rarement fait accès.

## <a name="troubleshoot"></a>Dépanner

Le problème que vous êtes le plus susceptible de rencontrer est un échec de la commande RoboCopy de type _« Volume plein » côté Windows Server. Dans ce cas, la vitesse de téléchargement est probablement supérieure à la vitesse de chargement. Toutes les heures, la hiérarchisation cloud retire le contenu du disque Windows Server local, qui a été synchronisé.

Laissez la synchronisation s’effectuer et la hiérarchisation cloud libérer l’espace disque. Vous pouvez observer l’opération dans l’Explorateur de fichiers sur votre serveur Windows Server.

Quand la capacité disponible de votre serveur Windows Server est suffisante, réexécutez la commande pour résoudre le problème. Cette situation n’entraîne aucune dégradation. Vous pouvez poursuivre en toute confiance. La nécessité de réexécuter la commande représente le seul inconvénient.

D’autres problèmes liés à Azure File Sync peuvent se présenter
de façon très occasionnelle. Dans ce cas, consultez le **guide de résolution des problèmes d’Azure File Sync (lien)** .

## <a name="relevant-links"></a>Liens pertinents

Contenu relatif à la migration :

* [Guide de migration à partir de StorSimple série 8000](storage-files-migration-storsimple-8000.md)

Contenu relatif à Azure File Sync :

* [Vue d’ensemble d’AFS](./storage-sync-files-planning.md)
* [Déployer AFS](storage-files-deployment-guide.md)
* [Résoudre les problèmes de synchronisation de fichiers Azure](storage-sync-files-troubleshoot.md)