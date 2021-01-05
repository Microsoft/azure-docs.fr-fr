---
title: Gérer le serveur de configuration pour les serveurs physiques dans Azure Site Recovery
description: Cet article explique comment gérer le serveur de configuration Azure Site Recovery pour la récupération d’urgence d’un serveur physique sur Azure.
services: site-recovery
author: mayurigupta13
ms.service: site-recovery
ms.topic: article
ms.date: 02/28/2019
ms.author: mayg
ms.openlocfilehash: ff612b7c052ead5658ea4bbfafd7aace51ba3c02
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017438"
---
# <a name="manage-the-configuration-server-for-physical-server-disaster-recovery"></a>Gérer le serveur de configuration pour la reprise après sinistre d’un serveur physique

Vous configurez un serveur de configuration local quand vous utilisez le service [Azure Site Recovery](site-recovery-overview.md) pour la reprise après sinistre de serveurs physiques sur Azure. Le serveur de configuration coordonne la communication entre les machines locales et Azure, et gère la réplication des données. Cet article résume les tâches courantes de gestion du serveur de configuration après son déploiement.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

Le tableau répertorie les prérequis du déploiement d'une machine de serveur de configuration locale.

| **Composant** | **Prérequis** |
| --- |---|
| Cœurs d’unité centrale| 8 |
| RAM | 16 Go|
| Nombre de disques | 3, y compris le disque du système d’exploitation, le disque de cache du serveur de processus et le lecteur de conservation pour la restauration automatique |
| Espace disque disponible (cache du serveur de traitement) | 600 Go
| Espace disque disponible (disque de rétention) | 600 Go|
| Système d’exploitation  | Windows Server 2012 R2 <br> Windows Server 2016 |
| Paramètres régionaux du système d’exploitation | Anglais (US)|
| Version VMware vSphere PowerCLI | Non requis|
| Rôles Windows Server | N’activez pas ces rôles : <br> - Active Directory Domain Services <br>- Internet Information Services <br> - Hyper-V |
| Stratégies de groupe| N’activez pas ces stratégies de groupe : <br> - Empêcher l’accès à l’invite de commandes <br> - Empêcher l’accès aux outils de modification du Registre <br> - Logique de confiance pour les pièces jointes <br> - Activer l’exécution des scripts <br> [En savoir plus](/previous-versions/windows/it-pro/windows-7/gg176671(v=ws.10))|
| IIS | - Aucun site web par défaut préexistant <br> - Activer l’[authentification anonyme](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc731244(v=ws.10)) <br> - Activer le paramètre [FastCGI](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc753077(v=ws.10))  <br> - Aucune application/aucun site web préexistants ne doivent écouter le port 443<br>|
| Type de carte réseau | VMXNET3 (en cas de déploiement comme machine virtuelle VMware) |
| Type d’adresse IP | statique |
| Accès à Internet | Le serveur doit également accéder à ces URL : <br> - \*.accesscontrol.windows.net<br> - \*.backup.windowsazure.com <br>- \*.store.core.windows.net<br> - \*.blob.core.windows.net<br> - \*.hypervrecoverymanager.windowsazure.com <br> - `https://management.azure.com` <br> - *.services.visualstudio.com <br> - https://dev.mysql.com/get/Downloads/MySQLInstaller/mysql-installer-community-5.7.20.0.msi (non requis pour les serveurs de processus de Scale-out) <br> - time.nist.gov <br> - time.windows.com |
| Ports | 443 (Orchestration du canal de contrôle)<br>9443 (Transport de données)|

## <a name="download-the-latest-installation-file"></a>Télécharger le fichier d’installation le plus récent

