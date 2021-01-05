---
title: Résoudre les problèmes touchant le serveur de configuration durant la récupération d’urgence de serveurs physiques et machines virtuelles VMware sur Azure avec Azure Site Recovery | Microsoft Docs
description: Cet article fournit des informations sur la résolution de problèmes de déploiement d’un serveur de configuration pour la récupération d’urgence de serveurs physiques et machines virtuelles VMware sur Azure avec Azure Site Recovery.
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 02/13/2019
ms.author: ramamill
ms.openlocfilehash: b5fd014732fd4cdfaa52f971b5e4d2c74db580d2
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92371951"
---
# <a name="troubleshoot-configuration-server-issues"></a>Résoudre les problèmes de serveur de configuration

Cet article vous aide à résoudre des problèmes qui se produisent lors du déploiement et de la gestion du serveur de configuration [Azure Site Recovery](site-recovery-overview.md). Le serveur de configuration agit comme un serveur de gestion. Utilisez le serveur de configuration pour configurer la récupération d’urgence de serveurs physiques et de machines virtuelles VMware sur Azure avec Azure Site Recovery. Les sections suivantes décrivent les défaillances les plus courantes que vous pourriez rencontrer lorsque vous ajoutez et gérez un serveur de configuration.

## <a name="registration-failures"></a>Échecs d’enregistrement

La machine source s’inscrit auprès du serveur de configuration lorsque vous installez l’agent de mobilité. Vous pouvez déboguer toute défaillance au cours de cette étape en suivant les instructions suivantes :

1. Ouvrez le fichier C:\ProgramData\ASR\home\svsystems\var\configurator_register_host_static_info.log. (Le dossier ProgramData peut être masqué. Si vous ne le voyez pas, dans l’Explorateur de fichiers, sous l’onglet **Affichage** , dans la section **Afficher/Masquer** , activez la case à cocher **Éléments masqués** .) Des défaillances peuvent être causées par plusieurs problèmes.

2. Recherchez la chaîne **Aucune adresse IP valide trouvée** . Si la chaîne est trouvée :
   1. Vérifiez que l’ID hôte demandé est identique à celui de la machine source.
   2. Vérifiez que la machine source dispose d’au moins une adresse IP affectée à la carte réseau physique. Pour que l’inscription de l’agent auprès du serveur de configuration réussisse, la machine source doit avoir au moins une adresse IP v4 valide affectée à la carte physique.
   3. Exécutez l’une des commandes suivantes sur la machine source pour obtenir toutes les adresses IP de celle-ci :
      - Pour Windows : `> ipconfig /all`
      - Pour Linux : `# ifconfig -a`

