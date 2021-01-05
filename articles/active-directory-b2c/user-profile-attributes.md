---
title: Attributs de profil utilisateur dans Azure Active Directory B2C
description: Découvrez les attributs de type de ressource utilisateur pris en charge par le profil utilisateur du répertoire Azure AD B2C Directory. Découvrez les attributs intégrés, les extensions et la façon dont les attributs sont mappés à Microsoft Graph.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 12/07/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 31926653950594b986aca19e2db2877cd655ca24
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97509595"
---
# <a name="user-profile-attributes"></a>Attributs de profil utilisateur

Le profil utilisateur de votre répertoire Azure Active Directory (Azure AD) B2C est fourni avec un ensemble intégré d’attributs, tels que le prénom, le nom, la ville, le code postal et le numéro de téléphone. Vous pouvez étendre le profil utilisateur avec vos propres données d’application sans avoir besoin d’un magasin de données externe. La plupart des attributs qui peuvent être utilisés avec les profils utilisateur Azure AD B2C sont également pris en charge par Microsoft Graph. Cet article décrit les attributs de profil utilisateur Azure AD B2C pris en charge. Il présente également les attributs qui ne sont pas pris en charge par Microsoft Graph, ainsi que les attributs Microsoft Graph qui ne doivent pas être utilisés avec Azure AD B2C.

> [!IMPORTANT]
> Vous ne devez pas utiliser d’attributs intégrés ou d’extension pour stocker des données personnelles sensibles, telles que les informations d’identification de compte, les numéros d’identification de gouvernement, les données de titulaires de carte, les données de compte financier, les informations médicales ou les informations sensibles.

