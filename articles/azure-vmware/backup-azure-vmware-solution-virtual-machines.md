---
title: Sauvegarder des machines virtuelles Azure VMware Solution avec le serveur de sauvegarde Azure
description: Configurez votre environnement Azure VMware Solution pour sauvegarder des machines virtuelles à l’aide du serveur de sauvegarde Azure.
ms.topic: how-to
ms.date: 06/09/2020
ms.openlocfilehash: d4273980a134fbdaabe64215aaf0b66a53253788
ms.sourcegitcommit: d6a739ff99b2ba9f7705993cf23d4c668235719f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92495700"
---
# <a name="back-up-azure-vmware-solution-vms-with-azure-backup-server"></a>Sauvegarder des machines virtuelles Azure VMware Solution avec le serveur de sauvegarde Azure

Dans cet article, nous allons sauvegarder des machines virtuelles VMware s’exécutant sur Azure VMware Solution avec le serveur de sauvegarde Azure. D’abord, passez en revue [Configurer le serveur de sauvegarde Microsoft Azure pour Azure VMware Solution](set-up-backup-server-for-azure-vmware-solution.md).

Ensuite, nous parcourrons pas à pas toutes les procédures nécessaires pour :

> [!div class="checklist"] 
> * Configurer un canal sécurisé afin que le serveur de sauvegarde Azure puisse communiquer avec des serveurs VMware via HTTPS. 
> * Ajouter les informations d’identification du compte au serveur de sauvegarde Azure. 
> * Ajoutez le vCenter au serveur de sauvegarde Azure. 
> * Configurer un groupe de protection qui contient les machines virtuelles VMware que vous souhaitez sauvegarder, spécifier les paramètres de sauvegarde et planifier la sauvegarde.

## <a name="create-a-secure-connection-to-the-vcenter-server"></a>Créer une connexion sécurisée au serveur vCenter

Par défaut, le serveur de sauvegarde Azure communique avec les serveurs VMware via HTTPS. Pour configurer la connexion HTTPS, téléchargez le certificat de l’autorité de certification (AC) VMware et importez-le sur le serveur de sauvegarde Azure.

### <a name="set-up-the-certificate"></a>Configurer le certificat

1. Dans le navigateur, sur le serveur de sauvegarde Azure, entrez l’URL du client web vSphere.

   > [!NOTE] 
   > Si la page **Prise en main** de VMware n’apparaît pas, vérifiez les paramètres de connexion et de proxy du navigateur et réessayez.

1. Dans la page **Prise en main** de VMware, sélectionnez **Télécharger les certificats d’autorité de certification racine approuvés** .

   :::image type="content" source="../backup/media/backup-azure-backup-server-vmware/vsphere-web-client.png" alt-text="Client web vSphere":::

1. Enregistrez le fichier **download.zip** sur l’orginateur du serveur de sauvegarde Azure, puis extrayez son contenu dans le dossier **certs** , qui contient les éléments suivants :

   - Le fichier de certificat racine doté d’une extension qui commence par une séquence numérotée comme .0 et .1.
   - Le fichier CRL doté d’une extension qui commence par une séquence comme .r0 ou .r1.

1. Dans le dossier **certs** , faites un clic droit sur le fichier de certificat racine et sélectionnez **Renommer** pour transformer l’extension en **.crt** .

   L’icône du fichier est remplacée par une icône représentant un certificat racine.

1. Faites un clic droit sur le certificat racine et sélectionnez **Installer le certificat** .

1. Dans l’ **Assistant Importation de certificat** , sélectionnez **Ordinateur local** comme destination du certificat et sélectionnez **Suivant** .

   ![Page d’accueil de l’Assistant](../backup/media/backup-azure-backup-server-vmware/certificate-import-wizard1.png)

   > [!NOTE] 
   > Si demandé, confirmez que vous voulez autoriser les modifications sur l’ordinateur.

1. Sélectionnez **Placez tous les certificats dans le magasin suivant** et sélectionnez **Parcourir** pour choisir le magasin de certificats.

   ![Stockage des certificats](../backup/media/backup-azure-backup-server-vmware/cert-import-wizard-local-store.png)

1. Sélectionnez **Autorités de certification racines de confiance** comme dossier de destination et sélectionnez **OK** .

