---
title: Installer le serveur de sauvegarde Azure sur Azure Stack
description: Dans cet article, vous allez découvrir comment utiliser un serveur de sauvegarde Azure pour protéger ou sauvegarder les charges de travail dans Azure Stack.
ms.topic: conceptual
ms.date: 01/31/2019
ms.openlocfilehash: 7153e2ff03a4f78ee1cc92ca04054fb2955d11a5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90970232"
---
# <a name="install-azure-backup-server-on-azure-stack"></a>Installer le serveur de sauvegarde Azure sur Azure Stack

Cet article explique comment installer un serveur de sauvegarde Azure sur Azure Stack. Un serveur de sauvegarde Azure vous permet de protéger des charges de travail IaaS (par exemple, des ordinateurs virtuels) lorsqu’elles sont en cours d’exécution sur Azure Stack. Le fait d’utiliser un serveur de sauvegarde Azure pour protéger vos charges de travail présente un avantage. Vous pouvez en effet gérer la protection de toutes les charges de travail depuis une seule et même console.

> [!NOTE]
> Pour découvrir les fonctionnalités de sécurité, voir la [documentation sur les fonctionnalités de sécurité de la sauvegarde Microsoft Azure](backup-azure-security-feature.md).
>

## <a name="azure-backup-server-protection-matrix"></a>Matrice de protection du serveur de sauvegarde Azure

Le serveur de sauvegarde Azure protège les charges de travail de machine virtuelle Azure Stack suivantes.

| Source de données protégée | Protection et récupération |
| --------------------- | ----------------------- |
| Canal semi-annuel Windows Server - Centre de données/Entreprise/Standard | Volumes, fichiers, dossiers |
| Windows Server 2016 - Centre de données/Entreprise/Standard | Volumes, fichiers, dossiers |
| Windows Server 2012 R2 - Centre de données/Entreprise/Standard | Volumes, fichiers, dossiers |
| Windows Server 2012 - Centre de données/Entreprise/Standard | Volumes, fichiers, dossiers |
| Windows Server 2008 R2 - Centre de données/Entreprise/Standard | Volumes, fichiers, dossiers |
| SQL Server 2016 | Base de données |
| SQL Server 2014 | Base de données |
| SQL Server 2012 SP1 | Base de données |
| SharePoint 2016 | Batterie de serveurs, base de données, serveur frontal, serveur web |
| SharePoint 2013 | Batterie de serveurs, base de données, serveur frontal, serveur web |
| SharePoint 2010 | Batterie de serveurs, base de données, serveur frontal, serveur web |

## <a name="prerequisites-for-the-azure-backup-server-environment"></a>Conditions préalables pour l’environnement du serveur de sauvegarde Azure

Au moment d’installer un serveur de sauvegarde Azure dans votre environnement Azure Stack, tenez compte des suggestions de cette section. Même si le programme d’installation du serveur de sauvegarde Azure vérifie que votre environnement suit les conditions préalables requises, vous gagnerez du temps en préparant l’installation.

### <a name="determining-size-of-virtual-machine"></a>Détermination de la taille de machine virtuelle

