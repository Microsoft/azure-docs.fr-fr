---
title: Remplissage de UserPrincipalName dans Azure AD
description: Le document suivant décrit la façon dont l’attribut UserPrincipalName est rempli.
author: billmath
ms.subservice: hybrid
ms.author: billmath
ms.date: 06/26/2018
ms.topic: conceptual
ms.workload: identity
ms.service: active-directory
manager: daveba
ms.collection: M365-identity-device-management
ms.openlocfilehash: 58b4bbcac110398ee4ff132b76ce8c4868ee17f8
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91317589"
---
# <a name="azure-ad-userprincipalname-population"></a>Remplissage de UserPrincipalName dans Azure AD

Cet article décrit la façon dont l’attribut UserPrincipalName est rempli dans Azure Active Directory (Azure AD).
La valeur d’attribut UserPrincipalName est le nom d’utilisateur Azure AD des comptes d’utilisateurs.

## <a name="upn-terminology"></a>Terminologie UPN
Cet article emploie la terminologie suivante :

|Terme|Description|
|-----|-----|
|Domaine initial|Domaine par défaut (onmicrosoft.com) dans le locataire Azure AD. Par exemple, contoso.onmicrosoft.com.|
|Adresse de routage des e-mails en ligne Microsoft (MOERA)|Azure AD calcule l’adresse MOERA à partir de l’attribut Azure AD MailNickName et du domaine initial Azure AD sous la forme &lt;MailNickName&gt;&#64;&lt;domaine initial&gt;.|
|Attribut mailNickName local|Attribut dans Active Directory, dont la valeur représente l’alias d’un utilisateur dans une organisation Exchange.|
|Attribut mail local|Attribut dans Active Directory, dont la valeur représente l’adresse e-mail d’un utilisateur.|
|Adresse SMTP principale|Adresse e-mail principale d’un objet destinataire Exchange. Par exemple, SMTP:utilisateur\@contoso.com.|
|ID de connexion de substitution|Attribut local autre que UserPrincipalName, par exemple l’attribut mail, utilisé pour la connexion.|

