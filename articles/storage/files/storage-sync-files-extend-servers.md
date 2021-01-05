---
title: 'Tutoriel : Étendre les serveurs de fichiers Windows avec Azure File Sync | Microsoft Docs'
description: Découvrez la procédure complète pour étendre les serveurs de fichiers Windows avec Azure File Sync.
author: roygara
ms.service: storage
ms.topic: tutorial
ms.date: 10/23/2018
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 1d2a0d79a5cdd53f8376c088fc986c20908575eb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91329452"
---
# <a name="tutorial-extend-windows-file-servers-with-azure-file-sync"></a>Tutoriel : Étendre les serveurs de fichiers Windows avec Azure File Sync

Cet article montre les étapes de base à suivre pour étendre la capacité de stockage d’un serveur Windows à l’aide d’Azure File Sync. Le tutoriel utilise Windows Server comme une machine virtuelle Azure, mais ce processus s’applique sans problème à vos serveurs locaux. Pour savoir comment déployer Azure File Sync dans votre propre environnement, consultez l’article [Déployer Azure File Sync](storage-sync-files-deployment-guide.md).

> [!div class="checklist"]
> * Déployer le service de synchronisation de stockage
> * Préparer Windows Server pour une utilisation avec Azure File Sync
> * Installer l’agent Azure File Sync
> * Inscrire Windows Server dans le service de synchronisation de stockage
> * Créer un groupe de synchronisation et un point de terminaison du cloud
> * Créer un point de terminaison de serveur

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="prepare-your-environment"></a>Préparation de votre environnement

Pour ce tutoriel, vous devez effectuer les opérations suivantes avant de déployer Azure File Sync :

- Créer un compte de stockage Azure et un partage de fichiers
- Configurer une machine virtuelle Windows Server 2016 Datacenter
- Préparer la machine virtuelle Windows Server pour Azure File Sync

### <a name="create-a-folder-and-txt-file"></a>Créer un dossier et un fichier .txt

Sur votre ordinateur local, créez un dossier nommé _FilesToSync_ et ajoutez un fichier texte nommé _mytestdoc.txt_. Vous chargerez ce fichier dans le partage de fichiers plus tard dans ce tutoriel.

### <a name="create-a-storage-account"></a>Créez un compte de stockage.

[!INCLUDE [storage-create-account-portal-include](../../../includes/storage-create-account-portal-include.md)]

### <a name="create-a-file-share"></a>Créer un partage de fichiers

Après avoir déployé un compte de stockage Azure, vous créez un partage de fichiers.

1. Dans le portail Azure, sélectionnez **Accéder à la ressource**.
1. Sélectionnez **Fichiers** dans le volet du compte de stockage.

    ![Sélectionner des fichiers](./media/storage-sync-files-extend-servers/click-files.png)

1. Sélectionnez **+ Partage de fichiers**.

    ![Sélectionner le bouton Ajouter un partage de fichiers](./media/storage-sync-files-extend-servers/create-file-share-portal2.png)

1. Nommez le nouveau partage de fichiers _afsfileshare_. Entrez « 1 » dans **Quota**, puis sélectionnez **Créer**. La valeur du quota peut être de 5 TiB au maximum, mais vous avez besoin uniquement de 1 Go pour ce didacticiel.

    ![Indiquer un nom et un quota pour le nouveau partage de fichiers](./media/storage-sync-files-extend-servers/create-file-share-portal3.png)

1. Sélectionnez le nouveau partage de fichiers. Dans l’emplacement du partage de fichiers, sélectionnez **Charger**.

    ![Charger un fichier](./media/storage-sync-files-extend-servers/create-file-share-portal5.png)

1. Accédez au dossier _FilesToSync_ où vous avez créé le fichier .txt, sélectionnez _mytestdoc.txt_ et sélectionnez **Charger**.

    ![Explorer le partage de fichiers](./media/storage-sync-files-extend-servers/create-file-share-portal6.png)

À ce stade, vous avez créé un compte de stockage et un partage de fichiers contenant un seul fichier. Vous déployez ensuite une machine virtuelle Azure avec Windows Server 2016 Datacenter qui représente le serveur local dans ce tutoriel.

### <a name="deploy-a-vm-and-attach-a-data-disk"></a>Déployer une machine virtuelle et attacher un disque de données

