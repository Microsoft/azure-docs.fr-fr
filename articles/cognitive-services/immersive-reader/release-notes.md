---
title: Notes de publication du SDK du Lecteur immersif
titleSuffix: Azure Cognitive Services
description: Découvrez les nouveautés du SDK du Lecteur immersif.
services: cognitive-services
author: dylankil
manager: guillasi
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: reference
ms.date: 10/12/2020
ms.author: dylankil
ms.openlocfilehash: 6b041916c6fa446732e95d49391d9ead63eb1b17
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93135457"
---
# <a name="immersive-reader-javascript-sdk-release-notes"></a>Notes de publication du SDK JavaScript du Lecteur immersif

## <a name="version-110"></a>Version 1.1.0

Cette version contient de nouvelles fonctionnalités, des correctifs de vulnérabilités de sécurité, des correctifs de bogues, des mises à jour des exemples de code et des options de configuration.

#### <a name="new-features"></a>Nouvelles fonctionnalités

* Activation de l’enregistrement et du chargement des préférences utilisateur dans différents navigateurs et appareils
* Activation de la configuration des options d’affichage par défaut
* Ajout d’une option pour définir la langue de traduction, activation de la traduction de mots et activation de la traduction de documents lors du lancement du Lecteur immersif
* Ajout de la prise en charge de la configuration de la lecture à voix haute par le biais d’options
* Ajout de la possibilité de désactiver l’expérience de première exécution
* Ajout d’ImmersiveReaderView pour la plateforme UWP

#### <a name="improvements"></a>Améliorations

* Mise à jour du HTML de l’exemple de code Android pour utiliser le SDK le plus récent
* Mise à jour de la réponse de lancement pour retourner le nombre de caractères traités
* Mise à jour des exemples de code pour utiliser v1.1.0
* Interdiction d’appeler launchAsync lorsque le chargement est déjà en cours
* Vérification du contenu non valide en ignorant les messages dans lesquels les données ne sont pas une chaîne
* Wrapping de l’appel dans la fenêtre dans une clause if pour vérifier la prise en charge par le navigateur des promesses

#### <a name="fixes"></a>Correctifs

* Correction de dependabot en supprimant yarn.lock de gitignore
* Correction d’une vulnérabilité de sécurité en mettant à niveau pug vers v3.0.0 dans l’exemple de code quickstart-nodejs
* Correction de plusieurs vulnérabilités de sécurité en mettant à niveau les packages Jest et TypeScript
* Correction d’une vulnérabilité de sécurité en mettant à niveau Microsoft.IdentityModel.Clients.ActiveDirectory vers v5.2.0

<br>

## <a name="version-100"></a>Version 1.0.0

Cette version contient des changements cassants, de nouvelles fonctionnalités, des améliorations des exemples de code et des correctifs de bogues.

#### <a name="breaking-changes"></a>Changements cassants

* Sous-domaine et jeton Azure AD obligatoires, et jetons utilisés dans les versions précédentes dépréciés.
* Définition de CookiePolicy sur Désactivé. La conservation des préférences utilisateur est désactivée par défaut. Le Lecteur démarre à chaque fois avec les paramètres par défaut, sauf si CookiePolicy est défini sur Activé.

#### <a name="new-features"></a>Nouvelles fonctionnalités

* Ajout de la prise en charge pour activer ou désactiver les cookies
* Ajout d’un exemple de code de démarrage rapide Android Kotlin
* Ajout d’un exemple de code de démarrage rapide Android Java
* Ajout d’un exemple de code de démarrage rapide Node.js

#### <a name="improvements"></a>Améliorations

* Mise à jour du README.md avancé Node.js
* Changement de l’exemple de code Python de « Avancé » en « Démarrage rapide »
* Déplacement de l’exemple de code iOS Swift vers js/samples
* Mise à jour des exemples de code pour utiliser v1.0.0

#### <a name="fixes"></a>Correctifs

* Correctif pour l’exemple de code avancé Node.js
* Ajout de fichiers manquants pour advanced-csharp-multiple-resources
* Suppression de en-US dans les liens hypertexte

<br>

## <a name="version-003"></a>Version 0.0.3

