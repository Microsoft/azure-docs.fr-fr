---
title: Configurer un serveur de sauvegarde Azure pour Azure VMware Solution
description: Configurez votre environnement Azure VMware Solution pour sauvegarder des machines virtuelles à l’aide du serveur de sauvegarde Azure.
ms.topic: how-to
ms.date: 10/23/2020
ms.openlocfilehash: e71ec19402d22643d51f1435d1abcf56b20a290b
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92517376"
---
# <a name="set-up-azure-backup-server-for-azure-vmware-solution"></a>Configurer un serveur de sauvegarde Azure pour Azure VMware Solution

Le serveur de sauvegarde Azure contribue à votre stratégie de continuité d’activité et reprise d’activité (BCDR). Avec Azure VMware Solution, vous pouvez configurer uniquement la sauvegarde au niveau de la machine virtuelle à l’aide du serveur de sauvegarde Azure. 

Le serveur de sauvegarde Azure peut stocker des données de sauvegarde dans :

- **Un disque**  : Dans le cas d’un stockage à court terme, le serveur de sauvegarde Azure sauvegarde les données dans des pools de disques.
- **Azure**  : Pour le stockage à court terme et à long terme hors site, les données stockées du serveur de sauvegarde Azure dans des pools de disques peuvent être sauvegardées dans le Cloud de Microsoft Azure à l’aide de la Sauvegarde Azure.

Utilisez le serveur de sauvegarde Azure pour restaurer les données sur la source ou à un autre emplacement. Ainsi, si les données d’origine ne sont pas disponibles en raison de problèmes planifiés ou inattendus, vous pouvez restaurer les données à partir vers un autre emplacement.

Cet article vous aide à préparer votre environnement Azure VMware Solution pour sauvegarder des machines virtuelles à l’aide du serveur de sauvegarde Azure. Nous vous guidons tout au long des étapes suivantes : 

> [!div class="checklist"]
> * Déterminer le type et la taille recommandés de disque de machine virtuelle à utiliser.
> * Créer un coffre Recovery Services qui stocke les points de récupération.
> * Définir la réplication de stockage pour un coffre Recovery Services.
> * Ajouter du stockage au serveur de sauvegarde Azure.

## <a name="supported-vmware-features"></a>Fonctionnalités VMware prises en charge

- **Sauvegarde sans agent :** Le serveur de sauvegarde Azure ne nécessite pas l’installation d’un agent sur le serveur vCenter ou ESXi pour sauvegarder la machine virtuelle. Au lieu de cela, il vous suffit de fournir l’adresse IP ou le nom de domaine complet (FQDN), ainsi que les informations d’identification utilisées pour authentifier le serveur VMware avec le serveur de sauvegarde Azure.
- **Sauvegarde intégrée au Cloud :** Le serveur de sauvegarde Azure protège les charges de travail sur disque et dans le Cloud. Le flux de travail de sauvegarde et de récupération du serveur de sauvegarde Azure vous aide à gérer la conservation à long terme et la sauvegarde hors site.
- **Détectez et protégez les machines virtuelles gérées par vCenter :** Le serveur de sauvegarde Azure détecte et protège les machines virtuelles déployées sur un serveur vCenter ou ESXi. Le serveur de sauvegarde Azure détecte également les machines virtuelles gérées par vCenter afin que vous puissiez protéger des déploiements à grande échelle.
- **Autoprotection au niveau du dossier :** vCenter vous permet d’organiser vos machines virtuelles dans des dossiers de machines virtuelles. Le serveur de sauvegarde Azure détecte ces dossiers. Vous pouvez l’utiliser pour protéger des machines virtuelles au niveau du dossier, y compris tous les sous-dossiers. Lors de la protection de dossiers, le serveur de sauvegarde Azure protège les machines virtuelles de ce dossier, ainsi que les machines virtuelles ajoutées ultérieurement. Le serveur de sauvegarde Azure détecte les nouvelles machines virtuelles quotidiennement et les protège automatiquement. Lorsque vous organisez vos machines virtuelles dans des dossiers récursifs, le serveur de sauvegarde Azure détecte et protège automatiquement les nouvelles machines virtuelles déployées dans les dossiers récursifs.
- **Le serveur de sauvegarde Azure continue de protéger les machines virtuelles vMotion au sein du cluster :** Étant donné que les machines virtuelles sont sous vMotion pour l’équilibrage de charge au sein du cluster, le serveur de sauvegarde Azure détecte et continue automatiquement la protection des machines virtuelles.
- **Récupérer plus rapidement les fichiers nécessaires :** Le serveur de sauvegarde Azure peut récupérer des fichiers ou des dossiers à partir d’une machine virtuelle Windows sans récupérer l’ensemble de la machine virtuelle.