Vous pouvez également intégrer des systèmes externes. Par exemple, vous pouvez utiliser Azure AD B2C pour l’authentification, mais déléguer les données client à une base de données externe de gestion de la relation client (CRM) ou de fidélisation des clients en tant que source faisant autorité. Pour plus d’informations, consultez la solution [Profil distant](https://github.com/azure-ad-b2c/samples/tree/master/policies/remote-profile).

Le tableau ci-dessous répertorie les attributs de [type de ressource utilisateur](/graph/api/resources/user) pris en charge par le profil utilisateur du répertoire Azure AD B2C. Il fournit les informations suivantes sur chaque attribut :

- Nom d’attribut utilisé par Azure AD B2C (suivi du nom Microsoft Graph entre parenthèses, si différent)
- Type de données de l’attribut
- Description de l’attribut
- Si l’attribut est disponible dans le portail Azure
- Si l’attribut peut être utilisé dans un flux d’utilisateur
- Si l’attribut peut être utilisé dans une stratégie personnalisée [profil technique Azure AD](active-directory-technical-profile.md) et dans quelle section (&lt;InputClaims&gt;, &lt;OutputClaims&gt; ou &lt;PersistedClaims&gt;)

|Nom     |Type     |Description|Portail Azure|Flux d’utilisateurs|Stratégie personnalisée|
|---------|---------|----------|------------|----------|-------------|
|accountEnabled  |Boolean|Si le compte d’utilisateur est activé ou désactivé : **true** si le compte est activé, sinon **false**.|Oui|Non|Persistant, Sortie|
|ageGroup        |String|Groupe d’âge de l’utilisateur. Valeurs possibles : null, Undefined, Minor, Adult, NotAdult.|Oui|Non|Persistant, sortie|
|alternativeSecurityId ([Identities](manage-user-accounts-graph-api.md#identities-property))|String|Identité d’utilisateur unique du fournisseur d’identité externe.|Non|Non|Entrée, Persistant, Sortie|
|alternativeSecurityIds ([Identities](manage-user-accounts-graph-api.md#identities-property))|Collection d’ID de sécurité alternatifs|Collection d’identités d’utilisateur provenant de fournisseurs d’identité externes.|Non|Non|Persistant, Sortie|
|city            |String|Localité où se trouve l’utilisateur. Longueur max. : 128.|Oui|Oui|Persistant, Sortie|
|consentProvidedForMinor|String|Indique si le consentement a été fourni pour un mineur. Valeurs autorisées : null, granted, denied, or notRequired.|Oui|Non|Persistant, Sortie|
|country         |String|Pays/région où l’utilisateur se trouve. Exemple : « US » ou « UK ». Longueur max. : 128.|Oui|Oui|Persistant, Sortie|
|createdDateTime|DateTime|Date de création de l’objet utilisateur. Lecture seule.|Non|Non|Persistant, Sortie|
|creationType    |String|Si le compte d’utilisateur a été créé en tant que compte local pour un locataire Azure Active Directory B2C, la valeur est LocalAccount ou nameCoexistence. Lecture seule.|Non|Non|Persistant, Sortie|
|dateOfBirth     |Date|Date de naissance.|Non|Non|Persistant, Sortie|
|department      |String|Nom du service où travaille l’utilisateur. Longueur max. : 64.|Oui|Non|Persistant, Sortie|
|displayName     |String|Nom d’affichage de l’utilisateur. Longueur max. : 256.|Oui|Oui|Persistant, Sortie|
|facsimileTelephoneNumber<sup>1</sup>|String|Numéro de téléphone de l’appareil fax professionnel de l’utilisateur.|Oui|Non|Persistant, Sortie|
|givenName       |String|Prénom de l’utilisateur. Longueur max. : 64.|Oui|Oui|Persistant, Sortie|
|jobTitle        |String|Intitulé du poste de l’utilisateur. Longueur max. : 128.|Oui|Oui|Persistant, Sortie|
|ImmutableID     |String|Identificateur généralement utilisé pour les utilisateurs migrés à partir de Windows Server Active Directory.|Non|Non|Persistant, Sortie|
|legalAgeGroupClassification|String|Classification de tranche d’âge légal. En lecture seule et calculée en fonction des propriétés ageGroup et consentProvidedForMinor. Valeurs autorisées : null, minorWithOutParentalConsent, minorWithParentalConsent, minorNoParentalConsentRequired, notAdult et adult.|Oui|Non|Persistant, Sortie|
|legalCountry<sup>1</sup>  |String|Pays/région à des fins juridiques.|Non|Non|Persistant, Sortie|
|mail            |String|Adresse SMTP de l’utilisateur, par exemple « bob@contoso.com ». Lecture seule.|Non|Non|Persistant, Sortie|
|mailNickName    |String|Alias de messagerie de l’utilisateur. Longueur max. : 64.|Non|Non|Persistant, Sortie|
|mobile (mobilePhone) |String|Numéro de portable principal de l’utilisateur. Longueur max. : 64.|Oui|Non|Persistant, Sortie|
|netId           |String|ID Net.|Non|Non|Persistant, Sortie|
|objectId        |String|Identificateur global unique (GUID) qui est l’identificateur unique de l’utilisateur. Exemple : 12345678-9abc-def0-1234-56789abcde. Lecture seule, non modifiable.|Lecture seule|Oui|Entrée, Persistant, Sortie|
|otherMails      |Collection de chaînes|Liste d’adresses e-mail supplémentaires pour l’utilisateur. Exemple: ["bob@contoso.com", "Robert@fabrikam.com"].|Oui (adresse e-mail secondaire)|Non|Persistant, Sortie|
|mot de passe        |String|Mot de passe du compte local lors de la création de l’utilisateur.|Non|Non|Persistant|
|passwordPolicies     |String|Stratégie du passe du compte. Il s’agit d’une chaîne composée d’un nom de stratégie différent, séparé par une virgule. Par exemple : « DisablePasswordExpiration, DisableStrongPassword ».|Non|Non|Persistant, Sortie|
|physicalDeliveryOfficeName (officeLocation)|String|Emplacement du bureau de l’utilisateur. Longueur max. : 128.|Oui|Non|Persistant, Sortie|
|postalCode      |String|Code postal correspondant à l’adresse postale de l’utilisateur. Le code postal est spécifique au pays ou à la région de l’utilisateur. Aux États-Unis, cet attribut contient le code ZIP. Longueur max. : 40.|Oui|Non|Persistant, Sortie|
|preferredLanguage    |String|Langue préférée pour l’utilisateur. Doit suivre le code ISO 639-1. Par exemple, « en-US ».|Non|Non|Persistant, Sortie|
|refreshTokensValidFromDateTime|DateTime|Les jetons d’actualisation émis avant cet instant ne sont pas valides et les applications reçoivent une erreur lors de l’utilisation d’un jeton d’actualisation non valide pour acquérir un nouveau jeton d’accès. Dans ce cas, l’application doit acquérir un nouveau jeton d’actualisation en effectuant une demande au point de terminaison d’autorisation. Lecture seule.|Non|Non|Output|
|signInNames ([Identities](manage-user-accounts-graph-api.md#identities-property)) |String|Nom de connexion unique de l’utilisateur de compte local de tout type dans le répertoire. Utilisez cet attribut pour obtenir un utilisateur avec une valeur de connexion sans spécifier le type de compte local.|Non|Non|Entrée|
|signInNames.userName ([Identities](manage-user-accounts-graph-api.md#identities-property)) |String|Nom d’utilisateur unique de l’utilisateur du compte local dans le répertoire. Utilisez cet attribut pour créer ou obtenir un utilisateur avec un nom d’utilisateur de connexion spécifique. Si vous spécifiez cela dans PersistedClaims seul pendant l’opération Patch, les autres types de signInNames sont supprimés. Si vous souhaitez ajouter un nouveau type de signInNames, vous devez également conserver les signInNames existants.|Non|Non|Entrée, Persistant, Sortie|
|signInNames.phoneNumber ([Identities](manage-user-accounts-graph-api.md#identities-property)) |String|Numéro de téléphone unique de l’utilisateur du compte local dans le répertoire. Utilisez cet attribut pour créer ou obtenir un utilisateur avec un numéro de téléphone de connexion spécifique. Si vous spécifiez cet attribut dans PersistedClaims seul pendant l’opération Patch, les autres types de signInNames sont supprimés. Si vous souhaitez ajouter un nouveau type de signInNames, vous devez également conserver les signInNames existants.|Non|Non|Entrée, Persistant, Sortie|
|signInNames.emailAddress ([Identities](manage-user-accounts-graph-api.md#identities-property))|String|Adresse e-mail unique de l’utilisateur du compte local dans le répertoire. Utilisez ceci fonction pour créer ou obtenir une adresse e-mail de connexion propre à un utilisateur. Si vous spécifiez cet attribut dans PersistedClaims seul pendant l’opération Patch, les autres types de signInNames sont supprimés. Si vous souhaitez ajouter un nouveau type de signInNames, vous devez également conserver les signInNames existants.|Non|Non|Entrée, Persistant, Sortie|
|state           |String|État ou province dans l’adresse de l’utilisateur. Longueur max. : 128.|Oui|Oui|Persistant, Sortie|
|streetAddress   |String|Adresse postale du lieu de travail de l’utilisateur. Longueur max. : 1024.|Oui|Oui|Persistant, Sortie|
|strongAuthentication AlternativePhoneNumber<sup>1</sup>|String|Numéro de téléphone secondaire de l’utilisateur, utilisé pour l’authentification multifacteur.|Oui|Non|Persistant, Sortie|
|strongAuthenticationEmailAddress<sup>1</sup>|String|Adresse SMTP de l’utilisateur. Exemple : « bob@contoso.com » Cet attribut est utilisé pour la connexion avec la stratégie de nom d’utilisateur, pour stocker l’adresse e-mail de l’utilisateur. L’adresse e-mail est ensuite utilisée dans un flow de réinitialisation de mot de passe.|Oui|Non|Persistant, Sortie|
|strongAuthenticationPhoneNumber<sup>2</sup>|String|Numéro de téléphone principal de l’utilisateur, utilisé pour l’authentification multifacteur.|Oui|Non|Persistant, Sortie|
|surname         |String|Patronyme de l’utilisateur (nom). Longueur max. : 64.|Oui|Oui|Persistant, Sortie|
|telephoneNumber (première entrée de businessPhones)|String|Numéro de téléphone principal du lieu de travail de l’utilisateur.|Oui|Non|Persistant, Sortie|
|userPrincipalName    |String|Nom d’utilisateur principal (UPN) de l’utilisateur. L’UPN est un nom de connexion au format Internet d’un utilisateur selon la norme Internet RFC 822. Le domaine doit être présent dans la collection de domaines vérifiés du locataire. Cette propriété est obligatoire quand un compte est créé. Non modifiable.|Non|Non|Entrée, Persistant, Sortie|
|usageLocation   |String|Requis pour les utilisateurs auxquels des licences seront attribuées en raison des exigences légales sur la vérification de la disponibilité des services dans les pays/régions. N'accepte pas la valeur NULL. Code de pays/région à deux lettres (norme ISO 3166). Exemples : « US », « JP » et « GB ».|Oui|Non|Persistant, Sortie|
|userType        |String|Valeur de chaîne qui permet de classer des types d'utilisateurs dans votre répertoire. La valeur doit être Member. Lecture seule.|Lecture seule|Non|Persistant, Sortie|
|userState (externalUserState)<sup>3</sup>|String|Pour un compte Azure AD B2B uniquement, indique si le statut de l’invitation est PendingAcceptance (En attente d’acceptation) ou Accepted (Acceptée).|Non|Non|Persistant, Sortie|
|userStateChangedOn (externalUserStateChangeDateTime)<sup>2</sup>|DateTime|Affiche l’horodatage de la modification la plus récente de la propriété UserState.|Non|Non|Persistant, Sortie|

<sup>1</sup>Non pris en charge par Microsoft Graph<br><sup>2</sup> Pour plus d’informations, consultez [Attribut de numéro de téléphone MFA](#mfa-phone-number-attribute)<br><sup>3</sup> Ne doit pas être utilisé avec Azure AD B2C

## <a name="mfa-phone-number-attribute"></a>Attribut de numéro de téléphone MFA

Lorsque vous utilisez un téléphone pour l’authentification multifacteur (MFA), le téléphone mobile est utilisé pour vérifier l’identité de l’utilisateur. Pour [ajouter](https://docs.microsoft.com/graph/api/authentication-post-phonemethods) un nouveau numéro de téléphone par programmation, [mettez à jour](https://docs.microsoft.com/graph/api/b2cauthenticationmethodspolicy-update), [obtenez](https://docs.microsoft.com/graph/api/b2cauthenticationmethodspolicy-get) ou [supprimez](https://docs.microsoft.com/graph/api/phoneauthenticationmethod-delete) le numéro de téléphone, et utilisez la [méthode d’authentification par téléphone](https://docs.microsoft.com/graph/api/resources/phoneauthenticationmethod) de l’API MS Graph.

Dans les [stratégies personnalisées](custom-policy-overview.md) d’Azure AD B2C, le numéro de téléphone est disponible via le type de revendication `strongAuthenticationPhoneNumber`.

## <a name="extension-attributes"></a>Attributs d’extension

Vous devez souvent créer vos propres attributs, comme dans les cas suivants :

- Une application côté client a besoin de conserver un attribut tel que **LoyaltyNumber**.
- Un fournisseur d’identité a un identificateur d’utilisateur unique, tel que **uniqueUserGUID**, qui doit être enregistré.
- Un parcours utilisateur personnalisé doit conserver l’état de l’utilisateur, par exemple **migrationStatus**.

Azure AD B2C étend l’ensemble d’attributs stocké dans chaque compte utilisateur. Les attributs d’extension [étendent le schéma](/graph/extensibility-overview#schema-extensions) des objets utilisateur dans le répertoire. Les attributs d’extension ne peuvent être inscrits que pour un objet application, même s’ils peuvent contenir les données d’un utilisateur. L’attribut d’extension est attaché à l’application appelée b2c-extensions-app. Ne modifiez pas cette application, car elle est utilisée par Azure AD B2C pour le stockage des données utilisateurs. Vous trouverez cette application sous les inscriptions d’applications Azure Active Directory.

> [!NOTE]
> - Jusqu’à 100 attributs d’extension peuvent être écrits dans un compte d’utilisateur donné.
> - Si l’application b2c-extensions-app est supprimée, ces attributs d’extension sont supprimés de tous les utilisateurs, ainsi que toutes les données qu’ils contiennent.
> - Si un attribut d’extension est supprimé par l’application, il est supprimé de tous les comptes d’utilisateur et les valeurs sont supprimées.
> - Le nom sous-jacent de l’attribut d’extension est généré au format « Extension_ » + ID d’application + « _ » + nom de l’attribut. Par exemple, si vous créez un attribut d’extension LoyaltyNumber et que l’ID d’application b2c-extensions-app est 831374b3-BD50-41bf-aa54-263ec9e050fc, le nom de l’attribut d’extension sous-jacent est le suivant : extension_831374b3bd5041bfaa54263ec9e050fc_LoyaltyNumber. Vous utilisez le nom sous-jacent lorsque vous exécutez des requêtes API Graph pour créer ou mettre à jour des comptes d’utilisateur.

Les types de données suivants sont pris en charge lors de la définition d’une propriété dans une extension de schéma :

|Type de propriété |Notes  |
|--------------|---------|
|Boolean    | Valeurs possibles : **True** ou **False**. |
|DateTime   | Doit être spécifié au format ISO 8601. Est stocké au format UTC.   |
|Integer    | Valeur 32 bits.               |
|String     | 256 caractères maximum.     |

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur les attributs d’extension :
- [Extensions de schéma](/graph/extensibility-overview#schema-extensions)
- [Définir des attributs personnalisés](user-flow-custom-attributes.md)