1. Accédez au portail Azure et développez le menu de gauche. Sélectionnez **Créer une ressource** dans le coin supérieur gauche.
1. Dans la zone de recherche au-dessus de la liste des ressources de la **Place de marché Azure**, recherchez **Windows Server 2016 Datacenter** et sélectionnez-le dans les résultats. Cliquez sur **Créer**.
1. Accédez à l’onglet **Fonctions de base**. Sous **Détails du projet**, sélectionnez le groupe de ressources que vous avez créé dans ce tutoriel.

   ![Entrer les informations de base concernant votre machine virtuelle dans le panneau du portail](./media/storage-sync-files-extend-servers/vm-resource-group-and-subscription.png)

1. Sous **Détails de l’instance**, indiquez un nom de machine virtuelle. Par exemple, utilisez _myVM_.
1. Ne changez pas les valeurs par défaut définies pour les paramètres **Région**, **Options de disponibilité**, **Image** et **Taille**.
1. Sous **Compte Administrateur**, indiquez un **nom d’utilisateur** et un **mot de passe** pour la machine virtuelle.
1. Sous **Règles des ports d’entrée**, choisissez **Autoriser les ports sélectionnés**, puis sélectionnez **RDP (3389)** et **HTTP** dans le menu déroulant.

1. Avant de créer la machine virtuelle, vous devez créer un disque de données.

   1. Sélectionnez **Suivant : Disques**.

      ![Ajouter des disques de données](./media/storage-sync-files-extend-servers/vm-add-data-disk.png)

   1. Sous l’onglet **Disques**, sous **Options du disque**, laissez les valeurs par défaut.
   1. Sous **DISQUES DE DONNÉES**, sélectionnez **Créer et attacher un disque**.

   1. Utilisez les valeurs par défaut pour tous les paramètres, à l’exception du paramètre **Taille (Gio)** , que vous pouvez changer à **1 Go** dans ce tutoriel.

      ![Détails du disque de données](./media/storage-sync-files-extend-servers/vm-create-new-disk-details.png)

   1. Sélectionnez **OK**.
1. Sélectionnez **Revoir + créer**.
1. Sélectionnez **Create** (Créer).

   Vous pouvez sélectionner l’icône **Notifications** pour voir la **progression du déploiement**. La création d’une machine virtuelle peut prendre plusieurs minutes.

1. Après avoir terminé le déploiement de votre machine virtuelle, sélectionnez **Accéder à la ressource**.

   ![Accéder à la ressource](./media/storage-sync-files-extend-servers/vm-gotoresource.png)

À ce stade, vous avez créé une machine virtuelle et attaché un disque de données. Connectez-vous ensuite à la machine virtuelle.

### <a name="connect-to-your-vm"></a>Connexion à votre machine virtuelle

1. Dans le portail Azure, sélectionnez **Se connecter** dans la page de propriétés de la machine virtuelle.

   ![Se connecter à une machine virtuelle Azure à partir du portail](./media/storage-sync-files-extend-servers/connect-vm.png)

1. Dans la page **Se connecter à la machine virtuelle**, conservez les options par défaut pour vous connecter par **adresse IP** sur le port 3389. Sélectionnez **Télécharger le fichier RDP**.

   ![Téléchargement du fichier RDP](./media/storage-sync-files-extend-servers/download-rdp.png)

1. Ouvrez le fichier RDP téléchargé et, à l’invite, sélectionnez **Connecter**.
1. Dans la fenêtre **Sécurité Windows**, sélectionnez **Plus de choix**, puis **Utiliser un autre compte**. Tapez le nom d’utilisateur sous la forme *localhost\username*, entrez le mot de passe créé pour la machine virtuelle, puis sélectionnez **OK**.

   ![Plus de choix](./media/storage-sync-files-extend-servers/local-host2.png)

1. Un avertissement de certificat peut s’afficher pendant le processus de connexion. Sélectionnez **Oui** ou **Continuer** pour créer la connexion.

### <a name="prepare-the-windows-server"></a>Préparer le serveur Windows Server

Pour le serveur Windows Server 2016 Datacenter, désactivez la configuration de sécurité renforcée d’Internet Explorer. Cette étape est nécessaire uniquement pour l’inscription initiale du serveur. Vous pouvez réactiver cette configuration une fois que le serveur a été inscrit.

Sur la machine virtuelle Windows Server 2016 Datacenter, le Gestionnaire de serveur s’ouvre automatiquement.  Si le Gestionnaire de serveur ne s’ouvre pas par défaut, recherchez-le dans le menu Démarrer.

1. Dans le **Gestionnaire de serveur**, sélectionnez **Serveur Local**.

   ![« Serveur local » sur le côté gauche de l’interface utilisateur du Gestionnaire de serveur](media/storage-sync-files-extend-servers/prepare-server-disable-ieesc-1.png)