## <a name="limitations"></a>Limites

- Le correctif cumulatif 1 mis à jour pour le serveur de sauvegarde Azure v3 doit être installé.
- Vous ne pouvez pas sauvegarder les captures instantanées d’utilisateur avant la première sauvegarde du serveur de sauvegarde Azure. Une fois que le serveur de sauvegarde Azure a terminé la première sauvegarde, vous pouvez sauvegarder des captures instantanées d’utilisateur.
- Le serveur de sauvegarde Azure ne peut pas protéger les machines virtuelles VMware avec des disques pass-through et des mappages de périphériques bruts (pRDM).
- Le serveur de sauvegarde Azure ne peut pas détecter ou protéger VMware vApps.

Pour configurer le serveur de sauvegarde Azure pour Azure VMware Solution, vous devez effectuer les étapes suivantes :

- Configurer l’environnement et remplir les conditions préalables.
- Créez un coffre Recovery Services.
- Télécharger et installer le serveur de sauvegarde Azure.
- Ajouter du stockage au serveur de sauvegarde Azure.

### <a name="deployment-architecture"></a>Architecture de déploiement

Le serveur de sauvegarde Azure est déployé en tant que machine virtuelle Azure infrastructure as a service (IaaS) pour protéger les machines virtuelles d’Azure VMware Solution.

:::image type="content" source="media/azure-vmware-solution-backup/deploy-backup-server-azure-vmware-solution-diagram.png" alt-text="Le serveur de sauvegarde Azure est déployé en tant que machine virtuelle Azure infrastructure as a service (IaaS) pour protéger les machines virtuelles d’Azure VMware Solution." border="false":::

## <a name="prerequisites-for-the-azure-backup-server-environment"></a>Conditions préalables pour l’environnement du serveur de sauvegarde Azure

Au moment d’installer un serveur de sauvegarde Azure dans votre environnement Azure, tenez compte des suggestions de cette section.

### <a name="azure-virtual-network"></a>Réseau virtuel Azure

Assurez-vous de [configurer le réseau pour votre cloud privé VMware dans Azure](tutorial-configure-networking.md).

### <a name="determine-the-size-of-the-vm"></a>Déterminer la taille de la machine virtuelle

Suivez les instructions du didacticiel [Créer votre première machine virtuelle Windows dans le portail Azure](../virtual-machines/windows/quick-create-portal.md).  Vous allez créer la machine virtuelle dans le réseau virtuel que vous avez créé à l’étape précédente. Commencez par une image de la galerie du centre de données Windows Server 2019 pour exécuter le serveur de sauvegarde Azure. 

Le tableau récapitule le nombre maximal de charges de travail protégées pour chaque taille de machine virtuelle du serveur de sauvegarde Azure. Les informations sont basées sur des tests de performances et de mise à l’échelle internes avec des valeurs canoniques pour la taille et l’évolution de la charge de travail. La taille réelle de la charge de travail peut être supérieure mais doit être prise en charge par les disques attachés à la machine virtuelle du serveur de sauvegarde Azure.

| Charges de travail protégées maximales | Taille moyenne de la charge de travail | Évolution moyenne de la charge de travail (quotidienne) | IOPS de stockage minimales | Type/taille de disque recommandé      | Taille de machine virtuelle recommandée |
|-------------------------|-----------------------|--------------------------------|------------------|-----------------------------------|---------------------|
| 20                      | 100 Go                | Évolution nette de 5 %                   | 2 000             | HDD Standard (taille de 8 To ou plus par disque)  | A4V2       |
| 40                      | 150 Go                | Évolution nette de 10 %                  | 4 500             | SSD Premium* (taille de 1 To ou plus par disque) | DS3_V2     |
| 60                      | 200 Go                | Évolution nette de 10 %                  | 10 500            | SSD Premium * (taille de 8 To ou plus par disque) | DS3_V2     |

\* Pour récupérer les IOPS requises, utilisez des disques de taille minimale recommandée, ou de taille supérieure. Les disques de plus petite taille offrent des IOPS inférieures.

