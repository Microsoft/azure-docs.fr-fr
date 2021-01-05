---
title: Gamme StorSimple 8000 en tant que cible de sauvegarde avec Backup Exec | Microsoft Docs
description: Décrit la configuration de la cible de sauvegarde StorSimple avec Veritas Backup Exec.
services: storsimple
documentationcenter: ''
author: harshakirank
manager: matd
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/05/2016
ms.author: matd
ms.openlocfilehash: 66a1e22282864d0425173504735d6beb42b76ad7
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967259"
---
# <a name="storsimple-as-a-backup-target-with-backup-exec"></a>StorSimple comme cible de sauvegarde avec Backup Exec

## <a name="overview"></a>Vue d’ensemble

Azure StorSimple est une solution de stockage cloud hybride de Microsoft. StorSimple s’attaque à la complexité de la croissance exponentielle des données en utilisant un compte de stockage Azure en tant qu’extension de la solution locale et en hiérarchisant automatiquement les données sur le stockage local et sur le stockage cloud.

Dans cet article, nous traitons de l’intégration de StorSimple à Veritas Backup Exec et des meilleures pratiques en matière d’intégration de ces deux solutions. Nous fournissons également diverses recommandations concernant la procédure de configuration de Backup Exec de manière à optimiser son intégration à StorSimple. Nous nous reposons sur les meilleures pratiques, concepteurs d’architecture de sauvegarde et administrateurs de Veritas quant à la meilleure manière de configurer Backup Exec pour répondre aux besoins de sauvegarde individuels et aux contrats de niveau de service.

Bien que cet article illustre les étapes de configuration et les concepts clés, cela ne constitue en aucun cas un guide de configuration ou d’installation pas à pas. Nous nous reposons sur l’hypothèse que l’infrastructure et les composants de base sont en état de fonctionnement et prêts à prendre en charge les concepts que nous décrivons.

### <a name="who-should-read-this"></a>Qui doit lire ce document ?

Les informations dans cet article s’adressent principalement aux administrateurs de sauvegarde, aux administrateurs de stockage et aux concepteurs d’architecture de stockage qui disposent de connaissances concernant le stockage, Windows Server 2012 R2, Ethernet, les services cloud et Backup Exec.

## <a name="supported-versions"></a>Versions prises en charge

