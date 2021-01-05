---
title: Modèle d’identité
titleSuffix: An Azure Communication Services concept
description: Découvrir les identités et les jetons d’accès
author: tomaschladek
manager: nmurav
services: azure-communication-services
ms.author: tchladek
ms.date: 10/26/2020
ms.topic: conceptual
ms.service: azure-communication-services
ms.openlocfilehash: dd2ffacb176ed3733acba8699d4e870b15dd3c42
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94888706"
---
# <a name="identity-model"></a>Modèle d’identité

Azure Communication Services est un service indépendant de l’identité. Cette conception offre plusieurs avantages :

- Elle réutilise les identités existantes de votre système de gestion des identités
- Elle offre de la flexibilité pour les scénarios d’intégration
- Elle conserve le caractère privé de vos identités dans Azure Communication Services

Au lieu de dupliquer les informations dans votre système, vous allez conserver la relation de mappage qui est nécessaire pour votre cas métier. Par exemple, vous pouvez mapper des identités selon une relation 1:1, 1:N, N:1 ou N:M. Les identificateurs externes, comme les numéros de téléphone, les utilisateurs, les appareils, les applications et les GUID, ne peuvent pas être utilisés pour l’identité dans Azure Communication Services. Les jetons d’accès qui sont générés pour une identité Azure Communication Services sont utilisés pour accéder aux primitives comme la conversation ou les appels.

## <a name="identity"></a>Identité

Vous pouvez créer des identités en utilisant la bibliothèque d’administration d’Azure Communication Services. Une identité sert d’identificateur dans les conversations. Elle est utilisée pour créer des jetons d’accès. La même identité peut participer à plusieurs sessions simultanées sur plusieurs appareils. Une identité peut avoir plusieurs jetons d’accès actifs en même temps. 

La suppression d’une identité, d’une ressource ou d’un abonnement invalide tous ses jetons d’accès. Cette action supprime également toutes les données stockées pour l’identité. Une identité supprimée ne peut pas créer de nouveaux jetons d’accès, ni accéder aux données précédemment stockées (par exemple les messages des conversations). 

Vous n’êtes pas facturé pour le nombre d’identités dont vous disposez. Par contre, vous êtes facturé pour l’utilisation des primitives. Le nombre de vos identités ne doit pas limiter la façon dont vous mappez les identités de votre application aux identités Azure Communication Services. 

La liberté de mappage va de pair avec la responsabilité en termes de confidentialité. Si un utilisateur veut être supprimé de votre système, vous devez supprimer toutes les identités qui sont associées à cet utilisateur.

Azure Communication Services ne fournit pas d’identités spéciales pour les utilisateurs anonymes. Il ne conserve pas le mappage entre les utilisateurs et les identités, et ne peut pas déterminer si une identité est anonyme. Vous pouvez adapter le concept d’identité en fonction de vos besoins. Nous vous recommandons de créer une identité pour chaque utilisateur anonyme sur chaque application. 

