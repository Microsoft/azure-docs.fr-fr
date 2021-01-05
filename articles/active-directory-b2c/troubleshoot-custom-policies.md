---
title: Résoudre les problèmes liés aux stratégies personnalisées dans Azure Active Directory B2C
description: Découvrez les approches pour résoudre les erreurs lorsque vous utilisez des stratégies personnalisées dans Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: troubleshooting
ms.date: 08/13/2019
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 87ea77540e5fcaac0c4231403473d25ebae46aac
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94840251"
---
# <a name="troubleshoot-azure-ad-b2c-custom-policies-and-identity-experience-framework"></a>Résoudre les problèmes liés aux stratégies personnalisées Azure AD B2C et à l’Infrastructure d’expérience d’identité.

Si vous utilisez des stratégies personnalisées Azure Active Directory B2C (Azure AD B2C), vous risquez de rencontrer des difficultés lors de la configuration de l’Infrastructure d’expérience d’identité dans son format XML de langage de stratégie. Apprendre à écrire des stratégies personnalisées peut s’apparenter à l’apprentissage d’une nouvelle langue. Dans cet article, nous décrivons certains outils et proposons des conseils qui peuvent vous aider à identifier et résoudre les problèmes.

Cet article est axé sur la résolution des problèmes de configuration de stratégie personnalisée Azure AD B2C. Il ne traite pas de l’application par partie de confiance ou de sa bibliothèque d’identités.

## <a name="xml-editing"></a>Édition XML

L’erreur la plus courante lors de la configuration des stratégies personnalisées concerne la mise en forme du code XML. Un bon éditeur XML est quasiment essentiel. Il affiche le code XML en mode natif, applique un code couleur au contenu, prérenseigne les termes courants, maintient les éléments indexés et peut valider contre un schéma XML.

Deux de nos éditeurs favoris sont [Visual Studio Code](https://code.visualstudio.com/) et [Notepad++](https://notepad-plus-plus.org/).

La validation du schéma XML identifie les erreurs avant le chargement de votre fichier XML. Dans le dossier racine du [pack de démarrage](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack), obtenez la définition du schéma XML *TrustFrameworkPolicy_0.3.0.0.xsd*. Pour savoir comment utiliser le fichier de schéma XSD pour la validation dans votre éditeur, recherchez *Outils XML* et *Validation XML* ou similaire dans la documentation de l’éditeur.

Il peut être utile de passer en revue les règles XML. Azure AD B2C rejette toute erreur de mise en forme XML qu’il détecte. Parfois, une mauvaise mise en forme XML peut entraîner des messages d’erreur trompeurs.

## <a name="upload-policies-and-policy-validation"></a>Chargement et validation de stratégies

La validation du fichier de stratégie XML s’effectue automatiquement lors du chargement. La plupart des erreurs provoquent l’échec du chargement. La validation inclut le fichier de stratégie que vous chargez. Elle inclut également la chaîne de fichiers à laquelle le fichier de chargement fait référence (le fichier de stratégie de la partie de confiance, le fichier d’extensions et le fichier de base).

Voici quelques-unes des erreurs de validation courantes :

> Extrait de code de l’erreur :`...makes a reference to ClaimType with id "displayName" but neither the policy nor any of its base policies contain such an element`

* La valeur ClaimType est mal orthographiée ou n’existe pas dans le schéma.
* Les valeurs ClaimTypes doivent être définies dans au moins l’un des fichiers de la stratégie.
    Par exemple : `<ClaimType Id="issuerUserId">`
* Si la valeur ClaimType est définie dans le fichier d’extensions mais utilisée également dans une valeur TechnicalProfile du fichier de base, le chargement du fichier de base provoque une erreur.

> Extrait de code de l’erreur :`...makes a reference to a ClaimsTransformation with id...`

* Les causes de l’erreur peuvent être les mêmes que pour l’erreur ClaimType.

> Extrait de code de l’erreur :`Reason: User is currently logged as a user of 'yourtenant.onmicrosoft.com' tenant. In order to manage 'yourtenant.onmicrosoft.com', please login as a user of 'yourtenant.onmicrosoft.com' tenant`

* Vérifiez que l’élément TenantId dans les éléments `<TrustFrameworkPolicy\>` et `<BasePolicy\>` correspondent à votre client cible Azure AD B2C.

## <a name="troubleshoot-the-runtime"></a>Résoudre les problèmes au moment de l’exécution

* Utilisez **Exécuter maintenant** et `https://jwt.ms` pour tester vos stratégies indépendamment de votre application web ou mobile. Ce site web agit comme une application par partie de confiance. Il affiche le contenu du jeton JWT (JSON Web Token) généré par votre stratégie Azure AD B2C.

    Pour créer une application de test pouvant rediriger vers `https://jwt.ms` pour l’inspection des jetons :

    [!INCLUDE [active-directory-b2c-appreg-idp](../../includes/active-directory-b2c-appreg-idp.md)]

* Pour suivre l’échange de messages entre votre navigateur client et Azure AD B2C, utilisez [Fiddler](https://www.telerik.com/fiddler). Il peut vous aider à obtenir une indication de l’endroit où vos étapes d’orchestration échouent dans le parcours utilisateur.

* En [Mode de développement](troubleshoot-with-application-insights.md), utilisez **Application Insights** pour suivre l’activité de votre parcours utilisateur d’Infrastructure d’expérience d’identité. En **Mode de développement**, vous pouvez observer l'échange de revendications entre l'Infrastructure d'expérience d'identité et les divers fournisseurs de revendications définis par les profils techniques, tels que les fournisseurs d'identité, les services basés sur les API, le répertoire utilisateur Azure AD B2C et d'autres services comme Azure AD Multi-Factor-Authentication.

## <a name="recommended-practices"></a>Pratiques recommandées

**Conservez plusieurs versions de vos scénarios. Regroupez-les dans un projet avec votre application.** Les fichiers de base, d’extensions et de partie de confiance dépendent directement les uns des autres. Enregistrez-les en tant que groupe. À mesure que de nouvelles fonctionnalités sont ajoutées à vos stratégies, conservez des versions de travail distinctes. Organisez les versions de travail dans votre propre système de fichiers avec le code d’application avec lequel ils interagissent. Vos applications peuvent appeler de nombreuses stratégies de partie de confiance différentes dans un locataire. Elles peuvent devenir dépendantes envers les revendications qu’elles attendent de vos stratégies Azure AD B2C.

**Développez et testez des profils techniques avec des parcours utilisateur connus.** Utilisez des stratégies de pack de démarrage testées pour configurer vos profils techniques. Testez-les séparément avant de les incorporer à vos propres parcours utilisateur.

**Développez et testez des parcours utilisateur avec des profils techniques testés.** Changez les étapes d’orchestration d’un parcours utilisateur de manière incrémentielle. Générez progressivement vos scénarios prévus.

## <a name="next-steps"></a>Étapes suivantes

Offert dans GitHub, téléchargez l’archive .zip [active-directory-b2c-custom-policy-starterpack](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/archive/master.zip). Vous pouvez également cloner le référentiel :

```
git clone https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack
```
