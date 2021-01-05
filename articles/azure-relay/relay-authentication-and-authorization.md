---
title: Authentification et autorisation Azure Relay | Microsoft Docs
description: Cet article présente l’authentification par signature d’accès partagé (SAS) auprès du service Azure Relay.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 4b0a5c7a092155a006419eedd170a63abed42bb3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87033375"
---
# <a name="azure-relay-authentication-and-authorization"></a>Authentification et autorisation Azure Relay

Les applications peuvent s’authentifier auprès d’Azure Relay à l’aide de l’authentification par signature d’accès partagé. L’authentification par signature d’accès partagé permet aux applications de s’authentifier auprès du service Azure Relay à l’aide d’une clé d’accès configurée dans l’espace de noms Relay. Vous pouvez ensuite utiliser cette clé pour générer un jeton de signature d’accès partagé que les clients peuvent alors utiliser pour s’authentifier auprès du service Azure Relay.

## <a name="shared-access-signature-authentication"></a>Authentification avec une signature d’accès partagé

[L’authentification par signature d’accès partagé](../service-bus-messaging/service-bus-sas.md) vous permet d’accorder un accès utilisateur aux ressources Azure Relay avec des droits spécifiques. L’authentification SAP implique la configuration d’une clé de chiffrement avec les droits associés sur une ressource. Les clients peuvent alors accéder à cette ressource en présentant un jeton SAS qui se compose de la ressource URI à laquelle accéder et une échéance signée avec la clé configurée.

Vous pouvez configurer des clés pour SAP dans un espace de noms Relay. Contrairement à la messagerie Service Bus, les [connexions hybrides Relay](relay-hybrid-connections-protocol.md) prennent en charge les expéditeurs non autorisés ou anonymes. Vous pouvez activer l’accès anonyme pour l’entité lors de sa création, comme indiqué dans la capture d’écran suivante du portail :

![Une boîte de dialogue intitulée « Créer une connexion hybride » contient une zone de texte « Nom » et une case à cocher intitulée « Nécessite l’authentification du client », qui est activée.][0]

Pour utiliser SAP, vous pouvez configurer un objet [SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule) dans un espace de noms Relay, qui est constitué des éléments suivants :

* *Nom de clé* qui identifie la règle.
* *clé primaire* est une clé de chiffrement utilisée pour signer/valider les jetons SAS.
* *clé secondaire* est une clé de chiffrement utilisée pour signer/valider les jetons SAS.
* *droits* représentant la collection des droits écouter, envoyer ou gérer les droits accordés.

Les règles d’autorisation configurées au niveau de l’espace de noms peuvent accorder l’accès à toutes les connexions Relay dans un espace de noms pour les clients disposant de jetons signés à l’aide de la clé correspondante. Un maximum de 12 règles d’autorisation peut être configuré dans un espace de noms Relay. Par défaut, un élément [SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule) avec tous les droits est configuré pour chaque espace de noms dès sa mise en service initiale.

Pour accéder à une entité, le client requiert un jeton SAP créé à l’aide d’une règle [SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule)spécifique. Le jeton SAS est généré à l’aide du code HMAC-SHA256 d’une chaîne de ressource qui se compose de l’URI de la ressource à laquelle vous souhaitez accéder et d’une échéance avec une clé de chiffrement associée à la règle d’autorisation.

La prise en charge de l’authentification SAP pour Azure Relay est incluse dans le Kit de développement logiciel (SDK) Azure .NET, versions 2.0 et ultérieures. SAP inclut l’assistance pour [SharedAccessAuthorizationRule](/dotnet/api/microsoft.servicebus.messaging.sharedaccessauthorizationrule). Toutes les API qui acceptent une chaîne de connexion en tant que paramètre incluent la prise en charge des chaînes de connexion des services SAS.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur la signature d’accès partagé (SAP), consultez [Authentification de Service Bus avec les signatures d’accès partagé](../service-bus-messaging/service-bus-sas.md).
- Pour en savoir plus sur la fonctionnalité Connexions hybrides, consultez le [Guide du protocole de connexions hybrides Azure Relay](relay-hybrid-connections-protocol.md).
- Pour obtenir les informations correspondantes sur les autorisations et l’authentification de la messagerie Service Bus, consultez [Authentification et autorisation Service Bus](../service-bus-messaging/service-bus-authentication-and-authorization.md). 

[0]: ./media/relay-authentication-and-authorization/hcanon.png