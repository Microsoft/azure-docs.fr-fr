---
title: Sauvegarder des machines virtuelles VMware avec le serveur de sauvegarde Azure
description: Dans cet article, découvrez comment utiliser le serveur de sauvegarde Azure pour sauvegarder des machines virtuelles VMware s’exécutant sur un serveur VMware vCenter/ESXi.
ms.topic: conceptual
ms.date: 05/24/2020
ms.openlocfilehash: db5e5c4bdac64e2faf5babb107ecec61a02d6468
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96002951"
---
# <a name="back-up-vmware-vms-with-azure-backup-server"></a>Sauvegarder des machines virtuelles VMware avec le serveur de sauvegarde Azure

Cet article explique comment sauvegarder des machines virtuelles VMware s’exécutant sur des hôtes VMware ESXi/vCenter Server vers Azure en utilisant le serveur de sauvegarde Azure(MABS).

Cet article explique comment :

- Configurer un canal sécurisé afin que le serveur de sauvegarde Azure puisse communiquer avec des serveurs VMware via HTTPS.
- Configurer un compte VMware que le serveur de sauvegarde Azure utilisera pour accéder au serveur VMware.
- Ajouter les informations d’identification du compte à la sauvegarde Azure.
- Ajouter le serveur vCenter ou ESXi au serveur de sauvegarde Azure.
- Configurer un groupe de protection qui contient les machines virtuelles VMware que vous souhaitez sauvegarder, spécifier les paramètres de sauvegarde et planifier la sauvegarde.

## <a name="supported-vmware-features"></a>Fonctionnalités VMware prises en charge

MABS fournit les fonctionnalités suivantes lors de la sauvegarde de machines virtuelles VMware :

- Sauvegarde sans agent : MABS ne nécessite pas l’installation d’un agent sur le serveur vCenter ou ESXi pour sauvegarder la machine virtuelle. Au lieu de cela, il vous suffit de fournir l’adresse IP ou le nom de domaine complet (FQDN) ainsi que les informations d’identification utilisées pour authentifier le serveur VMware auprès de MABS.
- Sauvegarde intégrée au cloud : MABS protège les charges de travail sur disque et dans le cloud. Le workflow de sauvegarde et de récupération de MABS vous aide à gérer la conservation à long terme et la sauvegarde hors site.
- Détectez et protégez les machines virtuelles gérées par vCenter : MABS détecte et protège les machines virtuelles déployées sur un serveur VMware (serveur vCenter ou ESXi). À mesure que la taille de votre déploiement augmente, utilisez vCenter pour gérer votre environnement VMware. MABS détecte également les machines virtuelles gérées par vCenter, ce qui vous permet de protéger de grands déploiements.
- Protection automatique au niveau du dossier : vCenter vous permet d’organiser vos machines virtuelles dans des dossiers de machines virtuelles. MABS détecte ces dossiers et vous permet de protéger les machines virtuelles au niveau du dossier et d’inclure tous les sous-dossiers. Lors de la protection de dossiers, MABS protège non seulement les machines virtuelles de ce dossier, mais également les machines virtuelles ajoutées ultérieurement. MABS détecte les nouvelles machines virtuelles quotidiennement et les protège automatiquement. Quand vous organisez vos machines virtuelles dans des dossiers récursifs, MABS détecte et protège automatiquement les nouvelles machines virtuelles déployées dans les dossiers récursifs.
- MABS protège les machines virtuelles stockées sur un disque local, un système NFS (Network File System) ou un stockage en cluster.
- MABS protège les machines virtuelles migrées pour l’équilibrage de charge : À mesure que les machines virtuelles sont migrées pour l’équilibrage de charge, MABS détecte et continue automatiquement la protection des machines virtuelles.
- MABS peut récupérer des fichiers/dossiers d’une machine virtuelle Windows sans récupérer l’ensemble de la machine virtuelle, ce qui permet de récupérer plus rapidement les fichiers nécessaires.

## <a name="prerequisites-and-limitations"></a>Conditions préalables et limitations

Avant de commencer la sauvegarde d’une machine virtuelle VMware, passez en revue la liste suivante des limitations et conditions préalables.

- Si vous avez utilisé MABS pour protéger un serveur vCenter (s’exécutant sur Windows) en tant que serveur Windows en utilisant le nom de domaine complet du serveur, vous ne pouvez pas protéger ce serveur vCenter en tant que serveur VMware en utilisant le nom de domaine complet du serveur.
  - Comme solution de contournement, vous pouvez utiliser l’adresse IP statique du serveur vCenter.
  - Si vous voulez utiliser le nom de domaine complet, vous devez arrêter la protection en tant que serveur Windows, supprimer l’agent de protection, puis l’ajouter en tant que serveur VMware en utilisant le nom de domaine complet.
- Si vous utilisez vCenter pour gérer des serveurs ESXi dans votre environnement, ajoutez vCenter (et non pas ESXi) au groupe de protection MABS.
- Vous ne pouvez pas sauvegarder des captures instantanées effectuées par l’utilisateur avant la première sauvegarde MABS. Une fois que MABS a terminé la première sauvegarde, vous pouvez sauvegarder des captures instantanées effectuées par l’utilisateur.
- MABS ne peut pas protéger les machines virtuelles VMware avec des disques pass-through et des mappages de périphériques bruts (pRDM, physical Raw Device Mapping).
- MABS ne peut pas détecter ou protéger des applications VMware vApps.
- MABS ne peut pas protéger des machines virtuelles VMware avec des captures instantanées existantes.