La dernière version du fichier d’installation du serveur configuration est disponible dans le portail Site Recovery. De plus, elle peut être téléchargée directement sur le [Centre de téléchargement Microsoft](https://aka.ms/unifiedsetup).

1. Connectez-vous au portail Azure et accédez à votre coffre Recovery Services.
2. Sélectionnez **Infrastructure Site Recovery** > **Serveurs de configuration** (sous For VMware & Physical Machines [Pour VMware et machines physiques]).
3. Cliquez sur le bouton **+Serveurs**.
4. Dans la page **Ajouter un serveur** , cliquez sur le bouton Télécharger pour télécharger la clé d’inscription. Vous avez besoin de cette clé pendant l’installation du serveur de configuration pour l’inscrire auprès du service Azure Site Recovery.
5. Cliquez sur le lien **Download the Microsoft Azure Site Recovery Unified Setup (Télécharger le programme d’installation unifiée de Microsoft Azure Site Recovery)** pour télécharger la dernière version du serveur de configuration.

   ![Page de téléchargement](./media/physical-manage-configuration-server/downloadcs.png)


## <a name="install-and-register-the-server"></a>Installer et inscrire le serveur

1. Exécutez le fichier d’installation unifiée.
2. Dans **Avant de commencer**, sélectionnez **Installer le serveur de configuration et le serveur de traitement**.

    ![Avant de commencer](./media/physical-manage-configuration-server/combined-wiz1.png)

3. Dans **Licence de logiciel tiers**, cliquez sur **J’accepte** pour télécharger et installer MySQL.
4. Dans **Paramètres Internet**, indiquez de quelle manière le fournisseur qui s’exécute sur le serveur de configuration doit se connecter à Azure Site Recovery par le biais d’Internet. Vérifiez que vous avez autorisé les URL requises.

    - Si vous voulez vous connecter avec le proxy actuellement configuré sur la machine, sélectionnez **Se connecter à Azure Site Recovery avec un serveur proxy**.
    - Si vous voulez que le fournisseur se connecte directement, sélectionnez **Se connecter directement à Azure Site Recovery sans serveur proxy**.
    - Si le proxy existant nécessite une authentification, ou si vous voulez utiliser un proxy personnalisé pour la connexion au fournisseur, sélectionnez **Se connecter avec des paramètres de proxy personnalisés**, et spécifiez l’adresse, le port et les informations d’authentification.
     ![Pare-feu](./media/physical-manage-configuration-server/combined-wiz4.png)
6. Dans **Vérification de la configuration requise**, le programme d’installation procède à une vérification afin de garantir le bon déroulement de l’installation. Si un avertissement s’affiche à propos de la **vérification de la synchronisation globale**, vérifiez que l’heure de l’horloge système (paramètres **Date et heure**) est identique à celle du fuseau horaire.

    ![Prérequis](./media/physical-manage-configuration-server/combined-wiz5.png)
7. Dans **Configuration MySQL**, créez des informations d’identification pour vous connecter à l’instance de serveur MySQL installée.

    ![MySQL](./media/physical-manage-configuration-server/combined-wiz6.png)
8. Dans **Détails de l’environnement**, indiquez si vous voulez répliquer des machines virtuelles VMware. Si c’est le cas, le programme d’installation vérifie que PowerCLI 6.0 est installé.
9. Dans **Emplacement d’installation**, sélectionnez l’emplacement où vous voulez installer les fichiers binaires et stocker le cache. Le lecteur sélectionné doit présenter au moins 5 Go d’espace disque disponible. Toutefois, nous vous recommandons d’utiliser un lecteur de cache présentant au moins 600 Go d’espace disponible.

    ![Emplacement d’installation](./media/physical-manage-configuration-server/combined-wiz8.png)
10. Dans **Sélection du réseau**, commencez par sélectionner la carte réseau que le serveur de processus intégré utilise pour l’installation push du service Mobilité sur les machines sources, puis sélectionnez la carte réseau que le serveur de configuration utilise pour la connectivité avec Azure. Le port 9443 est le port utilisé par défaut pour envoyer et recevoir le trafic de réplication, mais vous pouvez le modifier en fonction des exigences de votre environnement. Outre le port 9443, nous ouvrons également le port 443, qui est utilisé par un serveur web pour orchestrer les opérations de réplication. N’utilisez pas le port 443 pour envoyer ou recevoir le trafic de réplication.

    ![Sélection du réseau](./media/physical-manage-configuration-server/combined-wiz9.png)


11. Dans **Résumé**, passez en revue les informations, puis cliquez sur **Installer**. Une fois l’installation terminée, une phrase secrète est générée. Vous en aurez besoin lorsque vous activerez la réplication. Par conséquent, copiez-la et conservez-la en lieu sûr.


Une fois l’inscription terminée, le serveur s’affiche sur le panneau **Paramètres** > **Serveurs** du coffre.


## <a name="install-from-the-command-line"></a>Installer à partir de la ligne de commande

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
MySQLRootPassword = "Password"
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
## <a name="modify-proxy-settings"></a>Modifier les paramètres du proxy

Vous pouvez modifier les paramètres de proxy pour la machine du serveur de configuration de la manière suivante :

1. Connectez-vous au serveur de configuration.
2. Lancez l’exécutable cspsconfigtool.exe à l’aide du raccourci sur votre bureau.
3. Cliquez sur l’onglet **Vault Registration (Inscription du coffre)** .
4. Téléchargez un nouveau fichier d’inscription du coffre à partir du portail et indiquez-le comme entrée de l’outil.

   ![register-configuration-server](./media/physical-manage-configuration-server/register-csconfiguration-server.png)
5. Fournissez les informations sur le nouveau proxy, puis cliquez sur le bouton **Inscrire**.
6. Ouvrez une fenêtre de commandes PowerShell administrateur.
7. Exécutez la commande suivante :

   ```powershell
   $Pwd = ConvertTo-SecureString -String MyProxyUserPassword
   Set-OBMachineSetting -ProxyServer http://myproxyserver.domain.com -ProxyPort PortNumber –ProxyUserName domain\username -ProxyPassword $Pwd
   net stop obengine
   net start obengine
   ```

   > [!WARNING]
   > Si vous avez joint des serveurs de processus supplémentaires à ce serveur de configuration, vous devez [corriger les paramètres de proxy sur tous les serveurs de processus de scale-out](vmware-azure-manage-process-server.md#modify-proxy-settings-for-an-on-premises-process-server) de votre déploiement.

## <a name="reregister-a-configuration-server-with-the-same-vault"></a>Réinscrire un serveur de configuration auprès du même coffre
1. Connectez-vous à votre serveur de configuration.
2. Lancez l’exécutable cspsconfigtool.exe à l’aide du raccourci sur votre bureau.
3. Cliquez sur l’onglet **Vault Registration (Inscription du coffre)** .
4. Téléchargez un nouveau fichier d’inscription à partir du portail et indiquez-le comme entrée de l’outil.
      ![register-configuration-server](./media/physical-manage-configuration-server/register-csconfiguration-server.png)
5. Indiquez les détails du serveur proxy, puis cliquez sur le bouton **Inscrire**.  
6. Ouvrez une fenêtre de commandes PowerShell administrateur.
7. Exécutez la commande suivante :

    ```powershell
    $Pwd = ConvertTo-SecureString -String MyProxyUserPassword
    Set-OBMachineSetting -ProxyServer http://myproxyserver.domain.com -ProxyPort PortNumber –ProxyUserName domain\username -ProxyPassword $Pwd
    net stop obengine
    net start obengine
    ```

   > [!WARNING]
   > Si vous avez plusieurs serveurs de processus, vous devez les [réinscrire](vmware-azure-manage-process-server.md#reregister-a-process-server).

## <a name="register-a-configuration-server-with-a-different-vault"></a>Inscrire un serveur de configuration auprès d’un autre coffre

> [!WARNING]
> L’étape suivante dissocie le serveur de configuration du coffre actuel, entraînant l’arrêt de la réplication de toutes les machines virtuelles protégées sur le serveur de configuration.

1. Connectez-vous au serveur de configuration.
2. À partir d’une invite de commandes Administrateur, exécutez la commande suivante :

    ```
    reg delete HKLM\Software\Microsoft\Azure Site Recovery\Registration
    net stop dra
    ```
3. Lancez l’exécutable cspsconfigtool.exe à l’aide du raccourci sur votre bureau.
4. Cliquez sur l’onglet **Vault Registration (Inscription du coffre)** .
5. Téléchargez un nouveau fichier d’inscription à partir du portail et indiquez-le comme entrée de l’outil.
6. Indiquez les détails du serveur proxy, puis cliquez sur le bouton **Inscrire**.  
7. Ouvrez une fenêtre de commandes PowerShell administrateur.
8. Exécutez la commande suivante :
    ```powershell
    $pwd = ConvertTo-SecureString -String MyProxyUserPassword
    Set-OBMachineSetting -ProxyServer http://myproxyserver.domain.com -ProxyPort PortNumber –ProxyUserName domain\username -ProxyPassword $pwd
    net stop obengine
    net start obengine
    ```

## <a name="upgrade-a-configuration-server"></a>Mettre à niveau un serveur de configuration

Vous exécutez des correctifs cumulatifs pour mettre à jour le serveur de configuration. Les mises à jour peuvent être appliquées jusqu’aux versions N-4. Par exemple :

- Si vous exécutez la version 9.7, 9.8, 9.9 ou 9.10, vous pouvez mettre à niveau directement vers la version 9.11.
- Si vous exécutez la version 9.6 ou une version antérieure, et souhaitez mettre à niveau vers la version 9.11, vous devez tout d’abord mettre à niveau vers la version 9.7. avant d’effectuer la mise à niveau vers 9.11.

Des liens vers des correctifs cumulatifs pour la mise à niveau de toutes les versions du serveur de configuration sont disponibles dans la [page wiki relative aux mises à jour](https://social.technet.microsoft.com/wiki/contents/articles/38544.azure-site-recovery-service-updates.aspx).

Mettez à niveau le serveur comme suit :

1. Téléchargez le fichier du programme d’installation des mises à jour sur le serveur de configuration.
2. Double-cliquez pour exécuter le programme d’installation.
3. Le programme d’installation détecte la version actuelle en cours d’exécution sur la machine.
4. Cliquez sur **OK** pour confirmer, puis exécutez la mise à niveau. 


## <a name="delete-or-unregister-a-configuration-server"></a>Supprimer un serveur de configuration ou annuler son inscription

> [!WARNING]
> Effectuez les opérations suivantes avant de désaffecter votre serveur de configuration.
> 1. [Désactivez la protection](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure) de toutes les machines virtuelles relevant de ce serveur de configuration.
> 2. [Dissociez ](vmware-azure-set-up-replication.md#disassociate-or-delete-a-replication-policy) et [supprimez](vmware-azure-set-up-replication.md#disassociate-or-delete-a-replication-policy) toutes les stratégies de réplication du serveur de configuration.
> 3. [Supprimez](vmware-azure-manage-vcenter.md#delete-a-vcenter-server) tous les serveurs vCenter/hôtes vSphere associés au serveur de configuration.


### <a name="delete-the-configuration-server-from-azure-portal"></a>Supprimer le serveur de configuration du portail Azure
1. Dans le portail Azure, sélectionnez **Infrastructure Site Recovery** > **Serveurs de configuration** dans le menu Coffre.
2. Cliquez sur le serveur de configuration que vous souhaitez désaffecter.
3. Dans la page des détails du serveur de configuration, cliquez sur le bouton **Supprimer**.
4. Cliquez sur **Oui** pour confirmer la suppression du serveur.

### <a name="uninstall-the-configuration-server-and-its-dependencies"></a>Désinstaller le serveur de configuration et ses dépendances
> [!TIP]
>   Si vous envisagez de réutiliser le serveur de configuration avec Azure Site Recovery, vous pouvez passer directement à l’étape 4.

1. Connectez-vous au serveur de configuration en tant qu’administrateur.
2. Ouvrez le Panneau de configuration > Programme > Désinstaller des programmes
3. Désinstallez les programmes dans la séquence suivante :
   * Agent Microsoft Azure Recovery Services
   * Service Mobilité/Serveur cible maître Microsoft Azure Site Recovery
   * Fournisseur Microsoft Azure Site Recovery
   * Serveur de configuration Microsoft Azure Site Recovery/Serveur de traitement
   * Dépendances du serveur de configuration Microsoft Azure Site Recovery
   * MySQL Server 5.5
4. Exécutez la commande suivante à partir d’une invite de commandes d’administration :
   ```
   reg delete HKLM\Software\Microsoft\Azure Site Recovery\Registration
   ```

## <a name="delete-or-unregister-a-configuration-server-powershell"></a>Supprimer ou désinscrire un serveur de configuration (PowerShell)

1. [Installez](/powershell/azure/install-Az-ps) le module Azure PowerShell.
2. Connectez-vous à votre compte Azure à l’aide de la commande suivante :
    
    `Connect-AzAccount`
3. Sélectionnez l’abonnement sous lequel le coffre est présent.

     `Get-AzSubscription –SubscriptionName <your subscription name> | Select-AzSubscription`
3.  Configurez votre contexte de coffre.
    
    ```powershell
    $Vault = Get-AzRecoveryServicesVault -Name <name of your vault>
    Set-AzSiteRecoveryVaultSettings -ARSVault $Vault
    ```
4. Sélectionnez votre serveur de configuration.

    `$Fabric = Get-AzSiteRecoveryFabric -FriendlyName <name of your configuration server>`
6. Supprimez le serveur de configuration.

    `Remove-AzSiteRecoveryFabric -Fabric $Fabric [-Force]`

> [!NOTE]
> L’option **-Force** dans Remove-AzSiteRecoveryFabric peut être utilisée pour forcer la suppression du serveur de configuration.

## <a name="renew-tlsssl-certificates"></a>Renouveler les certificats TLS/SSL
Le serveur de configuration possède un serveur web intégré, qui orchestre les activités du service Mobilité, des serveurs de processus et des serveurs maîtres cibles connectés à celui-ci. Le serveur web utilise un certificat TLS/SSL pour authentifier les clients. Le certificat expire au bout de trois ans et peut être renouvelé à tout moment.

### <a name="check-expiry"></a>Vérifier la date d’expiration

Pour les déploiements de serveurs de configuration effectués avant mai 2016, le certificat expire au bout d’un an. Si votre certificat est sur le point d’expirer :

- Deux mois (ou moins) avant la date d’expiration, le service commence à envoyer des notifications via le portail et par e-mail (si vous avez souscrit aux notifications d’Azure Site Recovery).
- Une bannière de notification s’affiche sur la page de ressources du coffre. Cliquez sur la bannière pour plus de détails.
- Si un bouton **Mettre à niveau maintenant** apparaît, il indique que certains composants de votre environnement n’ont pas été mis à niveau vers 9.4.xxxx.x ou une version supérieure. Mettez à niveau les composants avant de renouveler le certificat. Vous ne pouvez pas renouveler de versions antérieures.

### <a name="renew-the-certificate"></a>Renouveler le certificat

1. Dans le coffre, accédez à **Infrastructure Site Recovery** > **Serveur de configuration**, puis cliquez sur le serveur de configuration requis.
2. La date d’expiration est indiquée sous **Intégrité de Configuration Server**.
3. Cliquez sur **Renouveler les certificats**. 




## <a name="common-issues"></a>Problèmes courants
[!INCLUDE [site-recovery-vmware-to-azure-install-register-issues](../../includes/site-recovery-vmware-to-azure-install-register-issues.md)]

## <a name="next-steps"></a>Étapes suivantes

Consultez les tutoriels pour en savoir plus sur la configuration de la reprise après sinistre de [serveurs physiques](./physical-azure-disaster-recovery.md) sur Azure.