1. Passez en revue les paramètres, puis sélectionnez **Terminer** pour commencer l’importation du certificat.

   ![Vérifier que le certificat se trouve dans le dossier approprié](../backup/media/backup-azure-backup-server-vmware/cert-wizard-final-screen.png)

1. Après avoir confirmé l’importation de certificat, connectez-vous au serveur vCenter pour vérifier que votre connexion est sécurisée.

### <a name="enable-tls-12-on-azure-backup-server"></a>Activer TLS 1.2 sur le serveur de sauvegarde Azure

Sur VMware 6.7 et ultérieur, TLS est le protocole de communication activé. 

1. Copiez les paramètres de registre suivants, puis collez-les dans le Bloc-notes. Enregistrez ensuite le fichier au format TLS.REG sans l’extension .txt.

   ```text
   
   Windows Registry Editor Version 5.00
   
   [HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v2.0.50727]
   
      "SystemDefaultTlsVersions"=dword:00000001
   
      "SchUseStrongCrypto"=dword:00000001
   
   [HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\.NETFramework\v4.0.30319]
   
      "SystemDefaultTlsVersions"=dword:00000001
   
      "SchUseStrongCrypto"=dword:00000001
   
   [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v2.0.50727]
   
      "SystemDefaultTlsVersions"=dword:00000001
   
      "SchUseStrongCrypto"=dword:00000001
   
   [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319]
   
      "SystemDefaultTlsVersions"=dword:00000001
   
      "SchUseStrongCrypto"=dword:00000001
   
   ```

1. Faites un clic droit sur le fichier TLS.REG, puis sélectionnez **Fusionner** ou **Ouvrir** pour ajouter les paramètres au registre.


## <a name="add-the-account-on-azure-backup-server"></a>Ajouter le compte sur le serveur de sauvegarde Azure

1. Ouvrez le serveur de sauvegarde Azure et, dans la console du serveur de sauvegarde Azure, sélectionnez **Gestion** > **Serveurs de production** > **Gérer VMware** .

   ![Console du serveur de sauvegarde Azure](../backup/media/backup-azure-backup-server-vmware/add-vmware-credentials.png)

1. Dans la boîte de dialogue **Gérer les informations d’identification** , sélectionnez **Ajouter** .

   ![Dans la boîte de dialogue Gérer les informations d’identification, sélectionnez Ajouter.](../backup/media/backup-azure-backup-server-vmware/mabs-manage-credentials-dialog.png)

1. Dans la boîte de dialogue **Ajouter des informations d’identification** , entrez un nom et une description pour les nouvelles informations d’identification, Spécifiez le nom d’utilisateur et le mot de passe que vous avez définis sur le serveur VMware.

   > [!NOTE] 
   > Si le serveur VMware et le serveur de sauvegarde Azure ne se trouvent pas sur le même domaine, spécifiez le domaine dans le champ **Nom d’utilisateur** .

   ![Boîte de dialogue Ajouter des informations d’identification du serveur de sauvegarde Azure](../backup/media/backup-azure-backup-server-vmware/mabs-add-credential-dialog2.png)

1. Sélectionnez **Ajouter** pour ajouter les nouvelles informations d’identification.

   ![Capture d’écran montrant la boîte de dialogue Gérer les informations d’identification de serveur de sauvegarde Azure avec les nouvelles informations d’identification affichées.](../backup/media/backup-azure-backup-server-vmware/new-list-of-mabs-creds.png)

## <a name="add-the-vcenter-server-to-azure-backup-server"></a>Ajouter le serveur vCenter au serveur de sauvegarde Azure

1. Dans la console du serveur de sauvegarde Azure, sélectionnez **Gestion** > **Serveurs de production** > **Ajouter** .

   ![Ouvrir l’Assistant Ajout d’un serveur de production](../backup/media/backup-azure-backup-server-vmware/add-vcenter-to-mabs.png)

1. Sélectionnez **Serveurs VMware** , puis sélectionnez **Suivant** .

   ![Assistant Ajout d’un serveur de production](../backup/media/backup-azure-backup-server-vmware/production-server-add-wizard.png)

1. Spécifiez l’adresse IP du vCenter.

   ![Spécifier le serveur VMware](../backup/media/backup-azure-backup-server-vmware/add-vmware-server-provide-server-name.png)

