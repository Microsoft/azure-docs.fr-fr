---
title: Fournisseurs de Microsoft Azure Multi-Factor Authentication - Azure Active Directory
description: Quand devriez-vous utiliser un fournisseur d’authentification avec Azure MFA ?
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 11/21/2019
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2c8454d2ca83d4f406149e7eb73feb19ce59554f
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96744123"
---
# <a name="when-to-use-an-azure-multi-factor-authentication-provider"></a>Quand utiliser un fournisseur Azure Multi-Factor Authentication

> [!IMPORTANT]
> À partir du 1er septembre 2018, il ne sera plus possible de créer des fournisseurs d’authentification. Il restera possible d’utiliser et de mettre à jour des fournisseurs d’authentification existants, mais la migration ne sera plus envisageable. L’authentification multifacteur restera une fonctionnalité disponible dans les licences Azure AD Premium.

La vérification en deux étapes est disponible par défaut pour les administrateurs généraux disposant d’Azure Active Directory et les utilisateurs Microsoft 365. Toutefois, si vous souhaitez tirer parti des [fonctionnalités avancées](howto-mfa-mfasettings.md), vous devez acheter la version complète d’Azure Multi-Factor Authentication (MFA).

Un fournisseur d’authentification multifacteur Azure permet de tirer parti des fonctionnalités fournies par l’authentification multifacteur Azure pour les utilisateurs qui **n’ont pas de licence**.

## <a name="caveats-related-to-the-azure-mfa-sdk"></a>Mises en garde liées au Kit de développement logiciel (SDK) Azure MFA

Notez que le SDK a été déprécié et fonctionnera seulement jusqu’au 14 novembre 2018. Passée cette date, les appels au Kit de développement logiciel (SDK) échoueront.

## <a name="what-is-an-mfa-provider"></a>Qu’est-ce qu’un fournisseur MFA ?

Il existe deux types de fournisseurs d’authentification, qui se distinguent par la façon dont votre abonnement Azure est facturé. L’option par authentification calcule le nombre d’authentifications effectuées pour votre locataire en un mois. Cette option est recommandée si de nombreux utilisateurs ne s’authentifient qu’occasionnellement. L’option par utilisateur calcule le nombre de personnes sur votre client qui effectuent la vérification en deux étapes au cours d’un mois. Cette option est recommandée si vous disposez de certains utilisateurs dotés de licences, mais que vous avez besoin d’étendre MFA à davantage d’utilisateurs au-delà des limites de vos licences.

## <a name="manage-your-mfa-provider"></a>Gestion de votre fournisseur MFA

Vous ne pouvez pas modifier le modèle d’utilisation (par utilisateur activé ou par authentification) après avoir créé un fournisseur MFA.

Si vous avez acheté suffisamment de licences pour couvrir tous les utilisateurs activés pour l’authentification multifacteur, vous pouvez supprimer complètement le fournisseur d’authentification multifacteur.

Si votre fournisseur MFA n’est pas associé à un locataire Azure AD, ou si vous associez le nouveau fournisseur MFA à un autre locataire Azure AD, les paramètres utilisateur et les options de configuration ne sont pas transférés. Par ailleurs, les serveurs Azure MFA existants doivent être réactivés à l’aide des informations d’identification d’activation générées via le fournisseur MFA.

### <a name="removing-an-authentication-provider"></a>Suppression d’un fournisseur d’authentification

> [!CAUTION]
> Il n’y pas de confirmation lors de la suppression d’un fournisseur d’authentification. La sélection de **Supprimer** est un processus qui a un effet définitif.

Les fournisseurs d'authentification se trouvent sous **Portail Azure** > **Azure Active Directory** > **Security** > **MFA** > **Fournisseurs**. Cliquez sur les fournisseurs listés pour voir les détails et les configurations associés à ce fournisseur.

Avant de supprimer un fournisseur d’authentification, prenez note de tous les paramètres personnalisés configurés dans votre fournisseur. Déterminez les paramètres qui doivent être migrés vers les paramètres d’authentification multifacteur généraux à partir de votre fournisseur et effectuez la migration de ces paramètres. 

Les serveurs Azure MFA liés aux fournisseurs doivent être réactivés à l'aide des informations d'identification générées sous **Portail Azure** > **Azure Active Directory** > **Securité** > **MFA** > **Paramètres du serveur**. Avant la réactivation, les fichiers suivants doivent être supprimés du répertoire `\Program Files\Multi-Factor Authentication Server\Data\` sur les serveurs Azure MFA dans votre environnement :

- caCert
- cert
- groupCACert
- groupKey
- groupName
- licenseKey
- pkey

![Supprimer un fournisseur d’authentification du portail Azure](./media/concept-mfa-authprovider/authentication-provider-removal.png)

Après avoir vérifié que tous les paramètres ont été migrés, vous pouvez accéder à **Portail Azure** > **Azure Active Directory** > **Securité** > **MFA** > **Fournisseurs**, sélectionner les points de suspension **...** , puis sélectionner **Supprimer**.

> [!WARNING]
> La suppression d’un fournisseur d’authentification entraînera la suppression de toutes les informations des rapports associées à ce fournisseur. Vous pouvez enregistrer les rapports d’activité avant de supprimer votre fournisseur.

> [!NOTE]
> Les utilisateurs disposant de versions antérieures de l’application Microsoft Authenticator et du serveur Azure MFA peuvent devoir réinscrire leur application.

## <a name="next-steps"></a>Étapes suivantes

[Configurer les paramètres d’Azure Multi-Factor Authentication](howto-mfa-mfasettings.md)
