---
title: Protection de la récupération de l’état du système et complète
description: Utilisez le serveur de sauvegarde Azure pour sauvegarder l’état de votre système et fournir une protection de récupération complète.
ms.topic: conceptual
ms.date: 05/15/2017
ms.openlocfilehash: c5096158ca0e76ca03577347d8dd3e1419a33ca0
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021620"
---
# <a name="back-up-system-state-and-restore-to-bare-metal-by-using-azure-backup-server"></a>Sauvegarder l’état du système et effectuer une récupération complète à l’aide du serveur de sauvegarde Azure

Le Serveur de sauvegarde Azure sauvegarde l’état de votre système et effectue une récupération complète de celui-ci.

* **Sauvegarde de l’état du système** : Sauvegarde les fichiers du système d’exploitation. Cette sauvegarde vous permet d’effectuer une récupération au démarrage de l’ordinateur, mais les fichiers système et le Registre sont perdus. Une sauvegarde de l’état du système comprend les éléments suivants :
  * Membre de domaine : fichiers de démarrage, base de données d’inscription de classe COM+, Registre
  * Contrôleur de domaine : Windows Server Active Directory (NTDS), fichiers de démarrage, base de données d’inscription de classe COM+, Registre, volume système (SYSVOL)
  * Ordinateur qui exécute les services de cluster : Métadonnées du serveur de cluster
  * Ordinateur exécutant les services de certificats : Données du certificat
* **Sauvegarde complète** : sauvegarde les fichiers du système d’exploitation et toutes les données sur les volumes critiques, sauf les données utilisateur. Par définition, une sauvegarde complète inclut une sauvegarde de l’état du système. Elle offre une protection quand un ordinateur ne démarre pas et que vous devez récupérer tous les éléments nécessaires.

Le tableau suivant résume ce que vous pouvez sauvegarder et récupérer. Pour plus d’informations sur les versions d’application que l’état du système et une récupération complète peuvent protéger, consultez [Qu’est-ce que le serveur de sauvegarde Azure ?](backup-mabs-protection-matrix.md).

