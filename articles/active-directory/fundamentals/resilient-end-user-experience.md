---
title: Expérience résiliente de l’utilisateur final à l’aide d’Azure AD B2C | Microsoft Docs
description: Méthodes pour renforcer la résilience de l’expérience des utilisateurs finaux à l’aide d’Azure AD B2C
services: active-directory
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: how-to
author: gargi-sinha
ms.author: gasinh
manager: martinco
ms.reviewer: ''
ms.date: 11/30/2020
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4c695466fbd50435a85c63842ceb50ce80765760
ms.sourcegitcommit: 8c3a656f82aa6f9c2792a27b02bbaa634786f42d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97630290"
---
# <a name="resilient-end-user-experience"></a>Expérience résiliente de l’utilisateur final

L’expérience d’inscription et de connexion de l’utilisateur final est constituée des éléments suivants :

- Les interfaces avec lesquelles l’utilisateur interagit (par exemple, CSS, HTML et JavaScript)

- Les flux d’utilisateurs et les stratégies personnalisées que vous créez, telles que l’inscription, la connexion et la modification du profil

- Les fournisseurs d’identité (IDP) de votre application, tels que le nom d’utilisateur/mot de passe du compte local, Outlook, Facebook et Google

![Image montrant les composants de l’expérience de l’utilisateur final](media/resilient-end-user-experiences/end-user-experience-architecture.png)

## <a name="choose-between-user-flow-and-custom-policy"></a>Choisir entre un flux d’utilisateurs et une stratégie personnalisée  

