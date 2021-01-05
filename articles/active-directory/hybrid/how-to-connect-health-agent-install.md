---
title: Installer les agents Connect Health dans Azure Active Directory
description: Cet article sur Azure AD Connect Health décrit l’installation de l’agent pour les services de fédération Active Directory (AD FS) et pour la synchronisation.
services: active-directory
documentationcenter: ''
author: zhiweiwangmsft
manager: daveba
editor: curtand
ms.assetid: 1cc8ae90-607d-4925-9c30-6770a4bd1b4e
ms.service: active-directory
ms.subservice: hybrid
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 10/20/2020
ms.topic: how-to
ms.author: billmath
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: b680c275b92340cc7efba187769cb17602b08b45
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95973284"
---
# <a name="azure-ad-connect-health-agent-installation"></a>Installation de l’agent Azure AD Connect Health

Dans cet article, vous allez apprendre à installer et à configurer les agents Azure Active Directory (Azure AD) Connect Health. Pour télécharger les agents, consultez [ces instructions](how-to-connect-install-roadmap.md#download-and-install-azure-ad-connect-health-agent).

## <a name="requirements"></a>Spécifications

Le tableau suivant répertorie les conditions requises pour l’utilisation d’Azure AD Connect Health.

| Condition requise | Description |
| --- | --- |
| Azure AD Premium est installé. |Azure AD Connect Health est une fonctionnalité d’Azure AD Premium. Pour plus d’informations, consultez [Inscription à Azure AD Premium](../fundamentals/active-directory-get-started-premium.md). <br /><br />Pour démarrer une période d’évaluation gratuite de 30 jours, consultez [Démarrer l’essai gratuit](https://azure.microsoft.com/trial/get-started-active-directory/). |
| Vous êtes administrateur général dans Azure AD. |Par défaut, seuls les administrateurs généraux peuvent installer et configurer les agents d’intégrité, accéder au portail et exécuter des opérations au sein d’Azure AD Connect Health. Pour plus d’informations, consultez l’article [Administration de votre annuaire Azure AD](../fundamentals/active-directory-whatis.md). <br /><br /> En utilisant le contrôle d’accès en fonction du rôle Azure (Azure RBAC), vous pouvez autoriser d’autres utilisateurs de votre organisation à accéder à Azure AD Connect Health. Pour plus d’informations, consultez [Azure RBAC pour Azure AD Connect Health](how-to-connect-health-operations.md#manage-access-with-azure-rbac). <br /><br />**Important !** Utilisez un compte professionnel ou scolaire pour installer les agents. Vous ne pouvez pas utiliser un compte Microsoft. Pour plus d’informations, consultez [Inscription à Azure en tant qu’organisation](../fundamentals/sign-up-organization.md). |
| L’agent Azure AD Connect Health est installé sur chaque serveur cible. | Les agents d’intégrité doivent être installés et configurés sur des serveurs ciblés pour pouvoir recevoir des données et fournir des capacités de surveillance et d’analytique. <br /><br />Par exemple, pour obtenir des données de votre infrastructure de services de fédération Active Directory (AD FS), vous devez installer l’agent sur le serveur AD FS et le serveur proxy d’application web. De même, pour récupérer des données de votre infrastructure Azure AD Domain Services (Azure AD DS) locale, vous devez installer l’agent sur les contrôleurs de domaine.  |
| Les points de terminaison de service Azure ont une connectivité sortante. | Pendant l’installation et l’exécution, l’agent nécessite une connectivité vers les points de terminaison de service Azure AD Connect Health. Si les pare-feu bloquent la connectivité sortante, ajoutez les [points de terminaison de connectivité sortante](how-to-connect-health-agent-install.md#outbound-connectivity-to-the-azure-service-endpoints) à la liste autorisée. |
|La connectivité sortante est basée sur les adresses IP. | Pour plus d’informations sur le filtrage de pare-feu basé sur des adresses IP, consultez [Plages d’adresses IP Azure](https://www.microsoft.com/download/details.aspx?id=41653).|
| L’inspection TLS pour le trafic sortant est filtrée ou désactivée. | L’étape d’inscription de l’agent ou les étapes de chargement de données peuvent échouer en cas d’inspection ou d’arrêt du protocole TLS pour le trafic sortant sur la couche réseau. Pour plus d’informations, consultez [Configurer l’inspection TLS](/previous-versions/tn-archive/ee796230(v=technet.10)). |
| Les ports de pare-feu sur le serveur exécutent l’agent. |L’agent requiert que les ports de pare-feu suivants soient ouverts pour pouvoir communiquer avec les points de terminaison du service Azure AD Connect Health : <br /><li>Port TCP 443</li><li>Port TCP 5671</li> <br />La version la plus récente de l’agent ne requiert pas le port 5671. Procédez à une mise à niveau vers la dernière version pour que seul le port 443 soit requis. Pour plus d’informations, consultez [Ports et protocoles nécessaires à l’identité hybride](./reference-connect-ports.md). |
| Si la sécurité renforcée d’Internet Explorer est activée, autorisez les sites web spécifiés.  |Si la sécurité renforcée d’Internet Explorer est activée, autorisez les sites web suivants sur le serveur sur lequel vous installez l’agent :<br /><li>https:\//login.microsoftonline.com</li><li>https:\//secure.aadcdn.microsoftonline-p.com</li><li>https:\//login.windows.net</li><li>https:\//aadcdn.msftauth.net</li><li>Le serveur de fédération de votre organisation approuvé par Azure AD (par exemple, https:\//sts.contoso.com)</li> <br />Pour plus d’informations, consultez [Guide pratique pour configurer Internet Explorer](https://support.microsoft.com/help/815141/internet-explorer-enhanced-security-configuration-changes-the-browsing). Si vous avez un proxy dans votre réseau, consultez la remarque qui figure à la fin de ce tableau.|
| PowerShell version 4.0 ou ultérieure est installé. | Windows Server 2012 comprend la version 3.0 de PowerShell. Cette version n’est *pas* suffisante pour l’agent.</br></br> Windows Server 2012 R2 et les versions ultérieures incluent une version suffisamment récente de PowerShell.|
|Les normes FIPS (Federal Information Processing Standard) sont désactivées.|Les agents Azure AD Connect Health ne prennent pas en charge les normes FIPS (Federal Information Processing Standard).|

> [!IMPORTANT]
> Windows Server Core ne prend pas en charge l’installation de l’agent Azure AD Connect Health.


> [!NOTE]
> Si vous disposez d’un environnement hautement verrouillé et restreint, vous devez ajouter plus d’URL que celles que le tableau énumère pour la sécurité renforcée d’Internet Explorer. Ajoutez également les URL qui sont répertoriées dans le tableau de la section suivante.  


### <a name="outbound-connectivity-to-the-azure-service-endpoints"></a>Connectivité sortante vers les points de terminaison de service Azure

Pendant l’installation et l’exécution, l’agent nécessite une connectivité vers les points de terminaison de service Azure AD Connect Health. Si les pare-feu bloquent la connectivité sortante, assurez-vous que les URL du tableau suivant ne sont pas bloquées par défaut. 

Ne désactivez pas la supervision ou l’inspection de sécurité de ces URL. Autorisez-les plutôt comme vous le feriez pour un autre trafic Internet. 

Ces URL permettent la communication avec les points de terminaison de service Azure AD Connect Health. Plus loin dans cet article, vous apprendrez à [vérifier la connectivité sortante](#test-connectivity-to-azure-ad-connect-health-service) à l’aide de `Test-AzureADConnectHealthConnectivity`.

| Environnement de domaine | Points de terminaison de service Azure nécessaires |
| --- | --- |
| Grand public | <li>&#42;.blob.core.windows.net </li><li>&#42;.aadconnecthealth.azure.com </li><li>&#42;.servicebus.windows.net - Port: 5671 (Ce point de terminaison n’est pas requis dans la version la plus récente de l’agent.)</li><li>&#42;.adhybridhealth.azure.com/</li><li>https:\//management.azure.com </li><li>https:\//policykeyservice.dc.ad.msft.net/</li><li>https:\//login.windows.net</li><li>https:\//login.microsoftonline.com</li><li>https:\//secure.aadcdn.microsoftonline-p.com </li><li>https:\//www.office.com (Ce point de terminaison est utilisé uniquement à des fins de détection pendant l’inscription.)</li> |
| Azure Germany | <li>&#42;.blob.core.cloudapi.de </li><li>&#42;.servicebus.cloudapi.de </li> <li>&#42;.aadconnecthealth.microsoftazure.de </li><li>https:\//management.microsoftazure.de </li><li>https:\//policykeyservice.aadcdi.microsoftazure.de </li><li>https:\//login.microsoftonline.de </li><li>https:\//secure.aadcdn.microsoftonline-p.de </li><li>https:\//www.office.de (Ce point de terminaison est utilisé uniquement à des fins de détection pendant l’inscription.)</li> |
| Azure Government | <li>&#42;.blob.core.usgovcloudapi.net </li> <li>&#42;.servicebus.usgovcloudapi.net </li> <li>&#42;.aadconnecthealth.microsoftazure.us </li> <li>https:\//management.usgovcloudapi.net </li><li>https:\//policykeyservice.aadcdi.azure.us </li><li>https:\//login.microsoftonline.us </li><li>https:\//secure.aadcdn.microsoftonline-p.com </li><li>https:\//www.office.com (Ce point de terminaison est utilisé uniquement à des fins de détection pendant l’inscription.)</li> |


## <a name="install-the-agent"></a>Installer l’agent

Pour télécharger et installer l’agent Azure AD Connect Health : 

* Vérifiez que vous disposez de la [configuration requise](how-to-connect-health-agent-install.md#requirements) pour Azure AD Connect Health.
* Prise en main d’Azure AD Connect Health pour AD FS :
    * [Téléchargez l’agent Azure AD Connect Health pour AD FS](https://go.microsoft.com/fwlink/?LinkID=518973).
    * Consultez les [instructions d’installation](#install-the-agent-for-ad-fs).
* Prise en main d’Azure AD Connect Health pour la synchronisation :
    * [Téléchargez et installez la dernière version d’Azure AD Connect](https://go.microsoft.com/fwlink/?linkid=615771). L’agent d’intégrité pour la synchronisation sera ajouté dans le cadre de l’installation d’Azure AD Connect (version 1.0.9125.0 ou ultérieure).
* Prise en main d’Azure AD Connect Health pour Azure AD DS :
    * [Téléchargez l’agent Azure AD Connect Health pour Azure AD DS](https://go.microsoft.com/fwlink/?LinkID=820540).
    * Consultez les [instructions d’installation](#install-the-agent-for-azure-ad-ds).

## <a name="install-the-agent-for-ad-fs"></a>Installer l’agent pour AD FS

> [!NOTE]
> Votre serveur AD FS doit être différent de votre serveur de synchronisation. N’installez pas l’agent AD FS sur votre serveur de synchronisation.
>

Avant d’installer l’agent, assurez-vous que le nom d’hôte de votre serveur AD FS est unique et qu’il n’existe pas dans le service AD FS.
Pour démarrer l’installation de l’agent, double-cliquez sur le fichier *.exe* que vous avez téléchargé. Dans la première fenêtre, sélectionnez **Installer**.

![Capture d’écran montrant la fenêtre d’installation pour l’agent Azure AD Connect Health AD FS.](./media/how-to-connect-health-agent-install/install1.png)

Une fois l’installation terminée, sélectionnez **Configurer**.

![Capture d’écran montrant le message de confirmation de l’installation de l’agent Azure AD Connect Health AD FS.](./media/how-to-connect-health-agent-install/install2.png)

Une fenêtre PowerShell s’ouvre pour démarrer le processus d’inscription de l’agent. Lorsque vous y êtes invité, connectez-vous à l’aide d’un compte Azure AD disposant des autorisations nécessaires pour inscrire l’agent. Par défaut, le compte d’administrateur général dispose des autorisations.

![Capture d’écran montrant la fenêtre de connexion pour Azure AD Connect Health AD FS.](./media/how-to-connect-health-agent-install/install3.png)

Une fois que vous êtes connecté, PowerShell continue. Une fois l’opération terminée, vous pouvez fermer PowerShell. La configuration est terminée.

À ce stade, les services de l’agent doivent démarrer automatiquement pour permettre à l’agent de charger en toute sécurité les données requises pour le service cloud.

Si vous n’avez pas respecté toutes les conditions préalables, des avertissements s’affichent dans la fenêtre PowerShell. Assurez-vous que vous disposez de la [configuration requise](how-to-connect-health-agent-install.md#requirements) avant d’installer l’agent. La capture d’écran suivante présente un exemple de ces avertissements.

![Capture d’écran montrant le script de configuration d’Azure AD Connect Health AD FS.](./media/how-to-connect-health-agent-install/install4.png)

Pour vérifier que l’agent a été installé, recherchez les services suivants sur le serveur. Si vous avez terminé la configuration, ils doivent déjà être en cours d’exécution. Dans le cas contraire, ils sont arrêtés tant que la configuration n’est pas terminée.

* Azure AD Connect Health AD FS Diagnostics Service
* Azure AD Connect Health AD FS Insights Service
* Azure AD Connect Health AD FS Monitoring Service

![Capture d’écran montrant les services Azure AD Connect Health AD FS.](./media/how-to-connect-health-agent-install/install5.png)


### <a name="enable-auditing-for-ad-fs"></a>Activer l’audit pour AD FS

> [!NOTE]
> Cette section s’applique uniquement aux serveurs AD FS. Il est inutile de suivre ces étapes sur les serveurs proxy d’application web.
>

La fonctionnalité Analyse de l’utilisation doit collecter et analyser les données. Ainsi, l’agent Azure AD Connect Health a besoin des informations contenues dans les journaux d’audit AD FS. Ces journaux ne sont pas activés par défaut. Utilisez les procédures suivantes pour activer l’audit AD FS et localiser les journaux d’audit AD FS sur vos serveurs AD FS.

#### <a name="to-enable-auditing-for-ad-fs-on-windows-server-2012-r2"></a>Pour activer l’audit pour AD FS sur Windows Server 2012 R2

1. Dans l’écran de démarrage, ouvrez **Gestionnaire de serveur**, puis ouvrez **Stratégie de sécurité locale**. Ou, dans la barre des tâches, ouvrez **Gestionnaire de serveur** et sélectionnez **Outils/Stratégie de sécurité locale**.
2. Accédez au dossier *Paramètres de sécurité\Stratégies locales\Attribution des droits utilisateur*. Double-cliquez ensuite sur **Générer des audits de sécurité**.
3. Sous l’onglet **Paramètre de sécurité locale** , vérifiez que le compte de service AD FS est répertorié. S’il n’est pas répertorié, sélectionnez **Ajouter un utilisateur ou un groupe** et ajoutez-le à la liste. Sélectionnez ensuite **OK**.
4. Pour activer l’audit, ouvrez une fenêtre d’invite de commandes avec des privilèges élevés. Exécutez ensuite la commande suivante : 
    
    `auditpol.exe /set /subcategory:{0CCE9222-69AE-11D9-BED3-505054503030} /failure:enable /success:enable`
1. Fermez **Stratégie de sécurité locale**.
    >[!Important]
    >Les étapes suivantes sont requises uniquement pour les serveurs AD FS principaux. 
1. Fermez le composant logiciel enfichable **Gestion AD FS**. (Dans **Gestionnaire de serveur**, sélectionnez **Outils** > **Gestion d’AD FS**.)
1. Dans le volet **Actions**, sélectionnez **Modifier les propriétés du service de fédération**.
1. Dans la boîte de dialogue **Propriétés du service de fédération**, sélectionnez l’onglet **Événements**.
1. Cochez les cases **Audits des succès et Audits des échecs**, puis sélectionnez **OK**.
1. Pour activer la journalisation commentée par le biais de PowerShell, utilisez la commande suivante : 

    `Set-AdfsProperties -LOGLevel Verbose`

#### <a name="to-enable-auditing-for-ad-fs-on-windows-server-2016"></a>Pour activer l’audit pour AD FS sur Windows Server 2016

1. Dans l’écran de démarrage, ouvrez **Gestionnaire de serveur**, puis ouvrez **Stratégie de sécurité locale**. Ou, dans la barre des tâches, ouvrez **Gestionnaire de serveur** et sélectionnez **Outils/Stratégie de sécurité locale**.
2. Accédez au dossier *Paramètres de sécurité\Stratégies locales\Attribution des droits utilisateur*, puis double-cliquez sur **Générer des audits de sécurité**.
3. Sous l’onglet **Paramètre de sécurité locale** , vérifiez que le compte de service AD FS est répertorié. S’il n’est pas répertorié, sélectionnez **Ajouter un utilisateur ou un groupe** et ajoutez le compte de service AD FS à la liste. Sélectionnez ensuite **OK**.
4. Pour activer l’audit, ouvrez une fenêtre d’invite de commandes avec des privilèges élevés. Exécutez ensuite la commande suivante : 

    `auditpol.exe /set /subcategory:{0CCE9222-69AE-11D9-BED3-505054503030} /failure:enable /success:enable`
1. Fermez **Stratégie de sécurité locale**.
    >[!Important]
    >Les étapes suivantes sont requises uniquement pour les serveurs AD FS principaux.
1. Fermez le composant logiciel enfichable **Gestion AD FS**. (Dans **Gestionnaire de serveur**, sélectionnez **Outils** > **Gestion d’AD FS**.)
1. Dans le volet **Actions**, sélectionnez **Modifier les propriétés du service de fédération**.
1. Dans la boîte de dialogue **Propriétés du service de fédération**, sélectionnez l’onglet **Événements**.
1. Cochez les cases **Audits des succès et Audits des échecs**, puis sélectionnez **OK**. Les audits des succès et les audits des échecs doivent être activés par défaut.
1. Ouvrez une fenêtre PowerShell et exécutez la commande suivante : 

    `Set-AdfsProperties -AuditLevel Verbose`

Le niveau d’audit « de base » est activé par défaut. Pour plus d’informations, consultez [Amélioration de l’audit AD FS dans Windows Server 2016](/windows-server/identity/ad-fs/technical-reference/auditing-enhancements-to-ad-fs-in-windows-server).


#### <a name="to-locate-the-ad-fs-audit-logs"></a>Pour localiser les journaux d’audit AD FS

1. Ouvrez l’ **Observateur d’événements**.
2. Accédez à **Journaux Windows**, puis sélectionnez **Sécurité**.
3. Sur la droite, sélectionnez **Filtrer les journaux d’activité actuels**.
4. Dans **Sources d’événement**, sélectionnez **Audit AD FS**.

    Pour plus d’informations sur les journaux d’audit, consultez [les questions relatives aux opérations](reference-connect-health-faq.md#operations-questions).

    ![Capture d’écran montrant la fenêtre Filtrer les journaux d’activité actuels. Dans le champ « Sources d’événement », « Audit AD FS » est sélectionné.](./media/how-to-connect-health-agent-install/adfsaudit.png)

> [!WARNING]
> Une stratégie de groupe peut désactiver l’audit AD FS. Si l’audit AD FS est désactivé, l’analyse de l’utilisation sur les activités de connexion n’est pas disponible. Vérifiez que vous n’avez pas de stratégie de groupe qui désactive l’audit AD FS.
>


## <a name="install-the-agent-for-sync"></a>Installer l’agent pour la synchronisation

L’agent Azure AD Connect Health pour la synchronisation est installé automatiquement dans la version la plus récente d’Azure AD Connect. Pour utiliser Azure AD Connect pour la synchronisation, [téléchargez la version la plus récente d’Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594) et installez-la.

Pour vérifier que l’agent a été installé, recherchez les services suivants sur le serveur. Si vous avez terminé la configuration, les services doivent déjà être en cours d’exécution. Dans le cas contraire, ils sont arrêtés tant que la configuration n’est pas terminée.

* Azure AD Connect Health Sync Insights Service
* Azure AD Connect Health Sync Monitoring Service

![Capture d’écran montrant les services Azure AD Connect Health pour la synchronisation en cours d’exécution sur le serveur.](./media/how-to-connect-health-agent-install/services.png)

> [!NOTE]
> N’oubliez pas que vous devez disposer d’Azure AD Premium pour utiliser Azure AD Connect Health. Si vous ne disposez pas d’Azure AD Premium, vous ne pouvez pas terminer la configuration dans le portail Azure. Pour plus d’informations, consultez la [configuration requise](how-to-connect-health-agent-install.md#requirements).
>
>

## <a name="manually-register-azure-ad-connect-health-for-sync"></a>Inscrire manuellement Azure AD Connect Health pour la synchronisation

Si l’inscription de l’agent Azure AD Connect Health pour la synchronisation échoue après avoir correctement installé Azure AD Connect, vous pouvez utiliser une commande PowerShell pour inscrire l’agent manuellement.

> [!IMPORTANT]
> Utilisez cette commande PowerShell uniquement si l’inscription de l’agent échoue après l’installation d’Azure AD Connect.
>
>

Inscrivez manuellement l’agent Azure AD Connect Health pour la synchronisation à l’aide de la commande PowerShell suivante. Les services Azure AD Connect Health démarreront une fois l’agent correctement enregistré.

`Register-AzureADConnectHealthSyncAgent -AttributeFiltering $false -StagingMode $false`

La commande prend les paramètres qui suivent :

* **AttributeFiltering** : `$true` (par défaut), si Azure AD Connect ne synchronise pas le jeu d’attributs par défaut et a été personnalisé pour utiliser un jeu d’attributs filtré. Sinon, utilisez `$false`.
* **StagingMode** : `$false` (par défaut), si le serveur Azure AD Connect n’est *pas* en mode intermédiaire. Si le serveur est configuré pour être en mode intermédiaire, utilisez `$true`.

Lorsque vous êtes invité à vous authentifier, utilisez le même compte d’administrateur général (par exemple admin@domain.onmicrosoft.com) que celui que vous avez utilisé pour configurer Azure AD Connect.

## <a name="install-the-agent-for-azure-ad-ds"></a>Installer l’agent pour Azure AD DS

Pour démarrer l’installation de l’agent, double-cliquez sur le fichier *.exe* que vous avez téléchargé. Dans la première fenêtre, sélectionnez **Installer**.

![Capture d’écran montrant la fenêtre d’installation de l’agent Azure AD Connect Health pour AD DS.](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install1.png)

Une fois l’installation terminée, sélectionnez **Configurer**.

![Capture d’écran montrant la fenêtre qui termine l’installation de l’agent Azure AD Connect Health pour Azure AD DS.](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install2.png)

Une fenêtre d’invite de commandes s’ouvre. PowerShell exécute `Register-AzureADConnectHealthADDSAgent`. Lorsque vous y êtes invité, connectez-vous à Azure.

![Capture d’écran montrant la fenêtre de connexion de l’agent Azure AD Connect Health pour Azure AD DS.](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install3.png)

Une fois que vous êtes connecté, PowerShell continue. Une fois l’opération terminée, vous pouvez fermer PowerShell. La configuration est terminée.

À ce stade, les services doivent être démarrés automatiquement, permettant à l’agent de surveiller et de collecter les données. Si vous n’avez pas satisfait la configuration requise décrite dans les sections précédentes, des avertissements s’affichent dans la fenêtre PowerShell. Assurez-vous que vous disposez de la [configuration requise](how-to-connect-health-agent-install.md#requirements) avant d’installer l’agent. La capture d’écran suivante présente un exemple de ces avertissements.

![Capture d’écran montrant un avertissement pour l’agent Azure AD Connect Health concernant la configuration d’Azure AD DS.](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install4.png)

Pour vérifier que l’agent a été installé, recherchez les services suivants sur le contrôleur de domaine :

* Azure AD Connect Health AD DS Insights Service
* Azure AD Connect Health AD DS Monitoring Service

Si vous avez terminé la configuration, ces services doivent déjà être en cours d’exécution. Dans le cas contraire, ils sont arrêtés jusqu’à la fin de la configuration.

![Capture d’écran montrant les services en cours d’exécution sur le contrôleur de domaine.](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install5.png)

### <a name="quickly-install-the-agent-on-multiple-servers"></a>Installer rapidement l’agent sur plusieurs serveurs

1. Créez un compte d’utilisateur dans Azure AD. Sécurisez-le à l’aide d’un mot de passe.
2. Attribuez le rôle **Propriétaire** à ce compte Azure AD local dans Azure AD Connect Health à l’aide du portail. Procédez [comme suit](how-to-connect-health-operations.md#manage-access-with-azure-rbac). Affectez le rôle à toutes les instances de services. 
3. Téléchargez le fichier MSI *.exe* dans le contrôleur de domaine local pour l’installation.
4. Exécutez le script suivant. Remplacez les paramètres par votre nouveau compte d’utilisateur et son mot de passe. 

    ```powershell
    AdHealthAddsAgentSetup.exe /quiet
    Start-Sleep 30
    $userName = "NEWUSER@DOMAIN"
    $secpasswd = ConvertTo-SecureString "PASSWORD" -AsPlainText -Force
    $myCreds = New-Object System.Management.Automation.PSCredential ($userName, $secpasswd)
    import-module "C:\Program Files\Azure Ad Connect Health Adds Agent\PowerShell\AdHealthAdds"
     
    Register-AzureADConnectHealthADDSAgent -Credential $myCreds
    
    ```

Lorsque vous avez terminé, vous pouvez supprimer l’accès du compte local en effectuant une ou plusieurs des tâches suivantes : 
* Supprimer l’attribution de rôle au compte local pour Azure AD Connect Health.
* retournez le mot de passe du compte local ; 
* Désactiver le compte local Azure AD.
* Supprimer le compte local Azure AD.  

## <a name="register-the-agent-by-using-powershell"></a>Inscrire l’agent à l’aide de PowerShell

Après avoir installé le fichier *setup.exe* de l’agent approprié, vous pouvez inscrire l’agent à l’aide des commandes PowerShell suivantes, en fonction du rôle. Ouvrez une fenêtre PowerShell et exécutez la commande appropriée :

```powershell
    Register-AzureADConnectHealthADFSAgent
    Register-AzureADConnectHealthADDSAgent
    Register-AzureADConnectHealthSyncAgent

```

Ces commandes acceptent `Credential` en tant que paramètre pour terminer l’inscription de manière non interactive ou pour terminer l’inscription sur un ordinateur qui exécute Server Core. N’oubliez pas les points suivants :
* Vous pouvez capturer `Credential` dans une variable PowerShell qui est transmise en tant que paramètre.
* Vous pouvez fournir n’importe quelle identité Azure AD qui dispose des autorisations nécessaires pour inscrire les agents et pour laquelle l’authentification multifacteur n’est *pas* activée.
* Par défaut, les administrateurs généraux sont autorisés à inscrire les agents. Vous pouvez également autoriser des identités moins privilégiées à effectuer cette étape. Pour plus d’informations, consultez [Azure RBAC](how-to-connect-health-operations.md#manage-access-with-azure-rbac).

```powershell
    $cred = Get-Credential
    Register-AzureADConnectHealthADFSAgent -Credential $cred

```

## <a name="configure-azure-ad-connect-health-agents-to-use-http-proxy"></a>Configuration des agents Azure AD Connect Health pour utiliser le proxy HTTP

Vous pouvez configurer des agents Azure AD Connect Health pour utiliser un proxy HTTP.

> [!NOTE]
> * `Netsh WinHttp set ProxyServerAddress` n’est pas pris en charge. L’agent utilise System.Net au lieu des services HTTP Windows pour effectuer des requêtes web.
> * L’adresse de proxy HTTP configurée sert à transmettre des messages HTTPS chiffrés.
> * Les serveurs proxy authentifiés (à l’aide de HTTPBasic) ne sont pas pris en charge.
>
>

### <a name="change-the-agent-proxy-configuration"></a>Modifier la configuration de l’agent proxy

Pour configurer l’agent Azure AD Connect Health pour qu’il utilise un proxy HTTP, vous pouvez :
* Importer les paramètres de proxy existants.
* Spécifier les adresses du proxy manuellement.
* Effacer la configuration de proxy existante.

> [!NOTE]
> Pour mettre à jour les paramètres de proxy, vous devez redémarrer tous les services de l’agent Azure AD Connect Health. Exécutez la commande suivante :
>
> `Restart-Service AzureADConnectHealth*`

#### <a name="import-existing-proxy-settings"></a>Importer les paramètres de proxy existants

Vous pouvez importer les paramètres du proxy HTTP d’Internet Explorer afin que les agents Azure AD Connect Health puissent les utiliser. Sur chacun des serveurs qui exécutent l’agent d’intégrité, exécutez la commande PowerShell suivante :

```powershell
Set-AzureAdConnectHealthProxySettings -ImportFromInternetSettings
```

Vous pouvez importer les paramètres du proxy WinHTTP afin que les agents Azure AD Connect Health puissent les utiliser. Sur chacun des serveurs qui exécutent l’agent d’intégrité, exécutez la commande PowerShell suivante :

```powershell
Set-AzureAdConnectHealthProxySettings -ImportFromWinHttp
```

#### <a name="specify-proxy-addresses-manually"></a>Spécifier les adresses du proxy manuellement

Vous pouvez spécifier manuellement un serveur proxy. Sur chacun des serveurs qui exécutent l’agent d’intégrité, exécutez la commande PowerShell suivante :

```powershell
Set-AzureAdConnectHealthProxySettings -HttpsProxyAddress address:port
```

Voici un exemple : 

`Set-AzureAdConnectHealthProxySettings -HttpsProxyAddress myproxyserver: 443`

Dans cet exemple :
* Le paramètre `address` peut être un nom de serveur DNS pouvant être résolu ou une adresse IPv4.
* Vous pouvez omettre `port`. Si vous le faites, le port 443 est le port par défaut.

#### <a name="clear-the-existing-proxy-configuration"></a>Effacer la configuration de proxy existante

Vous pouvez effacer la configuration du proxy en exécutant la commande suivante :

```powershell
Set-AzureAdConnectHealthProxySettings -NoProxy
```

### <a name="read-current-proxy-settings"></a>Lecture des paramètres de proxy actuels

Vous pouvez lire les paramètres de proxy actuels en exécutant la commande suivante :

```powershell
Get-AzureAdConnectHealthProxySettings
```

## <a name="test-connectivity-to-azure-ad-connect-health-service"></a>Tester la connectivité au service Azure AD Connect Health

Parfois, l’agent Azure AD Connect Health peut perdre la connectivité avec le service Azure AD Connect Health. Les causes de cette perte de connectivité peuvent inclure des problèmes de réseau, des problèmes d’autorisation et divers autres problèmes.

Si l’agent ne peut pas envoyer de données au service Azure AD Connect Health pendant plus de deux heures, l’alerte suivante s’affiche dans le portail : « Les données du service de contrôle d’intégrité ne sont pas à jour. » 

Vous pouvez savoir si l’agent Azure AD Connect Health concerné peut charger des données vers le service Azure AD Connect Health en exécutant la commande PowerShell suivante :

```powershell
Test-AzureADConnectHealthConnectivity -Role ADFS
```

Le paramètre de rôle peut avoir les valeurs suivantes :

* ADFS
* Synchronisation
* AJOUTE

> [!NOTE]
> Pour utiliser l’outil de connectivité, vous devez d’abord inscrire l’agent. Si vous ne pouvez pas terminer l’inscription de l’agent, assurez-vous d’avoir rempli toutes les [conditions requises](how-to-connect-health-agent-install.md#requirements) pour Azure AD Connect Health. La connectivité est testée par défaut lors de l’inscription de l’agent.
>
>

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles connexes suivants :

* [Azure AD Connect Health](./whatis-azure-ad-connect.md)
* [Opérations Azure AD Connect Health](how-to-connect-health-operations.md)
* [Utilisation d’Azure AD Connect Health avec AD FS](how-to-connect-health-adfs.md)
* [Utilisation d’Azure AD Connect Health pour la synchronisation](how-to-connect-health-sync.md)
* [Utilisation d’Azure AD Connect Health avec Azure AD DS](how-to-connect-health-adds.md)
* [Forum Aux Questions (FAQ) Azure AD Connect Health](reference-connect-health-faq.md)
* [Historique des versions d’Azure AD Connect Health](reference-connect-health-version-history.md)