Cette version contient de nouvelles fonctionnalités, des améliorations apportées aux exemples de code, des correctifs de vulnérabilités de sécurité et des correctifs de bogues.

#### <a name="new-features"></a>Nouvelles fonctionnalités

* Ajout d’un exemple de code iOS Swift
* Ajout d’un exemple de code avancé C# illustrant l’utilisation de plusieurs ressources 
* Ajout de la prise en charge pour désactiver la fonctionnalité de basculement en plein écran
* Ajout de la prise en charge pour masquer le bouton de fermeture de l’application du Lecteur immersif
* Ajout d’une fonction de rappel qui peut être utilisée par l’application hôte lors de la fermeture du Lecteur immersif
* Mise à jour des exemples de code pour utiliser l’authentification Azure Active Directory

#### <a name="improvements"></a>Améliorations

* Mise à jour de l’exemple de code avancé C# pour inclure un document Word
* Mise à jour des exemples de code pour utiliser v0.0.3

#### <a name="fixes"></a>Correctifs

* Mise à niveau de lodash vers la version 4.17.14 pour résoudre une vulnérabilité de sécurité
* Mise à jour de la bibliothèque MSAL C# pour résoudre une vulnérabilité de sécurité
* Mise à niveau de mixin-deep vers la version 1.3.2 pour résoudre une vulnérabilité de sécurité
* Mise à niveau de jest, webpack et webpack-cli, qui utilisaient des versions vulnérables de set-value et mixin-deep, afin de corriger une vulnérabilité de sécurité

<br>

## <a name="version-002"></a>Version 0.0.2

Cette version contient de nouvelles fonctionnalités, des améliorations apportées aux exemples de code, des correctifs de vulnérabilités de sécurité et des correctifs de bogues.

#### <a name="new-features"></a>Nouvelles fonctionnalités

* Ajout d’un exemple de code avancé Python
* Ajout d’un exemple de code de démarrage rapide Java
* Ajout d’un exemple de code simple

#### <a name="improvements"></a>Améliorations

* Renommage de resourceName en cogSvcsSubdomain
* Déplacement des secrets en dehors du code et utilisation de variables d’environnement
* Mise à jour des exemples de code pour utiliser v0.0.2

#### <a name="fixes"></a>Correctifs

* Correction de bogues d’accessibilité du bouton Lecteur immersif
* Correction du défilement rompu
* Mise à niveau du package handlebars vers la version 4.1.2 pour résoudre une vulnérabilité de sécurité
* Correction de bogues dans les tests unitaires du SDK
* Correction de bogues de compatibilité JavaScript Internet Explorer 11
* Mise à jour des URL du SDK

<br>

## <a name="version-001"></a>Version 0.0.1

Publication initiale du SDK JavaScript du Lecteur immersif.

* Ajout du SDK JavaScript du Lecteur immersif
* Ajout de la prise en charge pour spécifier la langue de l’interface utilisateur
* Ajout d’un délai d’expiration pour déterminer quand la fonction launchAsync doit échouer avec une erreur de délai d’expiration
* Ajout de la prise en charge pour spécifier l’index z de l’iframe du Lecteur immersif
* Ajout de la prise en charge pour utiliser une balise webview à la place d’un iframe, à des fins de compatibilité avec les applications Chrome
* Ajout de tests unitaires au SDK
* Ajout d’un exemple de code avancé Node.js
* Ajout d’un exemple de code avancé C#
* Ajout d’un exemple de code de démarrage rapide C#
* Ajout d’une configuration de package, de Yarn et d’autres fichiers de build
* Ajout de fichiers de configuration git
* Ajout de fichiers README.md aux exemples de code et au SDK
* Ajout d’une licence MIT
* Ajout d’instructions pour contributeur
* Ajout de ressources SVG au bouton d’icône statique

## <a name="next-steps"></a>Étapes suivantes

Prise en main du lecteur immersif :

* Lire les [Informations de référence sur la bibliothèque de client Lecteur immersif](./reference.md)
* Explorer la [bibliothèque de client Lecteur immersif sur GitHub](https://github.com/microsoft/immersive-reader-sdk)
