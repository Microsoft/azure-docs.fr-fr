---
title: Installer l’agent Microsoft Azure Recovery Services (MARS)
description: Découvrez comment installer l’agent Microsoft Azure Recovery Services (MARS) pour sauvegarder des machines Windows.
ms.topic: conceptual
ms.date: 03/03/2020
ms.openlocfilehash: fb59c245c469791233ce973b00426a127b116535
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90975304"
---
# <a name="install-the-azure-backup-mars-agent"></a>Installer l’agent MARS de sauvegarde Azure

Cet article décrit comment installer l’agent Microsoft Azure Recovery Services (MARS). MARS est également appelé agent de sauvegarde Azure.

## <a name="about-the-mars-agent"></a>À propos de l’agent MARS

La sauvegarde Azure utilise l’agent MARS pour sauvegarder les fichiers, les dossiers et l’état système des machines locales et des machines virtuelles Azure. Ces sauvegardes sont stockées dans un coffre Recovery Services dans Azure. Vous pouvez exécuter l’agent :

* Directement sur des machines Windows locales. Ces machines peuvent être directement sauvegardées dans un coffre Recovery Services dans Azure.
* Sur les machines virtuelles Azure qui exécutent Windows côte à côte avec l’extension de sauvegarde de machine virtuelle Azure. L’agent sauvegarde des fichiers et dossiers spécifiques sur la machine virtuelle.
* Sur une instance de serveur de sauvegarde Microsoft Azure (MABS) ou sur un serveur System Center Data Protection Manager (DPM). Dans ce scénario, les machines et les charges de travail sont sauvegardées dans MABS ou Data Protection Manager. Ensuite, MABS ou Data Protection Manager utilise l’agent MARS pour effectuer la sauvegarde dans un coffre dans Azure.

Les données qui sont disponibles pour la sauvegarde dépendent de là où est installé l’agent.

