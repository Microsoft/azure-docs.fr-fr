---
title: Configurer manuellement des appareils joints à Azure Active Directory hybride | Microsoft Docs
description: Découvrez comment configurer manuellement des appareils joints à Azure Active Directory hybride.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: tutorial
ms.date: 05/14/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sandeo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 5316a1647c96076696b14de157e74e2155a6b368
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96860012"
---
# <a name="tutorial-configure-hybrid-azure-active-directory-joined-devices-manually"></a>Tutoriel : Configurer manuellement des appareils joints à Azure Active Directory hybride

La fonction de gestion des appareils intégrée à Azure Active Directory (Azure AD) vous permet de vous assurer que les utilisateurs accèdent à vos ressources à partir d’appareils qui répondent à vos standards de conformité et de sécurité. Pour plus d’informations, consultez [Présentation de la gestion des appareils dans Azure Active Directory](overview.md).

> [!TIP]
> Si vous avez la possibilité d’utiliser Azure AD Connect, consultez les tutoriels connexes pour les domaines [managés](hybrid-azuread-join-managed-domains.md) ou [fédérés](hybrid-azuread-join-federated-domains.md). En recourant à Azure AD Connect, vous pouvez simplifier considérablement la configuration de la jonction Azure AD hybride.

Si vous disposez d’un environnement Active Directory local et que vous souhaitez lier à Azure AD vos appareils joints à un domaine, vous pouvez y parvenir en configurant simplement des appareils hybrides joints à Azure AD. Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Configurer des appareils joints à Azure AD hybrides manuellement
> * Configurer un point de connexion de service
> * Configurer l’émission de revendications
> * Activer des appareils Windows de bas niveau
> * Vérifier des appareils joints
> * Résoudre les problèmes liés à votre implémentation

## <a name="prerequisites"></a>Prérequis

Ce tutoriel part du principe que vous êtes familiarisé avec ce qui suit :

* [Présentation de la gestion des appareils dans Azure Active Directory](./overview.md)
* [Planifier l’implémentation de la jonction Azure Active Directory hybride](hybrid-azuread-join-plan.md)
* [Contrôler la jointure d’Azure AD hybride de vos appareils](hybrid-azuread-join-control.md)

Avant de commencer à activer des appareils hybrides joints à Azure AD dans votre organisation, vérifiez que :

* Vous avez une version à jour d’Azure AD Connect.
* Azure AD Connect a synchronisé les objets ordinateurs des appareils que vous souhaitez rendre hybrides et joints à Azure AD. Si les objets ordinateurs appartiennent à des unités d’organisation, celles-ci doivent être également configurées du point de vue de la synchronisation dans Azure AD Connect.

Azure AD Connect :

* Conserve l’association entre le compte d’ordinateur dans votre instance Active Directory locale et l’objet appareil dans Azure AD.
* Permet d’utiliser d’autres fonctionnalités liées aux appareils, telles que Windows Hello Entreprise.

Assurez-vous que les URL suivantes sont accessibles à partir d’ordinateurs au sein du réseau de votre organisation pour l’inscription d’ordinateurs à Azure AD :

* `https://enterpriseregistration.windows.net`
* `https://login.microsoftonline.com`
* `https://device.login.microsoftonline.com`
* Le STS de votre organisation (pour les domaines fédérés), qui doit être inclus dans les paramètres intranet locaux de l’utilisateur

> [!WARNING]
> Si votre organisation utilise des serveurs proxy qui interceptent le trafic SSL dans des scénarios de protection contre la perte de données ou de restrictions de locataire Azure AD par exemple, veillez à exclure le trafic à destination de « https://device.login.microsoftonline.com  » de l’inspection TLS. Faute d’exclure « https://device.login.microsoftonline.com  », l’authentification par certificat client peut être sujette à des interférences, ce qui occasionne des problèmes d’inscription d’appareil et d’accès conditionnel en fonction de l’appareil.

Si votre organisation envisage d’utiliser l’authentification unique fluide, l’URL suivante doit être accessible à partir des ordinateurs à l’intérieur de votre organisation. Elle doit également être ajoutée à la zone intranet local de l’utilisateur.

* `https://autologon.microsoftazuread-sso.com`

De plus, le paramètre suivant doit être activé dans la zone intranet de l’utilisateur : « Autoriser les mises à jour de la barre d’état via le script ».

Si votre organisation utilise une configuration gérée (non fédérée) avec Active Directory en local et n’utilise pas les services de fédération Active Directory (AD FS) pour établir la fédération avec Azure AD, la jonction Azure AD hybride sur Windows 10 s’appuie sur les objets ordinateurs dans Active Directory à synchroniser avec Azure AD. Assurez-vous que toutes les unités d’organisation qui contiennent les objets ordinateurs devant avoir une jonction Azure AD hybride peuvent être synchronisées dans la configuration de la synchronisation Azure AD Connect.

