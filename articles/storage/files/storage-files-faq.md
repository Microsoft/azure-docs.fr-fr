---
title: Questions fréquentes (FAQ) sur Azure Files | Microsoft Docs
description: Obtenez des réponses aux questions fréquemment posées sur Azure Files. Vous pouvez monter des partages de fichiers Azure simultanément sur des déploiements cloud ou locaux de Windows, Linux ou macOS.
author: roygara
ms.service: storage
ms.date: 02/23/2020
ms.author: rogarana
ms.subservice: files
ms.topic: conceptual
ms.openlocfilehash: 266862a1691387caf1b1ffa9a47b0e60e84970d8
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96492129"
---
# <a name="frequently-asked-questions-faq-about-azure-files"></a>Questions fréquentes (FAQ) sur Azure Files
[Azure Files](storage-files-introduction.md) offre des partages de fichiers pleinement managés dans le cloud qui sont accessibles via le [protocole SMB (Server Message Block)](/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) standard et le [protocole NFS (Network File System)](https://en.wikipedia.org/wiki/Network_File_System) (préversion). Vous pouvez monter des partages de fichiers Azure simultanément sur des déploiements cloud ou locaux de Windows, Linux et macOS. Vous pouvez également mettre en cache des partages de fichiers Azure sur des ordinateurs Windows Server à l’aide d’Azure File Sync pour bénéficier d’un accès rapide proche de l’endroit où les données sont utilisées.

Cet article répond à des questions courantes sur les fonctionnalités d’Azure Files, notamment concernant l’utilisation d’Azure File Sync avec Azure Files. Si vous ne trouvez pas de réponse à votre question ici, vous pouvez nous joindre par le biais des méthodes suivantes (par ordre de priorité) :

1. La section Commentaires de cet article
2. [Page de questions Microsoft Q&R sur le Stockage Azure](/answers/topics/azure-file-storage.html).
3. [Azure Files UserVoice](https://feedback.azure.com/forums/217298-storage/category/180670-files) 
4. Support Microsoft Pour créer une demande de support, dans le portail Azure, sous l’onglet **Aide**, sélectionnez le bouton **Aide et support**, puis **Nouvelle demande de support**.

## <a name="general"></a>Général
* <a id="why-files-useful"></a>
  **En quoi Azure Files est-il utile ?**  
   Vous pouvez utiliser Azure Files pour créer des partages de fichiers dans le cloud, sans être responsable de la gestion de la surcharge d’un serveur physique, d’un appareil ou d’un dispositif. Nous nous chargeons du travail monotone pour vous, notamment des mises à jour du système d’exploitation et du remplacement des disques défectueux. Pour en savoir plus sur les scénarios qui se prêtent à l’utilisation d’Azure Files, consultez [Pourquoi Azure Files est-il utile ?](storage-files-introduction.md#why-azure-files-is-useful).

* <a id="file-access-options"></a>
  **Quelles sont les différentes façons d’accéder aux fichiers dans Azure Files ?**  
    Les partages de fichiers SMB peuvent être montés sur votre ordinateur local à l’aide du protocole SMB 3.0, ou vous pouvez utiliser des outils tels que l’[Explorateur de stockage](https://storageexplorer.com/) pour accéder aux fichiers dans votre partage de fichiers. Les partages de fichiers NFS peuvent être montés sur votre ordinateur local par copie/collage du script fourni par le portail Azure. À partir de votre application, vous pouvez utiliser des bibliothèques clientes de stockage, des API REST, PowerShell ou Azure CLI pour accéder à vos fichiers dans le partage de fichiers Azure.

* <a id="what-is-afs"></a>
  **Qu’est-ce qu’Azure File Sync ?**  
    Vous pouvez utiliser Azure File Sync pour centraliser les partages de fichiers de votre organisation dans Azure Files tout en conservant la flexibilité, le niveau de performance et la compatibilité d’un serveur de fichiers local. Azure File Sync transforme vos ordinateurs Windows Server en un cache rapide de votre partage de fichiers Azure. Vous pouvez utiliser tout protocole disponible dans Windows Server pour accéder à vos données localement, notamment SMB, NFS (Network File System) et FTPS (File Transfer Protocol Service). Vous pouvez avoir autant de caches que nécessaire dans le monde entier.

* <a id="files-versus-blobs"></a>
  **Pourquoi utiliser un partage de fichiers Azure plutôt que le stockage Blob Azure pour mes données ?**  
    Azure Files et le stockage Blob Azure permettent tous les deux de stocker de grandes quantités de données dans le cloud, mais sont utiles à des fins légèrement différentes. 
    
    Le stockage Blob Azure est utile pour les applications cloud à grande échelle qui doivent stocker des données non structurées. Le stockage Blob Azure étant une abstraction de stockage plus simple qu’un véritable système de fichiers, le niveau de performance et la mise à l’échelle s’en trouvent optimisés. Vous pouvez accéder au stockage Blob Azure uniquement par le biais de bibliothèques clientes basées sur REST (ou directement par le biais du protocole basé sur REST).

    Azure Files est plus précisément un système de fichiers. Azure Files a toutes les abstractions de fichiers que vous connaissez et appréciez suite à vos années d’expérience avec les systèmes d’exploitation locaux. Comme le stockage Blob Azure, Azure Files offre une interface REST et des bibliothèques clientes basées sur REST. Contrairement au stockage Blob Azure, Azure Files offre un accès SMB ou NFS aux partages de fichiers Azure. Les partages de fichiers peuvent être montés directement sur Windows, Linux ou macOS, localement ou sur des machines virtuelles cloud, sans écrire aucun code ni joindre des pilotes spéciaux au système de fichiers. Vous pouvez également mettre en cache des partages de fichiers SMB Azure sur des serveurs de fichiers locaux à l’aide d’Azure File Sync pour bénéficier d’un accès rapide près de l’endroit où les données sont utilisées. 
   
    Pour obtenir une description plus approfondie des différences entre Azure Files et le stockage Blob Azure, consultez [Présentation des principaux services de stockage Azure](../common/storage-introduction.md). Pour en savoir plus sur le stockage Blob Azure, consultez [Présentation du Stockage Blob](../blobs/storage-blobs-introduction.md).

* <a id="files-versus-disks"></a>**Pourquoi utiliser un partage de fichiers Azure plutôt que Disques Azure ?**  
    Un disque dans Azure Disks est simplement un disque. Pour tirer parti de Disques Azure, vous devez attacher un disque à une machine virtuelle qui s’exécute dans Azure. Vous pouvez utiliser Disques Azure pour toutes les opérations que vous effectueriez sur un disque sur un serveur local. Vous pouvez l’utiliser en tant que disque de système d’exploitation, espace d’échange pour un système d’exploitation ou stockage dédié pour une application. Vous pouvez notamment recourir à Disques Azure pour créer un serveur de fichiers dans le cloud et l’utiliser là ou vous utiliseriez un partage de fichiers Azure. Déployer un serveur de fichiers sur des machines virtuelles Azure est un moyen très efficace d’obtenir un stockage de fichiers dans Azure quand vous avez besoin d’options de déploiement qui ne sont pas prises en charge par Azure Files. 

    Toutefois, exécuter un serveur de fichiers avec Disques Azure en tant que stockage backend est généralement beaucoup plus onéreux qu’utiliser un partage de fichiers Azure, pour plusieurs raisons. Tout d’abord, outre le stockage sur disque, vous devez payer les dépenses liées à l’exécution d’une ou plusieurs machines virtuelles Azure. Ensuite, vous devez également gérer les machines virtuelles servant à exécuter le serveur de fichiers. Par exemple, vous êtes responsable des mises à niveau du système d’exploitation. Pour finir, si des données doivent être mises en cache localement, il vous appartient de configurer et de gérer des technologies de réplication (telles que la réplication du système de fichiers DFS).

    Une approche intéressante pour tirer parti d’Azure Files et d’un serveur de fichiers hébergé sur des machines virtuelles Azure (en plus d’utiliser Disques Azure comme stockage backend) consiste à installer Azure File Sync sur un serveur de fichiers hébergé sur une machine virtuelle cloud. Si le partage de fichiers Azure se trouve dans la même région que votre serveur de fichiers, vous pouvez activer la hiérarchisation cloud et définir le pourcentage d’espace libre du volume sur la valeur maximale (99 %). Cela garantit la duplication minimale des données. Vous pouvez également utiliser les applications de votre choix avec vos serveurs de fichiers, telles que des applications qui exigent la prise en charge du protocole NFS.

    Pour plus d’informations sur une option permettant de configurer un serveur de fichiers à hautes performances et à disponibilité élevée dans Azure, consultez [Déploiement de clusters invités de machine virtuelle IaaS dans Microsoft Azure](https://blogs.msdn.microsoft.com/clustering/2017/02/14/deploying-an-iaas-vm-guest-clusters-in-microsoft-azure/). Pour obtenir une description plus approfondie des différences entre Azure Files et Disques Azure, consultez [Présentation des principaux services de stockage Azure](../common/storage-introduction.md). Pour en savoir plus sur Disques Azure, consultez [Vue d’ensemble d’Azure Disques managés](../../virtual-machines/managed-disks-overview.md).

* <a id="get-started"></a>
  **Comment commencer à utiliser Azure Files ?**  
   La prise en main d’Azure Files est simple. Tout d’abord, [créez un partage de fichiers SMB](storage-how-to-create-file-share.md) ou [créez un partage NFS](storage-files-how-to-create-nfs-shares.md), puis montez-le dans le système d’exploitation de votre choix : 

  * [Monter un partage SMB dans Windows](storage-how-to-use-files-windows.md)
  * [Monter un partage SMB dans Linux](storage-how-to-use-files-linux.md)
  * [Monter un partage SMB dans macOS](storage-how-to-use-files-mac.md)
  * [Monter un partage de fichiers NFS](storage-files-how-to-mount-nfs-shares.md)

    Pour obtenir un guide détaillé sur le déploiement d’un partage de fichiers Azure afin de remplacer des partages de fichiers de production dans votre organisation, consultez [Planification d’un déploiement Azure Files](storage-files-planning.md).

* <a id="redundancy-options"></a>
  **Quelles sont les options de redondance de stockage prises en charge par Azure Files ?**  
    À l’heure actuelle, Azure Files prend en charge le stockage localement redondant (LRS), le stockage redondant dans une zone (ZRS), le stockage géoredondant (GRS) et le stockage géoredondant dans une zone(GZRS). Le niveau Premium d’Azure Files ne prend actuellement en charge que LRS et ZRS.

* <a id="tier-options"></a>
  **Quels sont les niveaux de stockage pris en charge par Azure Files ?**  
    Azure Files prend en charge deux niveaux de stockage : Premium et standard. Les partages de fichiers standard sont créés dans les comptes de stockage (GPv1 ou GPv2) à usage général, alors que les partages de fichiers Premium sont créés dans les comptes de stockage FileStorage. Découvrez comment créer des [partages de fichiers standard](storage-how-to-create-file-share.md) et [des partages de fichiers Premium](storage-how-to-create-premium-fileshare.md). 
    
    > [!NOTE]
    > Vous ne pouvez pas créer de partages de fichiers Azure à partir de comptes de stockage d’objets blob ou de comptes de stockage *Premium* à usage général (GPv1 ou GPv2). Les partages de fichiers Azure standard doivent être créés dans des comptes *standard* à usage général uniquement, et les partages Azure Premium doivent être créés dans des comptes de stockage FileStorage uniquement. Les comptes de stockage *Premium* à usage général (GPv1 et GPv2) sont destinés aux objets blob de pages Premium uniquement. 

* <a id="file-locking"></a>
  **Azure Files prend-il en charge le verrouillage de fichiers ?**  
    Oui, Azure Files prend entièrement en charge le verrouillage de fichiers SMB/Windows, [voir les détails](/rest/api/storageservices/managing-file-locks).

* <a id="give-us-feedback"></a>
  **Je souhaite vraiment qu’une fonctionnalité soit ajoutée à Azure Files. Pouvez vous ajouter ?**  
    L’équipe Azure Files est à l’écoute de tous vos commentaires sur notre service. Veuillez voter pour les demandes de fonctionnalités sur le [forum UserVoice consacré à Azure Files](https://feedback.azure.com/forums/217298-storage/category/180670-files). Nous sommes impatients de mettre à votre disposition de nombreuses nouvelles fonctionnalités.

## <a name="azure-file-sync"></a>Azure File Sync

* <a id="afs-region-availability"></a>
  **Quelles sont les régions prises en charge par Azure File Sync ?**  
    Vous trouverez la liste des régions disponibles sur la section [Disponibilité des régions](storage-sync-files-planning.md#azure-file-sync-region-availability) du guide de planification Azure File Sync. Nous allons en permanence ajouter des supports pour des régions supplémentaires, y compris les régions privées.

* <a id="cross-domain-sync"></a>
  **Un même groupe de synchronisation peut-il contenir des serveurs joints à un domaine et des serveurs non joints à un domaine ?**  
    Oui. Un groupe de synchronisation peut contenir des points de terminaison de serveur qui ont des appartenances Active Directory différentes, même s’ils ne sont pas joints à un domaine. Bien que cette configuration fonctionne sur le plan technique, nous la déconseillons comme configuration standard, car les listes de contrôle d’accès (ACL, Access Control Lists) définies pour les fichiers et dossiers sur un serveur peuvent ne pas être applicables par d’autres serveurs dans le groupe de synchronisation. Pour de meilleurs résultats, nous vous recommandons d’effectuer une synchronisation entre des serveurs qui sont dans la même forêt Active Directory, entre des serveurs qui sont dans des forêts Active Directory différentes mais qui ont des relations d’approbation établies, ou entre des serveurs n’appartenant pas un domaine. Nous vous déconseillons d’utiliser une combinaison de ces configurations.

* <a id="afs-change-detection"></a>
  **J’ai créé un fichier directement dans mon partage de fichiers Azure à l’aide de SMB ou dans le portail. Combien de temps faut-il pour que le fichier soit synchronisé vers les serveurs du groupe de synchronisation ?**  
    [!INCLUDE [storage-sync-files-change-detection](../../../includes/storage-sync-files-change-detection.md)]


* <a id="afs-sync-time"></a>
  **Combien de temps faut-il pour télécharger 1 Tio de données pour Azure File Sync ?**
  
    Les performances varient en fonction de vos paramètres d’environnement, de votre configuration et du type de synchronisation (initiale ou continue). Pour plus d’informations, consultez la page [Mesures de performances d’Azure File Sync](storage-files-scale-targets.md#azure-file-sync-performance-metrics).

* <a id="afs-conflict-resolution"></a>**Si le même fichier est modifié sur deux serveurs à peu près au même moment, que se passe-t-il ?**  
    Azure File Sync utilise une stratégie de résolution de conflit simple : nous conservons les modifications apportées aux fichiers modifiés dans deux points de terminaison en même temps. Le fichier le plus récemment écrit conserve son nom d’origine. L’ancien fichier (déterminé par LastWriteTime) a le nom du point de terminaison et le numéro de conflit ajoutés au nom de fichier. Pour les points de terminaison de serveur, le nom du point de terminaison est le nom du serveur. Pour les points de terminaison cloud, le nom du point de terminaison est **Cloud**. La taxonomie du nom est la suivante : 
   
    \<FileNameWithoutExtension\>-\<endpointName\>\[-#\].\<ext\>  

    Par exemple, le premier conflit de CompanyReport.docx deviendra CompanyReport-CentralServer.docx si CentralServer est l’endroit où l’écriture la plus ancienne s’est produite. Le deuxième conflit sera nommé CompanyReport-CentralServer-1.docx. Azure File Sync prend en charge 100 fichiers conflictuels par fichier. Une fois le nombre maximal de fichiers conflictuels atteint, la synchronisation du fichier échoue. Pour qu’elle aboutisse, le nombre de fichiers conflictuels doit être inférieur à 100.

* <a id="afs-storage-redundancy"></a>
  **Le stockage géoredondant est-il pris en charge par Azure File Sync ?**  
    Oui, Azure Files prend en charge le stockage localement redondant (LRS) et le stockage géoredondant (GRS). Si vous lancez un basculement de compte de stockage entre régions appariées à partir d'un compte configuré pour GRS, Microsoft vous recommande de traiter la nouvelle région comme une sauvegarde des données uniquement. Azure File Sync ne démarre pas automatiquement la synchronisation avec la nouvelle région principale. 

* <a id="sizeondisk-versus-size"></a>
  **Pourquoi la propriété *Taille* sur le disque pour un fichier ne correspond-elle pas à la propriété *Taille* après l’utilisation d’Azure File Sync ?**  
  Voir [Introduction à la hiérarchisation cloud](storage-sync-cloud-tiering.md#sizeondisk-versus-size).

* <a id="is-my-file-tiered"></a>
  **Comment puis-je savoir si un fichier a été hiérarchisé ?**  
  Voir [Introduction à la hiérarchisation cloud](storage-sync-cloud-tiering.md#is-my-file-tiered).

* <a id="afs-recall-file"></a>**Un fichier que je souhaite utiliser a été hiérarchisé. Comment puis-je rappeler le fichier sur le disque pour l’utiliser localement ?**  
  Voir [Introduction à la hiérarchisation cloud](storage-sync-cloud-tiering.md#afs-recall-file).

* <a id="afs-force-tiering"></a>
  **Comment faire pour imposer la hiérarchisation d’un fichier ou répertoire ?**  
  Voir [Introduction à la hiérarchisation cloud](storage-sync-cloud-tiering.md#afs-force-tiering).

* <a id="afs-effective-vfs"></a>
  **Comment *l’espace libre du volume* est-il interprété quand il y a plusieurs points de terminaison de serveur sur un volume ?**  
  Voir [Introduction à la hiérarchisation cloud](storage-sync-cloud-tiering.md#afs-effective-vfs).
  
* <a id="afs-tiered-files-tiering-disabled"></a>
  **La hiérarchisation cloud est désactivée. Pourquoi y a-t-il des fichiers hiérarchisés dans l’emplacement du point de terminaison de serveur ?**  
  Voir [Introduction à la hiérarchisation cloud](storage-sync-cloud-tiering.md#afs-tiering-disabled).

* <a id="afs-files-excluded"></a>
  **Quels fichiers ou dossiers sont automatiquement exclus par Azure File Sync ?**  
  Voir [Fichiers ignorés](storage-sync-files-planning.md#files-skipped).

* <a id="afs-os-support"></a>
  **Puis-je utiliser Azure File Sync avec Windows Server 2008 R2, Linux ou mon périphérique de stockage NAS (Network-Attached Storage) ?**  
    À l’heure actuelle, Azure File Sync prend uniquement en charge Windows Server 2019, Windows Server 2016 et Windows Server 2012 R2. À ce stade, nous n’avons pas d’autres plans à partager, mais nous sommes ouverts à l’idée de prendre en charge des plateformes supplémentaires en fonction de la demande des clients. Indiquez-nous les plateformes que vous souhaiteriez voir prises en charge sur le [forum UserVoice consacré à Azure Files](https://feedback.azure.com/forums/217298-storage/category/180670-files).

* <a id="afs-tiered-files-out-of-endpoint"></a>
  **Pourquoi les fichiers hiérarchisés existent-ils en dehors de l’espace de noms du point de terminaison du serveur ?**  
    Avant la version 3 de l’agent Azure File Sync, Azure File Sync bloquait le déplacement des fichiers hiérarchisés à l’extérieur du point de terminaison du serveur, mais sur le même volume que le point de terminaison du serveur. Les opérations de copie, les déplacements de fichiers non hiérarchisés et les déplacements de fichiers hiérarchisés vers d’autres volumes n’étaient pas concernés. Ce comportement était dû au fait que l’Explorateur de fichiers et les autres API Windows supposent de manière implicite que les opérations de déplacement sur un même volume sont des opérations de changement de nom (presque) instantanées. Cela signifie que les déplacements donneront l’impression que l’Explorateur de fichiers ou d’autres méthodes de déplacement (par exemple, la ligne de commande ou PowerShell) ne répondent plus pendant qu’Azure File Sync rappelle les données à partir du cloud. À partir de la [version 3.0.12.0 de l’agent Azure File Sync](storage-files-release-notes.md#supported-versions), Azure File Sync vous permettra de déplacer un fichier hiérarchisé en dehors du point de terminaison du serveur. Nous évitons ainsi les effets négatifs mentionnés précédemment en autorisant l’existence du fichier hiérarchisé sous la forme d’un fichier hiérarchisé en dehors du point de terminaison du serveur, puis en rappelant le fichier en arrière-plan. Cela signifie que les déplacements sur un même volume sont instantanés et que nous faisons tout le travail visant à rappeler le fichier sur le disque une fois le déplacement terminé. 

* <a id="afs-do-not-delete-server-endpoint"></a>
  **Je rencontre un problème avec Azure File Sync sur mon serveur (synchronisation, hiérarchisation cloud, etc.). Dois-je supprimer et recréer le point de terminaison de mon serveur ?**  
    [!INCLUDE [storage-sync-files-remove-server-endpoint](../../../includes/storage-sync-files-remove-server-endpoint.md)]
    
* <a id="afs-resource-move"></a>
  **Puis-je déplacer le service de synchronisation de stockage et/ou le compte de stockage vers un groupe de ressources, un abonnement ou un locataire Azure AD différents ?**  
   Oui. Le service de synchronisation de stockage et/ou le compte de stockage peuvent être déplacés vers un autre groupe de ressources, un autre abonnement ou un autre locataire Azure AD. Une fois le service de synchronisation de stockage ou le compte de stockage déplacé, vous devez donner à l’application Microsoft.StorageSync l’accès au compte de stockage (consultez [Vérifiez qu’Azure File Sync a accès au compte de stockage](./storage-sync-files-troubleshoot.md?tabs=portal1%252cportal#troubleshoot-rbac)).

    > [!Note]  
    > Lors de la création du point de terminaison cloud, le service de synchronisation du stockage et le compte de stockage doivent se trouver dans le même locataire Azure AD. Une fois le point de terminaison cloud créé, le service de synchronisation du stockage et le compte de stockage peuvent être déplacés vers des locataires Azure AD différents.
    
* <a id="afs-ntfs-acls"></a>
  **Azure File Sync conserve-t-il les ACL NTFS de niveau répertoire/fichier en plus des données stockées dans Azure Files ?**

    À compter du 24 février 2020, les listes de contrôle d’accès (ACL, access-control list) nouvelles et existantes hiérarchisées par Azure File Sync sont conservées au format NTFS, et les modifications apportées aux ACL directement dans le partage de fichiers Azure sont synchronisées avec tous les serveurs du groupe de synchronisation. Toutes les modifications apportées aux ACL dans Azure Files sont synchronisées via Azure File Sync. Lorsque vous copiez des données vers Azure Files, veillez à utiliser un outil de copie qui offre la « fidélité » nécessaire pour copier des attributs, des horodateurs et des listes de contrôle d’accès dans un partage de fichiers Azure via le protocole SMB ou REST. Lorsque vous utilisez des outils de copie Azure tels qu’AzCopy, il est important d’utiliser la version la plus récente. Pour une vue d’ensemble des outils de copie Azure qui vous garantissent de pouvoir copier toutes les métadonnées importantes d’un fichier, consultez le [tableau des outils de copie de fichiers](storage-files-migration-overview.md#file-copy-tools).

    Si vous avez activé Sauvegarde Azure sur vos partages de fichiers gérés par synchronisation de fichiers, les ACL de fichier peuvent continuer à être restaurées dans le cadre du workflow de restauration de la sauvegarde. Cela fonctionne pour l’ensemble du partage de fichiers ou pour des fichiers/répertoires individuels.

    Si vous utilisez des captures instantanées dans le cadre de la solution de sauvegarde autogérée pour les partages de fichiers gérés par synchronisation de fichiers, vos ACL peuvent ne pas être restaurées correctement en ACL NTFS si les captures instantanées ont été effectuées avant le 24 février 2020. Si cela se produit, pensez à contacter le support Azure.
    
## <a name="security-authentication-and-access-control"></a>Sécurité, authentification et contrôle d’accès
* <a id="ad-support"></a>
**Azure Files prend-il en charge le contrôle d’accès et l’authentification basée sur l’identité ?**  
    
    Oui, Azure Files prend en charge le contrôle d’accès et l’authentification basée sur l’identité. Vous pouvez choisir l’une des deux façons d’utiliser le contrôle d’accès basé sur l’identité : Active Directory Domain Services en local ou Azure Active Directory Domain Services (Azure AD DS). Active Directory Domain Services (AD DS) en local prend en charge l’authentification à l’aide d’ordinateurs joints au domaine AD, localement ou dans Azure, pour accéder aux partages de fichiers Azure via SMB. L’authentification Azure AD DS sur SMB pour Azure Files tire parti d’Azure AD DS pour permettre aux machines virtuelles Windows jointes à un domaine d’accéder aux partages, répertoires et fichiers à l’aide des informations d’identification Azure AD. Pour plus d’informations, consultez [Vue d’ensemble de la prise en charge de l’authentification basée sur l’identité Azure Files pour l’accès SMB](storage-files-active-directory-overview.md). 

    Azure Files offre deux autres façons de gérer le contrôle d’accès :

    - Vous pouvez utiliser des signatures d’accès partagé (SAP) pour générer des jetons qui ont des autorisations spécifiques et qui sont valides pendant un laps de temps spécifié. Par exemple, vous pouvez générer un jeton offrant un accès en lecture seule à un fichier spécifique, valide pendant 10 minutes. Toute personne possédant le jeton pendant sa période de validité dispose d’un accès en lecture seule à ce fichier pendant 10 minutes. Les clés de signature d’accès partagé ne sont prises en charge que par le biais de l’API REST ou dans les bibliothèques de client. Vous devez monter le partage de fichiers Azure sur SMB à l’aide des clés de compte de stockage.

    - Azure File Sync conserve et réplique toutes les listes de contrôle d’accès discrétionnaire, ou listes DACL (qu’elles soient locales ou basées sur Active Directory) sur tous les points de terminaison de serveur avec lesquels il effectue une synchronisation. 
    
    Pour obtenir une représentation complète de tous les protocoles pris en charge sur les services de stockage Azure, vous pouvez consulter la rubrique [Autorisation de l’accès à Azure Storage](../common/storage-auth.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json). 
    
* <a id="encryption-at-rest"></a>
**Comment puis-je m’assurer que mon partage de fichiers Azure est chiffré au repos ?**  

    Oui. Pour plus d’informations, voir [Azure Storage Service Encryption](../common/storage-service-encryption.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

* <a id="access-via-browser"></a>
**Comment donner accès à un fichier spécifique à l’aide d’un navigateur web ?**  

    Vous pouvez utiliser des signatures d’accès partagé (SAP) pour générer des jetons qui ont des autorisations spécifiques et qui sont valides pendant un laps de temps spécifié. Par exemple, vous pouvez générer un jeton qui procure un accès en lecture seule à un fichier spécifique pendant un laps de temps donné. Toute personne possédant l’URL peut accéder au fichier directement à partir de n’importe quel navigateur web tant que le jeton est valide. Vous pouvez facilement générer une clé de signature d’accès partagé à partir d’une interface utilisateur comme l’Explorateur de stockage.

* <a id="file-level-permissions"></a>
**Est-il possible de spécifier des autorisations en lecture seule ou en écriture seule sur des dossiers au sein du partage ?**  

    Si vous montez le partage de fichiers à l’aide de SMB, vous ne bénéficiez pas du contrôle au niveau des dossiers sur les autorisations. Toutefois, si vous créez une signature d’accès partagé à l’aide de l’API REST ou des bibliothèques clientes, vous pouvez spécifier des autorisations en lecture seule ou en écriture seule sur des dossiers dans le partage.

* <a id="ip-restrictions"></a>
**Puis-je implémenter des restrictions d’adresses IP pour un partage de fichiers Azure ?**  

    Oui. L’accès à votre partage de fichiers Azure peut être limité au niveau du compte de stockage. Pour plus d’informations, consultez [Configurer Pare-feu et réseaux virtuels dans Stockage Azure](../common/storage-network-security.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

* <a id="data-compliance-policies"></a>
**Quelles sont les stratégies de conformité des données prises en charge par Azure Files ?**  

   Azure Files s’exécute sur la même architecture de stockage que d’autres services de stockage dans Stockage Azure. Azure Files applique les mêmes stratégies de conformité des données que celles utilisées dans d’autres services de stockage Azure. Pour plus d’informations sur la conformité des données de stockage Azure, vous pouvez vous référer aux [Offres de conformité du stockage Azure](../common/storage-compliance-offerings.md), et accéder au [Centre de gestion de la confidentialité Microsoft](https://microsoft.com/trustcenter/default.aspx).

* <a id="file-auditing"></a>
**Comment puis-je auditer la consultation et les modifications de fichiers dans Azure Files ?**

  Deux options fournissent la fonctionnalité d’audit pour Azure Files :
  - Si les utilisateurs accèdent directement au partage de fichiers Azure, les [journaux de Stockage Azure (préversion)](../blobs/monitor-blob-storage.md?tabs=azure-powershell#analyzing-logs) permettent de suivre les modifications apportées aux fichiers et les accès utilisateur. Vous pouvez utiliser ces journaux à des fins de dépannage et les demandes sont journalisées de manière optimale.
  - Si des utilisateurs accèdent au partage de fichiers Azure via un serveur Windows Server sur lequel l’agent Azure File Sync est installé, utilisez une [stratégie d’audit](/windows/security/threat-protection/auditing/apply-a-basic-audit-policy-on-a-file-or-folder) ou un produit tiers pour suivre les modifications de fichiers et les accès utilisateur sur le serveur Windows Server. 
   
### <a name="ad-ds--azure-ad-ds-authentication"></a>Authentification AD DS et Azure AD DS
* <a id="ad-support-devices"></a>
**L’authentification Azure Active Directory Domain Services (Azure AD DS) pour Azure Files prend-elle en charge l’accès SMB à l’aide des informations d’identification Azure AD à partir d’appareils joints ou inscrit à Azure AD ?**

    Non, ce scénario n’est pas pris en charge.

* <a id="ad-vm-subscription"></a>
**Puis-je accéder aux partages de fichiers Azure avec des informations d’identification Azure AD à partir d’une machine virtuelle sous un autre abonnement ?**

    Si l’abonnement sous lequel est déployé le partage de fichiers est associé au même locataire Azure AD que le déploiement Azure AD DS auquel la machine virtuelle est jointe, vous pouvez accéder aux partages de fichiers Azure avec les mêmes informations d’identification Azure AD. La limitation est imposée non pas sur l’abonnement, mais sur le locataire Azure AD associé.
    
* <a id="ad-support-subscription"></a>
**Puis-je activer l’authentification Azure AD DS ou AD DS locale pour les partages de fichiers Azure à l’aide d’un locataire Azure AD différent du locataire principal du partage de fichiers Azure ?**

    Non, Azure Files prend uniquement en charge l’intégration d’Azure AD DS ou d’AD DS en local à un locataire Azure AD qui se trouve dans le même abonnement que le partage de fichiers. Un seul abonnement peut être associé à un locataire Azure AD. Cette limitation s’applique à la fois aux méthodes d’authentification Azure AD DS et AD DS en local. Lorsque vous utilisez AD DS en local pour l’authentification, les [informations d’identification AD DS](../../active-directory/hybrid/how-to-connect-install-roadmap.md) doivent être synchronisées avec le compte Azure AD auquel le compte de stockage est associé.

* <a id="ad-linux-vms"></a>
**L’authentification Azure AD DS ou AD DS en local pour les partages de fichiers Azure prend-elles en charge les machines virtuelles Linux ?**

    Non, l’authentification à partir de machines virtuelles Linux n’est pas prise en charge.

* <a id="ad-aad-smb-afs"></a>
**Les partages de fichiers gérés par Azure File Sync prennent-ils en charge l’authentification Azure AD DS ou AD DS en local ?**

    Oui, vous pouvez activer l’authentification Azure AD DS ou AD DS en local sur un partage de fichiers géré par Azure File Sync. Les modifications apportées aux ACL NTFS de répertoire/fichier sur les serveurs de fichiers locaux seront hiérarchisées vers Azure Files et vice versa.

* <a id="ad-aad-smb-files"></a>
**Comment puis-je vérifier si j’ai activé l’authentification AD DS sur mon compte de stockage et récupérer les informations du domaine ?**

    Pour obtenir des instructions, consultez [cette page](./storage-files-identity-ad-ds-enable.md#confirm-the-feature-is-enabled).

* <a id=""></a>
**Est-ce que l’authentification Azure AD pour Azure Files prend en charge les machines virtuelles Linux ?**

    Non, l’authentification à partir de machines virtuelles Linux n’est pas prise en charge.

* <a id="ad-multiple-forest"></a>
**L’authentification AD DS en local pour les partages de fichiers Azure prend-elle en charge l’intégration à un environnement AD DS utilisant plusieurs forêts ?**    

    L’authentification AD DS en local d’Azure Files s’intègre uniquement à la forêt du service de domaine sur lequel le compte de stockage est inscrit. Pour prendre en charge l’authentification à partir d’une autre forêt, votre environnement doit disposer d’une approbation de forêt correctement configurée. La façon dont Azure Files s’inscrit dans AD DS est essentiellement la même que celle d’un serveur de fichiers ordinaire, où il crée une identité (compte d’ouverture de session d’ordinateur ou de service) dans AD DS pour l’authentification. La seule différence est que le nom de principal du service qui est inscrit pour le compte de stockage se termine par « file.core.windows.net », ce qui ne correspond pas au suffixe de domaine. Consultez votre administrateur de domaine pour savoir si une mise à jour de votre stratégie de routage des suffixes est nécessaire pour activer l’authentification de plusieurs forêts en raison du suffixe de domaine différent. Vous trouverez ci-dessous un exemple de configuration de la stratégie de routage des suffixes.
    
    Exemple : Lorsque les utilisateurs du domaine de la forêt A souhaitent accéder à un partage de fichiers avec le compte de stockage inscrit auprès d’un domaine de la forêt B, cela ne fonctionne pas automatiquement car le principal du service du compte de stockage ne présente pas le suffixe correspondant au suffixe d’un domaine de la forêt A. Nous pouvons résoudre ce problème en configurant manuellement une règle de routage des suffixes de la forêt A vers la forêt B pour un suffixe personnalisé « file.core.windows.net ».
    Pour commencer, vous devez ajouter un nouveau suffixe personnalisé à la forêt B. Vérifiez que vous disposez des autorisations d’administration qui conviennent pour modifier la configuration, puis procédez comme suit :   
    1. Connectez-vous à un domaine de machines joint à la forêt B.
    2.  Ouvrez la console « Domaines et approbations Active Directory ».
    3.  Cliquez avec le bouton droit sur « Domaines et approbations Active Directory ».
    4.  Cliquer sur « Propriétés ».
    5.  Cliquez sur « Ajouter ».
    6.  Ajoutez « file.core.windows.net » en tant que suffixes UPN.
    7.  Cliquez sur « Appliquer », puis sur « OK » pour fermer l’Assistant.
    
    Ajoutez ensuite la règle de routage des suffixes à la forêt A à des fins de redirection vers la forêt B.
    1.  Connectez-vous à un domaine de machines joint à la forêt A.
    2.  Ouvrez la console « Domaines et approbations Active Directory ».
    3.  Cliquez avec le bouton droit sur le domaine pour lequel vous souhaitez accéder au partage de fichiers, puis cliquez sur l’onglet « Approbations » et sélectionnez le domaine de la forêt B dans les approbations sortantes. Si vous n’avez pas configuré d’approbation entre les deux forêts, vous devez commencer par le faire.
    4.  Cliquez sur « Propriétés... », puis « Routage des suffixes de noms ».
    5.  Vérifiez si le suffixe « *.file.core.windows.net » s’affiche. Si ce n’est pas le cas, cliquez sur « Actualiser ».
    6.  Sélectionnez « *.file.core.windows.net » et cliquez sur « Activer », puis sur « Appliquer ».

* <a id=""></a>
**Quelles sont les régions disponibles pour l’authentification AD DS Azure Files ?**

    Pour plus d’informations, consultez la [disponibilité régionale AD DS](storage-files-identity-auth-active-directory-enable.md#regional-availability).
    
* <a id="ad-aad-smb-afs"></a>
**Puis-je tirer parti de l’authentification Active Directory (AD) pour Azure Files sur des partages de fichiers gérés par Azure File Sync ?**

    Oui, vous pouvez activer l’authentification AD sur un partage de fichiers géré par Azure File Sync. Les modifications apportées aux ACL NTFS de répertoire/fichier sur les serveurs de fichiers locaux seront hiérarchisées vers Azure Files et vice versa.

* <a id="ad-aad-smb-files"></a>
**Y a-t-il une différence dans la création d’un compte d’ordinateur ou d’un compte de connexion au service pour représenter mon compte de stockage dans Active Directory ?**

    La création d’un [compte d’ordinateur](/windows/security/identity-protection/access-control/active-directory-accounts#manage-default-local-accounts-in-active-directory) (par défaut) ou d’un [compte de connexion au service](/windows/win32/ad/about-service-logon-accounts) ne présente aucune différence quant à la façon dont l’authentification fonctionne avec Azure Files. Vous pouvez librement choisir votre méthode de représentation d’un compte de stockage en tant qu’identité dans votre environnement AD. La valeur DomainAccountType par défaut définie dans l’applet de commande join-AzStorageAccountForAuth est un compte d’ordinateur. Toutefois, la durée de vie du mot de passe configurée dans votre environnement AD peut être différente pour le compte d’ordinateur ou le compte de connexion au service, et vous devez prendre cela en considération pour [mettre à jour le mot de passe de l’identité de votre compte de stockage dans AD](./storage-files-identity-ad-ds-update-password.md).
 
* <a id="ad-support-rest-apis"></a>
**Existe-t-il des API REST pour prendre en charge les ACL Windows de répertoire/fichier Get/Set/Copy ?**

    Oui, nous prenons en charge les API REST qui obtiennent, définissent ou copient des ACL NTFS pour des répertoires ou des fichiers lors de l’utilisation de l’API REST [2019-07-07](/rest/api/storageservices/versioning-for-the-azure-storage-services#version-2019-07-07) (ou version ultérieure). Nous prenons également en charge les ACL Windows persistantes avec les outils REST : [AzCopy v10.4+](https://github.com/Azure/azure-storage-azcopy/releases).

## <a name="network-file-system"></a>Système de gestion de fichiers en réseau

* <a id="when-to-use-nfs"></a>
**Quand dois-je utiliser Azure Files NFS ?**

    Consultez [Partages NFS (préversion)](storage-files-compare-protocols.md#nfs-shares-preview).

* <a id="backup-nfs-data"></a>
**Comment sauvegarder les données stockées dans les partages NFS ?**

    La sauvegarde de vos données dans des partages NFS peut être orchestrée à l’aide d’outils familiers tels que rsync ou de produits de l’un de nos partenaires de sauvegarde tiers. Plusieurs partenaires de sauvegarde, dont [CommVault](https://documentation.commvault.com/commvault/v11/article?p=92634.htm), [Veeam](https://www.veeam.com/blog/?p=123438) et [Veritas](https://players.brightcove.net/4396107486001/default_default/index.html?videoId=6189967101001), faisaient partie de notre préversion initiale et ont étendu leurs solutions pour fonctionner avec SMB 3.0 et NFS 4.1 pour Azure Files.

* <a id="migrate-nfs-data"></a>
**Puis-je migrer des données existantes vers un partage NFS ?**

    Dans une région, vous pouvez utiliser des outils standard comme SCP, rsync ou SSHFS pour déplacer des données. Comme Azure Files NFS est accessible à partir de plusieurs instances de calcul simultanément, vous pouvez améliorer les vitesses de copie avec des chargements parallèles. Pour importer des données non issues d’une région, utilisez un réseau VPN ou une connexion Expressroute pour les monter dans votre système de fichiers à partir de votre centre de données local.

## <a name="on-premises-access"></a>Accès local

* <a id="port-445-blocked"></a>
**Mon fournisseur de services Internet ou mon service informatique bloque le port 445, ce qui provoque l’échec du montage Azure Files. Que dois-je faire ?**

    Vous pouvez en apprendre davantage sur les [différentes manières de contourner un port 445 bloqué ici](./storage-troubleshoot-windows-file-connection-problems.md#cause-1-port-445-is-blocked). Azure Files autorise seulement les connexions SMB 3.0 (avec prise en charge du cryptage) non issues de la région ou du centre de données. Le protocole SMB 3.0 a introduit de nombreuses fonctionnalités de sécurité, notamment le chiffrement du canal, particulièrement sécurisé sur Internet. Toutefois, il est possible que le port 445 ait été bloqué pour des raisons historiques de vulnérabilités trouvées dans des versions antérieures de SMB. Idéalement, le port ne devrait être bloqué que pour le trafic SMB 1.0, qui devrait être désactivé sur tous les clients.

* <a id="expressroute-not-required"></a>
**Dois-je utiliser Azure ExpressRoute pour me connecter à Azure Files ou pour utiliser Azure File Sync localement ?**  

    Non. ExpressRoute n’est pas nécessaire pour accéder à un partage de fichiers Azure. Si vous montez un partage de fichiers Azure directement localement, la seule contrainte est que le port 445 (TCP sortant) soit ouvert pour l’accès à Internet (il s’agit du port sur lequel SMB communique). Si vous utilisez Azure File Sync, seul est nécessaire le port 443 (TCP sortant) pour l’accès HTTPS (aucun protocole SMB requis). Toutefois, vous *pouvez* utiliser ExpressRoute avec l’une ou l’autre de ces options d’accès.

* <a id="mount-locally"></a>
**Comment monter un partage de fichiers Azure sur mon ordinateur local ?**  

    Vous pouvez monter le partage de fichiers à l’aide du protocole SMB si le port 445 (TCP sortant) est ouvert et que votre client prend en charge le protocole SMB 3.0 (par exemple si vous utilisez Windows 10 ou Windows Server 2016). Si le port 445 est bloqué par la stratégie de votre organisation ou par votre fournisseur de services Internet, vous pouvez utiliser Azure File Sync pour accéder à votre partage de fichiers Azure.

## <a name="backup"></a>Sauvegarde
* <a id="backup-share"></a>
**Comment faire pour sauvegarder mon partage de fichiers Azure ?**  
    Vous pouvez utiliser des [instantanés de partage](storage-snapshots-files.md) périodiques pour la protection contre les suppressions accidentelles. Vous pouvez aussi utiliser AzCopy, RoboCopy ou un outil de sauvegarde tiers capable de sauvegarder un partage de fichiers monté. Le service Sauvegarde Azure propose une sauvegarde d’Azure Files. En savoir plus sur la [sauvegarder de partages de fichiers Azure par le service Sauvegarde Azure](../../backup/backup-afs.md).

## <a name="share-snapshots"></a>Instantanés de partage

### <a name="share-snapshots-general"></a>Instantanés de partage : Général
* <a id="what-are-snaphots"></a>
**Qu’est-ce qu’un instantané de partage de fichiers ?**  
    Vous pouvez utiliser des instantanés de partage de fichiers Azure pour créer une version en lecture seule de vos partages de fichiers. Vous pouvez également utiliser Azure Files pour copier une version antérieure de votre contenu sur le même partage, à un autre emplacement dans Azure ou localement en vue d’effectuer d’autres modifications. Pour en savoir plus sur les instantanés de partage, consultez [Vue d’ensemble des instantanés de partage](storage-snapshots-files.md).

* <a id="where-are-snapshots-stored"></a>
**Où sont stockés mes instantanés de partage ?**  
    Les instantanés de partage sont stockés dans le même compte de stockage que celui du partage de fichiers.

* <a id="snapshot-consistency"></a>
**Les instantanés de partage sont-ils cohérents par rapport aux applications ?**  
    Non, les instantanés de partage ne sont pas cohérents par rapport aux applications. L’utilisateur doit vider les écritures à partir de l’application vers le partage avant de capturer l’instantané de partage.

* <a id="snapshot-limits"></a>
**Existe-t-il des limites quant au nombre d’instantanés de partage que je peux utiliser ?**  
    Oui. Azure Files peut conserver un maximum de 200 instantanés de partage. Les instantanés de partage n’étant pas comptabilisés dans le quota de partages, il n’y a pas de limite par partage quant à l’espace total utilisé par tous les instantanés de partage. Les limites de compte de stockage continuent de s’appliquer. Au-delà de 200 instantanés de partage, vous devez supprimer les anciens instantanés pour en créer de nouveaux.

* <a id="snapshot-cost"></a>
**Quel est le coût des instantanés de partage ?**  
    Le coût d’une transaction standard et du stockage standard s’applique à l’instantané. Les instantanés sont incrémentiels par nature. L’instantané de base est le partage lui-même. Tous les instantanés suivants étant incrémentiels, chacun ne stocke que la différence par rapport à l’instantané précédent. Cela signifie que les modifications delta qui apparaîtront dans la facture seront minimales si l’évolution de votre charge de travail est minime. Consultez la [page Tarification](https://azure.microsoft.com/pricing/details/storage/files/) pour obtenir des informations sur la tarification Azure Files standard. Actuellement, la façon d’examiner la taille utilisée par un instantané de partage consiste à comparer la capacité facturée avec la capacité utilisée. Nous travaillons sur des outils pour améliorer la génération de rapports.

* <a id="ntfs-acls-snaphsots"></a>
**Les ACL NTFS sur les répertoires et les fichiers sont-elles conservées dans des instantanés de partage ?**  
    Les ACL NTFS sur les répertoires et les fichiers sont conservées dans des instantanés de partage.

### <a name="create-share-snapshots"></a>Créer des instantanés de partage
* <a id="file-snaphsots"></a>
**Puis-je créer des instantanés de partage de fichiers individuels ?**  
    Les instantanés de partage sont créés au niveau du partage de fichiers. Vous pouvez restaurer des fichiers individuels à partir de l’instantané de partage de fichiers, mais vous ne pouvez pas créer d’instantanés de partage au niveau du fichier. Toutefois, si vous avez capturé un instantané de partage au niveau du partage et que vous souhaitez répertorier les instantanés de partage où un fichier spécifique a changé, vous pouvez effectuer cette opération sous **Versions précédentes** sur un partage monté sur Windows. 
    
    N’hésitez pas à nous indiquer si vous avez besoin d’une fonctionnalité de capture d’instantanés de fichiers sur [Azure Files UserVoice](https://feedback.azure.com/forums/217298-storage/category/180670-files).

* <a id="encrypted-snapshots"></a>
**Puis-je créer des instantanés d’un partage de fichiers chiffré ?**  
    Vous pouvez capturer un instantané de partages de fichiers Azure pour lesquels le chiffrement est activé au repos. Vous pouvez restaurer des fichiers depuis un instantané de partage vers un partage de fichiers chiffré. Si votre partage est chiffré, l’instantané de partage l’est également.

* <a id="geo-redundant-snaphsots"></a>
**Mes instantanés de partage sont-ils géoredondants ?**  
    Les instantanés de partage ont la même redondance que le partage de fichiers Azure auquel ils sont destinés. Si vous avez sélectionné un stockage géoredondant pour votre compte, votre instantané de partage est également stocké de façon redondante dans la région associée.

### <a name="manage-share-snapshots"></a>Gérer les instantanés de partage
* <a id="browse-snapshots-linux"></a>
**Puis-je parcourir mes instantanés de partage à partir de Linux ?**  
    Vous pouvez utiliser Azure CLI pour créer, lister, parcourir et restaurer des instantanés de partage dans Linux.

* <a id="copy-snapshots-to-other-storage-account"></a>
**Puis-je copier les instantanés de partage vers un autre compte de stockage ?**  
    Vous pouvez copier des fichiers des instantanés de partage vers un autre emplacement, mais vous ne pouvez pas copier les instantanés de partage eux-mêmes.

### <a name="restore-data-from-share-snapshots"></a>Restaurer des données à partir d’instantanés de partage
* <a id="promote-share-snapshot"></a>
**Puis-je promouvoir un instantané de partage vers le partage de base ?**  
    Vous pouvez copier des données d’un instantané de partage vers toute autre destination. Vous ne pouvez pas promouvoir un instantané de partage en partage de base.

* <a id="restore-snapshotted-file-to-other-share"></a>
**Puis-je restaurer des données depuis mon instantané de partage vers un autre compte de stockage ?**  
    Oui. Vous pouvez copier les fichiers d’un instantané de partage vers l’emplacement d’origine ou vers un autre emplacement qui comprend le compte de stockage ou un autre compte de stockage, dans la même région ou dans des régions différentes. Vous pouvez également copier des fichiers vers un emplacement local ou vers n’importe quel autre cloud.    
  
### <a name="clean-up-share-snapshots"></a>Nettoyer des instantanés de partage
* <a id="delete-share-keep-snapshots"></a>
**Puis-je supprimer mon partage sans supprimer mes instantanés de partage ?**  
    Si vous avez des instantanés de partage actifs sur votre partage, vous ne pouvez pas le supprimer. Vous pouvez utiliser une API pour supprimer des instantanés de partage, ainsi que le partage. Vous pouvez également supprimer les instantanés de partage et le partage dans le portail Azure.

* <a id="delete-share-with-snapshots"></a>
**Qu’advient-il de mes instantanés de partage si je supprime mon compte de stockage ?**  
    Si vous supprimez votre compte de stockage, les instantanés de partage sont également supprimés.

## <a name="billing-and-pricing"></a>Facturation et tarification
* <a id="vm-file-share-network-traffic"></a>
**Le trafic réseau entre une machine virtuelle Azure et un partage de fichiers Azure est-il considéré comme de la bande passante externe facturée dans le cadre de l’abonnement ?**  
    Si le partage de fichiers et la machine virtuelle sont dans la même région Azure, il n’y a aucun frais supplémentaire pour le trafic entre le partage de fichiers et la machine virtuelle. Si le partage de fichiers et la machine virtuelle se trouvent dans des régions différentes, le trafic entre eux est facturé en tant que bande passante externe.

* <a id="share-snapshot-price"></a>
**Quel est le coût des instantanés de partage ?**  
     Durant la période de préversion, il n’y a aucun frais pour la capacité d’instantané de partage. Les coûts de sortie standard et de transaction liés au stockage s’appliquent. Après la disponibilité générale, la capacité et les transactions sur les instantanés de partage seront facturées aux abonnements.
     
     Les instantanés de partage sont incrémentiels par nature. L’instantané de partage de base est le partage lui-même. Tous les instantanés de partage suivants sont incrémentiels et ne stockent que la différence par rapport à l’instantané de partage précédent. Vous n’êtes facturé que pour le contenu changé. Si vous disposez d’un partage de 100 Go de données, mais que seulement 5 Go ont changé depuis le dernier instantané de partage, l’instantané de partage consomme seulement 5 Go supplémentaires ; ainsi, 105 Go vous sont facturés. Pour plus d’informations sur les frais de sortie standard et de transaction, consultez la [page de tarification](https://azure.microsoft.com/pricing/details/storage/files/).

## <a name="scale-and-performance"></a>Scalabilité et performances
* <a id="files-scale-limits"></a>
**Quelles sont les limites d’extensibilité d’Azure Files ?**  
    Pour plus d’informations sur les objectifs de scalabilité et de performances d’Azure Files, consultez [Objectifs de performance et d’extensibilité d’Azure Files](storage-files-scale-targets.md).

* <a id="need-larger-share"></a>
**Quelles sont les tailles disponibles pour les partages de fichiers Azure ?**  
    Les tailles des partages de fichiers Azure (Premium et standard) ne peuvent pas dépasser 100 Tio. Consultez la section relative à [l’intégration de partages de fichiers plus grands (niveau standard)](storage-files-planning.md#enable-standard-file-shares-to-span-up-to-100-tib) du guide de planification pour savoir comment intégrer des partages de fichiers plus grands au niveau standard.

* <a id="lfs-performance-impact"></a>
**L’extension du quota de partage de fichiers a-t-elle un impact sur mes charges de travail ou Azure File Sync ?**
    
    Non. Non, elle n’a d’impact ni sur vos charges de travail, ni sur Azure File Sync.

* <a id="open-handles-quota"></a>
**Combien de clients peuvent accéder simultanément au même fichier ?**    
    Il existe un quota de 2 000 handles ouverts sur un seul fichier. Quand vous avez 2 000 handles ouverts, un message d’erreur s’affiche pour signaler que le quota est atteint.

* <a id="zip-slow-performance"></a>
**Les performances sont lentes quand je décompresse des fichiers dans Azure Files. Que dois-je faire ?**  
    Pour transférer un grand nombre de fichiers dans Azure Files, nous vous recommandons d’utiliser AzCopy (pour Windows, en préversion pour Linux/Unix) ou Azure Powershell. Ces outils ont été optimisés pour le transfert réseau.

* <a id="slow-perf-windows-81-2012r2"></a>
**Pourquoi les performances sont-elles lentes depuis que j’ai monté mon partage de fichiers Azure sur Windows Server 2012 R2 ou Windows 8.1 ?**  
    Il existe un problème connu lors du montage d’un partage de fichiers Azure sur Windows Server 2012 R2 et Windows 8.1. Ce problème a été corrigé dans la mise à jour cumulative d’avril 2014 pour Windows 8.1 et Windows Server 2012 R2. Pour des performances optimales, vérifiez que ce correctif est appliqué sur toutes les instances de Windows Server 2012 R2 et Windows 8.1. (Vous devez toujours recevoir les correctifs Windows par le biais de Windows Update). Pour plus d’informations, consultez l’article de la Base de connaissances associé [Ralentissement des performances lors de l’accès à Azure Files à partir de Windows 8.1 ou Server 2012 R2](https://support.microsoft.com/kb/3114025).

## <a name="features-and-interoperability-with-other-services"></a>Fonctionnalités et interopérabilité avec d’autres services
* <a id="cluster-witness"></a>
**Puis-je utiliser mon partage de fichiers Azure en tant que *témoin de partage de fichiers* pour mon cluster de basculement Windows Server ?**  
    Actuellement, cette configuration n’est pas prise en charge pour un partage de fichiers Azure. Pour plus d’informations sur la façon d’obtenir cette configuration pour le stockage Blob Azure, consultez [Déployer un témoin cloud pour un cluster de basculement](/windows-server/failover-clustering/deploy-cloud-witness).

* <a id="containers"></a>
**Puis-je monter un partage de fichiers Azure dans une instance Azure Container ?**  
    Oui, les partages de fichiers Azure sont une bonne option quand vous souhaitez conserver les informations au-delà de la durée de vie d’une instance de conteneur. Pour plus d’informations, consultez [Montage d’un partage de fichiers Azure avec Azure Container Instances](../../container-instances/container-instances-volume-azure-files.md).

* <a id="rest-rename"></a>
**Existe-t-il une opération de renommage dans l’API REST ?**  
    Pas pour l'instant.

* <a id="nested-shares"></a>
**Puis-je configurer des partages imbriqués ? Autrement dit, un partage sous un partage ?**  
    Non. Le partage de fichiers *est* le pilote virtuel que vous pouvez monter, et les partages imbriqués ne sont donc pas pris en charge.

* <a id="ibm-mq"></a>
**Comment faire pour utiliser Azure Files avec IBM MQ ?**  
    IBM a publié un document qui aide les clients IBM MQ à configurer Azure Files avec son service IBM. Pour plus d’informations, consultez [How to set up an IBM MQ multi-instance queue manager with Microsoft Azure Files service](https://github.com/ibm-messaging/mq-azure/wiki/How-to-setup-IBM-MQ-Multi-instance-queue-manager-with-Microsoft-Azure-File-Service)(Guide pratique pour configurer le gestionnaire de file d’attente multi-instance IBM MQ avec le service Microsoft Azure Files).

## <a name="see-also"></a>Voir aussi
* [Résoudre les problèmes d’Azure Files dans Windows](storage-troubleshoot-windows-file-connection-problems.md)
* [Résoudre les problèmes d’Azure Files dans Linux](storage-troubleshoot-linux-file-connection-problems.md)
* [Résoudre les problèmes d’Azure File Sync](storage-sync-files-troubleshoot.md)