1. Dans la boîte de dialogue **Port SSL** , entrez le port utilisé pour communiquer avec le vCenter.

   > [!TIP] 
   > Le port 443 est le port par défaut, mais vous pouvez le modifier si votre vCenter effectue l’écoute sur un port différent.

1. Dans la boîte de dialogue **Spécifier les informations d’identification** , sélectionnez les informations d’identification que vous avez créées dans la section précédente.

1. Sélectionnez **Ajouter** pour ajouter le vCenter à la liste des serveurs, puis sélectionnez **Suivant** .

   ![Ajouter un serveur VMware et des informations d’identification](../backup/media/backup-azure-backup-server-vmware/add-vmware-server-credentials.png)

1. Dans la page **Résumé** , sélectionnez **Ajouter** pour ajouter le vCenter au serveur de sauvegarde Azure.

   Le nouveau serveur est ajouté immédiatement. vCenter n’a pas besoin d’un agent.

   ![Ajouter le serveur VMware au serveur de sauvegarde Azure](../backup/media/backup-azure-backup-server-vmware/tasks-screen.png)

1. Dans la page **Terminer** , vérifiez les paramètres, puis sélectionnez **Fermer** .

   ![Page Terminer](../backup/media/backup-azure-backup-server-vmware/summary-screen.png)

   Vous voyez le serveur vCenter répertorié sous **Serveur de production** avec :
   - Le type **Serveur VMware** 
   - État de l’agent : **OK** 
   
      Si vous voyez l’ **État de l’agent** comme **Inconnu** , sélectionnez **Actualiser** .

## <a name="configure-a-protection-group"></a>Configurer un groupe de protection

Les groupes de protection rassemblent plusieurs machines virtuelles et appliquent les mêmes paramètres de sauvegarde et de conservation des données à toutes les machines virtuelles du groupe.

1. Dans la console du serveur de sauvegarde Azure, sélectionnez **Protection** > **Nouveau** .

   ![Ouvrir l’Assistant Création d’un nouveau groupe de protection](../backup/media/backup-azure-backup-server-vmware/open-protection-wizard.png)

1. Dans la page d’accueil de l’Assistant **Création d’un nouveau groupe de protection** , sélectionnez **Suivant** .

   ![Boîte de dialogue Assistant Création d’un nouveau groupe de protection](../backup/media/backup-azure-backup-server-vmware/protection-wizard.png)

1. Dans la page **Sélectionner le type de groupe de protection** , sélectionnez **Serveurs** , puis **Suivant** . La page **Sélectionner les membres du groupe** s’affiche.

1. Dans la page **Sélectionner les membres du groupe** , sélectionnez les machines virtuelles (ou les dossiers de machines virtuelles) que vous souhaitez sauvegarder, puis sélectionnez **Suivant** .

   > [!NOTE]
   > Lorsque vous sélectionnez un dossier ou des machines virtuelles, les dossiers à l’intérieur de ce dossier sont également sélectionnés pour la sauvegarde. Vous pouvez désactiver les dossiers ou machines virtuelles que vous ne souhaitez pas sauvegarder. Si une machine virtuelle ou un dossier est déjà en cours de sauvegarde, vous ne pouvez pas le sélectionner, ce qui garantit que des points de récupération en double ne sont pas créés pour une machine virtuelle.

   ![Sélectionner les membres du groupe](../backup/media/backup-azure-backup-server-vmware/server-add-selected-members.png)

1. Dans la page **Sélectionner la méthode de protection des données** , entrez un nom pour le groupe de protection et les paramètres de protection. 

1. Définissez la protection à court terme sur **Azure Data Box Disk** , activez la protection en ligne puis sélectionnez **Suivant** .

   ![Sélectionner la méthode de protection des données](../backup/media/backup-azure-backup-server-vmware/name-protection-group.png)

1. Spécifiez la durée pendant laquelle vous souhaitez conserver les données sauvegardées sur disque.

   - **Durée de rétention**  : Le nombre de jours pendant lesquels les points de récupération de disque sont conservés.
   - **Sauvegarde complète rapide**  : La fréquence à laquelle les points de récupération de disque sont utilisés. Pour modifier les heures ou dates des sauvegardes à court terme, sélectionnez **Modifier** .

   :::image type="content" source="media/azure-vmware-solution-backup/new-protection-group-specify-short-term-goals.png" alt-text="Client web vSphere":::