1. Dans le volet **Propriétés**, sélectionnez le lien vers **Configuration de sécurité renforcée d’Internet Explorer**.  

    ![Volet « Configuration de sécurité renforcée d’Internet Explorer » dans l’interface utilisateur du Gestionnaire de serveur](media/storage-sync-files-extend-servers/prepare-server-disable-ieesc-2.png)

1. Dans la boîte de dialogue **Configuration de sécurité renforcée d’Internet Explorer**, sélectionnez **Désactivé** pour **Administrateurs** et **Utilisateurs**.

    ![Fenêtre contextuelle Configuration de sécurité renforcée d’Internet Explorer avec l’option « Désactivé » sélectionnée](media/storage-sync-files-extend-servers/prepare-server-disable-ieesc-3.png)

Vous pouvez maintenant ajouter le disque de données à la machine virtuelle.

### <a name="add-the-data-disk"></a>Ajouter le disque de données

1. Toujours sur la machine virtuelle **Windows Server 2016 Datacenter**, sélectionnez **Fichiers et services de stockage** > **Volumes** > **Disques**.

    ![Disque de données](media/storage-sync-files-extend-servers/your-disk.png)

1. Faites un clic droit sur le disque de 1 Go nommé **Msft Virtual Disk** et sélectionnez **Nouveau volume**.
1. Terminez l'Assistant. Utilisez les paramètres par défaut et notez la lettre de lecteur assignée.
1. Sélectionnez **Create** (Créer).
1. Sélectionnez **Fermer**.

   À ce stade, vous avez mis le disque en ligne et créé un volume. Ouvrez l’Explorateur de fichiers sur la machine virtuelle Windows Server pour vérifier la présence du disque de données que vous venez d’ajouter.

1. Dans l’Explorateur de fichiers sur la machine virtuelle, développez **Ce PC** et ouvrez le nouveau lecteur. Dans cet exemple, il s’agit du lecteur F: .
1. Faites un clic droit et sélectionnez **Nouveau** > **Dossier**. Nommez le dossier _FilesToSync_.
1. Ouvrez le dossier **FilesToSync**.
1. Faites un clic droit et sélectionnez **Nouveau** > **Document texte**. Nommez le fichier texte _MyTestFile_.

    ![Ajouter un nouveau fichier texte](media/storage-sync-files-extend-servers/new-file.png)

1. Fermez l’**Explorateur de fichiers** et le **Gestionnaire de serveur**.

### <a name="download-the-azure-powershell-module"></a>Télécharger le module Azure PowerShell

Ensuite, à partir de la machine virtuelle Windows Server 2016 Datacenter, vous devez installer le module Azure PowerShell sur le serveur.

1. Sur la machine virtuelle, ouvrez une fenêtre Windows PowerShell avec des privilèges élevés.
1. Exécutez la commande suivante :

   ```powershell
   Install-Module -Name Az
   ```

   > [!NOTE]
   > Si votre version de NuGet est antérieure à 2.8.5.201, vous êtes invité à télécharger et installer la dernière version de NuGet.

   Par défaut, la galerie PowerShell n’est pas configurée comme un référentiel de confiance pour PowerShellGet. La première fois que vous utilisez PSGallery, le message d’invite suivant s’affiche :

   ```output
   Untrusted repository

   You are installing the modules from an untrusted repository. If you trust this repository, change its InstallationPolicy value by running the Set-PSRepository cmdlet.

   Are you sure you want to install the modules from 'PSGallery'?
   [Y] Yes  [A] Yes to All  [N] No  [L] No to All  [S] Suspend  [?] Help (default is "N"):
   ```

1. Répondez **Oui** ou **Oui pour tous** afin de poursuivre l’installation.

Le module `Az` est un module cumulatif pour les cmdlets Azure PowerShell. Son installation permet de télécharger tous les modules Azure Resource Manager disponibles, et rend leurs cmdlets disponibles.

Vous avez terminé la configuration de votre environnement pour suivre le tutoriel. Vous êtes prêt à déployer le service de synchronisation de stockage.

## <a name="deploy-the-service"></a>Déployer le service

Pour déployer Azure File Sync, vous commencez par créer une ressource **Service de synchronisation de stockage** dans un groupe de ressources de l’abonnement sélectionné. Le service de synchronisation de stockage hérite les autorisations d’accès de l’abonnement et du groupe de ressources associés.

