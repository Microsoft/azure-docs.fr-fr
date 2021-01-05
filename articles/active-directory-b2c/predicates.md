---
title: Predicates et PredicateValidations
titleSuffix: Azure AD B2C
description: Empêchez l’ajout de données mal mises en forme à votre locataire Azure AD B2C en utilisant des stratégies personnalisées dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 03/30/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 46f04c55b40d4f1bdbbf5fd55eb648d1d3294056
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97108414"
---
# <a name="predicates-and-predicatevalidations"></a>Predicates et PredicateValidations

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Les éléments **Predicates** et **PredicateValidations** vous permettent d’effectuer un processus de validation afin de vous assurer que seules des données correctement formées sont entrées dans votre locataire Azure Active Directory B2C (Azure AD B2C).

Le diagramme suivant illustre la relation entre les éléments :

![Diagramme montrant la relation entre les prédicats et les validations de prédicat](./media/predicates/predicates.png)

## <a name="predicates"></a>Prédicats

L’élément **Predicate** définit une validation de base pour vérifier la valeur d’un type de revendication et retourne `true` ou `false`. La validation est effectuée en utilisant un élément **Method** et un ensemble d’éléments **Parameter** pertinents à la méthode. Par exemple, un prédicat peut vérifier si la longueur d’une valeur de revendication de chaîne est dans la plage des paramètres minimaux et maximaux spécifiés, ou si une valeur de revendication de chaîne contient un jeu de caractères. L’élément **UserHelpText** fournit un message d’erreur présenté aux utilisateurs si la vérification échoue. La valeur de l’élément **UserHelpText** peut être localisée à l’aide de la [personnalisation de la langue](localization.md).

L’élément **Predicates** doit apparaître directement après l’élément **ClaimsSchema** dans l’élément [BuildingBlocks](buildingblocks.md).

L’élément **Predicates** contient l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| Predicate | 1:n | Liste de prédicats. |