Pour vous aider à configurer les tâches d’identité les plus courantes, Azure AD B2C fournit des [flux d’utilisateurs](https://docs.microsoft.com/azure/active-directory-b2c/user-flow-overview) configurables intégrés. Vous pouvez également créer vos propres [stratégies personnalisées](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-overview), ce qui vous offre un maximum de flexibilité. Toutefois, il est recommandé d’utiliser des stratégies personnalisées uniquement pour répondre à des scénarios complexes.

### <a name="how-to-decide-between-user-flow-and-custom-policy"></a>Choix entre un flux d’utilisateurs et une stratégie personnalisée

Choisissez des flux d’utilisateurs intégrés si les besoins de votre entreprise peuvent ainsi être satisfaits. Dans la mesure où ils ont été largement testés par Microsoft, vous pouvez réduire au minimum les tests nécessaires à la validation des fonctions, des performances ou de l’échelle de ces flux d’utilisateurs d’identité au niveau des stratégies. Vous devez toujours tester vos applications sur le plan des fonctionnalités, des performances et de la mise à l’échelle.

Si vous [choisissez des stratégies personnalisées](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-get-started) en raison des exigences de votre entreprise, veillez à effectuer des tests au niveau des stratégies pour les fonctions, les performances ou l’échelle en plus des tests au niveau des applications.

Reportez-vous à l’article qui [compare les flux d’utilisateurs et les stratégies personnalisées](https://docs.microsoft.com/azure/active-directory-b2c/custom-policy-overview#comparing-user-flows-and-custom-policies) pour vous aider à décider.

## <a name="choose-multiple-idps"></a>Choisir plusieurs IDP

Lorsque vous utilisez un [fournisseur d’identité externe](https://docs.microsoft.com/azure/active-directory-b2c/technical-overview#external-identity-providers) tel que Facebook, assurez-vous de disposer d’un plan de secours au cas où le fournisseur externe deviendrait indisponible.

### <a name="how-to-set-up-multiple-idps"></a>Configuration de plusieurs IDP

Dans le cadre du processus d’inscription du fournisseur d’identité externe, incluez une revendication d’identité vérifiée, telle que le numéro de téléphone portable ou l’adresse e-mail de l’utilisateur. Validez les revendications vérifiées sur l’instance de répertoire Azure AD B2C sous-jacente. Si le fournisseur externe n’est pas disponible, restaurez la revendication d’identité vérifiée et revenez au numéro de téléphone comme méthode d’authentification. Une autre option consiste à envoyer à l’utilisateur un code secret à usage unique pour lui permettre de se connecter.

 Procédez comme suit pour [créer d’autres chemins d’authentification](https://github.com/azure-ad-b2c/samples/tree/master/policies/idps-filter) :

 1. Configurez votre stratégie d’inscription pour permettre l’inscription par compte local et par des IDP externes.

 2. Configurez une stratégie de profil pour permettre aux utilisateurs de [relier l’autre identité à leur compte](https://github.com/Azure-Samples/active-directory-b2c-advanced-policies/tree/master/account-linking) après s’être connectés.

 3. Informez les utilisateurs et autorisez-les à [basculer sur un autre IDP](https://docs.microsoft.com/azure/active-directory-b2c/customize-ui-with-html#configure-dynamic-custom-page-content-uri) lors d’une panne.

## <a name="availability-of-multi-factor-authentication"></a>Disponibilité de l’authentification multifacteur

Lorsque vous utilisez un [service téléphonique pour l’authentification multifacteur (MFA)](https://docs.microsoft.com/azure/active-directory-b2c/phone-authentication), veillez à envisager un autre fournisseur de services. L’opérateur téléphonique local ou le fournisseur de services téléphoniques peut connaître des perturbations dans son service.

### <a name="how-to-choose-an-alternate-mfa"></a>Choix d’une autre authentification multifacteur  

Le service Azure AD B2C utilise un fournisseur MFA par téléphone intégré pour fournir des codes secrets à usage unique (OTP). Il se présente sous la forme d’un appel vocal et d’un SMS adressé au numéro de téléphone préenregistré de l’utilisateur. Les autres méthodes suivantes sont disponibles pour différents scénarios :

Lorsque vous utilisez des **flux d’utilisateurs**, il existe deux méthodes pour renforcer la résilience :

- **Modifiez la configuration des flux d’utilisateurs** :  Lors de la détection d’une interruption dans la livraison OTP par téléphone, modifiez la méthode de livraison OTP du téléphone à l’adresse e-mail et redéployez le flux d’utilisateurs, en laissant les applications inchangées.

![Capture d’écran montrant l’inscription et la connexion](media/resilient-end-user-experiences/create-sign-in.png)

- **Modifiez les applications** : Pour chaque tâche d’identité, telle que l’inscription et la connexion, définissez deux ensembles de flux d’utilisateurs. Configurez le premier ensemble pour utiliser l’OTP par téléphone et le second pour l’OTP par adresse e-mail. Lors de la détection d’une interruption dans la livraison OTP par téléphone, modifiez et redéployez les applications pour passer du premier ensemble de flux d’utilisateurs au second, en laissant les flux d’utilisateurs inchangés.  

Lorsque vous utilisez des **stratégies personnalisées**, il existe quatre méthodes pour renforcer la résilience. La liste ci-dessous est classée par ordre de complexité et vous devrez redéployer les stratégies mises à jour.

- **Permettez à l’utilisateur de sélectionner soit l’OTP par téléphone, soit l’OTP par adresse e-mail** : Exposez les deux options aux utilisateurs et permettez-leur de sélectionner eux-mêmes l’une des options. Il n’est pas nécessaire d’apporter des modifications aux stratégies ou aux applications.

- **Basculez dynamiquement entre l’OTP par téléphone et l’OTP par adresse e-mail** :  Collectez le numéro de téléphone et l’adresse e-mail lors de l’inscription. Définissez à l’avance une stratégie personnalisée pour basculer, sous conditions, d’une livraison OTP par téléphone à une livraison par adresse e-mail lors d’une interruption du service téléphonique. Il n’est pas nécessaire d’apporter des modifications aux stratégies ou aux applications.

- **Utilisez une application Authenticator** : Mettez à jour la stratégie personnalisée pour utiliser une [application Authenticator](https://github.com/azure-ad-b2c/samples/tree/master/policies/custom-mfa-totp). Si votre authentification multifacteur normale utilise un OTP par téléphone ou par adresse e-mail, redéployez vos stratégies personnalisées pour passer à l’utilisation de l’application Authenticator.

>[!Note]
>Les utilisateurs doivent configurer l’intégration de l’application Authenticator lors de l’inscription.

- **Utilisez des questions de sécurité** : Si aucune des méthodes ci-dessus n’est applicable, implémentez des questions de sécurité comme solution de secours. Configurez des questions de sécurité pour les utilisateurs pendant l’intégration ou la modification du profil et stockez les réponses dans une base de données distincte de celle du répertoire. Cette méthode ne répond pas à l’exigence de l’authentification multifacteur de « quelque chose que vous avez » (par exemple, un téléphone), mais propose une information « quelque chose que vous savez » secondaire.

## <a name="use-a-content-delivery-network"></a>Utiliser un réseau de distribution de contenu

Les réseaux de distribution de contenu (CDN) sont plus performants et moins onéreux que les magasins de blobs pour le stockage de l’IU personnalisée des flux d’utilisateurs. Le contenu des pages web est diffusé plus rapidement à partir d’un réseau géographiquement distribué de serveurs à haute disponibilité.  

Testez régulièrement la disponibilité de votre CDN et les performances de la distribution de contenu par le biais de tests de scénario de bout en bout et de tests de charge. Si vous prévoyez une augmentation de la demande en raison du trafic induit par une promotion ou des vacances, modifiez vos estimations de test de charge.
  
## <a name="next-steps"></a>Étapes suivantes

- [Ressources de résilience pour les développeurs Azure AD B2C](resilience-b2c.md)
  
  - [Interfaces résilientes avec processus externes](resilient-external-processes.md)
  - [Résilience grâce aux meilleures pratiques des développeurs](resilience-b2c-developer-best-practices.md)
  - [Résilience par la surveillance et l’analytique](resilience-with-monitoring-alerting.md)
- [Renforcer la résilience de votre infrastructure d’authentification](resilience-in-infrastructure.md)
- [Augmenter la résilience de l’authentification et de l’autorisation dans vos applications](resilience-app-development-overview.md)
