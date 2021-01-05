---
title: ClaimsSchema  - Azure Active Directory B2C | Microsoft Docs
description: Spécifiez l’élément ClaimsSchema d’une stratégie personnalisée dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 03/05/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 2ff43408cfa6d95dbd5a235a950269c47d57a416
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97654028"
---
# <a name="claimsschema"></a>ClaimsSchema

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

L’élément **ClaimsSchema** définit les types de revendications qui peuvent être référencés dans le cadre de la stratégie. Le schéma de revendications est l’endroit où vous déclarez vos revendications. Une revendication peut être le prénom, le nom de famille, le nom d’affichage, le numéro de téléphone, et bien plus encore. L’élément ClaimsSchema contient une liste d’éléments **ClaimType**. L’élément **ClaimType** contient l’attribut **Id**, qui est le nom de la revendication.

```xml
<BuildingBlocks>
  <ClaimsSchema>
    <ClaimType Id="Id">
      <DisplayName>Surname</DisplayName>
      <DataType>string</DataType>
      <DefaultPartnerClaimTypes>
        <Protocol Name="OAuth2" PartnerClaimType="family_name" />
        <Protocol Name="OpenIdConnect" PartnerClaimType="family_name" />
        <Protocol Name="SAML2" PartnerClaimType="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname" />
      </DefaultPartnerClaimTypes>
      <UserHelpText>Your surname (also known as family name or last name).</UserHelpText>
      <UserInputType>TextBox</UserInputType>
```

## <a name="claimtype"></a>ClaimType

L’élément **ClaimType** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Id | Oui | Identificateur utilisé pour le type de revendication. D’autres éléments peuvent utiliser cet identificateur dans la stratégie. |

L’élément **ClaimType** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| DisplayName | 1:1 | Titre présenté aux utilisateurs sur différents écrans. La valeur peut être [localisée](localization.md). |
| DataType | 1:1 | Type de la revendication. |
| DefaultPartnerClaimTypes | 0:1 | Types de revendications par défaut de partenaire à utiliser pour un protocole spécifié. La valeur peut être substituée dans le **PartnerClaimType** spécifié dans l’élément **InputClaim** ou **OutputClaim**. Utilisez cet élément pour spécifier le nom par défaut d’un protocole.  |
| Mask | 0:1 | Chaîne facultative de caractères de masquage qui peuvent être appliqués lors de l’affichage de la revendication. Par exemple, le numéro téléphone 324-232-4343 peut être masqué comme suit : XXX-XXX-4343. |
| UserHelpText | 0:1 | Description du type de revendication qui peut aider les utilisateurs à comprendre sa raison d’être. La valeur peut être [localisée](localization.md). |
| UserInputType | 0:1 | Type de contrôle d’entrée qui doit être accessible à l’utilisateur lors de l’entrée manuelle des données de revendication pour le type de revendication. Voir les types d’entrées utilisateur définis plus loin dans cette page. |
| AdminHelpText | 0:1 | Description du type de revendication qui peut aider les administrateurs à comprendre sa raison d’être. |
| Restriction | 0:1 | Restrictions de la valeur de cette revendication, par exemple une expression régulière (Regex) ou une liste de valeurs acceptables. La valeur peut être [localisée](localization.md). |
PredicateValidationReference| 0:1 | Référence à un élément **PredicateValidationsInput**. Les éléments **PredicateValidationReference** vous permettent d’effectuer un processus de validation afin de vous assurer que seules des données correctement formées sont entrées. Pour plus d’informations, consultez [Predicates](predicates.md). |



### <a name="datatype"></a>DataType

L’élément **DataType** prend en charge les valeurs suivantes :

