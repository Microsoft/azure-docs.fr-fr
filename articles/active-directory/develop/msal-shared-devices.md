---
title: Vue d’ensemble du mode d’appareil partagé
titleSuffix: Microsoft identity platform | Azure
description: Découvrez le mode d’appareil partagé afin de permettre le partage d’appareil pour vos travailleurs en première ligne.
services: active-directory
author: brandwe
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 03/31/2020
ms.author: brandwe
ms.reviewer: brandwe
ms.custom: aaddev
ms.openlocfilehash: 3cd7074467332f89d4d6c60830be34f4e2a638c1
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94562081"
---
# <a name="overview-of-shared-device-mode"></a>Vue d’ensemble du mode d’appareil partagé

Le mode d’appareil partagé est une fonctionnalité Azure Active Directory qui vous permet de créer des applications prenant en charge les travailleurs en première ligne et d’activer le mode d’appareil partagé sur les appareils déployés pour ceux-ci.

>[!IMPORTANT]
> Cette fonctionnalité [!INCLUDE [PREVIEW BOILERPLATE](../../../includes/active-directory-develop-preview.md)]

## <a name="what-are-firstline-workers"></a>Que sont les travailleurs en première ligne ?

Les travailleurs en première ligne sont les employés en magasin, les agents de maintenance et sur le terrain, le personnel médical et autres utilisateurs qui ne restent pas assis devant un ordinateur et qui n’utilisent pas l’e-mail de l’entreprise pour collaborer. Les sections suivantes présentent les aspects et les défis liés à la prise en charge des travailleurs en première ligne, suivis d’une introduction aux fonctionnalités fournies par Microsoft qui permettent à votre application d’être utilisée par les travailleurs en première ligne d’une organisation.

### <a name="challenges-of-supporting-firstline-workers"></a>Défis liés à la prise en charge des travailleurs en première ligne

La mise en place des workflows des travailleurs en première ligne comprend des défis qui ne sont généralement pas présents avec les travailleurs de l’information standard. Ces défis peuvent inclure un taux de renouvellement élevé et une familiarité moindre avec les outils de productivité de base d’une organisation. Pour autonomiser leurs travailleurs en première ligne, les organisations adoptent des stratégies diverses. Certaines adoptent une stratégie BYOD (apportez votre propre appareil) dans laquelle les employés utilisent des applications métier sur leur téléphone personnel, tandis que d’autres fournissent à leurs employés des appareils partagés comme des iPads ou des tablettes Android.

### <a name="supporting-multiple-users-on-devices-designed-for-one-user"></a>Prise en charge de plusieurs utilisateurs sur des appareils conçus pour un seul utilisateur

Étant donné que les appareils mobiles qui exécutent iOS ou Android ont été conçus pour des utilisateurs individuels, la plupart des applications optimisent leur expérience d’utilisation pour un seul utilisateur. L’activation de l’authentification unique entre les applications et le maintien de la connexion des utilisateurs sur leur appareil s’inscrivent dans le cadre de cette expérience optimisée. Lorsqu’un utilisateur supprime son compte d’une application, l’application ne considère généralement pas cela comme un événement lié à la sécurité. De nombreuses applications conservent même les informations d’identification d’un utilisateur pour permettre une connexion rapide. Vous avez peut-être déjà rencontré ce problème en supprimant une application de votre appareil mobile, puis en la réinstallant pour constater que vous êtes toujours connecté.

### <a name="global-sign-in-and-sign-out-sso"></a>Connexion et déconnexion (SSO) globales

Pour permettre aux employés d’une organisation d’utiliser ses applications sur un pool d’appareils partagés par ces employés, les développeurs doivent activer l’expérience inverse. Les employés doivent être en mesure de choisir un appareil dans le pool et d’effectuer un mouvement unique pour « se l’approprier » pendant toute leur journée de travail. À la fin de la journée, ils doivent être en mesure d’effectuer un autre geste pour se déconnecter globalement sur l’appareil et voir la suppression de toutes leurs informations personnelles et d’entreprise, afin de pouvoir retourner l’appareil dans le pool d’appareils. De plus, si un employé oublie de se déconnecter, l’appareil doit effectuer une déconnexion automatique à la fin de sa journée de travail et/ou après une période d’inactivité.

Azure Active Directory permet ces scénarios avec une fonctionnalité appelée **mode d’appareil partagé**.

## <a name="introducing-shared-device-mode"></a>Présentation du mode d’appareil partagé

