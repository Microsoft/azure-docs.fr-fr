---
title: Modèle de sécurité des notification Hubs
description: En savoir plus sur le modèle de sécurité pour Azure Notification Hubs.
services: notification-hubs
documentationcenter: .net
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: ''
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: multiple
ms.topic: article
ms.date: 09/23/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 09/23/2019
ms.openlocfilehash: b871775bc7a6d795e86147ae9cffa27bdd2f3348
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76263759"
---
# <a name="notification-hubs-security"></a>Sécurité de Notification Hubs

## <a name="overview"></a>Vue d’ensemble

Cette rubrique décrit le modèle de sécurité d'Azure Notification Hubs.

## <a name="shared-access-signature-security"></a>Sécurité de signature d’accès partagé

Notification Hubs implémente un modèle de sécurité de niveau entité appelé *Signature d’accès partagé* (SAS). Chaque règle contient un nom, une valeur de clé (secret partagé) et un ensemble de droits, comme expliqué plus loin dans [Revendications de sécurité](#security-claims). 

Quand vous créez un hub, deux règles sont créées automatiquement : une avec des droits **Écouter** (utilisée par l’application cliente) et l’autre avec **tous** les droits (utilisée par le back-end d’application) :

- **DefaultListenSharedAccessSignature** : accorde uniquement l’autorisation **Écouter**.
- **DefaultFullSharedAccessSignature** : accorde les autorisations **Écouter**, **Gérer** et **Envoyer**. Cette stratégie doit être utilisée uniquement dans le back-end de votre application. Ne l’utilisez pas dans les applications clientes ; utilisez une stratégie uniquement avec l’accès **Écouter**. Pour créer une stratégie d’accès personnalisée avec un nouveau jeton SAS, consultez [Jetons SAS pour les stratégies d’accès](#sas-tokens-for-access-policies) plus loin dans cet article.

Lors de la gestion de l'inscription à partir d’applications clientes, si les informations envoyées via des notifications ne sont pas sensibles (par exemple, des mises à jour météorologiques), une méthode courante pour accéder à un concentrateur de notification consiste à attribuer la valeur clé de la règle d'accès en écoute uniquement à l'application cliente et à attribuer la valeur clé de la règle d’accès complet au serveur principal.

Les applications ne doivent pas incorporer la valeur de la clé dans les applications clientes Windows Store ; au lieu de cela, le client doit être invité à la récupérer à partir du back-end d’application au démarrage.

La clé avec un accès en **écoute** permet à une application cliente de s’inscrire pour n’importe quelle balise. Si votre application doit limiter les inscriptions à des balises spécifiques pour des clients spécifiques (par exemple, lorsque les balises représentent des ID d'utilisateur), votre serveur principal d’application doit effectuer ces inscriptions. Pour plus d’informations, consultez [Gestion des inscriptions](notification-hubs-push-notification-registration-management.md). Notez que de cette façon, l'application cliente ne disposera pas d'un accès direct à Notification Hubs.

## <a name="security-claims"></a>Revendications de sécurité

Comme pour les autres entités, les opérations Notification Hubs sont autorisées pour trois revendications de sécurité : **Écoute**, **Envoyer** et **Gérer**.

| Revendication   | Description                                          | Opérations autorisées |
| ------- | ---------------------------------------------------- | ------------------ |
| Écouter  | Créer/mettre à jour, lire et supprimer des inscriptions uniques | Créer/mettre à jour une inscription<br><br>Lire une inscription<br><br>Lire toutes les inscriptions pour un handle<br><br>Supprimer l'inscription |
| Envoyer    | Envoyer des messages au hub de notification                | Envoyer un message |
| Gérer  | Opérations CRUD sur Notification Hubs (y compris la mise à jour des informations d'identification PNS et les clés de sécurité) et lecture des inscriptions en fonction des balises |Créer/mettre à jour/lire/supprimer des hubs<br><br>Lire des inscriptions par balise |

Notification Hubs accepte les jetons SAS générés avec des clés partagées configurées directement sur le hub.

Il n’est pas possible d’envoyer une notification à plusieurs espaces de noms. Les espaces de noms sont des conteneurs logiques pour Notification Hubs, et ne sont pas impliqués dans l’envoi de notifications.

Utilisez les stratégies d’accès au niveau de l’espace de noms (informations d’identification) pour les opérations au niveau de l’espace de noms, par exemple lister les hubs de notification, créer ou supprimer des hubs, et ainsi de suite. Seules les stratégies au niveau du hub vous permettent d’envoyer des notifications.

### <a name="sas-tokens-for-access-policies"></a>Jetons SAS pour les stratégies d’accès

Pour créer une nouvelle revendication de sécurité ou voir les clés SAS existantes, effectuez les étapes suivantes :

1. Connectez-vous au portail Azure.
2. Sélectionnez **Toutes les ressources**.
3. Sélectionnez le nom du hub de notification pour lequel vous souhaitez créer la revendication ou voir la clé SAS.
4. Dans le menu de gauche, sélectionnez **Stratégies d’accès**.
5. Sélectionnez **Nouvelle stratégie** pour créer une revendication de sécurité. Donnez un nom à la stratégie et sélectionnez les autorisations que vous souhaitez accorder. Sélectionnez ensuite **OK**.
6. La chaîne de connexion complète (y compris la nouvelle clé SAS) s’affiche dans la fenêtre Stratégies d’accès. Vous pouvez copier cette chaîne dans le Presse-papiers pour l’utiliser plus tard.

Pour extraire la clé SAS d’une stratégie spécifique, sélectionnez le bouton **Copier** en regard de la stratégie contenant la clé SAS de votre choix. Collez cette valeur dans un emplacement temporaire, puis copiez la partie clé SAS de la chaîne de connexion. Cet exemple utilise un espace de noms Notification Hubs nommé **mytestnamespace1** et une stratégie nommée **policy2**. La clé SAS est la valeur qui se trouve près de la fin de la chaîne, spécifiée par **SharedAccessKey** :

```shell
Endpoint=sb://mytestnamespace1.servicebus.windows.net/;SharedAccessKeyName=policy2;SharedAccessKey=<SAS key value here>
```

![Obtenir des clés SAS](media/notification-hubs-push-notification-security/access1.png)

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble de Notification Hubs](notification-hubs-push-notification-overview.md)