|Sauvegarde|Problème|Récupère à partir de la sauvegarde effectuée par le Serveur de sauvegarde Azure|Récupère à partir de la sauvegarde de l’état du système|Récupération complète|
|----------|---------|---------------------------|------------------------------------|-------|
|**Données de fichier**<br /><br />Sauvegarde des données régulières<br /><br />Récupération complète/sauvegarde de l’état du système|Données de fichiers perdues|O|N|N|
|**Données de fichier**<br /><br />Sauvegarde effectuée par le Serveur de sauvegarde Azure des données de fichier<br /><br />Récupération complète/sauvegarde de l’état du système|Système d’exploitation perdu ou endommagé|N|O|O|
|**Données de fichier**<br /><br />Sauvegarde effectuée par le Serveur de sauvegarde Azure des données de fichier<br /><br />Récupération complète/sauvegarde de l’état du système|Serveur perdu (volumes de données intacts)|N|N|O|
|**Données de fichier**<br /><br />Sauvegarde effectuée par le Serveur de sauvegarde Azure des données de fichier<br /><br />Récupération complète/sauvegarde de l’état du système|Serveur perdu (volumes de données perdus)|O|N|O<br /><br />Récupération complète, suivie d’une récupération régulière des données de fichiers sauvegardées|
|**Données SharePoint**<br /><br />Sauvegarde effectuée par le Serveur de sauvegarde Azure des données de batterie de serveurs<br /><br />Récupération complète/sauvegarde de l’état du système|Site perdu, listes, éléments de liste, documents|O|N|N|
|**Données SharePoint**<br /><br />Sauvegarde effectuée par le Serveur de sauvegarde Azure des données de batterie de serveurs<br /><br />Récupération complète/sauvegarde de l’état du système|Système d’exploitation perdu ou endommagé|N|O|O|
|**Données SharePoint**<br /><br />Sauvegarde effectuée par le Serveur de sauvegarde Azure des données de batterie de serveurs<br /><br />Récupération complète/sauvegarde de l’état du système|Récupération d'urgence|N|N|N|
|Windows Server 2012 R2 Hyper-V<br /><br />Sauvegarde effectuée par le Serveur de sauvegarde Azure de l’hôte ou de l’invité Hyper-V<br /><br />Récupération complète/Sauvegarde de l’état du système|Machine virtuelle perdue|O|N|N|
|Hyper-V<br /><br />Sauvegarde effectuée par le Serveur de sauvegarde Azure de l’hôte ou de l’invité Hyper-V<br /><br />Récupération complète/Sauvegarde de l’état du système|Système d’exploitation perdu ou endommagé|N|O|O|
|Hyper-V<br /><br />Sauvegarde effectuée par le Serveur de sauvegarde Azure de l’hôte ou de l’invité Hyper-V<br /><br />Récupération complète/Sauvegarde de l’état du système|Hôte Hyper-V perdu (machines virtuelles intactes)|N|N|O|
|Hyper-V<br /><br />Sauvegarde effectuée par le Serveur de sauvegarde Azure de l’hôte ou de l’invité Hyper-V<br /><br />Récupération complète/Sauvegarde de l’état du système|Hôte Hyper-V perdu (machines virtuelles perdues)|N|N|O<br /><br />Récupération complète, suivie d’une récupération régulière du Serveur de sauvegarde Azure|
|SQL Server/Exchange<br /><br />Sauvegarde d’application effectuée par le Serveur de sauvegarde Azure<br /><br />Récupération complète/sauvegarde de l’état du système|Données d’application perdues|O|N|N|
|SQL Server/Exchange<br /><br />Sauvegarde d’application effectuée par le Serveur de sauvegarde Azure<br /><br />Récupération complète/sauvegarde de l’état du système|Système d’exploitation perdu ou endommagé|N|O|O|
|SQL Server/Exchange<br /><br />Sauvegarde d’application effectuée par le Serveur de sauvegarde Azure<br /><br />Récupération complète/sauvegarde de l’état du système|Serveur perdu (base de données/journaux d’activité des transactions intacts)|N|N|O|
|SQL Server/Exchange<br /><br />Sauvegarde d’application effectuée par le Serveur de sauvegarde Azure<br /><br />Récupération complète/sauvegarde de l’état du système|Serveur perdu (base de données/journaux d’activité des transactions perdus)|N|N|O<br /><br />Récupération complète, suivie d’une récupération régulière du Serveur de sauvegarde Azure|

## <a name="how-system-state-backup-works"></a>Fonctionnement de la sauvegarde de l’état du système

Lors d’une sauvegarde de l’état du système, le Serveur de sauvegarde communique avec l’application Sauvegarde Windows Server pour demander une sauvegarde de l’état du système du serveur. Par défaut, le Serveur de sauvegarde et l’application Sauvegarde Windows Server utilisent le lecteur offrant le plus d’espace libre. Les informations relatives à ce lecteur sont enregistrées dans le fichier *PSDataSourceConfig.xml*.

Vous pouvez personnaliser le lecteur que le Serveur de sauvegarde utilise pour sauvegarder l’état du système :

1. Sur le serveur protégé, accédez à *C:\Program Files\Microsoft Data Protection Manager\MABS\Datasources*.
1. Ouvrez le fichier *PSDataSourceConfig.xml* pour le modifier.
1. Définissez la valeur \<FilesToProtect\> sur la lettre du lecteur.
1. Enregistrez et fermez le fichier.

Si un groupe de protection est défini pour protéger l’état du système de l’ordinateur, exécutez une vérification de cohérence. Si une alerte est générée, sélectionnez **Modifier le groupe de protection** dans l’alerte, puis suivez les instructions des pages de l’Assistant. Ensuite, exécutez une autre vérification de cohérence.

Si le serveur de protection se trouve dans un cluster, un lecteur du cluster peut être sélectionné en tant que lecteur offrant le plus d’espace libre. Si cette propriété du lecteur est basculée vers un autre nœud, en cas d’exécution d’une sauvegarde de l’état du système, le lecteur n’est pas disponible et la sauvegarde échoue. Dans ce scénario, modifiez *PSDataSourceConfig.xml* pour pointer vers un lecteur local.