> [!NOTE]
> En règle générale, vous sauvegardez une machine virtuelle Azure à l’aide d’une extension de sauvegarde Azure sur la machine virtuelle. Cette méthode sauvegarde l’ensemble de la machine virtuelle. Si vous voulez sauvegarder des fichiers et dossiers spécifiques sur la machine virtuelle, installez et utilisez l’agent MARS avec l’extension. Pour plus d’informations, consultez [Architecture d’une sauvegarde de machine virtuelle Azure intégrée](backup-architecture.md#architecture-built-in-azure-vm-backup).

![Étapes du processus de sauvegarde](./media/backup-configure-vault/initial-backup-process.png)

## <a name="before-you-start"></a>Avant de commencer

* Découvrez comment le service [Sauvegarde Azure utilise l’agent MARS pour sauvegarder les ordinateurs Windows](backup-architecture.md#architecture-direct-backup-of-on-premises-windows-server-machines-or-azure-vm-files-or-folders).
* Découvrez l’[architecture de sauvegarde](backup-architecture.md#architecture-back-up-to-dpmmabs) qui exécute l’agent MARS sur un serveur MABS ou Data Protection Manager secondaire.
* Passez en revue [ce qui est pris en charge et ce qui peut être sauvegardé](backup-support-matrix-mars-agent.md) par l’agent MARS.
* Vérifiez que vous avez un compte Azure si vous devez sauvegarder un serveur ou un client dans Azure. Si vous n’avez pas de compte, vous pouvez en créer un [gratuit](https://azure.microsoft.com/free/) en quelques minutes.
* Vérifiez l’accès à Internet sur les machines à sauvegarder.
* Assurez-vous que l’utilisateur effectuant l’installation et la configuration de l’agent MARS dispose des privilèges d’administrateur local sur le serveur à protéger.

[!INCLUDE [How to create a Recovery Services vault](../../includes/backup-create-rs-vault.md)]

## <a name="modify-storage-replication"></a>Modifier la réplication du stockage

Par défaut, les coffres utilisent le [stockage géoredondant (GRS)](../storage/common/storage-redundancy.md#geo-redundant-storage).

* Si le coffre est votre principal mécanisme de sauvegarde, nous vous suggérons d’utiliser l’option GRS.
* Vous pouvez utiliser le [stockage localement redondant (LRS)](../storage/common/storage-redundancy.md#locally-redundant-storage) pour réduire les coûts de stockage Azure.

Modifiez le type de réplication de stockage :

1. Dans le nouveau coffre, dans la section **Paramètres**, sélectionnez **Propriétés**.

1. Dans la page **Propriétés**, sous **Configuration de la sauvegarde**, sélectionnez **Mettre à jour**.

1. Choisissez le type de réplication de stockage, puis sélectionnez **Enregistrer**.

    ![Mettre à jour la configuration de sauvegarde](./media/backup-afs/backup-configuration.png)

> [!NOTE]
> Vous ne pouvez pas modifier le type de réplication de stockage une fois que le coffre est configuré et qu’il contient des éléments de sauvegarde. Pour ce faire, vous devez recréer le coffre.
>

### <a name="verify-internet-access"></a>Vérifiez l’accès à Internet

Si votre machine a un accès limité à Internet, vérifiez que les paramètres de pare-feu sur la machine ou le proxy autorisent les URL et adresses IP suivantes :

* URLs
  * `www.msftncsi.com`
  * `*.Microsoft.com`
  * `*.WindowsAzure.com`
  * `*.microsoftonline.com`
  * `*.windows.net`
  * `www.msftconnecttest.com`
* Adresses IP
  * 20.190.128.0/18
  * 40.126.0.0/18

### <a name="use-azure-expressroute"></a>Utiliser Azure ExpressRoute

Vous pouvez sauvegarder vos données sur Azure ExpressRoute avec le peering public (disponible pour les anciens circuits) et le peering Microsoft. La sauvegarde sur un peering privé n’est pas prise en charge.

Pour utiliser le peering public, commencez par vérifier l’accès aux domaines et adresses suivants :

* `http://www.msftncsi.com/ncsi.txt`
* `http://www.msftconnecttest.com/connecttest.txt`
* `microsoft.com`
* `.WindowsAzure.com`
* `.microsoftonline.com`
* `.windows.net`

Pour utiliser le peering Microsoft, sélectionnez les services, les régions et les valeurs de communauté pertinentes suivants :

* Azure Active Directory (12076:5060)
* Région Azure, en fonction de la localisation de votre coffre Recovery Services
* Stockage Azure, en fonction de la localisation de votre coffre Recovery Services

Pour plus d’informations, consultez [Exigences du routage ExpressRoute](../expressroute/expressroute-routing.md).

> [!NOTE]
> Le peering public est déprécié pour les nouveaux circuits.

Toutes les adresses IP et URL précédentes utilisent le protocole HTTPS sur le port 443.

### <a name="private-endpoints"></a>Points de terminaison privés

[!INCLUDE [Private Endpoints](../../includes/backup-private-endpoints.md)]

## <a name="download-the-mars-agent"></a>Télécharger l’agent MARS

Téléchargez l’agent MARS pour pouvoir l’installer sur les machines à sauvegarder.

Si vous avez déjà installé l’agent sur des machines, veillez à exécuter la dernière version. Recherchez la dernière version dans le portail ou accédez directement au [téléchargement](https://aka.ms/azurebackup_agent).

1. Dans le coffre, sous **Démarrer**, sélectionnez **Sauvegarde**.

    ![Ouvrir l’objectif de sauvegarde](./media/backup-try-azure-backup-in-10-mins/open-backup-settings.png)

1. Sous **Où s’exécute votre charge de travail ?** , sélectionnez **Localement**. Sélectionnez cette option même si vous souhaitez installer l’agent MARS sur une machine virtuelle Azure.
1. Sous **Que voulez-vous sauvegarder ?** , sélectionnez **Fichiers et dossiers**. Vous pouvez aussi sélectionner **État système**. De nombreuses autres options sont disponibles, mais elles sont prises en charge seulement si vous exécutez un serveur de sauvegarde secondaire. Sélectionnez **Préparer l’infrastructure**.

    ![Configurer des fichiers et des dossiers](./media/backup-try-azure-backup-in-10-mins/set-file-folder.png)

1. Pour **Préparer l’infrastructure**, sous **Installer l’agent Recovery Services**, téléchargez l’agent MARS.

    ![Préparer l’infrastructure](./media/backup-try-azure-backup-in-10-mins/choose-agent-for-server-client.png)

1. Dans le menu de téléchargement, sélectionnez **Enregistrer**. Par défaut, le fichier *MARSagentinstaller.exe* est enregistré dans le dossier Téléchargements.

1. Sélectionnez **J’ai déjà téléchargé ou j’utilise déjà la dernière version de l’agent Recovery Services**, puis téléchargez les informations d’identification du coffre.

    ![Télécharger les informations d'identification de coffre](./media/backup-try-azure-backup-in-10-mins/download-vault-credentials.png)

1. Sélectionnez **Enregistrer**. Le fichier est téléchargé dans votre dossier de téléchargement. Vous ne pouvez pas le fichier d’informations d’identification du coffre.

## <a name="install-and-register-the-agent"></a>Installer et inscrire l’agent

1. Exécutez le fichier *MARSagentinstaller.exe* sur les machines à sauvegarder.
1. Dans l’Assistant Installation de l’agent MARS, sélectionnez **Paramètres d’installation**. Choisissez où installer l’agent et choisissez un emplacement pour le cache. Sélectionnez ensuite **Suivant**.
   * Sauvegarde Azure utilise le cache pour stocker les instantanés de données avant de les envoyer à Azure.
   * L’emplacement du cache doit avoir comme espace libre l’équivalent d’au moins 5 % de la taille des données à sauvegarder.

    ![Choisir les paramètres d’installation dans l’Assistant Installation de l’agent MARS](./media/backup-configure-vault/mars1.png)

1. Pour **Configuration du proxy**, spécifiez comment l’agent qui s’exécute sur la machine Windows se connecte à Internet. Sélectionnez ensuite **Suivant**.

   * Si vous utilisez un proxy personnalisé, spécifiez les paramètres et les informations d’identification du proxy, si nécessaire.
   * Souvenez-vous que l’agent doit avoir accès à des [URL spécifiques](#before-you-start).

    ![Configurer l’accès Internet dans l’Assistant MARS](./media/backup-configure-vault/mars2.png)

1. Pour **Installation**, passez en revue les prérequis et sélectionnez **Installer**.
1. Une fois l’agent installé, sélectionnez **Procéder à l’inscription**.
1. Dans l’**Assistant Inscription de serveur** > **Identification du coffre**, accédez au fichier d’informations d’identification que vous avez téléchargé et sélectionnez-le. Sélectionnez ensuite **Suivant**.

    ![Ajouter des informations d’identification de coffre à l’aide de l’Assistant Inscription de serveur](./media/backup-configure-vault/register1.png)

1. Dans la page **Paramètre de chiffrement**, spécifiez une phrase secrète pour chiffrer et déchiffrer les sauvegardes de la machine. Pour plus d’informations sur les caractères de phrase secrète autorisés, [voyez ici](backup-azure-file-folder-backup-faq.md#what-characters-are-allowed-for-the-passphrase).

    * Enregistrez la phrase secrète dans un emplacement sécurisé. Il vous sera nécessaire pour restaurer une sauvegarde.
    * En cas de perte ou d’oubli de la phrase secrète, Microsoft ne peut pas vous aider à récupérer les données de sauvegarde.

1. Sélectionnez **Terminer**. L’agent est désormais installé et votre machine est inscrite dans le coffre. Vous êtes prêt à configurer et à planifier votre sauvegarde.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [sauvegarder des ordinateurs Windows à l’aide de l’agent MARS de Sauvegarde Azure](backup-windows-with-mars-agent.md)
