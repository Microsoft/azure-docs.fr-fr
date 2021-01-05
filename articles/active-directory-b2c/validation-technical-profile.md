---
title: Définir un profil technique de validation dans une stratégie personnalisée
titleSuffix: Azure AD B2C
description: Validez les revendications en utilisant un profil technique de validation dans une stratégie personnalisée dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 03/16/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 2d4c538a9292698fecc8b44c055ab201748e292c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85202991"
---
# <a name="define-a-validation-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>Définir un profil technique de validation dans une stratégie personnalisée Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Un profil technique de validation est un profil technique ordinaire issu de n’importe quel protocole, comme [Azure Active Directory](active-directory-technical-profile.md) ou une [API REST](restful-technical-profile.md). Le profil technique de validation retourne des revendications de sortie ou un code d’état 4xx HTTP comprenant les données suivantes : Pour plus d’informations, consultez [Retour de message d’erreur](restful-technical-profile.md#returning-validation-error-message).

```json
{
    "version": "1.0.0",
    "status": 409,
    "userMessage": "Your error message"
}
```

La portée des revendications de sortie d’un profil technique de validation se limite au [profil technique autodéclaré](self-asserted-technical-profile.md) qui appelle le profil technique de validation, ainsi qu’à ses propres profils techniques de validation. Si vous souhaitez utiliser les revendications de sortie à l’étape d’orchestration suivante, ajoutez les revendications de sortie au profil technique autodéclaré qui appelle le profil technique de validation.

Les profils techniques de validation sont exécutés dans l’ordre où ils apparaissent dans l’élément **ValidationTechnicalProfiles**. Dans un profil technique de validation, vous pouvez choisir si l’exécution des profils techniques de validation suivants doit se poursuivre dans le cas où le profil technique de validation génère une erreur ou réussit.

Un profil technique de validation peut être exécuté de manière conditionnelle en fonction de conditions préalables définies dans l’élément **ValidationTechnicalProfile**. Par exemple, vous pouvez vérifier si une revendication existe, ou si une revendication est égale ou non à la valeur spécifiée.

Un profil technique autodéclaré peut définir un profil technique de validation à utiliser pour valider toutes ou certaines de ses revendications de sortie. Toutes les revendications d’entrée du profil technique référencé doivent apparaître dans les revendications de sortie du profil technique de validation de référencement.

> [!NOTE]
> Seuls les profils techniques à déclaration automatique peuvent utiliser des profils techniques de validation. Si vous devez valider les revendications de sortie à partir de profils techniques non déclarés automatiquement, envisagez d’utiliser une étape d’orchestration supplémentaire dans votre parcours utilisateur pour prendre en compte le profil technique en charge de la validation.

## <a name="validationtechnicalprofiles"></a>ValidationTechnicalProfiles

L’élément **ValidationTechnicalProfiles** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| ValidationTechnicalProfile | 1:n | Un profil technique pour valider toutes ou certaines des revendications de sortie du profil technique de référencement. |

L’élément **ValidationTechnicalProfile** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| ReferenceId | Oui | L’identificateur d’un profil technique déjà défini dans la stratégie ou dans la stratégie parente. |
|ContinueOnError|Non| Indique si la validation des profils techniques de validation suivants doit se poursuivre si ce profil technique de validation génère une erreur. Valeurs possibles : `true` ou `false` (par défaut, arrête le traitement des profils de validation suivants et renvoie une erreur). |
|ContinueOnSuccess | Non | Indique si la validation des profils de validation suivants doit se poursuivre si ce profil technique de validation réussit. Valeurs possibles : `true` ou `false`. La valeur par défaut est `true`, ce qui signifie que le traitement des profils de validation suivants continue. |

L’élément **ValidationTechnicalProfile** contient l’élément suivant :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| Preconditions | 0:1 | Une liste de conditions préalables qui doivent être satisfaites pour que le profil technique de validation puisse s’exécuter. |

