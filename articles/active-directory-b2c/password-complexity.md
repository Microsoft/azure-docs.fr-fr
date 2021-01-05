---
title: Configurer les critères de complexité des mots de passe
titleSuffix: Azure AD B2C
description: Guide pratique pour configurer les exigences de complexité des mots de passe fournis par les consommateurs dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 12/10/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
zone_pivot_groups: b2c-policy-type
ms.openlocfilehash: 0cd9f0d6cf9529439f7b5ce46b1a5807d0a68a7c
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97110951"
---
# <a name="configure-complexity-requirements-for-passwords-in-azure-active-directory-b2c"></a>Configurer les exigences de complexité pour les mots de passe dans Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-choose-user-flow-or-custom-policy](../../includes/active-directory-b2c-choose-user-flow-or-custom-policy.md)]

Azure Active Directory B2C (Azure AD B2C) prend en charge la modification des exigences de complexité des mots de passe fournis par un utilisateur final à la création d’un compte. Par défaut, Azure AD B2C utilise des mots de passe **forts**. Azure AD B2C prend également en charge les options de configuration permettant de contrôler la complexité des mots de passe utilisables par les clients.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [active-directory-b2c-customization-prerequisites](../../includes/active-directory-b2c-customization-prerequisites.md)]

::: zone pivot="b2c-user-flow"

## <a name="password-rule-enforcement"></a>Application des règles de mot de passe

Lors de l’inscription ou de la réinitialisation de mot de passe, un utilisateur final doit fournir un mot de passe qui respecte les règles de complexité. Les règles de complexité du mot de passe s’appliquent flux d’utilisateur par flux d’utilisateur. Il est possible qu’un flux d’utilisateur demande un code confidentiel à quatre chiffres et un autre une chaîne de huit caractères à l’inscription. Par exemple, vous pouvez utiliser un flux d’utilisateur avec différents niveaux de complexité du mot de passe pour les adultes et pour les enfants.

La complexité du mot de passe ne s’applique jamais lors de la connexion. Les utilisateurs ne sont jamais invités à modifier leur mot de passe parce qu’il ne répond pas à l’exigence de complexité actuelle.

Vous pouvez configurer la complexité des mot de passe dans les types de flux d’utilisateurs suivants :

- Flux d’utilisateur d’inscription ou de connexion
- Flux d’utilisateur de réinitialisation du mot de passe

Si vous utilisez des stratégies personnalisées, vous pouvez ([configurer la complexité des mots de passe dans une stratégie personnalisée](password-complexity.md)).

## <a name="configure-password-complexity"></a>Configurer la complexité du mot de passe

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Sélectionnez l’icône **Annuaire et abonnement** dans la barre d’outils du portail, puis sélectionnez l’annuaire qui contient votre locataire Azure AD B2C.
3. Dans la Portail Azure, recherchez et sélectionnez **Azure AD B2C**.
4. Sélectionnez **Flux d’utilisateurs**.
2. Sélectionnez un flux d’utilisateur, puis cliquez sur **Propriétés**.
3. Sous **Complexité du mot de passe**, changez la complexité du mot de passe de ce flux d’utilisateur : **Simple**, **Fort** ou **Personnalisé**.

### <a name="comparison-chart"></a>Tableau comparatif

| Complexité | Description |
| --- | --- |
| Simple | Un mot de passe entre 8 et 64 caractères. |
| Remarque | Un mot de passe entre 8 et 64 caractères. Trois de ces quatre types d’éléments : minuscules, majuscules, chiffres ou symboles. |
| Custom | C’est l’option qui fournit le contrôle le plus poussé sur les règles de complexité du mot de passe.  Elle permet de configurer une longueur personnalisée,  et d’accepter les mots de passe composés exclusivement de chiffres (codes confidentiels). |

## <a name="custom-options"></a>Options personnalisées

### <a name="character-set"></a>Jeu de caractères

Permet d’accepter des chiffres uniquement (code confidentiel) ou la totalité du jeu de caractères.

- **Numbers only** (Chiffres uniquement) autorise la saisie de mots de passe composés exclusivement de chiffres (0-9).
- **All** (Tout) accepte toutes les lettres, tous les chiffres et tous les symboles.

