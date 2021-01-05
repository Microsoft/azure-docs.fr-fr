---
title: Gérer le serveur de configuration pour la récupération d’urgence avec Azure Site Recovery
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 04/15/2019
ms.author: ramamill
ms.openlocfilehash: 2f1edc14efdeaf70bf4c2acc0e31e1517753ed3e
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92546345"
---
# <a name="manage-the-configuration-server-for-vmware-vmphysical-server-disaster-recovery"></a>Gérer le serveur de configuration pour la récupération d'urgence d’un serveur physique ou d’une machine virtuelle VMware

Quand vous utilisez [Azure Site Recovery](site-recovery-overview.md) pour la reprise après sinistre de machines virtuelles VMware et de serveurs physiques sur Azure, vous devez configurer un serveur de configuration local. Le serveur de configuration coordonne la communication entre les machines locales VMware et Azure, et gère la réplication des données. Cet article récapitule les tâches courantes de gestion du serveur de configuration que vous devez effectuer après son déploiement.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="update-windows-license"></a>Mettre à jour la licence Windows

La licence fournie avec le modèle OVF est une licence d’évaluation valide 180 jours. Pour une utilisation ininterrompue, vous devez activer Windows avec une licence achetée. La mise à jour de licence peut être effectuée à l’aide d’une clé autonome ou d’une clé KMS standard. Vous trouverez des conseils sur la [ligne de commande Windows DISM pour exécuter le système d’exploitation](/windows-hardware/manufacture/desktop/dism-windows-edition-servicing-command-line-options). Pour obtenir des clés, reportez-vous à [Configurer un client KMS](/windows-server/get-started/kmsclientkeys).

## <a name="access-configuration-server"></a>Accéder au serveur de configuration

Vous pouvez accéder au serveur de configuration comme suit :

* Connectez-vous à la machine virtuelle où il est déployé et lancez le **gestionnaire de configuration Azure Site Recovery** en utilisant le raccourci Bureau.
* Vous pouvez aussi accéder au serveur de configuration à distance depuis https:// *ConfigurationServerName* /:44315/. Connectez-vous avec des informations d'identification d'administrateur.

## <a name="modify-vmware-server-settings"></a>Modifier les paramètres du serveur VMware