| Type | Description |
| ------- | ----------- |
|boolean|Représente une valeur booléenne (`true` ou `false`).|
|Date| Représente un instant, généralement exprimé sous la forme d’une date. La valeur de la date est conforme à la convention ISO 8601.|
|dateTime|Représente un instant, généralement exprimé sous la forme d’une date ou d’une heure. La valeur de la date est conforme à la convention ISO 8601.|
|duration|Représente un intervalle de temps en années, mois, jours, heures, minutes et secondes. Le format est `PnYnMnDTnHnMnS`, où `P` indique une valeur positive, ou `N` pour une valeur négative. `nY` correspond au nombre d’années suivi du littéral `Y`. `nMo` correspond au nombre de mois suivi du littéral `Mo`. `nD` correspond au nombre de jours suivi du littéral `D`. Exemples : `P21Y` représente 21 ans. `P1Y2Mo` représente un an et deux mois. `P1Y2Mo5D` représente un an, deux mois et cinq jours.  `P1Y2M5DT8H5M620S` représente un an, deux mois, cinq jours, huit heures, cinq minutes et vingt secondes.  |
|phoneNumber|Représente un numéro de téléphone. |
|int| Représente un nombre compris entre -2,147,483,648 et 2,147,483,647.|
|long| Représente un nombre compris entre -9,223,372,036,854,775,808 et 9,223,372,036,854,775,807. |
|string| Représente du texte sous la forme d’une séquence d’unités de code UTF-16.|
|stringCollection|Représente une collection de `string`.|
|userIdentity| Représente une identité d’utilisateur.|
|userIdentityCollection|Représente une collection de `userIdentity`.|

### <a name="defaultpartnerclaimtypes"></a>DefaultPartnerClaimTypes

**DefaultPartnerClaimTypes** peut contenir l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| Protocol | 1:n | Liste de protocoles avec leur nom de type de revendication de partenaire par défaut. |

L’élément **Protocol** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Nom | Oui | Nom d’un protocole valide pris en charge par Azure AD B2C. Les valeurs possibles sont les suivantes :  OAuth1, OAuth2, SAML2, OpenIdConnect. |
| PartnerClaimType | Oui | Nom du type de revendication à utiliser. |

Dans l’exemple suivant, quand l’Infrastructure d’expérience d’identité interagit avec un fournisseur d’identité SAML2 ou une application de confiance, la revendication **surname** est mappée à `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname`. Avec OpenIdConnect et OAuth2, la revendication est mappée à `family_name`.

```xml
<ClaimType Id="surname">
  <DisplayName>Surname</DisplayName>
  <DataType>string</DataType>
  <DefaultPartnerClaimTypes>
    <Protocol Name="OAuth2" PartnerClaimType="family_name" />
    <Protocol Name="OpenIdConnect" PartnerClaimType="family_name" />
    <Protocol Name="SAML2" PartnerClaimType="http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname" />
  </DefaultPartnerClaimTypes>
</ClaimType>
```

Ainsi, le jeton JWT émis par Azure AD B2C émet le `family_name` au lieu du nom de ClaimType **surname**.

```json
{
  "sub": "6fbbd70d-262b-4b50-804c-257ae1706ef2",
  "auth_time": 1535013501,
  "given_name": "David",
  "family_name": "Williams",
  "name": "David Williams",
}
```

### <a name="mask"></a>Mask

L’élément **Mask** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| `Type` | Oui | Type du masque de revendication. Valeurs possibles : `Simple` ou `Regex`. La valeur `Simple` indique qu’un masque de texte simple est appliqué à la partie gauche d’une revendication de chaîne. La valeur `Regex` indique qu’une expression régulière est appliquée à la revendication de chaîne dans son ensemble.  Si la valeur `Regex` est spécifiée, un attribut facultatif doit également être défini avec l’expression régulière à utiliser. |
| `Regex` | Non | Si **`Type`** a la valeur `Regex`, spécifiez l’expression régulière à utiliser.

L’exemple suivant configure une revendication **PhoneNumber** avec le masque `Simple` :

```xml
<ClaimType Id="PhoneNumber">
  <DisplayName>Phone Number</DisplayName>
  <DataType>string</DataType>
  <Mask Type="Simple">XXX-XXX-</Mask>
  <UserHelpText>Your telephone number.</UserHelpText>
</ClaimType>
```

L’Infrastructure d’expérience d’identité affiche le numéro de téléphone tout en masquant les six premiers chiffres :

![Revendication de numéro de téléphone affichée dans le navigateur avec les six premiers chiffres masqués par des X](./media/claimsschema/mask.png)

L’exemple suivant configure une revendication **AlternateEmail** avec le masque `Regex` :

```xml
<ClaimType Id="AlternateEmail">
  <DisplayName>Please verify the secondary email linked to your account</DisplayName>
  <DataType>string</DataType>
  <Mask Type="Regex" Regex="(?&lt;=.).(?=.*@)">*</Mask>
  <UserInputType>Readonly</UserInputType>
</ClaimType>
```

L’Infrastructure d’expérience d’identité affiche uniquement la première lettre de l’adresse e-mail et le nom de domaine de messagerie :

![Revendication d’e-mail affichée dans le navigateur avec les caractères masqués par des astérisques](./media/claimsschema/mask-regex.png)