Ensuite, l’application Sauvegarde Windows Server crée un dossier nommé *WindowsImageBackup* dans la racine du dossier de restauration. Lorsque l’application Sauvegarde Windows Server crée la sauvegarde, toutes les données sont placées dans ce dossier. Au terme de la sauvegarde, le fichier est transféré vers l’ordinateur Serveur de sauvegarde. Notez les informations suivantes :

* Ce dossier et son contenu ne sont pas nettoyés à la fin de la sauvegarde ou du transfert. La meilleure façon de comprendre cela est que l’espace est réservé en vue de la prochaine fois qu’une sauvegarde s’achève.
* Le dossier est créé pour chaque sauvegarde. L’horodatage reflète l’heure de la dernière sauvegarde de l’état du système.

## <a name="how-bmr-backup-works"></a>Fonctionnement de la sauvegarde complète

Pour une récupération complète (incluant la sauvegarde de l’état du système), le travail de sauvegarde est enregistré directement dans un partage sur l’ordinateur Serveur de sauvegarde. Il n’est pas enregistré dans un dossier sur le serveur protégé.

Le Serveur de sauvegarde appelle l’application Sauvegarde Windows Server, et partage le volume du réplica pour cette sauvegarde complète. Dans ce cas, il ne nécessite pas que l’application Sauvegarde Windows Server utilise le lecteur offrant le plus d’espace libre. Au lieu de cela, il utilise le partage créé pour le travail.

Au terme de la sauvegarde, le fichier est transféré vers l’ordinateur Serveur de sauvegarde. Les journaux sont stockés dans *C:\Windows\Logs\WindowsServerBackup*.

## <a name="prerequisites-and-limitations"></a>Conditions préalables et limitations

* La récupération complète n’est pas prise en charge pour les ordinateurs exécutant Windows Server 2003 ou un système d’exploitation de client.

* Vous ne pouvez pas protéger la récupération complète et l’état du système pour le même ordinateur dans des groupes de protection différents.

* Un ordinateur Serveur de sauvegarde ne peut pas se protéger lui-même pour une récupération complète.

* Une protection à court terme sur bande (D2T ou de disque à bande) n’est pas prise en charge pour la récupération complète. Un stockage à long terme sur bande (D2D2T ou disque à disque à bande) est pris en charge.

* Pour la protection de la récupération complète, l’application Sauvegarde Windows Server doit être installée sur l’ordinateur protégé.

* Pour la protection complète, à la différence de la protection de l’état du système, le Serveur de sauvegarde n’impose pas de conditions d’espace disque requis sur l’ordinateur protégé. L’application Sauvegarde Windows Server transfère directement les sauvegardes vers l’ordinateur Serveur de sauvegarde. Le travail de transfert de la sauvegarde n’apparaît pas dans l’affichage des **travaux** du Serveur de sauvegarde.

* Le Serveur de sauvegarde réserve 30 Go d’espace sur le volume du réplica pour la récupération complète. Vous pouvez changer cette allocation d’espace dans la page **Allocation de disque** de l’Assistant Modification du groupe de protection. Si vous préférez, vous pouvez utiliser les applets de commande PowerShell DatasourceDiskAllocation et Set-DatasourceDiskAllocation. Sur le volume des points de récupération, la protection de la récupération complète nécessite environ 6 Go pour une durée de rétention de cinq jours.
  * Vous ne pouvez pas réduire la taille du volume du réplica à moins de 15 Go.
  * Le Serveur de sauvegarde ne calcule pas la taille de la source de données de récupération complète. Il suppose qu’elle est de 30 Go pour tous les serveurs. Modifiez la valeur en fonction de la taille des sauvegardes complètes que vous anticipez dans votre environnement. Vous pouvez calculer approximativement la taille d’une sauvegarde complète comme étant la totalité de l’espace utilisé sur tous les volumes critiques. Volumes critiques = volume de démarrage + volume système + volume hébergeant les données d’état du système, tel qu’Active Directory.

