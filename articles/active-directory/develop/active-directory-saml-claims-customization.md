---
title: Personnaliser les revendications de jeton SAML pour les applications
titleSuffix: Microsoft identity platform
description: Découvrez comment personnaliser les revendications émises par Plateforme d’identités Microsoft dans le jeton SAML pour les applications d’entreprise.
services: active-directory
author: kenwith
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: how-to
ms.date: 12/09/2020
ms.author: kenwith
ms.reviewer: luleon, paulgarn, jeedes
ms.custom: aaddev
ms.openlocfilehash: 1c88b9f77513021609b99c81ea572c2b5b3d365b
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96936792"
---
# <a name="how-to-customize-claims-issued-in-the-saml-token-for-enterprise-applications"></a>Procédure : personnaliser des revendications émises dans le jeton SAML pour les applications d’entreprise

Actuellement, Plateforme d’identités Microsoft prend en charge l’authentification unique avec la plupart des applications d’entreprise, y compris les applications pré-intégrées dans la galerie d’applications Azure AD et les applications personnalisées. Quand un utilisateur s’authentifie auprès d’une application par l’intermédiaire de Plateforme d’identités Microsoft en utilisant le protocole SAML 2.0, Plateforme d’identités Microsoft envoie un jeton à l’application (via une requête HTTP POST). Après quoi, l’application valide et utilise ce jeton pour connecter l’utilisateur au lieu de lui demander un nom d’utilisateur et un mot de passe. Ces jetons SAML contiennent des informations sur l’utilisateur appelées *revendications*.

