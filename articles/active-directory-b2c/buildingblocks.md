---
title: BuildingBlocks
titleSuffix: Azure AD B2C
description: Spécifiez l’élément BuildingBlocks d’une stratégie personnalisée dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 12/10/2019
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 0170877995573bdfcb13ebc1c0387bed0893deac
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85201223"
---
# <a name="buildingblocks"></a>BuildingBlocks

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

L’élément **BuildingBlocks** est ajouté à l’intérieur de l’élément [TrustFrameworkPolicy](trustframeworkpolicy.md).

```xml
<TrustFrameworkPolicy
  xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
  xmlns:xsd="https://www.w3.org/2001/XMLSchema"
  xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
  PolicySchemaVersion="0.3.0.0"
  TenantId="mytenant.onmicrosoft.com"
  PolicyId="B2C_1A_TrustFrameworkBase"
  PublicPolicyUri="http://mytenant.onmicrosoft.com/B2C_1A_TrustFrameworkBase">

  <BuildingBlocks>
    <ClaimsSchema>
      ...
    </ClaimsSchema>
    <Predicates>
    ...
    </Predicates>
    <PredicateValidations>
    ...
    </PredicateValidations>
    <ClaimsTransformations>
      ...
    </ClaimsTransformations>
    <ContentDefinitions>
      ...
    </ContentDefinitions>
    <Localization>
      ...
    </Localization>
    <DisplayControls>
      ...
    </DisplayControls>
 </BuildingBlocks>
```

L’élément **BuildingBlocks** contient les éléments suivants qui doivent être spécifiés dans l’ordre défini :

- [ClaimsSchema](claimsschema.md) : définit les types de revendications qui peuvent être référencés dans le cadre de la stratégie. Le schéma de revendications est l’endroit où vous déclarez vos types de revendications. Un type de revendication est similaire à une variable dans de nombreux langages de programmation. Vous pouvez utiliser le type de revendication pour recueillir des données à partir de l’utilisateur de votre application, recevoir des revendications à partir de fournisseurs d’identité sociale, envoyer et recevoir des données à partir d’une API REST personnalisée, ou stocker des données internes utilisées par votre stratégie personnalisée.

- [Predicates et PredicateValidationsInput](predicates.md) : vous permettent d’effectuer un processus de validation afin de vous assurer que seules des données correctement formées sont entrées dans une revendication.

- [ClaimsTransformations](claimstransformations.md) : contient une liste des transformations de revendications qui peuvent être utilisées dans votre stratégie.  Une transformation de revendication convertit une revendication en une autre. Dans la transformation de revendication, vous spécifiez une méthode de transformation, telle que :
  - Remplacement de la casse d’une revendication de chaîne par celle spécifiée. Par exemple, changement d’une chaîne de minuscules en majuscules.
  - Comparaison de deux revendications et retour d’une revendication avec la valeur true indiquant que les revendications correspondent, ou false dans le cas contraire.
  - Création d’une revendication de chaîne à partir du paramètre fourni dans la stratégie.
  - Création d’une chaîne aléatoire à l’aide du générateur de nombres aléatoires.
  - Mise en forme d’une revendication en fonction de la chaîne de format fournie. Cette transformation utilise la méthode C# `String.Format`.

- InputValidation – Cet élément vous permet d’effectuer des agrégations booléennes similaires à *et* et *ou*.

- [ContentDefinitions](contentdefinitions.md) : contient des URL pour les modèles HTML5 à utiliser dans votre parcours utilisateur. Dans une stratégie personnalisée, une définition de contenu définit l’URI de page HTML5 qui est utilisée pour une étape spécifiée dans le parcours utilisateur. Il peut s’agir, par exemple, de pages de connexion ou d’inscription, de réinitialisation de mot de passe ou d’erreur. Vous pouvez modifier l’apparence en remplaçant l’URI LoadUri du fichier HTML5. Vous pouvez aussi créer des définitions de contenu en fonction de vos besoins. Cet élément peut contenir une référence de ressources localisées à l’aide d’un ID de localisation.

- [Localization](localization.md) : vous permet de prendre en charge plusieurs langues. La prise en charge de la localisation dans les stratégies vous permet de configurer la liste des langues prises en charge dans une stratégie et de choisir une langue par défaut. Les collections et les chaînes propres à une langue sont également prises en charge.

- [DisplayControls](display-controls.md) : définit les contrôles à afficher sur une page. Les contrôles d’affichage ont des fonctionnalités spéciales et interagissent avec les profils techniques de validation de serveur principal. Les contrôles d’affichage sont actuellement en **préversion**.