## <a name="before-you-start"></a>Avant de commencer

- Veillez à exécuter une version de vCenter/ESXi prise en charge pour la sauvegarde. Reportez-vous au tableau de prise en charge [ici](./backup-mabs-protection-matrix.md).
- Assurez-vous que vous avez configuré le serveur de sauvegarde Azure. Dans le cas contraire, [effectuez cette opération](backup-azure-microsoft-azure-backup.md) avant de commencer. Vous devez exécuter le serveur de sauvegarde Azure avec les dernières mises à jour.
- Vérifiez que les ports réseau suivants sont ouverts :
  - TCP 443 entre MABS et vCenter
  - TCP 443 et TCP 902 entre MABS et l'hôte ESXi

## <a name="create-a-secure-connection-to-the-vcenter-server"></a>Créer une connexion sécurisée au serveur vCenter

Par défaut, le serveur de sauvegarde Azure communique avec les serveurs VMware via HTTPS. Pour configurer la connexion HTTPS, téléchargez le certificat de l’autorité de certification (AC) VMware et importez-le sur le serveur de sauvegarde Azure.

### <a name="before-you-begin"></a>Avant de commencer

- Si vous ne souhaitez pas utiliser le protocole HTTPS, vous pouvez [désactiver la validation de certificat HTTPS pour tous les serveurs VMware](backup-azure-backup-server-vmware.md#disable-https-certificate-validation).
- En général, vous vous connectez au serveur vCenter/ESXi depuis un navigateur sur le serveur de sauvegarde Azure via le client web vSphere. La première fois, la connexion n’est pas sécurisée et le message suivant s’affiche.
- Il est important de comprendre comment le serveur de sauvegarde Azure gère les sauvegardes.
  - Dans un premier temps, le serveur de sauvegarde Azure sauvegarde les données dans le stockage sur disque local. Le serveur de sauvegarde Azure utilise un pool de stockage, un ensemble de disques et de volumes sur lequel il stocke les points de récupération de disque pour ses données protégées. Il peut s’agir d’un stockage en attachement direct (DAS), un SAN Fibre Channel, ou un appareil de stockage ou SAN iSCSI. Il est important de vérifier que vous disposez de suffisamment de stockage pour la sauvegarde locale de vos données de machines virtuelles VMware.
  - Le serveur de sauvegarde Azure effectue ensuite la sauvegarde à partir du stockage de disque local vers Azure.
  - [Obtenez de l’aide](/system-center/dpm/create-dpm-protection-groups#figure-out-how-much-storage-space-you-need) pour déterminer la quantité d’espace de stockage dont vous avez besoin. Ces informations concernent DPM, mais elles peuvent être utilisées pour le serveur de sauvegarde Azure.

### <a name="set-up-the-certificate"></a>Configurer le certificat

Pour configurer un canal sécurisé, procédez comme suit :

1. Dans le navigateur sur le serveur de sauvegarde Azure, entrez l’URL du client web vSphere. Si la page de connexion n’apparaît pas, vérifiez les paramètres de connexion et de proxy du navigateur.

    ![Client web vSphere](./media/backup-azure-backup-server-vmware/vsphere-web-client.png)

2. Dans la page de connexion du client web vSphere, sélectionnez **Download trusted root CA certificates** (Télécharger les certificats d’autorité de certification racine de confiance).

    ![Télécharger un certificat d’autorité de certification racine de confiance](./media/backup-azure-backup-server-vmware/vmware-download-ca-cert-prompt.png)

3. Un fichier nommé **download** est téléchargé. En fonction de votre navigateur, vous recevez un message vous demandant d’ouvrir ou d’enregistrer le fichier.

    ![Télécharger le certificat d’autorité de certification](./media/backup-azure-backup-server-vmware/download-certs.png)

4. Enregistrez le fichier sur l’ordinateur du serveur de sauvegarde Azure avec une extension .zip.

5. Cliquez avec le bouton droit de la souris sur **download.zip** > **Extraire tout**. Le contenu du fichier .zip est extrait dans un dossier nommé **certs**, qui contient les éléments suivants :
   - Le fichier de certificat racine est doté d’une extension qui commence par une séquence numérotée comme .0 et .1.
   - Le fichier CRL est doté d’une extension qui commence par une séquence comme .r0 ou .r1. Le fichier CRL est associé à un certificat.

    ![Certificats téléchargés](./media/backup-azure-backup-server-vmware/extracted-files-in-certs-folder.png)

6. Dans le dossier **certs**, cliquez avec le bouton droit sur le fichier de certificat racine et sélectionnez > **Renommer**.

    ![Renommer un certificat racine](./media/backup-azure-backup-server-vmware/rename-cert.png)

7. Remplacez l’extension du certificat racine par .crt et confirmez. L’icône du fichier est remplacée par une icône représentant un certificat racine.

8. Cliquez sur le certificat racine et dans le menu contextuel, sélectionnez **Installer le certificat**.

9. Dans l’**Assistant Importation de certificat**, sélectionnez **Ordinateur local** comme destination du certificat, puis **Suivant** pour continuer. Si un message vous demande si vous voulez autoriser les modifications sur cet ordinateur, confirmez.

    ![Accueil de l’Assistant](./media/backup-azure-backup-server-vmware/certificate-import-wizard1.png)

10. Dans la page **Magasin de certificats**, sélectionnez **Placer tous les certificats dans le magasin suivant**, puis **Parcourir** pour choisir un magasin.

    ![Stockage des certificats](./media/backup-azure-backup-server-vmware/cert-import-wizard-local-store.png)

11. Dans **Sélectionner un magasin de certificats**, sélectionnez **Autorités de certification racines de confiance** comme dossier de destination des certificats, puis sélectionnez **OK**.

    ![Dossier de destination des certificats](./media/backup-azure-backup-server-vmware/certificate-store-selected.png)

12. À la **fin de l’Assistant Importation de certificat**, vérifiez le dossier, puis sélectionnez **Terminer**.

    ![Vérifier que le certificat se trouve dans le dossier approprié](./media/backup-azure-backup-server-vmware/cert-wizard-final-screen.png)

13. Après avoir confirmé l’importation de certificat, connectez-vous au serveur vCenter pour vérifier que votre connexion est sécurisée.

### <a name="disable-https-certificate-validation"></a>Désactiver la validation de certificat HTTPS

Si vous avez des limites sécurisées au sein de votre organisation et que vous ne souhaitez pas utiliser le protocole HTTPS entre les serveurs VMware et l’ordinateur du serveur de sauvegarde Azure, désactivez HTTPS comme suit :

1. Copiez et collez le texte suivant dans un fichier .txt.

    ```text
    Windows Registry Editor Version 5.00
    [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft Data Protection Manager\VMWare]
    "IgnoreCertificateValidation"=dword:00000001
    ```

2. Enregistrez le fichier sous le nom **DisableSecureAuthentication.reg** sur votre serveur de sauvegarde Azure.

3. Double-cliquez sur le fichier pour activer l’entrée de Registre.

## <a name="create-a-vmware-role"></a>Créer un rôle VMware

Le serveur de sauvegarde Azure a besoin d’un compte d’utilisateur avec des autorisations d’accès à vCenter Server/l’hôte ESXi. Créez un rôle VMware avec des privilèges spécifiques, puis associez-y un compte utilisateur.

1. Connectez-vous à vCenter Server (ou à l’hôte ESXi si vous n’utilisez pas vCenter Server).
2. Dans le volet **Navigateur**, sélectionnez **Administration**.

    ![Administration](./media/backup-azure-backup-server-vmware/vmware-navigator-panel.png)

3. Dans **Administration** > **Rôles**, sélectionnez l’icône d’ajout de rôle (symbole +).

    ![Ajouter un rôle](./media/backup-azure-backup-server-vmware/vmware-define-new-role.png)

4. Dans **Créer un rôle** > **Nom du rôle**, entrez *BackupAdminRole*. Vous pouvez choisir n’importe quel nom de rôle, à condition qu’il permettre de reconnaître l’objectif du rôle.

5. Sélectionnez les privilèges, comme décrit dans le tableau ci-dessous, puis sélectionnez **OK**.  Le nouveau rôle s’affiche dans la liste du panneau **Rôles**.
   - Sélectionnez l’icône située à côté de l’étiquette du parent pour développer le parent et afficher les privilèges enfant.
   - Pour sélectionner les privilèges VirtualMachine, vous devez atteindre plusieurs niveaux dans la hiérarchie parent-enfant.
   - Vous n’avez pas besoin de sélectionner tous les privilèges enfant au sein d’un privilège parent.

    ![Hiérarchie des privilèges parent-enfant](./media/backup-azure-backup-server-vmware/cert-add-privilege-expand.png)

### <a name="role-permissions"></a>Autorisations des rôles

Le tableau suivant indique les privilèges que vous devez attribuer au compte d'utilisateur que vous créez :

| Privilèges du compte d'utilisateur vCenter 6.5                          | Privilèges du compte d'utilisateur vCenter 6.7                            |
|----------------------------------------------------------------------------|----------------------------------------------------------------------------|
| Datastore cluster.Configure a datastore cluster                           | Datastore cluster.Configure a datastore cluster                           |
| Datastore.AllocateSpace                                                    | Datastore.AllocateSpace                                                    |
| Datastore.Browse datastore                                                 | Datastore.Browse datastore                                                 |
| Opérations sur les fichiers Datastore.Low-level                                        | Opérations sur les fichiers Datastore.Low-level                                        |
| Global.Disable methods                                                     | Global.Disable methods                                                     |
| Global.Enable methods                                                      | Global.Enable methods                                                      |
| Global.Licenses                                                            | Global.Licenses                                                            |
| Global.Log event                                                           | Global.Log event                                                           |
| Attributs personnalisés Global.Manage                                            | Attributs personnalisés Global.Manage                                            |
| Global.Set custom attribute                                                | Global.Set custom attribute                                                |
| Host.Local operations.Create virtual machine                               | Host.Local operations.Create virtual machine                               |
| Network.Assign network                                                     | Network.Assign network                                                     |
| Resource. Attribuer une machine virtuelle à une liste de ressources partagées                          | Resource. Attribuer une machine virtuelle à une liste de ressources partagées                          |
| vApp.Add virtual machine                                                   | vApp.Add virtual machine                                                   |
| vApp.Assign resource pool                                                  | vApp.Assign resource pool                                                  |
| vApp.Unregister                                                            | vApp.Unregister                                                            |
| VirtualMachine.Configuration. Ajouter ou supprimer un appareil                         | VirtualMachine.Configuration. Ajouter ou supprimer un appareil                         |
| Virtual machine.Configuration.Disk lease                                   | Virtual machine.Configuration.Acquire disk lease                           |
| Virtual machine.Configuration.Add new disk                                 | Virtual machine.Configuration.Add new disk                                 |
| Virtual machine.Configuration.Advanced                                     | Virtual machine.Configuration.Advanced configuration                       |
| Virtual machine.Configuration.Disk change tracking                         | Virtual machine.Configuration.Toggle disk change tracking                  |
| Virtual machine.Configuration.Host USB device                              | Virtual machine.Configuration.Configure Host USB device                    |
| Virtual machine.Configuration.Extend virtual disk                          | Virtual machine.Configuration.Extend virtual disk                          |
| Virtual machine.Configuration.Query unowned files                          | Virtual machine.Configuration.Query unowned files                          |
| Virtual machine.Configuration.Swapfile placement                           | Virtual machine.Configuration.Change Swapfile placement                    |
| Virtual machine.Guest Operations.Guest Operation Program Execution         | Virtual machine.Guest Operations.Guest Operation Program Execution         |
| Virtual machine.Guest Operations.Guest Operation Modifications             | Virtual machine.Guest Operations.Guest Operation Modifications             |
| Virtual machine.Guest Operations.Guest Operation Queries                   | Virtual machine.Guest Operations.Guest Operation Queries                   |
| Virtual machine .Interaction .Device connection                            | Virtual machine .Interaction .Device connection                            |
| Virtual machine .Interaction .Guest operating system management by VIX API | Virtual machine .Interaction .Guest operating system management by VIX API |
| Virtual machine.Interaction.Power Off                                    | Virtual machine.Interaction.Power Off                                    |
| Virtual machine.Inventory.Create new                                      | Virtual machine.Inventory.Create new                                      |
| Virtual machine .Inventory.Remove                                          | Virtual machine .Inventory.Remove                                          |
| Virtual machine .Inventory.Register                                        | Virtual machine .Inventory.Register                                        |
| Virtual machine .Provisioning.Allow disk access                            | Virtual machine .Provisioning.Allow disk access                            |
| Virtual machine .Provisioning.Allow file access                            | Virtual machine .Provisioning.Allow file access                            |
| Virtual machine .Provisioning.Allow read-only disk access                  | Virtual machine .Provisioning.Allow read-only disk access                  |
| Virtual machine .Provisioning.Allow virtual machine download               | Virtual machine .Provisioning.Allow virtual machine download               |
| Virtual machine .Snapshot management. Créer l’instantané                      | Virtual machine .Snapshot management. Créer l’instantané                      |
| Virtual machine .Snapshot management.Remove Snapshot                       | Virtual machine .Snapshot management.Remove Snapshot                       |
| Virtual machine .Snapshot management.Revert to snapshot                    | Virtual machine .Snapshot management.Revert to snapshot                    |

> [!NOTE]
> Le tableau suivant répertorie les privilèges des comptes d'utilisateur vCenter 6.0 et vCenter 5.5.

| Privilèges du compte d'utilisateur vCenter 6.0 | Privilèges du compte d'utilisateur vCenter 5.5 |
| --- | --- |
| Datastore.AllocateSpace | Network.Assign |
| Attributs personnalisés Global.Manage | Datastore.AllocateSpace |
| Global.Set custom attribute | VirtualMachine.Config.ChangeTracking |
| Host.Local operations.Create virtual machine | VirtualMachine.State.RemoveSnapshot |
| Réseau. Attribuer un réseau | VirtualMachine.State.CreateSnapshot |
| Resource. Attribuer une machine virtuelle à une liste de ressources partagées | VirtualMachine.Provisioning.DiskRandomRead |
| Virtual machine.Configuration.Add new disk | VirtualMachine.Interact.PowerOff |
| Virtual machine.Configuration.Advanced | VirtualMachine.Inventory.Create |
| Virtual machine.Configuration.Disk change tracking | VirtualMachine.Config.AddNewDisk |
| Virtual machine.Configuration.Host USB device | VirtualMachine.Config.HostUSBDevice |
| Virtual machine.Configuration.Query unowned files | VirtualMachine.Config.AdvancedConfig |
| Virtual machine.Configuration.Swapfile placement | VirtualMachine.Config.SwapPlacement |
| Virtual machine.Interaction.Power Off | Global.ManageCustomFields |
| Virtual machine.Inventory. Création |   |
| Virtual machine.Provisioning.Allow disk access |   |
| Virtual machine.Provisioning. Autoriser l’accès au disque en lecture seule |   |
| Virtual machine.Snapshot management.Create snapshot |   |
| Virtual machine.Snapshot management.Remove Snapshot |   |

## <a name="create-a-vmware-account"></a>Créer un compte VMware

1. Dans le panneau **Navigateur** de vCenter Server, sélectionnez **Utilisateurs et groupes**. Si vous n’utilisez pas vCenter Server, créez le compte sur l’hôte ESXi approprié.

    ![Option Utilisateurs et groupes](./media/backup-azure-backup-server-vmware/vmware-userandgroup-panel.png)

    Le panneau **Utilisateurs et groupes vCenter** s’affiche.

2. Dans le panneau **Utilisateurs et groupes vCenter**, sélectionnez l’onglet **Utilisateurs**, puis l’icône d’ajout d’utilisateurs (symbole +).

    ![Panneau Utilisateurs et groupes vCenter](./media/backup-azure-backup-server-vmware/usersandgroups.png)

3. Dans la boîte de dialogue **Nouvel utilisateur**, ajoutez les informations utilisateur > **OK**. Dans cette procédure, le nom d’utilisateur est BackupAdmin.

    ![Boîte de dialogue Nouvel utilisateur](./media/backup-azure-backup-server-vmware/vmware-new-user-account.png)

4. Pour associer le compte d’utilisateur au rôle, dans le panneau **Navigateur**, sélectionnez **Autorisations globales**. Dans le panneau **Autorisations globales**, sélectionnez l’onglet **Gérer**, puis l’icône d’ajout (symbole +).

    ![Panneau Autorisations globales](./media/backup-azure-backup-server-vmware/vmware-add-new-perms.png)

5. Dans la boîte de dialogue **Racine des autorisations globales – Ajouter une autorisation**, sélectionnez **Ajouter** pour choisir l’utilisateur ou le groupe.

    ![Choisir un utilisateur ou un groupe](./media/backup-azure-backup-server-vmware/vmware-add-new-global-perm.png)

6. Dans **Sélectionner des utilisateurs/groupes**, choisissez **BackupAdmin** > **Ajouter**. Dans la zone **Utilisateurs**, le format *domain\username* est utilisé pour le compte d’utilisateur. Si vous souhaitez utiliser un autre domaine, sélectionnez-le dans la liste **Domaine**. Sélectionnez **OK** pour ajouter les utilisateurs sélectionnés à la boîte de dialogue **Ajouter une autorisation**.

    ![Ajouter un utilisateur BackupAdmin](./media/backup-azure-backup-server-vmware/vmware-assign-account-to-role.png)

7. Dans **Rôle attribué**, dans la liste déroulante, sélectionnez **BackupAdminRole** > **OK**.

    ![Affecter un utilisateur à un rôle](./media/backup-azure-backup-server-vmware/vmware-choose-role.png)

Dans l’onglet **Gérer** du panneau **Autorisations globales**, le nouveau compte d’utilisateur et le rôle associé apparaissent dans la liste.

## <a name="add-the-account-on-azure-backup-server"></a>Ajouter le compte sur le serveur de sauvegarde Azure

1. Ouvrez le serveur de sauvegarde Azure. Si vous ne trouvez pas l’icône sur le Bureau, ouvrez la Sauvegarde Microsoft Azure dans la liste des applications.

    ![Icône du serveur de sauvegarde Azure](./media/backup-azure-backup-server-vmware/mabs-icon.png)

2. Dans la console du serveur de sauvegarde Azure, sélectionnez **Gestion** >  **Serveurs de production** > **Gérer VMware**.

    ![Console du serveur de sauvegarde Azure](./media/backup-azure-backup-server-vmware/add-vmware-credentials.png)

3. Dans la boîte de dialogue **Gérer les informations d’identification**, sélectionnez **Ajouter**.

    ![Boîte de dialogue Gérer les informations d’identification](./media/backup-azure-backup-server-vmware/mabs-manage-credentials-dialog.png)

4. Dans **Ajouter des informations d’identification**, entrez le nom et la description des nouvelles informations d’identification, puis spécifiez le nom d’utilisateur et le mot de passe définis sur le serveur VMware. Le nom *Contoso Vcenter credential* est utilisé pour identifier les informations d’identification dans cette procédure. Si le serveur VMware et le serveur de sauvegarde Azure ne se trouvent pas sur le même domaine, spécifiez le domaine dans le nom d’utilisateur.

    ![Boîte de dialogue Ajouter des informations d’identification du serveur de sauvegarde Azure](./media/backup-azure-backup-server-vmware/mabs-add-credential-dialog2.png)

5. Sélectionnez **Ajouter** pour ajouter les nouvelles informations d’identification.

    ![Ajouter de nouvelles informations d’identification](./media/backup-azure-backup-server-vmware/new-list-of-mabs-creds.png)

## <a name="add-the-vcenter-server"></a>Ajouter le serveur vCenter Server

Ajoutez le serveur vCenter Server au serveur de sauvegarde Azure.

1. Dans la console du serveur de sauvegarde Azure, sélectionnez **Gestion** > **Serveurs de production** > **Ajouter**.

    ![Ouvrir l’Assistant Ajout d’un serveur de production](./media/backup-azure-backup-server-vmware/add-vcenter-to-mabs.png)

2. Dans la page **Assistant Ajout d’un serveur de production** > **Sélectionner un type de serveur de production**, sélectionnez **Serveurs VMware**, puis **Suivant**.

    ![Assistant Ajout d’un serveur de production](./media/backup-azure-backup-server-vmware/production-server-add-wizard.png)

3. Dans **Sélectionner les ordinateurs**  **Nom/adresse IP du serveur**, spécifiez le nom de domaine complet (FQDN) ou l’adresse P du serveur VMware. Spécifiez le nom vCenter si tous les serveurs ESXi sont gérés par le même serveur vCenter. Sinon, ajoutez l’hôte ESXi.

    ![Spécifier le serveur VMware](./media/backup-azure-backup-server-vmware/add-vmware-server-provide-server-name.png)

4. Dans **Port SSL**, entrez le port utilisé pour communiquer avec le serveur VMware. 443 est le port par défaut, mais vous pouvez le modifier si votre serveur VMware effectue l’écoute sur un port différent.

5. Dans **Indiquer les informations d’identification**, sélectionnez les informations d’identification que vous avez créées précédemment.

    ![Indiquer les informations d’identification](./media/backup-azure-backup-server-vmware/identify-creds.png)

6. Sélectionnez **Ajouter** pour ajouter le serveur VMware à la liste des serveurs. Sélectionnez ensuite **Suivant**.

    ![Ajouter un serveur VMware et des informations d’identification](./media/backup-azure-backup-server-vmware/add-vmware-server-credentials.png)

7. Dans la page **Résumé**, sélectionnez **Ajouter** pour ajouter le serveur VMware au serveur de sauvegarde Azure. Le nouveau serveur est ajouté immédiatement. Aucun agent n’est nécessaire sur le serveur VMware.

    ![Ajouter le serveur VMware au serveur de sauvegarde Azure](./media/backup-azure-backup-server-vmware/tasks-screen.png)

8. Vérifiez les paramètres dans la page **Terminer**.

   ![Page Terminer](./media/backup-azure-backup-server-vmware/summary-screen.png)

Si plusieurs hôtes ESXi ne sont pas gérés par le serveur vCenter, ou si vous avez plusieurs instances de vCenter Server, vous devez réexécuter l’Assistant pour ajouter les serveurs.

## <a name="configure-a-protection-group"></a>Configurer un groupe de protection

Ajoutez des machines virtuelles VMware pour la sauvegarde. Les groupes de protection rassemblent plusieurs machines virtuelles et appliquent les mêmes paramètres de sauvegarde et de conservation des données à toutes les machines virtuelles du groupe.

1. Dans la console du serveur de sauvegarde Azure, sélectionnez **Protection** > **Nouveau**.

    ![Ouvrir l’Assistant Création d’un nouveau groupe de protection](./media/backup-azure-backup-server-vmware/open-protection-wizard.png)

1. Dans la page d’accueil de l’Assistant **Création d’un nouveau groupe de protection**, sélectionnez **Suivant**.

    ![Boîte de dialogue Assistant Création d’un nouveau groupe de protection](./media/backup-azure-backup-server-vmware/protection-wizard.png)

1. Dans la page **Sélectionner le type de groupe de protection**, sélectionnez **Serveurs**, puis **Suivant**. La page **Sélectionner les membres du groupe** s’affiche.

1. Dans **Sélectionner les membres du groupe**, sélectionnez les machines virtuelles (ou dossiers de machines virtuelles) que vous souhaitez sauvegarder. Sélectionnez ensuite **Suivant**.

    - Lorsque vous sélectionnez un dossier, les machines virtuelles ou dossiers à l’intérieur de ce dossier sont également sélectionnés pour la sauvegarde. Vous pouvez désactiver les dossiers ou machines virtuelles que vous ne souhaitez pas sauvegarder.
1. Si une machine virtuelle ou un dossier fait déjà l’objet d’une sauvegarde, vous ne pouvez pas le sélectionner. Cela garantit que des points de récupération ne sont pas créés en double pour la même machine virtuelle.

    ![Sélectionner les membres du groupe](./media/backup-azure-backup-server-vmware/server-add-selected-members.png)

1. Dans la page **Sélectionner la méthode de protection des données**, entrez le nom du groupe de protection, et définissez les paramètres de protection. Pour sauvegarder sur Azure, définissez la protection à court terme sur **Disque** et activez la protection en ligne. Sélectionnez ensuite **Suivant**.

    ![Sélectionner la méthode de protection des données](./media/backup-azure-backup-server-vmware/name-protection-group.png)

1. Dans **Spécifier les objectifs à court terme**, spécifiez la durée pendant laquelle vous souhaitez conserver les données sauvegardées sur disque.
   - Dans **Durée de rétention**, spécifiez le nombre de jours les points de récupération de disque doivent être conservés.
   - Dans **Fréquence de synchronisation**, spécifiez la fréquence d’enregistrement des points de récupération de disque.
       - Si vous ne souhaitez pas définir d’intervalle de sauvegarde, vous pouvez activer l’option **Juste avant un point de récupération** afin qu’une sauvegarde s’exécute juste avant chaque point de récupération planifié.
       - Les sauvegardes à court terme sont des sauvegardes complètes et non incrémentielles.
       - Sélectionnez **Modifier** pour modifier les heures/dates es sauvegardes à court terme.

         ![Spécifier les objectifs à court terme](./media/backup-azure-backup-server-vmware/short-term-goals.png)

1. Dans **Vérifier l’allocation de disque**, vérifiez l’espace disque fourni pour les sauvegardes. de machines virtuelles.

   - Les allocations de disques recommandées dépendent de la durée de rétention spécifiée, du type de charge de travail et de la taille des données protégées. Apportez les modifications nécessaires, puis sélectionnez **Suivant**.
   - **Taille des données :** taille des données dans le groupe de protection.
   - **Espace disque :** quantité d’espace disque recommandée pour le groupe de protection. Pour modifier ce paramètre, vous devez allouer un espace total légèrement supérieur au volume de croissance estimé pour chaque source de données.
   - **Colocaliser des données :** si vous activez la colocalisation, plusieurs sources de données dans la protection peuvent être mappées à un seul réplica et à un seul volume de points de récupération. La colocalisation n’est pas prise en charge pour toutes les charges de travail.
   - **Augmenter automatiquement :** si vous activez ce paramètre et si les données contenues dans le groupe protégé dépassent l’allocation initiale, le serveur de sauvegarde Azure tentera d’augmenter la taille du disque de 25 %.
   - **Détails de pool de stockage :** affiche l’état du pool de stockage, notamment la taille totale et la taille de disque restante.

    ![Vérifier l’allocation de disque](./media/backup-azure-backup-server-vmware/review-disk-allocation.png)

1. Dans la page **Choisir la méthode de création d’un réplica**, spécifiez la façon dont vous souhaitez effectuer la sauvegarde initiale, puis sélectionnez **Suivant**.
   - Les valeurs **Automatiquement sur le réseau** et **Maintenant** sont définies par défaut.
   - Si vous utilisez la valeur par défaut, nous vous recommandons de spécifier une heure creuse. Choisissez **Ultérieurement** et spécifiez le jour et l’heure.
   - Pour de grandes quantités de données ou des conditions réseau peu optimales, pensez à répliquer les données hors connexion à l’aide d’un média amovible.

    ![Choisir la méthode de création d’un réplica](./media/backup-azure-backup-server-vmware/replica-creation.png)

1. Dans **Options de vérification de cohérence**, indiquez comment et quand automatiser les vérifications de cohérence. Sélectionnez ensuite **Suivant**.
      - Vous pouvez exécuter les vérifications de cohérence lorsque les données du réplica deviennent incohérentes ou en fonction d’une planification définie.
      - Si vous ne voulez pas configurer de vérifications de cohérence automatiques, vous pouvez exécuter une vérification manuelle. Pour ce faire, cliquez avec le bouton droit sur le groupe de protection > **Effectuer une vérification de cohérence**.

1. Dans **Spécifier les données de protection en ligne**, sélectionnez les machines virtuelles ou les dossiers de machines virtuelles à sauvegarder. Vous pouvez sélectionner les membres un à un ou **Sélectionner tout** pour choisir tous les membres. Sélectionnez ensuite **Suivant**.

    ![Spécifier les données de protection en ligne](./media/backup-azure-backup-server-vmware/select-data-to-protect.png)

1. Dans la page **Spécifier la planification de sauvegarde en ligne**, spécifiez la fréquence à laquelle vous souhaitez sauvegarder les données du stockage local vers Azure.

    - Des points de récupération cloud des données seront générés en fonction du calendrier. Sélectionnez ensuite **Suivant**.
    - Une fois le point de récupération généré, il est transféré vers le coffre Recovery Services dans Azure.

    ![Spécifier la planification de sauvegarde en ligne](./media/backup-azure-backup-server-vmware/online-backup-schedule.png)

1. Dans la page **Spécifier la stratégie de rétention en ligne**, sélectionnez la durée pendant laquelle vous souhaitez conserver les points de récupération créés à partir des sauvegardes quotidiennes, hebdomadaires, mensuelles et annuelles dans Azure. Sélectionnez ensuite **Suivant**.

    - La durée de conservation des données dans Azure n’est soumise à aucune restriction.
    - La seule limite que vous devez respecter est que vous ne pouvez pas avoir plus de 9999 points de récupération par instance protégée. Dans cet exemple, l’instance protégée est le serveur VMware.

    ![Spécifier la stratégie de rétention en ligne](./media/backup-azure-backup-server-vmware/retention-policy.png)

1. Dans la page **Résumé**, vérifiez les paramètres, puis sélectionnez **Créer un groupe**.

    ![Résumé des paramètres et des membres du groupe de protection](./media/backup-azure-backup-server-vmware/protection-group-summary.png)

## <a name="vmware-parallel-backups"></a>Sauvegardes parallèles VMware

>[!NOTE]
> Cette fonctionnalité s’applique à MABS v3 UR1.

Dans les versions antérieures de MABS, les sauvegardes parallèles n’étaient effectuées que sur les groupes de protection. Avec MABS v3 UR1, toutes les sauvegardes des machines virtuelles VMWare appartenant à un même groupe de protection sont effectuées en parallèle, ce qui permet d’accélérer le processus de sauvegarde. Tous les travaux de réplication delta VMWare s’exécutent en parallèle. Par défaut, le nombre de travaux qui s’exécutent en parallèle est configuré sur 8.

Vous pouvez modifier le nombre de travaux à l’aide de la clé de Registre, comme indiqué ci-dessous (celle-ci n’est pas présente par défaut, vous devez donc l’ajouter) :

**Chemin de la clé** : `Software\Microsoft\Microsoft Data Protection Manager\Configuration\ MaxParallelIncrementalJobs\VMWare`<BR>
**Type de clé** : Valeur DWORD (32 bits).

> [!NOTE]
> Vous pouvez augmenter le nombre de travaux. Si vous définissez ce nombre sur 1, les travaux de réplication seront exécutés les uns après les autres. Pour augmenter le nombre de travaux à exécuter en parallèle, vous devez prendre en compte les performances de VMWare. Tenez compte du nombre de ressources en cours d’utilisation et de toute autre utilisation nécessaire sur le serveur VMWare vSphere, puis déterminez le nombre de travaux de réplication delta à exécuter en parallèle. Cette modification affectera uniquement les groupes de protection qui viennent d’être créés. Pour les groupes de protection existants, vous devez ajouter temporairement une autre machine virtuelle au groupe de protection. La configuration du groupe de protection doit être mise à jour en conséquence. Vous pouvez supprimer cette machine virtuelle du groupe de protection une fois la procédure terminée.

## <a name="vmware-vsphere-67"></a>VMWare vSphere 6.7

Pour sauvegarder vSphere 6.7, effectuez les opérations suivantes :

- Activer TLS 1.2 sur le serveur MABS

>[!NOTE]
>Sur VMWare 6.7 et ultérieur, TLS est le protocole de communication activé.

- Définissez les clés de Registre comme suit :

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

## <a name="exclude-disk-from-vmware-vm-backup"></a>Exclure un disque de la sauvegarde de machine virtuelle VMware

> [!NOTE]
> Cette fonctionnalité s’applique à MABS v3 UR1.

Avec MABS v3 UR1, vous pouvez exclure un disque de la sauvegarde de machine virtuelle VMware. Le script de configuration **ExcludeDisk.ps1** se trouve dans `C:\Program Files\Microsoft Azure Backup Server\DPM\DPM\bin folder`.

Pour configurer l’exclusion du disque, effectuez les étapes suivantes :

### <a name="identify-the-vmware-vm-and-disk-details-to-be-excluded"></a>Identifier les détails de la machine virtuelle VMWare et du disque à exclure

  1. Dans la console VMware, accédez aux paramètres de la machine virtuelle dont vous souhaitez exclure le disque.
  2. Sélectionnez le disque que vous souhaitez exclure et notez le chemin de ce disque.

        Par exemple, pour exclure le disque dur 2 du TestVM4, le chemin du disque dur 2 doit être **[datastore1] TestVM4/TestVM4\_1.vmdk**.

        ![Disque dur à exclure](./media/backup-azure-backup-server-vmware/test-vm.png)

### <a name="configure-mabs-server"></a>Configurer le serveur MABS

Accédez au serveur MABS sur lequel la machine virtuelle VMware est configurée pour la protection, afin de configurer l’exclusion du disque.

  1. Récupérez les détails de l’hôte VMware qui est protégé sur le serveur MABS.

        ```powershell
        $psInfo = get-DPMProductionServer
        $psInfo
        ```

        ```output
        ServerName   ClusterName     Domain            ServerProtectionState
        ----------   -----------     ------            ---------------------
        Vcentervm1                   Contoso.COM       NoDatasourcesProtected
        ```

  2. Sélectionnez l’hôte VMware, puis listez la protection des machines virtuelles pour l’hôte VMware.

        ```powershell
        $vmDsInfo = get-DPMDatasource -ProductionServer $psInfo[0] -Inquire
        $vmDsInfo
        ```

        ```output
        Computer     Name     ObjectType
        --------     ----     ----------
        Vcentervm1  TestVM2      VMware
        Vcentervm1  TestVM1      VMware
        Vcentervm1  TestVM4      VMware
        ```

  3. Sélectionnez la machine virtuelle de laquelle vous souhaitez exclure un disque.

        ```powershell
        $vmDsInfo[2]
        ```

        ```output
        Computer     Name      ObjectType
        --------     ----      ----------
        Vcentervm1   TestVM4   VMware
        ```

  4. Pour exclure le disque, accédez au dossier `Bin`, puis exécutez le script *ExcludeDisk.ps1* avec les paramètres suivants :

        > [!NOTE]
        > Avant d’exécuter cette commande, arrêtez le service DPMRA sur le serveur MABS. Si vous ne le faites pas, le script indiquera la réussite du processus, mais ne mettra pas à jour la liste d’exclusions. Vérifiez qu’aucun travail n’est en cours avant d’arrêter le service.

     **Pour ajouter ou retirer le disque de l’exclusion, exécutez la commande suivante :**

      ```powershell
      ./ExcludeDisk.ps1 -Datasource $vmDsInfo[0] [-Add|Remove] "[Datastore] vmdk/vmdk.vmdk"
      ```

     **Exemple** :

     Pour ajouter l’exclusion du disque à TestVM4, exécutez la commande suivante :

       ```powershell
      C:\Program Files\Microsoft Azure Backup Server\DPM\DPM\bin> ./ExcludeDisk.ps1 -Datasource $vmDsInfo[2] -Add "[datastore1] TestVM4/TestVM4\_1.vmdk"
       ```

      ```output
       Creating C:\Program Files\Microsoft Azure Backup Server\DPM\DPM\bin\excludedisk.xml
       Disk : [datastore1] TestVM4/TestVM4\_1.vmdk, has been added to disk exclusion list.
      ```

  5. Vérifiez que le disque a été ajouté en vue de son exclusion.

     **Pour afficher l’exclusion existante de certaines machines virtuelles, exécutez la commande suivante :**

        ```powershell
        ./ExcludeDisk.ps1 -Datasource $vmDsInfo[0] [-view]
        ```

     **Exemple**

        ```powershell
        C:\Program Files\Microsoft Azure Backup Server\DPM\DPM\bin> ./ExcludeDisk.ps1 -Datasource $vmDsInfo[2] -view
        ```

        ```output
        <VirtualMachine>
        <UUID>52b2b1b6-5a74-1359-a0a5-1c3627c7b96a</UUID>
        <ExcludeDisk>[datastore1] TestVM4/TestVM4\_1.vmdk</ExcludeDisk>
        </VirtualMachine>
        ```

     Une fois que la protection est configurée pour cette machine virtuelle, le disque exclu n’est plus listé.

        > [!NOTE]
        > Si vous effectuez ces étapes pour une machine virtuelle déjà protégée, vous devez exécuter manuellement la vérification de cohérence, après avoir ajouté le disque à la liste d’exclusions.

### <a name="remove-the-disk-from-exclusion"></a>Retirer le disque de l’exclusion

Pour retirer le disque de l’exclusion, exécutez la commande suivante :

```powershell
C:\Program Files\Microsoft Azure Backup Server\DPM\DPM\bin> ./ExcludeDisk.ps1 -Datasource $vmDsInfo[2] -Remove "[datastore1] TestVM4/TestVM4\_1.vmdk"
```

## <a name="next-steps"></a>Étapes suivantes

Pour résoudre les problèmes lors de la configuration des sauvegardes, passez en revue le [guide de dépannage du serveur de sauvegarde Azure](./backup-azure-mabs-troubleshoot.md).
