---
title: Comprendre la gestion des fonctionnalités avec Azure App Configuration
description: Activer et désactiver des fonctionnalités avec Azure App Configuration
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.custom: devx-track-dotnet
ms.topic: conceptual
ms.date: 02/20/2020
ms.openlocfilehash: cfd676092bcaede58909a3ec1eefeabb4c80f86b
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96930226"
---
# <a name="feature-management-overview"></a>Vue d’ensemble de la gestion des fonctionnalités

Généralement, fournir une nouvelle fonctionnalité d’application nécessite un redéploiement complet de l’application elle-même. Le test d’une fonctionnalité nécessite souvent plusieurs déploiements d’une même application.  Chaque déploiement peut modifier la fonctionnalité ou exposer la fonctionnalité à différents clients à des fins de test.  

La gestion des fonctionnalités est une pratique de développement de logiciels moderne qui dissocie la publication des fonctionnalités et le déploiement du code. De plus, elle permet de changer rapidement la disponibilité des fonctionnalités à la demande. Elle utilise une technique nommée *indicateurs de fonctionnalités* (également appelée *bascules de fonctionnalité* ou *commutateurs de fonctionnalité*, etc.) pour gérer dynamiquement le cycle de vie d’une fonctionnalité.

La gestion des fonctionnalités permet aux développeurs de résoudre les problèmes suivants :

* **Gestion des branches de code** : Utilisez des indicateurs de fonctionnalités pour wrapper les nouvelles fonctionnalités d’application qui sont en cours de développement. Ces fonctionnalités sont « masquées » par défaut. Bien qu’elles ne soient pas terminées, vous pouvez les fournir sans problème. Elles restent dormantes en production. À l’aide de cette approche, appelée *déploiement masqué (dark)* , vous pouvez fournir l’ensemble de votre code à la fin de chaque cycle de développement. Vous n’avez plus besoin de gérer des branches de code sur plusieurs cycles de développement parce qu’une fonctionnalité donnée a besoin de plus d’un cycle pour se terminer.
* **Test en production** : Utilisez des indicateurs de fonctionnalités pour accorder un accès en avant-première aux nouvelles fonctionnalités en production. Par exemple, vous pouvez limiter l’accès aux membres de votre équipe ou à des bêta-testeurs internes. Ces utilisateurs obtiendront l’expérience de production de fidélité optimale au lieu d’une expérience simulée ou partielle dans un environnement de test.
* **Distribution de version d’évaluation** : Utilisez des indicateurs de fonctionnalités pour déployer de façon incrémentielle de nouvelles fonctionnalités auprès des utilisateurs finaux. Vous pouvez d’abord cibler un petit pourcentage de votre population d’utilisateurs, puis augmenter progressivement ce pourcentage au fil du temps.
* **Interrupteur instantané** : Les indicateurs de fonctionnalités offrent un dispositif de sécurité inhérent pour la publication de nouvelles fonctionnalités. Vous pouvez activer et désactiver les fonctionnalités de l’application sans redéployer de code. Si nécessaire, vous pouvez rapidement désactiver une fonctionnalité sans regénérer et redéployer votre application.
* **Activation sélective** : Utilisez ces indicateurs de fonctionnalités pour segmenter vos utilisateurs et fournir un ensemble de fonctionnalités à chaque groupe. Vous pouvez avoir une fonctionnalité qui fonctionne seulement sur un navigateur web donné. Vous pouvez définir un indicateur de fonctionnalité de sorte que seuls les utilisateurs de ce navigateur peuvent voir et utiliser la fonctionnalité. Avec cette approche, vous pouvez facilement étendre par la suite la liste des navigateurs pris en charge sans avoir à changer le code.

## <a name="basic-concepts"></a>Concepts de base

Voici plusieurs nouveaux termes liés à la gestion des fonctionnalités :

* **Indicateur de fonctionnalités** : Un indicateur de fonctionnalité est une variable dont l’état binaire est *activé* ou *désactivé*. L’indicateur de fonctionnalité a également un bloc de code associé. L’état de l’indicateur de fonctionnalité détermine si le bloc de code s’exécute.
* **Gestionnaire de fonctionnalités** : Un gestionnaire de fonctionnalités est un package d’application qui gère le cycle de vie de tous les indicateurs de fonctionnalités dans une application. Il fournit également des fonctionnalités supplémentaires comme la mise en cache des indicateurs de fonctionnalités et la mise à jour de leur état.
* **Filtre** : Un filtre est une règle destinée à évaluer l’état d’un indicateur de fonctionnalité. Les filtres peuvent être basés sur des groupes d’utilisateurs, des types d’appareils ou de navigateurs, des emplacements géographiques et des fenêtres de temps.

Une implémentation efficace de la gestion des fonctionnalités est constituée d’au moins deux composants qui fonctionnent de concert :

* Une application qui utilise les indicateurs de fonctionnalités.
* Un référentiel distinct qui stocke les indicateurs de fonctionnalités et leur état actuel.

## <a name="using-feature-flags-in-your-code"></a>Utilisation des indicateurs de fonctionnalités dans votre code

Le modèle de base pour l’implémentation d’indicateurs de fonctionnalités dans une application est simple. Un indicateur de fonctionnalité est une variable d’état booléenne qui permet de contrôler une instruction conditionnelle dans votre code :

```csharp
if (featureFlag) {
    // Run the following code
}
```

Vous pouvez définir la valeur de `featureFlag` de manière statique.

```csharp
bool featureFlag = true;
```

Vous pouvez évaluer l’état de l’indicateur selon certaines règles :

```csharp
bool featureFlag = isBetaUser();
```

Vous pouvez étendre l’instruction conditionnelle afin de définir le comportement de l’application pour l’un des états :

```csharp
if (featureFlag) {
    // This following code will run if the featureFlag value is true
} else {
    // This following code will run if the featureFlag value is false
}
```

## <a name="feature-flag-repository"></a>Référentiel d’indicateurs de fonctionnalités

Pour utiliser efficacement les indicateurs de fonctionnalités, vous devez externaliser tous ceux qui sont utilisés dans une application. Cela vous permet de changer l’état des indicateurs de fonctionnalités sans modifier puis redéployer l’application.

Azure App Configuration fournit un référentiel centralisé pour les indicateurs de fonctionnalités. Vous pouvez l’utiliser pour définir différents types d’indicateurs de fonctionnalités et manipuler leur état de façon rapide et sûre. Vous pouvez ensuite utiliser les bibliothèques App Configuration pour différents frameworks de langage de programmation pour accéder facilement à ces indicateurs de fonctionnalités à partir de votre application.

La rubrique [Utiliser des indicateurs de fonctionnalités dans une application ASP.NET Core](./use-feature-flags-dotnet-core.md) montre comment le fournisseur App Configuration .NET Core et les bibliothèques de gestion des fonctionnalités sont utilisés ensemble pour implémenter des indicateurs de fonctionnalités pour votre application web ASP.NET.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Ajouter des indicateurs de fonctionnalités dans une application ASP.NET Core](./quickstart-feature-flag-aspnet-core.md)  
