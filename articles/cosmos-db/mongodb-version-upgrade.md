---
title: Mettre à niveau la version Mongo de votre API Azure Cosmos DB pour le compte MongoDB
description: Comment mettre à niveau la version de protocole filaire MongoDB pour l’API existante Azure Cosmos DB pour les comptes MongoDB
author: christopheranderson
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: guide
ms.date: 09/22/2020
ms.author: chrande
ms.openlocfilehash: 9ce444e41d19ece984071d0f62e705a09d5f23c9
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93356448"
---
# <a name="upgrade-the-mongodb-wire-protocol-version-of-your-azure-cosmos-dbs-api-for-mongodb-account"></a>Mettre à niveau la version de protocole filaire MongoDB de votre API Azure Cosmos DB pour le compte MongoDB
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Cet article décrit comment mettre à niveau la version de protocole filaire de votre API Azure Cosmos DB pour le compte MongoDB. Après la mise à niveau de la version du protocole, vous pouvez utiliser les fonctionnalités les plus récentes de l’API Azure Cosmos DB pour MongoDB. Le processus de mise à niveau n’interrompt pas la disponibilité de votre compte et ne consomme pas de RU/s, et ne réduit à aucun moment la capacité de la base de données. Les index ou données existants ne seront pas affectés par ce processus.

>[!Note]
> À ce stade, seuls les comptes éligibles utilisant la version serveur 3.2 peuvent être mis à niveau vers la version 3.6. Si votre compte n’affiche pas l’option de mise à niveau, [entrez un ticket de support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

## <a name="upgrading-from-version-32-to-36"></a>Mise à niveau de la version 3.2 à 3.6

### <a name="benefits-of-upgrading-to-version-36"></a>Avantages de la mise à niveau vers la version 3.6

Voici les nouvelles fonctionnalités de la version 3.6 :
- Amélioration de la stabilité et des performances
- Prise en charge de nouvelles commandes de base de données
- Prise en charge du pipeline d’agrégation par défaut et de nouvelles étapes d’agrégation
- Prise en charge de la modification de flux
- Prise en charge des index composés
- Prise en charge inter-partitions pour les opérations suivantes : mise à jour, suppression, décompte et tri
- Amélioration des performances pour les opérations d’agrégation suivantes : $count, $skip, $limit et $group
- L’indexation de caractères génériques est maintenant prise en charge

### <a name="changes-from-version-32"></a>Modifications par rapport à la version 3.2

- **Les erreurs de RequestRateIsLarge ont été supprimées**. Les requêtes de l’application cliente ne retournent plus 16500 erreurs. Les requêtes reprennent jusqu’à ce qu’elles se terminent ou remplissent le délai.
- Le délai d’expiration par requête est de 60 secondes.
- Les collections MongoDB créées sur la nouvelle version du protocole auront uniquement la propriété `_id` indexée par défaut.

### <a name="action-required"></a>Action requise

Pour la mise à niveau vers la version 3.6, le suffixe du point de terminaison du compte de base de données sera mis à jour au format suivant :

```
<your_database_account_name>.mongo.cosmos.azure.com
```

Vous devez remplacer le point de terminaison existant dans vos applications et pilotes qui se connectent à ce compte de base de données. **Seules les connexions qui utilisent le nouveau point de terminaison auront accès aux fonctionnalités de la version 3.6 de MongoDB**. Le point de terminaison précédent doit avoir le suffixe `.documents.azure.com`.

>[!Note]
> Le point de terminaison peut être légèrement différent si votre compte se trouve dans un cloud Azure souverain, gouvernemental ou limité.

### <a name="how-to-upgrade"></a>Mise à niveau

1. Accédez d’abord au portail Azure et accédez au panneau de vue d’ensemble de votre compte MongoDB via l’API Azure Cosmos DB. Vérifiez que la version de votre serveur est `3.2`. 

    :::image type="content" source="./media/mongodb-version-upgrade/1.png" alt-text="Portail Azure avec vue d'ensemble du compte MongoDB" border="false":::

2. Dans les options sur la gauche, sélectionnez le panneau `Features`. Cela permet d’afficher les fonctionnalités de niveau compte disponibles pour votre compte de base de données.

    :::image type="content" source="./media/mongodb-version-upgrade/2.png" alt-text="Portail Azure avec vue d’ensemble du compte MongoDB avec le panneau des fonctionnalités mis en surbrillance" border="false":::

3. Cliquez sur la ligne `Upgrade to Mongo server version 3.6`. Si vous ne voyez pas cette option, votre compte peut ne pas être éligible pour cette mise à niveau. Créez un [ticket de support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) si c’est le cas.

    :::image type="content" source="./media/mongodb-version-upgrade/3.png" alt-text="Panneau de fonctionnalités avec options." border="false":::

4. Passez en revue les informations affichées sur cette mise à niveau. Notez que la mise à niveau ne sera effectuée que lorsque vos applications utiliseront le point de terminaison mis à jour, comme indiqué dans cette section. Cliquez sur `Enable` dès que vous êtes prêt à démarrer le processus.

    :::image type="content" source="./media/mongodb-version-upgrade/4.png" alt-text="Guide de mise à niveau développé." border="false":::

5. Après le démarrage du processus, le menu `Features` affiche l’état de la mise à niveau. L’état passe de `Pending` à `In Progress`, puis à `Upgraded`. Ce processus n’affecte pas les fonctionnalités ou opérations existantes du compte de base de données.

    :::image type="content" source="./media/mongodb-version-upgrade/5.png" alt-text="État de la mise à niveau après le lancement." border="false":::

6. Une fois la mise à niveau terminée, l’état s’affiche comme `Upgraded`. Cliquez dessus pour en savoir plus sur les étapes et les actions que vous devez suivre pour finaliser le processus. Veuillez [contacter le support technique](https://azure.microsoft.com/en-us/support/create-ticket/) en cas de problème lors du traitement de votre demande.

    :::image type="content" source="./media/mongodb-version-upgrade/6.png" alt-text="État du compte mis à niveau." border="false":::

7. **Pour commencer à utiliser la version mise à niveau de votre compte de base de données** , revenez au panneau `Overview` et copiez la nouvelle chaîne de connexion à utiliser dans votre application. Les applications commencent à utiliser la version mise à niveau dès qu’elles se connectent au nouveau point de terminaison. Les connexions existantes ne seront pas interrompues et pourront être mises à jour à votre convenance. Pour garantir une expérience cohérente, toutes vos applications doivent utiliser le nouveau point de terminaison.

    :::image type="content" source="./media/mongodb-version-upgrade/7.png" alt-text="Nouveau panneau de vue d’ensemble." border="false":::

## <a name="next-steps"></a>Étapes suivantes

- Découvrez les [fonctionnalités prises en charge et non prises en charge de MongoDB version 3.6](mongodb-feature-support-36.md).
- Pour plus d’informations, consultez [Fonctionnalités de Mongo version 3.6](https://devblogs.microsoft.com/cosmosdb/azure-cosmos-dbs-api-for-mongodb-now-supports-server-version-3-6/)