1. Dans la page **Vérifier l’allocation du stockage sur disque** , vérifiez l’espace disque fourni pour les sauvegardes de machine virtuelle.

   - Les allocations de disques recommandées dépendent de la durée de rétention spécifiée, du type de charge de travail et de la taille des données protégées. Apportez les modifications nécessaires, puis sélectionnez **Suivant** .
   - **Taille des données :** taille des données dans le groupe de protection.
   - **Espace disque :** Quantité d’espace disque recommandée pour le groupe de protection. Pour modifier ce paramètre, sélectionnez un espace légèrement supérieur au volume de croissance estimé pour chaque source de données.
   - **Détails de pool de stockage :** Affiche l’état du pool de stockage, qui inclut la taille totale et la taille restante du disque.

   :::image type="content" source="media/azure-vmware-solution-backup/review-disk-allocation.png" alt-text="Client web vSphere":::

   > [!NOTE]
   > Dans certains scénarios, la taille des données signalée est supérieure à la taille réelle de la machine virtuelle. Nous sommes conscients du problème et nous sommes actuellement en train de l’étudier.

1. Dans la page **Choisir la méthode de création d’un réplica** , spécifiez la façon dont vous souhaitez effectuer la sauvegarde initiale, puis sélectionnez **Suivant** .

   - Les valeurs **Automatiquement sur le réseau** et **Maintenant** sont définies par défaut. Si vous utilisez la valeur par défaut, spécifiez une heure creuse. Si vous choisissez **Ultérieurement** , spécifiez le jour et l’heure.
   - Pour de grandes quantités de données ou des conditions réseau peu optimales, pensez à répliquer les données hors connexion à l’aide d’un média amovible.

   ![Choisir la méthode de création d’un réplica](../backup/media/backup-azure-backup-server-vmware/replica-creation.png)

1. Pour **Options de vérification de cohérence** , indiquez comment et quand automatiser les vérifications de cohérence, puis sélectionnez **Suivant** .

   - Vous pouvez exécuter les vérifications de cohérence lorsque les données du réplica deviennent incohérentes ou en fonction d’une planification définie.
   - Si vous ne souhaitez pas configurer des vérifications de cohérence automatiques, vous pouvez exécuter une vérification manuelle à tout moment en faisant un clic droit sur le groupe de protection **Effectuer une vérification de cohérence** .

1. Dans la page **Spécifier les données de protection en ligne** , sélectionnez les machines virtuelles ou les dossiers de machines virtuelles à sauvegarder, puis sélectionnez **Suivant** . 

   > [!TIP]
   > Vous pouvez sélectionner les membres un à un ou choisir **Sélectionner tout** pour choisir tous les membres.

   ![Spécifier les données de protection en ligne](../backup/media/backup-azure-backup-server-vmware/select-data-to-protect.png)

1. Dans la page **Spécifier la planification de sauvegarde en ligne** , indiquez la fréquence à laquelle vous souhaitez sauvegarder les données du stockage local vers Azure. 

   - Des points de récupération cloud des données à générer en fonction de la planification. 
   - Une fois le point de récupération généré, il est transféré vers le coffre Recovery Services dans Azure.

   ![Spécifier la planification de sauvegarde en ligne](../backup/media/backup-azure-backup-server-vmware/online-backup-schedule.png)

1. Dans la page **Spécifier la stratégie de rétention en ligne** , sélectionnez la durée pendant laquelle vous souhaitez conserver les points de récupération créés à partir des sauvegardes dans Azure.

   - La durée de conservation des données dans Azure n’est soumise à aucune restriction.
   - La seule limite que vous devez respecter est que vous ne pouvez pas avoir plus de 9 999 points de récupération par instance protégée. Dans cet exemple, l’instance protégée est le serveur VMware.

   ![Spécifier la stratégie de rétention en ligne](../backup/media/backup-azure-backup-server-vmware/retention-policy.png)