Comme nous l’avons vu, le mode d’appareil partagé est une fonctionnalité Azure Active Directory qui vous permet d’effectuer les opérations suivantes :

* Créer des applications qui prennent en charge les travailleurs en première ligne
* Déployer des appareils pour les travailleurs en première ligne et activer le mode d’appareil partagé

### <a name="build-applications-that-support-firstline-workers"></a>Créer des applications qui prennent en charge les travailleurs en première ligne

Vous pouvez prendre en charge les travailleurs en première ligne dans vos applications à l’aide de la bibliothèque d’authentification Microsoft (MSAL) et de l’[application Microsoft Authenticator](../user-help/user-help-auth-app-overview.md) pour activer un état d’appareil appelé *mode d’appareil partagé*. Quand un appareil est en mode d’appareil partagé, Microsoft fournit à votre application des informations pour lui permettre de modifier son comportement en fonction de l’état de l’utilisateur sur l’appareil, en protégeant les données utilisateur.

Fonctionnalités prises en charge :

* **Connexion d’un utilisateur au niveau de l’appareil** via n’importe quelle application prise en charge.
* **Déconnexion d’un utilisateur au niveau de l’appareil** via n’importe quelle application prise en charge.
* **Requête de l’état de l’appareil** pour déterminer si votre application se trouve sur un appareil qui est en mode d’appareil partagé.
* **Requête de l’état de l’appareil de l’utilisateur** sur l’appareil pour déterminer si quelque chose a changé depuis la dernière utilisation de votre application.

La prise en charge du mode d’appareil partagé doit être considérée comme une amélioration de la sécurité et une mise à niveau des fonctionnalités pour votre application. Elle peut aider à augmenter son adoption dans des environnements hautement réglementés tels que la santé et la finance.

Vos utilisateurs dépendent de vous et vous devez leur garantir que leurs données ne seront pas divulguées à un autre utilisateur. Le mode de partage d’appareil fournit des signaux utiles pour indiquer à votre application qu’une modification que vous devez gérer s’est produite. Votre application est chargée de vérifier l’état de l’utilisateur sur l’appareil chaque fois que l’application est utilisée, en effaçant les données de l’utilisateur précédent. Cela inclut l’hypothèse où il est rechargé à partir de l’arrière-plan en mode multitâche. En cas de changement d’utilisateur, vous devez vous assurer que les données de l’utilisateur précédent sont effacées et que toutes les données en mémoire cache affichées dans votre application sont supprimées. Nous vous recommandons de toujours effectuer un processus de révision de sécurité approfondi après avoir ajouté la fonctionnalité de mode d’appareil partagé à votre application.

Pour plus d’informations sur la façon de modifier vos applications pour prendre en charge le mode d’appareil partagé, consultez la section [Étapes suivantes](#next-steps) à la fin de cet article.

### <a name="deploy-devices-to-firstline-workers-and-turn-on-shared-device-mode"></a>Déployer des appareils pour les travailleurs en première ligne et activer le mode d’appareil partagé

Une fois que vos applications prennent en charge le mode d’appareil partagé et incluent les données et les modifications de sécurité requises, vous pouvez les publier comme étant utilisables par les travailleurs en première ligne.

Les administrateurs d’appareils d’une organisation peuvent déployer leurs appareils et vos applications dans leurs magasins et lieux de travail via une solution de gestion des appareils mobiles (MDM) comme Microsoft Intune. Une partie du processus de configuration consiste à marquer l’appareil en tant qu’*appareil partagé*. Les administrateurs configurent le mode d’appareil partagé en déployant l’[application Microsoft Authenticator](../user-help/user-help-auth-app-overview.md) et en définissant le mode d’appareil partagé par le biais des paramètres de configuration. Après avoir effectué ces étapes, toutes les applications qui prennent en charge le mode d’appareil partagé utilisent l’application Microsoft Authenticator pour gérer son état d’utilisateur et fournir des fonctionnalités de sécurité pour l’appareil et l’organisation.

## <a name="next-steps"></a>Étapes suivantes

Nous prenons en charge les plateformes iOS et Android pour le mode d’appareil partagé. Consultez la documentation ci-dessous pour que votre plateforme commence à prendre en charge les travailleurs en première ligne dans vos applications.

* [Prise en charge du mode d’appareil partagé pour iOS](msal-ios-shared-devices.md)
* [Prise en charge du mode d’appareil partagé pour Android](msal-android-shared-devices.md)