Une *revendication* concerne ce qu’un fournisseur d’identité déclare sur un utilisateur dans le jeton qu’il émet sur cet utilisateur. Dans un [jeton SAML](https://en.wikipedia.org/wiki/SAML_2.0), ces données sont généralement contenues dans l’instruction d’attribut SAML. L’ID unique de l’utilisateur est généralement présent dans le SAML Subject, également appelé « identificateur de nom ».

Par défaut, Plateforme d’identités Microsoft émet un jeton SAML à destination de votre application qui contient une revendication `NameIdentifier`, dont la valeur est le nom de l’utilisateur (ou nom d’utilisateur principal) dans Azure AD et qui peut identifier l’utilisateur de façon unique. Le jeton SAML contient également des revendications supplémentaires contenant l’adresse de messagerie de l’utilisateur, son prénom et son nom.

Pour afficher ou modifier les revendications émises dans le jeton SAML à destination de l’application, ouvrez l’application dans le portail Azure. Ensuite, ouvrez la section **Attributs et revendications de l’utilisateur**.

![Ouvrez la section Attributs et revendications de l’utilisateur dans le Portail Microsoft Azure](./media/active-directory-saml-claims-customization/sso-saml-user-attributes-claims.png)

Vous pouvez être amené à modifier les revendications émises dans le jeton SAML pour les deux raisons suivantes :

* La revendication `NameIdentifier` ou NameID doit être différente du nom d’utilisateur (ou du nom d’utilisateur principal) stocké dans Azure AD pour l’application.
* L’application a été écrite pour exiger un ensemble différent d’URI de revendication ou de valeurs de revendication.

## <a name="editing-nameid"></a>Modification de nameID

Pour modifier le NameID (valeur d’identificateur du nom) :

1. Ouvrez la page **Valeur de nom d’identificateur**.
1. Sélectionnez l’attribut ou la transformation à appliquer à l’attribut. Si vous le souhaitez, vous pouvez spécifier le format de la revendication NameID.

   ![Modifier la valeur NameID (identificateur du nom)](./media/active-directory-saml-claims-customization/saml-sso-manage-user-claims.png)

### <a name="nameid-format"></a>Format NameID

Si la requête SAML contient l’élément NameIDPolicy dans un format spécifique, Plateforme d’identités Microsoft respecte le format de la requête.

Si la requête SAML ne contient aucun élément pour NameIDPolicy, alors Plateforme d’identités Microsoft émet la revendication NameID au format que vous spécifiez. Si aucun format n’est spécifié, Plateforme d’identités Microsoft utilise le format source par défaut associé à la source de la revendication sélectionnée.

Dans le menu déroulant **Choisir le format du nom de l’identificateur**, vous pouvez sélectionner l’une des options suivantes.

| Format NameID | Description |
|---------------|-------------|
| **Par défaut** | Plateforme d’identités Microsoft utilise le format source par défaut. |
| **Persistent** | Plateforme d’identités Microsoft utilise Persistent comme format de NameID. |
| **EmailAddress** | Plateforme d’identités Microsoft utilise EmailAddress comme format de NameID. |
| **Unspecified** | Plateforme d’identités Microsoft utilise Unspecified comme format de NameID. |

Un élément NameID temporaire est également pris en charge, mais il n’est pas disponible dans la liste déroulante et ne peut pas être configuré côté Azure. Pour en savoir plus sur l’attribut NameIDPolicy, consultez [Protocole SAML d’authentification unique](single-sign-on-saml-protocol.md).

### <a name="attributes"></a>Attributs

Sélectionnez la source souhaitée pour la revendication `NameIdentifier` (ou NameID). Vous pouvez sélectionner les options suivantes.

| Nom | Description |
|------|-------------|
| E-mail | Adresse e-mail de l’utilisateur |
| userprincipalName | Nom d’utilisateur principal (UPN) de l’utilisateur |
| onpremisessamaccount | Nom de compte SAM qui a été synchronisé à partir d’Azure AD en local |
| objectid | ID d’objet de l’utilisateur dans Azure AD |
| employeeid | ID d’employé de l’utilisateur |
| Extensions d’annuaire | Extensions d’annuaire [ synchronisées à partir d’Active Directory local à l’aide d’Azure AD Connect Sync](../hybrid/how-to-connect-sync-feature-directory-extensions.md) |
| Attributs d’extension 1-15 | Attributs d’extension local utilisés pour étendre le schéma Azure AD |

Pour plus d’informations, consultez [Tableau 3 : Validez les valeurs d’ID par source](active-directory-claims-mapping.md#table-3-valid-id-values-per-source).

Vous pouvez également attribuer une valeur constante (statique) quelconque à toute revendication que vous définissez dans Azure AD. Suivez les étapes ci-dessous pour attribuer une valeur constante :

1. Dans le [Portail Azure](https://portal.azure.com/), dans la section **Attributs et revendications de l’utilisateur**, cliquez sur l’icône **Modifier** pour modifier les revendications.

1. Cliquez sur la revendication requise que vous souhaitez modifier.

1. Entrez la valeur constante sans guillemets dans **Attribut source** conformément à votre organisation, puis cliquez sur **Enregistrer**.

    ![Organisation de la section Attributs et revendications de l’utilisateur dans le Portail Microsoft Azure](./media/active-directory-saml-claims-customization/organization-attribute.png)

1. La valeur constante sera affichée comme ci-dessous.

    ![Modification de la section Attributs et revendications de l’utilisateur dans le Portail Microsoft Azure](./media/active-directory-saml-claims-customization/edit-attributes-claims.png)

### <a name="special-claims---transformations"></a>Revendications spéciales - Transformations

Vous pouvez également utiliser les fonctions de transformation de revendication.

| Fonction | Description |
|----------|-------------|
| **ExtractMailPrefix()** | Supprime le suffixe de domaine de l’adresse e-mail ou du nom d’utilisateur principal. Ainsi, seule la première partie du nom d’utilisateur transmis est extraite (par exemple, « joe_smith » au lieu de joe_smith@contoso.com). |
| **Join()** | Joint un attribut avec un domaine vérifié. Si la valeur d’identificateur utilisateur sélectionné possède un domaine, extrait le nom d’utilisateur pour ajouter le domaine vérifié sélectionné. Par exemple, si vous sélectionnez l’adresse e-mail (joe_smith@contoso.com) comme valeur d’identificateur utilisateur et que vous sélectionnez contoso.onmicrosoft.com comme domaine vérifié, le résultat est joe_smith@contoso.onmicrosoft.com. |
| **ToLower()** | Convertit les caractères de l’attribut sélectionné en minuscules. |
| **ToUpper()** | Convertit les caractères de l’attribut sélectionné en majuscules. |

## <a name="adding-application-specific-claims"></a>Ajout de revendications spécifiques à l’application

Pour ajouter des revendications spécifiques à l’application :

1. Dans **Attributs utilisateur et revendications**, sélectionnez **Ajouter une revendication** pour ouvrir la page **Gérer les revendications des utilisateurs**.
1. Entrez le **nom** des revendications. La valeur n’a pas besoin de suivre strictement un modèle URI, selon la spécification SAML. Si vous avez besoin d’un modèle d’URI, vous pouvez l’indiquer dans le champ **Namespace**.
1. Sélectionnez la **Source** où la revendication va récupérer sa valeur. Vous pouvez sélectionner un attribut utilisateur dans le menu déroulant d’attribut de la source ou appliquer une transformation à l’attribut utilisateur avant de l’émettre en tant que réclamation.

### <a name="claim-transformations"></a>Transformations de revendication

Pour appliquer une transformation à un attribut utilisateur :

1. Dans **Gérer les revendications**, sélectionnez *Transformation* comme source de revendication pour ouvrir la page **Gérer la transformation**.
2. Sélectionnez la fonction dans la liste déroulante de transformation. Selon la fonction sélectionnée, vous devrez fournir des paramètres et une valeur constante à évaluer dans la transformation. Pour plus d’informations sur les fonctions disponibles, reportez-vous au tableau ci-dessous.
3. Pour appliquer plusieurs transformations, cliquez sur **Ajouter une transformation**. Vous pouvez appliquer un maximum de deux transformations à une revendication. Par exemple, vous pouvez d’abord extraire le préfixe e-mail de `user.mail`. Ensuite, mettez la chaîne en majuscules.

   ![Transformation de plusieurs revendications](./media/active-directory-saml-claims-customization/sso-saml-multiple-claims-transformation.png)

Vous pouvez utiliser les fonctions suivantes pour transformer des revendications.

| Fonction | Description |
|----------|-------------|
| **ExtractMailPrefix()** | Supprime le suffixe de domaine de l’adresse e-mail ou du nom d’utilisateur principal. Ainsi, seule la première partie du nom d’utilisateur transmis est extraite (par exemple, « joe_smith » au lieu de joe_smith@contoso.com). |
| **Join()** | Crée une nouvelle valeur en joignant deux attributs. Si vous le souhaitez, vous pouvez utiliser un séparateur entre les deux attributs. Pour la transformation de revendication NameID, la jointure est limitée à un domaine vérifié. Si la valeur d’identificateur utilisateur sélectionné possède un domaine, extrait le nom d’utilisateur pour ajouter le domaine vérifié sélectionné. Par exemple, si vous sélectionnez l’adresse e-mail (joe_smith@contoso.com) comme valeur d’identificateur utilisateur et que vous sélectionnez contoso.onmicrosoft.com comme domaine vérifié, le résultat est joe_smith@contoso.onmicrosoft.com. |
| **ToLowercase()** | Convertit les caractères de l’attribut sélectionné en minuscules. |
| **ToUppercase()** | Convertit les caractères de l’attribut sélectionné en majuscules. |
| **Contains()** | Génère un attribut ou une constante si l’entrée correspond à la valeur spécifiée. Sinon, vous pouvez spécifier une autre sortie s’il n’existe aucune correspondance.<br/>Par exemple, vous pouvez émettre une revendication où la valeur est l’adresse e-mail utilisateur si elle contient le domaine «@contoso.com». Dans ce cas, nous vous conseillons de générer le nom d’utilisateur principal. Pour ce faire, configurez les valeurs suivantes :<br/>*Paramètre 1 (entrée)*  : user.email<br/>*Valeur* : "@contoso.com"<br/>Paramètre 2 (sortie) : user.email<br/>Paramètre 3 (sortie s’il n’existe aucune correspondance) : user.userprincipalname |
| **EndWith()** | Génère un attribut ou une constante si l’entrée se termine par la valeur spécifiée. Sinon, vous pouvez spécifier une autre sortie s’il n’existe aucune correspondance.<br/>Par exemple, vous pouvez émettre une revendication dont la valeur est l’ID d’employé de l’utilisateur, si l’ID d’employé se termine par « 000 ». Dans ce cas, nous vous recommandons d’obtenir un attribut d’extension en sortie. Pour ce faire, configurez les valeurs suivantes :<br/>*Paramètre 1 (entrée)*  : user.employeeid<br/>*Valeur* : "000"<br/>Paramètre 2 (sortie) : user.employeeid<br/>Paramètre 3 (sortie, s’il n’existe aucune correspondance) : user.extensionattribute1 |
| **StartWith()** | Génère un attribut ou une constante si l’entrée commence par la valeur spécifiée. Sinon, vous pouvez spécifier une autre sortie s’il n’existe aucune correspondance.<br/>Par exemple, vous pouvez émettre une revendication dont la valeur est l’ID d’employé de l’utilisateur, si la valeur de Pays/région commence par « US ». Dans ce cas, nous vous recommandons d’obtenir un attribut d’extension en sortie. Pour ce faire, configurez les valeurs suivantes :<br/>*Paramètre 1 (entrée)*  : user.country<br/>*Valeur* : "US"<br/>Paramètre 2 (sortie) : user.employeeid<br/>Paramètre 3 (sortie, s’il n’existe aucune correspondance) : user.extensionattribute1 |
| **Extract() - After matching** | Retourne la sous-chaîne après qu’elle ait atteint la valeur spécifiée.<br/>Par exemple, si la valeur de l’entrée est « Finance_BSimon », la valeur correspondante est « Finance_ » et la sortie de la revendication est « BSimon ». |
| **Extract() - Before matching** | Retourne la sous-chaîne jusqu’à ce qu’elle corresponde à la valeur spécifiée.<br/>Par exemple, si la valeur de l’entrée est « BSimon_US », la valeur correspondante est « _US » et la sortie de la revendication est « BSimon ». |
| **Extract() - Between matching** | Retourne la sous-chaîne jusqu’à ce qu’elle corresponde à la valeur spécifiée.<br/>Par exemple, si la valeur de l’entrée est « Finance_BSimon_US », la première valeur correspondante est « Finance_ », la seconde « _US » et la sortie de la revendication est « BSimon ». |
| **ExtractAlpha() - Prefix** | Retourne la partie alphabétique du préfixe de la chaîne.<br/>Par exemple, si la valeur de l’entrée est « BSimon_123 », elle retourne « BSimon ». |
| **ExtractAlpha() - Suffix** | Retourne la partie alphabétique du suffixe de la chaîne.<br/>Par exemple, si la valeur de l’entrée est « 123_Simon », elle retourne « Simon ». |
| **ExtractNumeric() - Prefix** | Retourne la partie numérique du préfixe de la chaîne.<br/>Par exemple, si la valeur de l’entrée est « 123_BSimon », elle retourne « 123 ». |
| **ExtractNumeric() - Suffix** | Retourne la partie numérique du suffixe de la chaîne.<br/>Par exemple, si la valeur de l’entrée est « BSimon_123 », elle retourne « 123 ». |
| **IfEmpty()** | Génère un attribut ou une constante en sortie si l’entrée est Null ou vide.<br/>Par exemple, si vous souhaitez générer un attribut stocké dans un extensionattribute si l’ID d’employé pour un utilisateur donné est vide. Pour ce faire, configurez les valeurs suivantes :<br/>Paramètre 1 (entrée) : user.employeeid<br/>Paramètre 2 (sortie) : user.extensionattribute1<br/>Paramètre 3 (sortie s’il n’existe aucune correspondance) : user.employeeid |
| **IfNotEmpty()** | Génère un attribut ou une constante en sortie si l’entrée n’est pas Null ou vide.<br/>Par exemple, si vous souhaitez générer un attribut stocké dans un extensionattribute si l’ID d’employé pour un utilisateur donné n’est pas vide. Pour ce faire, configurez les valeurs suivantes :<br/>Paramètre 1 (entrée) : user.employeeid<br/>Paramètre 2 (sortie) : user.extensionattribute1 |

Si vous avez besoin de transformations supplémentaires, soumettez votre idée dans le [Forum de commentaires Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=160599) sous la catégorie *Application SaaS*.

## <a name="emitting-claims-based-on-conditions"></a>Émission de revendications basées sur des conditions

Vous pouvez spécifier la source d’une revendication en fonction du type d’utilisateur et du groupe auquel appartient l’utilisateur. 

Le type d’utilisateur peut être :
- **Quelconque** : Tous les utilisateurs sont autorisés à accéder à l’application.
- **Membres** : Membre natif du locataire
- **Tous les invités** : L’utilisateur est importé depuis une organisation externe avec ou sans Azure AD.
- **Invités AAD** : L’utilisateur invité appartient à une autre organisation utilisant Azure AD.
- **Invités externes** : L’utilisateur invité appartient à une organisation externe qui n’a pas Azure AD.


Un cas où cela est utile : lorsque la source d’une revendication est différente pour un invité et un employé accédant à une application. Vous pouvez spécifier que si l’utilisateur est un employé, NameID provient de user.email, mais si l’utilisateur est un invité, NameID provient de user.extensionattribute1.

Pour ajouter une condition de revendication :

1. Dans **Gérer les revendications**, développez les conditions de la revendication.
2. Sélectionnez le type d’utilisateur.
3. Sélectionnez le ou les groupes auxquels l’utilisateur doit appartenir. Vous pouvez sélectionner jusqu’à 50 groupes uniques sur l’ensemble des revendications pour une application donnée. 
4. Sélectionnez la **Source** où la revendication va récupérer sa valeur. Vous pouvez sélectionner un attribut utilisateur dans le menu déroulant d’attribut de la source ou appliquer une transformation à l’attribut utilisateur avant de l’émettre en tant que réclamation.

L’ordre dans lequel vous ajoutez les conditions est important. Azure AD évalue les conditions de haut en bas pour décider quelle valeur émettre dans la revendication. La dernière valeur qui correspond à l’expression sera émise dans la revendication.

Par exemple, Britta Simon est figure parmi les utilisateurs invités dans le locataire Contoso. Elle appartient à une autre organisation qui utilise également Azure AD. Étant donné la configuration ci-dessous pour l’application Fabrikam, quand Britta tente de se connecter à Fabrikam, Plateforme d’identités Microsoft évalue les conditions comme suit.

Tout d’abord, Plateforme d’identités Microsoft vérifie si le type d’utilisateur de Britta est `All guests`. Dans la mesure où la valeur est True, Plateforme d’identités Microsoft attribue la source de la revendication à `user.extensionattribute1`. Ensuite, Plateforme d’identités Microsoft vérifie si le type d’utilisateur de Britta est `AAD guests`. Puisque la valeur est également True, alors Plateforme d’identités Microsoft attribue la source de la revendication à `user.mail`. Enfin, la revendication est émise avec la valeur `user.mail` pour Britta.

![Configuration conditionnelle des revendications](./media/active-directory-saml-claims-customization/sso-saml-user-conditional-claims.png)

## <a name="next-steps"></a>Étapes suivantes

* [Gestion des applications dans Azure AD](../manage-apps/what-is-application-management.md)
* [Configurer l’authentification unique pour les applications ne faisant pas partie de la galerie d’applications Azure AD](../manage-apps/configure-saml-single-sign-on.md)
* [Résoudre les erreurs d’authentification unique basée sur SAML](../manage-apps/debug-saml-sso-issues.md)