1. Dans la page **Résumé** , vérifiez les paramètres puis sélectionnez **Créer un groupe** .

   ![Résumé des paramètres et des membres du groupe de protection](../backup/media/backup-azure-backup-server-vmware/protection-group-summary.png)

## <a name="monitor-with-the-azure-backup-server-console"></a>Surveillance avec la console du serveur de sauvegarde Azure

Après avoir configuré le groupe de protection pour sauvegarder les machines virtuelles Azure VMware Solution, vous pouvez surveiller l’état du travail et de l’alerte de sauvegarde et à l’aide de la console du serveur de sauvegarde Azure. Voici ce que vous pouvez surveiller.

- Dans la zone de tâches **Analyse** :
   - Sous **Alertes** , vous pouvez surveiller les erreurs, les avertissements et les informations générales.  Vous pouvez afficher les alertes actives et inactives et configurer des notifications par courrier électronique.
   - Sous **Tâches** , vous pouvez afficher les tâches démarrées par le serveur de sauvegarde Azure pour une source de données protégée ou un groupe de protection spécifique. Vous pouvez suivre la progression de la tâche ou vérifier les ressources consommées par les tâches.
- Dans la zone de tâches **Protection** , vous pouvez vérifier l’état des volumes et des partages dans le groupe de protection. Vous pouvez également vérifier les paramètres de configuration tels que les paramètres de récupération, l’allocation de disque et la planification de sauvegarde.
- Dans la zone de tâches **Gestion** , vous pouvez afficher les onglets **Disques, En ligne** et **Agents** pour vérifier l’état des disques dans le pool de stockage, l’inscription vers Azure et l’état de l’agent DPM déployé.

:::image type="content" source="media/azure-vmware-solution-backup/monitor-backup-jobs.png" alt-text="Client web vSphere":::

## <a name="restore-vmware-virtual-machines"></a>Restaurer les machines virtuelles VMware

Dans la console Administrateur du serveur de sauvegarde Azure, il y a deux façons de rechercher des données récupérables. Vous pouvez rechercher ou parcourir. Lorsque vous récupérez des données, vous pouvez vouloir ou non restaurer des données ou une machine virtuelle au même emplacement. Pour cette raison, le serveur de sauvegarde Azure prend en charge trois options de récupération pour les sauvegardes de machines virtuelles VMware :

- **Récupération à l’emplacement d’origine (OLR)**  : Utilisez la récupération à l’emplacement d’origine pour restaurer une machine virtuelle protégée à son emplacement d’origine. Vous ne pouvez restaurer une machine virtuelle à son emplacement d’origine que si aucun disque n’a été ajouté ou supprimé depuis la sauvegarde. Si des disques ont été ajoutés ou supprimés, vous devez utiliser un autre emplacement de récupération.
- **Récupération à un autre emplacement (ALR)**  : Utilisez cela lorsque la machine virtuelle d’origine est manquante ou que vous ne souhaitez pas déranger la machine virtuelle d’origine. Indiquez l’emplacement d’un hôte ESXi, le pool de ressources, le dossier ainsi que le magasin de données de stockage et le chemin. Pour mieux différencier la machine virtuelle restaurée de la machine virtuelle d’origine, le serveur de sauvegarde Azure ajoute *« -Récupérée »* au nom de la machine virtuelle.
- **Récupération individuelle de l’emplacement du fichier (ILR)**  : Si la machine virtuelle protégée est une machine virtuelle Windows Server, des fichiers ou dossiers individuels à l’intérieur de la machine virtuelle peuvent être récupérés à l’aide de la fonctionnalité ILR du serveur de sauvegarde Azure. Pour récupérer des fichiers individuels, reportez-vous à la procédure décrite plus loin dans cet article. La restauration d’un fichier individuel à partir d’une machine virtuelle est disponible uniquement pour les points de récupération de disque et de machine virtuelle Windows.

### <a name="restore-a-recovery-point"></a>Restaurer un point de récupération

1. Dans la console Administrateur du serveur de sauvegarde Azure, sélectionnez la vue **Récupération** . 

1. À l’aide du volet **Parcourir** , parcourez ou filtrez pour rechercher la machine virtuelle que vous souhaitez récupérer. Une fois que vous sélectionnez une machine virtuelle ou un dossier, les **Points de récupération du volet affichent les points de récupération disponibles.

   ![Points de récupération disponibles](../backup/media/restore-azure-backup-server-vmware/recovery-points.png)