L’élément **Predicate** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Id | Oui | Identificateur utilisé pour le prédicat. D’autres éléments peuvent utiliser cet identificateur dans la stratégie. |
| Méthode | Oui | Type de la méthode à utiliser pour la validation. Valeurs possibles : [IsLengthRange](#islengthrange), [MatchesRegex](#matchesregex), [IncludesCharacters](#includescharacters) ou [IsDateRange](#isdaterange).  |
| HelpText | Non | Message d’erreur présenté aux utilisateurs si la vérification échoue. Vous pouvez localiser cette chaîne à l’aide de la [personnalisation de la langue](localization.md). |

L’élément **Predicate** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| UserHelpText | 0:1 | (Déconseillé) Message d’erreur présenté aux utilisateurs si la vérification échoue. |
| Paramètres | 1:1 | Paramètres pour le type de méthode de la validation de chaîne. |

L’élément **Parameters** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| Paramètre | 1:n | Paramètres pour le type de méthode de la validation de chaîne. |

L’élément **Parameter** contient les attributs suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| Id | 1:1 | Identificateur du paramètre. |

### <a name="predicate-methods"></a>Méthodes de prédicat

#### <a name="islengthrange"></a>IsLengthRange

La méthode IsLengthRange vérifie si la longueur d’une valeur de revendication de chaîne est dans la plage des paramètres minimaux et maximaux spécifiés. L’élément de prédicat prend en charge les paramètres suivants :

| Paramètre | Obligatoire | Description |
| ------- | ----------- | ----------- |
| Maximale | Oui | Nombre maximal de caractères autorisé. |
| Minimum | Oui | Nombre minimal de caractères autorisé. |


L’exemple suivant montre une méthode IsLengthRange avec les paramètres `Minimum` et `Maximum` qui spécifient la plage de la longueur de la chaîne :

```xml
<Predicate Id="IsLengthBetween8And64" Method="IsLengthRange" HelpText="The password must be between 8 and 64 characters.">
  <Parameters>
    <Parameter Id="Minimum">8</Parameter>
    <Parameter Id="Maximum">64</Parameter>
  </Parameters>
</Predicate>
```

#### <a name="matchesregex"></a>MatchesRegex

La méthode MatchesRegex vérifie si une valeur de revendication de chaîne correspond à une expression régulière. L’élément de prédicat prend en charge les paramètres suivants :

| Paramètre | Obligatoire | Description |
| ------- | ----------- | ----------- |
| RegularExpression | Oui | Modèle d’expression régulière à mettre en correspondance. |

L’exemple suivant montre une méthode `MatchesRegex` avec le paramètre `RegularExpression` qui spécifie une expression régulière :

```xml
<Predicate Id="PIN" Method="MatchesRegex" HelpText="The password must be numbers only.">
  <Parameters>
    <Parameter Id="RegularExpression">^[0-9]+$</Parameter>
  </Parameters>
</Predicate>
```

#### <a name="includescharacters"></a>IncludesCharacters

La méthode IncludesCharacters vérifie si une valeur de revendication de chaîne contient un jeu de caractères. L’élément de prédicat prend en charge les paramètres suivants :

| Paramètre | Obligatoire | Description |
| ------- | ----------- | ----------- |
| CharacterSet | Oui | Jeu de caractères qui peut être entré. Par exemple, les caractères minuscules `a-z`, les caractères majuscules `A-Z`, les chiffres `0-9` ou une liste de symboles, tels que `@#$%^&amp;*\-_+=[]{}|\\:',?/~"();!`. |

L’exemple suivant montre une méthode `IncludesCharacters` avec le paramètre `CharacterSet` qui spécifie le jeu de caractères :

```xml
<Predicate Id="Lowercase" Method="IncludesCharacters" HelpText="a lowercase letter">
  <Parameters>
    <Parameter Id="CharacterSet">a-z</Parameter>
  </Parameters>
</Predicate>
```

#### <a name="isdaterange"></a>IsDateRange

La méthode IsDateRange vérifie si une valeur de revendication de date est dans la plage des paramètres minimaux et maximaux spécifiés. L’élément de prédicat prend en charge les paramètres suivants :

| Paramètre | Obligatoire | Description |
| ------- | ----------- | ----------- |
| Maximale | Oui | La plus grande date possible qui peut être entrée. Le format de la date suit la convention `yyyy-mm-dd`, ou `Today`. |
| Minimum | Oui | La plus petite date possible qui peut être entrée. Le format de la date suit la convention `yyyy-mm-dd`, ou `Today`.|

L’exemple suivant montre une méthode `IsDateRange` avec les paramètres `Minimum` et `Maximum` qui spécifient la plage de dates avec le format `yyyy-mm-dd` et `Today`.

```xml
<Predicate Id="DateRange" Method="IsDateRange" HelpText="The date must be between 1970-01-01 and today.">
  <Parameters>
    <Parameter Id="Minimum">1970-01-01</Parameter>
    <Parameter Id="Maximum">Today</Parameter>
  </Parameters>
</Predicate>
```

## <a name="predicatevalidations"></a>PredicateValidations

Alors que les prédicats définissent la validation à effectuer par rapport à un type de revendication, les **PredicateValidations** regroupent un ensemble de prédicats pour former une validation de l’entrée utilisateur qui peut être appliquée à un type de revendication. Chaque élément **PredicateValidation** contient un ensemble d’éléments **PredicateGroup** qui contiennent un ensemble d’éléments **PredicateReference** qui pointe vers un **Predicate**. Pour que la validation réussisse, la valeur de la revendication doit réussir tous les tests des prédicats qui se trouvent sous tous les éléments **PredicateGroup** avec leur ensemble d’éléments **PredicateReference**.

L’élément **PredicatesValidations** doit apparaître directement après l’élément **Predicates** dans l’élément [BuildingBlocks](buildingblocks.md).

```xml
<PredicateValidations>
  <PredicateValidation Id="">
    <PredicateGroups>
      <PredicateGroup Id="">
        <UserHelpText></UserHelpText>
        <PredicateReferences MatchAtLeast="">
          <PredicateReference Id="" />
          ...
        </PredicateReferences>
      </PredicateGroup>
      ...
    </PredicateGroups>
  </PredicateValidation>
...
</PredicateValidations>
```

L’élément **PredicateValidations** contient l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| PredicateValidation | 1:n | Liste de validations de prédicat. |

L’élément **PredicateValidation** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Id | Oui | Identificateur utilisé pour la validation de prédicat. L’élément **ClaimType** peut utiliser cet identificateur dans la stratégie. |

L’élément **PredicateValidation** contient l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| PredicateGroups | 1:n | Liste de groupes de prédicats. |

L’élément **PredicateGroups** contient l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| PredicateGroup | 1:n | Liste de prédicats. |

L’élément **PredicateGroup** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Id | Oui | Identificateur utilisé pour le groupe de prédicats.  |

L’élément **PredicateGroup** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| UserHelpText | 0:1 |  Description du prédicat qui peut être aider les utilisateurs à déterminer quelle valeur ils doivent taper. |
| PredicateReferences | 1:n | Liste de références de prédicats. |

L’élément **PredicateReferences** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| MatchAtLeast | Non | Indique que la valeur doit correspondre au moins à cette quantité de définitions de prédicat pour que l’entrée soit acceptée. En l’absence de précision, la valeur doit correspondre à toutes les définitions de prédicat. |

L’élément **PredicateReferences** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| PredicateReference | 1:n | Référence à un prédicat. |

L’élément **PredicateReference** contient les attributs suivants :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| Id | Oui | Identificateur utilisé pour la validation de prédicat.  |


## <a name="configure-password-complexity"></a>Configurer la complexité du mot de passe

Avec **Predicates** et **PredicateValidationsInput**, vous pouvez contrôler les exigences de complexité des mots de passe fournis par un utilisateur lors de la création d’un compte. Par défaut, Azure AD B2C utilise des mots de passe forts. Azure AD B2C prend également en charge les options de configuration permettant de contrôler la complexité des mots de passe utilisables par les clients. Vous pouvez définir la complexité du mot de passe à l’aide de ces éléments de prédicats :

- **IsLengthBetween8And64** à l’aide de la méthode `IsLengthRange` valide le fait que le mot de passe doit compter entre 8 et 64 caractères.
- **Lowercase** à l’aide de la méthode `IncludesCharacters` valide le fait que le mot de passe contient une lettre minuscule.
- **Uppercase** à l’aide de la méthode `IncludesCharacters` valide le fait que le mot de passe contient une lettre majuscule.
- **Number** à l’aide de la méthode `IncludesCharacters` valide le fait que le mot de passe contient un chiffre.
- **Symbol** à l’aide de la méthode `IncludesCharacters` valide le fait que le mot de passe contient l’un des caractères symboles.
- **PIN** à l’aide de la méthode `MatchesRegex` valide le fait que le mot de passe contient uniquement des chiffres.
- **AllowedAADCharacters** à l’aide de la méthode `MatchesRegex` valide le fait que le mot de passe fourni contenait uniquement des caractère non valides.
- **DisallowedWhitespace** à l’aide de la méthode `MatchesRegex` valide le fait que le mot de passe ne commence pas ou ne se termine pas par un espace.

```xml
<Predicates>
  <Predicate Id="IsLengthBetween8And64" Method="IsLengthRange" HelpText="The password must be between 8 and 64 characters.">
    <Parameters>
      <Parameter Id="Minimum">8</Parameter>
      <Parameter Id="Maximum">64</Parameter>
    </Parameters>
  </Predicate>

  <Predicate Id="Lowercase" Method="IncludesCharacters" HelpText="a lowercase letter">
    <Parameters>
      <Parameter Id="CharacterSet">a-z</Parameter>
    </Parameters>
  </Predicate>

  <Predicate Id="Uppercase" Method="IncludesCharacters" HelpText="an uppercase letter">
    <Parameters>
      <Parameter Id="CharacterSet">A-Z</Parameter>
    </Parameters>
  </Predicate>

  <Predicate Id="Number" Method="IncludesCharacters" HelpText="a digit">
    <Parameters>
      <Parameter Id="CharacterSet">0-9</Parameter>
    </Parameters>
  </Predicate>

  <Predicate Id="Symbol" Method="IncludesCharacters" HelpText="a symbol">
    <Parameters>
      <Parameter Id="CharacterSet">@#$%^&amp;*\-_+=[]{}|\\:',.?/`~"();!</Parameter>
    </Parameters>
  </Predicate>

  <Predicate Id="PIN" Method="MatchesRegex" HelpText="The password must be numbers only.">
    <Parameters>
      <Parameter Id="RegularExpression">^[0-9]+$</Parameter>
    </Parameters>
  </Predicate>

  <Predicate Id="AllowedAADCharacters" Method="MatchesRegex" HelpText="An invalid character was provided.">
    <Parameters>
      <Parameter Id="RegularExpression">(^([0-9A-Za-z\d@#$%^&amp;*\-_+=[\]{}|\\:',?/`~"();! ]|(\.(?!@)))+$)|(^$)</Parameter>
    </Parameters>
  </Predicate>

  <Predicate Id="DisallowedWhitespace" Method="MatchesRegex" HelpText="The password must not begin or end with a whitespace character.">
    <Parameters>
      <Parameter Id="RegularExpression">(^\S.*\S$)|(^\S+$)|(^$)</Parameter>
    </Parameters>
  </Predicate>
```

Après avoir défini les validations de base, vous pouvez les combiner et créer un ensemble de stratégies de mot de passe utilisable dans votre stratégie :

- **SimplePassword** valide DisallowedWhitespace, AllowedAADCharacters et IsLengthBetween8And64
- **StrongPassword** valide DisallowedWhitespace, AllowedAADCharacters et IsLengthBetween8And64. Le dernier groupe `CharacterClasses` exécute un jeu supplémentaire de prédicats avec `MatchAtLeast` défini sur 3. Le mot de passe de l’utilisateur doit comprendre entre 8 et 16 caractères et trois des caractères suivants : minuscule, majuscule, nombre ou symbole.
- **CustomPassword** valide uniquement DisallowedWhitespace, AllowedAADCharacters. Ainsi, l’utilisateur peut fournir n’importe quel mot de passe de n’importe quelle longueur, tant que les caractères sont valides.

```xml
<PredicateValidations>
  <PredicateValidation Id="SimplePassword">
    <PredicateGroups>
      <PredicateGroup Id="DisallowedWhitespaceGroup">
        <PredicateReferences>
          <PredicateReference Id="DisallowedWhitespace" />
        </PredicateReferences>
      </PredicateGroup>
      <PredicateGroup Id="AllowedAADCharactersGroup">
        <PredicateReferences>
          <PredicateReference Id="AllowedAADCharacters" />
        </PredicateReferences>
      </PredicateGroup>
      <PredicateGroup Id="LengthGroup">
        <PredicateReferences>
          <PredicateReference Id="IsLengthBetween8And64" />
        </PredicateReferences>
      </PredicateGroup>
    </PredicateGroups>
  </PredicateValidation>

  <PredicateValidation Id="StrongPassword">
    <PredicateGroups>
      <PredicateGroup Id="DisallowedWhitespaceGroup">
        <PredicateReferences>
          <PredicateReference Id="DisallowedWhitespace" />
       </PredicateReferences>
      </PredicateGroup>
      <PredicateGroup Id="AllowedAADCharactersGroup">
        <PredicateReferences>
          <PredicateReference Id="AllowedAADCharacters" />
        </PredicateReferences>
      </PredicateGroup>
      <PredicateGroup Id="LengthGroup">
        <PredicateReferences>
          <PredicateReference Id="IsLengthBetween8And64" />
        </PredicateReferences>
      </PredicateGroup>
      <PredicateGroup Id="CharacterClasses">
        <UserHelpText>The password must have at least 3 of the following:</UserHelpText>
        <PredicateReferences MatchAtLeast="3">
          <PredicateReference Id="Lowercase" />
          <PredicateReference Id="Uppercase" />
          <PredicateReference Id="Number" />
          <PredicateReference Id="Symbol" />
        </PredicateReferences>
      </PredicateGroup>
    </PredicateGroups>
  </PredicateValidation>

  <PredicateValidation Id="CustomPassword">
    <PredicateGroups>
      <PredicateGroup Id="DisallowedWhitespaceGroup">
        <PredicateReferences>
          <PredicateReference Id="DisallowedWhitespace" />
        </PredicateReferences>
      </PredicateGroup>
      <PredicateGroup Id="AllowedAADCharactersGroup">
        <PredicateReferences>
          <PredicateReference Id="AllowedAADCharacters" />
        </PredicateReferences>
      </PredicateGroup>
    </PredicateGroups>
  </PredicateValidation>
</PredicateValidations>
```

Dans votre type de revendication, ajoutez l’élément **PredicateValidationReference** et spécifiez l’identificateur en tant que l’une des validations de prédicat, comme SimplePassword, StrongPassword ou CustomPassword.

```xml
<ClaimType Id="password">
  <DisplayName>Password</DisplayName>
  <DataType>string</DataType>
  <AdminHelpText>Enter password</AdminHelpText>
  <UserHelpText>Enter password</UserHelpText>
  <UserInputType>Password</UserInputType>
  <PredicateValidationReference Id="StrongPassword" />
</ClaimType>
```

Le schéma suivant montre comment les éléments sont organisés quand Azure AD B2C affiche le message d’erreur :

![Diagramme montrant des exemples de complexité de mot de passe PredicateGroup Predicate](./media/predicates/predicates-pass.png)

## <a name="configure-a-date-range"></a>Configurer une plage de dates

Avec les éléments **Predicates** et **PredicateValidations**, vous pouvez contrôler les valeurs de date minimale et maximale de **UserInputType** en utilisant un `DateTimeDropdown`. Pour ce faire, créez un **Predicate** avec la méthode `IsDateRange` et spécifiez les paramètres minimal et maximal.

```xml
<Predicates>
  <Predicate Id="DateRange" Method="IsDateRange" HelpText="The date must be between 01-01-1980 and today.">
    <Parameters>
      <Parameter Id="Minimum">1980-01-01</Parameter>
      <Parameter Id="Maximum">Today</Parameter>
    </Parameters>
  </Predicate>
</Predicates>
```

Ajoutez un **PredicateValidation** avec une référence au prédicat `DateRange`.

```xml
<PredicateValidations>
  <PredicateValidation Id="CustomDateRange">
    <PredicateGroups>
      <PredicateGroup Id="DateRangeGroup">
        <PredicateReferences>
          <PredicateReference Id="DateRange" />
        </PredicateReferences>
      </PredicateGroup>
    </PredicateGroups>
  </PredicateValidation>
</PredicateValidations>
```

Dans votre type de revendication, ajoutez un élément **PredicateValidationReference** et spécifiez l’identificateur en tant que `CustomDateRange`.

```xml
<ClaimType Id="dateOfBirth">
  <DisplayName>Date of Birth</DisplayName>
  <DataType>date</DataType>
  <AdminHelpText>The user's date of birth.</AdminHelpText>
  <UserHelpText>Your date of birth.</UserHelpText>
  <UserInputType>DateTimeDropdown</UserInputType>
  <PredicateValidationReference Id="CustomDateRange" />
</ClaimType>
 ```

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [configurer la complexité du mot de passe avec des stratégies personnalisées dans Azure Active Directory B2C](password-complexity.md) à l’aide de validations de prédicat.