> [!NOTE]
> Le serveur de sauvegarde Azure est conçu pour s’exécuter sur un serveur dédié spécialisé. Vous ne pouvez pas installer le serveur de sauvegarde Azure sur un ordinateur qui :
> * S’exécute en tant que contrôleur de domaine.
> * A le rôle du serveur d'applications installé.
> * Est un serveur d’administration de System Center Operations Manager.
> * Exécute Exchange Server.
> * Est un nœud d’un cluster.

### <a name="disks-and-storage"></a>Disques et stockage

Le serveur de sauvegarde Azure nécessite des disques pour l’installation. 

| Condition requise                      | Taille recommandée  |
|----------------------------------|-------------------------|
| Installation du serveur de sauvegarde Azure                | Emplacement de l'installation : 3 Go<br />Lecteur des fichiers de base de données : 900 Mo<br />Lecteur système : 1 Go pour l’installation de SQL Server<br /><br />Vous aurez également besoin d’espace pour le serveur de sauvegarde Azure afin de copier le catalogue de fichiers dans un emplacement d’installation temporaire lors de l’archivage.      |
| Disque pour le pool de stockage<br />(Utilise des volumes de base, ne peut pas se trouver sur un disque dynamique) | Deux à trois fois la taille des données protégées.<br />Pour plus d’informations sur le calcul du stockage, voir [DPM Capacity Planner](https://www.microsoft.com/download/details.aspx?id=54301).   |

Pour savoir comment attacher un nouveau disque de données managées à une machine virtuelle Azure existante, consultez [Attacher un disque de données managées à une machine virtuelle Windows à l’aide du Portail Azure](../virtual-machines/windows/attach-managed-disk-portal.md).

> [!NOTE]
> Un serveur de sauvegarde Azure unique a une limite conditionnelle de 120 To pour le pool de stockage.

### <a name="store-backup-data-on-local-disk-and-in-azure"></a>Stockage de données de sauvegarde sur disque local et dans Azure

Stocker des données de sauvegarde dans Azure réduit l’infrastructure de sauvegarde sur la machine virtuelle du serveur de sauvegarde Azure. Pour la récupération opérationnelle, le serveur de sauvegarde Azure stocke les données de sauvegarde sur des disques Azure attachés à la machine virtuelle. Une fois les disques et l’espace de stockage attachés à la machine virtuelle, le serveur de sauvegarde Azure gère le stockage pour vous. Le volume de stockage varie selon le nombre et la taille des disques attachés à chaque machine virtuelle Azure. Chaque taille de machine virtuelle Azure est associée à un nombre maximal de disques pouvant être attachés à la machine virtuelle. Par exemple, A2 correspond à quatre disques, A3 à huit disques et A4 à 16 disques. Ici encore, la taille et le nombre de disques déterminent la capacité totale du pool de stockage de sauvegarde.

> [!IMPORTANT]
> Vous ne devez *pas* conserver des données de récupération opérationnelle sur des disques attachés à un serveur de sauvegarde Azure pendant plus de cinq jours. Si les données ont plus de cinq jours, stockez-les dans un coffre Recovery Services.

Pour stocker des données de sauvegarde dans Azure, créez ou utilisez un coffre Recovery Services. Lorsque vous vous préparez à sauvegarder la charge de travail du serveur de sauvegarde Azure, vous [configurez le coffre Recovery Services](#create-a-recovery-services-vault). Une fois la configuration effectuée, chaque fois qu’une opération de sauvegarde en ligne a lieu, un point de récupération est créé dans le coffre. Chaque coffre Recovery Services conserve jusqu’à 9 999 points de récupération. En fonction du nombre de points de récupération créés et de la durée de leur conservation, vous pouvez conserver des données de sauvegarde pendant de nombreuses années. Par exemple, vous pouvez créer des points de récupération mensuellement et les conserver pendant cinq ans.

> [!IMPORTANT]
> Que vous envoyiez des données de sauvegarde vers Azure ou que vous les conserviez localement, le serveur de sauvegarde Azure doit être inscrit auprès d’un coffre Recovery Services.

### <a name="scale-deployment"></a>Mettre à l'échelle le déploiement

Si vous souhaitez mettre à l’échelle votre déploiement, vous disposez des options suivantes :

- **Monter en puissance** : Augmentez la taille de la machine virtuelle du serveur de sauvegarde Azure d’une série A à une série DS3 et augmentez le stockage local.
- **Déchargement de données**  : Envoyez des données plus anciennes vers Azure en ne conservant que les données les plus récentes sur le stockage attaché à la machine du serveur de sauvegarde Azure.
- **Effectuer un scale-out** : Ajoutez plus de machines du serveur de sauvegarde Azure pour protéger les charges de travail.

### <a name="net-framework"></a>.NET Framework

.NET Framework 3.5 SP1 ou ultérieur doit être installé sur la machine virtuelle.

### <a name="join-a-domain"></a>Joindre un domaine

Vous devez associer l’ordinateur virtuel du serveur de sauvegarde Azure à un domaine. Un utilisateur du domaine ayant des privilèges d’administrateur doit installer le serveur de sauvegarde Azure sur l’ordinateur virtuel.

Le serveur de sauvegarde Azure déployé dans une machine virtuelle Azure peut sauvegarder des charges de travail sur les machines virtuelles dans Azure VMware Solution. Les charges de travail doivent se trouver dans le même domaine pour permettre l’opération de sauvegarde.

## <a name="create-a-recovery-services-vault"></a>Créer un coffre Recovery Services

Un coffre Recovery Services est une entité de stockage qui stocke les points de récupération créés au fil du temps. Il contient également les stratégies de sauvegarde associées aux éléments protégés.

1. Connectez-vous à votre abonnement sur le [portail Azure](https://portal.azure.com/).

1. Dans le menu de gauche, sélectionnez **Tous les services** .

   ![Dans le menu de gauche, sélectionnez Tous les services.](../backup/media/backup-create-rs-vault/click-all-services.png)

1. Dans la boîte de dialogue **Tous les services** , entrez **Recovery Services** et sélectionnez **coffres Recovery Services** dans la liste.

   ![Entrez et choisissez Coffres Recovery Services.](../backup/media/backup-create-rs-vault/all-services.png)

   La liste des coffres Recovery Services de l’abonnement s’affiche.

1. Dans le tableau de bord **Coffres Recovery Services** , cliquez sur **Ajouter** .

   ![Ajoutez un coffre Recovery Services.](../backup/media/backup-create-rs-vault/add-button-create-vault.png)

   La boîte de dialogue **Coffre Recovery Services** s’ouvre.

1. Attribuez des valeurs aux champs **Nom** , **Abonnement** , **Groupe de ressources** et **Emplacement** .

   ![Configurez le coffre Recovery Services.](../backup/media/backup-create-rs-vault/create-new-vault-dialog.png)

   - **Name**  : entrez un nom convivial pour identifier le coffre. Le nom doit être unique pour l’abonnement Azure. Spécifiez un nom composé d’au moins deux caractères, mais sans dépasser 50 caractères. Il doit commencer par une lettre et ne peut être constitué que de lettres, chiffres et traits d’union.
   - **Abonnement** : choisissez l’abonnement à utiliser. Si vous êtes membre d’un seul abonnement, son nom s’affiche. Si vous ne savez pas quel abonnement utiliser, utilisez l’abonnement par défaut (suggéré). Vous ne disposez de plusieurs choix que si votre compte professionnel ou scolaire est associé à plusieurs abonnements Azure.
   - **Groupe de ressources**  : Utilisez un groupe de ressources existant ou créez-en un. Pour voir la liste des groupes de ressources disponibles dans votre abonnement, sélectionnez **Utiliser existant** , puis sélectionnez une ressource dans la liste déroulante. Pour créer un groupe de ressources, sélectionnez **Créer** et entrez le nom.
   - **Emplacement** : sélectionnez la région géographique du coffre. Pour créer un coffre afin de protéger les machines virtuelles Azure VMware Solution, le coffre *doit* se trouver dans la même région que le cloud privé Azure VMware Solution.

1. Quand vous êtes prêt à créer le coffre Recovery Services, sélectionnez **Créer** .

   ![Créez le coffre Recovery Services.](../backup/media/backup-create-rs-vault/click-create-button.png)

   La création du coffre Recovery Services peut prendre un certain temps. Surveillez les notifications d’état dans la zone **Notifications** dans l’angle supérieur droit du portail. Une fois que le coffre est créé, il apparaît dans la liste des coffres Recovery Services. Si vous ne voyez pas votre coffre, sélectionnez **Actualiser** .

   ![Actualisez la liste des coffres de sauvegarde.](../backup/media/backup-create-rs-vault/refresh-button.png)

## <a name="set-storage-replication"></a>Définir la réplication du stockage

L’option de réplication du stockage vous permet de choisir entre le stockage géoredondant (par défaut) et le stockage localement redondant. Le stockage géoredondant copie les données de votre compte de stockage vers une région secondaire, ce qui rend vos données durables. Le stockage localement redondant est une option plus économique qui n’est pas aussi durable. Pour en savoir plus sur les options de stockage localement redondant et localement redondant, consultez [Redondance du stockage Azure](../storage/common/storage-redundancy.md).

> [!IMPORTANT]
> Vous devez remplacer la valeur du paramètre **Type de réplication du stockage Localement redondant/Géoredondant** par un coffre Recovery Services avant de configurer les sauvegardes dans le coffre. Une fois que vous avez configuré les sauvegardes, l’option de modification est désactivée et vous ne pouvez pas modifier le type de réplication de stockage.

1. Dans **Coffres Recovery Services** , sélectionnez le nouveau coffre. 

1. Sous **Paramètres** , sélectionnez **Propriétés** . Dans **Configuration de la sauvegarde** sélectionnez **Mettre à jour** .

1. Choisissez le type de réplication de stockage, puis sélectionnez **Enregistrer** .

## <a name="download-and-install-the-software-package"></a>Télécharger et installer le package logiciel

Suivez les étapes de cette section pour télécharger, extraire et installer le package logiciel.

### <a name="download-the-software-package"></a>Téléchargez le package logiciel

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Si l’un de vos coffres Recovery Services est déjà ouvert, passez à l’étape suivante. Si vous n’avez aucun coffre Recovery Services ouvert, mais que vous vous trouvez dans le Portail Azure, sélectionnez **Parcourir** dans le menu principal.

   1. Dans la liste des ressources, saisissez **Recovery Services** .

   1. Au fur et à mesure de la saisie, la liste est filtrée. Lorsque vous voyez **Coffres Recovery Services** , cliquez dessus.

   ![Créer un coffre Recovery Services - Étape 1](../backup/media/backup-azure-microsoft-azure-backup/open-recovery-services-vault.png)

1. Dans la liste des archivages de Recovery Services, sélectionnez un archivage.

   Le tableau de bord de l’archivage sélectionné s'ouvre.

   ![Le tableau de bord de l’archivage sélectionné s'ouvre.](../backup/media/backup-azure-microsoft-azure-backup/vault-dashboard.png)

   L’option **Paramètres** s’ouvre par défaut. Si elle est fermée, sélectionnez **Paramètres** pour l’ouvrir.

   ![L’option Paramètres s’ouvre par défaut. Si elle est fermée, sélectionnez Paramètres pour l’ouvrir.](../backup/media/backup-azure-microsoft-azure-backup/vault-setting.png)

1. Sélectionnez **Sauvegarde** pour ouvrir l’Assistant **Prise en main** .

   ![Sélectionnez Sauvegarde pour ouvrir l’Assistant Prise en main.](../backup/media/backup-azure-microsoft-azure-backup/getting-started-backup.png)

1. Dans la fenêtre qui s’ouvre :

   1. Dans le menu **Où s’exécute votre charge de travail ?** , sélectionnez **Local** .

      :::image type="content" source="media/azure-vmware-solution-backup/deploy-mabs-on-premises-workload.png" alt-text="Le serveur de sauvegarde Azure est déployé en tant que machine virtuelle Azure infrastructure as a service (IaaS) pour protéger les machines virtuelles d’Azure VMware Solution.":::

   1. À partir du menu **Que voulez-vous sauvegarder ?** , sélectionnez les charges de travail que vous souhaitez protéger à l’aide du serveur de sauvegarde Azure.

   1. Sélectionnez **Préparer l’infrastructure** pour télécharger et installer le serveur de sauvegarde Azure et les informations d’identification du coffre.

      :::image type="content" source="media/azure-vmware-solution-backup/deploy-mabs-prepare-infrastructure.png" alt-text="Le serveur de sauvegarde Azure est déployé en tant que machine virtuelle Azure infrastructure as a service (IaaS) pour protéger les machines virtuelles d’Azure VMware Solution.":::

1. Dans la fenêtre **Préparer l’infrastructure** qui s’ouvre :

   1. Sélectionnez le lien **Télécharger** pour installer le serveur de sauvegarde Azure.

   1. 1. Sélectionnez **Déjà téléchargé ou en utilisant la dernière installation du serveur de sauvegarde Azure** , puis sélectionnez **Télécharger** pour télécharger les informations d’identification du coffre. Vous utiliserez ces informations d’identification lorsque vous enregistrerez le serveur de sauvegarde Azure dans le coffre Recovery Services. Les liens vous dirigent vers le Centre de téléchargement à partir duquel vous pouvez télécharger le package logiciel.

   :::image type="content" source="media/azure-vmware-solution-backup/deploy-mabs-prepare-infrastructure2.png" alt-text="Le serveur de sauvegarde Azure est déployé en tant que machine virtuelle Azure infrastructure as a service (IaaS) pour protéger les machines virtuelles d’Azure VMware Solution.":::

1. Sur la page de téléchargement, sélectionnez tous les fichiers et sélectionnez **Suivant** .

   > [!NOTE]
   > Vous devez télécharger tous les fichiers dans le même dossier. Puisque la taille de téléchargement de l’ensemble des fichiers est supérieure à 3Go, le téléchargement peut prendre jusqu’à 60 minutes. 

   ![Sur la page de téléchargement, sélectionnez tous les fichiers et sélectionnez Suivant.](../backup/media/backup-azure-microsoft-azure-backup/downloadcenter.png)

### <a name="extract-the-software-package"></a>Extraire le package logiciel

Si vous avez téléchargé le package logiciel sur un autre serveur, copiez les fichiers sur la machine virtuelle que vous avez créée pour déployer le serveur de sauvegarde Azure.

> [!WARNING]
> Au moins 4 Go d’espace libre sont nécessaires pour l’extraction des fichiers d’installation.

1. Une fois que vous avez téléchargé tous les fichiers, double-cliquez sur **MicrosoftAzureBackupInstaller. exe** pour ouvrir l’Assistant Installation **Sauvegarde Microsoft Azure** , puis sélectionnez **Suivant** .

1. Sélectionnez l’emplacement d’extraction des fichiers, puis sélectionnez **Suivant** .

1. Sélectionnez **Extraire** pour démarrer le processus d'extraction.

   ![Sélectionnez Extraire pour démarrer le processus d'extraction.](../backup/media/backup-azure-microsoft-azure-backup/extract/03.png)

1. Une fois l’extraction effectuée, sélectionnez l’option pour **Exécuter setup. exe** puis sélectionnez **Terminer** .

> [!TIP]
> Vous pouvez également localiser le fichier setup. exe à partir du dossier dans lequel vous avez extrait le package logiciel.

### <a name="install-the-software-package"></a>Installer le package logiciel

1. Dans la fenêtre d’installation, sous **Installer** , sélectionnez **Sauvegarde Microsoft Azure** pour ouvrir l’Assistant Installation.

   ![Dans la fenêtre d’installation, sous Installer, sélectionnez Sauvegarde Microsoft Azure pour ouvrir l’Assistant Installation.](../backup/media/backup-azure-microsoft-azure-backup/launch-screen2.png)

1. Sur l’écran de **Bienvenue** , sélectionnez **Suivant** pour passer à la page **Vérifications des conditions préalables** .

1. Sélectionnez **Vérifier à nouveau** afin de déterminer si les configurations matérielles et logicielles requises pour le serveur de sauvegarde Azure sont respectées. Si c’est le cas, sélectionnez **Suivant** .

   ![ Sélectionnez Vérifier à nouveau afin de déterminer si les configurations matérielles et logicielles requises pour le serveur de sauvegarde Azure sont respectées. Si c’est le cas, sélectionnez Suivant.](../backup/media/backup-azure-microsoft-azure-backup/prereq/prereq-screen2.png)

1. Le package d’installation du serveur de sauvegarde Azure fait partie d’une offre groupée. Il est fourni avec les fichiers binaires SQL Server appropriés requis. Lorsque vous démarrez une nouvelle installation de serveur de sauvegarde Azure, sélectionnez l’option **Installer une nouvelle instance de SQL Server avec ce programme d’installation** . Sélectionnez ensuite **Vérifier et Installer** .

   ![Le package d’installation du serveur de sauvegarde Azure fait partie d’une offre groupée. Il est fourni avec les fichiers binaires SQL Server appropriés requis.](../backup/media/backup-azure-microsoft-azure-backup/sql/01.png)

   > [!NOTE]
   > Si vous souhaitez utiliser votre propre instance de SQL Server, les versions de SQL Server prises en charge sont SQL Server 2014 SP1 ou ultérieur, 2016 et 2017. Toutes les versions de SQL Server doivent être Standard ou Entreprise 64 bits. L’instance utilisée par le serveur de sauvegarde Azure doit être installée en local uniquement, et non à distance. Si vous utilisez une instance de SQL Server existante pour le serveur de sauvegarde Azure, la configuration prend uniquement en charge l’utilisation d’ *instances nommées* de SQL Server.

   Si une défaillance se produit et qu’il est conseillé de redémarrer la machine, faites-le, puis sélectionnez **Vérifier à nouveau** . En cas de problèmes de configuration de SQL Server, reconfigurez SQL Server selon les instructions SQL Server. Réessayez ensuite d’installer ou de mettre à jour le serveur de sauvegarde Azure à l’aide de l’instance existante de SQL Server.

   **Configuration manuelle**

   Lorsque vous utilisez votre propre instance de SQL Server, veillez à ajouter builtin\Administrators au rôle d’administrateur système pour la base de données MASTER.

   **Configurer les services de rapport avec SQL Server 2017**

   Si vous utilisez votre instance de SQL Server 2017, vous devez configurer SQL Server 2017 Reporting Services (SSRS) manuellement. Après avoir configuré SSRS, veillez à affecter à la propriété **IsInitialized** de SSRS la valeur **True** . Lorsqu’elle est définie sur **True** , le serveur de sauvegarde Azure suppose que SSRS est déjà configuré et ignorera la configuration de SSRS.

   Pour vérifier l’état de la configuration de SSRS, exécutez :

   ```powershell
   $configset =Get-WmiObject –namespace 
   "root\Microsoft\SqlServer\ReportServer\RS_SSRS\v14\Admin" -class 
   MSReportServer_ConfigurationSetting -ComputerName localhost

   $configset.IsInitialized
   ```

   Utilisez les valeurs suivantes pour la configuration SSRS :

   * **Compte de service**  : **Utiliser un compte intégré** doit être **Service réseau** .
   * **URL du service web**  : **Répertoire virtuel** doit être **ReportServer_\<SQLInstanceName>** .
   * **Base de données** : **DatabaseName** doit être **ReportServer$\<SQLInstanceName>** .
   * **URL du portail web**  : **Répertoire virtuel** doit être **Reports_\<SQLInstanceName>** .

   [En savoir plus](/sql/reporting-services/report-server/configure-and-administer-a-report-server-ssrs-native-mode) sur la configuration de SSRS.

   > [!NOTE]
   > Les [Conditions des services en ligne Microsoft](https://www.microsoft.com/licensing/product-licensing/products) (OST) régissent la gestion des licences pour SQL Server utilisé comme base de données pour le serveur de sauvegarde Azure. D’après les OST, SQL Server fourni avec le serveur de sauvegarde Azure peut être utilisé uniquement comme base de données pour le serveur de sauvegarde Azure.

1. Une fois l’installation terminée, sélectionnez **Suivant** .

1. Fournissez un emplacement pour l’installation des fichiers du serveur Sauvegarde Microsoft Azure, puis sélectionnez **Suivant** .

   > [!NOTE]
   > Un emplacement temporaire est requis pour la sauvegarde sur Azure. Vérifiez que l’emplacement temporaire correspond à au moins 5 % du volume qu’il est prévu de sauvegarder dans le cloud. Pour la protection de disque, des disques séparés doivent être configurés une fois l’installation terminée. Pour plus d’informations sur les pools de stockage, consultez [Configurer des pools de stockage et le stockage sur disque](/previous-versions/system-center/system-center-2012-r2/hh758075(v=sc.12)).

   ![Fournissez un emplacement pour l’installation des fichiers du serveur de sauvegarde Microsoft Azure puis cliquez sur Suivant.](../backup/media/backup-azure-microsoft-azure-backup/space-screen.png)

1. Fournissez un mot de passe fort pour les comptes utilisateur locaux restreints et sélectionnez **Suivant** .

   ![Fournissez un mot de passe fort pour les comptes utilisateur locaux restreints et sélectionnez Suivant.](../backup/media/backup-azure-microsoft-azure-backup/security-screen.png)

1. Indiquez si vous souhaitez utiliser Microsoft Update pour vérifier les mises à jour et sélectionnez **Suivant** .

   > [!NOTE]
   > Nous conseillons de faire en sorte que Windows Update soit redirigé vers Microsoft Update, qui offre une sécurité et des mises à jour importantes pour Windows et d’autres produits tels que le serveur le serveur de sauvegarde Azure.

   ![Indiquez si vous souhaitez utiliser Microsoft Update pour vérifier les mises à jour et sélectionnez Suivant.](../backup/media/backup-azure-microsoft-azure-backup/update-opt-screen2.png)

1. Examinez le **Résumé des paramètres** et sélectionnez **Installer** .

   L’installation se déroule en plusieurs phases. 
   - La première phase installe l’agent Microsoft Azure Recovery Services.
   - La deuxième phase vérifie la connectivité à Internet. Si disponible, vous pouvez poursuivre l’installation. Si non disponible, vous devez fournir les détails du proxy pour vous connecter à Internet. 
   - La dernière phase vérifie les logiciels requis. Si un ou plusieurs logiciels ne sont pas installés, tout logiciel manquant sera installé en même temps que l’agent Microsoft Azure Recovery Services.

1. Sélectionnez **Parcourir** pour localiser les informations d’identification de votre coffre afin d’inscrire la machine dans le coffre Recovery Services, puis sélectionnez **Suivant** .

1. Sélectionnez une phrase secrète pour chiffrer ou déchiffrer les données circulant entre Azure et votre environnement local.

   > [!TIP]
   > Vous pouvez automatiquement générer une phrase secrète ou fournir votre phrase secrète d’au minimum 16 caractères.

1. Entrez l’emplacement pour enregistrer la phrase secrète, puis sélectionnez **Suivant** pour inscrire le serveur.

   > [!IMPORTANT]
   > Enregistrez la phrase secrète dans un emplacement sûr autre que le serveur local. Nous vous recommandons vivement d’utiliser Azure Key Vault pour stocker la phrase secrète.

   Une fois l’installation de l’agent Microsoft Azure Recovery Services terminée, l’étape d’installation passe à l’installation et à la configuration de SQL Server et des composants du serveur de sauvegarde Azure.

   ![Une fois l’installation de l’agent Microsoft Azure Recovery Services terminée, l’étape d’installation passe à l’installation et à la configuration de SQL Server et des composants du serveur de sauvegarde Azure.](../backup/media/backup-azure-microsoft-azure-backup/final-install/venus-installation-screen.png)

1. Une fois l’étape de l’installation terminée, sélectionnez **Fermer** .

### <a name="install-update-rollup-1"></a>Installer le correctif cumulatif 1

L’installation du correctif cumulatif 1 pour le serveur de sauvegarde Azure v3 est obligatoire avant que vous puissiez protéger les charges de travail.  Vous trouverez les correctifs de bogues et les instructions d’installation dans l’[article de la base de connaissances](https://support.microsoft.com/en-us/help/4534062/).

## <a name="add-storage-to-azure-backup-server"></a>Ajouter du stockage à un serveur de sauvegarde Azure

Le serveur de sauvegarde Azure v3 prend en charge le stockage de sauvegarde moderne qui offre :

-  Des économies de stockage de 50 %.
-  Des sauvegardes trois fois plus rapides.
-  Un stockage plus efficace.
-  Un stockage prenant en compte la charge de travail.

### <a name="volumes-in-azure-backup-server"></a>Volumes dans le serveur de sauvegarde Azure

Ajoutez les disques de données avec la capacité de stockage requise à la machine virtuelle du serveur de sauvegarde Azure s’ils ne sont pas déjà ajoutés.

Le serveur de sauvegarde Azure v3 accepte uniquement les volumes de stockage. Lorsque vous ajoutez un volume, le serveur de sauvegarde Azure formate le volume au format Resilient File System (ReFS), qui nécessite le stockage de sauvegarde moderne.

### <a name="add-volumes-to-azure-backup-server-disk-storage"></a>Ajouter des volumes au stockage sur disque du serveur de sauvegarde Azure

1. Dans le volet **Gestion** , relancer l'analyse du stockage, puis sélectionnez **Ajouter** . 

1. Sélectionnez parmi les volumes disponibles à ajouter au pool de stockage. 

1. Après avoir ajouté les volumes disponibles, donnez-leur un nom convivial pour vous aider à les gérer. 

1. Sélectionnez **OK** pour formater ces volumes au format ReFS de façon à ce que le serveur de sauvegarde Azure puisse tirer parti des avantages du stockage de sauvegarde moderne.


## <a name="next-steps"></a>Étapes suivantes

Passez au didacticiel suivant pour découvrir comment configurer la sauvegarde de machines virtuelles VMware s’exécutant sur Azure VMware Solution à l’aide du serveur de sauvegarde Azure.

> [!div class="nextstepaction"]
> [Configurer la sauvegarde des machines virtuelles de Azure VMware Solution](backup-azure-vmware-solution-virtual-machines.md)