### <a name="length"></a>Longueur

Permet de contrôler les critères de longueur du mot de passe.

- **Minimum Length** (Longueur minimale) doit avoir au moins la valeur 4.
- **Maximum Length** (Longueur maximale) doit avoir une valeur supérieure ou égale à la longueur minimale, au maximum 64 caractères.

### <a name="character-classes"></a>Classes de caractères

Permet de contrôler les différents types de caractères utilisés dans le mot de passe.

- **2 of 4: Lowercase character, Uppercase character, Number (0-9), Symbol** (2 sur 4 : minuscule, majuscule, chiffre (0-9), symbole) garantit que le mot de passe contient au moins trois types de caractères, par exemple, un nombre et une minuscule.
- **3 of 4: Lowercase character, Uppercase character, Number (0-9), Symbol** (3 sur 4 : Minuscule, Majuscule, Chiffre (0-9), Symbole) garantit que le mot de passe contient au moins trois types de caractères. par exemple, un chiffre, une minuscule et une majuscule.
- **4 of 4: Lowercase character, Uppercase character, Number (0-9), Symbol** (4 sur 4 : minuscule, majuscule, chiffre (0-9), symbole) garantit que le mot de passe contient les quatre types de caractères.

    > [!NOTE]
    > La condition **4 sur 4** peut contrarier l’utilisateur final. Des études ont montré qu’elle n’améliore pas l’entropie du mot de passe. Consultez les [Recommandations du NIST en matière de mots de passe](https://pages.nist.gov/800-63-3/sp800-63b.html#appA).

::: zone-end

::: zone pivot="b2c-custom-policy"

## <a name="password-predicate-validation"></a>Validation de prédicats de mot de passe

Pour configurer la complexité du mot de passe, remplacez les [types de revendication](claimsschema.md) `newPassword` et `reenterPassword` par une référence aux [validations de prédicat](predicates.md#predicatevalidations). L’élément PredicateValidations regroupe un ensemble de prédicats pour former une validation d’entrée utilisateur qui peut être appliquée à un type de revendication. Ouvrez le fichier d’extensions de votre stratégie. Par exemple <em>`SocialAndLocalAccounts/`**`TrustFrameworkExtensions.xml`**</em>.

1. Recherchez l’élément [BuildingBlocks](buildingblocks.md). Si l’élément n’existe pas, ajoutez-le.
1. Localisez l’élément [ClaimsSchema](claimsschema.md). Si l’élément n’existe pas, ajoutez-le.
1. Ajoutez les revendications `newPassword` et `reenterPassword` à l’élément **ClaimsSchema**.

    ```xml
    <ClaimType Id="newPassword">
      <PredicateValidationReference Id="CustomPassword" />
    </ClaimType>
    <ClaimType Id="reenterPassword">
      <PredicateValidationReference Id="CustomPassword" />
    </ClaimType>
    ```

1. [Predicates](predicates.md) définit une validation de base pour vérifier la valeur d’un type de revendication et retourne true ou false. La validation est effectuée en utilisant un élément de méthode spécifié, et un ensemble de paramètres appropriés à la méthode. Ajoutez les prédicats suivants à l’élément **BuildingBlocks**, juste après la fermeture de l’élément `</ClaimsSchema>` :

    ```xml
    <Predicates>
      <Predicate Id="LengthRange" Method="IsLengthRange">
        <UserHelpText>The password must be between 6 and 64 characters.</UserHelpText>
        <Parameters>
          <Parameter Id="Minimum">6</Parameter>
          <Parameter Id="Maximum">64</Parameter>
        </Parameters>
      </Predicate>
      <Predicate Id="Lowercase" Method="IncludesCharacters">
        <UserHelpText>a lowercase letter</UserHelpText>
        <Parameters>
          <Parameter Id="CharacterSet">a-z</Parameter>
        </Parameters>
      </Predicate>
      <Predicate Id="Uppercase" Method="IncludesCharacters">
        <UserHelpText>an uppercase letter</UserHelpText>
        <Parameters>
          <Parameter Id="CharacterSet">A-Z</Parameter>
        </Parameters>
      </Predicate>
      <Predicate Id="Number" Method="IncludesCharacters">
        <UserHelpText>a digit</UserHelpText>
        <Parameters>
          <Parameter Id="CharacterSet">0-9</Parameter>
        </Parameters>
      </Predicate>
      <Predicate Id="Symbol" Method="IncludesCharacters">
        <UserHelpText>a symbol</UserHelpText>
        <Parameters>
          <Parameter Id="CharacterSet">@#$%^&amp;*\-_+=[]{}|\\:',.?/`~"();!</Parameter>
        </Parameters>
      </Predicate>
    </Predicates>
    ```

1. Ajoutez les validations de prédicat suivantes à l’élément **BuildingBlocks**, juste après la fermeture de l’élément `</Predicates>` :

    ```xml
    <PredicateValidations>
      <PredicateValidation Id="CustomPassword">
        <PredicateGroups>
          <PredicateGroup Id="LengthGroup">
            <PredicateReferences MatchAtLeast="1">
              <PredicateReference Id="LengthRange" />
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
    </PredicateValidations>
    ```

## <a name="disable-strong-password"></a>Désactiver un mot de passe fort 

Les profils techniques suivants sont des [profils techniques Active Directory](active-directory-technical-profile.md) qui lisent et écrivent des données dans Azure Active Directory. Remplacez ces profils techniques dans le fichier d’extension. Utilisez `PersistedClaims` pour désactiver la stratégie de mot de passe fort. Recherchez l’élément **ClaimsProviders**.  Ajoutez les fournisseurs de revendication suivants de cette façon :

```xml
<ClaimsProvider>
  <DisplayName>Azure Active Directory</DisplayName>
  <TechnicalProfiles>
    <TechnicalProfile Id="AAD-UserWriteUsingLogonEmail">
      <PersistedClaims>
        <PersistedClaim ClaimTypeReferenceId="passwordPolicies" DefaultValue="DisablePasswordExpiration, DisableStrongPassword"/>
      </PersistedClaims>
    </TechnicalProfile>
    <TechnicalProfile Id="AAD-UserWritePasswordUsingObjectId">
      <PersistedClaims>
        <PersistedClaim ClaimTypeReferenceId="passwordPolicies" DefaultValue="DisablePasswordExpiration, DisableStrongPassword"/>
      </PersistedClaims>
    </TechnicalProfile>
  </TechnicalProfiles>
</ClaimsProvider>
```

Enregistrez le fichier de stratégie.

## <a name="test-your-policy"></a>Tester votre stratégie

### <a name="upload-the-files"></a>Téléchargement des fichiers

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Veillez à utiliser l’annuaire qui contient votre locataire Azure AD B2C en sélectionnant le filtre **Annuaire + abonnement** dans le menu du haut et en choisissant l’annuaire qui contient votre locataire.
3. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez **Azure AD B2C**.
4. Sélectionnez **Infrastructure d’expérience d’identité**.
5. Dans la page Stratégies personnalisées, cliquez sur **Charger une stratégie**.
6. Sélectionnez **Remplacer la stratégie si elle existe**, puis recherchez et sélectionnez le fichier *TrustFrameworkExtensions.xml*.
7. Cliquez sur **Télécharger**.

### <a name="run-the-policy"></a>Exécuter la stratégie

1. Ouvrez la stratégie d’inscription ou de connexion. Par exemple, *B2C_1A_signup_signin*.
2. Pour **Application**, sélectionnez l’application que vous avez précédemment inscrite. Pour voir le jeton, l’**URL de réponse** doit indiquer `https://jwt.ms`.
3. Cliquez sur **Exécuter maintenant**.
4. Sélectionnez **Inscrivez-vous maintenant**, entrez une adresse e-mail et entrez un nouveau mot de passe. Des conseils sur les restrictions de mot de passe s’affichent. Finissez d’entrer les informations utilisateur, puis cliquez sur **Créer**. Vous devez voir le contenu du jeton qui a été retourné.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [configurer la modification du mot de passe avec des stratégies personnalisées dans Azure Active Directory B2C](custom-policy-password-change.md).
- Apprenez-en davantage sur les éléments [Predicates](predicates.md) et [PredicateValidations](predicates.md#predicatevalidations) dans la référence IEF.


::: zone-end