## <a name="what-is-userprincipalname"></a>À quoi correspond UserPrincipalName ?
L’attribut UserPrincipalName est un nom de connexion au format Internet d’un utilisateur selon la norme Internet [RFC 822](https://www.ietf.org/rfc/rfc0822.txt). 

### <a name="upn-format"></a>Format UPN
Un UPN se compose d’un préfixe UPN (nom de compte d’utilisateur) et d’un suffixe UPN (nom de domaine DNS). Le préfixe et le suffixe sont liés par le symbole « \@ ». Par exemple, « utilisateur\@exemple.com ». Un UPN doit être unique parmi tous les objets principaux de sécurité d’une forêt de répertoires. 

## <a name="upn-in-azure-ad"></a>UPN dans Azure AD 
L’UPN est utilisé par Azure AD pour permettre aux utilisateurs de se connecter.  L’UPN utilisable par un utilisateur est différent selon que le domaine a été vérifié ou non.  Si c’est le cas, un utilisateur présentant ce suffixe pourra se connecter à Azure AD.  

L’attribut est synchronisé par Azure AD Connect.  Lors de l’installation, vous pouvez voir les domaines qui ont été vérifiés et ceux qui ne l’ont pas été.

   ![Domaines non vérifiés](./media/plan-connect-userprincipalname/unverifieddomain.png) 

## <a name="alternate-login-id"></a>ID de connexion de substitution
Dans certains environnements, les utilisateurs finaux connaissent uniquement leur adresse e-mail, et pas leur nom d’utilisateur principal.  L’utilisation de l’adresse e-mail peut être due à une stratégie d’entreprise ou à une dépendance d’application métier locale.

Un ID de connexion de substitution permet de configurer une expérience de connexion où les utilisateurs peuvent se connecter avec un attribut autre que leur UPN, comme leur adresse e-mail.

Aucune configuration supplémentaire n’est nécessaire pour activer l’ID de connexion de remplacement avec Azure AD dès lors qu’Azure AD Connect est utilisé. L’Assistant permet de configurer directement l’ID de remplacement. Consultez la configuration de connexion Azure AD de vos utilisateurs sous la section Synchronisation. Sous la liste déroulante **Nom d’utilisateur principal**, sélectionnez l’attribut ID de connexion de remplacement.

![Capture d’écran mettant en évidence la liste Nom d’utilisateur principal où vous sélectionnez l’attribut ID de connexion de substitution.](./media/plan-connect-userprincipalname/altloginid.png)  

Pour plus d’informations, consultez les sections [Configurer un ID de connexion de remplacement](/windows-server/identity/ad-fs/operations/configuring-alternate-login-id) et [Configuration de connexion Azure AD](how-to-connect-install-custom.md#azure-ad-sign-in-configuration).

## <a name="non-verified-upn-suffix"></a>Suffixe UPN non vérifié
Si le suffixe d’ID de connexion de remplacement/attribut UserPrincipalName local n’est pas vérifié par le locataire Azure AD, la valeur d’attribut UserPrincipalName Azure AD est définie sur l’adresse MOERA. Azure AD calcule l’adresse MOERA à partir de l’attribut Azure AD MailNickName et du domaine initial Azure AD sous la forme &lt;MailNickName&gt;&#64;&lt;domaine initial&gt;.

## <a name="verified-upn-suffix"></a>Suffixe UPN vérifié
Si le suffixe d’ID de connexion de remplacement/attribut UserPrincipalName local est vérifié par le locataire Azure AD, la valeur d’attribut UserPrincipalName Azure AD sera la même que la valeur d’ID de connexion de remplacement/attribut UserPrincipalName local.

## <a name="azure-ad-mailnickname-attribute-value-calculation"></a>Calcul de la valeur d’attribut MailNickName Azure AD
La valeur d’attribut UserPrincipalName Azure AD pouvant être définie pour l’adresse MOERA, il est important de comprendre la façon dont est calculée la valeur d’attribut MailNickName Azure AD, qui correspond au préfixe MOERA.

Lorsqu’un objet utilisateur est synchronisé avec un locataire Azure AD pour la première fois, Azure AD vérifie les éléments suivants dans l’ordre indiqué et définit la valeur d’attribut MailNickName sur le premier trouvé :

- Attribut mailNickName local
- Préfixe d’adresse SMTP principale
- Préfixe d’attribut mail local
- Préfixe d’ID de connexion de remplacement/attribut UserPrincipalName local
- Préfixe d’adresse SMTP secondaire

Lorsque les mises à jour d’un objet utilisateur sont synchronisées avec le locataire Azure AD, Azure AD ne met à jour la valeur d’attribut MailNickName qu’en cas de mise à jour de la valeur d’attribut mailNickName locale.

>[!IMPORTANT]
>Azure AD ne recalcule la valeur d’attribut UserPrincipalName que dans le cas où une mise à jour de la valeur d’ID de connexion de remplacement/attribut UserPrincipalName locale est synchronisée avec le client Azure AD. 
>
>Chaque fois qu’Azure AD recalcule l’attribut UserPrincipalName, il recalcule également l’adresse MOERA. 

## <a name="upn-scenarios"></a>Scénarios UPN
Voici quelques exemples de scénarios, ainsi que la façon dont l’UPN est calculé pour chacun d’entre eux.

### <a name="scenario-1-non-verified-upn-suffix--initial-synchronization"></a>Scénario 1 : Suffixe UPN non vérifié – synchronisation initiale

![Scénario 1](./media/plan-connect-userprincipalname/example1.png)

Objet utilisateur local :
- mailNickName : &lt;non défini&gt;
- proxyAddresses : {SMTP:us1@contoso.com}
- mail : us2@contoso.com
- UserPrincipalName : us3@contoso.com

Synchroniser l’objet utilisateur avec le locataire Azure AD la première fois.
- Configurez l’attribut MailNickName d’Azure AD sur le préfixe de l’adresse SMTP principale.
- Définir l’adresse MOERA sur &lt;MailNickName&gt;&#64;&lt;domaine initial&gt;.
- Définir l’attribut UserPrincipalName Azure AD sur l’adresse MOERA.

Objet utilisateur du locataire Azure AD :
- MailNickName : us1           
- UserPrincipalName : us1@contoso.onmicrosoft.com


### <a name="scenario-2-non-verified-upn-suffix--set-on-premises-mailnickname-attribute"></a>Scénario 2 : Suffixe UPN non vérifié – définition de l'attribut mailNickName local

![Scénario 2](./media/plan-connect-userprincipalname/example2.png)

Objet utilisateur local :
- mailNickName : us4
- proxyAddresses : {SMTP:us1@contoso.com}
- mail : us2@contoso.com
- UserPrincipalName : us3@contoso.com

Synchroniser la mise à jour de l’attribut mailNickName local avec le locataire Azure AD.
- Mettre à jour l’attribut Azure AD MailNickName Azure AD avec l’attribut mailNickName local.
- Étant donné l’absence de mise à jour de l’attribut UserPrincipalName local, l’attribut UserPrincipalName Azure AD reste inchangé.

Objet utilisateur du locataire Azure AD :
- MailNickName : us4
- UserPrincipalName : us1@contoso.onmicrosoft.com

### <a name="scenario-3-non-verified-upn-suffix--update-on-premises-userprincipalname-attribute"></a>Scénario 3 : Suffixe UPN non vérifié – mise à jour de l'attribut UserPrincipalName local

![Scénario 3](./media/plan-connect-userprincipalname/example3.png)

Objet utilisateur local :
- mailNickName : us4
- proxyAddresses : {SMTP:us1@contoso.com}
- mail : us2@contoso.com
- UserPrincipalName : us5@contoso.com

Synchroniser la mise à jour de l’attribut UserPrincipalName local avec le locataire Azure AD.
- La mise à jour de l’attribut UserPrincipalName local déclenche le recalcul de l’adresse MOERA et de l’attribut UserPrincipalName Azure AD.
- Définir l’adresse MOERA sur &lt;MailNickName&gt;&#64;&lt;domaine initial&gt;.
- Définir l’attribut UserPrincipalName Azure AD sur l’adresse MOERA.

Objet utilisateur du locataire Azure AD :
- MailNickName : us4
- UserPrincipalName : us4@contoso.onmicrosoft.com

### <a name="scenario-4-non-verified-upn-suffix--update-primary-smtp-address-and-on-premises-mail-attribute"></a>Scénario 4 : Suffixe UPN non vérifié – mise à jour de l'adresse SMTP principale et de l'attribut mail local

![Scénario 4](./media/plan-connect-userprincipalname/example4.png)

Objet utilisateur local :
- mailNickName : us4
- proxyAddresses : {SMTP:us6@contoso.com}
- mail : us7@contoso.com
- UserPrincipalName : us5@contoso.com

Synchroniser la mise à jour de l’attribut mail local et de l’adresse SMTP principale avec le locataire Azure AD
- Après la synchronisation initiale de l’objet utilisateur, les mises à jour de l’attribut mail local et de l’adresse SMTP principale n’affecteront ni l’attribut MailNickName, ni l’attribut UserPrincipalName Azure AD.

Objet utilisateur du locataire Azure AD :
- MailNickName : us4
- UserPrincipalName : us4@contoso.onmicrosoft.com

### <a name="scenario-5-verified-upn-suffix--update-on-premises-userprincipalname-attribute-suffix"></a>Scénario 5 : Suffixe UPN vérifié – mise à jour du suffixe d'attribut UserPrincipalName local

![Scénario 5](./media/plan-connect-userprincipalname/example5.png)

Objet utilisateur local :
- mailNickName : us4
- proxyAddresses : {SMTP:us6@contoso.com}
- mail : us7@contoso.com
- UserPrincipalName : us5@verified.contoso.com

Synchroniser la mise à jour de l’attribut UserPrincipalName local avec le locataire Azure AD.
- La mise à jour de l’attribut UserPrincipalName local déclenche le recalcul de l’attribut UserPrincipalName Azure AD.
- Définir l’attribut UserPrincipalName Azure AD sur l’attribut UserPrincipalName local, car le suffixe UPN est vérifié par le locataire Azure AD.

Objet utilisateur du locataire Azure AD :
- MailNickName : us4     
- UserPrincipalName : us5@verified.contoso.com

## <a name="next-steps"></a>Étapes suivantes
- [Intégrer vos répertoires locaux à Azure Active Directory](whatis-hybrid-identity.md)
- [Installation personnalisée d’Azure AD Connect](how-to-connect-install-custom.md)