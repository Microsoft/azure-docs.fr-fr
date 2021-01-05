---
title: Résoudre les problèmes de l’agent Sauvegarde Azure
description: Dans cet article, découvrez comment résoudre les problèmes liés à l’installation et l’inscription de l’agent Sauvegarde Azure.
ms.topic: troubleshooting
ms.date: 07/15/2019
ms.openlocfilehash: 4ae4142652d9d38d5bf384e5a10d6eeb7e3cc608
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95993837"
---
# <a name="troubleshoot-the-microsoft-azure-recovery-services-mars-agent"></a>Résoudre les problèmes liés à l’agent Microsoft Azure Recovery Services (MARS)

Cet article explique comment résoudre les erreurs qui peuvent survenir lors de la configuration, de l’inscription, de la sauvegarde et de la restauration.

## <a name="basic-troubleshooting"></a>Dépannage de base

Nous vous recommandons de passer en revue les points suivants avant de commencer à résoudre les problèmes liés à l’agent MARS :

- [Vérifiez que l’agent MARS est à jour](https://go.microsoft.com/fwlink/?linkid=229525&clcid=0x409).
- [Vérifiez la connectivité réseau entre l’agent MARS et Azure](#the-microsoft-azure-recovery-service-agent-was-unable-to-connect-to-microsoft-azure-backup).
- Vérifiez que MARS est en cours d’exécution (dans la console Service). Si nécessaire, redémarrez le système et recommencez l’opération.
- [Vérifiez qu’il existe entre 5 et 10 % d’espace de volume disponible à l’emplacement du dossier de travail](./backup-azure-file-folder-backup-faq.md#whats-the-minimum-size-requirement-for-the-cache-folder).
- [Vérifiez si un autre processus ou logiciel antivirus interfère avec le service Sauvegarde Azure](./backup-azure-troubleshoot-slow-backup-performance-issue.md#cause-another-process-or-antivirus-software-interfering-with-azure-backup).
- Si le travail de sauvegarde s’est effectué avec des avertissements, consultez [Travaux de sauvegarde effectués avec des avertissements](#backup-jobs-completed-with-warning)
- Si la sauvegarde planifiée échoue, mais que la sauvegarde manuelle fonctionne, consultez [Les sauvegardes ne s’exécutent pas comme prévu](#backups-dont-run-according-to-schedule).
- Assurez-vous que votre système d’exploitation dispose des dernières mises à jour.
- [Vérifiez que les lecteurs non pris en charge et les fichiers avec des attributs non pris en charge sont exclus de la sauvegarde](backup-support-matrix-mars-agent.md#supported-drives-or-volumes-for-backup).
- Vérifiez que l’horloge sur le système protégé est configurée sur le bon fuseau horaire.
- [Assurez-vous que .NET Framework 4.5.2 (ou une version ultérieure) est installé sur le serveur](https://www.microsoft.com/download/details.aspx?id=30653).
- Si vous essayez de réinscrire votre serveur dans un coffre :
  - Vérifiez si l’agent est désinstallé sur le serveur et s’il est supprimé du portail.
  - Utilisez la même phrase secrète que celle initialement utilisée pour l’inscription du serveur.
- Pour les sauvegardes hors connexion, vérifiez qu’Azure PowerShell 3.7.0 est installé sur l’ordinateur source et l’ordinateur de copie avant de commencer la sauvegarde.
- Si l’agent de sauvegarde est en cours d’exécution sur une machine virtuelle Azure, consultez [cet article](./backup-azure-troubleshoot-slow-backup-performance-issue.md#cause-backup-agent-running-on-an-azure-virtual-machine).

## <a name="invalid-vault-credentials-provided"></a>Informations d’identification du coffre fournies non valides

**Message d’erreur** : Informations d’identification du coffre fournies non valides. Cela signifie que le fichier est endommagé ou qu’il ne contient pas les dernières informations d’identification associées au service de récupération. (ID : 34513)

| Cause | Actions recommandées |
| ---     | ---    |
| **Les informations d’identification du coffre ne sont pas valides** <br/> <br/> Il est possible que les fichiers d’informations de connexion au coffre soient corrompus, aient expiré ou aient une extension de fichier différente de *.vaultCredentials*. (Par exemple, ils peuvent avoir été téléchargés plus de 48 heures avant l’heure de l’inscription.)| [Téléchargez les nouvelles informations d’identification](backup-azure-file-folder-backup-faq.md#where-can-i-download-the-vault-credentials-file) à partir du coffre Recovery Services sur le portail Azure. Suivez ensuite ces étapes : <ul><li> Si vous avez déjà installé et inscrit MARS, ouvrez la console MMC de l’agent Sauvegarde Microsoft Azure. Sélectionnez ensuite **Inscription du serveur** dans le volet **Actions** pour effectuer l’inscription avec les nouvelles informations d’identification. <br/> <li> Si la nouvelle installation échoue, réessayez avec les nouvelles informations d’identification.</ul> **Remarque** : Si plusieurs fichiers d’informations d’identification de coffre ont été téléchargés, seul le plus récent est valide pendant 48 heures. Nous vous recommandons de télécharger un nouveau fichier d’informations d’identification de coffre.
| **Le serveur proxy/pare-feu bloque l’inscription** <br/>or <br/>**Aucune connectivité Internet** <br/><br/> Si votre machine ou serveur proxy a une connectivité à Internet limitée et que vous ne garantissez pas l’accès pour les URL nécessaires, l’inscription échoue.| Suivez ces étapes :<br/> <ul><li> Travaillez avec votre équipe informatique pour vous assurer que le système dispose d’une connectivité Internet.<li> Si vous n’avez pas de serveur proxy, vérifiez que l’option de proxy n’est pas sélectionnée lorsque vous inscrivez l’agent. [Vérifiez vos paramètres de proxy](#verifying-proxy-settings-for-windows).<li> Si vous avez un pare-feu/serveur proxy, travaillez avec votre équipe de mise en réseau pour vous assurer que ces URL et adresses IP sont accessibles :<br/> <br> **URLs**<br> `www.msftncsi.com` <br> .Microsoft.com <br> .WindowsAzure.com <br> .microsoftonline.com <br> .windows.net <br>**Adresses IP**<br>  20.190.128.0/18 <br>  40.126.0.0/18 <br/></ul></ul>Essayez de procéder une nouvelle fois à l’inscription après avoir effectué les étapes précédentes de résolution des problèmes.<br></br> Si vous êtes connecté via Azure ExpressRoute, vérifiez que les paramètres sont configurés comme indiqué dans [Support Azure ExpressRoute](backup-support-matrix-mars-agent.md#azure-expressroute-support).
| **Un logiciel antivirus bloque l’inscription** | Si vous avez un logiciel antivirus installé sur le serveur, ajoutez les règles d’exclusion nécessaires à l’analyse antivirus pour ces fichiers et dossiers : <br/><ul> <li> CBengine.exe <li> CSC.exe<li> Le dossier de travail. Son emplacement par défaut est C:\Program Files\Microsoft Azure Recovery Services Agent\Scratch. <li> Le dossier Bin à l’emplacement C:\Program Files\Microsoft Azure Recovery Services Agent\Bin.

### <a name="additional-recommendations"></a>Recommandations supplémentaires

- Accédez à C:/Windows/Temp et vérifiez s’il y a plus de 60 000 ou 65 000 fichiers comportant l’extension .tmp. Si c’est le cas, supprimez ces fichiers.
- Vérifiez que la date et l’heure de la machine correspondent au fuseau horaire local.
- Vérifiez que [ces sites](install-mars-agent.md#verify-internet-access) sont ajoutés à vos sites de confiance dans Internet Explorer.

### <a name="verifying-proxy-settings-for-windows"></a>Vérification des paramètres de proxy pour Windows

1. Téléchargez PsExec à partir de la page [Sysinternals](/sysinternals/downloads/psexec).
1. À partir d’une invite de commandes avec élévation de privilèges, exécutez `psexec -i -s "c:\Program Files\Internet Explorer\iexplore.exe"`.

   Cette commande ouvre Internet Explorer.
1. Accédez à **Outils** > **Options Internet** > **Connexions** > **Paramètres réseau**.
1. Vérifiez les paramètres de proxy pour le compte système.
1. Si aucun proxy n’est configuré et que les détails du proxy sont fournis, supprimez-les.
1. Si un proxy est configuré et que les détails du proxy sont incorrects, vérifiez que les informations **d’adresse IP de proxy** et de **port** sont correctes.
1. Fermez Internet Explorer.

## <a name="unable-to-download-vault-credential-file"></a>Impossible de télécharger le fichier d’informations d’identification de coffre

| Error   | Actions recommandées |
| ---     | ---    |
|Échec de téléchargement du fichier d’informations d’identification du coffre. (ID : 403) | <ul><li> Essayez de télécharger les informations d’identification de coffre à l’aide d’un autre navigateur, ou procédez comme suit : <ul><li> Démarrez Internet Explorer. Sélectionnez F12. </li><li> Accédez à l’onglet **Réseau** et effacez le cache et les cookies. </li> <li> Actualisez la page.<br></li></ul> <li> Vérifiez si l’abonnement est désactivé ou a expiré.<br></li> <li> Vérifiez si une règle de pare-feu bloque le téléchargement. <br></li> <li> Assurez-vous de ne pas avoir atteint la limite relative au coffre (50 machines par coffre).<br></li>  <li> Vérifiez que l’utilisateur a les autorisations Sauvegarde Azure nécessaires pour télécharger les informations d’identification du coffre et inscrire un serveur dans le coffre. Consultez [Utilisation du contrôle d’accès Azure en fonction du rôle pour gérer les points de récupération Sauvegarde Azure](backup-rbac-rs-vault.md).</li></ul> |

## <a name="the-microsoft-azure-recovery-service-agent-was-unable-to-connect-to-microsoft-azure-backup"></a>L’agent Microsoft Azure Recovery Service n’a pas pu se connecter à la Sauvegarde Microsoft Azure

| Error  | Cause probable | Actions recommandées |
| ---     | ---     | ---    |
| <br /><ul><li>L’agent MARS n’a pas pu se connecter à Sauvegarde Microsoft Azure. (ID : 100050) Vérifiez vos paramètres réseau et assurez-vous que vous pouvez vous connecter à Internet.<li>(407) Authentification proxy requise. |Un proxy bloque la connexion. |  <ul><li>Dans Internet Explorer, accédez à **Outils** > **Options Internet** > **Sécurité** > **Internet**. Sélectionnez **Personnaliser le niveau** et faites défiler l’écran jusqu’à la section **Télécharger le fichier**. Sélectionnez **Activer**.<p>En outre, vous devez peut-être ajouter des [URL et adresses IP](install-mars-agent.md#verify-internet-access) aux sites de confiance dans Internet Explorer.<li>Modifiez les paramètres pour utiliser un serveur proxy. Fournissez ensuite les détails du serveur proxy.<li> Si votre machine a un accès à Internet limité, assurez-vous que les paramètres du pare-feu sur la machine ou le proxy autorisent ces [URL et adresses IP](install-mars-agent.md#verify-internet-access). <li>Si un logiciel antivirus est installé sur le serveur, excluez ces fichiers de l’analyse antivirus : <ul><li>CBEngine.exe (au lieu de dpmra.exe).<li>CSC.exe (lié à .NET Framework). Il existe un fichier CSC.exe pour chaque version de .NET Framework installée sur le serveur. Excluez les fichiers CSC.exe de toutes les versions de .NET Framework sur le serveur concerné. <li>L’emplacement du dossier de travail ou du cache. <br>L’emplacement par défaut du dossier de travail ou le chemin d’accès à l’emplacement du cache est C:\Program Files\Microsoft Azure Recovery Services Agent\Scratch.<li>Le dossier Bin à l’emplacement C:\Program Files\Microsoft Azure Recovery Services Agent\Bin.

## <a name="the-specified-vault-credential-file-cannot-be-used-as-it-is-not-downloaded-from-the-vault-associated-with-this-server"></a>Impossible d’utiliser le fichier d’informations d’identification de coffre spécifié, car il n’a pas été téléchargé à partir du coffre associé à ce serveur.

| Error  | Cause probable | Actions recommandées |
| ---     | ---     | ---    |
| Impossible d’utiliser le fichier d’informations d’identification de coffre spécifié, car il n’a pas été téléchargé à partir du coffre associé à ce serveur. (ID : 100110) Fournissez les informations d’identification de coffre appropriées. | Le fichier d’informations d’identification de coffre provient d’un coffre différent de celui auprès duquel ce serveur est déjà inscrit. | Vérifiez que l’ordinateur cible et l’ordinateur source sont inscrits auprès du même coffre Recovery Services. Si le serveur cible a déjà été inscrit auprès d’un autre coffre, utilisez l’option **Inscription du serveur** pour vous l’inscrire auprès du coffre approprié.  

## <a name="backup-jobs-completed-with-warning"></a>Travaux de sauvegarde effectués avec des avertissements

- Quand l’agent MARS itère au sein de fichiers et de dossiers durant la sauvegarde, il peut rencontrer plusieurs conditions susceptibles d’indiquer que la sauvegarde s’est effectuée avec des avertissements. Dans ces conditions, un travail s’affiche comme s’étant effectué avec des avertissements. Parfait, mais cela signifie qu’au moins un fichier n’a pas pu être sauvegardé. Le travail a donc ignoré ce fichier, mais a sauvegardé tous les autres fichiers en question dans la source de données.

  ![Travail de sauvegarde effectué avec des avertissements](./media/backup-azure-mars-troubleshoot/backup-completed-with-warning.png)

- Les conditions susceptibles d’entraîner l’omission de fichiers par les sauvegardes sont les suivantes :
  - Attributs de fichier non pris en charge (exemples : contenu d’un dossier OneDrive, flux compressé, points d’analyse). Pour la liste complète, consultez la [matrice de prise en charge](./backup-support-matrix-mars-agent.md#supported-file-types-for-backup).
  - Problème lié au système de fichiers
  - Un autre processus interfère (par exemple, si un logiciel antivirus détient des handles sur des fichiers, cela peut empêcher l’agent MARS d’accéder à ces derniers)
  - Fichiers verrouillés par une application  

- Le service Sauvegarde marque ces fichiers en leur affectant un état d’échec dans le fichier journal, avec la convention de nommage suivante : *FichiersAvecÉtatÉchecDernièreSauvegardexxxx.txt* dans le dossier *C:\Program Files\Microsoft Azure Recovery Service Agent\temp*.
- Pour résoudre le problème, passez en revue le fichier journal afin de comprendre ce qui n’a pas fonctionné correctement :

  | Code d'erreur             | Raisons                                             | Recommandations                                              |
  | ---------------------- | --------------------------------------------------- | ------------------------------------------------------------ |
  | 0x80070570             | Le fichier ou le répertoire est endommagé et illisible. | Exécutez **chkdsk** sur le volume source.                             |
  | 0x80070002, 0x80070003 | Le système ne peut pas localiser le fichier spécifié.         | [Vérifiez si le dossier de travail n’est pas plein](./backup-azure-file-folder-backup-faq.md#manage-the-backup-cache-folder)  <br><br>  Vérifiez si le volume où l’espace de travail est configuré existe (c’est-à-dire qu’il n’a pas été supprimé)  <br><br>   [Vérifiez si l’agent MARS fait partie des exclusions de l’antivirus installé sur la machine](./backup-azure-troubleshoot-slow-backup-performance-issue.md#cause-another-process-or-antivirus-software-interfering-with-azure-backup)  |
  | 0x80070005             | Accès refusé                                    | [Vérifiez si un antivirus ou tout autre logiciel tiers ne bloque pas l’accès](./backup-azure-troubleshoot-slow-backup-performance-issue.md#cause-another-process-or-antivirus-software-interfering-with-azure-backup)     |
  | 0x8007018b             | L’accès au fichier cloud est refusé.                | Fichiers OneDrive, fichiers Git ou tout autre fichier pouvant être hors connexion sur la machine |

- Vous pouvez utiliser [Add Exclusion rules to existing policy](./backup-azure-manage-mars.md#add-exclusion-rules-to-existing-policy) (Ajouter des règles d’exclusion à la stratégie existante) pour exclure les fichiers non pris en charge, manquants ou supprimés de votre stratégie de sauvegarde afin de garantir la réussite des sauvegardes.

- Évitez de supprimer et de recréer des dossiers protégés portant les mêmes noms dans le dossier de premier niveau. Cela peut entraîner l’exécution de la sauvegarde avec des avertissements et avec l’erreur suivante : *Une incohérence critique a été détectée. Par conséquent, les modifications ne peuvent pas être répliquées.*  Si vous devez supprimer et recréer des dossiers, faites-le dans les sous-dossiers du dossier de premier niveau protégé.

## <a name="failed-to-set-the-encryption-key-for-secure-backups"></a>Échec de définition de la clé de chiffrement pour les sauvegardes sécurisées

| Error | Causes possibles | Actions recommandées |
| ---     | ---     | ---    |
| <br />Échec de définition de la clé de chiffrement pour les sauvegardes sécurisées. L’activation n’a pas entièrement abouti, mais la phrase secrète de chiffrement a été enregistrée dans le fichier suivant. |<li>Le serveur est déjà inscrit auprès d’un autre coffre.<li>Lors de la configuration, la phrase secrète a été endommagée.| Désinscrivez le serveur du coffre et réinscrivez-le avec une nouvelle phrase secrète.

## <a name="the-activation-did-not-complete-successfully"></a>L’activation n’a pas abouti

| Error  | Causes possibles | Actions recommandées |
|---------|---------|---------|
|<br />L’activation n’a pas réussi. Échec de l’opération en cours en raison d’une erreur de service interne [0x1FC07]. Réessayez l’opération après un certain temps. Si le problème persiste, contactez le support technique Microsoft.     | <li> Le dossier de travail se situe sur un volume dont l’espace est insuffisant. <li> Le dossier de travail a été déplacé de façon incorrecte. <li> Le fichier OnlineBackup.KEK est manquant.         | <li>Effectuez une mise à niveau vers la [dernière version](https://aka.ms/azurebackup_agent) de l’agent MARS.<li>Déplacez l’emplacement du dossier de travail ou du cache vers un volume avec un espace disponible compris entre 5 et 10 % de la taille totale des données de sauvegarde. Pour déplacer correctement l’emplacement du cache, reportez-vous aux étapes contenues dans [Questions courantes sur la sauvegarde de fichiers et de dossiers](./backup-azure-file-folder-backup-faq.md#manage-the-backup-cache-folder).<li> Assurez-vous que le fichier OnlineBackup.KEK est présent. <br>*L’emplacement par défaut du dossier de travail ou le chemin d’accès à l’emplacement du cache est C:\Program Files\Microsoft Azure Recovery Services Agent\Scratch*.        |

## <a name="encryption-passphrase-not-correctly-configured"></a>La phrase secrète de chiffrement n’est pas correctement configurée

| Error  | Causes possibles | Actions recommandées |
|---------|---------|---------|
| <br />Erreur 34506. La phrase secrète de chiffrement stockée sur cet ordinateur n’est pas configurée correctement.    | <li> Le dossier de travail se situe sur un volume dont l’espace est insuffisant. <li> Le dossier de travail a été déplacé de façon incorrecte. <li> Le fichier OnlineBackup.KEK est manquant.        | <li>Effectuez une mise à niveau vers la [dernière version](https://aka.ms/azurebackup_agent) de l’Agent MARS.<li>Déplacez l’emplacement du dossier de travail ou du cache vers un volume avec un espace disponible compris entre 5 et 10 % de la taille totale des données de sauvegarde. Pour déplacer correctement l’emplacement du cache, reportez-vous aux étapes contenues dans [Questions courantes sur la sauvegarde de fichiers et de dossiers](./backup-azure-file-folder-backup-faq.md#manage-the-backup-cache-folder).<li> Assurez-vous que le fichier OnlineBackup.KEK est présent. <br>*L’emplacement par défaut du dossier de travail ou le chemin d’accès à l’emplacement du cache est C:\Program Files\Microsoft Azure Recovery Services Agent\Scratch*.         |

## <a name="backups-dont-run-according-to-schedule"></a>Les sauvegardes ne s’exécutent pas comme prévu

Si les sauvegardes planifiées ne se déclenchent pas automatiquement, alors que les sauvegardes manuelles fonctionnent correctement, essayez ceci :

- Vérifiez que la planification de la sauvegarde de Windows Server n’est pas en conflit avec la planification de la sauvegarde des fichiers et dossiers Azure.

- Vérifiez que l’état de la sauvegarde en ligne est défini sur **Activer**. Pour vérifier l’état, procédez comme suit :

  1. Dans le Planificateur de tâches, développez **Microsoft** et sélectionnez **Sauvegarde en ligne**.
  1. Double-cliquez sur **Microsoft-OnlineBackup** et accédez à l’onglet **Déclencheurs**.
  1. Assurez-vous que l’état est défini sur **Activé**. Si ce n’est pas le cas, sélectionnez **Modifier**, **Activé**, puis **OK**.

- Vérifiez que le compte d’utilisateur sélectionné pour l’exécution de la tâche est **SYSTÈME** ou le **groupe Administrateurs locaux** du serveur. Pour vérifier le compte d’utilisateur, accédez à l’onglet **Général** et vérifiez les options **Sécurité**.

- Vérifiez que PowerShell 3.0 ou version ultérieure est installé sur le serveur. Pour vérifier la version de PowerShell, exécutez cette commande et vérifiez que le numéro de version `Major` est 3 ou plus :

  `$PSVersionTable.PSVersion`

- Vérifiez que ce chemin d’accès fait partie de la variable d’environnement `PSMODULEPATH` :

  `<MARS agent installation path>\Microsoft Azure Recovery Services Agent\bin\Modules\MSOnlineBackup`

- Si la stratégie d’exécution de PowerShell pour `LocalMachine` est définie sur `restricted`, la cmdlet PowerShell qui déclenche la tâche de sauvegarde risque d’échouer. Exécutez ces commandes en mode avec élévation de privilèges pour vérifier la stratégie d’exécution et la définir sur `Unrestricted` ou `RemoteSigned` :

 ```PowerShell
 Get-ExecutionPolicy -List

Set-ExecutionPolicy Unrestricted
 ```

- Vérifiez qu’il n’y a aucun fichier MSOnlineBackup de module PowerShell endommagé ou manquant. En cas de fichiers manquants ou endommagés, procédez comme suit :

  1. Sur n’importe quelle machine disposant d’un agent MARS opérationnel, copiez le dossier MSOnlineBackup depuis C:\Program Files\Microsoft Azure Recovery Services Agent\bin\Modules.
  1. Sur la machine qui pose problème, collez les fichiers copiés au même emplacement de dossier (C:\Program Files\Microsoft Azure Recovery Services Agent\bin\Modules).

     S’il existe déjà un dossier MSOnlineBackup sur la machine, collez les fichiers dessus ou remplacez les fichiers existants.

> [!TIP]
> Pour veiller à ce que les modifications soient appliquées de manière cohérente, redémarrez le serveur après avoir suivi les étapes ci-dessus.

## <a name="resource-not-provisioned-in-service-stamp"></a>Ressource non approvisionnée dans l'horodatage du service

Error | Causes possibles | Actions recommandées
--- | --- | ---
L'opération actuelle a échoué en raison d'une erreur de service interne « Ressource non approvisionnée dans l'horodatage du service ». Veuillez réessayer l’opération après un certain temps. (ID : 230006) | Le serveur protégé a été renommé. | <li> Renommez le serveur avec le nom d'origine inscrit auprès du coffre. <br> <li> Réinscrivez le serveur dans le coffre avec le nouveau nom.

## <a name="job-could-not-be-started-as-another-job-was-in-progress"></a>Impossible de démarrer le travail car un autre travail est en cours

Si vous voyez dans la **console MARS** > **historique des travaux**, le message d’avertissement « Impossible de démarrer le travail car un autre travail est en cours », cela peut être dû à une instance en double du travail déclenchée par le Planificateur de tâches.

![Impossible de démarrer le travail car un autre travail est en cours](./media/backup-azure-mars-troubleshoot/job-could-not-be-started.png)

Pour résoudre ce problème :

1. Lancez le composant logiciel enfichable Planificateur de tâches en tapant *taskschd. msc* dans la fenêtre Exécuter
1. Dans le volet gauche, accédez à **Bibliothèque du Planificateur de tâches** -> **Microsoft** -> **OnlineBackup**.
1. Pour chaque tâche figurant dans cette bibliothèque, double-cliquez dessus pour ouvrir ses propriétés, puis procédez comme suit :
    1. Basculez vers l’onglet **Paramètres** .

         ![Onglet Paramètres](./media/backup-azure-mars-troubleshoot/settings-tab.png)

    1. Modifiez le paramétrage de l’option **Si la tâche s’exécute déjà, la règle suivante s’applique**. Choisissez **Ne pas démarrer une nouvelle instance**.

         ![Modifiez la règle pour ne pas démarrer une nouvelle instance](./media/backup-azure-mars-troubleshoot/change-rule.png)

## <a name="troubleshoot-restore-problems"></a>Résoudre les problèmes de restauration

Sauvegarde Azure peut ne pas correctement monter le volume de récupération, même au bout de plusieurs minutes. Vous pouvez également recevoir des messages d’erreur pendant le processus. Pour entamer normalement la récupération, procédez comme suit :

1. Si le processus de montage s’exécute depuis plusieurs minutes, annulez-le.

2. Vérifiez que vous disposez de la dernière version de l’agent Sauvegarde. Pour vérifier la version, dans le volet **Actions** de la console MARS, sélectionnez **À propos de l’agent Microsoft Azure Recovery Services**. Vérifiez que le numéro de la **version** est supérieur ou égal à la celui mentionné dans [cet article](https://go.microsoft.com/fwlink/?linkid=229525). Sélectionnez ce lien pour [télécharger la dernière version](https://go.microsoft.com/fwLink/?LinkID=288905).

3. Cliquez sur **Gestionnaire de périphériques** > **Contrôleurs de stockage** et localisez le service **Initiateur Microsoft iSCSI**. Si vous le localisez, passez directement à l’étape 7.

4. Si vous ne parvenez pas à localiser le service Initiateur Microsoft iSCSI, essayez de trouver sous **Gestionnaire de périphériques** > **Contrôleurs de stockage** l’entrée **Appareil inconnu** associée à l’ID matériel **ROOT\ISCSIPRT**.

5. Cliquez avec le bouton droit sur l’entrée **Appareil inconnu** et sélectionnez **Mettre à jour le pilote**.

6. Mettez à jour le pilote, en sélectionnant l’option **Rechercher automatiquement un pilote logiciel mis à jour**. Cette mise à jour doit remplacer l’entrée **Appareil inconnu** par **Initiateur Microsoft iSCSI** :

    ![Capture d’écran du Gestionnaire de périphériques de Sauvegarde Azure, avec Contrôleurs de stockage en surbrillance](./media/backup-azure-restore-windows-server/UnknowniSCSIDevice.png)

7. Cliquez sur **Gestionnaire des tâches** > **Services (local)**  > **Service Initiateur iSCSI de Microsoft** :

    ![Capture d’écran du Gestionnaire des tâches de Sauvegarde Azure, avec Services (local) en surbrillance](./media/backup-azure-restore-windows-server/MicrosoftInitiatorServiceRunning.png)

8. Redémarrez le service Initiateur Microsoft iSCSI. Pour ce faire, cliquez avec le bouton droit sur le service, puis sélectionnez **Arrêter**. Cliquez de nouveau avec le bouton droit et sélectionnez **Démarrer**.

9. Retentez la récupération à l’aide de la [Restauration instantanée](backup-instant-restore-capability.md).

Si celle-ci échoue encore, redémarrez votre serveur ou client. Si vous ne souhaitez pas redémarrer, ou si la récupération échoue encore après le redémarrage du serveur, essayez de [l’exécuter depuis une autre machine](backup-azure-restore-windows-server.md#use-instant-restore-to-restore-data-to-an-alternate-machine).

## <a name="troubleshoot-cache-problems"></a>Résoudre les problèmes de cache

L’opération de sauvegarde risque d’échouer si le dossier du cache (également nommé « dossier de travail ») n’est pas configuré correctement, ne respecte pas les prérequis ou présente un accès restreint.

### <a name="prerequisites"></a>Prérequis

Pour que les opérations de l’agent MARS fonctionnent correctement, le dossier du cache doit respecter les exigences suivantes :

- [Vérifiez qu’il existe entre 5 et 10 % d’espace de volume disponible à l’emplacement du dossier temporaire](backup-azure-file-folder-backup-faq.md#whats-the-minimum-size-requirement-for-the-cache-folder)
- [Vérifiez que l’emplacement du dossier temporaire est valide et accessible](backup-azure-file-folder-backup-faq.md#how-to-check-if-scratch-folder-is-valid-and-accessible)
- [Vérifiez que les attributs de fichier sur le dossier du cache sont pris en charge](backup-azure-file-folder-backup-faq.md#are-there-any-attributes-of-the-cache-folder-that-arent-supported)
- [Vérifiez que l’espace de stockage de cliché instantané alloué est suffisant pour le processus de sauvegarde](#increase-shadow-copy-storage)
- [Vérifiez qu’aucun autre processus (par exemple, un logiciel antivirus) ne restreint l’accès au dossier du cache](#another-process-or-antivirus-software-blocking-access-to-cache-folder)

### <a name="increase-shadow-copy-storage"></a>Augmenter l’espace de stockage de cliché instantané

Les opérations de sauvegarde peuvent échouer si l’espace de stockage de cliché instantané est insuffisant pour protéger la source de données. Pour résoudre ce problème, augmentez l’espace de stockage de cliché instantanés sur le volume protégé à l’aide de **vssadmin** comme illustré ci-dessous :

- Vérifiez l’espace de stockage de cliché instantané actuel à partir de l’invite de commandes avec élévation de privilèges :<br/>
  `vssadmin List ShadowStorage /For=[Volume letter]:`
- Augmentez l’espace de stockage de la mise en mémoire fantôme en utilisant la commande suivante :<br/>
  `vssadmin Resize ShadowStorage /On=[Volume letter]: /For=[Volume letter]: /Maxsize=[size]`

### <a name="another-process-or-antivirus-software-blocking-access-to-cache-folder"></a>Un autre processus ou logiciel antivirus bloque l’accès au dossier du cache

Si vous avez un logiciel antivirus installé sur le serveur, ajoutez les règles d’exclusion nécessaires à l’analyse antivirus pour ces fichiers et dossiers :  

- Le dossier de travail. Son emplacement par défaut est `C:\Program Files\Microsoft Azure Recovery Services Agent\Scratch`
- Dossier bin à l’emplacement `C:\Program Files\Microsoft Azure Recovery Services Agent\Bin`
- CBengine.exe
- CSC.exe

## <a name="common-issues"></a>Problèmes courants

Cette section décrit les erreurs courantes que vous rencontrez lors de l’utilisation de l’agent MARS.

### <a name="salchecksumstoreinitializationfailed"></a>SalChecksumStoreInitializationFailed

Message d’erreur | Action recommandée
--|--
L’agent Microsoft Azure Recovery Services n’a pas pu accéder à la somme de contrôle de sauvegarde stockée sur l’emplacement temporaire | Pour résoudre ce problème, procédez comme suit, puis redémarrez le serveur <br/> - [Vérifiez si un antivirus ou tout autre type de processus verrouille les fichiers sur l’emplacement temporaire](#another-process-or-antivirus-software-blocking-access-to-cache-folder)<br/> - [Vérifiez si l’agent MARS peut accéder à l’emplacement temporaire, et si ce dernier est valide.](backup-azure-file-folder-backup-faq.md#how-to-check-if-scratch-folder-is-valid-and-accessible)

### <a name="salvhdinitializationerror"></a>SalVhdInitializationError

Message d’erreur | Action recommandée
--|--
L’agent Microsoft Azure Recovery Services n’a pas pu accéder à l’emplacement temporaire pour initialiser le disque dur virtuel (VHD) | Pour résoudre ce problème, procédez comme suit, puis redémarrez le serveur <br/> - [Vérifiez si un antivirus ou un autre processus verrouille les fichiers dans l’emplacement temporaire](#another-process-or-antivirus-software-blocking-access-to-cache-folder)<br/> - [Vérifiez si l’agent MARS peut accéder à l’emplacement temporaire, et si ce dernier est valide.](backup-azure-file-folder-backup-faq.md#how-to-check-if-scratch-folder-is-valid-and-accessible)

### <a name="sallowdiskspace"></a>SalLowDiskSpace

Message d’erreur | Action recommandée
--|--
La sauvegarde a échoué en raison d’un espace de stockage insuffisant dans le volume où se trouve le dossier temporaire | Pour résoudre ce problème, procédez comme suit, puis retentez l’opération :<br/>- [Vérifiez que la version de l’agent MARS est à jour](https://go.microsoft.com/fwlink/?linkid=229525&clcid=0x409)<br/> - [Vérifiez et résolvez les problèmes de stockage impactant l’espace temporaire de la sauvegarde](#prerequisites)

### <a name="salbitmaperror"></a>SalBitmapError

Message d’erreur | Action recommandée
--|--
Impossible de localiser les changements apportés à un fichier. Cela peut être dû à diverses raisons. Réessayez l’opération | Pour résoudre ce problème, procédez comme suit, puis retentez l’opération :<br/> - [Vérifiez que la version de l’agent MARS est à jour](https://go.microsoft.com/fwlink/?linkid=229525&clcid=0x409) <br/> - [Vérifiez et résolvez les problèmes de stockage impactant l’espace temporaire de la sauvegarde](#prerequisites)

## <a name="next-steps"></a>Étapes suivantes

- Obtenez plus de détails sur [comment sauvegarder votre serveur Windows avec l’agent Sauvegarde Azure](tutorial-backup-windows-server-to-azure.md).
- Si vous avez besoin de restaurer une sauvegarde, consultez l’article dédié à la [restauration de fichiers sur une machine Windows](backup-azure-restore-windows-server.md).