### <a name="restriction"></a>Restriction

L’élément **Restriction** peut contenir l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| MergeBehavior | Non | Méthode utilisée pour fusionner des valeurs d’énumération avec un ClaimType dans une stratégie parente avec le même identificateur. Utilisez cet attribut quand vous remplacez une revendication spécifiée dans la stratégie de base. Valeurs possibles : `Append`, `Prepend` ou `ReplaceAll`. La valeur `Append` est une collection de données qui doivent être ajoutées à la fin de la collection spécifiée dans la stratégie parente. La valeur `Prepend` est une collection de données qui doivent être ajoutées avant la collection spécifiée dans la stratégie parente. La valeur `ReplaceAll` est une collection de données spécifiée dans la stratégie parente qui doit être ignorée. |

L’élément **Restriction** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| Énumération | 1:n | Options que l’utilisateur peut sélectionner dans l’interface utilisateur pour une revendication, telles qu’une valeur dans une liste déroulante. |
| Modèle | 1:1 | Expression régulière à utiliser. |

#### <a name="enumeration"></a>Énumération

L’élément **Enumeration** définit les options que l’utilisateur peut sélectionner pour une revendication dans l’interface utilisateur, telle qu’une valeur dans un `CheckboxMultiSelect`, un `DropdownSingleSelect`, ou un `RadioSingleSelect`. Vous pouvez également définir et localiser les options disponibles avec l’élément [LocalizedCollections](localization.md#localizedcollections). Pour rechercher un élément à partir d’une collection de revendications **Enumeration**, utilisez la transformation de revendications [GetMappedValueFromLocalizedCollection](string-transformations.md#getmappedvaluefromlocalizedcollection).

L’élément **Enumeration** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Texte | Oui | Chaîne d’affichage présentée à l’utilisateur dans l’interface utilisateur pour cette option. |
|Valeur | Oui | Valeur de revendication associée à la sélection de cette option. |
| SelectByDefault | Non | Indique si cette option doit être sélectionnée par défaut dans l’interface utilisateur. Valeurs possibles : True ou False. |

L’exemple suivant configure une revendication de liste déroulante **city** avec une valeur par défaut définie sur `New York` :

```xml
<ClaimType Id="city">
  <DisplayName>city where you work</DisplayName>
  <DataType>string</DataType>
  <UserInputType>DropdownSingleSelect</UserInputType>
  <Restriction>
    <Enumeration Text="Bellevue" Value="bellevue" SelectByDefault="false" />
    <Enumeration Text="Redmond" Value="redmond" SelectByDefault="false" />
    <Enumeration Text="New York" Value="new-york" SelectByDefault="true" />
  </Restriction>
</ClaimType>
```

Liste déroulante de villes avec New York comme valeur par défaut :

![Contrôle de liste déroulante affiché dans le navigateur et présentant la valeur par défaut](./media/claimsschema/dropdownsingleselect.png)

### <a name="pattern"></a>Modèle

L’élément **Pattern** peut contenir les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| RegularExpression | Oui | Expression régulière auxquelles les revendications de ce type doivent correspondre pour être valides. |
| HelpText | Non | Message d’erreur présenté aux utilisateurs si la vérification de l’expression régulière échoue. |

L’exemple suivant configure une revendication **email** avec validation de l’entrée d’expression régulière et texte d’aide :

```xml
<ClaimType Id="email">
  <DisplayName>Email Address</DisplayName>
  <DataType>string</DataType>
  <DefaultPartnerClaimTypes>
  <Protocol Name="OpenIdConnect" PartnerClaimType="email" />
  </DefaultPartnerClaimTypes>
  <UserHelpText>Email address that can be used to contact you.</UserHelpText>
  <UserInputType>TextBox</UserInputType>
  <Restriction>
    <Pattern RegularExpression="^[a-zA-Z0-9.+!#$%&amp;'^_`{}~-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)*$" HelpText="Please enter a valid email address." />
  </Restriction>