1. Dans le portail Azure, sélectionnez **Créer une ressource**, puis recherchez **Azure File Sync**.
1. Dans les résultats de recherche, sélectionnez **Azure File Sync**.
1. Sélectionnez **Créer** pour ouvrir l’onglet **Déployer la synchronisation du stockage**.

   ![Déployer la synchronisation du stockage](media/storage-sync-files-extend-servers/afs-info.png)

   Dans le volet qui s’ouvre, entrez les informations suivantes :

   | Valeur | Description |
   | ----- | ----- |
   | **Nom** | Nom unique (par abonnement) du service de synchronisation de stockage.<br><br>Utilisez le nom _afssyncservice02_ dans ce tutoriel. |
   | **Abonnement** | L’abonnement Azure que vous utilisez pour ce tutoriel. |
   | **Groupe de ressources** | groupe de ressources qui contient le service de synchronisation de stockage.<br><br>Utilisez le groupe _afsresgroup101918_ dans ce tutoriel. |
   | **Lieu** | USA Est |

1. Quand vous avez terminé, sélectionnez **Créer** pour déployer le **service de synchronisation de stockage**.
1. Sélectionnez l’onglet **Notifications** > **Accéder à la ressource**.

## <a name="install-the-agent"></a>Installer l’agent

L’agent Azure File Sync est un package téléchargeable qui permet à Windows Server de rester synchronisé avec un partage de fichiers Azure.