L’élément **Precondition** contient l’attribut suivant :

| Attribut | Obligatoire | Description |
| --------- | -------- | ----------- |
| `Type` | Oui | Le type de vérification ou de requête à exécuter pour la condition préalable. Soit vous spécifiez `ClaimsExist` pour vous assurer que les actions seront effectuées si les revendications spécifiées existent dans l’ensemble de revendications actuel de l’utilisateur, soit vous spécifiez `ClaimEquals` pour que les actions soient exécutées si la revendication spécifiée existe et que sa valeur est égale à la valeur spécifiée. |
| `ExecuteActionsIf` | Oui | Indique si les actions de la condition préalable doivent être effectuées si le test est true ou false. |

L’élément **Precondition** contient les éléments suivants :

| Élément | Occurrences | Description |
| ------- | ----------- | ----------- |
| Valeur | 1:n | Les données qui sont utilisées par la vérification. Si le type de cette vérification est `ClaimsExist`, ce champ spécifie un ClaimTypeReferenceId à rechercher. Si le type de la vérification est `ClaimEquals`, ce champ spécifie un ClaimTypeReferenceId à rechercher. Un autre élément de valeur contient la valeur à vérifier.|
| Action | 1:1 | L’action à entreprendre si la vérification de condition préalable dans une étape d’orchestration a la valeur true. La valeur de l’**Action** est définie sur `SkipThisValidationTechnicalProfile`. Indique que le profil technique de validation associé ne doit pas être exécuté. |

### <a name="example"></a>Exemple

Les exemples suivants utilisent ces profils techniques de validation :

1. Le premier profil technique de validation vérifie les informations d’identification de l’utilisateur et ne continue pas si une erreur se produit, par exemple en cas de nom d’utilisateur non valide ou de mot de passe incorrect.
2. Le profil technique de validation suivant ne s’exécute pas si la revendication userType n’existe pas, ou si la valeur de userType est `Partner`. Le profil technique de validation tente de lire le profil utilisateur à partir de la base de données client interne et continue si une erreur se produit, par exemple si le service de l’API REST n’est pas disponible ou en cas d’erreur interne.
3. Le dernier profil technique de validation ne s’exécute pas si la revendication userType n’a pas existé, ou si la valeur de userType est `Customer`. Le profil technique de validation tente de lire le profil utilisateur à partir de la base de données partenaire interne et continue si une erreur se produit, par exemple si le service de l’API REST n’est pas disponible ou en cas d’erreur interne.

```xml
<ValidationTechnicalProfiles>
  <ValidationTechnicalProfile ReferenceId="login-NonInteractive" ContinueOnError="false" />
  <ValidationTechnicalProfile ReferenceId="REST-ReadProfileFromCustomertsDatabase" ContinueOnError="true" >
    <Preconditions>
      <Precondition Type="ClaimsExist" ExecuteActionsIf="false">
        <Value>userType</Value>
        <Action>SkipThisValidationTechnicalProfile</Action>
      </Precondition>
      <Precondition Type="ClaimEquals" ExecuteActionsIf="true">
        <Value>userType</Value>
        <Value>Partner</Value>
        <Action>SkipThisValidationTechnicalProfile</Action>
      </Precondition>
    </Preconditions>
  </ValidationTechnicalProfile>
  <ValidationTechnicalProfile ReferenceId="REST-ReadProfileFromPartnersDatabase" ContinueOnError="true" >
    <Preconditions>
      <Precondition Type="ClaimsExist" ExecuteActionsIf="false">
        <Value>userType</Value>
        <Action>SkipThisValidationTechnicalProfile</Action>
      </Precondition>
      <Precondition Type="ClaimEquals" ExecuteActionsIf="true">
        <Value>userType</Value>
        <Value>Customer</Value>
        <Action>SkipThisValidationTechnicalProfile</Action>
      </Precondition>
    </Preconditions>
  </ValidationTechnicalProfile>
</ValidationTechnicalProfiles>
```