* Si vous passez d’une protection de l’état du système à une protection complète, celle-ci nécessite moins d’espace sur le *volume des points de récupération*. Toutefois, l’espace restant sur le volume n’est pas récupéré. Vous pouvez réduire manuellement la taille du volume dans la page **Modifier l’allocation de disque** de l’Assistant Modification du groupe de protection. Si vous préférez, vous pouvez utiliser les applets de commande PowerShell DatasourceDiskAllocation et Set-DatasourceDiskAllocation.

    Si vous passez d’une protection de l’état du système à une protection complète, celle-ci nécessite davantage d’espace sur le *volume du réplica*. Le volume est automatiquement étendu. Si vous souhaitez changer les allocations d’espace par défaut, utilisez l’applet de commande PowerShell Modify-DiskAllocation.

* Si vous passez d’une protection complète à une protection de l’état du système, vous avez besoin de davantage d’espace sur le volume des points de récupération. Le Serveur de sauvegarde peut tenter d’augmenter automatiquement le volume. Si le pool de stockage n’a pas suffisamment d’espace, une erreur se produit.

    Si vous passez d’une protection complète à une protection de l’état du système, vous avez besoin d’espace sur l’ordinateur protégé. Vous avez besoin de l’espace, car la protection de l’état du système commence par écrire le réplica sur l’ordinateur local, avant de transférer le réplica vers l’ordinateur Serveur de sauvegarde.

## <a name="before-you-begin"></a>Avant de commencer