1. Sur la machine virtuelle **Windows Server 2016 Datacenter**, ouvrez **Internet Explorer**.
1. Accédez au [Centre de téléchargement Microsoft](https://go.microsoft.com/fwlink/?linkid=858257). Faites défiler jusqu’à la section **Agent Azure File Sync** et sélectionnez **Télécharger**.

   ![Téléchargement de l’agent de synchronisation](media/storage-sync-files-extend-servers/sync-agent-download.png)

1. Cochez la case **StorageSyncAgent_V3_WS2016.EXE** et sélectionnez **Suivant**.

   ![Sélectionner l’agent](media/storage-sync-files-extend-servers/select-agent.png)

1. Sélectionnez **Autoriser une fois** > **Exécuter** > **Ouvrir**.
1. Si vous ne l’avez pas déjà fait, fermez la fenêtre PowerShell.
1. Acceptez les valeurs par défaut dans l’**Assistant Configuration de l’agent de synchronisation de stockage**.
1. Sélectionnez **Installer**.
1. Sélectionnez **Terminer**.

Vous avez déployé le service Azure Sync et installé l’agent sur la machine virtuelle Windows Server 2016 Datacenter. Maintenant, vous devez inscrire la machine virtuelle auprès du service de synchronisation de stockage.

## <a name="register-windows-server"></a>Inscrire le serveur Windows Server

L’inscription de votre serveur Windows Server à un service de synchronisation de stockage a pour effet d’établir une relation d’approbation entre votre serveur (ou cluster) et le service de synchronisation de stockage. Un serveur peut être inscrit à un seul service de synchronisation de stockage. Il peut se synchroniser avec d’autres serveurs et partages de fichiers Azure qui sont eux-mêmes associés à ce service de synchronisation de stockage.

L’interface utilisateur d’inscription du serveur s’ouvre normalement automatiquement après l’installation de l’agent Azure File Sync. Si ce n’est pas le cas, ouvrez-la manuellement à partir de son emplacement de fichier : `C:\Program Files\Azure\StorageSyncAgent\ServerRegistration.exe.`

1. Une fois que l’interface utilisateur d’inscription du serveur s’est ouverte sur la machine virtuelle, sélectionnez **OK**.
1. Sélectionnez **Connexion** pour commencer.
1. Entrez les informations d’identification de votre compte Azure, puis sélectionnez **Connexion**.
1. Fournissez les informations suivantes :

   ![Capture d’écran de l’interface utilisateur d’inscription de serveur](media/storage-sync-files-extend-servers/signin.png)

   | Valeur | Description |
   | ----- | ----- |
   | **Abonnement Azure** | L’abonnement qui contient le Service de synchronisation de stockage pour ce didacticiel. |
   | **Groupe de ressources** | groupe de ressources qui contient le service de synchronisation de stockage. Utilisez le groupe _afsresgroup101918_ dans ce tutoriel. |
   | **Service de synchronisation de stockage** | Nom du service de synchronisation de stockage. Utilisez le nom _afssyncservice02_ dans ce tutoriel. |

1. Sélectionnez **Inscrire** pour terminer le processus d’inscription du serveur.
1. Dans le cadre du processus d’inscription, vous êtes invité à effectuer une nouvelle connexion. Connectez-vous et sélectionnez **Suivant**.
1. Sélectionnez **OK**.

## <a name="create-a-sync-group"></a>Créer un groupe de synchronisation

Un groupe de synchronisation définit la topologie de synchronisation d’un ensemble de fichiers. Un groupe de synchronisation doit contenir un point de terminaison cloud, qui représente un partage de fichiers Azure. Un groupe de synchronisation doit également contenir un ou plusieurs points de terminaison de serveur. Un point de terminaison de serveur représente un chemin d’accès vers un serveur inscrit. Pour créer un groupe de synchronisation :

1. Dans le [portail Azure](https://portal.azure.com/), sélectionnez **+ Groupe de synchronisation** à partir du service de synchronisation de stockage. Utilisez le nom *afssyncservice02* dans ce tutoriel.

   ![Créer un groupe de synchronisation dans le portail Azure](media/storage-sync-files-extend-servers/add-sync-group.png)

1. Entrez les informations suivantes pour créer un groupe de synchronisation avec un point de terminaison cloud :

   | Valeur | Description |
   | ----- | ----- |
   | **Nom du groupe de synchronisation** | Ce nom doit être unique au sein du service de synchronisation de stockage, mais vous pouvez choisir tout nom ayant un sens pour vous. Utilisez le groupe *afssyncgroup* dans ce tutoriel.|
   | **Abonnement** | L’abonnement pour lequel vous avez déployé le Service de synchronisation de stockage pour ce didacticiel. |
   | **Compte de stockage** | Choisissez **Sélectionner un compte de stockage**. Dans le volet qui s’affiche, sélectionnez le compte de stockage contenant le partage de fichiers Azure que vous avez créé. Utilisez le compte *afsstoracct101918* dans ce tutoriel. |
   | **Partage de fichiers Azure** | Nom du partage de fichiers Azure que vous avez créé. Utilisez le partage *afsfileshare* dans ce tutoriel. |

1. Sélectionnez **Create** (Créer).

Si vous sélectionnez votre groupe de synchronisation, vous pouvez voir que vous disposez maintenant d’un **point de terminaison cloud**.

## <a name="add-a-server-endpoint"></a>Ajouter un point de terminaison de serveur

Un point de terminaison de serveur représente un emplacement spécifique sur un serveur inscrit, par exemple, un dossier sur un volume de serveur. Pour ajouter un point de terminaison de serveur :

1. Sélectionnez le nouveau groupe de synchronisation, puis sélectionnez **Ajouter un point de terminaison de serveur**.

   ![Ajouter un nouveau point de terminaison de serveur dans le volet Groupe de synchronisation](media/storage-sync-files-extend-servers/add-server-endpoint.png)

1. Dans le volet **Ajouter un point de terminaison de serveur**, entrez les informations suivantes pour créer un point de terminaison de serveur :

   | Valeur | Description |
   | ----- | ----- |
   | **Serveur inscrit** | Nom du serveur que vous avez créé. Utilisez le serveur *afsvm101918* dans ce tutoriel. |
   | **Chemin d’accès** | Chemin Windows Server vers le lecteur que vous avez créé. Utilisez le chemin *f:\filestosync* dans ce tutoriel. |
   | **Hiérarchisation cloud** | Laissez le champ désactivé pour ce didacticiel. |
   | **Espace libre du volume** | Laissez le champ vide pour ce didacticiel. |

1. Sélectionnez **Create** (Créer).

Vos fichiers sont maintenant synchronisés entre le partage de fichiers Azure et Windows Server.

![Le Stockage Azure s’est synchronisé avec succès](media/storage-sync-files-extend-servers/files-synced-in-azurestorage.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez nettoyer les ressources que vous avez créées dans ce tutoriel, commencez par supprimer les points de terminaison du service de synchronisation du stockage. Ensuite, désinscrivez le serveur de votre service de synchronisation de stockage, supprimez les groupes de synchronisation et supprimez le service de synchronisation.

[!INCLUDE [storage-files-clean-up-portal](../../../includes/storage-files-clean-up-portal.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris les étapes de base à suivre pour étendre la capacité de stockage d’un serveur Windows à l’aide d’Azure File Sync. Pour plus de détails concernant la planification d’un déploiement Azure File Sync, consultez .

> [!div class="nextstepaction"]
> [Planification d’un déploiement de synchronisation de fichiers Azure](./storage-sync-files-planning.md)
