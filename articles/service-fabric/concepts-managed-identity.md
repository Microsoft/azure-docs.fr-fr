---
title: Identités managées pour Azure
description: En savoir plus sur l’utilisation des identités managées pour Azure avec Service Fabric.
ms.topic: conceptual
ms.date: 12/09/2019
ms.openlocfilehash: 28c992792d4572a43e12f5d32855f8411b0f4c6f
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96574749"
---
# <a name="using-managed-identities-for-azure-with-service-fabric"></a>Utilisation des identités managées pour Azure avec Service Fabric

Une difficulté courante lors de la création d’applications cloud consiste à gérer en toute sécurité les informations d’identification dans votre code pour l’authentification auprès de différents services sans les enregistrer localement sur une station de travail de développeur ou dans le contrôle de code source. *Les identités managées pour Azure* résolvent ce problème pour toutes vos ressources dans Azure Active Directory (Azure AD) en leur fournissant des identités managées automatiquement dans Azure AD. Vous pouvez utiliser l’identité d’un service pour vous authentifier auprès d’un service qui prend en charge l’authentification Azure AD, y compris Key Vault, sans aucune information d’identification stockée dans votre code.

*Les identités managées pour les ressources Azure* sont gratuites avec Azure AD pour les abonnements Azure. Aucun coût supplémentaire n’est facturé.

> [!NOTE]
> *Identités managées pour Azure* est le nouveau nom du service, anciennement nommé Managed Service Identity (MSI).

## <a name="concepts"></a>Concepts

Identités managées pour Azure s’appuie sur plusieurs concepts clés :

- **ID client** : identificateur unique généré par Azure AD lié à une application et à un principal de service lors de son approvisionnement initial (consultez également [ID d’application](../active-directory/develop/developer-glossary.md#application-id-client-id).)

- **ID du principal** : l’ID d’objet de l’objet du principal de service pour votre identité managée, utilisé pour octroyer un accès basé sur le rôle à une ressource Azure.

- **Principal du service** : un objet Azure Active Directory, qui représente la projection d’une application AAD dans un abonné donné (consultez également [principal du service](../active-directory/develop/developer-glossary.md#service-principal-object).)

Il existe deux types d’identités administrées :

- Une **identité managée attribué par le système** est activée directement sur une instance de service Azure.  Le cycle de vie d’une identité attribuée par le système est propre à l’instance de service Azure sur laquelle elle est activée.
- Une **identité managée attribuée par l’utilisateur** est créée en tant que ressource Azure autonome. L’identité peut être assignée à une ou plusieurs instances de service Azure et est managée séparément des cycles de vie de ces instances.

Pour mieux comprendre la différence entre les types d’identités managées, consultez [Comment fonctionnent les identités managées pour les ressources Azure ?](../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types).

## <a name="supported-scenarios-for-service-fabric-applications"></a>Scénarios pris en charge pour les applications Service Fabric

Les identités managées pour Service Fabric sont prises en charge uniquement dans les clusters Service Fabric déployés dans Azure et uniquement pour les applications déployées en tant que ressources Azure. Une application qui n’est pas déployée en tant que ressource Azure ne peut pas se voir attribuer une identité. En théorie, la prise en charge des identités managées dans un cluster Azure Service Fabric est constitué de deux phases :

1. Attribuez une ou plusieurs identités managées à la ressource d’application ; une application peut se voir attribuer une seule identité attribuée par le système et/ou des identités attribuées par des utilisateurs, jusqu’à 32, respectivement.

2. Dans la définition de l’application, mappez l’une des identités attribuées à l’application à un service individuel comprenant l’application.

L’identité attribuée par le système d’une application est propre à cette application ; une identité attribuée par l’utilisateur est une ressource autonome, qui peut être attribuée à plusieurs applications. Dans une application, une seule identité (qu’elle soit attribuée par le système ou par l’utilisateur) peut être attribuée à plusieurs services de l’application, mais chaque service individuel ne peut être attribué qu’à une seule identité. Enfin, une identité doit être attribuée explicitement à un service pour pouvoir accéder à cette fonctionnalité. Dans les faits, le mappage des identités d’une application à ses services constitutifs permet une isolation dans l’application : un service peut uniquement utiliser l’identité qui lui est mappée.  

Voici les scénarios suivants pris en charge actuellement pour cette fonctionnalité :

- Déployer une nouvelle application avec un ou plusieurs services et une ou plusieurs identités attribuées

- Attribuer une ou plusieurs identités managées à une application existante (déployée dans Azure) afin d’accéder aux ressources Azure

Les scénarios suivants ne sont pas pris en charge ni recommandés ; notez que ces actions peuvent ne pas être bloquées, mais peuvent entraîner des pannes dans vos applications :

- Supprimez ou modifiez les identités attribuées à une application. Si vous devez apporter des modifications, envoyez des déploiements distincts pour ajouter d’abord une nouvelle attribution d’identité, puis en supprimer une précédemment attribuée. La suppression d’une identité d’une application existante peut avoir des effets indésirables, notamment la sortie de votre application dans un état qui ne peut pas être mis à niveau. Il est sans danger de supprimer entièrement l’application si la suppression d’une identité est nécessaire ; notez que cette opération supprimera l’identité attribuée par le système (si définie) associée à l’application et supprimera toutes les associations avec les identités attribuées par l’utilisateur et attribuées à l’application.

- La prise en charge par Service Fabric des identités managées n’est pas intégrée à l’heure actuelle dans [AzureServiceTokenProvider](../key-vault/general/service-to-service-authentication.md).

## <a name="next-steps"></a>Étapes suivantes

- [Déployer un nouveau cluster Azure Service Fabric avec un support d’identité managée](./configure-new-azure-service-fabric-enable-managed-identity.md)
- [Activer le support d’identités managées dans un cluster Azure Service Fabric existant](./configure-existing-cluster-enable-managed-identity-token-service.md)
- [Déployer une application Azure Service Fabric avec une identité managée attribuée par le système](./how-to-deploy-service-fabric-application-system-assigned-managed-identity.md)
- [Déployer une application Azure Service Fabric avec une identité managée attribuée par l’utilisateur](./how-to-deploy-service-fabric-application-user-assigned-managed-identity.md)
- [Tirer parti de l’identité managée d’une application Service Fabric à partir du code de service](./how-to-managed-identity-service-fabric-app-code.md)
- [Accorder à une application Azure Service Fabric l’accès à d’autres ressources Azure](./how-to-grant-access-other-resources.md)
- [Déclaration et utilisation de secrets d’application en tant que KeyVaultReferences](./service-fabric-keyvault-references.md)