1. Dans le volet **Points de récupération pour** , sélectionnez une date à laquelle un point de récupération a été pris. Les dates du calendrier en gras ont des points de récupération disponibles. Autrement, vous pouvez faire un clic droit sur la machine virtuelle et sélectionner **Afficher tous les points de récupération** puis sélectionner le point de récupération dans la liste.

   > [!NOTE] 
   > Pour une protection à court terme, sélectionnez un point de récupération sur disque pour une récupération plus rapide. Après l’expiration des points de récupération à court terme, vous ne voyez que les points de récupération **En ligne** à récupérer.

1. Avant la récupération à partir d’un point de récupération en ligne, assurez-vous que l’emplacement intermédiaire contient suffisamment d’espace libre pour héberger la taille non compressée complète de la machine virtuelle que vous souhaitez récupérer. Vous pouvez afficher ou modifier l’emplacement intermédiaire en exécutant l’ **Assistant Configurer les paramètres d’abonnement** .

   :::image type="content" source="media/azure-vmware-solution-backup/mabs-recovery-folder-settings.png" alt-text="Client web vSphere":::

1. Sélectionnez **Récupérer** pour ouvrir **l’Assistant Récupération** .

   ![Page Assistant Récupération, Vérifier la sélection à récupérer](../backup/media/restore-azure-backup-server-vmware/recovery-wizard.png)

1. Sélectionnez **Suivant** pour accéder à l’écran **Spécifier les options de récupération** . Sélectionnez de nouveau **Suivant** pour accéder à l’écran **Sélectionner le type de récupération** . 

   > [!NOTE]
   > Les charges de travail VMware ne prennent pas en charge l’activation de la limitation de bande passante réseau.

1. Dans la page **Sélectionner le type de récupération** , choisissez si vous souhaitez récupérer vers l’instance d’origine ou vers un nouvel emplacement.

   - Si vous choisissez **Récupérer sur l’instance d’origine** , vous n’avez plus besoin d’effectuer d’autres choix dans l’Assistant. Les données de l’instance d’origine sont utilisées.
   - Si vous choisissez **Récupérer en tant que machine virtuelle sur n’importe quel hôte** , sur l’écran **Spécifier la destination** , fournissez les informations pour l’ **Hôte ESXi** , le **Pool de ressources** , **le Dossier** et le **Chemin** .

   ![Page Sélectionnez le type de récupération](../backup/media/restore-azure-backup-server-vmware/recovery-type.png)

1. Dans la page **Résumé** , passez en revue vos paramètres et sélectionnez **Récupérer** pour démarrer le processus de récupération. 

   L'écran **État de la récupération** affiche la progression de l’opération de récupération.

### <a name="restore-an-individual-file-from-a-vm"></a>Restaurer un fichier individuel à partir d’une machine virtuelle

Vous pouvez restaurer des fichiers individuels à partir d’un point de récupération de machine virtuelle protégé. Cette fonctionnalité est uniquement disponible pour les machines virtuelles Windows Server. La restauration de fichiers individuels est similaire à la restauration de l’ensemble de la machine virtuelle, sauf que vous parcourez le VMDK et que vous recherchez les fichiers souhaités avant de commencer le processus de récupération. 

> [!NOTE]
> La restauration d’un fichier individuel à partir d’une machine virtuelle est disponible uniquement pour les points de récupération de disque et de machine virtuelle Windows.

1. Dans la console Administrateur du serveur de sauvegarde Azure, sélectionnez la vue **Récupération** .

1. À l’aide du volet **Parcourir** , parcourez ou filtrez pour rechercher la machine virtuelle que vous souhaitez récupérer. Une fois que vous sélectionnez une machine virtuelle ou un dossier, les **Points de récupération du volet affichent les points de récupération disponibles.

   ![Points de récupération disponibles](../backup/media/restore-azure-backup-server-vmware/vmware-rp-disk.png)

1. Dans le volet **Points de récupération pour** , utilisez le calendrier pour sélectionner la date qui contient les points de récupération souhaités. Selon la façon dont la stratégie de sauvegarde a été configurée, les dates peuvent avoir plusieurs points de récupération. 

