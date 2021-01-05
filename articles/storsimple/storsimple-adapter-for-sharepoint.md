---
title: Installer un adaptateur StorSimple pour SharePoint | Microsoft Docs
description: Décrit comment installer et configurer ou supprimer StorSimple Adapter pour SharePoint dans une batterie de serveurs SharePoint.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 36c20b75-f2e5-4184-a6b5-9c5e618f79b2
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 06/06/2017
ms.author: alkohli
ms.openlocfilehash: 639efab3a9eaef400b3fbe3d0b134d02f458b865
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966290"
---
# <a name="install-and-configure-the-storsimple-adapter-for-sharepoint"></a>Installation et configuration de StorSimple Adapter for SharePoint
## <a name="overview"></a>Vue d’ensemble
StorSimple Adapter for SharePoint est un composant qui vous permet de fournir un stockage Microsoft Azure StorSimple flexible et une protection des données aux batteries de serveurs SharePoint. Vous pouvez utiliser l’adaptateur pour déplacer le contenu d’un objet Binary Large Object (BLOB) des bases de données de contenu SQL Server vers le dispositif de stockage cloud hybride Microsoft Azure StorSimple.

StorSimple Adapter for SharePoint fonctionne comme un fournisseur Remote BLOB Storage (RBS) et utilise la fonctionnalité SQL Server Remote BLOB Storage pour stocker le contenu SharePoint non structuré (sous la forme d’objets blob) sur un serveur de fichiers appuyé par un appareil StorSimple.

> [!NOTE]
> StorSimple Adapter for SharePoint prend en charge SharePoint Server 2010 Remote BLOB Storage (RBS). Il ne prend pas en charge SharePoint Server 2010 External BLOB Storage (EBS).


* Pour télécharger StorSimple Adapter pour SharePoint, accédez à [StorSimple Adapter pour SharePoint][1] dans le Centre de téléchargement Microsoft.
* Pour plus d’informations sur la planification et les limitations de RBS, accédez à [Choix de l’utilisation de RBS dans SharePoint 2013][2] ou [Planification de RBS (SharePoint Server 2010)][3].