</ClaimType>
```

L’Infrastructure d’expérience d’identité affiche la revendication d’adresse e-mail avec validation de l’entrée du format d’e-mail :

![TextBox présentant le message d’erreur déclenché par la restriction regex](./media/claimsschema/pattern.png)

### <a name="userinputtype"></a>UserInputType

Azure AD B2C prend en charge toute une gamme de types d’entrée d’utilisateur, telles qu’une zone de texte, un mot de passe et une liste déroulante pouvant être utilisées lors de l’entrée manuelle de données de revendications pour le type de revendication. Vous devez spécifier le **UserInputType** quand vous recueillez des informations de l’utilisateur à l’aide d’un [profil technique autodéclaré](self-asserted-technical-profile.md) et de [contrôles d’affichage](display-controls.md).

Les types d’entrée utilisateur disponibles pour l’élément **UserInputType** :

| UserInputType | ClaimType pris en charge | Description |
| --------- | -------- | ----------- |
|CheckboxMultiSelect| `string` |Zone de liste déroulante à sélection multiple. La valeur de la revendication est représentée dans une chaîne délimitée par des virgules qui contient les valeurs sélectionnées. |
|DateTimeDropdown | `date`, `dateTime` |Listes déroulantes pour sélectionner un jour, un mois et une année. |
|DropdownSingleSelect |`string` |Zone de liste déroulante à sélection simple. La valeur de la revendication est la valeur sélectionnée.|
|EmailBox | `string` |Champ d’entrée de l’e-mail. |
|Paragraph | `boolean`, `date`, `dateTime`, `duration`, `int`, `long`, `string`|Champ qui affiche le texte uniquement dans une balise de paragraphe. |
|Mot de passe | `string` |Zone de texte pour mot de passe.|
|RadioSingleSelect |`string` | Collection de cases d’option. La valeur de la revendication est la valeur sélectionnée.|
|Readonly | `boolean`, `date`, `dateTime`, `duration`, `int`, `long`, `string`| Zone de texte en lecture seule. |
|TextBox |`boolean`, `int`, `string` |Zone de texte sur une seule ligne. |


#### <a name="textbox"></a>TextBox

Le type d’entrée d’utilisateur **TextBox** sert à fournir une zone de texte sur une seule ligne.

![TextBox présentant les propriétés spécifiées dans le type de revendication](./media/claimsschema/textbox.png)

```xml
<ClaimType Id="displayName">
  <DisplayName>Display Name</DisplayName>
  <DataType>string</DataType>
  <UserHelpText>Your display name.</UserHelpText>
  <UserInputType>TextBox</UserInputType>
</ClaimType>
```

#### <a name="emailbox"></a>EmailBox

Le type d’entrée d’utilisateur **EmailBox** sert à fournir un champ d’entrée d’e-mail de base.

![EmailBox présentant les propriétés spécifiées dans le type de revendication](./media/claimsschema/emailbox.png)

```xml
<ClaimType Id="email">
  <DisplayName>Email Address</DisplayName>
  <DataType>string</DataType>
  <UserHelpText>Email address that can be used to contact you.</UserHelpText>
  <UserInputType>EmailBox</UserInputType>
  <Restriction>
    <Pattern RegularExpression="^[a-zA-Z0-9.+!#$%&amp;'+^_`{}~-]+(?:\.[a-zA-Z0-9!#$%&amp;'+^_`{}~-]+)*@(?:[a-zA-Z0-9](?:[a-zA-Z0-9-]*[a-zA-Z0-9])?\.)+[a-zA-Z0-9](?:[a-zA-Z0-9-]*[a-zA-Z0-9])?$" HelpText="Please enter a valid email address." />
  </Restriction>
</ClaimType>
```

#### <a name="password"></a>Mot de passe

Le type d’entrée d’utilisateur **Password** sert à enregistrer un mot de passe entré par l’utilisateur.

![Utilisation du type de revendication avec un mot de passe](./media/claimsschema/password.png)

```xml
<ClaimType Id="password">
  <DisplayName>Password</DisplayName>
  <DataType>string</DataType>
  <UserHelpText>Enter password</UserHelpText>
  <UserInputType>Password</UserInputType>
</ClaimType>
```

#### <a name="datetimedropdown"></a>DateTimeDropdown

Le type d’entrée d’utilisateur **DateTimeDropdown** sert à fournir un ensemble de listes déroulantes pour sélectionner un jour, un mois et une année. Vous pouvez utiliser des éléments Predicates et PredicateValidations pour contrôler les valeurs de date minimale et maximale. Pour plus d’informations, consultez la section **Configurer une plage de dates** de [Predicates et PredicateValidations](predicates.md).

![Utilisation du type de revendication avec datetimedropdown](./media/claimsschema/datetimedropdown.png)

```xml
<ClaimType Id="dateOfBirth">
  <DisplayName>Date Of Birth</DisplayName>
  <DataType>date</DataType>
  <UserHelpText>The date on which you were born.</UserHelpText>
  <UserInputType>DateTimeDropdown</UserInputType>