Toute personne disposant d’un jeton d’accès valide peut accéder au contenu actuel de l’identité. Par exemple, les utilisateurs peuvent accéder aux messages de conversation qu’ils ont envoyés. L’accès est limité aux seules étendues qui font partie du jeton d’accès. Pour plus d’informations, consultez la section [Jetons d’accès](#access-tokens) de cet article.

### <a name="identity-mapping"></a>Mappage d’identité

Azure Communication Services ne réplique pas les fonctionnalités du système de gestion des identités Azure. Il ne permet pas aux clients d’utiliser des identités qui leur sont propres. Par exemple, les clients ne peuvent pas utiliser un numéro de téléphone ou une adresse e-mail. Au lieu de cela, Azure Communication Services fournit des identificateurs uniques. Vous pouvez affecter ces identificateurs uniques aux identités de votre application. Azure Communication Services ne stocke aucune sorte d’informations susceptibles de révéler la vraie identité de vos utilisateurs.

Pour éviter de dupliquer les informations dans votre système, planifiez la façon de mapper les utilisateurs de votre domaine d’identité aux identités d’Azure Communication Services. Vous pouvez suivre n’importe quel type de modèle. Par exemple, vous pouvez utiliser 1:1, 1:N, N:1 ou M:N. Décidez si un utilisateur donné est mappé à une ou à plusieurs identités. 

Quand une nouvelle identité est créée, stockez son mappage à l’utilisateur ou aux utilisateurs de votre application. Comme les identités nécessitent des jetons d’accès pour l’utilisation des primitives, l’identité doit être connue du ou des utilisateurs de votre application.

Si vous utilisez une base de données relationnelle pour stocker des informations utilisateur, vous pouvez ajuster votre conception en fonction de votre scénario de mappage. Pour les scénarios avec un mappage 1:1 ou N:1, vous pouvez ajouter une colonne `CommunicationServicesId` à la table pour stocker votre identité Azure Communication Services. Dans les scénarios utilisant une relation 1:N ou N:M, vous pouvez envisager de créer une table distincte dans la base de données relationnelle.

## <a name="access-tokens"></a>Jetons d’accès

Un jeton d’accès est un jeton JWT (JSON Web Token) qui peut être utilisé pour obtenir l’accès aux primitives Azure Communication Services. Un jeton d’accès émis a une protection de l’intégrité. Autrement dit, ses revendications ne peuvent pas être changées une fois qu’il a été émis. Ainsi, une modification manuelle de propriétés comme l’identité, l’expiration ou les étendues va invalider le jeton d’accès. Si des primitives sont utilisées avec des jetons non valides, l’accès aux primitives est refusé. 

Les propriétés d’un jeton d’accès sont :
* L’identité.
* Expiration.
* Étendues.

Un jeton d’accès est toujours valide pour 24 heures. Une fois expiré, le jeton d’accès est invalidé et ne peut pas être utilisé pour accéder à une primitive. 

Une identité doit avoir un moyen de demander un nouveau jeton d’accès auprès d’un service côté serveur. Le paramètre d’*étendue* définit un ensemble non vide de primitives qui peut être utilisées. Azure Communication Services prend en charge les étendues suivantes pour les jetons d’accès.

|Nom|Description|
|---|---|
|Conversation|  Permet de participer à une conversation|
|VoIP|  Octroie la possibilité d’appeler des identités et des numéros de téléphone|


Pour révoquer un jeton d’accès avant son heure d’expiration, utilisez la bibliothèque d’administration d’Azure Communication Services. La révocation du jeton n’est pas immédiate. Sa propagation prend jusqu’à 15 minutes. La suppression d’une identité, d’une ressource ou d’un abonnement entraîne la révocation de tous les jetons d’accès. 

Si vous voulez retirer à un utilisateur la possibilité d’accéder à des fonctionnalités spécifiques, révoquez tous les jetons d’accès. Émettez ensuite un nouveau jeton d’accès avec un ensemble d’étendues plus limité.

Dans Azure Communication Services, une rotation des clés d’accès révoque tous les jetons d’accès actifs qui ont été créés en utilisant une clé d’accès antérieure. Toutes les identités perdent l’accès à Azure Communication Services et elles doivent émettre de nouveaux jetons d’accès. 

Nous vous recommandons d’émettre les jetons d’accès dans votre service côté serveur, et non dans l’application du client. En effet, l’émission nécessite une clé d’accès ou une identité managée. Pour des raisons de sécurité, le partage des clés d’accès avec l’application du client n’est pas recommandé. 

L’application cliente doit utiliser un point de terminaison de service approuvé qui peut authentifier vos clients. Le point de terminaison doit émettre des jetons d’accès en leur nom. Pour plus d’informations, consultez [Architecture client et serveur](./client-and-server-architecture.md).

Si vous mettez en cache les jetons d’accès dans un magasin de stockage, nous vous recommandons d’utiliser le chiffrement. Un jeton d’accès est une donnée sensible. Il peut être utilisé pour une activité malveillante s’il n’est pas protégé. Une personne disposant d’un jeton d’accès peut démarrer le SDK et accéder à l’API. L’API accessible est limitée seulement en fonction des étendues dont dispose le jeton d’accès. Nous vous recommandons d’émettre des jetons d’accès avec seulement les étendues qui sont nécessaires.

## <a name="next-steps"></a>Étapes suivantes

* Pour une présentation de la gestion des jetons d’accès, consultez [Créer et gérer des jetons d’accès](../quickstarts/access-tokens.md).
* Pour une présentation de l’authentification, consultez [S’authentifier auprès d’Azure Communication Services](./authentication.md).
* Pour une présentation de la confidentialité et de la résidence des données, consultez [Disponibilité des régions et résidence des données](./privacy.md).