Le reste de cette présentation décrit brièvement le rôle de StorSimple Adapter for SharePoint et les limites de capacité et de performances de SharePoint que vous devez connaître avant d'installer et de configurer l'adaptateur. Après avoir pris connaissance de ces informations, consultez la rubrique [Installation de StorSimple Adapter for SharePoint](#storsimple-adapter-for-sharepoint-installation) pour commencer la configuration de l'adaptateur.

### <a name="storsimple-adapter-for-sharepoint-benefits"></a>Avantages de StorSimple Adapter for SharePoint
Dans un site SharePoint, le contenu est stocké comme des données BLOB non structurées dans une ou plusieurs bases de données de contenu. Par défaut, ces bases de données sont hébergées sur des ordinateurs qui exécutent SQL Server et se trouvent dans la batterie de serveurs SharePoint. Les objets blob peuvent rapidement augmenter en taille, consommant ainsi une grande quantité de stockage local. Pour cette raison, vous souhaiterez sans doute trouver une autre solution de stockage moins onéreuse. SQL Server fournit une technologie appelée Remote Blob Storage (RBS) qui vous permet de stocker le contenu de l'objet blob dans le système de fichiers, en dehors de la base de données SQL Server. Avec RBS, des objets blob peuvent résider dans le système de fichiers sur l'ordinateur qui exécute SQL Server, ou être stockés dans le système de fichiers sur un autre serveur.

Avec RBS, vous devez utiliser un fournisseur RBS comme StorSimple Adapter for SharePoint pour activer RBS dans SharePoint. StorSimple Adapter for SharePoint fonctionne avec RBS, vous permettant de déplacer des objets blob vers un serveur soutenu par le système Microsoft Azure StorSimple. Microsoft Azure StorSimple stocke ensuite les données d’objets blob localement ou dans le cloud, selon leur utilisation. Les objets blob très actifs (généralement appelés données à chaud ou de niveau 1) résident localement. Les données moins actives et les données archivées résident dans le cloud. Après avoir activé RBS sur une base de données de contenu, tout nouveau contenu bloc créé dans SharePoint est stocké sur l’appareil StorSimple et non dans la base de données de contenu.

L’implémentation Microsoft Azure StorSimple de RBS offre les avantages suivants :

* En déplaçant le contenu blob vers un serveur distinct, vous pouvez réduire la charge de requête sur SQL Server, ce qui peut améliorer la réactivité du serveur SQL. 
* Azure StorSimple utilise la déduplication et la compression pour réduire la taille des données.
* Azure StorSimple assure la protection des données sous la forme d’instantanés locaux et sur le cloud. En outre, si vous placez la base de données elle-même sur l’appareil StorSimple, vous pouvez sauvegarder ensemble la base de données de contenu et les objets blob afin de limiter les risques de défaillance. (Le déplacement de la base de données de contenu sur l’appareil est uniquement pris en charge pour la série StorSimple 8000. Aucune prise en charge pour les séries 5000 ou 7000.)
* Azure StorSimple inclut des fonctionnalités de récupération d'urgence, notamment le basculement, la récupération de fichier et de volume (y compris une récupération test) et la restauration rapide des données.
* Vous pouvez utiliser un logiciel de récupération de données comme Kroll Ontrack PowerControls avec des instantanés StorSimple de données blob pour effectuer une récupération au niveau des éléments du contenu SharePoint. (Ce logiciel de récupération des données est vendu séparement).
* StorSimple Adapter for SharePoint se connecte au portail d'administration centrale de SharePoint, ce qui vous permet de gérer votre solution SharePoint à partir d'un emplacement central.

Le déplacement du contenu blob vers le système de fichiers peut entraîner d’autres réductions de coût et avantages. Par exemple, RBS peut réduire les besoins en stockage de niveau 1 coûteux et, puisqu’il réduit la base de données de contenu, réduire le nombre de bases de données requises dans la batterie de serveurs SharePoint. Toutefois, d'autres facteurs comme les limites de taille de base de données et la quantité de contenu non RBS, peuvent également affecter les exigences de stockage. Pour plus d’informations sur les coûts et les avantages de RBS, consultez [Planification de RBS (SharePoint Foundation 2010)][4] et [Choix de l’utilisation de RBS dans SharePoint 2013][5].

### <a name="capacity-and-performance-limits"></a>Limites de capacité et de performances
Avant d'envisager l'utilisation de RBS dans votre solution SharePoint, vous devez être conscient des limites de performances et de capacité de SharePoint Server 2010 et de SharePoint Server 2013, et comment ces limites se situent par rapport à des performances acceptables. Pour plus d'informations, consultez la rubrique [Limites des logiciels et limites pour SharePoint 2013](/SharePoint/install/software-boundaries-and-limits-0).

Avant de configurer RBS, passez en revue les éléments suivants :

* Assurez-vous que la taille totale du contenu (la taille d'une base de données de contenu plus celle de tous les objets blob externalisés associés) ne dépasse pas la limite de taille RBS prise en charge par SharePoint. Cette limite est de 200 Go. 
  
    **Mesure de la taille de la base de données de contenu et des objets blob**
  
  1. Exécutez cette requête sur le serveur web frontal d’administration centrale. Démarrez SharePoint Management Shell, puis entrez la commande Windows PowerShell suivante pour obtenir la taille des bases de données de contenu :
     
     `Get-SPContentDatabase | Select-Object -ExpandProperty DiskSizeRequired`
     
      Cette étape permet d’obtenir la taille de la base de données de contenu sur le disque.
  2. Exécutez une des requêtes SQL suivantes dans SQL Management Studio sur la zone du serveur SQL pour chaque base de données de contenu, puis ajoutez le résultat au nombre obtenu à l’étape 1.
     
     Dans les bases de données de contenu SharePoint 2013, entrez :
     
     `SELECT SUM([Size]) FROM [ContentDatabaseName].[dbo].[DocStreams] WHERE [Content] IS NULL`
     
     Dans les bases de données de contenu SharePoint 2010, entrez :
     
     `SELECT SUM([Size]) FROM [ContentDatabaseName].[dbo].[AllDocs] WHERE [Content] IS NULL`
     
     Cette étape permet d’obtenir la taille des objets blob qui ont été externalisés.
* Nous vous recommandons de stocker tous les objets blob et tout le contenu des bases de données localement sur l’appareil StorSimple. L’appareil StorSimple est un cluster à deux nœuds garantissant une haute disponibilité. Le placement des bases de données de contenu et des objets blob sur l’appareil StorSimple offre une haute disponibilité.
  
    Utilisez les meilleures pratiques de migration SQL Server pour déplacer la base de données de contenu vers l’appareil StorSimple. Déplacez la base de données uniquement après que tout le contenu blob de la base de données a été transféré vers le partage de fichiers via RBS. Si vous décidez de déplacer la base de données de contenu vers l’appareil StorSimple, nous vous recommandons de configurer le stockage de base de données de contenu sur l’appareil en tant que volume principal.
* Dans Microsoft Azure StorSimple, il n'existe aucun moyen de garantir que le contenu stocké localement sur l’appareil StorSimple ne sera pas stocké sur plusieurs niveaux dans le stockage en cloud Microsoft Azure si vous utilisez des volumes hiérarchisés. Par conséquent, nous recommandons d’utiliser des volumes StorSimple épinglés localement conjointement avec SharePoint RBS. Cela garantit que tout le contenu de l’objet BLOB est conservé localement sur l’appareil StorSimple et n’est pas déplacé vers Microsoft Azure.
* Si vous ne stockez pas les bases de données de contenu sur l’appareil StorSimple, utilisez les meilleures pratiques à haute disponibilité SQL Server qui prennent en charge RBS. La gestion de clusters SQL Server prend en charge RBS, ce qui n’est pas le cas de la mise en miroir SQL Server. 

> [!WARNING]
> Si vous n’avez pas activé RBS, il est déconseillé de déplacer la base de données de contenu sur l’appareil StorSimple. Il s'agit d'une configuration non testée.

## <a name="storsimple-adapter-for-sharepoint-installation"></a>Installation de StorSimple Adapter for SharePoint
Avant d'installer StorSimple Adapter for SharePoint, vous devez configurer l’appareil StorSimple et vous assurer que la batterie de serveurs SharePoint et l'instanciation SQL Server répondent à toutes les conditions préalables. Ce didacticiel décrit la configuration requise ainsi que les procédures d’installation et de mise à niveau de StorSimple Adapter for SharePoint.

## <a name="configure-prerequisites"></a>Configuration préalable requise
Avant d'installer StorSimple Adapter for SharePoint, assurez-vous que l’appareil StorSimple, la batterie de serveurs SharePoint et l'instanciation SQL Server répondent aux conditions préalables suivantes.

### <a name="system-requirements"></a>Configuration système requise
StorSimple Adapter for SharePoint fonctionne avec les matériels et logiciels suivants :

* Système d’exploitation pris en charge – Windows Server 2008 R2 SP1, Windows Server 2012 ou Windows Server 2012 R2
* Versions SharePoint prises en charge – SharePoint Server 2010 ou SharePoint Server 2013
* Versions SQL Server prises en charge – SQL Server 2008 Enterprise Edition, SQL Server 2008 R2 Enterprise Edition ou SQL Server 2012 Enterprise Edition
* Appareils StorSimple pris en charge – StorSimple série 8000, StorSimple série 7000 ou StorSimple série 5000.

### <a name="storsimple-device-configuration-prerequisites"></a>Conditions préalables à la configuration de l’appareil StorSimple
L’appareil StorSimple est un appareil en mode bloc et nécessite en tant quel tel un serveur de fichiers sur lequel les données peuvent être hébergées. Nous vous recommandons d'utiliser un serveur distinct plutôt qu’un serveur existant d’une batterie de serveurs SharePoint. Ce serveur de fichiers doit figurer sur le même réseau local (LAN) que l'ordinateur SQL Server qui héberge les bases de données de contenu.

> [!TIP]
> * Si vous configurez votre batterie de serveurs SharePoint pour garantir une haute disponibilité, vous devez également déployer le serveur de fichiers pour une haute disponibilité.
> * Si vous ne stockez pas les bases de données de contenu sur l’appareil StorSimple, utilisez les meilleures pratiques à haute disponibilité qui prennent en charge RBS. La gestion de clusters SQL Server prend en charge RBS, ce qui n’est pas le cas de la mise en miroir SQL Server. 


Assurez-vous que votre appareil StorSimple est correctement configuré et que les volumes appropriés pour prendre en charge votre déploiement SharePoint sont configurés et accessibles depuis votre ordinateur SQL Server. Consultez la rubrique [Déploiement de votre appareil StorSimple local](storsimple-8000-deployment-walkthrough-u2.md) si vous n'avez pas encore déployé et configuré votre appareil StorSimple. Notez l'adresse IP de l’appareil StorSimple ; vous en aurez besoin lors de l’installation de StorSimple Adapter for SharePoint.

En outre, assurez-vous que le volume à utiliser pour l'externalisation des objets blob remplit les conditions suivantes :

* Le volume doit être formaté avec une taille d'unité d'allocation de 64 Ko.
* Votre serveur web frontal (WFE) et les serveurs d'applications doivent être en mesure d'accéder au volume via un chemin UNC (Universal Naming Convention).
* La batterie de serveurs SharePoint doit être configurée pour écrire sur le volume.

> [!NOTE]
> Après avoir installé et configuré l’adaptateur, l’intégralité de l’externalisation blob doit passer par l’appareil StorSimple (l’appareil présentera les volumes à SQL Server et gérera les niveaux de stockage). Vous ne pouvez pas utiliser d'autres cibles pour l'externalisation blob.


Si vous prévoyez d'utiliser StorSimple Snapshot Manager pour prendre des instantanés des données blob et des bases de données, veillez à installer StorSimple Snapshot Manager sur le serveur de base de données afin qu'il puisse utiliser le service SQL Writer pour implémenter le service Windows Volume Shadow Copy (VSS).

> [!IMPORTANT]
> StorSimple Snapshot Manager ne prend pas en charge le service SharePoint VSS Writer et ne peut pas prendre des instantanés de données SharePoint cohérents avec l’application. Dans un scénario SharePoint, StorSimple Snapshot Manager fournit uniquement des sauvegardes cohérentes après incident.


## <a name="sharepoint-farm-configuration-prerequisites"></a>Conditions préalables à la configuration d’une batterie de serveurs SharePoint
Assurez-vous que votre batterie de serveurs SharePoint est correctement configurée, comme suit :

* Vérifiez que votre batterie de serveurs SharePoint est dans un état sain, puis vérifiez les éléments suivants :
* Tous les serveurs WFE et d’application SharePoint enregistrés dans la batterie sont en cours d'exécution et accessibles à partir du serveur sur lequel vous installerez StorSimple Adapter for SharePoint.
* Le service SharePoint Timer (SPTimerV3 ou SPTimerV4) s'exécute sur chaque serveur WFE et serveur d'applications.
* Le service SharePoint Timer et le pool d'applications IIS sur lequel le site d'Administration centrale de SharePoint s'exécute disposent des privilèges d'administration.
* Assurez-vous que Internet Explorer Enhanced Security Context (IE ESC) est désactivé. Suivez ces étapes pour désactiver IE ESC :
  
  1. Fermez toutes les instances d'Internet Explorer.
  2. Démarrez le Gestionnaire de serveur.
  3. Dans le volet gauche, cliquez sur **Serveur local**.
  4. Dans le volet droit, en regard de **Configuration de sécurité renforcée d'Internet Explorer**, cliquez sur **Activé**.
  5. Sous **Administrateurs**, cliquez sur **Désactivé**.
  6. Cliquez sur **OK**.

## <a name="remote-blob-storage-rbs-prerequisites"></a>Conditions préalables pour Remote BLOB Storage (RBS)
Assurez-vous que vous utilisez une version SQL Server prise en charge. Seules les versions suivantes sont prises en charge et peuvent exécuter RBS :

* SQL Server 2008 Enterprise Edition
* SQL Server 2008 R2 Enterprise Edition
* SQL Server 2012 Enterprise Edition

Les objets blob peuvent être externalisés uniquement sur les volumes que l’appareil StorSimple présente à SQL Server. Aucune autre cible n'est prise en charge pour l’externalisation blob.

Lorsque vous avez effectué toutes les étapes de configuration requises, consultez la rubrique [Installation de StorSimple Adapter for SharePoint](#install-the-storsimple-adapter-for-sharepoint).

## <a name="install-the-storsimple-adapter-for-sharepoint"></a>Installation de StorSimple Adapter for SharePoint
Utilisez les étapes suivantes pour installer StorSimple Adapter for SharePoint. Si vous réinstallez le logiciel, consultez la rubrique [Mise à niveau ou réinstallation de StorSimple Adapter for SharePoint](#upgrade-or-reinstall-the-storsimple-adapter-for-sharepoint). Le temps nécessaire à l'installation varie selon le nombre total de bases de données SharePoint dans votre batterie de serveurs SharePoint.

[!INCLUDE [storsimple-install-sharepoint-adapter](../../includes/storsimple-install-sharepoint-adapter.md)]

## <a name="configure-rbs"></a>Configuration de RBS
Après avoir installé StorSimple Adapter for SharePoint, configurez RBS comme décrit dans la procédure suivante.

> [!TIP]
> StorSimple Adapter for SharePoint s'intègre à la page Administration centrale de SharePoint, ce qui permet d’activer ou de désactiver RBS sur chaque base de données présente dans la batterie de serveurs SharePoint. Cependant, l'activation ou la désactivation de RBS sur la base de données de contenu entraîne une réinitialisation IIS qui, selon la configuration de votre batterie de serveurs, peut interrompre momentanément la disponibilité du serveur web frontal (WFE) SharePoint. (Des facteurs tels que l'utilisation d'un équilibreur de charge frontal, la charge de travail actuelle du serveur, etc., peuvent limiter voire empêcher cette indisponibilité). Pour protéger les utilisateurs d'une telle interruption, nous vous recommandons d'activer ou de désactiver RBS uniquement lors d’une fenêtre de maintenance planifiée.


[!INCLUDE [storsimple-sharepoint-adapter-configure-rbs](../../includes/storsimple-sharepoint-adapter-configure-rbs.md)]

## <a name="configure-garbage-collection"></a>Configuration du nettoyage de mémoire
Lorsque des objets sont supprimés d'un site SharePoint, ils ne sont pas automatiquement supprimés du volume de stockage RBS. Au lieu de cela, un programme de maintenance en arrière-plan asynchrone supprime les objets blob orphelins du magasin de fichiers. Les administrateurs système peuvent planifier ce processus pour une exécution périodique ou le démarrer chaque fois que cela est nécessaire.

Ce programme de maintenance (Microsoft.Data.SqlRemoteBlobs.Maintainer.exe) est automatiquement installé sur tous les serveurs WFE SharePoint et serveurs d'applications lorsque vous activez RBS. Le programme est installé à l'emplacement suivant : *disque de démarrage*:\Program Files\Microsoft SQL Remote Blob Storage 10.50\Maintainer\

Pour plus d’informations sur la configuration et l’utilisation du programme de maintenance, consultez [Maintenance de RBS dans SharePoint Server 2013][8].

> [!IMPORTANT]
> Le programme de maintenance RBS consomme beaucoup de ressources. Vous devez planifier son exécution uniquement pendant les périodes de faible activité au niveau de la batterie de serveurs SharePoint.


### <a name="delete-orphaned-blobs-immediately"></a>Suppression immédiate des objets blob orphelins
Si vous avez besoin de supprimer immédiatement des objets blob orphelins, vous pouvez utiliser les instructions suivantes. Notez que ces instructions sont un exemple de la procédure dans un environnement SharePoint 2013 avec les composants suivants :

* Le nom de la base de données de contenu est WSS_Content.
* Le nom du serveur SQL est SHRPT13-SQL12\SHRPT13.
* Le nom de l'application web est SharePoint – 80.

[!INCLUDE [storsimple-sharepoint-adapter-garbage-collection](../../includes/storsimple-sharepoint-adapter-garbage-collection.md)]

## <a name="upgrade-or-reinstall-the-storsimple-adapter-for-sharepoint"></a>Mise à niveau ou réinstallation de StorSimple Adapter for SharePoint
Utilisez la procédure suivante pour mettre à niveau le serveur SharePoint puis réinstaller StorSimple Adapter for SharePoint ou pour simplement mettre à niveau ou réinstaller l'adaptateur dans une batterie de serveurs SharePoint existante.

> [!IMPORTANT]
> Passez en revue les informations suivantes avant d'essayer de mettre à niveau votre logiciel SharePoint et/ou mettre à niveau ou réinstaller StorSimple Adapter for SharePoint :
> 
> * Les fichiers précédemment déplacés vers le stockage externe via RBS ne sont pas disponibles, jusqu'à ce que la réinstallation soit terminée et la fonctionnalité RBS activée. Pour limiter l'impact sur l'utilisateur, effectuez la mise à niveau ou la réinstallation pendant une fenêtre de maintenance planifiée.
> * Le temps nécessaire à la mise à niveau/installation peut varier selon le nombre total de bases de données SharePoint dans la batterie de serveurs SharePoint.
> * Une fois la mise à niveau/réinstallation terminée, vous devez activer RBS pour les bases de données de contenu. Consultez [Configuration de RBS](#configure-rbs) pour plus d'informations.
> * Si vous configurez RBS pour une batterie de serveurs SharePoint avec un très grand nombre de bases de données (plus de 200), la page de l' **Administration centrale de SharePoint** peut expirer. Si cela se produit, actualisez la page. Cela n’affecte pas le processus de configuration.


[!INCLUDE [storsimple-upgrade-sharepoint-adapter](../../includes/storsimple-upgrade-sharepoint-adapter.md)]

## <a name="storsimple-adapter-for-sharepoint-removal"></a>Suppression de StorSimple Adapter pour SharePoint
Les procédures suivantes décrivent comment déplacer les objets BLOB vers les bases de données de contenu SQL Server, puis désinstaller StorSimple Adapter pour SharePoint. 

> [!IMPORTANT]
> Vous devez renvoyer les objets BLOB vers les bases de données de contenu avant de désinstaller le logiciel de l'adaptateur.


### <a name="before-you-begin"></a>Avant de commencer
Rassemblez les informations suivantes avant de déplacer les données vers les bases de données de contenu SQL Server et de commencer le processus de suppression de l'adaptateur :

* Les noms de toutes les bases de données pour lesquelles RBS est activé
* Le chemin d'accès UNC du magasin d'objets BLOB configuré

### <a name="move-the-blobs-back-to-the-content-databases"></a>Déplacer les objets BLOB vers les bases de données de contenu
Avant de désinstaller le logiciel StorSimple Adapter pour SharePoint, vous devez migrer tous les objets BLOB externalisés vers les bases de données de contenu SQL Server. Si vous essayez de désinstaller StorSimple Adapter pour SharePoint avant de déplacer tous les objets BLOB vers les bases de données de contenu, le message d'avertissement suivant s'affiche.

![Message d'avertissement](./media/storsimple-adapter-for-sharepoint/sasp1.png)

#### <a name="to-move-the-blobs-back-to-the-content-databases"></a>Pour déplacer les objets BLOB vers les bases de données de contenu
1. Téléchargez tous les objets externalisés.
2. Ouvrez la page **Administration centrale de SharePoint** et accédez à **Paramètres du système**.
3. Sous **Azure StorSimple**, cliquez sur **Configuration de l'adaptateur StorSimple**.
4. Sur la page **Configuration de l'adaptateur StorSimple**, cliquez sur le bouton **Désactiver** sous chaque base de données de contenu que vous souhaitez supprimer du stockage d'objets BLOB externe. 
5. Supprimez les objets de SharePoint et téléchargez-les à nouveau.

Vous pouvez aussi utiliser l’applet de commande Microsoft `RBS Migrate()` PowerShell fournie avec SharePoint. Pour plus d'informations, consultez [Migration du contenu vers ou à partir de RBS](/previous-versions/office/sharepoint-foundation-2010/ff628255(v=office.14)).

Après avoir redéplacé les objets BLOB vers la base de données de contenu, passez à l'étape suivante : [Désinstaller l’adaptateur](#uninstall-the-adapter).

### <a name="uninstall-the-adapter"></a>Désinstallation de l'adaptateur
Après avoir déplacé les objets BLOB vers les bases de données de contenu SQL Server, utilisez l'une des options suivantes pour désinstaller StorSimple Adapter pour SharePoint.

#### <a name="to-use-the-installation-program-to-uninstall-the-adapter"></a>Pour utiliser le programme d'installation pour désinstaller l'adaptateur
1. Utilisez un compte avec des privilèges d'administrateur pour ouvrir une session sur le serveur web frontal.
2. Double-cliquez sur le programme d'installation de StorSimple Adapter pour SharePoint. L'Assistant d'installation démarre.
   
    ![Assistant d'installation](./media/storsimple-adapter-for-sharepoint/sasp2.png)
3. Cliquez sur **Suivant**. La page suivante apparaît.
   
    ![Page de suppression de l'Assistant d'installation](./media/storsimple-adapter-for-sharepoint/sasp3.png)
4. Cliquez sur **Supprimer** pour sélectionner le processus de suppression. La page suivante apparaît.
   
    ![Page de confirmation de l'Assistant d'installation](./media/storsimple-adapter-for-sharepoint/sasp4.png)
5. Cliquez sur **Supprimer** pour confirmer la suppression. La page de progression suivante s'affiche.
   
    ![Page de progression de l'Assistant d'installation](./media/storsimple-adapter-for-sharepoint/sasp5.png)
6. Lorsque la suppression est terminée, la page de fin s'affiche. Cliquez sur **Terminer** pour fermer l’Assistant Installation.

#### <a name="to-use-the-control-panel-to-uninstall-the-adapter"></a>Pour utiliser le Panneau de configuration pour désinstaller l'adaptateur
1. Ouvrez le Panneau de configuration, puis cliquez sur **Programmes et fonctionnalités**.
2. Sélectionnez **StorSimple Adapter pour SharePoint**, puis cliquez sur **Désinstaller**.

## <a name="next-steps"></a>Étapes suivantes
[En savoir plus sur StorSimple](storsimple-overview.md).

<!--Reference links-->
[1]: https://www.microsoft.com/download/details.aspx?id=44073
[2]: /SharePoint/administration/rbs-planning
[3]: /previous-versions/office/sharepoint-server-2010/ff628583(v=office.14)
[4]: /previous-versions/office/sharepoint-foundation-2010/ff628569(v=office.14)
[5]: /SharePoint/administration/rbs-planning
[8]: /SharePoint/administration/maintain-rbs