---
title: Azure AD Connect - Gestion et personnalisation d’AD FS | Microsoft Docs
description: Gestion d’AD FS avec Azure AD Connect et personnalisation de la connexion de l’utilisateur à AD FS avec Azure AD Connect et PowerShell.
keywords: AD FS, ADFS, AD FS management, AAD Connect, Connect, sign-in, AD FS customization, repair trust, M365, federation, relying party
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 2593b6c6-dc3f-46ef-8e02-a8e2dc4e9fb9
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 07/18/2017
ms.subservice: hybrid
ms.author: billmath
ms.custom: seohack1
ms.collection: M365-identity-device-management
ms.openlocfilehash: cc0c8c40e370579100c562e0289c97e3f5ce4236
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91274110"
---
# <a name="manage-and-customize-active-directory-federation-services-by-using-azure-ad-connect"></a>Gérer et personnaliser Active Directory Federation Services à l’aide d’Azure AD Connect
Cet article décrit comment gérer et personnaliser Active Directory Federation Services (ADFS) à l’aide d’Azure Active Directory (Azure AD) Connect. Il indique également d’autres tâches courantes liées à AD FS que vous devrez peut-être effectuer pour terminer la configuration d’une batterie de serveurs AD FS.

| Rubrique | Sujet traité |
|:--- |:--- |
| **Gérer AD FS** | |
| [Réparation de l’approbation](#repairthetrust) |Réparation de l’approbation de fédération avec Microsoft 365. |
| [Fédérer avec Azure AD à l’aide d’un ID de connexion de substitution](#alternateid) | Configurer la fédération à l’aide d’un ID de connexion de substitution  |
| [Ajout d’un serveur AD FS](#addadfsserver) |Extension d’une batterie de serveurs AD FS à l’aide d’un serveur AD FS supplémentaire. |
| [Ajouter un serveur de proxy d’application web AD FS](#addwapserver) |Extension d’une batterie de serveurs AD FS à l’aide d’un serveur de proxy d’application web. |
| [Ajout d’un domaine fédéré](#addfeddomain) |Ajout d’un domaine fédéré. |
| [Mettre à jour le certificat TLS/SSL](how-to-connect-fed-ssl-update.md)| Mise à jour du certificat TLS/SSL pour une batterie de serveurs AD FS. |
| **Personnaliser AD FS** | |
| [Ajout d’une illustration ou d’un logo de société personnalisé](#customlogo) |Personnalisation de la page de connexion AD FS à l’aide d’une illustration ou d’un logo de société. |
| [Ajout d’une description de connexion](#addsignindescription) |Ajout d’une description de la page de connexion. |
| [Modification des règles de revendication AD FS](#modclaims) |Modification des revendications AD FS pour différents scénarios de fédération. |

## <a name="manage-ad-fs"></a>Gérer AD FS
Vous pouvez effectuer différentes tâches associées à AD FS dans Azure AD Connect avec une intervention minime de l’utilisateur à l’aide de l’Assistant Azure AD Connect. Une fois que vous avez terminé l’installation d’Azure AD Connect à l’aide de l’Assistant, vous pouvez à nouveau exécuter l’Assistant pour effectuer des tâches supplémentaires.

## <a name="repair-the-trust"></a><a name="repairthetrust"></a>Réparation de l’approbation 
Vous pouvez utiliser Azure AD Connect pour vérifier l’état actuel des services AD FS et Azure AD et prendre des mesures appropriées pour réparer l’approbation. Pour réparer l’approbation des services Azure AD et AD FS, procédez comme suit :

1. Sélectionnez **Réparer la confiance AAD et ADFS** dans la liste des tâches disponibles.
   ![Réparer la confiance AAD et ADFS](./media/how-to-connect-fed-management/RepairADTrust1.PNG)

2. Dans la page **Connexion à Azure AD**, saisissez vos informations d’identification d’administrateur global d’Azure AD, puis cliquez sur **Suivant**.
   ![Screenshot that shows the "Connect to Azure AD" page with example credentials entered.](./media/how-to-connect-fed-management/RepairADTrust2.PNG)

3. Dans la page **Informations d’identification d’accès à distance** , indiquez les informations d’identification de l’administrateur de domaine.

   ![Capture d’écran montrant la page « Informations d’identification d’accès à distance » avec des exemples d’informations d’identification entrées.](./media/how-to-connect-fed-management/RepairADTrust3.PNG)

    Lorsque vous cliquez sur **Suivant**, Azure AD Connect vérifie l’intégrité du certificat et affiche les éventuels problèmes.

    ![État des certificats](./media/how-to-connect-fed-management/RepairADTrust4.PNG)

    La page **Prêt à configurer** affiche la liste des actions qui seront effectuées afin de réparer l’approbation.

    ![Capture d’écran montrant la page « Prêt à configurer » avec une liste d’actions.](./media/how-to-connect-fed-management/RepairADTrust5.PNG)

4. Cliquez sur **Installer** pour réparer l’approbation.

> [!NOTE]
> Azure AD Connect peut seulement réparer ou modifier les certificats auto-signés. Azure AD Connect ne permet pas de réparer les certificats tiers.

## <a name="federate-with-azure-ad-using-alternateid"></a><a name="alternateid"></a>Fédération avec Azure AD via AlternateID 
Il est recommandé que le nom d’utilisateur principal (UPN) local et le nom d’utilisateur principal cloud soient identiques. Si l’UPN local utilise un domaine non routable (par ex. Contoso.local) ou ne peut pas être modifié en raison des dépendances d’application locales, nous vous recommandons de configurer un ID de connexion de substitution. Un ID de connexion de substitution permet de configurer une expérience de connexion où les utilisateurs peuvent se connecter avec un attribut autre que leur UPN, comme leur adresse e-mail. Le choix de nom d’utilisateur principal dans Azure AD Connect est par défaut l’attribut userPrincipalName dans Active Directory. Si vous choisissez n’importe quel autre attribut pour le nom d’utilisateur principal et fédérez avec AD FS, Azure AD Connect configurera AD FS pour l’ID de connexion de substitution. Vous trouverez ci-dessous un exemple de choix d’un autre attribut pour le nom d’utilisateur principal :

![Sélection d’un attribut d’ID de substitution](./media/how-to-connect-fed-management/attributeselection.png)

La configuration d’un ID de connexion de substitution pour AD FS comprend deux étapes principales :
1. **Configuration du jeu de revendications d'émission** : les règles de revendication d'émission dans la partie de confiance Azure AD sont modifiées pour utiliser l'attribut UserPrincipalName sélectionné en tant qu'ID de substitution de l'utilisateur.
2. **Activation d'un ID de connexion de substitution dans la configuration d'AD FS** : la configuration d'AD FS est mise à jour afin qu'AD FS puisse rechercher des utilisateurs dans les forêts appropriées à l'aide de l'ID de substitution. Cette configuration est prise en charge pour AD FS sur Windows Server 2012 R2 (avec KB2919355) ou version ultérieure. Si les serveurs AD FS sont sous 2012 R2, Azure AD Connect vérifie la présence de la base de connaissances requise. Si la base de connaissances n’est pas détectée, un avertissement s’affiche après la configuration, comme indiqué ci-dessous :

    ![Avertissement d’absence de base de connaissances sur 2012 R2](./media/how-to-connect-fed-management/kbwarning.png)

    Pour corriger la configuration en cas de bases de connaissances manquantes, installez la mise à jour [KB2919355](https://go.microsoft.com/fwlink/?LinkID=396590) requise, puis réparez l’approbation à l’aide de [Réparer l’approbation AAD et AD FS](#repairthetrust).

> [!NOTE]
> Pour plus d’informations sur ID de substitution et les étapes de configuration manuelle, lisez [Configuration d’un ID de connexion de substitution](/windows-server/identity/ad-fs/operations/configuring-alternate-login-id)

## <a name="add-an-ad-fs-server"></a><a name="addadfsserver"></a>Ajout d’un serveur AD FS 

> [!NOTE]
> Pour ajouter un serveur AD FS, Azure AD Connect requiert le certificat PFX. Par conséquent, vous ne pouvez effectuer cette opération que si vous avez configuré la batterie de serveurs AD FS à l’aide d’Azure AD Connect.

1. Sélectionnez **Déploiement d’un serveur de fédération supplémentaire**, puis cliquez sur **Suivant**.

   ![Serveur de fédération supplémentaire](./media/how-to-connect-fed-management/AddNewADFSServer1.PNG)

2. Dans la page **Connexion à Azure AD**, saisissez vos informations d’identification d’administrateur global d’Azure AD, puis cliquez sur **Suivant**.

   ![Capture d’écran montrant la page « Connexion à Azure AD » avec des exemples d’informations d’identification entrées.](./media/how-to-connect-fed-management/AddNewADFSServer2.PNG)

3. Indiquez les informations d’identification de l’administrateur de domaine.

   ![Informations d’identification de l’administrateur de domaine](./media/how-to-connect-fed-management/AddNewADFSServer3.PNG)

4. Azure AD Connect vous demande le mot de passe du fichier PFX que vous avez fourni lors de la configuration de votre nouvelle batterie de serveurs AD FS avec Azure AD Connect. Cliquez sur **Saisie du mot de passe** pour fournir le mot de passe du fichier PFX.

   ![Capture d’écran montrant la page « Spécifier le certificat SSL » avec la fenêtre « Mot de passe du certificat » ouverte.](./media/how-to-connect-fed-management/AddNewADFSServer4.PNG)

    ![Capture d’écran montrant la page « Spécifier le certificat SSL » après l’entrée d’un mot de passe pour le fichier PFX.](./media/how-to-connect-fed-management/AddNewADFSServer5.PNG)

5. Dans la page **Serveurs AD FS** , entrez le nom ou l’adresse IP du serveur à ajouter à la batterie de serveurs AD FS.

   ![Serveurs AD FS](./media/how-to-connect-fed-management/AddNewADFSServer6.PNG)

6. Cliquez sur **Suivant** et parcourez la page **Configurer** finale. Une fois qu’Azure AD Connect a fini d’ajouter les serveurs à la batterie de serveurs AD FS, vous avez la possibilité de vérifier la connectivité.

   ![Capture d’écran montrant la page « Prêt à configurer » avec une liste d’actions à effectuer après avoir cliqué sur « Installer ».](./media/how-to-connect-fed-management/AddNewADFSServer7.PNG)

    ![Capture d’écran montrant la page « Installation terminée » avec le message « La configuration de l’intranet a été vérifiée avec succès ». ](./media/how-to-connect-fed-management/AddNewADFSServer8.PNG)

## <a name="add-an-ad-fs-wap-server"></a><a name="addwapserver"></a>Ajout d’un serveur WAP AD FS 

> [!NOTE]
> Pour ajouter un serveur de proxy d’application web AD FS, Azure AD Connect requiert le certificat PFX. Par conséquent, vous ne pouvez effectuer cette opération que si vous avez configuré la batterie de serveurs AD FS à l’aide d’Azure AD Connect.

1. Sélectionnez **Déployer le proxy d’application web** dans la liste des tâches disponibles.

   ![Déployer le proxy d’application web](./media/how-to-connect-fed-management/WapServer1.PNG)

2. Indiquez les informations d’identification de l’administrateur global Azure.

   ![Capture d’écran montrant la page « Connexion à Azure AD » avec un exemple de nom d’utilisateur et de mot de passe entré.](./media/how-to-connect-fed-management/wapserver2.PNG)

3. Dans la page **Spécifiez le certificat SSL** , indiquez le mot de passe du fichier PFX que vous avez fourni lors de la configuration de la batterie de serveurs AD FS avec Azure AD Connect.
   ![Mot de passe du certificat](./media/how-to-connect-fed-management/WapServer3.PNG)

    ![Spécifier le certificat TLS/SSL](./media/how-to-connect-fed-management/WapServer4.PNG)

4. Ajoutez le serveur à ajouter en tant que serveur de proxy d’application web. Étant donné que le serveur de proxy d’application web peut être joint ou non au domaine, l’Assistant vous demande les informations d’identification administratives du serveur en cours d’ajout.

   ![Informations d’identification du serveur d’administration](./media/how-to-connect-fed-management/WapServer5.PNG)

5. Dans la page **Informations d’identification de confiance du proxy** , indiquez les informations d’identification de l’administrateur pour configurer l’approbation du proxy et accéder au serveur principal dans la batterie de serveurs AD FS.

   ![Informations d’identification de confiance du proxy](./media/how-to-connect-fed-management/WapServer6.PNG)

6. Dans la page **Prêt à configurer** , l’Assistant affiche la liste des actions qui seront effectuées.

   ![Capture d’écran montrant la page « Prêt à configurer » avec une liste d’actions à effectuer.](./media/how-to-connect-fed-management/WapServer7.PNG)

7. Cliquez sur **Installer** pour terminer la configuration. Une fois la configuration terminée, l’Assistant vous permet de vérifier la connectivité aux serveurs. Cliquez sur **Vérifier** pour vérifier la connectivité.

   ![Installation terminée](./media/how-to-connect-fed-management/WapServer8.PNG)

## <a name="add-a-federated-domain"></a><a name="addfeddomain"></a>Ajout d’un domaine fédéré 

Il est facile d’ajouter un domaine à fédérer avec Azure AD à l’aide d’Azure AD Connect. Azure AD Connect ajoute le domaine de la fédération et modifie les règles de revendication pour identifier correctement l’émetteur lorsque plusieurs domaines sont fédérés avec Azure AD.

1. Pour ajouter un domaine fédéré, sélectionnez la tâche **Ajout d’un domaine Azure AD supplémentaire**.

   ![Domaine Azure AD supplémentaire](./media/how-to-connect-fed-management/AdditionalDomain1.PNG)

2. Dans la page suivante de l’Assistant, indiquez les informations d’identification de l’administrateur global d’Azure AD.

   ![Se connecter à Azure AD](./media/how-to-connect-fed-management/AdditionalDomain2.PNG)

3. Dans la page **Informations d’identification d’accès à distance** , indiquez les informations d’identification de l’administrateur de domaine.

   ![Informations d’identification d’accès à distance](./media/how-to-connect-fed-management/additionaldomain3.PNG)

4. Dans la page suivante, l’Assistant affiche une liste de domaines Azure AD dans lesquels vous pouvez fédérer votre annuaire local. Sélectionnez le domaine dans la liste.

   ![Domaine Azure AD](./media/how-to-connect-fed-management/AdditionalDomain4.PNG)

    Après avoir choisi le domaine, l’Assistant présente des informations utiles concernant les autres actions qu’il va effectuer et l’impact de la configuration. Dans certains cas, si vous sélectionnez un domaine qui n’est pas encore vérifié dans Azure AD, l’Assistant affiche des informations pour vous aider à vérifier le domaine. Pour plus d’informations, consultez [Ajouter un nom de domaine personnalisé à Azure Active Directory](../fundamentals/add-custom-domain.md) .

5. Cliquez sur **Suivant**. La page **Prêt à configurer** affiche la liste des actions qui seront effectuées par Azure AD Connect. Cliquez sur **Installer** pour terminer la configuration.

   ![Prêt à configurer](./media/how-to-connect-fed-management/AdditionalDomain5.PNG)

> [!NOTE]
> Les utilisateurs du domaine fédéré ajouté doivent être synchronisés pour pouvoir se connecter à Azure AD.

## <a name="ad-fs-customization"></a>Personnalisation d’AD FS
Les sections suivantes fournissent des informations concernant certaines tâches courantes que vous devrez peut-être effectuer pour personnaliser votre page de connexion AD FS.

## <a name="add-a-custom-company-logo-or-illustration"></a><a name="customlogo"></a>Ajout d’une illustration ou d’un logo de société personnalisé 
Pour changer le logo de la société affiché sur la page **Connexion**, utilisez l’applet de commande et la syntaxe Windows PowerShell suivantes.

> [!NOTE]
> Les dimensions recommandées pour le logo sont 260 x 35 \@ 96 PPP, avec une taille de fichier maximale de 10 Ko.

```azurepowershell-interactive
Set-AdfsWebTheme -TargetName default -Logo @{path="c:\Contoso\logo.PNG"}
```

> [!NOTE]
> Le paramètre *TargetName* est obligatoire. Le thème par défaut publié avec AD FS est nommé Par défaut.

## <a name="add-a-sign-in-description"></a><a name="addsignindescription"></a>Ajout d’une description de connexion 
Pour ajouter une description de la **page de connexion**, utilisez l’applet de commande et la syntaxe Windows PowerShell suivantes.

```azurepowershell-interactive
Set-AdfsGlobalWebContent -SignInPageDescriptionText "<p>Sign-in to Contoso requires device registration. Click <A href='http://fs1.contoso.com/deviceregistration/'>here</A> for more information.</p>"
```

## <a name="modify-ad-fs-claim-rules"></a><a name="modclaims"></a>Modification des règles de revendication AD FS 
AD FS prend en charge un langage complet, qui permet de créer des règles de revendication personnalisées. Pour plus d’informations, consultez [Rôle du langage de règle de revendication](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dd807118(v=ws.11)).

Les sections suivantes décrivent comment écrire des règles personnalisées pour certains scénarios se rapportant à la fédération des services Azure AD et AD FS.

### <a name="immutable-id-conditional-on-a-value-being-present-in-the-attribute"></a>ID non modifiable à condition que la valeur soit présente dans l’attribut
Azure AD Connect vous permet de spécifier un attribut à utiliser comme ancre source, lorsque les objets sont synchronisés avec Azure AD. Si la valeur de l’attribut personnalisé n’est pas vide, vous souhaiterez peut-être émettre une revendication d’ID non modifiable.

Par exemple, vous pouvez sélectionner **ms-ds-consistencyguid** comme attribut de l’ancre source et émettre **ms-ds-consistencyguid** comme **ImmutableID**, si l’attribut a une valeur. Si l’attribut n’a pas de valeur, émettez l’ID non modifiable **objectGuid** . Vous pouvez construire le jeu de règles de revendication personnalisées, comme indiqué dans la section suivante.

**Règle 1 : attributs de la requête**

```claim-rule-language
c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname"]
=> add(store = "Active Directory", types = ("http://contoso.com/ws/2016/02/identity/claims/objectguid", "http://contoso.com/ws/2016/02/identity/claims/msdsconsistencyguid"), query = "; objectGuid,ms-ds-consistencyguid;{0}", param = c.Value);
```

Dans cette règle, vous interrogez les valeurs de **ms-ds-consistencyguid** et **objectGuid** de l’utilisateur à partir d’Active Directory. Remplacez le nom du magasin par un nom de magasin approprié dans votre déploiement AD FS. Remplacez également le type de revendication par un type approprié à votre fédération, comme défini pour **objectGuid** et **ms-ds-consistencyguid**.

Par ailleurs, utiliser **add** à la place de **issue** évite d’ajouter un problème à la sortie de l’entité et permet d’utiliser les valeurs en tant que valeurs intermédiaires. Vous allez émettre la revendication dans une règle ultérieure, après avoir établi la valeur à utiliser comme ID non modifiable.

**Règle 2 : s'assurer que l'ID ms-ds-consistencyguid existe pour l'utilisateur**

```claim-rule-language
NOT EXISTS([Type == "http://contoso.com/ws/2016/02/identity/claims/msdsconsistencyguid"])
=> add(Type = "urn:anandmsft:tmp/idflag", Value = "useguid");
```

Cette règle définit un indicateur temporaire **idflag** dont la valeur est **useguid** si aucun ID **ms-ds-concistencyguid** n’est renseigné pour l’utilisateur. Il y a une logique à cela : AD FS n’autorise pas les revendications vides. De ce fait, lorsque vous ajoutez des revendications `http://contoso.com/ws/2016/02/identity/claims/objectguid` et `http://contoso.com/ws/2016/02/identity/claims/msdsconsistencyguid` dans la règle 1, vous vous retrouvez avec une revendication **msdsconsistencyguid** uniquement si la valeur est renseignée pour l’utilisateur. Si elle n’est pas indiquée, AD FS voit que sa valeur sera vide et le supprime immédiatement. Tous les objets auront un **objectGuid**. Donc, cette revendication sera toujours là après l’exécution de la règle 1.

**Règle 3 : émettre ms-ds-consistencyguid comme ID non modifiable s'il est présent**

```claim-rule-language
c:[Type == "http://contoso.com/ws/2016/02/identity/claims/msdsconsistencyguid"]
=> issue(Type = "http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID", Value = c.Value);
```

Il s’agit d’un contrôle **Exist** implicite. Si la valeur de la revendication existe, alors émettez-la en tant qu’ID non modifiable. L’exemple précédent utilise le **nameidentifier** de revendication. Vous devez le remplacer par un type de revendication approprié pour l’ID non modifiable dans votre environnement.

**Règle 4 : émettre objectGuid comme ID non modifiable si ms-ds-consistencyGuid n'est pas présent**

```claim-rule-language
c1:[Type == "urn:anandmsft:tmp/idflag", Value =~ "useguid"]
&& c2:[Type == "http://contoso.com/ws/2016/02/identity/claims/objectguid"]
=> issue(Type = "http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID", Value = c2.Value);
```

Dans cette règle, vous vérifiez simplement l’indicateur temporaire **idflag**. Vous décidez s’il faut émettre la revendication en fonction de sa valeur.

> [!NOTE]
> L’ordre de ces règles est important.

### <a name="sso-with-a-subdomain-upn"></a>Authentification unique avec un UPN de sous-domaine

Vous pouvez ajouter plusieurs domaines à fédérer à l’aide d’Azure AD Connect, comme indiqué dans [Ajout d’un domaine fédéré](how-to-connect-fed-management.md#addfeddomain). Azure AD Connect version 1.1.553.0 et ultérieures crée automatiquement la règle de revendication issuerID appropriée. Si vous ne pouvez pas utiliser Azure AD Connect version 1.1.553.0 ou ultérieures, il est recommandé d’utiliser l’outil [Règles de revendication Azure AD RPT](https://aka.ms/aadrptclaimrules) afin de générer et de définir les règles de revendication appropriées pour l’approbation de partie de confiance Azure AD.

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur les [options de connexion de l’utilisateur](plan-connect-user-signin.md).