Pour les appareils Windows 10 exécutant la version 1703 ou antérieure, si votre organisation nécessite un accès à Internet via un proxy sortant, vous devez implémenter la découverte automatique de proxy web (WPAD) pour permettre aux ordinateurs Windows 10 de s’inscrire à Azure AD.

Tout d’abord, pour Windows 10 1803, même si un appareil tente une jonction Azure AD hybride dans un domaine fédéré par le biais d’AD FS et échoue, et si Azure AD Connect est configuré pour synchroniser les objets ordinateur/d’appareil avec Azure AD, l’appareil essaie d’effectuer la jonction Azure AD hybride en utilisant l’ordinateur/appareil synchronisé.

Pour vérifier si l’appareil est en mesure d’accéder aux ressources Microsoft ci-dessus sous le compte système, vous pouvez utiliser le script [Tester la connectivité de l’inscription des appareils](https://gallery.technet.microsoft.com/Test-Device-Registration-3dc944c0).

## <a name="verify-configuration-steps"></a>Vérifier les étapes de configuration

Vous pouvez configurer des appareils hybrides joints à Azure AD pour différents types de plateformes d’appareils Windows. Cet article inclut les étapes requises pour tous les scénarios de configuration classiques.  

Pour obtenir une vue d’ensemble des étapes requises par votre scénario, utilisez le tableau ci-après :  

| Étapes | Appareils Windows actuels et synchronisation du hachage de mot de passe | Appareils Windows actuels et fédération | Appareils Windows de bas niveau |
| :--- | :---: | :---: | :---: |
| Configurer le point de connexion de service | ![Vérification][1] | ![Vérification][1] | ![Vérification][1] |
| Configurer l’émission de revendications |     | ![Vérification][1] | ![Vérification][1] |
| Activer les appareils non-Windows 10 |       |        | ![Vérification][1] |
| Vérifier des appareils joints | ![Vérification][1] | ![Vérification][1] | ![Vérification][1] |

## <a name="configure-a-service-connection-point"></a>Configurer un point de connexion de service

Vos appareils utilisent un objet point de connexion de service (SCP) lors de l’inscription pour découvrir les informations de locataire Azure AD. Dans votre instance Active Directory locale, l’objet SCP des appareils hybrides joints à Azure AD doit exister dans la partition de contexte d’appellation de configuration de la forêt de l’ordinateur. Il n’existe qu’un seul contexte d’appellation de configuration par forêt. Dans une configuration Active Directory multi-forêt, le point de connexion de service doit exister dans toutes les forêts qui contiennent des ordinateurs joints à un domaine.

Pour récupérer le contexte d’appellation de configuration de votre forêt, vous pouvez utiliser l’applet de commande [**Get-ADRootDSE**](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/ee617246(v=technet.10)).  

Pour une forêt avec le nom de domaine Active Directory *fabrikam.com*, le contexte d’appellation de configuration est le suivant :

`CN=Configuration,DC=fabrikam,DC=com`

Dans votre forêt, l’objet SCP pour l’inscription automatique des appareils joints à un domaine se trouve à l’emplacement suivant :  

`CN=62a0ff2e-97b9-4513-943f-0d221bd30080,CN=Device Registration Configuration,CN=Services,[Your Configuration Naming Context]`

Selon la façon dont vous avez déployé Azure AD Connect, l’objet SCP est peut-être déjà configuré.
Vous pouvez vérifier l’existence de l’objet et récupérer les valeurs de détection à l’aide du script Windows PowerShell suivant :

   ```PowerShell
   $scp = New-Object System.DirectoryServices.DirectoryEntry;

   $scp.Path = "LDAP://CN=62a0ff2e-97b9-4513-943f-0d221bd30080,CN=Device Registration Configuration,CN=Services,CN=Configuration,DC=fabrikam,DC=com";

   $scp.Keywords;
   ```

La sortie de **$scp.Keywords** présente les informations de locataire Azure AD. Voici un exemple :

   ```
   azureADName:microsoft.com
   azureADId:72f988bf-86f1-41af-91ab-2d7cd011db47
   ```

Si le point de connexion de service n’existe pas, vous pouvez le créer en exécutant l’applet de commande `Initialize-ADSyncDomainJoinedComputerSync` sur votre serveur Azure AD Connect. Des informations d’identification d’administrateur d’entreprise sont nécessaires pour exécuter cette applet de commande.  

Cette applet de commande :

* Crée le point de connexion de service dans la forêt Active Directory à laquelle Azure AD Connect est connecté.
* Vous demande de spécifier le paramètre `AdConnectorAccount`. Ce compte est configuré en tant que compte de connecteur Active Directory dans Azure AD Connect.


Le script ci-après présente un exemple d’utilisation de l’applet de commande. Dans ce script, la chaîne `$aadAdminCred = Get-Credential` exige que vous tapiez un nom d’utilisateur. Vous devez indiquer le nom d’utilisateur au format de nom d’utilisateur principal (UPN) (`user@example.com`).

   ```PowerShell
   Import-Module -Name "C:\Program Files\Microsoft Azure Active Directory Connect\AdPrep\AdSyncPrep.psm1";

   $aadAdminCred = Get-Credential;

   Initialize-ADSyncDomainJoinedComputerSync –AdConnectorAccount [connector account name] -AzureADCredentials $aadAdminCred;
   ```

L’applet de commande `Initialize-ADSyncDomainJoinedComputerSync` :

* Utilise le module Active Directory PowerShell et les outils Azure Active Directory Domain Services (Azure AD DS). Ces outils supposent que les services web Active Directory s’exécutent sur un contrôleur de domaine. Les services Web Active Directory sont pris en charge sur les contrôleurs de domaine exécutant Windows Server 2008 R2 et les versions ultérieures.
* Est pris en charge seulement par le module MSOnline PowerShell version 1.1.166.0. Pour télécharger ce module, utilisez ce [lien](https://www.powershellgallery.com/packages/MSOnline/1.1.166.0).
* Si les outils AD DS ne sont pas installés, `Initialize-ADSyncDomainJoinedComputerSync` échoue. Vous pouvez installer les outils AD DS via le Gestionnaire de serveur, sous **Fonctionnalités** > **Outils d’administration de serveur distant** > **Outils d’administration de rôles**.

Dans le cas de contrôleurs de domaine exécutant Windows Server 2008 ou des versions antérieures, utilisez le script suivant pour créer le point de connexion de service. Dans une configuration multi-forêt, utilisez le script ci-dessous pour créer le point de connexion de service dans chacune des forêts contenant des ordinateurs.

   ```PowerShell
   $verifiedDomain = "contoso.com" # Replace this with any of your verified domain names in Azure AD
   $tenantID = "72f988bf-86f1-41af-91ab-2d7cd011db47" # Replace this with you tenant ID
   $configNC = "CN=Configuration,DC=corp,DC=contoso,DC=com" # Replace this with your Active Directory configuration naming context

   $de = New-Object System.DirectoryServices.DirectoryEntry
   $de.Path = "LDAP://CN=Services," + $configNC
   $deDRC = $de.Children.Add("CN=Device Registration Configuration", "container")
   $deDRC.CommitChanges()

   $deSCP = $deDRC.Children.Add("CN=62a0ff2e-97b9-4513-943f-0d221bd30080", "serviceConnectionPoint")
   $deSCP.Properties["keywords"].Add("azureADName:" + $verifiedDomain)
   $deSCP.Properties["keywords"].Add("azureADId:" + $tenantID)

   $deSCP.CommitChanges()
   ```

Dans le script précédent, `$verifiedDomain = "contoso.com"` est un espace réservé. Remplacez-le par un de vos noms de domaine vérifiés dans Azure AD. Vous devez posséder le domaine pour pouvoir l’utiliser.

Pour plus d’informations sur les noms de domaine vérifiés, consultez [Ajouter un nom de domaine personnalisé à Azure Active Directory](../fundamentals/add-custom-domain.md).

Pour obtenir la liste de vos domaines d’entreprise vérifiés, vous pouvez utiliser le cmdlet [Get-AzureADDomain](/powershell/module/Azuread/Get-AzureADDomain).

![Liste des domaines d’entreprise](./media/hybrid-azuread-join-manual/01.png)

## <a name="set-up-issuance-of-claims"></a>Configurer l’émission de revendications

Dans une configuration Azure AD fédérée, les appareils s’appuient sur AD FS ou sur un service de fédération local à partir d’un partenaire Microsoft pour s’authentifier auprès d’Azure AD. Les appareils s’authentifient pour obtenir un jeton d’accès afin de s’inscrire auprès du service Azure Active Directory Device Registration Service (Azure DRS).

Les appareils Windows actuels s’authentifient à l’aide de l’authentification Windows intégrée auprès d’un point de terminaison WS-Trust actif (version 1.3 ou 2005) hébergé par le service de fédération local.

Lorsque vous utilisez AD FS, vous devez activer les points de terminaison WS-Trust suivants
- `/adfs/services/trust/2005/windowstransport`
- `/adfs/services/trust/13/windowstransport`
- `/adfs/services/trust/2005/usernamemixed`
- `/adfs/services/trust/13/usernamemixed`
- `/adfs/services/trust/2005/certificatemixed`
- `/adfs/services/trust/13/certificatemixed`

> [!WARNING]
> **adfs/services/trust/2005/windowstransport** et **adfs/services/trust/13/windowstransport** doivent tous les deux être activés en tant que points de terminaison uniquement accessibles sur intranet et ils NE doivent PAS être exposés comme points de terminaison accessibles sur extranet via le proxy d’application web. Pour en savoir plus sur la désactivation des points de terminaison Windows WS-Trust, consultez [Désactiver les points de terminaison Windows WS-Trust sur le proxy](/windows-server/identity/ad-fs/deployment/best-practices-securing-ad-fs#disable-ws-trust-windows-endpoints-on-the-proxy-ie-from-extranet). Vous pouvez visualiser les points de terminaison qui sont activés par le biais de la console de gestion AD FS sous **Service** > **Points de terminaison**.

> [!NOTE]
>Si vous n’utilisez pas AD FS en tant que service de fédération local, suivez les instructions de votre fournisseur pour vous assurer que celui-ci prend en charge les points de terminaison WS-Trust 1.3 ou 2005 et que ces derniers sont publiés par le biais du fichier Metadata Exchange (MEX).

Pour que l’inscription d’appareils puisse se terminer, les revendications ci-après doivent exister dans le jeton que reçoit Azure DRS. Azure DRS crée un objet appareil dans Azure AD avec certaines de ces informations. Azure AD Connect se sert de ces informations pour associer l’objet appareil créé au compte d’ordinateur localement.

* `http://schemas.microsoft.com/ws/2012/01/accounttype`
* `http://schemas.microsoft.com/identity/claims/onpremobjectguid`
* `http://schemas.microsoft.com/ws/2008/06/identity/claims/primarysid`

Si vous disposez de plusieurs noms de domaine vérifiés, vous devez fournir la revendication ci-après pour les ordinateurs :

* `http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid`

Si vous émettez déjà une revendication ImmutableID (par exemple, en utilisant `mS-DS-ConsistencyGuid` ou un autre attribut comme valeur source pour ImmutableID), vous devez fournir une revendication correspondante pour les ordinateurs :

* `http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID`

Dans les sections ci-après, vous trouverez des informations concernant :

* Les valeurs requises pour chaque revendication.
* L’aspect d’une définition dans AD FS.

La définition vous permet de vérifier si les valeurs sont présentes ou si vous devez les créer.

> [!NOTE]
> Si vous n’utilisez pas AD FS pour votre serveur de fédération local, suivez les instructions de votre fournisseur afin de créer la configuration appropriée pour l’émission de ces revendications.

### <a name="issue-account-type-claim"></a>Émission de la revendication du type de compte

La revendication `http://schemas.microsoft.com/ws/2012/01/accounttype` doit contenir la valeur **DJ**, qui identifie l’appareil en tant qu’ordinateur joint à un domaine. Dans AD FS, vous pouvez ajouter une règle de transformation d’émission ressemblant à ceci :

   ```
   @RuleName = "Issue account type for domain-joined computers"
   c:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid",
      Value =~ "-515$",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   => issue(
      Type = "http://schemas.microsoft.com/ws/2012/01/accounttype",
      Value = "DJ"
   );
   ```

### <a name="issue-objectguid-of-the-computer-account-on-premises"></a>Émission de la valeur ObjectGUID du compte d’ordinateur local

La revendication `http://schemas.microsoft.com/identity/claims/onpremobjectguid` doit contenir la valeur **objectGUID** du compte d’ordinateur local. Dans AD FS, vous pouvez ajouter une règle de transformation d’émission ressemblant à ceci :

   ```
   @RuleName = "Issue object GUID for domain-joined computers"
   c1:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid",
      Value =~ "-515$", 
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   &&
   c2:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   => issue(
      store = "Active Directory",
      types = ("http://schemas.microsoft.com/identity/claims/onpremobjectguid"),
      query = ";objectguid;{0}",
      param = c2.Value
   );
   ```

### <a name="issue-objectsid-of-the-computer-account-on-premises"></a>Émission de la valeur objectSID du compte d’ordinateur local

La revendication `http://schemas.microsoft.com/ws/2008/06/identity/claims/primarysid` doit contenir la valeur **objectSid** du compte d’ordinateur local. Dans AD FS, vous pouvez ajouter une règle de transformation d’émission ressemblant à ceci :

   ```
   @RuleName = "Issue objectSID for domain-joined computers"
   c1:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid",
      Value =~ "-515$",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   &&
   c2:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/primarysid",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   => issue(claim = c2);
   ```

### <a name="issue-issuerid-for-the-computer-when-multiple-verified-domain-names-are-in-azure-ad"></a>Émission de la valeur issuerID pour l’ordinateur s’il existe plusieurs noms de domaine vérifiés dans Azure AD

La revendication `http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid` doit contenir l’URI (Uniform Resource Identifier) des noms de domaine vérifiés qui se connectent avec le service de fédération local (AD FS ou partenaire) émettant le jeton. Dans AD FS, vous pouvez ajouter des règles de transformation d’émission qui ressemblent aux suivantes dans l’ordre indiqué, après les règles précédentes. Il doit exister une règle régissant l’émission explicite de la règle pour les utilisateurs. Dans les règles suivantes, une première règle qui identifie une authentification de l’utilisateur ou de l’ordinateur est ajoutée.

   ```
   @RuleName = "Issue account type with the value User when its not a computer"
   NOT EXISTS(
   [
      Type == "http://schemas.microsoft.com/ws/2012/01/accounttype",
      Value == "DJ"
   ]
   )
   => add(
      Type = "http://schemas.microsoft.com/ws/2012/01/accounttype",
      Value = "User"
   );

   @RuleName = "Capture UPN when AccountType is User and issue the IssuerID"
   c1:[
      Type == "http://schemas.xmlsoap.org/claims/UPN"
   ]
   &&
   c2:[
      Type == "http://schemas.microsoft.com/ws/2012/01/accounttype",
      Value == "User"
   ]
   => issue(
      Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid",
      Value = regexreplace(
      c1.Value,
      ".+@(?<domain>.+)",
      "http://${domain}/adfs/services/trust/"
      )
   );

   @RuleName = "Issue issuerID for domain-joined computers"
   c:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid",
      Value =~ "-515$",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   => issue(
      Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid",
      Value = "http://<verified-domain-name>/adfs/services/trust/"
   );
   ```

Dans la revendication précédente, `<verified-domain-name>` est un espace réservé. Remplacez-le par un de vos noms de domaine vérifiés dans Azure AD. Par exemple, utilisez `Value = "http://contoso.com/adfs/services/trust/"`.

Pour plus d’informations sur les noms de domaine vérifiés, consultez [Ajouter un nom de domaine personnalisé à Azure Active Directory](../fundamentals/add-custom-domain.md).  

Pour obtenir une liste de vos domaines d’entreprise vérifiés, vous pouvez utiliser l’applet de commande [Get-MsolDomain](/powershell/module/msonline/get-msoldomain).

![Liste des domaines d’entreprise](./media/hybrid-azuread-join-manual/01.png)

### <a name="issue-immutableid-for-the-computer-when-one-for-users-exists-for-example-using-ms-ds-consistencyguid-as-the-source-for-immutableid"></a>Émettre un ImmutableID pour l’ordinateur quand il en existe un pour les utilisateurs (par exemple, en utilisant mS-DS-ConsistencyGuid comme source pour ImmutableID)

La revendication `http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID` doit contenir une valeur valide pour les ordinateurs. Dans AD FS, vous pouvez créer une règle de transformation d’émission comme suit :

   ```
   @RuleName = "Issue ImmutableID for computers"
   c1:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid",
      Value =~ "-515$",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   &&
   c2:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   => issue(
      store = "Active Directory",
      types = ("http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID"),
      query = ";objectguid;{0}",
      param = c2.Value
   );
   ```

### <a name="helper-script-to-create-the-ad-fs-issuance-transform-rules"></a>Script d’assistance pour la création des règles de transformation d’émission AD FS

Le script ci-après vous aide à créer les règles de transformation d’émission décrites précédemment.

   ```
   $multipleVerifiedDomainNames = $false
   $immutableIDAlreadyIssuedforUsers = $false
   $oneOfVerifiedDomainNames = 'example.com'   # Replace example.com with one of your verified domains

   $rule1 = '@RuleName = "Issue account type for domain-joined computers"
   c:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid",
      Value =~ "-515$",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   => issue(
      Type = "http://schemas.microsoft.com/ws/2012/01/accounttype",
      Value = "DJ"
   );'

   $rule2 = '@RuleName = "Issue object GUID for domain-joined computers"
   c1:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid",
      Value =~ "-515$",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   &&
   c2:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   => issue(
      store = "Active Directory",
      types = ("http://schemas.microsoft.com/identity/claims/onpremobjectguid"),
      query = ";objectguid;{0}",
      param = c2.Value
   );'

   $rule3 = '@RuleName = "Issue objectSID for domain-joined computers"
   c1:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid",
      Value =~ "-515$",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   &&
   c2:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/primarysid",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   => issue(claim = c2);'

   $rule4 = ''
   if ($multipleVerifiedDomainNames -eq $true) {
   $rule4 = '@RuleName = "Issue account type with the value User when it is not a computer"
   NOT EXISTS(
   [
      Type == "http://schemas.microsoft.com/ws/2012/01/accounttype",
      Value == "DJ"
   ]
   )
   => add(
      Type = "http://schemas.microsoft.com/ws/2012/01/accounttype",
      Value = "User"
   );

   @RuleName = "Capture UPN when AccountType is User and issue the IssuerID"
   c1:[
      Type == "http://schemas.xmlsoap.org/claims/UPN"
   ]
   &&
   c2:[
      Type == "http://schemas.microsoft.com/ws/2012/01/accounttype",
      Value == "User"
   ]
   => issue(
      Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid",
      Value = regexreplace(
      c1.Value,
      ".+@(?<domain>.+)",
      "http://${domain}/adfs/services/trust/"
      )
   );

   @RuleName = "Issue issuerID for domain-joined computers"
   c:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid",
      Value =~ "-515$",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   => issue(
      Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid",
      Value = "http://' + $oneOfVerifiedDomainNames + '/adfs/services/trust/"
   );'
   }

   $rule5 = ''
   if ($immutableIDAlreadyIssuedforUsers -eq $true) {
   $rule5 = '@RuleName = "Issue ImmutableID for computers"
   c1:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid",
      Value =~ "-515$",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   &&
   c2:[
      Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname",
      Issuer =~ "^(AD AUTHORITY|SELF AUTHORITY|LOCAL AUTHORITY)$"
   ]
   => issue(
      store = "Active Directory",
      types = ("http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID"),
      query = ";objectguid;{0}",
      param = c2.Value
   );'
   }

   $existingRules = (Get-ADFSRelyingPartyTrust -Identifier urn:federation:MicrosoftOnline).IssuanceTransformRules 

   $updatedRules = $existingRules + $rule1 + $rule2 + $rule3 + $rule4 + $rule5

   $crSet = New-ADFSClaimRuleSet -ClaimRule $updatedRules

   Set-AdfsRelyingPartyTrust -TargetIdentifier urn:federation:MicrosoftOnline -IssuanceTransformRules $crSet.ClaimRulesString
   ```

#### <a name="remarks"></a>Notes

* Ce script ajoute les règles aux règles existantes. N’exécutez pas le script à deux reprises, car l’ensemble de règles serait alors ajouté deux fois. Avant de réexécuter le script, assurez-vous qu’il n’existe aucune règle correspondante pour ces revendications (sous les conditions associées).
* Si vous disposez de plusieurs noms de domaine vérifiés (comme indiqué dans le portail Azure AD ou par le biais de l’applet de commande **Get-MsolDomain**), définissez l’élément **$multipleVerifiedDomainNames** du script sur la valeur **$true**. Veillez également à supprimer toute revendication **issuerid** existante pouvant avoir été créée par Azure AD Connect ou par d’autres moyens. Voici un exemple de cette règle :

   ```
   c:[Type == "http://schemas.xmlsoap.org/claims/UPN"]
   => issue(Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/issuerid", Value = regexreplace(c.Value, ".+@(?<domain>.+)",  "http://${domain}/adfs/services/trust/")); 
   ```

Si vous avez déjà émis une revendication **ImmutableID** pour les comptes d’utilisateurs, définissez l’élément **$immutableIDAlreadyIssuedforUsers** du script sur la valeur **$true**.

## <a name="enable-windows-down-level-devices"></a>Activer des appareils Windows de bas niveau

Si certains de vos appareils joints à un domaine sont des appareils Windows de bas niveau, vous devez :

* Définir une stratégie dans Azure AD pour permettre aux utilisateurs d’inscrire des appareils
* Configurer votre service de fédération local pour l’émission de revendications afin de prendre en charge l’authentification Windows intégrée (IWA) pour l’inscription des appareils
* Ajouter le point de terminaison d’authentification d’appareil Azure AD aux zones intranet local afin d’éviter les invites de certificat lors de l’authentification des appareils
* Contrôler des appareils Windows de bas niveau

### <a name="set-a-policy-in-azure-ad-to-enable-users-to-register-devices"></a>Définir une stratégie dans Azure AD pour permettre aux utilisateurs d’inscrire des appareils

Pour inscrire des appareils Windows de bas niveau, assurez-vous que le paramètre permettant aux utilisateurs d’inscrire des appareils dans Azure AD est activé. Dans le portail Azure, vous pouvez trouver ce paramètre sous **Azure Active Directory** > **Utilisateurs et groupes** > **Paramètres de l’appareil**.

La stratégie suivante doit être définie sur **All** (Tout) : **Les utilisateurs peuvent inscrire leurs appareils sur Azure AD**.

![Bouton Tous permettant aux utilisateurs d’inscrire des appareils](./media/hybrid-azuread-join-manual/23.png)

### <a name="configure-the-on-premises-federation-service"></a>Configurer le service de fédération local

Votre service de fédération local doit prendre en charge l’émission des revendications **authenticationmethod** et **wiaormultiauthn** quand il reçoit une requête d’authentification auprès de la partie de confiance Azure AD qui contient un paramètre resource_params avec la valeur encodée suivante :

   ```
   eyJQcm9wZXJ0aWVzIjpbeyJLZXkiOiJhY3IiLCJWYWx1ZSI6IndpYW9ybXVsdGlhdXRobiJ9XX0

   which decoded is {"Properties":[{"Key":"acr","Value":"wiaormultiauthn"}]}
   ```

Quand une telle demande est reçue, le service de fédération local doit authentifier l’utilisateur à l’aide de l’authentification Windows intégrée. Quand l’authentification réussit, le service de fédération doit émettre les deux revendications suivantes :

   `http://schemas.microsoft.com/ws/2008/06/identity/authenticationmethod/windows` `http://schemas.microsoft.com/claims/wiaormultiauthn`

Dans AD FS, vous devez ajouter une règle de transformation d’émission qui est transmise directement par le biais de la méthode d’authentification. Pour ajouter cette règle :

1. Dans la console de gestion d’AD FS, accédez à **AD FS** > **Relations d’approbation** > **Approbations de partie de confiance**.
1. Cliquez avec le bouton droit sur l’objet d’approbation de partie de confiance de la plateforme d’identité Microsoft Office 365 et sélectionnez **Modifier les règles de revendication**.
1. Sous l’onglet **Règles de transformation d’émission**, sélectionnez **Ajouter une règle**.
1. Sélectionnez **Envoyer les revendications en utilisant une règle personnalisée** dans la liste de modèles **Règle de revendication**.
1. Sélectionnez **Suivant**.
1. Dans la zone **Nom de la règle de revendication**, tapez **Règle de revendication de méthode d’authentification**.
1. Dans la zone **Règle de revendication**, entrez la règle suivante :

   `c:[Type == "http://schemas.microsoft.com/claims/authnmethodsreferences"] => issue(claim = c);`

1. Sur votre serveur de fédération, entrez la commande PowerShell suivante. Remplacez **\<RPObjectName\>** par le nom de l’objet de partie de confiance de votre objet d’approbation de partie de confiance Azure AD. Cet objet est généralement nommé **plateforme d’identité Microsoft Office 365**.

   `Set-AdfsRelyingPartyTrust -TargetName <RPObjectName> -AllowedAuthenticationClassReferences wiaormultiauthn`

### <a name="add-the-azure-ad-device-authentication-endpoint-to-the-local-intranet-zones"></a>Ajouter le point de terminaison d’authentification de l’appareil Azure AD aux zones intranet local

Pour éviter les invites de certificat quand les utilisateurs des appareils inscrits s’authentifient auprès d’Azure AD, vous pouvez transmettre une stratégie à vos appareils joints à un domaine pour ajouter l’URL ci-après à la zone intranet local dans Internet Explorer :

`https://device.login.microsoftonline.com`

### <a name="control-windows-down-level-devices"></a>Contrôler des appareils Windows de bas niveau

Pour inscrire des appareils Windows de bas niveau, vous devez télécharger et installer un package Windows Installer (.msi) à partir du Centre de téléchargement. Pour plus d’informations, consultez la section [Validation contrôlée de la jonction Azure AD hybride sur les appareils Windows de bas niveau](hybrid-azuread-join-control.md#controlled-validation-of-hybrid-azure-ad-join-on-windows-down-level-devices).

## <a name="verify-joined-devices"></a>Vérifier des appareils joints

Voici trois méthodes pour rechercher et vérifier l’état de l’appareil :

### <a name="locally-on-the-device"></a>Localement sur l’appareil

1. Ouvrez Windows PowerShell.
2. Entrez `dsregcmd /status`.
3. Vérifiez que **AzureAdJoined** et **DomainJoined** ont la valeur **OUI**.
4. Vous pouvez utiliser le **DeviceId** et comparer l’état du service à l’aide du portail Azure ou de PowerShell.

### <a name="using-the-azure-portal"></a>Utilisation du portail Azure

1. Accédez à la page des appareils à l’aide d’un [lien direct](https://portal.azure.com/#blade/Microsoft_AAD_IAM/DevicesMenuBlade/Devices).
2. Pour plus d’informations sur la localisation d’un appareil, consultez [Gérer les identités de l’appareil à l’aide du portail Microsoft Azure](./device-management-azure-portal.md#manage-devices).
3. Si la colonne **Inscrit** indique **En attente**, cela signifie que la jonction Azure AD Hybride n’a pas été effectuée complètement. Dans les environnements fédérés, cela peut se produire uniquement si l’inscription a échoué et qu’AAD Connect est configuré pour synchroniser les appareils.
4. Si la colonne **Inscrit** contient une **date/heure**, cela signifie que la jonction Azure AD Hybride a été effectuée correctement.

### <a name="using-powershell"></a>Utilisation de PowerShell

Vérifiez l’état d’inscription de l’appareil dans votre locataire Azure à l’aide de **[Get-MsolDevice](/powershell/module/msonline/get-msoldevice)** . Cette applet de commande figure dans le [module Azure Active Directory PowerShell](/powershell/azure/active-directory/install-msonlinev1).

Quand vous utilisez l’applet de commande **Get-MSolDevice** pour vérifier les détails du service :

- Un objet dont l’**ID d’appareil** correspond à l’ID défini sur le client Windows doit exister.
- La valeur pour **DeviceTrustType** est **Joint au domaine**. Ce paramètre équivaut à l’état **Joint à une version hybride d’Azure AD** dans la page **Appareils** du portail Azure AD.
- Pour les appareils utilisés dans l’accès conditionnel, la valeur pour **Activé** est **True** et celle pour **DeviceTrustLevel** est **Géré**.

1. Ouvrez Windows PowerShell en tant qu’administrateur.
2. Entrez `Connect-MsolService` pour vous connecter à votre locataire Azure.

#### <a name="count-all-hybrid-azure-ad-joined-devices-excluding-pending-state"></a>Compter tous les appareils joints Azure AD Hybride (sauf ceux qui sont à l’état **En attente**)

```azurepowershell
(Get-MsolDevice -All -IncludeSystemManagedDevices | where {($_.DeviceTrustType -eq 'Domain Joined') -and (([string]($_.AlternativeSecurityIds)).StartsWith("X509:"))}).count
```

#### <a name="count-all-hybrid-azure-ad-joined-devices-with-pending-state"></a>Compter tous les appareils joints Azure AD Hybride qui sont à l’état **En attente**

```azurepowershell
(Get-MsolDevice -All -IncludeSystemManagedDevices | where {($_.DeviceTrustType -eq 'Domain Joined') -and (-not([string]($_.AlternativeSecurityIds)).StartsWith("X509:"))}).count
```

#### <a name="list-all-hybrid-azure-ad-joined-devices"></a>Lister tous les appareils joints Azure AD Hybride

```azurepowershell
Get-MsolDevice -All -IncludeSystemManagedDevices | where {($_.DeviceTrustType -eq 'Domain Joined') -and (([string]($_.AlternativeSecurityIds)).StartsWith("X509:"))}
```

#### <a name="list-all-hybrid-azure-ad-joined-devices-with-pending-state"></a>Lister tous les appareils joints Azure AD Hybride qui sont à l’état **En attente**

```azurepowershell
Get-MsolDevice -All -IncludeSystemManagedDevices | where {($_.DeviceTrustType -eq 'Domain Joined') -and (-not([string]($_.AlternativeSecurityIds)).StartsWith("X509:"))}
```

#### <a name="list-details-of-a-single-device"></a>Lister les détails d’un seul appareil :

1. Entrez `get-msoldevice -deviceId <deviceId>` (il s’agit du **DeviceId** obtenu localement sur l’appareil).
2. Vérifiez que le paramètre **Enabled** est défini sur **True**.

## <a name="troubleshoot-your-implementation"></a>Résoudre les problèmes liés à votre implémentation

Si vous rencontrez des problèmes en réalisant une jointure Azure AD hybride pour des appareils Windows joints à un domaine, consultez :

- [Dépannage des appareils à l’aide de la commande dsregcmd](./troubleshoot-device-dsregcmd.md)
- [Résolution des problèmes liés aux appareils hybrides joints à Azure Active Directory](troubleshoot-hybrid-join-windows-current.md)
- [Dépanner des appareils hybrides de bas niveau joints à Azure Active Directory](troubleshoot-hybrid-join-windows-legacy.md)

## <a name="next-steps"></a>Étapes suivantes

* [Présentation de la gestion des appareils dans Azure Active Directory](overview.md)

<!--Image references-->
[1]: ./media/hybrid-azuread-join-manual/12.png