Pour exécuter un serveur de sauvegarde Azure sur une machine virtuelle Azure Stack, optez pour la taille A2 ou une taille supérieure. Pour obtenir de l’assistance pour le choix de la taille d’une machine virtuelle, téléchargez le [calculateur de taille de machine virtuelle Azure Stack](https://www.microsoft.com/download/details.aspx?id=56832).

### <a name="virtual-networks-on-azure-stack-virtual-machines"></a>Réseaux virtuels sur machines virtuelles Azure Stack

Toutes les machines virtuelles utilisées dans une charge de travail Azure Stack doivent appartenir aux mêmes réseau virtuel et abonnement Azure.

### <a name="azure-backup-server-vm-performance"></a>Performances de machine virtuelle de serveur de sauvegarde Azure

En cas de partage avec d’autres machines virtuelles, la taille du compte de stockage et les limites d’E/S par seconde ont une incidence sur les performances de la machine virtuelle d’un serveur de sauvegarde Azure. C’est pourquoi vous devez utiliser un compte de stockage distinct pour la machine virtuelle du serveur de sauvegarde Azure. L’agent Sauvegarde Azure s’exécutant sur le serveur de sauvegarde Azure a besoin d’un stockage temporaire pour :

- son propre usage (emplacement de cache) ;
- les données restaurées à partir du cloud (zone de transit locale).

### <a name="configuring-azure-backup-temporary-disk-storage"></a>Configuration d’un stockage sur disque temporaire pour Sauvegarde Azure

Chaque machine virtuelle Azure Stack est fournie avec un stockage sur disque temporaire qui est disponible pour l’utilisateur en tant que volume `D:\`. Il est possible de configurer la zone de transit locale dont Sauvegarde Azure a besoin de façon à ce qu’elle réside sur `D:\`, et de placer le cache sur `C:\`. De cette façon, aucun stockage ne doit être effectué à l’écart des disques de données attachés à la machine virtuelle d’un serveur de sauvegarde Azure.

### <a name="storing-backup-data-on-local-disk-and-in-azure"></a>Stockage de données de sauvegarde sur disque local et dans Azure

Un serveur de sauvegarde Azure stocke les données de sauvegarde sur des disques Azure attachés à la machine virtuelle à des fins de récupération opérationnelle. Une fois les disques et l’espace de stockage attachés à la machine virtuelle, le serveur de sauvegarde Azure gère le stockage pour vous. Le volume de stockage de données de sauvegarde varie selon le nombre et la taille des disques attachés à chaque [machine virtuelle Azure Stack](/azure-stack/user/azure-stack-storage-overview). Chaque taille de machine virtuelle Azure Stack est associée à un nombre maximal de disques pouvant être attachés à la machine virtuelle. Par exemple, la taille A2 correspond à quatre disques. La taille A3 correspond à huit disques. La taille A4 correspond à seize disques. Ici encore, la taille et le nombre de disques déterminent la capacité totale du pool de stockage de sauvegarde.

> [!IMPORTANT]
> Vous ne devez **pas** conserver des données de récupération (sauvegarde) opérationnelle sur des disques attachés à un serveur de sauvegarde Azure pendant plus de cinq jours.
>

Stocker des données de sauvegarde dans Azure réduit l’infrastructure de sauvegarde sur Azure Stack. Les données de plus de cinq jours doivent être stockées dans Azure.

Pour stocker des données de sauvegarde dans Azure, créez ou utilisez un coffre Recovery Services. Lors de la préparation de la sauvegarde de la charge de travail d’un serveur de sauvegarde Azure, vous [configurez le coffre Recovery Services](backup-azure-microsoft-azure-backup.md#create-a-recovery-services-vault). Une fois la configuration effectuée, chaque fois qu’une opération de sauvegarde a lieu, un point de récupération est créé dans le coffre. Chaque coffre Recovery Services conserve jusqu’à 9 999 points de récupération. En fonction du nombre de points de récupération créés et de la durée de leur conservation, vous pouvez conserver des données de sauvegarde pendant de nombreuses années. Par exemple, vous créez des points de récupération mensuellement et les conserver pendant cinq ans.

### <a name="scaling-deployment"></a>Déploiement avec mise à l’échelle

Si vous souhaitez mettre à l’échelle votre déploiement, vous disposez des options suivantes :

- Monter en puissance : augmenter la taille de la machine virtuelle du serveur de sauvegarde Azure en passant de la série A à la série D, et augmenter le stockage local [conformément aux instructions relatives à la machine virtuelle Azure Stack](/azure-stack/user/azure-stack-manage-vm-disks).
- Décharger des données : envoyer des données plus anciennes vers Azure en ne conservant que les données les plus récentes sur le stockage attaché au serveur de sauvegarde Azure.
- Effectuer un scale-out : ajouter des serveurs de sauvegarde Azure pour protéger les charges de travail.

### <a name="net-framework"></a>.NET Framework

.NET Framework 3.5 SP1 ou version ultérieure doit être installé sur l’ordinateur virtuel.

### <a name="joining-a-domain"></a>Rejoindre un domaine

Vous devez associer l’ordinateur virtuel du serveur de sauvegarde Azure à un domaine. Un utilisateur du domaine ayant des privilèges d’administrateur doit installer le serveur de sauvegarde Azure sur l’ordinateur virtuel.

## <a name="using-an-iaas-vm-in-azure-stack"></a>Utiliser un ordinateur virtuel IaaS dans Azure Stack

Lorsque vous choisissez un serveur et décidez d’en faire un serveur de sauvegarde Azure, mieux vaut commencer par une image de la galerie de Windows Server 2012 R2 Datacenter ou de Windows Server 2016 Datacenter. L’article [Créer votre première machine virtuelle Windows dans le portail Azure](../virtual-machines/windows/quick-create-portal.md?toc=/azure/virtual-machines/windows/toc.json) propose un didacticiel de prise en main avec la machine virtuelle recommandée. Configuration minimale recommandée pour la machine virtuelle serveur : A2 Standard avec deux cœurs et 3,5 Go de RAM.

La protection des charges de travail à l’aide d’Azure Backup Server peut prendre plusieurs formes. La [matrice de protection pour MABS](./backup-mabs-protection-matrix.md) permet d’expliquer ces nuances. Avant de déployer la machine, lisez cet entièrement cet article.

> [!NOTE]
> Le serveur de sauvegarde Azure est conçu pour s’exécuter sur une machine virtuelle dédiée et spécialisée. Vous ne pouvez pas installer le serveur de sauvegarde Azure sur :
>
> - Un ordinateur servant de contrôleur de domaine
> - Un ordinateur sur lequel est installé le rôle de serveur d’applications
> - Un ordinateur sur lequel Exchange Server s’exécute
> - Un ordinateur qui est un nœud de cluster

Joignez toujours le serveur de sauvegarde Azure à un domaine. Si vous envisagez de déplacer le serveur de sauvegarde Azure dans un autre domaine, installez-le tout d’abord, puis joignez-le au nouveau domaine. Une fois déployé, le serveur de sauvegarde Azure ne peut pas être déplacé dans un nouveau domaine.

[!INCLUDE [backup-create-rs-vault.md](../../includes/backup-create-rs-vault.md)]

### <a name="set-storage-replication"></a>Définir la réplication du stockage

L’option de réplication du stockage du coffre Recovery Services vous permet de choisir entre un stockage géoredondant et un stockage localement redondant. Par défaut, les coffres Recovery Services utilisent le stockage géoredondant. Si cet archivage est votre archivage principal, laissez l’option de stockage définie sur un stockage géoredondant. Choisissez un stockage localement redondant si vous souhaitez une option plus économique, mais moins durable. Pour en savoir plus sur les options de stockage [géo-redondant](../storage/common/storage-redundancy.md#geo-redundant-storage), [localement redondant](../storage/common/storage-redundancy.md#locally-redundant-storage) et [redondant interzone](../storage/common/storage-redundancy.md#zone-redundant-storage), consultez l’article [Réplication Stockage Azure](../storage/common/storage-redundancy.md).

Pour modifier le paramètre de réplication du stockage :

1. Sélectionnez votre coffre pour ouvrir le tableau de bord correspondant et le menu Paramètres. Si le menu **Paramètres** ne s’ouvre pas, sélectionnez **Tous les paramètres** dans le tableau de bord du coffre.
2. Dans le menu **Paramètres**, sélectionnez **Infrastructure de sauvegarde** > **Configuration de la sauvegarde** pour ouvrir le menu **Configuration de la sauvegarde**. Dans le menu **Configuration de la sauvegarde**, choisissez l’option de réplication de stockage pour votre coffre.

    ![Liste des archivages de sauvegarde](./media/backup-azure-vms-first-look-arm/choose-storage-configuration-rs-vault.png)

## <a name="download-azure-backup-server-installer"></a>Télécharger le programme d’installation du serveur de sauvegarde Azure

Il existe deux façons de télécharger le programme d’installation du serveur de sauvegarde Azure. Vous pouvez télécharger le programme d’installation du serveur de sauvegarde Azure depuis le [Centre de téléchargement Microsoft](https://www.microsoft.com/download/details.aspx?id=55269). Vous pouvez également télécharger le programme d’installation du serveur de sauvegarde Azure quand vous configurez un coffre Recovery Services. Les étapes suivantes vous guident tout au long du téléchargement du programme d’installation à partir du portail Azure, au moment de la configuration d’un coffre Recovery Services.

1. Depuis votre machine virtuelle Azure Stack, [connectez-vous à votre abonnement Azure dans le portail Azure](https://portal.azure.com/).
2. Dans le menu de gauche, sélectionnez **Tous les services**.

    ![Choisir l’option Tous les services dans le menu principal](./media/backup-mabs-install-azure-stack/click-all-services.png)

3. Dans la boîte de dialogue **Tous les services**, tapez *Recovery Services*. Au fur et à mesure des caractères saisis, la liste des ressources est filtrée. Cliquez sur l’option **Coffres Recovery Services** dès qu’elle apparaît.

    ![Dans la boîte de dialogue Tous les services, taper Recovery Services](./media/backup-mabs-install-azure-stack/all-services.png)

    La liste des coffres Recovery Services de l’abonnement s’affiche.

4. Dans la liste des coffres Recovery Services, sélectionnez le vôtre pour ouvrir son tableau de bord.

    ![Sélectionnez votre coffre pour ouvrir le tableau de bord](./media/backup-mabs-install-azure-stack/rs-vault-dashboard.png)

5. Dans menu Prise en main du coffre, sélectionnez **Sauvegarde** pour ouvrir l’Assistant Prise en main.

    ![Prise en main de la sauvegarde](./media/backup-mabs-install-azure-stack/getting-started-backup.png)

    Le menu de sauvegarde s’ouvre.

    ![Backup-goals-default-opened](./media/backup-mabs-install-azure-stack/getting-started-menu.png)

6. Dans le menu de sauvegarde, dans **Where is your workload running** (Où s’exécute votre charge de travail ?), sélectionnez **Local**. À partir du menu déroulant **What do you want to backup?** (Que souhaitez-vous sauvegarder ?), sélectionnez les charges de travail que vous souhaitez protéger à l’aide du serveur de sauvegarde Azure. Si vous n’êtes pas sûr des charges de travail à sélectionner, choisissez **Machines virtuelles Hyper-V**, puis sélectionnez **Préparer l’infrastructure**.

    ![en local et charges de travail comme objectifs](./media/backup-mabs-install-azure-stack/getting-started-menu-onprem-hyperv.png)

    Le menu **Préparer l’infrastructure** s’ouvre.

7. Dans le menu **Préparer l’infrastructure**, sélectionnez **Télécharger** pour ouvrir une page web et télécharger les fichiers d’installation du serveur de sauvegarde Azure.

    ![Modification de l’Assistant Mise en route](./media/backup-mabs-install-azure-stack/prepare-infrastructure.png)

    La page web de Microsoft qui héberge les fichiers téléchargeables pour le serveur de sauvegarde Azure s’ouvre.

8. Sur la page de téléchargement du serveur de sauvegarde Microsoft Azure, choisissez une langue, puis sélectionnez **Télécharger**.

    ![Le centre de téléchargement s’ouvre](./media/backup-mabs-install-azure-stack/mabs-download-center-page.png)

9. Le programme d’installation du serveur de sauvegarde Azure se compose de huit fichiers : un programme d’installation et sept fichiers .bin. Cochez **Nom de fichier** pour sélectionner tous les fichiers requis, puis sélectionnez **Suivant**. Téléchargez tous les fichiers dans le même dossier.

    ![Centre de téléchargement, fichiers sélectionnés](./media/backup-mabs-install-azure-stack/download-center-selected-files.png)

    La taille de tous les fichiers d’installation une fois téléchargés est supérieure à 3 Go. Avec un lien de téléchargement de 10 Mbits/s, le téléchargement de tous les fichiers d’installation peut prendre jusqu’à 60 minutes. Le téléchargement des fichiers s’effectue à l’emplacement indiqué.

## <a name="extract-azure-backup-server-install-files"></a>Extraire les fichiers d’installation du serveur de sauvegarde Azure

Après avoir téléchargé tous les fichiers sur votre machine virtuelle Azure Stack, accédez à l’emplacement de téléchargement. La première phase de l’installation du serveur de sauvegarde Azure consiste à extraire les fichiers.

![Télécharger le programme d’installation de MABS](./media/backup-mabs-install-azure-stack/download-mabs-installer.png)

1. Pour démarrer l’installation, dans la liste des fichiers téléchargés, sélectionnez **MicrosoftAzureBackupserverInstaller.exe**.

    > [!WARNING]
    > Au moins 4 Go d’espace libre sont nécessaires pour l’extraction des fichiers d’installation.
    >

2. Dans l’Assistant Serveur de sauvegarde Azure, cliquez sur **Suivant** pour continuer.

    ![L’Assistant Installation de Microsoft Azure Backup](./media/backup-mabs-install-azure-stack/mabs-install-wiz-1.png)

3. Choisissez le chemin d’accès des fichiers de sauvegarde du serveur Azure, puis sélectionnez **Suivant**.

   ![Sélectionner la destination des fichiers](./media/backup-mabs-install-azure-stack/mabs-install-wizard-select-destination-1.png)

4. Vérifiez l’emplacement d’extraction, puis sélectionnez **Extraire**.

   ![Vérifier l’emplacement d’extraction](./media/backup-mabs-install-azure-stack/mabs-install-wizard-extract-2.png)

5. L’Assistant extrait les fichiers et prépare le processus d’installation.

   ![L’Assistant extrait les fichiers](./media/backup-mabs-install-azure-stack/mabs-install-wizard-install-3.png)

6. Une fois le processus d’extraction terminé, sélectionnez **Terminer**. Par défaut, **setup.exe** s’exécute. Lorsque vous sélectionnez **Terminer**, Setup.exe installe le serveur de sauvegarde Microsoft Azure à l’emplacement indiqué.

   ![Le programme d’installation extrait les fichiers du serveur de sauvegarde Azure](./media/backup-mabs-install-azure-stack/mabs-install-wizard-finish-4.png)

## <a name="install-the-software-package"></a>Installer le package logiciel

Au cours de l’étape précédente, vous avez sélectionné **Terminer** pour quitter la phase d’extraction, puis vous avez démarré l’Assistant Installation du serveur de sauvegarde Azure.

![L’Assistant Installation de Sauvegarde Azure démarre](./media/backup-mabs-install-azure-stack/mabs-install-wizard-local-5.png)

Le serveur de sauvegarde Azure et Data Protection Manager partagent des lignes de code. Vous trouverez des références à Data Protection Manager et DPM dans le programme d’installation du serveur de sauvegarde Azure. Bien que le serveur de sauvegarde Azure et Data Protection Manager soient des produits différents, ils sont étroitement liés.

1. Pour lancer l’Assistant Installation, sélectionnez **Serveur Sauvegarde Microsoft Azure**.

   ![Sélectionnez Serveur Sauvegarde Microsoft Azure](./media/backup-mabs-install-azure-stack/mabs-install-wizard-local-5b.png)

2. Dans l’écran **Bienvenue**, sélectionnez **Suivant**.

    ![Serveur de sauvegarde Azure – Bienvenue](./media/backup-mabs-install-azure-stack/mabs-install-wizard-setup-6.png)

3. Dans l’écran **Vérification des conditions préalables**, sélectionnez **Vérifier** afin de déterminer si les configurations matérielles et logicielles requises pour le serveur de sauvegarde Azure sont respectées.

    ![Serveur de sauvegarde Azure – Vérification des conditions préalables](./media/backup-mabs-install-azure-stack/mabs-install-wizard-pre-check-7.png)

    Si les conditions préalables sont remplies dans l’environnement, un message indiquant que la machine présente la configuration requise s’affiche. Sélectionnez **Suivant**.  

    ![Validation des conditions préalables requises du serveur de sauvegarde Azure](./media/backup-mabs-install-azure-stack/mabs-install-wizard-pre-check-passed-8.png)

    Si votre environnement ne répond pas aux conditions préalables requises, un écran répertorie les problèmes rencontrés. Les conditions préalables qui n’étaient pas remplies sont également mentionnées dans le fichier DpmSetup.log. Tentez de résoudre les erreurs de conditions préalables, puis exécutez **Vérifier à nouveau**. Vous ne pouvez pas poursuivre l’installation tant que toutes les conditions préalables ne sont pas remplies.

    ![Absence de validation des conditions préalables requises pour l’installation du serveur de sauvegarde Azure](./media/backup-mabs-install-azure-stack/installation-errors.png)

4. Le serveur de sauvegarde Microsoft Azure requiert SQL Server. Le package d’installation du serveur de sauvegarde Azure fait partie d’une offre groupée. Il est fourni avec les fichiers binaires SQL Server appropriés. Libre à vous d’utiliser votre propre installation de SQL si vous le souhaitez. Toutefois, nous vous recommandons de laisser le programme d’installation ajouter une nouvelle instance de SQL Server. Pour vérifier que votre choix est en adéquation avec votre environnement, cliquez sur **Vérifier et installer**.

   > [!NOTE]
   > Le serveur de sauvegarde Azure ne fonctionne pas avec une instance de SQL Server distante. L’instance utilisée par le serveur de sauvegarde Azure doit être installée en local.
   >

    ![Serveur de sauvegarde Azure – Paramètres SQL](./media/backup-mabs-install-azure-stack/mabs-install-wizard-sql-install-9.png)

    Après vérification, si l’ordinateur virtuel possède les conditions préalables requises pour installer le serveur de sauvegarde Azure, sélectionnez **Suivant**.

    ![Serveur de sauvegarde Azure – Exigences respectées](./media/backup-mabs-install-azure-stack/mabs-install-wizard-sql-ready-10.png)

    Si une défaillance se produit et qu’il est suggéré de redémarrer l’ordinateur, faites-le. Après avoir redémarré l’ordinateur, redémarrez également le programme d’installation, puis, lorsque vous accédez à l’écran **Paramètres SQL**, sélectionnez **Vérifier à nouveau**.

5. Dans l’écran **Paramètres d’installation**, fournissez un emplacement pour l’installation des fichiers du serveur de sauvegarde Microsoft Azure, puis sélectionnez **Suivant**.

    ![Indiquer un emplacement d’installation des fichiers](./media/backup-mabs-install-azure-stack/mabs-install-wizard-settings-11.png)

    Un emplacement temporaire est requis pour la sauvegarde sur Azure. Vérifiez que l’emplacement temporaire occupe au moins 5 % du volume des données qu’il est prévu de sauvegarder dans le cloud. Pour la protection de disque, des disques séparés doivent être séparés une fois l’installation terminée. Pour plus d’informations sur les pools de stockage, consultez [Préparer le stockage des données](/system-center/dpm/plan-long-and-short-term-data-storage).

6. Dans l’écran **Paramètres de sécurité**, fournissez un mot de passe fort pour les comptes utilisateur locaux restreints, puis sélectionnez **Suivant**.

    ![Écran Paramètres de sécurité](./media/backup-mabs-install-azure-stack/mabs-install-wizard-security-12.png)

7. Dans l’écran **Abonnement à Microsoft Update**, indiquez si vous souhaitez utiliser *Microsoft Update* pour vérifier les mises à jour, sélectionnez **Suivant**.

   > [!NOTE]
   > Nous conseillons de faire en sorte que Windows Update soit redirigé vers Microsoft Update, qui offre une sécurité et des mises à jour importantes pour Windows et autres produits, tels que le serveur Microsoft Azure Backup.
   >

    ![Écran Abonnement à Microsoft Update](./media/backup-mabs-install-azure-stack/mabs-install-wizard-update-13.png)

8. Examinez le *Résumé des paramètres*, puis sélectionnez **Installer**.

    ![Résumé des paramètres](./media/backup-mabs-install-azure-stack/mabs-install-wizard-summary-14.png)

    Au moment où le serveur de sauvegarde Azure termine l’installation, le programme d’installation démarre immédiatement le programme d’installation de l’agent Microsoft Azure Recovery Services.

9. Le programme d’installation de l’agent Microsoft Azure Recovery Services s’ouvre et une vérification de la connexion Internet intervient alors. Si une connexion Internet est disponible, poursuivez l’installation. Si aucune connexion Internet n’est disponible, indiquez les détails du proxy pour la connexion à Internet. Une fois les paramètres du proxy spécifiés, cliquez sur **Suivant**.

    ![Configuration du proxy](./media/backup-mabs-install-azure-stack/mabs-install-wizard-proxy-15.png)

10. Pour installer l’agent Microsoft Azure Recovery Services, sélectionnez **Installer**.

    ![Installation de l’agent](./media/backup-mabs-install-azure-stack/mabs-install-wizard-mars-agent-16.png)

    L’agent Microsoft Azure Recovery Services, également appelé Azure Backup Agent, configure le serveur de sauvegarde Azure dans le coffre Recovery Services. Une fois configuré, le serveur de sauvegarde Azure sauvegarde toujours les données dans le même coffre Recovery Services.

11. À partir du moment où l’agent Microsoft Azure Recovery Services termine l’installation, sélectionnez **Suivant** pour passer à la phase suivante, l’inscription du serveur de sauvegarde Azure auprès du coffre Recovery Services.

    ![Installation de l’agent effectuée correctement](./media/backup-mabs-install-azure-stack/mabs-install-wizard-complete-16.png)

    Le programme d’installation lance **l’Assistant Inscrire un serveur**.

12. Basculez vers votre abonnement Azure et votre coffre Recovery Services. Dans le menu **Préparer l’infrastructure**, sélectionnez **Télécharger** pour télécharger les informations d’identification du coffre. Si le bouton **Télécharger** à l’étape 2 n’est pas actif, sélectionnez l’option **Déjà téléchargé ou utilisation de la dernière installation du serveur de sauvegarde Azure** pour l’activer. Le téléchargement des informations d’identification du coffre intervient et son emplacement correspond à celui auquel vous stockez les téléchargements. Il vous faut connaître cet emplacement, car vous en aurez besoin à l’étape suivante.

    ![Télécharger les informations d'identification de coffre](./media/backup-mabs-install-azure-stack/download-mars-credentials-17.png)

13. Dans le menu **Identification du coffre**, sélectionnez **Parcourir** pour rechercher les informations d’identification du coffre Recovery Services.

    ![Menu Identification du coffre](./media/backup-mabs-install-azure-stack/mabs-install-wizard-vault-id-18.png)

    Dans la boîte de dialogue **Sélectionner les informations d’identification du coffre**, accédez à l’emplacement de téléchargement, sélectionnez les informations d’identification de votre coffre, puis sélectionnez **Ouvrir**.

    Le chemin d’accès aux informations d’identification s’affiche dans le menu Identification du coffre. Cliquez sur **Suivant** pour accéder aux **Paramètres de chiffrement**.

14. Dans la boîte de dialogue **Paramètres de chiffrement**, entrez une phrase secrète pour le chiffrement de la sauvegarde et un emplacement pour stocker cette phrase, puis sélectionnez **Suivant**.

    ![Paramètres de chiffrement](./media/backup-mabs-install-azure-stack/mabs-install-wizard-encryption-19.png)

    Vous pouvez indiquer votre propre phrase secrète ou utiliser le générateur de phrases secrètes pour en créer une. Cette phrase secrète vous appartient. Microsoft ne l’enregistre pas et n’en assure pas la gestion. Pour être prêt en cas d’urgence, enregistrez votre phrase secrète dans un emplacement accessible.

    À partir du moment où sélectionnez **Suivant**, le serveur de sauvegarde Azure s’inscrit auprès du coffre Recovery Services. Le programme d’installation poursuit l’installation de SQL Server et du serveur de sauvegarde Azure.

    ![Le programme d’installation installe SQL et serveur de sauvegarde Azure](./media/backup-mabs-install-azure-stack/mabs-install-wizard-sql-still-installing-20.png)

15. Une fois l’exécution du programme d’installation terminée, l’**État** indique que tous les logiciels ont été correctement installés.

    ![Le logiciel a été correctement installé](./media/backup-mabs-install-azure-stack/mabs-install-wizard-done-22.png)

    Une fois l’installation terminée, la console et les icônes PowerShell du serveur de sauvegarde Azure sont créées sur le Bureau du serveur.

## <a name="add-backup-storage"></a>Ajouter de l’espace de stockage pour la sauvegarde

La première copie de sauvegarde est conservée sur l’espace de stockage associé à l’ordinateur du serveur de sauvegarde Azure. Pour plus d’informations sur l’ajout de disques, consultez [Ajouter un stockage de sauvegarde moderne](/system-center/dpm/add-storage).

> [!NOTE]
> Vous devez ajouter un stockage de sauvegarde même si vous prévoyez d’envoyer des données à Azure. Dans l’architecture du serveur de sauvegarde Azure, le coffre Recovery Services conserve la *deuxième* copie des données, tandis que le stockage local conserve la première copie de sauvegarde (obligatoire).
>
>

## <a name="network-connectivity"></a>Connectivité réseau

Pour que le produit fonctionne correctement, Azure Backup Server exige la connectivité au service Azure Backup. Pour vérifier que la machine est bien connectée à Azure, utilisez l’applet de commande ```Get-DPMCloudConnection``` dans la console Azure Backup Server PowerShell. Si la sortie de la cmdlet est TRUE, la connexion existe. Autrement, elle n’existe pas.

Dans le même temps, l’abonnement Azure doit être dans un état correct. Pour déterminer l’état de votre abonnement et le gérer, connectez-vous au [portail d’abonnement](https://ms.portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade).

Une fois que vous connaissez l’état de la connectivité d’Azure et de l’abonnement Azure, vous pouvez utiliser le tableau ci-dessous pour déterminer l’impact sur la fonctionnalité de sauvegarde/restauration proposée.

| État de la connectivité | Abonnement Azure | Sauvegarde sur Azure | Sauvegarde sur disque | Restauration à partir d’Azure | Restauration à partir d’un disque |
| --- | --- | --- | --- | --- | --- |
| Connecté |Actif |Autorisé |Autorisé |Autorisé |Autorisé |
| Connecté |Expiré |Arrêté |Arrêté |Autorisé |Autorisé |
| Connecté |Approvisionnement annulé |Arrêté |Arrêté |Arrêté et points de restauration Azure supprimés |Arrêté |
| Connectivité perdue depuis > 15 jours |Actif |Arrêté |Arrêté |Autorisé |Autorisé |
| Connectivité perdue depuis > 15 jours |Expiré |Arrêté |Arrêté |Autorisé |Autorisé |
| Connectivité perdue depuis > 15 jours |Approvisionnement annulé |Arrêté |Arrêté |Arrêté et points de restauration Azure supprimés |Arrêté |

### <a name="recovering-from-loss-of-connectivity"></a>Récupération après la perte de connectivité

Si vous êtes équipé d’un pare-feu ou d’un proxy qui empêche l’accès à Azure, ajoutez les adresses de domaine suivantes à la liste verte du profil de pare-feu/proxy :

- `http://www.msftncsi.com/ncsi.txt`
- \*.Microsoft.com
- \*.WindowsAzure.com
- \*.microsoftonline.com
- \*.windows.net

Une fois la connexion à Azure restaurée sur le serveur de sauvegarde Azure, l’état de l’abonnement Azure détermine les opérations qu’il est possible d’effectuer. Une fois que le serveur est **connecté**, utilisez le tableau présenté dans la section [Connectivité réseau](backup-mabs-install-azure-stack.md#network-connectivity) pour voir les opérations disponibles.

### <a name="handling-subscription-states"></a>Gestion des états d’abonnement

Il est possible de faire passer un abonnement Azure de l’état *Expiré* ou *Approvisionnement annulé* à l’état *Actif*. Si l’état de l’abonnement n’est pas *Actif* :

- Un abonnement dont l’état est *Approvisionnement annulé* perd ses fonctionnalités. En restaurant l’état *Actif* d’un abonnement, vous rétablissez ses fonctionnalités de sauvegarde et de restauration. Si les données de sauvegarde présentes sur le disque local peuvent être conservées pendant une période suffisamment longue, elles sont récupérables. Toutefois, les données de sauvegarde dans Azure sont irrémédiablement perdues une fois que l’abonnement passe à l’état *Approvisionnement annulé*.
- Un abonnement dont l’état est *Expiré* perd ses fonctionnalités. Les sauvegardes planifiées ne s’exécutent pas dès lors que l’état d’un abonnement est *Expiré*.

## <a name="troubleshooting"></a>Dépannage

Si le serveur de sauvegarde Microsoft Azure échoue et que des erreurs apparaissent pendant la phase d’installation (de sauvegarde ou de restauration), consultez le [document répertoriant les codes d’erreur](https://support.microsoft.com/kb/3041338).
Vous pouvez également vous reporter au [FAQ relatives à la sauvegarde Azure](backup-azure-backup-faq.md)

## <a name="next-steps"></a>Étapes suivantes

L’article [Préparation de votre environnement pour DPM](/system-center/dpm/prepare-environment-for-dpm) contient des informations sur les configurations de serveur de sauvegarde Azure prises en charge.

Vous pouvez utiliser les articles suivants pour mieux appréhender la notion de protection des charges de travail à l’aide du serveur de sauvegarde Microsoft Azure.

- [Sauvegarde SQL Server](./backup-mabs-sql-azure-stack.md)
- [Sauvegarde de serveur SharePoint](./backup-mabs-sharepoint-azure-stack.md)
- [Sauvegarde sur un autre serveur](backup-azure-alternate-dpm-server.md)