1. Pour associer un autre serveur VMware au serveur de configuration, sélectionnez **Ajouter un serveur vCenter Server/vSphere ESXi** après la [connexion](#access-configuration-server).
2. Entrez les détails et sélectionnez **OK** .

## <a name="modify-credentials-for-automatic-discovery"></a>Modifier les informations d’identification pour la découverte automatique

1. Pour mettre à jour les informations d’identification à utiliser pour la connexion au serveur VMware et la découverte automatique des machines virtuelles VMware, après la [connexion](#access-configuration-server), sélectionnez le compte et cliquez sur **Modifier** .
2. Entrez les nouvelles informations d’identification, puis sélectionnez **OK** .

    ![Modification de VMware](./media/vmware-azure-manage-configuration-server/modify-vmware-server.png)

Vous pouvez également modifier les informations d’identification via CSPSConfigtool.exe.

1. Se connecter au serveur de configuration et lancer CSPSConfigtool.exe
2. Choisissez le compte que vous souhaitez modifier, puis cliquez sur **Modifier** .
3. Entrez les informations d’identification modifiées et cliquez sur **Ok**

## <a name="modify-credentials-for-mobility-service-installation"></a>Modifier les informations d’identification pour l’installation du service Mobilité

Modifiez les informations d’identification à utiliser pour installer automatiquement le service Mobilité sur les machines virtuelles VMware sur lesquelles vous activez la réplication.

1. Après la [connexion](#access-configuration-server), sélectionnez **Gérer les informations d’identification de machine virtuelle**
2. Choisissez le compte que vous souhaitez modifier, puis cliquez sur **Modifier**
3. Entrez les nouvelles informations d’identification, puis sélectionnez **OK** .

    ![Modifier les informations d’identification du service Mobilité](./media/vmware-azure-manage-configuration-server/modify-mobility-credentials.png)

Vous pouvez également modifier les informations d’identification via CSPSConfigtool.exe.

1. Se connecter au serveur de configuration et lancer CSPSConfigtool.exe
2. Choisissez le compte que vous souhaitez modifier, puis cliquez sur **Modifier**
3. Entrez les nouvelles informations d’identification et cliquez sur **OK** .

## <a name="add-credentials-for-mobility-service-installation"></a>Ajouter des informations d’identification pour l’installation du service Mobilité

Si vous n’avez pas ajouté des informations d’identification lors du déploiement OVF du serveur de configuration,

1. Après la [connexion](#access-configuration-server), sélectionnez **Gérer les informations d’identification de machine virtuelle** .
2. Cliquez sur **Ajouter des informations d’identification de machine virtuelle** .
    ![Capture d’écran montrant le volet Gérer les informations d’identification de machine virtuelle avec le lien Ajouter des informations d’identification de machine virtuelle.](media/vmware-azure-manage-configuration-server/add-mobility-credentials.png)
3. Entrez les nouvelles informations d’identification et cliquez sur **Ajouter** .

Vous pouvez également ajouter des informations d’identification via CSPSConfigtool.exe.

1. Se connecter au serveur de configuration et lancer CSPSConfigtool.exe
2. Cliquez sur **Ajouter** , entrez les nouvelles informations d’identification et cliquez sur **OK** .

## <a name="modify-proxy-settings"></a>Modifier les paramètres du proxy

Modifiez les paramètres du proxy que la machine serveur de configuration doit utiliser pour l’accès Internet à Azure. Si vous disposez d’une machine serveur de processus, en plus du serveur de processus par défaut exécuté sur la machine serveur de configuration, modifiez les paramètres sur les deux machines.

1. Après la [connexion](#access-configuration-server) au serveur de configuration, sélectionnez **Gérer la connectivité** .
2. Mettez à jour les valeurs de proxy. Ensuite, sélectionnez **Enregistrer** pour mettre à jour les paramètres.

## <a name="add-a-network-adapter"></a>Ajouter une carte réseau

Le modèle OVF (Open Virtualization Format) déploie la machine virtuelle du serveur de configuration avec une seule carte réseau.

- Vous pouvez [ajouter une autre carte réseau à la machine virtuelle](vmware-azure-deploy-configuration-server.md#add-an-additional-adapter), mais vous devez le faire avant d’inscrire le serveur de configuration dans le coffre.
- Pour ajouter une carte après avoir inscrit le serveur de configuration dans le coffre, ajoutez la carte dans les propriétés de la machine virtuelle. Ensuite, vous devez [réinscrire](#reregister-a-configuration-server-in-the-same-vault) le serveur dans le coffre.

## <a name="how-to-renew-ssl-certificates"></a>Procédure de renouvellement des certificats SSL

Le serveur de configuration comprend un serveur web intégré, qui orchestre les activités des agents Mobilité sur tous les ordinateurs protégés, des serveurs de traitement intégrés/scale-out et des serveurs cibles maîtres qui y sont connectés. Le serveur web utilise un certificat SSL pour authentifier les clients. Le certificat expire au bout de trois ans et peut être renouvelé à tout moment.

### <a name="check-expiry"></a>Vérifier la date d’expiration

La date d’expiration est indiquée sous **Intégrité de Configuration Server** . Pour les déploiements de serveurs de configuration effectués avant mai 2016, le certificat expire au bout d’un an. Si votre certificat est sur le point d’expirer, voici ce qui se produit :

- Deux mois (ou moins) avant la date d’expiration, le service commence à envoyer des notifications via le portail et par e-mail (si vous avez souscrit aux notifications Site Recovery).
- Une bannière de notification s’affiche sur la page de ressources du coffre. Pour plus d’informations, sélectionnez la bannière.
- Si le bouton **Mettre à niveau maintenant** s’affiche, cela indique que certains composants de votre environnement n’ont pas été mis à niveau vers 9.4.xxxx.x ou une version supérieure. Mettez à niveau les composants avant de renouveler le certificat. Vous ne pouvez pas renouveler de versions antérieures.

### <a name="if-certificates-are-yet-to-expire"></a>Si les certificats n’ont pas encore expiré

1. Pour effectuer le renouvellement, ouvrez **Infrastructure Site Recovery** > **Serveur de configuration** dans le coffre. Sélectionnez le serveur de configuration dont vous avez besoin.
2. Vérifiez que tous les composants (serveurs de traitement scale-out, serveurs cibles maîtres et agents Mobilité) sur tous les ordinateurs protégés sont à jour et connectés.
3. Maintenant, sélectionnez **Renouveler les certificats** .
4. Suivez attentivement les instructions de cette page et cliquez sur OK pour renouveler les certificats sur le serveur de configuration sélectionné et les composants associés.

### <a name="if-certificates-have-already-expired"></a>Si les certificats ont déjà expiré

1. Après expiration, **il n’est pas possible de renouveler les certificats sur le Portail Azure** . Avant toute chose, vérifiez que tous les composants (serveurs de traitement scale-out, serveurs cibles maîtres et agents Mobilité) sur tous les ordinateurs protégés sont à jour et connectés.
2. **Ne suivez cette procédure que si les certificats ont déjà expiré.** Connectez-vous au serveur de configuration, accédez au lecteur C > Program Data > Site Recovery > home > svsystems > bin et exécutez l’outil exécuteur « RenewCerts » en tant qu’administrateur.
3. Une fenêtre d’exécution PowerShell s’ouvre et déclenche le renouvellement des certificats. Cette opération peut durer jusqu’à 15 minutes. Ne fermez pas la fenêtre avant la fin du renouvellement.

:::image type="content" source="media/vmware-azure-manage-configuration-server/renew-certificates.png" alt-text="RenewCertificates":::

## <a name="reregister-a-configuration-server-in-the-same-vault"></a>Réinscrire un serveur de configuration dans le même coffre

Vous pouvez réinscrire le serveur de configuration dans le même coffre, si nécessaire. Si vous disposez d’une machine serveur de processus supplémentaire, en plus du serveur de processus par défaut exécuté sur la machine serveur de configuration, réinscrivez les deux machines.


1. Dans le coffre, cliquez sur **Gérer** > **Infrastructure Site Recovery** > **Serveurs de configuration** .
2. Dans **Serveurs** , sélectionnez **Télécharger une clé d’inscription** pour télécharger le fichier d’informations d’identification du coffre.
3. Connectez-vous à la machine du serveur de configuration.
4. Sous **%ProgramData%\ASR\home\svsystems\bin** , ouvrez **cspsconfigtool.exe** .
5. Sous l’onglet **Inscription du coffre** , sélectionnez **Parcourir** et recherchez le fichier d’informations d’identification du coffre que vous avez téléchargé.
6. Si nécessaire, fournissez les détails du serveur proxy. Sélectionnez ensuite **Inscription** .
7. Ouvrez une fenêtre Commande d’administration PowerShell, puis exécutez la commande suivante :
   ```
    $pwd = ConvertTo-SecureString -String MyProxyUserPassword
    Set-OBMachineSetting -ProxyServer http://myproxyserver.domain.com -ProxyPort PortNumber – ProxyUserName domain\username -ProxyPassword $pwd
   ```

    >[!NOTE]
    >Afin **d’extraire les derniers certificats** du serveur de configuration et de les transmettre au serveur de traitement Scale-out, exécutez la commande *"\<Installation Drive\Microsoft Azure Site Recovery\agent\cdpcli.exe>"--registermt* .

8. Enfin, redémarrez la machine en exécutant la commande suivante.
   ```
        net stop obengine
        net start obengine
   ```


## <a name="register-a-configuration-server-with-a-different-vault"></a>Inscrire un serveur de configuration auprès d’un autre coffre

> [!WARNING]
> L’étape suivante dissocie le serveur de configuration du coffre actuel, entraînant l’arrêt de la réplication de toutes les machines virtuelles protégées sur le serveur de configuration.

1. Connectez-vous au serveur de configuration.
2. Ouvrez une fenêtre Commande d’administration PowerShell, puis exécutez la commande suivante :

    ```
    reg delete "HKLM\Software\Microsoft\Azure Site Recovery\Registration"
    net stop dra
    ```
3. Lancez le portail du navigateur de l’appliance du serveur de configuration à l’aide du raccourci sur votre bureau.
4. Exécutez les étapes d’inscription comme pour [l’inscription](vmware-azure-tutorial.md#register-the-configuration-server) d’un nouveau serveur de configuration.

## <a name="upgrade-the-configuration-server"></a>Mettre à niveau le serveur de configuration

Vous exécutez des correctifs cumulatifs pour mettre à jour le serveur de configuration. Les mises à jour peuvent être appliquées jusqu’aux versions N-4. Par exemple :

- Si vous exécutez la version 9.7, 9.8, 9.9 ou 9.10, vous pouvez mettre à niveau directement vers la version 9.11.
- Si vous exécutez la version 9.6 ou une version antérieure, et souhaitez mettre à niveau vers la version 9.11, vous devez d’abord mettre à niveau vers la version 9.7, avant de mettre à niveau vers la version 9.11.

Pour obtenir des instructions détaillées sur la déclaration de support relative aux composants Azure Site Recovery, cliquez [ici](./service-updates-how-to.md#support-statement-for-azure-site-recovery).
Des liens vers des correctifs cumulatifs pour la mise à niveau de toutes les versions du serveur de configuration sont disponibles [ici](./service-updates-how-to.md#links-to-currently-supported-update-rollups).

> [!IMPORTANT]
> À chaque publication d’une nouvelle version « N » d’un composant Azure Site Recovery, toutes les versions inférieures à « N-4 » sont considérées comme non couvertes par le support. Il est toujours recommandé de procéder à une mise à niveau vers les dernières versions disponibles.</br>
> Pour obtenir des instructions détaillées sur la déclaration de support relative aux composants Azure Site Recovery, cliquez [ici](./service-updates-how-to.md#support-statement-for-azure-site-recovery).

Mettez à niveau le serveur comme suit :

1. Dans le coffre, accédez à **Gérer** > **Infrastructure Site Recovery** > **Serveurs de configuration** .
2. Si une mise à jour est disponible, un lien s’affiche dans la colonne **Version de l’agent** >.
    ![Mettre à jour](./media/vmware-azure-manage-configuration-server/update2.png)
3. Téléchargez le fichier du programme d’installation des mises à jour sur le serveur de configuration.

    ![Capture d’écran montrant où cliquer pour télécharger le fichier du programme d’installation de mise à jour.](./media/vmware-azure-manage-configuration-server/update1.png)

4. Double-cliquez pour exécuter le programme d’installation.
5. Le programme d’installation détecte la version actuelle en cours d’exécution sur la machine. Cliquez sur **Oui** pour démarrer la mise à niveau.
6. À l’issue de la mise à niveau, la configuration du serveur valide.

    ![Capture montrant la configuration de validation du serveur terminée.](./media/vmware-azure-manage-configuration-server/update3.png)

7. Cliquez sur **Terminer** pour fermer le programme d’installation.
8. Pour mettre à niveau le reste des composants Site Recovery, reportez-vous à nos [conseils de mise à niveau](./service-updates-how-to.md#vmware-vmphysical-server-disaster-recovery-to-azure).

## <a name="upgrade-configuration-serverprocess-server-from-the-command-line"></a>Mettre à niveau le serveur de configuration/serveur de traitement à partir de la ligne de commande

Exécutez le fichier d’installation de la manière suivante :

  ```
  UnifiedSetup.exe [/ServerMode <CS/PS>] [/InstallDrive <DriveLetter>] [/MySQLCredsFilePath <MySQL credentials file path>] [/VaultCredsFilePath <Vault credentials file path>] [/EnvType <VMWare/NonVMWare>] [/PSIP <IP address to be used for data transfer] [/CSIP <IP address of CS to be registered with>] [/PassphraseFilePath <Passphrase file path>]
  ```

### <a name="sample-usage"></a>Exemple d’utilisation
  ```
  MicrosoftAzureSiteRecoveryUnifiedSetup.exe /q /x:C:\Temp\Extracted
  cd C:\Temp\Extracted
  UNIFIEDSETUP.EXE /AcceptThirdpartyEULA /servermode "CS" /InstallLocation "D:\" /MySQLCredsFilePath "C:\Temp\MySQLCredentialsfile.txt" /VaultCredsFilePath "C:\Temp\MyVault.vaultcredentials" /EnvType "VMWare"
  ```


### <a name="parameters"></a>Paramètres

|Nom du paramètre| Type | Description| Valeurs|
|-|-|-|-|
| /ServerMode|Obligatoire|Spécifie si les serveurs de configuration et de processus doivent être installés ou si seul le serveur de processus doit être installé|CS<br>PS|
|/InstallLocation|Obligatoire|Dossier d’installation des composants| N’importe quel dossier sur l’ordinateur|
|/MySQLCredsFilePath|Obligatoire|Chemin d’accès du fichier où sont stockées les informations d’identification du serveur MySQL|Le fichier doit être au format spécifié ci-dessous|
|/VaultCredsFilePath|Obligatoire|Chemin d’accès du fichier d’informations d’identification du coffre|Chemin d’accès valide du fichier|
|/EnvType|Obligatoire|Type d’environnement à protéger |VMware<br>NonVMware|
|/PSIP|Obligatoire|Adresse IP de la carte réseau à utiliser pour le transfert de données de réplication| Une adresse IP valide|
|/CSIP|Obligatoire|Adresse IP de la carte réseau sur laquelle le serveur de configuration écoute| Une adresse IP valide|
|/PassphraseFilePath|Obligatoire|Chemin d’accès complet vers l’emplacement du fichier de phrase secrète|Chemin d’accès valide du fichier|
|/BypassProxy|Facultatif|Indique que le serveur de configuration se connecte à Azure sans proxy|Pour obtenir cette valeur de Venu|
|/ProxySettingsFilePath|Facultatif|Paramètres de proxy (Le proxy par défaut nécessite une authentification, sinon il faut un proxy personnalisé)|Le fichier doit être au format spécifié ci-dessous|
|DataTransferSecurePort|Facultatif|Numéro de port sur le PSIP à utiliser pour les données de réplication| Numéro de port valide (valeur par défaut : 9433)|
|/SkipSpaceCheck|Facultatif|Ignorer la vérification des espaces pour le disque du cache| |
|/AcceptThirdpartyEULA|Obligatoire|Cet indicateur implique l’acceptation du CLUF tiers| |
|/ShowThirdpartyEULA|Facultatif|Affiche le CLUF tiers. S’il est fourni en entrée, tous les autres paramètres sont ignorés| |



### <a name="create-file-input-for-mysqlcredsfilepath"></a>Créer le fichier d’entrée pour MYSQLCredsFilePath

Le paramètre MySQLCredsFilePath accepte un fichier comme entrée. Créez le fichier au format suivant et transmettez-le comme paramètre MySQLCredsFilePath d’entrée.
```ini
[MySQLCredentials]
MySQLRootPassword = "Password>"
MySQLUserPassword = "Password"
```
### <a name="create-file-input-for-proxysettingsfilepath"></a>Créer le fichier d’entrée pour ProxySettingsFilePath
Le paramètre ProxySettingsFilePath prend un fichier en tant qu’entrée. Créez le fichier au format suivant et transmettez-le comme paramètre ProxySettingsFilePath d’entrée.

```ini
[ProxySettings]
ProxyAuthentication = "Yes/No"
Proxy IP = "IP Address"
ProxyPort = "Port"
ProxyUserName="UserName"
ProxyPassword="Password"
```

## <a name="delete-or-unregister-a-configuration-server"></a>Supprimer un serveur de configuration ou annuler son inscription

1. [Désactivez la protection](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure) de toutes les machines virtuelles sous le serveur de configuration.
2. [Dissociez](vmware-azure-set-up-replication.md#disassociate-or-delete-a-replication-policy) et [supprimez](vmware-azure-set-up-replication.md#disassociate-or-delete-a-replication-policy) toutes les stratégies de réplication du serveur de configuration.
3. [Supprimez](vmware-azure-manage-vcenter.md#delete-a-vcenter-server) tous les serveurs vCenter/hôtes vSphere associés au serveur de configuration.
4. Dans le coffre, ouvrez **Infrastructure Site Recovery** > **Serveurs de configuration** .
5. Sélectionnez le serveur de configuration que vous souhaitez supprimer. Ensuite, dans la page **Détails** , sélectionnez **Supprimer** .

    ![Suppression d’un serveur de configuration](./media/vmware-azure-manage-configuration-server/delete-configuration-server.png)


### <a name="delete-with-powershell"></a>Supprimer avec PowerShell

Vous pouvez également supprimer le serveur de configuration à l’aide de PowerShell.

1. [Installez](/powershell/azure/install-Az-ps) le module Azure PowerShell.
2. Connectez-vous à votre compte Azure à l’aide de cette commande :

    `Connect-AzAccount`
3. Sélectionnez l’abonnement du coffre.

     `Get-AzSubscription –SubscriptionName <your subscription name> | Select-AzSubscription`
3.  Définir le contexte d’archivage.

    ```
    $vault = Get-AzRecoveryServicesVault -Name <name of your vault>
    Set-AzSiteRecoveryVaultSettings -ARSVault $vault
    ```
4. Récupérez le serveur de configuration.

    `$fabric = Get-AzSiteRecoveryFabric -FriendlyName <name of your configuration server>`
6. Supprimez le serveur de configuration.

    `Remove-AzSiteRecoveryFabric -Fabric $fabric [-Force]`

> [!NOTE]
> Vous pouvez utiliser l’option **-Force** dans Remove-AzSiteRecoveryFabric pour forcer la suppression du serveur de configuration.

## <a name="generate-configuration-server-passphrase"></a>Générer la phrase secrète du serveur de configuration

1. Connectez-vous à votre serveur de configuration, puis ouvrez une fenêtre d’invite de commandes en tant qu’administrateur.
2. Pour accéder au dossier bin, exécutez la commande **cd %ProgramData%\ASR\home\svsystems\bin** .
3. Pour générer le fichier de phrase secrète, exécutez la commande **genpassphrase.exe -v > MobSvc.passphrase** .
4. Votre phrase secrète est stockée dans le fichier situé à l’emplacement **%ProgramData%\ASR\home\svsystems\bin\MobSvc.passphrase** .

## <a name="refresh-configuration-server"></a>Actualiser le serveur de configuration

1. Dans le portail Azure, accédez à **Coffre Recovery Services** > **Gérer** > **Infrastructure Site Recovery** > **Pour les machines VMware et physiques** > **Serveurs de configuration** .
2. Cliquez sur le serveur de configuration que vous souhaitez actualiser.
3. Dans le panneau avec les détails du serveur de configuration choisi, cliquez sur **Plus** > **Actualiser le serveur** .
4. Suivez la progression du travail sous **Coffre Recovery Services** > **Surveillance** > **Travaux Site Recovery** .

## <a name="failback-requirements"></a>Conditions requises pour la restauration automatique

Le serveur de configuration local doit être en cours d’exécution et connecté au cours de la reprotection et de la restauration automatique. Pour que la restauration automatique réussisse, la machine virtuelle faisant l’objet d’une restauration doit exister dans la base de données du serveur de configuration.

Veillez à effectuer des sauvegardes régulières de votre serveur de configuration. Si un incident se produit et que le serveur de configuration est perdu, vous devez tout d’abord le restaurer à partir d’une copie de sauvegarde, et vérifier que le serveur de configuration restauré a la même adresse IP que celle avec laquelle il a été inscrit dans le coffre. La restauration automatique ne fonctionnera pas si une adresse IP différente est utilisée pour le serveur de configuration restauré.

## <a name="next-steps"></a>Étapes suivantes

Consultez les tutoriels pour en savoir plus sur la configuration de la reprise après sinistre des [machines virtuelles VMware](vmware-azure-tutorial.md) dans Azure.