1. **Déployez le Serveur de sauvegarde Azure**. Vérifiez que le Serveur de sauvegarde est correctement déployé. Pour plus d'informations, consultez les pages suivantes :
    * [Configuration requise pour le Serveur de sauvegarde Azure](/system-center/dpm/install-dpm#setup-prerequisites)
    * [Matrice de protection du Serveur de sauvegarde](backup-mabs-protection-matrix.md)

1. **Configurez le stockage**. Vous pouvez stocker les données de sauvegarde sur disque, sur bande et dans le cloud avec Azure. Pour plus d’informations, voir [Préparer l’espace de stockage](/system-center/dpm/plan-long-and-short-term-data-storage).

1. **Configurez l’agent de protection**. Installez l’agent de protection sur l’ordinateur que vous souhaitez sauvegarder. Pour plus d’informations, voir [Déployer l’agent de protection DPM](/system-center/dpm/deploy-dpm-protection-agent).

## <a name="back-up-system-state-and-bare-metal"></a>Sauvegarde de l’état du système et récupération complète

Pour sauvegarder l’état du système et la récupération complète :

1. Pour ouvrir l’Assistant Création d’un groupe de protection dans la Console Administrateur du Serveur de sauvegarde, sélectionnez **Protection** > **Actions** > **Créer un groupe de Protection**.

1. Dans la page **Sélectionner le type de groupe de protection**, sélectionnez **Serveurs**, puis **Suivant**.

1. Dans la page **Sélectionner les membres du groupe**, développez l’ordinateur, puis sélectionnez **Récupération complète** ou **État du système**.

    N’oubliez pas que vous ne pouvez pas protéger la récupération complète et l’état du système pour un même ordinateur dans des groupes différents. Par ailleurs, lorsque vous sélectionnez la récupération complète, la protection de l’état du système est automatiquement activée. Pour plus d’informations, voir [Déployer des groupes de protection](/system-center/dpm/create-dpm-protection-groups).

1. Dans la page **Sélectionner la méthode de protection des données**, choisissez comment gérer les sauvegardes à court terme et les sauvegardes à long terme.

    La sauvegarde à court terme est toujours effectuée d’abord sur disque, avec l’option de sauvegarder du disque dans Azure à l’aide de Sauvegarde Microsoft Azure (à court ou long terme). Une alternative à la sauvegarde à long terme dans le cloud consiste à configurer la sauvegarde à long terme sur un lecteur de bandes ou dans une bibliothèque de bandes autonomes connectés au Serveur de sauvegarde.

1. Dans la page **Sélectionner les objectifs à court terme**, choisissez comment sauvegarder dans le stockage à court terme sur disque :
    * Pour **Durée de rétention**, choisissez la durée de conservation des données sur disque.
    * Pour **Fréquence de synchronisation**, choisissez la fréquence à laquelle exécuter une sauvegarde incrémentielle sur disque. Si vous ne souhaitez pas définir un intervalle de sauvegarde, vous pouvez sélectionner **Juste avant un point de récupération**. Le Serveur de sauvegarde exécute une sauvegarde complète rapide juste avant chaque point de récupération planifié.

1. Si vous souhaitez stocker des données sur bande pour un stockage à long terme, dans la page **Spécifier les objectifs à long terme**, choisissez la durée de conservation des données sur bande (1 à 99 ans).
    1. Pour **Fréquence de sauvegarde**, choisissez la fréquence à laquelle exécuter la sauvegarde sur bande. La fréquence est basée sur la durée de conservation que vous avez sélectionnée :
        * Quand la durée de conservation est de 1 à 99 ans, vous pouvez effectuer des sauvegardes quotidiennes, hebdomadaires, bihebdomadaires, mensuelles, trimestrielles, semestrielles ou annuelles.
        * Quand la durée de conservation est de 1 à 11 mois, vous pouvez effectuer des sauvegardes quotidiennes, hebdomadaires, bihebdomadaires ou mensuelles.
        * Quand la durée de conservation est de 1 à 4 semaines, vous pouvez effectuer des sauvegardes quotidiennes ou hebdomadaires.

    1. Dans la page **Sélectionner la bande et les détails de la bibliothèque**, sélectionnez la bande et la bibliothèque à utiliser. Indiquez également si les données doivent être compressées et chiffrées.

1. Dans la page **Vérifier l’allocation de disque**, vérifiez l’espace disque de pool de stockage disponible pour ce groupe de protection.

    * La **Taille totale des données** est la taille des données que vous souhaitez sauvegarder.
    * L’**Espace disque à approvisionner sur le Serveur de sauvegarde Azure** est l’espace que le Serveur de sauvegarde recommande pour le groupe de protection. Le Serveur de sauvegarde utilise ces paramètres pour choisir le volume de sauvegarde idéal. Vous pouvez modifier les choix de volume de sauvegarde dans **Détails de l’allocation de disque**.
    * Pour les charges de travail, dans le menu déroulant, sélectionnez le stockage par défaut. Vos modifications changent les valeurs de **Stockage Total** et **Stockage libre** dans le volet **Stockage sur disque disponible**. L’espace sous-provisionné est la quantité de stockage que le Serveur de sauvegarde suggère que vous ajoutiez au volume pour garantir des sauvegardes sans heurt.

1. Dans la page **choisir la méthode de création de réplica**, sélectionnez la façon de gérer la réplication initiale complète des données.

    Si vous choisissez de répliquer sur le réseau, nous vous recommandons de choisir une heure creuse. Si les quantités de données sont importantes ou si les conditions réseau ne sont pas optimales, envisagez de répliquer les données hors connexion à l’aide d’un support amovible.

1. Dans la page **Sélectionner les options de vérification de cohérence**, indiquez comment automatiser les vérifications de cohérence.

    Vous pouvez choisir d’exécuter une vérification uniquement lorsque les données du réplica deviennent incohérentes, ou selon une planification. Si vous ne voulez pas configurer une vérification de cohérence automatique, vous pouvez effectuer une vérification manuelle à toute moment. Pour effectuer une vérification manuelle, dans la zone **Protection** de la Console Administrateur du Serveur de sauvegarde, cliquez avec le bouton droit sur le groupe de protection, puis sélectionnez **Effectuer une vérification de cohérence**.

1. Si vous avez choisi de sauvegarder dans le cloud à l’aide de Sauvegarde Azure, dans la page **Indiquer les données de protection en ligne**, sélectionnez les charges de travail que vous souhaitez sauvegarder dans Azure.

1. Dans la page **Spécifier la planification de sauvegarde en ligne**, sélectionnez la fréquence des sauvegardes incrémentielles dans Azure.

    Vous pouvez planifier des sauvegardes pour qu’elles s’exécutent tous les jours, toutes les semaines, tous les mois et toutes les années. Vous pouvez également sélectionner la date et l’heure auxquelles les sauvegardes doivent s’exécuter. Les sauvegardes peuvent avoir lieu jusqu’à deux fois par jour. Chaque fois qu’une sauvegarde s’exécute, un point de récupération de données est créé dans Azure à partir de la copie des données de sauvegarde qui sont stockées sur le disque du Serveur de sauvegarde.

1. Dans la page **Spécifier la stratégie de rétention en ligne**, sélectionnez la façon dont les points de récupération créés à partir des sauvegardes quotidiennes, hebdomadaires, mensuelles et annuelles sont conservés dans Azure.

1. Dans la page **Choisir la réplication en ligne**, sélectionnez la façon dont la réplication initiale complète des données doit se produire.

    Vous pouvez répliquer sur le réseau ou effectuer une sauvegarde en mode hors connexion (essaimage hors connexion). Une sauvegarde en mode hors connexion utilise la fonctionnalité Azure Import. Pour plus d’informations, voir [Flux de travail de la sauvegarde hors connexion dans la sauvegarde Azure](offline-backup-azure-data-box.md).

1. Dans la page **Résumé**, vérifiez vos paramètres. Après que vous avez sélectionné **Créer un groupe**, la réplication initiale des données est effectuée. Une fois la réplication des données terminée, dans la page **État**, l’état du groupe de protection est **OK**. Les sauvegardes se produisent ensuite conformément aux paramètres du groupe de protection.

## <a name="recover-system-state-or-bmr"></a>Récupérer l’état du système ou la récupération complète

Vous pouvez récupérer la récupération complète ou l’état du système dans un emplacement réseau. Si vous avez sauvegardé une récupération complète, utilisez l’environnement de récupération Windows (WinRE) pour démarrer votre système et le connecter au réseau. Ensuite, utilisez l’application Sauvegarde Windows Server pour récupérer à partir de l’emplacement réseau. Si vous avez sauvegardé l’état du système, utilisez simplement l’application Sauvegarde Windows Server pour récupérer à partir de l’emplacement réseau.

### <a name="restore-bmr"></a>Restaurer une récupération complète

Pour exécuter la récupération sur l’ordinateur Serveur de sauvegarde :

1. Dans le volet **Récupération**, recherchez l’ordinateur à récupérer. Ensuite, sélectionnez **Récupération complète**.

1. Les points de récupération disponibles sont indiqués en gras dans le calendrier. Sélectionnez la date et l’heure du point de récupération à utiliser.

1. Dans la page **Sélectionner le type de récupération**, sélectionnez **Copier dans un dossier réseau**.

1. Dans la page **Spécifier la destination**, sélectionnez la destination des données copiées.

    N’oubliez pas que la destination doit offrir suffisamment d’espace. Nous vous recommandons de créer un dossier pour la destination.

1. Dans la page **Spécifier les options de récupération**, sélectionnez les paramètres de sécurité. Ensuite, indiquez s’il faut utiliser des instantanés matériels basés sur le réseau de zone de stockage (SAN) pour accélérer la récupération. Cette option est disponible uniquement si :
    * Vous disposez d’un réseau SAN qui fournit cette fonctionnalité.
    * Vous pouvez créer et diviser un clone pour le rendre accessible en écriture.
    * L’ordinateur protégé et l’ordinateur Serveur de sauvegarde sont connectés au même réseau.

1. Configurez les options de notification.
1. Dans la page **Confirmation**, sélectionnez **Récupérer**.

Pour configurer l’emplacement du partage :

1. Dans l’emplacement de restauration, accédez au dossier contenant la sauvegarde.

1. Partagez le dossier situé un niveau au-dessus de *WindowsImageBackup*, de façon à ce que la racine du dossier partagé soit le dossier *WindowsImageBackup*.

    Si vous ne partagez pas ce dossier, la restauration ne trouve pas la sauvegarde. Pour vous connecter en utilisant WinRE, vous avez besoin d’un partage auquel vous pouvez accéder dans WinRE avec l’adresse IP et les informations d’identification correctes.

Pour restaurer le système :

1. Démarrez l’ordinateur sur lequel vous souhaitez restaurer l’image à l’aide du DVD Windows du système que vous restaurez.

1. Dans la première page, vérifiez les paramètres de langue et régionaux. Dans la page **Installer**, sélectionnez **Réparer votre ordinateur**.

1. Dans la page **Options de récupération système**, sélectionnez **Restaurer votre ordinateur avec une image système créée précédemment**.

1. Dans la page **Sélectionner une sauvegarde d’image système**, choisissez **Sélectionner une image système** > **Avancé** > **Chercher une image système sur le réseau**. Si un avertissement s’affiche, sélectionnez **Oui**. Accédez au chemin d’accès du partage, entrez les informations d’identification, puis sélectionnez le point de récupération. Le système déclenche une analyse des sauvegardes spécifiques disponibles dans ce point de récupération. Sélectionnez le point de récupération que vous souhaitez utiliser.

1. Sur la page **Choisir comment restaurer la sauvegarde**, sélectionnez **Formater et repartitionner les disques**. Dans la page suivante, vérifiez les paramètres.

1. Pour commencer la restauration, sélectionnez **Terminer**. Un redémarrage est requis.

### <a name="restore-system-state"></a>Restaurer l’état du système

Pour exécuter la récupération dans le Serveur de sauvegarde :

1. Dans le volet **Récupération**, recherchez l’ordinateur à récupérer, puis sélectionnez **Récupération complète**.

1. Les points de récupération disponibles sont indiqués en gras dans le calendrier. Sélectionnez la date et l’heure du point de récupération à utiliser.

1. Sur la page **Sélectionner le type de récupération**, sélectionnez **Copier dans un dossier réseau**.

1. Dans la page **Spécifier la destination**, sélectionnez l’emplacement auquel copier les données.

    N’oubliez pas que la destination que vous sélectionnez doit offrir suffisamment d’espace. Nous vous recommandons de créer un dossier pour la destination.

1. Dans la page **Spécifier les options de récupération**, sélectionnez les paramètres de sécurité. Ensuite, indiquez s’il faut utiliser des instantanés matériels basés sur le SAN pour accélérer la récupération. Cette option est disponible uniquement si :
    * Vous disposez d’un réseau SAN qui fournit cette fonctionnalité.
    * Vous pouvez créer et diviser un clone pour le rendre accessible en écriture.
    * L’ordinateur protégé et l’ordinateur Serveur de sauvegarde sont connectés au même réseau.

1. Configurez les options de notification.
1. Dans la page **Confirmation**, sélectionnez **Récupérer**.

Pour exécuter l’application Sauvegarde Windows Server :

1. Sélectionnez **Actions** > **Récupérer** > **Ce serveur** > **Suivant**.

1. Sélectionnez **Un autre serveur**, choisissez la page **Spécifier un type d’emplacement**, puis sélectionnez **Dossier partagé distant**. Entrez le chemin d’accès au dossier contenant le point de récupération.

1. Dans la page **Sélectionner le type de récupération**, choisissez **État du système**.

1. Dans la page **Sélectionner l’emplacement pour la récupération de l’état du système**, choisissez **Emplacement d’origine**.

1. Dans la page **Confirmation**, sélectionnez **Récupérer**.

1. Après la restauration, redémarrez le serveur.

Vous pouvez également exécuter la restauration de l’état du système depuis une invite de commandes :

1. Démarrez l’application Sauvegarde Windows Server sur l’ordinateur que vous souhaitez récupérer.

1. Pour obtenir l’identificateur de version, depuis une invite de commandes, entrez ce qui suit :

   ```wbadmin get versions -backuptarget \<servername\sharename\>```

1. Utilisez l’identificateur de version pour démarrer la restauration de l’état du système. À l’invite de commandes, tapez :

    ```wbadmin start systemstaterecovery -version:<versionidentified> -backuptarget:<servername\sharename>```

1. Confirmez que vous souhaitez démarrer la récupération. Vous pouvez voir le processus dans la fenêtre d’invite de commandes. Un journal de restauration est créé.

1. Après la restauration, redémarrez le serveur.