3. Si la chaîne **Aucune adresse IP valide trouvée** n’est pas trouvée, recherchez la chaîne **Reason=>NULL** . Cette erreur se produit si la machine source utilise un hôte vide pour s’inscrire auprès du serveur de configuration. Si la chaîne est trouvée :
    - Après avoir résolu les problèmes, suivez les instructions de la section [Inscrire la machine source auprès du serveur de configuration](vmware-azure-troubleshoot-configuration-server.md#register-source-machine-with-configuration-server) pour réessayer d’effectuer l’inscription manuellement.

4. Si la chaîne **Reason= >NULL** n’est pas trouvée, sur la machine source, ouvrez le fichier C:\ProgramData\ASRSetupLogs\UploadedLogs\ASRUnifiedAgentInstaller.log. (Le dossier ProgramData peut être masqué. Si vous ne le voyez pas, dans l’Explorateur de fichiers, sous l’onglet **Affichage** , dans la section **Afficher/Masquer** , activez la case à cocher **Éléments masqués** .) Des défaillances peuvent être causées par plusieurs problèmes. 

5. Recherchez la chaîne **post request: (7) - Couldn't connect to server** . Si la chaîne est trouvée :
    1. Résolvez les problèmes de réseau entre la machine source et le serveur de configuration. Vérifiez que le serveur de configuration est accessible à partir de la machine source à l’aide d’outils réseau tels que ping et Détermination d’itinéraire, ou d’un navigateur web. Assurez-vous que la machine source peut atteindre le serveur de configuration via le port 443.
    2. Vérifiez si des règles de pare-feu sur la machine source bloquent la connexion entre celle-ci et le serveur de configuration. Collaborez avec vos administrateurs réseau pour débloquer d’éventuels problèmes de connexion.
    3. Vérifiez que les dossiers répertoriés dans [Site Recovery folder exclusions from antivirus programs](vmware-azure-set-up-source.md#azure-site-recovery-folder-exclusions-from-antivirus-program) (Dossiers de Site Recovery exclus des programmes antivirus) sont exclus du logiciel antivirus.
    4. Une fois les problèmes de réseau résolus, recommencez l’inscription en suivant les instructions de la section [Inscrire la machine source auprès du serveur de configuration](vmware-azure-troubleshoot-configuration-server.md#register-source-machine-with-configuration-server).

6. Si la chaîne **post request: (7) - Couldn't connect to server** est introuvable, dans le même fichier journal, recherchez la chaîne **request: (60) - Peer certificate cannot be authenticated with given CA certificates** . Cette erreur peut se produire si le certificat du serveur de configuration a expiré ou si la machine source ne prend pas en charge TLS 1.0 ou des protocoles ultérieurs. Elle peut également se produire si un pare-feu bloque les communications TLS entre la machine source et le serveur de configuration. Si la chaîne est trouvée : 
    1. Pour résoudre le problème, connectez-vous à l’adresse IP du serveur de configuration via un navigateur web sur la machine source. Utilisez l’URI https :\/\/<adresse IP du serveur configuration\>: 443/. Assurez-vous que la machine source peut atteindre le serveur de configuration via le port 443.
    2. Vérifier si des règles de pare-feu sur la machine source doivent être ajoutées ou supprimées pour que celle-ci puisse communiquer avec le serveur de configuration. Compte tenu de la diversité des logiciels de pare-feu susceptibles d’être utilisés, nous ne pouvons pas répertorier toutes les configurations de pare-feu requises. Collaborez avec vos administrateurs réseau pour débloquer d’éventuels problèmes de connexion.
    3. Vérifiez que les dossiers répertoriés dans [Site Recovery folder exclusions from antivirus programs](vmware-azure-set-up-source.md#azure-site-recovery-folder-exclusions-from-antivirus-program) (Dossiers de Site Recovery exclus des programmes antivirus) sont exclus du logiciel antivirus.  
    4. Après avoir résolu les problèmes, réessayez d’effectuer l’inscription en suivant les instructions de la section [Inscrire la machine source auprès du serveur de configuration](vmware-azure-troubleshoot-configuration-server.md#register-source-machine-with-configuration-server).

7. Sous Linux, si la valeur de la plateforme dans <INSTALLATION_DIR\>/etc/drscout.conf est endommagée, l’inscription échoue. Pour identifier ce problème, ouvrez le fichier /var/log/ua_install.log. Recherchez la chaîne **Abandon de la configuration car la valeur de VM_PLATFORM est null ou n’est pas VmWare/Azure** . La plateforme définie doit être **VmWare** ou **Azure** . Si le fichier drscout.conf est endommagé, il est recommandé de [désinstaller l’agent de mobilité](vmware-physical-manage-mobility-service.md#uninstall-mobility-service), puis de le réinstaller. Si la désinstallation échoue, effectuez les étapes suivantes : a. Ouvrez le fichier Installation_Directory/uninstall.sh et commentez l’appel de la fonction **StopServices** .
    b. Ouvrez le fichier de Installation_Directory/Vx/bin/uninstall.sh et commentez l’appel de la fonction **stop_services** .
    c. Ouvrez le fichier Installation_Directory/Fx/uninstall et commentez toute la section qui tente d’arrêter le service Fx.
    d. [Désinstallez](vmware-physical-manage-mobility-service.md#uninstall-mobility-service) l’agent de mobilité. Après la désinstallation, redémarrez le système, puis tentez de réinstaller l’agent de mobilité.

8. Assurez-vous que l’authentification multifacteur n’est pas activée pour le compte d’utilisateur. À ce jour, Azure Site Recovery ne prend pas en charge l’authentification multifacteur pour le compte d’utilisateur. Inscrivez le serveur de configuration sans compte d’utilisateur activé pour l’authentification multifacteur.  

## <a name="installation-failure-failed-to-load-accounts"></a>Échec de l’installation : Échec du chargement des comptes

Cette erreur se produit quand le service ne parvient pas à lire les données de la connexion de transport au moment de l’installation de l’agent de mobilité et de l’inscription auprès du serveur de configuration. Pour résoudre ce problème, assurez-vous que TLS 1.0 est activé sur votre machine source.

## <a name="vcenter-discovery-failures"></a>Échec de découverte de vCenter

Pour résoudre les échecs de découverte de vCenter, ajoutez le serveur vCenter aux paramètres de proxy de la liste de contournement. 

- Téléchargez l’outil PsExec [ici](/sysinternals/downloads/psexec) pour accéder au contenu de l’utilisateur système.
- Ouvrez Internet Explorer dans le contenu de l’utilisateur système en exécutant la ligne de commande suivante : psexec -s -i "%programfiles%\Internet Explorer\iexplore.exe"
- Ajoutez les paramètres du proxy dans Internet Explorer et redémarrez le service tmanssvc.
- Pour configurer les paramètres du proxy DRA, exécutez cd C:\Program Files\Microsoft Azure Site Recovery Provider.
- Ensuite, exécutez DRCONFIGURATOR.EXE /configure /AddBypassUrls [ajouter l’adresse IP ou le FQDN du serveur vCenter fourni durant l’étape **Configurer le serveur vCenter/vSphere ESXi** du [déploiement du serveur de configuration](vmware-azure-deploy-configuration-server.md#configure-settings)]

## <a name="change-the-ip-address-of-the-configuration-server"></a>Modifier l’adresse IP du serveur de configuration

Il est fortement recommandé de ne pas modifier l’adresse IP d’un serveur de configuration. Assurez-vous que toutes les adresses IP affectées au serveur de configuration sont statiques. N’utilisez pas d’adresses IP DHCP.

## <a name="acs50008-saml-token-is-invalid"></a>ACS50008 : Le jeton SAML n'est pas valide

Pour éviter cette erreur, vérifiez que l’heure de votre système ne diffère pas de plus de 15 minutes de l’heure locale. Réexécutez le programme d’installation pour terminer l’inscription.

## <a name="failed-to-create-a-certificate"></a>Échec de création d’un certificat

Un certificat requis pour l’authentification de Site Recovery ne peut pas être créé. Recommencez la configuration après vous être assuré que vous opérez en tant qu’administrateur local.

## <a name="failure-to-activate-windows-license-from-server-standard-evaluation-to-server-standard"></a>Échec de l’activation de la Licence Windows de Server Standard Evaluation vers Server Standard

1. Dans le cadre du déploiement du serveur de configuration par la biais du modèle OVF, une licence d’évaluation qui est valide pendant 180 jours est utilisée. Vous devez activer cette licence avant l’expiration de ce délai. Sinon, cela peut entraîner un arrêt fréquent du serveur de configuration, et donc constituer une entrave aux activités de réplication.
2. Si vous ne parvenez pas à activer la licence Windows, contactez l’[équipe de support technique Windows](https://aka.ms/Windows_Support) pour résoudre le problème.

## <a name="register-source-machine-with-configuration-server"></a>Inscrire l'ordinateur source auprès du serveur de configuration

### <a name="if-the-source-machine-runs-windows"></a>Si la machine source exécute Windows

Exécutez la commande suivante sur la machine source :

```
  cd C:\Program Files (x86)\Microsoft Azure Site Recovery\agent
  UnifiedAgentConfigurator.exe  /CSEndPoint <configuration server IP address> /PassphraseFilePath <passphrase file path>
```

Paramètre | Détails
--- | ---
Usage | UnifiedAgentConfigurator.exe /CSEndPoint <adresse IP du serveur configuration\>/PassphraseFilePath <chemin d’accès du fichier de phrase secrète\>
Journaux d’activité de configuration de l’agent | Situés sous %ProgramData%\ASRSetupLogs\ASRUnifiedAgentConfigurator.log.
/CSEndPoint | Paramètre obligatoire. Spécifie l’adresse IP du serveur de configuration. Utilisez une adresse IP valide.
/PassphraseFilePath |  Mandatory. Emplacement de la phrase secrète. Utilisez n’importe quel chemin d’accès UNC ou local valide.

### <a name="if-the-source-machine-runs-linux"></a>Si la machine source exécute Linux

Exécutez la commande suivante sur la machine source :

```
  /usr/local/ASR/Vx/bin/UnifiedAgentConfigurator.sh -i <configuration server IP address> -P /var/passphrase.txt
  ```

Paramètre | Détails
--- | ---
Usage | cd /usr/local/ASR/Vx/bin<br /><br /> UnifiedAgentConfigurator.sh -i <adresse IP du serveur configuration\> - P <chemin d’accès du fichier de phrase secrète\>
-i | Paramètre obligatoire. Spécifie l’adresse IP du serveur de configuration. Utilisez une adresse IP valide.
-P |  Mandatory. Chemin d’accès complet du fichier dans lequel la phrase secrète est enregistrée. Utilisez n’importe quel dossier valide.

## <a name="unable-to-configure-the-configuration-server"></a>Impossible de configurer le serveur de configuration

Si vous installez des applications autres que le serveur de configuration sur la machine virtuelle, vous risquez de ne pas pouvoir configurer le serveur cible maître. 

Le serveur de configuration doit être un serveur à usage unique et il n’est pas possible de l’utiliser comme serveur partagé. 

Pour plus d’informations, consultez les questions fréquentes (FAQ) relatives à la configuration dans [Déployer un serveur de configuration](vmware-azure-deploy-configuration-server.md#faqs). 

## <a name="remove-the-stale-entries-for-protected-items-from-the-configuration-server-database"></a>Supprimer les entrées obsolètes des éléments protégés de la base de données du serveur de configuration 

Pour supprimer une machine protégée obsolète sur le serveur de configuration, utilisez les étapes suivantes. 
 
1. Pour déterminer la machine source et l’adresse IP de l’entrée obsolète : 

    1. Ouvrez la ligne de commande MYSQL en mode administrateur. 
    2. Exécutez les commandes suivantes. 
   
        ```
        mysql> use svsdb1;
        mysql> select id as hostid, name, ipaddress, ostype as operatingsystem, from_unixtime(lasthostupdatetime) as heartbeat from hosts where name!='InMageProfiler'\G;
        ```

        Cette opération retourne la liste des machines inscrites, ainsi que leurs adresses IP et la dernière pulsation. Recherchez l’hôte qui comporte les paires de réplication obsolètes.

2. Ouvrez une invite de commandes avec élévation de privilèges, puis accédez à C:\ProgramData\ASR\home\svsystems\bin. 
4. Pour supprimer les détails des hôtes inscrits et les informations des entrée obsolète du serveur de configuration, exécutez la commande suivante à l’aide de la machine source et l’adresse IP de l’entrée obsolète. 
   
    `Syntax: Unregister-ASRComponent.pl -IPAddress <IP_ADDRESS_OF_MACHINE_TO_UNREGISTER> -Component <Source/ PS / MT>`
 
    Si vous avez une entrée de serveur source « OnPrem-VM01 » avec l’adresse IP 10.0.0.4, utilisez la commande suivante à la place.
 
    `perl Unregister-ASRComponent.pl -IPAddress 10.0.0.4 -Component Source`
 
5. Redémarrez les services suivants sur la machine source pour réinscrire le serveur de configuration. 
 
    - Service d’application InMage Scout
    - InMage Scout VX Agent - Sentinel/Outpost

## <a name="upgrade-fails-when-the-services-fail-to-stop"></a>Échec de la mise à niveau quand l’arrêt des services échoue

La mise à niveau du serveur de configuration échoue quand certains services ne s’arrêtent pas. 

Pour identifier le problème, accédez à C:\ProgramData\ASRSetupLogs\CX_TP_InstallLogFile sur le serveur de configuration. Si vous rencontrez les erreurs suivantes, utilisez les étapes ci-dessous pour résoudre le problème : 

```output
2018-06-28 14:28:12.943   Successfully copied php.ini to C:\Temp from C:\thirdparty\php5nts
2018-06-28 14:28:12.943   svagents service status - SERVICE_RUNNING
2018-06-28 14:28:12.944   Stopping svagents service.
2018-06-28 14:31:32.949   Unable to stop svagents service.
2018-06-28 14:31:32.949   Stopping svagents service.
2018-06-28 14:34:52.960   Unable to stop svagents service.
2018-06-28 14:34:52.960   Stopping svagents service.
2018-06-28 14:38:12.971   Unable to stop svagents service.
2018-06-28 14:38:12.971   Rolling back the install changes.
2018-06-28 14:38:12.971   Upgrade has failed.
```

Pour résoudre le problème :

Arrêtez manuellement les services suivants :

- cxprocessserver
- InMage Scout VX Agent - Sentinel/Outpost, 
- Agent Microsoft Azure Recovery Services, 
- Service Microsoft Azure Site Recovery, 
- tmansvc
  
Pour mettre à jour le serveur de configuration, réexécutez l’[installation unifiée](service-updates-how-to.md#links-to-currently-supported-update-rollups).

## <a name="azure-active-directory-application-creation-failure"></a>Échec de la création d’application Azure Active Directory

Vos autorisations sont insuffisantes pour créer une application dans Azure Active Directory (AAD) à l’aide du modèle [OVA (Open Virtualization Application)](vmware-azure-deploy-configuration-server.md#deploy-a-configuration-server-through-an-ova-template
).

Pour résoudre ce problème, connectez-vous au portail Azure, puis effectuez l’une des opérations suivantes :

- Demandez le rôle Développeur d’applications dans AAD. Pour plus d’informations sur le rôle Développeur d’applications, consultez [Autorisations des rôles d’administrateur dans Azure Active Directory](../active-directory/roles/permissions-reference.md).
- Vérifiez que l’indicateur **L’utilisateur peut créer une application** a la valeur *true* dans AAD. Pour plus d’informations, consultez [Procédure : Utilisez le portail pour créer une application Azure AD et un principal du service pouvant accéder aux ressources](../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).

## <a name="process-servermaster-target-are-unable-to-communicate-with-the-configuration-server"></a>Le serveur de processus/le serveur cible maître ne sont pas en mesure de communiquer avec le serveur de configuration 

Le serveur de processus (PS) et le serveur cible maître (MT) ne peuvent pas communiquer avec le serveur de configuration (CS) et leur état indique qu’ils ne sont pas connectés au portail Azure.

En général, cela est dû à une erreur avec le port 443. Utilisez les étapes suivantes pour débloquer le port et réactiver la communication avec le serveur CS.

**Vérifier que l’agent MARS est appelé par l’agent du serveur cible maître**

Pour vérifier que l’agent du serveur cible maître peut créer une session TCP pour l’adresse IP du serveur de configuration, recherchez une trace semblable à ce qui suit dans les journaux d’activité de l’agent du serveur cible maître :

TCP \<Replace IP with CS IP here>:52739 \<Replace IP with CS IP here>:443 SYN_SENT 

TCP    192.168.1.40:52739     192.168.1.40:443      SYN_SENT  // Remplacer l’adresse IP par l’adresse IP du serveur CS ici

Si vous trouvez des traces similaires à ce qui suit dans les journaux d’activité de l’agent du serveur cible maître, l’agent du serveur cible maître signale les erreurs sur le port 443 :

```output
#~> (11-20-2018 20:31:51):   ERROR  2508 8408 313 FAILED : PostToSVServer with error [at curlwrapper.cpp:CurlWrapper::processCurlResponse:212]   failed to post request: (7) - Couldn't connect to server
#~> (11-20-2018 20:31:54):   ERROR  2508 8408 314 FAILED : PostToSVServer with error [at curlwrapper.cpp:CurlWrapper::processCurlResponse:212]   failed to post request: (7) - Couldn't connect to server
```
 
Cette erreur peut se produire quand d’autres applications utilisent également le port 443 ou en raison d’un paramètre de pare-feu qui bloque le port.

Pour résoudre le problème :

- Vérifiez que le port 443 n’est pas bloqué par votre pare-feu.
- Si le port est inaccessible en raison d’une autre application qui utilise ce port, arrêtez et désinstallez l’application.
  - Si l’arrêt de l’application n’est pas possible, configurez un nouveau serveur CS propre.
- Redémarrez le serveur de configuration.
- Redémarrez le service IIS.

### <a name="configuration-server-is-not-connected-due-to-incorrect-uuid-entries"></a>Le serveur de configuration n’est pas connecté en raison d’entrées d’UUID incorrectes

Cette erreur peut se produire quand il y a plusieurs entrées d’UUID d’instance de serveur de configuration (CS) dans la base de données. Le problème se produit souvent quand vous clonez la machine virtuelle du serveur de configuration.

Pour résoudre le problème :

1. Supprimez la machine virtuelle obsolète/ancienne du serveur CS du serveur vCenter. Pour plus d’informations, consultez [Supprimer des serveurs et désactiver la protection](site-recovery-manage-registration-and-protection.md).
2. Connectez-vous à la machine virtuelle du serveur de configuration et à la base de données MySQL svsdb1. 
3. Exécutez la requête suivante :

    > [!IMPORTANT]
    >
    > Vérifiez que vous avez entré les détails d’UUID du serveur de configuration cloné ou l’entrée obsolète du serveur de configuration qui n’est plus utilisé pour protéger les machines virtuelles. Le fait d’entrer un UUID incorrect entraîne la perte des informations relatives à tous les éléments protégés existants.
   
    ```
        MySQL> use svsdb1;
        MySQL> delete from infrastructurevms where infrastructurevmid='<Stale CS VM UUID>';
        MySQL> commit; 
    ```
4. Actualisez la page du portail.

## <a name="an-infinite-sign-in-loop-occurs-when-entering-your-credentials"></a>Une boucle de connexion infinie se produit quand vous entrez vos informations d’identification

Après avoir entré le nom d’utilisateur et le mot de passe corrects sur le modèle OVF du serveur de configuration, la connexion à Azure continue à demander les informations d’identification correctes.

Ce problème peut se produire quand l’heure système est incorrecte.

Pour résoudre le problème :

Définissez l’heure correcte sur l’ordinateur, puis réessayez la connexion. 