-   [Backup Exec 16 et versions ultérieures](https://www.veritas.com/content/support/en_US/article.100040087)
-   [StorSimple Update 3 et versions ultérieures](storsimple-overview.md#storsimple-workload-summary)


## <a name="why-storsimple-as-a-backup-target"></a>Pourquoi utiliser StorSimple comme cible de sauvegarde ?

StorSimple est un bon choix en tant que cible de sauvegarde, car :

-   Il offre un stockage local standard utilisable par les applications de sauvegarde pour fournir une destination de sauvegarde rapide, sans aucune modification. Vous pouvez utiliser StorSimple pour les restaurations rapides des sauvegardes récentes.
-   Sa hiérarchisation cloud s’intègre de façon transparente à un compte de stockage cloud Azure pour tirer parti d’un stockage Azure économique.
-   StorSimple fournit automatiquement un stockage hors site pour la récupération d’urgence.

## <a name="key-concepts"></a>Concepts clés

Comme dans le cas de toute solution de stockage, une évaluation en profondeur des performances de stockage de la solution, des contrats de niveau de service, de la fréquence de changement et de l’augmentation des besoins en matière de capacité se révèle un facteur décisif de réussite. L’idée générale consiste à introduire un niveau cloud, de sorte que les temps d’accès et débits que vous offre le cloud joueront un rôle fondamental dans l’efficacité de StorSimple.

StorSimple est conçu pour fournir une capacité de stockage aux applications qui opèrent sur un jeu de données de travail (données chaudes) bien défini. Dans ce modèle, le jeu de données de travail est stocké sur les niveaux locaux, tandis que les données inactives/froides/archivées sont hiérarchisées vers le cloud. Ce modèle est illustré dans la figure ci-après. La ligne verte quasiment plate représente les données stockées sur les niveaux locaux de l’appareil StorSimple. La ligne rouge représente la quantité totale de données stockées sur tous les niveaux de la solution StorSimple. L’espace entre la ligne verte plate et la courbe exponentielle rouge représente la quantité totale de données stockées dans le cloud.

**Hiérarchisation StorSimple**
![Diagramme de la hiérarchisation StorSimple](./media/storsimple-configure-backup-target-using-backup-exec/image1.jpg)

En gardant cette architecture à l’esprit, vous découvrirez que la solution StorSimple constitue une cible de sauvegarde idéale. Vous pouvez utiliser StorSimple pour :
-   procéder aux restaurations les plus fréquentes à partir du jeu de données de travail local ;
-   utiliser le cloud pour la récupération d’urgence hors site et pour les données plus anciennes impliquant des restaurations moins fréquentes.

## <a name="storsimple-benefits"></a>Avantages de StorSimple

StorSimple fournit une solution locale qui s’intègre parfaitement à Microsoft Azure en tirant parti d’un accès transparent au stockage local et cloud.

StorSimple utilise la hiérarchisation automatique entre l’appareil local, qui contient un stockage SSD (solid-state device) et SAS (serial-attached SCSI), et le service Stockage Azure. La hiérarchisation automatique conserve les données fréquemment sollicitées en local, sur les niveaux SSD et SAS. Elle déplace les données rarement utilisées vers le Stockage Azure.

StorSimple offre les avantages suivants :

-   Algorithmes de déduplication et de compression uniques qui tirent parti du cloud pour atteindre des niveaux de déduplication inégalés
-   Haute disponibilité
-   Géoréplication reposant sur la fonctionnalité de géoréplication Azure
-   Intégration d’Azure
-   Chiffrement de données dans le cloud
-   Récupération d’urgence et conformité améliorées

Bien que StorSimple présente deux principaux scénarios de déploiement (cible de sauvegarde principale et secondaire), il s’agit essentiellement d’un dispositif de stockage de bloc. StorSimple exécute la totalité des tâches de compression et de déduplication. Il envoie et récupère de façon transparente les données entre le cloud, l’application et le système de fichiers.

Pour plus d’informations sur StorSimple, consultez l’article [StorSimple série 8000 : une solution de stockage de cloud hybride](storsimple-overview.md). En outre, vous pouvez consulter les [caractéristiques techniques de StorSimple série 8000](./storsimple-8000-technical-specifications-and-compliance.md).

> [!IMPORTANT]
> L’utilisation d’un appareil StorSimple comme cible de sauvegarde n’est prise en charge que pour StorSimple 8000 Update 3 et les versions ultérieures.

## <a name="architecture-overview"></a>Vue d’ensemble de l’architecture

Les tableaux ci-après contiennent les informations d’aide initiales concernant l’architecture liée aux différents modèles d’appareil.

**Capacités de stockage local et cloud offertes par StorSimple**

| Capacité de stockage       | 8100          | 8600            |
|------------------------|---------------|-----------------|
| Capacité de stockage local | &lt; 10 Tio\*  | &lt; 20 Tio\*  |
| Capacité de stockage cloud | &gt; 200 Tio\* | &gt; 500 Tio\* |

\* La taille de stockage indiquée ne prend en compte aucune déduplication ni compression.

**Capacités de StorSimple dans les scénarios de sauvegarde principale et secondaire**

| Scénario de sauvegarde  | Capacité de stockage local  | Capacité de stockage cloud  |
|---|---|---|
| Sauvegarde principale  | Sauvegardes récentes stockées sur le stockage local afin d’accélérer la récupération pour se conformer à l’objectif de point de récupération (RPO) | Capacité du cloud adaptée à l’historique des sauvegardes (RPO) |
| Sauvegarde secondaire | Possibilité de stocker une copie secondaire des données de sauvegarde dans la capacité du cloud  | N/A  |

## <a name="storsimple-as-a-primary-backup-target"></a>Utiliser StorSimple comme cible de sauvegarde principale

Dans ce scénario, les volumes StorSimple sont présentés à l’application de sauvegarde comme l’unique référentiel pour les sauvegardes. La figure ci-dessous illustre l’architecture de la solution dans laquelle toutes les sauvegardes utilisent les volumes hiérarchisés StorSimple pour les sauvegardes et pour les restaurations.

![Diagramme logique de l’utilisation de StorSimple comme cible de sauvegarde principale](./media/storsimple-configure-backup-target-using-backup-exec/primarybackuptargetlogicaldiagram.png)

### <a name="primary-target-backup-logical-steps"></a>Procédure logique de sauvegarde de la cible principale

1.  Le serveur de sauvegarde contacte l’agent de sauvegarde cible qui transmet alors des données au serveur de sauvegarde.
2.  Le serveur de sauvegarde écrit les données sur les volumes hiérarchisés StorSimple.
3.  Le serveur de sauvegarde met à jour la base de données de catalogues et accomplit le travail de sauvegarde.
4.  Un script de capture instantanée déclenche le gestionnaire des captures instantanées cloud de StorSimple (démarrage ou suppression).
5.  Conformément à une stratégie de rétention, le serveur de sauvegarde supprime les sauvegardes arrivées à expiration.


### <a name="primary-target-restore-logical-steps"></a>Procédure logique de restauration de la cible principale

1.  Le serveur de sauvegarde commence à restaurer les données appropriées du référentiel de stockage.
2.  L’agent de sauvegarde reçoit les données en provenance du serveur de sauvegarde.
3.  Le serveur de sauvegarde achève le travail de restauration.

## <a name="storsimple-as-a-secondary-backup-target"></a>Utiliser StorSimple comme cible de sauvegarde secondaire

Dans ce scénario, les volumes StorSimple sont principalement utilisés à des fins de rétention à long terme ou d’archivage.

La figure ci-après illustre l’architecture dans laquelle les sauvegardes et restaurations initiales ciblent un volume hautes performances. Ces sauvegardes sont copiées et archivées sur un volume hiérarchisé StorSimple conformément à une planification donnée.

Il est important que vous configuriez correctement votre volume hautes performances pour lui permettre de prendre en charge les exigences en matière de performances et de capacité de stratégie de rétention.

![Diagramme logique de l’utilisation de StorSimple comme cible de sauvegarde secondaire](./media/storsimple-configure-backup-target-using-backup-exec/secondarybackuptargetlogicaldiagram.png)

### <a name="secondary-target-backup-logical-steps"></a>Procédure logique de sauvegarde de la cible secondaire

1.  Le serveur de sauvegarde contacte l’agent de sauvegarde cible qui transmet alors des données au serveur de sauvegarde.
2.  Le serveur de sauvegarde écrit les données dans le stockage hautes performances.
3.  Le serveur de sauvegarde met à jour la base de données de catalogues et accomplit le travail de sauvegarde.
4.  Conformément à une stratégie de rétention, le serveur de sauvegarde copie les sauvegardes dans StorSimple.
5.  Un script de capture instantanée déclenche le gestionnaire des captures instantanées cloud de StorSimple (démarrage ou suppression).
6.  Conformément à une stratégie de rétention, le serveur de sauvegarde supprime les sauvegardes arrivées à expiration.

### <a name="secondary-target-restore-logical-steps"></a>Procédure logique de restauration de la cible secondaire

1.  Le serveur de sauvegarde commence à restaurer les données appropriées du référentiel de stockage.
2.  L’agent de sauvegarde reçoit les données en provenance du serveur de sauvegarde.
3.  Le serveur de sauvegarde achève le travail de restauration.

## <a name="deploy-the-solution"></a>Déployer la solution

Trois étapes sont nécessaires pour le déploiement de la solution :
1. Préparer l’infrastructure réseau.
2. Déployer votre appareil StorSimple comme cible de sauvegarde.
3. Déployer Veritas Backup Exec.

Chacune de ces étapes est abordée en détail dans les sections suivantes.

### <a name="set-up-the-network"></a>Configurer le réseau

En tant que solution intégrée au cloud Azure, StorSimple requiert une connexion active et opérationnelle à ce dernier. Cette connexion est utilisée pour des opérations telles que les captures instantanées cloud, la gestion, le transfert de métadonnées et la hiérarchisation des données les plus anciennes et les moins sollicitées vers le stockage cloud Azure.

Pour que la solution fonctionne de manière optimale, nous vous recommandons d’appliquer les meilleures pratiques de mise en réseau suivantes :

-   La liaison qui connecte la hiérarchisation StorSimple à Azure doit répondre à vos exigences de bande passante. Pour ce faire, appliquez la Qualité de service (QoS) appropriée aux commutateurs de votre infrastructure conformément à vos contrats de niveau de service en matière d’objectifs de point de récupération (RPO) et de temps de récupération (RTO).
-   Les latences d’accès maximales au service Stockage Blob Azure doivent être de l’ordre de 80 ms.

### <a name="deploy-storsimple"></a>Déployer StorSimple

Pour découvrir un guide de déploiement de StorSimple pas à pas, consultez l’article [Déploiement de votre appareil StorSimple local](./storsimple-8000-deployment-walkthrough-u2.md).

### <a name="deploy-backup-exec"></a>Déployer Backup Exec

Pour découvrir les meilleures pratiques en matière d’installation de Backup Exec, consultez l’article [Best practices for Backup Exec installation (Meilleures pratiques en matière d’installation de Backup Exec)](https://www.veritas.com/content/support/en_US/doc/72686287-131623464-0/v70444238-131623464).

## <a name="set-up-the-solution"></a>Configurer la solution

Dans cette section, nous fournissons quelques exemples de configuration. Les exemples et recommandations ci-après illustrent l’implémentation la plus élémentaire et fondamentale. Cette implémentation peut ne pas s’appliquer directement à vos exigences spécifiques en matière de sauvegarde.

### <a name="set-up-storsimple"></a>Configurer StorSimple

| Tâches de déploiement StorSimple  | Commentaires supplémentaires |
|---|---|
| Déploiement de votre appareil StorSimple local | Versions prises en charge : Update 3 et versions ultérieures. |
| Activez la cible de sauvegarde. | Utilisez ces commandes pour activer ou désactiver le mode de cible de sauvegarde et pour obtenir l’état. Pour plus d’informations, consultez l’article [Connexion à distance à un appareil StorSimple](./storsimple-8000-remote-connect.md).</br> Pour activer le mode de sauvegarde : `Set-HCSBackupApplianceMode -enable`. </br> Pour désactiver le mode de sauvegarde : `Set-HCSBackupApplianceMode -disable`. </br> Pour obtenir l’état actuel des paramètres de mode de sauvegarde : `Get-HCSBackupApplianceMode`. |
| Création d’un conteneur de volumes commun pour votre volume qui stocke les données de sauvegarde. Toutes les données d’un conteneur de volumes sont dédupliquées. | Les conteneurs de volumes StorSimple définissent les domaines de déduplication.  |
| Créez les volumes StorSimple. | Créez des volumes en les dimensionnant le plus conformément possible à l’usage que vous prévoyez d’en faire, car la taille du volume a une incidence sur la durée des captures instantanées cloud. Pour plus d’informations sur la taille d’un volume, consultez les [stratégies de rétention](#retention-policies).</br> </br> Utilisez les volumes hiérarchisés StorSimple et cochez la case **Utiliser ce volume pour les données d’archivage auxquelles vous accédez moins souvent**. </br> L’utilisation de volumes épinglés localement uniquement n’est pas prise en charge. |
| Création d’une stratégie de sauvegarde StorSimple unique pour tous les volumes cibles de sauvegarde. | Une stratégie de sauvegarde StorSimple définit le groupe de cohérence de volume. |
| Désactivez la planification lorsque les captures instantanées arrivent à expiration. | Les captures instantanées sont déclenchées sous la forme d’une opération post-traitement. |

### <a name="set-up-the-host-backup-server-storage"></a>Configurer le stockage du serveur de sauvegarde hôte

Configurez le stockage du serveur de sauvegarde hôte conformément aux instructions suivantes :  

- N’utilisez pas des volumes fractionnés (créés par le Gestionnaire de disque Windows). Les disques fractionnés ne sont pas pris en charge.
- Formatez vos volumes à l’aide du système de fichiers NTFS avec une taille d’allocation de 64 Ko.
- Mappez les volumes StorSimple directement sur le serveur Backup Exec.
    - Utilisez iSCSI pour les serveurs physiques.
    - Utilisez des disques pass-through pour les serveurs virtuels.

## <a name="best-practices-for-storsimple-and-backup-exec"></a>Meilleures pratiques pour StorSimple et Backup Exec

Configurez votre solution en respectant les instructions dans les sections suivantes.

### <a name="operating-system-best-practices"></a>Meilleures pratiques concernant le système d’exploitation

- Désactivez le chiffrement et la déduplication Windows Server pour le système de fichiers NTFS.
- Désactivez la défragmentation Windows Server sur les volumes StorSimple.
- Désactivez l’indexation Windows Server sur les volumes StorSimple.
- Exécutez une analyse antivirus au niveau de l’hôte source (et non par rapport aux volumes StorSimple).
- Désactivez l’activité [Maintenance Windows Server](/windows/win32/w8cookbook/automatic-maintenance) par défaut dans le Gestionnaire des tâches. Pour ce faire, procédez de l’une des manières suivantes :
  - Désactivez le programme de configuration de l’activité de maintenance dans le Planificateur de tâches Windows.
  - Téléchargez [PsExec](/sysinternals/downloads/psexec) à partir de Windows Sysinternals. Après avoir téléchargé PsExec, exécutez Azure PowerShell en tant qu’administrateur, puis tapez :
    ```powershell
    psexec \\%computername% -s schtasks /change /tn “MicrosoftWindowsTaskSchedulerMaintenance Configurator" /disable
    ```

### <a name="storsimple-best-practices"></a>Meilleures pratiques concernant StorSimple

  -   Assurez-vous que l’appareil StorSimple est mis à jour avec la version [Update 3 ou une version ultérieure](./index.yml).
  -   Isolez le trafic iSCSI et cloud. Utilisez des connexions iSCSI dédiées pour le trafic entre StorSimple et le serveur de sauvegarde.
  -   Assurez-vous que l’appareil StorSimple est une cible de sauvegarde dédiée. Les charges de travail mixtes ne sont pas prises en charge, car elles affectent le RTO et le RPO.

### <a name="backup-exec-best-practices"></a>Meilleures pratiques concernant Backup Exec

-   Backup Exec doit être installé sur un lecteur local du serveur et non sur un volume StorSimple.
-   Définissez les **opérations d’écriture simultanées** de stockage Backup Exec sur la valeur maximale autorisée.
    -   Définissez la **taille des blocs et de la mémoire tampon** de stockage Backup Exec sur 512 Ko.
    -   Activez les **opérations de lecture et d’écriture en mémoire tampon** de stockage Backup Exec.
-   StorSimple prend en charge les sauvegardes Backup Exec complètes et incrémentielles. Nous vous déconseillons d’utiliser les sauvegardes synthétiques et différentielles.
-   Les fichiers de données de sauvegarde doivent uniquement contenir les données d’un travail spécifique. Par exemple, aucun ajout de supports n’est autorisé entre les différents travaux.
-   Désactivez la vérification des travaux. Si nécessaire, la vérification doit être planifiée après le dernier travail de sauvegarde. Il est important de comprendre que ce travail a une incidence sur votre fenêtre de sauvegarde.
-   Sélectionnez **Storage (Stockage)**  > **Your disk (Votre disque)**  > **Details (Détails)**  > **Properties (Propriétés)** . Désactivez **Pre-allocate disk space (Pré-allouer de l’espace disque)** .

Pour obtenir les paramètres et les meilleures pratiques de Backup Exec relatifs à l’implémentation de ces exigences, consultez [le site web Veritas](https://www.veritas.com).

## <a name="retention-policies"></a>Stratégies de rétention

L’une des stratégies de rétention de sauvegarde les plus utilisées est la stratégie GFS (Grandfather, Father, and Son - Grand-père, père et fils). Dans une stratégie GFS, une sauvegarde incrémentielle est effectuée tous les jours et les sauvegardes complètes sont effectuées de façon hebdomadaire et mensuelle. Cette stratégie donne lieu à six volumes hiérarchisés StorSimple. Un même volume contient les sauvegardes complètes hebdomadaires, mensuelles et annuelles. Les cinq autres volumes stockent les sauvegardes incrémentielles quotidiennes.

Dans l’exemple ci-après, nous utilisons une rotation GFS. Cet exemple repose sur les données suivantes :

-   Utilisation de données non dédupliquées ni compressées.
-   Sauvegardes complètes de 1 Tio chacune.
-   Sauvegardes incrémentielles quotidiennes de 500 Gio chacune.
-   Quatre sauvegardes hebdomadaires sont conservées pendant un mois.
-   Douze sauvegardes mensuelles sont conservées pendant un an.
-   Une sauvegarde annuelle est conservée pendant 10 ans.

Sur la base des hypothèses qui précèdent, créez un volume hiérarchisé StorSimple de 26 Tio pour les sauvegardes complètes mensuelles et annuelles. Créez un volume hiérarchisé StorSimple de 5 Tio pour chacune des sauvegardes incrémentielles quotidiennes.

| Type de sauvegarde et rétention | Taille (Tio) | Multiplicateur GFS\* | Capacité totale (Tio)  |
|---|---|---|---|
| Complète hebdomadaire | 1 | 4  | 4 |
| Incrémentielle quotidienne | 0.5 | 20 (cycles correspondant au nombre de semaines par mois) | 12 (2 pour le quota supplémentaire) |
| Complète mensuelle | 1 | 12 | 12 |
| Complète annuelle | 1  | 10 | 10 |
| Exigence GFS |   | 38 |   |
| Quota supplémentaire  | 4  |   | 42 au total pour l’exigence GFS  |

\* Le multiplicateur GFS est le nombre de copies que vous devez protéger et conserver pour vous conformer à vos stratégies de sauvegarde.

## <a name="set-up-backup-exec-storage"></a>Configurer le stockage Backup Exec

### <a name="to-set-up-backup-exec-storage"></a>Pour configurer le stockage Backup Exec

1.  Dans la console d’administration de Backup Exec, sélectionnez **Storage (Stockage)**  > **Configure Storage (Configurer le stockage)**  > **Disk-Based Storage (Stockage sur disque)**  > **Next (Suivant)** .

    ![Console d’administration de Backup Exec, page de configuration du stockage](./media/storsimple-configure-backup-target-using-backup-exec/image4.png)

2.  Sélectionnez **Disk Storage (Stockage sur disque)** , puis sélectionnez **Next (Suivant)** .

    ![Console d’administration de Backup Exec, page de sélection du stockage](./media/storsimple-configure-backup-target-using-backup-exec/image5.png)

3.  Entrez un nom explicite, par exemple, **Samedi complète**, ainsi qu’une description. Sélectionnez **Suivant**.

    ![Console d’administration de Backup Exec, page de saisie d’un nom et d’une description](./media/storsimple-configure-backup-target-using-backup-exec/image7.png)

4.  Sélectionnez le disque sur lequel vous souhaitez créer le dispositif de stockage sur disque, puis sélectionnez **Next (Suivant)** .

    ![Console d’administration de Backup Exec, page de sélection du disque de stockage](./media/storsimple-configure-backup-target-using-backup-exec/image9.png)

5.  Incrémentez le nombre d’opérations d’écriture à **16**, puis cliquez sur **Next (Suivant)** .

    ![Console d’administration de Backup Exec, page des paramètres d’opérations d’écriture simultanées](./media/storsimple-configure-backup-target-using-backup-exec/image10.png)

6.  Passez en revue les paramètres, puis sélectionnez **Finish (Terminer)** .

    ![Console d’administration de Backup Exec, page récapitulative de configuration du stockage](./media/storsimple-configure-backup-target-using-backup-exec/image11.png)

7.  À la fin de chaque affectation de volume, modifiez les paramètres du dispositif de stockage conformément aux paramètres recommandés dans les [Meilleures pratiques pour StorSimple et Backup Exec](#best-practices-for-storsimple-and-backup-exec).

    ![Console d’administration de Backup Exec, page de paramètres du dispositif de stockage](./media/storsimple-configure-backup-target-using-backup-exec/image12.png)

8.  Répétez les étapes 1 à 7 jusqu’à avoir terminé d’affecter vos volumes StorSimple à Backup Exec.

## <a name="set-up-storsimple-as-a-primary-backup-target"></a>Configurer StorSimple comme cible de sauvegarde principale

> [!NOTE]
> En cas de restauration de données à partir d’une sauvegarde qui a été hiérarchisée vers le cloud, la restauration se produit à la vitesse du cloud.

La figure ci-après illustre le mappage d’un volume classique sur un travail de sauvegarde. Dans ce cas précis, toutes les sauvegardes hebdomadaires sont mappées sur le disque des sauvegardes complètes effectuées le samedi, et les sauvegardes incrémentielles sont mappées sur les disques des sauvegardes incrémentielles exécutées du lundi au vendredi. Toutes les sauvegardes et restaurations se produisent à partir d’un volume hiérarchisé StorSimple.

![Diagramme logique de la configuration de la cible de sauvegarde principale](./media/storsimple-configure-backup-target-using-backup-exec/primarybackuptargetdiagram.png)

### <a name="storsimple-as-a-primary-backup-target-gfs-schedule-example"></a>Exemple de planification GFS pour StorSimple comme cible de sauvegarde principale

Voici un exemple de planification de rotation GFS pour quatre semaines, mensuellement et annuellement :

| Fréquence/type de sauvegarde | Complète | Incrémentielle (jours 1-5)  |   
|---|---|---|
| Hebdomadaire (semaines 1-4) | Samedi | Lundi-vendredi |
| Mensuelle  | Samedi  |   |
| Annuelle | Samedi  |   |


### <a name="assign-storsimple-volumes-to-a-backup-exec-backup-job"></a>Affecter des volumes StorSimple à un travail de sauvegarde Backup Exec

La procédure ci-après repose sur l’hypothèse que Backup Exec et l’hôte cible sont configurés conformément aux instructions de l’agent Backup Exec.

#### <a name="to-assign-storsimple-volumes-to-a-backup-exec-backup-job"></a>Pour affecter des volumes StorSimple à un travail de sauvegarde Backup Exec

1.  Dans la console d’administration de Backup Exec, sélectionnez **Host (Hôte)**  > **Backup (Sauvegarde)**  > **Backup to Disk (Sauvegarde sur disque)** .

    ![Console d’administration de Backup Exec, sélection de l’hôte, de la sauvegarde et de la sauvegarde sur disque](./media/storsimple-configure-backup-target-using-backup-exec/image14.png)

2.  Dans la boîte de dialogue **Backup Definition Properties (Propriétés de définition de sauvegarde)** , sélectionnez **Edit (Modifier)** (sous **Backup (Sauvegarde)** ).

    ![Console d’administration de Backup Exec, boîte de dialogue des propriétés de définition de sauvegarde](./media/storsimple-configure-backup-target-using-backup-exec/image15.png)

3.  Configurez vos sauvegardes complètes et incrémentielles conformément à vos exigences en matière de RPO et de RTO et aux bonnes pratiques Veritas.

4.  Dans la boîte de dialogue **Backup Options (Options de sauvegarde)** , sélectionnez **Storage (Stockage)** .

    ![Console d’administration de Backup Exec, boîte de dialogue des options de sauvegarde de stockage](./media/storsimple-configure-backup-target-using-backup-exec/image16.png)

5.  Attribuez les volumes StorSimple correspondants à votre planification de sauvegarde.

    > [!NOTE]
    > Les options **Compression (Compression)** et **Encryption type (Type de chiffrement)** sont définies sur **None (Aucun)** .

6.  Sous **Verify (Vérifier)** , cochez la case **Do not verify data for this job (Ne pas vérifier les données de ce travail)** . Cette option risque d’affecter la hiérarchisation StorSimple.

    > [!NOTE]
    > La défragmentation, l’indexation et la vérification en arrière-plan ont une incidence négative sur la hiérarchisation StorSimple.

    ![Console d’administration de Backup Exec, paramètres de vérification des options de sauvegarde](./media/storsimple-configure-backup-target-using-backup-exec/image17.png)

7.  Une fois que vous avez configuré le reste de vos options de sauvegarde conformément à vos besoins, sélectionnez **OK** pour terminer.

## <a name="set-up-storsimple-as-a-secondary-backup-target"></a>Configurer StorSimple comme cible de sauvegarde secondaire

> [!NOTE]
>En cas de restauration de données à partir d’une sauvegarde qui a été hiérarchisée vers le cloud, la restauration se produit à la vitesse du cloud.

Dans ce modèle, vous devez disposer d’un média de stockage (autre que StorSimple) faisant office de cache temporaire. Par exemple, vous pouvez utiliser une matrice redondante de volumes de disques indépendants (RAID) pour prendre en charge l’espace, l’entrée/la sortie (E/S) et la bande passante. Nous vous recommandons d’utiliser les valeurs RAID 5, 50 et 10.

La figure ci-après illustre des volumes locaux de rétention à court terme (sur le serveur) et des volumes d’archive de rétention à long terme classiques. Dans ce scénario, toutes les sauvegardes s’exécutent sur le volume RAID local (sur le serveur). Ces sauvegardes sont régulièrement dupliquées et archivées sur un volume d’archive. Veillez à dimensionner votre volume RAID local (sur le serveur) conformément aux exigences en matière de performances et de capacité de rétention à court terme.

### <a name="storsimple-as-a-secondary-backup-target-gfs-example"></a>Exemple GFS pour StorSimple utilisé comme cible de sauvegarde secondaire

![Diagramme logique de l’utilisation de StorSimple comme cible de sauvegarde secondaire](./media/storsimple-configure-backup-target-using-backup-exec/secondarybackuptargetdiagram.png)

Le tableau suivant montre comment configurer des sauvegardes à exécuter sur les disques locaux et StorSimple. Il inclut les besoins en capacité individuelle et totale.

### <a name="backup-configuration-and-capacity-requirements"></a>Configuration des sauvegardes et exigences de capacité

| Type de sauvegarde et rétention | Stockage configuré | Taille (Tio) | Multiplicateur GFS | Capacité totale\* (Tio) |
|---|---|---|---|---|
| Semaine 1 (sauvegardes complètes et incrémentielles) |Disque local (court terme)| 1 | 1 | 1 |
| StorSimple semaines 2 à 4 |Disque StorSimple (long terme) | 1 | 4 | 4 |
| Complète mensuelle |Disque StorSimple (long terme) | 1 | 12 | 12 |
| Complète annuelle |Disque StorSimple (long terme) | 1 | 1 | 1 |
|Exigence en matière de taille des volumes GFS |  |  |  | 18*|

\* La capacité totale inclut 17 Tio de disques StorSimple et 1 Tio de volume RAID local.


### <a name="gfs-example-schedule-gfs-rotation-weekly-monthly-and-yearly-schedule"></a>Exemple de planification GFS : planification de rotation GFS hebdomadaire, mensuelle et annuelle

| Week | Complète | Incrémentielle jour 1 | Incrémentielle jour 2 | Incrémentielle jour 3 | Incrémentielle jour 4 | Incrémentielle jour 5 |
|---|---|---|---|---|---|---|
| Semaine 1 | Volume RAID local  | Volume RAID local | Volume RAID local | Volume RAID local | Volume RAID local | Volume RAID local |
| Semaine 2 | StorSimple semaines 2 à 4 |   |   |   |   |   |
| Semaine 3 | StorSimple semaines 2 à 4 |   |   |   |   |   |
| Semaine 4 | StorSimple semaines 2 à 4 |   |   |   |   |   |
| Mensuelle | StorSimple mensuelle |   |   |   |   |   |
| Annuelle | StorSimple annuelle  |   |   |   |   |   |


### <a name="assign-storsimple-volumes-to-a-backup-exec-archive-and-deduplication-job"></a>Attribuer des volumes StorSimple à un travail d’archivage et de déduplication Backup Exec

#### <a name="to-assign-storsimple-volumes-to-a-backup-exec-archive-and-duplication-job"></a>Pour attribuer des volumes StorSimple à un travail d’archivage et de duplication Backup Exec

1.  Dans la console d’administration Backup Exec, cliquez avec le bouton droit sur le travail que vous souhaitez archiver sur un volume StorSimple, puis sélectionnez **Backup Definition Properties (Propriétés de définition de sauvegarde)**  > **Edit (Modifier)** .

    ![Console d’administration de Backup Exec, onglet des propriétés de définition de sauvegarde](./media/storsimple-configure-backup-target-using-backup-exec/image19.png)

2.  Sélectionnez **Add Stage (Ajouter une étape)**  > **Duplicate to Disk (Dupliquer sur le disque)**  > **Edit (Modifier)** .

    ![Console d’administration de Backup Exec, ajouter une étape](./media/storsimple-configure-backup-target-using-backup-exec/image20.png)

3.  Dans la boîte de dialogue **Duplicate Options (Options de duplication)** , sélectionnez les valeurs que vous souhaitez utiliser pour **Source** et **Schedule (Planification)** .

    ![Console d’administration de Backup Exec, propriétés de définition de sauvegarde et options de duplication](./media/storsimple-configure-backup-target-using-backup-exec/image21.png)

4.  Dans la liste déroulante **Storage (Stockage)** , sélectionnez le volume StorSimple sur lequel vous voulez que le travail d’archivage stocke les données.

    ![Capture d’écran montrant la liste dans laquelle vous devez sélectionner le stockage.](./media/storsimple-configure-backup-target-using-backup-exec/image22.png)

5.  Sélectionnez **Verify (Vérifier)** , puis cochez la case **Do not verify data for this job (Ne pas vérifier les données de ce travail)** .

    ![Capture d’écran montrant où vous sélectionnez l’option Do not verify data for this job.](./media/storsimple-configure-backup-target-using-backup-exec/image23.png)

6.  Sélectionnez **OK**.

    ![Capture d’écran montrant les propriétés de définition de sauvegarde.](./media/storsimple-configure-backup-target-using-backup-exec/image24.png)

7.  Dans la colonne **Backup (Sauvegarde)** , ajoutez une nouvelle étape. Pour la source, utilisez l’option **incrémentielle**. Pour la cible, choisissez un volume StorSimple sur lequel le travail de sauvegarde incrémentielle est archivé. Répétez les étapes 1 à 6.

## <a name="storsimple-cloud-snapshots"></a>Captures instantanées cloud StorSimple

Les captures instantanées cloud StorSimple protègent les données qui se trouvent sur votre appareil StorSimple. La création d’une capture instantanée cloud est équivalente à l’envoi des bandes de sauvegarde locales vers une installation hors site. Si vous utilisez le stockage géoredondant Azure, la création d’une capture instantanée cloud équivaut à l’envoi des bandes de sauvegarde vers plusieurs sites. Si vous devez restaurer un appareil après un sinistre, vous pouvez mettre un autre appareil StorSimple en ligne et procéder à un basculement. Après le basculement, vous serez ainsi en mesure d’accéder aux données (à la vitesse du cloud) à partir de la capture instantanée cloud la plus récente.

La section ci-après décrit comment créer un bref script pour déclencher et supprimer des captures instantanées cloud StorSimple au cours du post-traitement de sauvegarde.

> [!NOTE]
> Les captures instantanées qui sont créées manuellement ou par programme ne suivent pas la stratégie d’expiration des captures instantanées StorSimple. Ces captures instantanées doivent être supprimées manuellement ou par programme.

### <a name="start-and-delete-cloud-snapshots-by-using-a-script"></a>Démarrer et supprimer des captures instantanées cloud avec un script

> [!NOTE]
> Évaluez soigneusement les répercussions sur la conformité et sur la rétention des données avant de supprimer une capture instantanée StorSimple. Pour plus d’informations sur l’exécution d’un script après la sauvegarde, consultez la [documentation Backup Exec](https://www.veritas.com/support/en_US/article.100032497.html).

### <a name="backup-lifecycle"></a>Cycle de vie de sauvegarde

![Diagramme du cycle de vie de sauvegarde](./media/storsimple-configure-backup-target-using-backup-exec/backuplifecycle.png)

### <a name="requirements"></a>Spécifications

-   Le serveur qui exécute le script doit avoir accès aux ressources du cloud Azure.
-   Le compte d’utilisateur doit disposer des autorisations nécessaires.
-   Une stratégie de sauvegarde StorSimple avec les volumes StorSimple associés doit être configurée mais non activée.
-   Vous devez disposer du nom de la ressource StorSimple, de la clé d’inscription, du nom de l’appareil et de l’ID de la stratégie de sauvegarde.

### <a name="to-start-or-delete-a-cloud-snapshot"></a>Pour démarrer ou supprimer une capture instantanée cloud

1. [Installez Azure PowerShell](/powershell/azure/).
2. Téléchargez et installez le script PowerShell [Manage-CloudSnapshots.ps1](https://github.com/anoobbacker/storsimpledevicemgmttools/blob/master/Manage-CloudSnapshots.ps1).
3. Sur le serveur qui exécute le script, exécutez PowerShell en tant qu’administrateur. Vérifiez que vous exécutez le script avec `-WhatIf $true` pour voir les modifications qu’il apporte. Une fois la validation terminée, passez `-WhatIf $false`. Exécutez la commande ci-dessous :
   ```powershell
   .\Manage-CloudSnapshots.ps1 -SubscriptionId [Subscription Id] -TenantId [Tenant ID] -ResourceGroupName [Resource Group Name] -ManagerName [StorSimple Device Manager Name] -DeviceName [device name] -BackupPolicyName [backup policyname] -RetentionInDays [Retention days] -WhatIf [$true or $false]
   ```
4. Ajoutez le script à votre travail de sauvegarde dans Backup Exec, en modifiant vos commandes de prétraitement et de post-traitement des options de travail Backup Exec.

   ![Console Backup Exec, options de sauvegarde, onglet de commandes de prétraitement et de post-traitement](./media/storsimple-configure-backup-target-using-backup-exec/image25.png)

> [!NOTE]
> Nous vous recommandons d’exécuter votre stratégie de sauvegarde de captures instantanées cloud StorSimple à la fin de votre travail de sauvegarde quotidien sous la forme d’un script de post-traitement. Pour plus d’informations sur la façon de sauvegarder et de restaurer votre environnement d’application de sauvegarde pour vous aider à atteindre vos objectifs RPO et RTO, consultez votre concepteur d’architecture de sauvegarde.

## <a name="storsimple-as-a-restore-source"></a>Utiliser StorSimple comme source de restauration

Les restaurations à partir d’un appareil StorSimple fonctionnent comme les restaurations effectuées à partir de n’importe quel dispositif de stockage de bloc. Les restaurations de données hiérarchisées vers le cloud se produisent à la vitesse du cloud. Pour les données locales, les restaurations sont effectuées à la vitesse du disque local de l’appareil. Pour plus d’informations sur l’exécution d’une restauration, consultez la documentation Backup Exec. Nous vous recommandons de respecter les meilleures pratiques de Backup Exec relatives à la restauration.

## <a name="storsimple-failover-and-disaster-recovery"></a>Basculement et récupération d’urgence StorSimple

> [!NOTE]
> Pour les scénarios relatifs aux cibles de sauvegarde, l’appliance cloud StorSimple n’est pas prise en charge en tant que cible de restauration.

Un sinistre peut être dû à plusieurs facteurs. Le tableau ci-après répertorie les scénarios de récupération d’urgence courants.

| Scénario | Impact | Procédure de récupération | Notes |
|---|---|---|---|
| Défaillance d’appareil StorSimple | Les opérations de sauvegarde et de restauration sont interrompues. | Remplacez l’appareil défaillant et effectuez [un basculement et une récupération d’urgence StorSimple](./storsimple-8000-device-failover-disaster-recovery.md). | Si vous devez procéder à une restauration après la récupération de l’appareil, les jeux de données de travail complets sont récupérés sur le nouvel appareil à partir du cloud. Toutes les opérations sont exécutées à la vitesse du cloud. Le processus de nouvelle analyse des index et catalogues peut entraîner l’analyse et l’extraction de tous les jeux de données de sauvegarde du niveau cloud vers un niveau d’appareil local, ce qui peut prendre un certain temps. |
| Défaillance du serveur Backup Exec | Les opérations de sauvegarde et de restauration sont interrompues. | Régénérez le serveur de sauvegarde et effectuez la restauration de base de données en suivant les instructions de l’article [How to do a manual Backup and Restore of Backup Exec (BEDB) database (Procédure de sauvegarde et de restauration manuelle d’une base de données Backup Exec (BEDB))](http://www.veritas.com/docs/000041083). | Vous devez reconstruire ou restaurer le serveur Backup Exec sur le site de récupération d’urgence. Restaurez la base de données à partir du point de défaillance le plus récent. Si la base de données Backup Exec restaurée n’est pas synchronisée avec vos derniers travaux de sauvegarde, vous devez procéder à une indexation et à un catalogage. Cette nouvelle analyse des index et catalogues peut entraîner l’analyse et l’extraction de tous les jeux de données de sauvegarde du niveau cloud vers un niveau d’appareil local. Ce processus peut donc prendre un certain temps. |
| Défaillance du site entraînant la perte du serveur de sauvegarde et de l’appareil StorSimple | Les opérations de sauvegarde et de restauration sont interrompues. | Commencez par restaurer StorSimple, puis restaurez Backup Exec. | Commencez par restaurer StorSimple, puis restaurez Backup Exec. Si vous devez procéder à une restauration après la récupération de l’appareil, les jeux de données de travail complets sont récupérés sur le nouvel appareil à partir du cloud. Toutes les opérations sont exécutées à la vitesse du cloud. |

## <a name="references"></a>References

Les documents référencés dans cet article sont les suivants :

- [StorSimple multipath I/O setup (Configuration de StorSimple MPIO)](./storsimple-8000-configure-mpio-windows-server.md)
- [Storage scenarios: Thin provisioning (Scénarios de stockage : allocation dynamique)](/windows-hardware/drivers/storage/thin-provisioning)
- [Using GPT drives (Utilisation de disques de table de partition GUID)](/previous-versions/windows/hardware/design/dn653580(v=vs.85)#EHD)
- [Set up shadow copies for shared folders (Configurer des clichés instantanés de dossiers partagés)](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc771893(v=ws.11))

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur la [restauration à partir d’un jeu de sauvegarde](./storsimple-8000-restore-from-backup-set-u2.md).
- En savoir plus sur l’exécution [d’un basculement et d’une récupération d’urgence pour un appareil](./storsimple-8000-device-failover-disaster-recovery.md).