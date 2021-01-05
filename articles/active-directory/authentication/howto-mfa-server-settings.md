---
title: Configurer le serveur MFA - Azure Active Directory
description: Découvrir comment configurer les paramètres de Serveur Azure MFA dans le Portail Azure
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 06/05/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 53763e62e8d7aa5b77ae451ef6427c1e09bff9f0
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96742695"
---
# <a name="configure-mfa-server-settings"></a>Configurer les paramètres du Serveur MFA

Cet article vous explique comment gérer les paramètres du Serveur Azure MFA dans le Portail Azure.

> [!IMPORTANT]
> À compter du 1er juillet 2019, Microsoft ne propose plus MFA Server pour les nouveaux déploiements. Les nouveaux clients qui souhaitent imposer une authentification multifacteur à leurs utilisateurs doivent utiliser la fonctionnalité Azure AD Multi-Factor Authentication basée sur le cloud. Les clients existants qui ont activé MFA Server avant le 1er juillet peuvent télécharger la dernière version et les futures mises à jour, et générer des informations d’identification d’activation comme d’habitude.

Les paramètres du serveur MFA par défaut suivants sont disponibles :

| Fonctionnalité | Description |
| ------- | ----------- |
| Paramètres du serveur | Télécharge MFA Server et génère des informations d’identification d’activation pour initialiser votre environnement |
| [Contournement à usage unique](#one-time-bypass) | Autorisez un utilisateur à s’authentifier sans effectuer d’authentification multifacteur pendant une durée limitée. |
| [Règles de mise en cache](#caching-rules) |  La mise en cache s’utilise principalement lorsque des systèmes locaux, comme un VPN, envoient plusieurs demandes de vérification alors que la première demande est toujours en cours. Cette fonctionnalité permet aux demandes suivantes d’aboutir automatiquement, une fois que l’utilisateur passe la première vérification en cours. |
| État du serveur | Consultez l’état de vos serveurs MFA locaux, notamment la version, l’état, l’adresse IP, et l’heure et la date de la dernière communication. |

## <a name="one-time-bypass"></a>Contournement à usage unique

La fonctionnalité de contournement à usage unique permet à un utilisateur de s’authentifier une seule fois sans passer par l’authentification multifacteur. Le contournement est temporaire et expire après le nombre de secondes spécifié. Lorsque l’application mobile ou le téléphone ne reçoit pas de notification ou d’appel téléphonique, vous pouvez autoriser un contournement à usage unique pour que l’utilisateur puisse accéder à la ressource souhaitée.

Pour créer un contournement à usage unique, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur.
1. Recherchez et sélectionnez **Azure Active Directory**, puis accédez à **Sécurité** > **MFA** > **Contournement à usage unique**.
1. Sélectionnez **Ajouter**.
1. Si nécessaire, sélectionnez le groupe de réplication de ce contournement.
1. Entrez le nom d’utilisateur sous la forme `username\@domain.com`. Entrez le nombre de secondes pendant lesquelles le contournement se produira ainsi que la raison du contournement.
1. Sélectionnez **Ajouter**. La limite de temps entre immédiatement en vigueur. L’utilisateur doit se connecter avant que le délai de contournement à usage unique ne soit écoulé.

Vous pouvez également afficher le rapport de contournement à usage unique à partir de cette même fenêtre.

## <a name="caching-rules"></a>Règles de mise en cache

Vous pouvez définir une période pendant laquelle autoriser les tentatives d’authentification lorsqu’un utilisateur est authentifié à l’aide de la fonctionnalité de _mise en cache_. Les tentatives d’authentification suivantes de l’utilisateur durant la période spécifiée aboutissent automatiquement.

La mise en cache s’utilise principalement lorsque des systèmes locaux, comme un VPN, envoient plusieurs demandes de vérification alors que la première demande est toujours en cours. Cette fonctionnalité permet aux demandes suivantes d’aboutir automatiquement, une fois que l’utilisateur passe la première vérification en cours.

>[!NOTE]
> La fonctionnalité de mise en cache n’est pas destinée à être utilisée pour les connexions à Azure Active Directory (Azure AD).

Pour configurer une mise en cache, effectuez les étapes suivantes :

1. Accédez à **Azure Active Directory** > **Sécurité** > **MFA** > **Règles de mise en cache**.
1. Sélectionnez **Ajouter**.
1. Dans la liste déroulante, sélectionnez le **type de cache**. Entrez le nombre maximum de **secondes de mise en cache**.
1. Au besoin, sélectionnez un type d’authentification et spécifiez une application.
1. Sélectionnez **Ajouter**.

## <a name="next-steps"></a>Étapes suivantes

Des options de configuration de serveur MFA supplémentaires sont disponibles à partir de la console Web du serveur MFA lui-même. Vous pouvez aussi [configurer le serveur Azure MFA pour la haute disponibilité](howto-mfaserver-deploy-ha.md).