</ClaimType>
```

#### <a name="radiosingleselect"></a>RadioSingleSelect

Le type d’entrée d’utilisateur **RadioSingleSelect** sert à fournir une collection de cases d’option qui permet à l’utilisateur de sélectionner une option.

![Utilisation du type de revendication avec radiodsingleselect](./media/claimsschema/radiosingleselect.png)

```xml
<ClaimType Id="color">
  <DisplayName>Preferred color</DisplayName>
  <DataType>string</DataType>
  <UserInputType>RadioSingleSelect</UserInputType>
  <Restriction>
    <Enumeration Text="Blue" Value="Blue" SelectByDefault="false" />
    <Enumeration Text="Green " Value="Green" SelectByDefault="false" />
    <Enumeration Text="Orange" Value="Orange" SelectByDefault="true" />
  </Restriction>
</ClaimType>
```

#### <a name="dropdownsingleselect"></a>DropdownSingleSelect

Le type d’entrée d’utilisateur **DropdownSingleSelect** sert à fournir une zone de liste déroulante qui permet à l’utilisateur de sélectionner une option.

![Utilisation du type de revendication avec dropdownsingleselect](./media/claimsschema/dropdownsingleselect.png)

```xml
<ClaimType Id="city">
  <DisplayName>City where you work</DisplayName>
  <DataType>string</DataType>
  <UserInputType>DropdownSingleSelect</UserInputType>
  <Restriction>
    <Enumeration Text="Bellevue" Value="bellevue" SelectByDefault="false" />
    <Enumeration Text="Redmond" Value="redmond" SelectByDefault="false" />
    <Enumeration Text="New York" Value="new-york" SelectByDefault="true" />
  </Restriction>
</ClaimType>
```

#### <a name="checkboxmultiselect"></a>CheckboxMultiSelect

Le type d’entrée d’utilisateur **CheckboxMultiSelect** sert à fournir une collection de cases à cocher qui permet à l’utilisateur de sélectionner plusieurs options.

![Utilisation du type de revendication avec checkboxmultiselect](./media/claimsschema/checkboxmultiselect.png)

```xml
<ClaimType Id="languages">
  <DisplayName>Languages you speak</DisplayName>
  <DataType>string</DataType>
  <UserInputType>CheckboxMultiSelect</UserInputType>
  <Restriction>
    <Enumeration Text="English" Value="English" SelectByDefault="true" />
    <Enumeration Text="France " Value="France" SelectByDefault="false" />
    <Enumeration Text="Spanish" Value="Spanish" SelectByDefault="false" />
  </Restriction>
</ClaimType>
```

#### <a name="readonly"></a>Readonly

Le type d’entrée d’utilisateur **Readonly** sert à fournir un champ en lecture seule pour afficher la revendication et la valeur.

![Utilisation du type de revendication avec readonly](./media/claimsschema/readonly.png)

```xml
<ClaimType Id="membershipNumber">
  <DisplayName>Membership number</DisplayName>
  <DataType>string</DataType>
  <UserHelpText>Your membership number (read only)</UserHelpText>
  <UserInputType>Readonly</UserInputType>
</ClaimType>
```


#### <a name="paragraph"></a>Paragraph

Le type d’entrée d’utilisateur **Paragraph** sert à fournir un champ qui affiche du texte uniquement dans une balise de paragraphe.  Par exemple, &lt;p&gt;texte&lt;/p&gt;. Un type d’entrée utilisateur **Paragraphe** `OutputClaim` du profil technique autodéclaré doit définir l’attribut `Required` sur `false` (valeur par défaut).

![Utilisation du type de revendication avec paragraph](./media/claimsschema/paragraph.png)

```xml
<ClaimType Id="responseMsg">
  <DisplayName>Error message: </DisplayName>
  <DataType>string</DataType>
  <AdminHelpText>A claim responsible for holding response messages to send to the relying party</AdminHelpText>
  <UserHelpText>A claim responsible for holding response messages to send to the relying party</UserHelpText>
  <UserInputType>Paragraph</UserInputType>
  <Restriction>
    <Enumeration Text="B2C_V1_90001" Value="You cannot sign in because you are a minor" />
    <Enumeration Text="B2C_V1_90002" Value="This action can only be performed by gold members" />
    <Enumeration Text="B2C_V1_90003" Value="You have not been enabled for this operation" />
  </Restriction>
</ClaimType>
```