1. Une fois que vous avez sélectionné le jour de la prise du point de récupération, assurez-vous d’avoir choisi la bonne **Heure de récupération** . 

   > [!NOTE]
   > Si la date sélectionnée comporte plusieurs points de récupération, choisissez votre point de récupération en le sélectionnant dans le menu déroulant **Heure de récupération** . 

   Une fois que vous avez choisi le point de récupération, la liste des éléments récupérables s’affiche dans le volet **Chemin** .

1. Pour rechercher les fichiers que vous souhaitez récupérer, dans le volet **Chemin** , double-cliquez sur l'élément dans la colonne **Élément récupérable** pour l’ouvrir. Ensuite, sélectionnez le fichier ou les dossiers que vous souhaitez récupérer. Pour sélectionner plusieurs éléments, sélectionnez la touche **Ctrl** tout en sélectionnant chaque élément. Utilisez le volet **Chemin** pour effectuer une recherche dans la liste des fichiers ou des dossiers figurant dans la colonne **Élément récupérable** .
    
   > [!NOTE]
   > **Rechercher dans la liste ci-dessous** n’effectue pas de recherche dans les sous-dossiers. Pour effectuer une recherche dans des sous-dossiers, double-cliquez sur le dossier. Utilisez le bouton **Haut** pour passer d'un dossier d’un dossier au dossier parent. Vous pouvez sélectionner plusieurs éléments (fichiers et dossiers), mais ils doivent se trouver dans le même dossier parent. Vous ne pouvez pas récupérer d’éléments de dossiers différents lors de la même tâche de récupération.

   ![Vérifier la sélection à récupérer](../backup/media/restore-azure-backup-server-vmware/vmware-rp-disk-ilr-2.png)

1. Une fois que vous avez sélectionné les éléments à récupérer, dans la barre d’outils Console Administrateur, sélectionnez **Récupérer**  pour ouvrir l’ **Assistant Récupération** . Dans l’ **Assistant Récupération** , l'écran **Vérifier la sélection à récupérer** affiche les éléments sélectionnés à récupérer.

1. Dans l’écran **Spécifier les options de récupération** , effectuez les étapes suivantes :

   - Sélectionnez **Modifier** pour activer la limitation d'utilisation du réseau. Dans la boîte de dialogue **Limitation** , sélectionnez **Activer la limitation de l'utilisation de la bande passante réseau** pour l’activer. Une fois la limitation activée, configurez les **Paramètres** et la **Planification des tâches** .
   - Sélectionnez **Suivant** pour désactiver la limitation du réseau.

1. Dans l'écran **Sélectionner le type de récupération** , sélectionnez **Suivant** . Vous ne pouvez récupérer vos fichiers ou dossiers que dans un dossier réseau.

1. Dans l'écran **Spécifier la destination** , sélectionnez **Parcourir** afin de rechercher un emplacement réseau pour vos fichiers ou dossiers. Le serveur de sauvegarde Azure crée un dossier où tous les éléments récupérés sont copiés. Le nom du dossier a le préfixe MABS_day-month-year. Lorsque vous sélectionnez un emplacement pour les fichiers ou le dossier récupérés, les détails de cet emplacement sont fournis.

   ![Spécifier l’emplacement de récupération des fichiers](../backup/media/restore-azure-backup-server-vmware/specify-destination.png)

1. Dans l’écran **Spécifier les options de récupération** , choisissez quels paramètres de sécurité appliquer. Vous pouvez choisir de modifier la limitation de l’utilisation de la bande passante réseau, mais la limitation est désactivée par défaut. Par ailleurs, **Récupération SAN** et **Notification** ne sont pas activées.

1. Dans l'écran **Résumé** , passez en revue vos paramètres et sélectionnez **Récupérer** pour démarrer le processus de récupération. L'écran **État de la récupération** affiche la progression de l’opération de récupération.

## <a name="next-steps"></a>Étapes suivantes

Pour résoudre les problèmes lors de la configuration des sauvegardes, passez en revue le guide de résolution des problèmes du serveur de sauvegarde Azure.

> [!div class="nextstepaction"]
> [Guide de résolution des problèmes pour le serveur de sauvegarde Azure](../backup/backup-azure-mabs-troubleshoot.md)
