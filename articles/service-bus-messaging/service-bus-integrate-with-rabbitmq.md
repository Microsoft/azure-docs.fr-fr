---
title: Intégration de RabbitMQ à Azure Service Bus
description: Guide pas à pas sur l’intégration de RabbitMQ à Azure Service Bus
services: Azure Service Bus
author: videlalvaro
ms.topic: how-to
ms.service: service-bus
ms.date: 07/02/2020
ms.author: alvidela
ms.openlocfilehash: 6366824b8dc7f63f99ebda2a542d95d3eb1c6146
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91301103"
---
# <a name="how-to-integrate-rabbitmq-with-azure-service-bus"></a>Intégration de RabbitMQ à Azure Service Bus

Dans ce guide, nous allons apprendre à envoyer des messages de RabbitMQ à Azure Service Bus.

Voici quelques scénarios dans lesquels nous pouvons utiliser ces fonctionnalités :

- **Configurations de périphérie** : Nous disposons d’une configuration de périphérie dans laquelle nous envoyons des messages à RabbitMQ. Toutefois, nous voulons transférer ces messages vers [Azure Service Bus](./service-bus-messaging-overview.md) pour un traitement ultérieur, afin que nous puissions utiliser un grand nombre de [fonctionnalités Azure Big Data](/azure/architecture/guide/architecture-styles/big-data).
- **Cloud hybride** : Votre entreprise vient d’acquérir un tiers qui utilise RabbitMQ pour ses besoins en matière de messagerie. Ils se trouvent sur un autre Cloud. Lors de la transition vers Azure, vous pouvez déjà commencer à partager des données en utilisant le pontage RabbitMQ avec Azure Service Bus.
- **Intégration de tiers** : Un tiers utilise RabbitMQ comme service de répartiteur et souhaite nous envoyer ses données, mais il est externe à notre organisation. Nous pouvons lui fournir une clé SAS lui donnant accès à un ensemble limité de files d’attente dans Azure Service Bus où il pourra transférer ses messages.

La liste est activée, mais nous pouvons résoudre la plupart de ces cas d’utilisation grâce au pontage de RabbitMQ vers Azure.

Tout d’abord, vous devez créer un compte Azure gratuit en vous inscrivant [ici](https://azure.microsoft.com/free/)

Une fois connecté à votre compte, accédez au [Portail Azure](https://portal.azure.com/) et créez un espace de noms [Azure Service Bus](./service-bus-create-namespace-portal.md). Les espaces de noms sont les conteneurs d’étendue dans lesquels nos composants de messagerie sont actifs, comme les files d’attente et les rubriques.

## <a name="adding-a-new-azure-service-bus-namespace"></a>Ajouter un nouvel espace de noms Azure Service Bus

Dans le portail Azure, cliquez sur le grand bouton plus pour ajouter une nouvelle ressource

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/create-resource.png" alt-text="Créer une ressource":::

Sélectionnez ensuite Intégration, puis cliquez sur Azure Service Bus pour créer un espace de noms de messagerie :

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/integration.png" alt-text="Créer une ressource":::

Vous êtes invité à entrer les informations de l’espace de noms. Sélectionnez l’abonnement Azure à utiliser. Si vous n’avez pas de [groupe de ressources](../azure-resource-manager/management/manage-resource-groups-portal.md), vous pouvez en créer un.

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/create-namespace.png" alt-text="Créer une ressource":::

Utilisez `rabbitmq` pour `Namespace name`, mais cela peut être tout ce que vous voulez. Ensuite, définissez `East US` comme emplacement. Choisissez `Basic` comme niveau tarifaire.

Si tout s’est bien passé, vous devriez voir l’écran de confirmation suivant :

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/create-namespace-confirm.png" alt-text="Créer une ressource":::

Ensuite, revenez au portail Azure vous verrez votre nouvel espace de noms `rabbitmq` y figurer. Cliquez dessus pour accéder à la ressource afin de pouvoir y ajouter une file d’attente.

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/resource-view-with-namespace.png" alt-text="Créer une ressource":::

## <a name="creating-our-azure-service-bus-queue"></a>Création de nos files d’attente Azure Service Bus

Maintenant que vous disposez de votre espace de noms Azure Service Bus, cliquez sur le bouton `Queues` sur la gauche, sous `Entities`, afin d’ajouter une nouvelle file d’attente :

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/create-queue.png" alt-text="Créer une ressource":::

Le nom de la file d’attente est `from-rabbitmq` comme un rappel de l’emplacement des messages. Vous pouvez conserver toutes les autres options comme valeurs par défaut, mais vous pouvez les modifier en fonction des besoins de votre application.

## <a name="enabling-the-rabbitmq-shovel-plugin"></a>Activation du plug-in RabbitMQ Shovel

Pour envoyer des messages de RabbitMQ à Azure Service Bus, nous allons utiliser le plug-in [Shovel](https://www.rabbitmq.com/shovel.html) intégré à RabbitMQ. Vous pouvez activer le plug-in et son interface visuelle à l’aide de cette commande :

```bash
rabbitmq-plugins enable rabbitmq_shovel_management
```

>Vous devrez peut-être exécuter cette commande à la racine.

Il est maintenant temps d’obtenir les informations d’identification requises pour connecter RabbitMQ à Azure.

## <a name="connecting-rabbitmq-to-azure-service-bus"></a>Connexion de RabbitMQ à Azure Service Bus

Vous devez créer une [Stratégie d’accès partagé](../storage/common/storage-sas-overview.md) (SAP) pour votre file d’attente, afin que RabbitMQ puisse y publier des messages. Une stratégie SAP vous permet de spécifier ce que le tiers externe est autorisé à faire avec votre ressource. L’idée est que RabbitMQ peut envoyer des messages, mais pas écouter ou gérer la file d’attente.

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/create-sas-policy.png" alt-text="Créer une ressource":::

Cochez la case `Send`, puis cliquez sur `Create` pour mettre en place la stratégie SAP.

Une fois la stratégie créée, cliquez dessus pour afficher la **chaîne de connexion principale**. Nous allons l’utiliser pour permettre à RabbitMQ de communiquer avec Azure Service Bus :

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/sas-policy-key.png" alt-text="Créer une ressource":::

Avant de pouvoir utiliser cette chaîne de connexion, vous devez la convertir au format de connexion AMQP de RabbitMQ. Donc, accédez à l’[outil de convertisseur de chaîne de connexion](https://red-mushroom-0f7446a0f.azurestaticapps.net/) et collez votre chaîne de connexion dans le formulaire, puis cliquez sur Convertir. Vous obtiendrez une chaîne de connexion RabbitMQ. (Ce site web exécute tout ce qui est local dans votre navigateur pour que vos données ne soient pas envoyées sur le réseau). Vous pouvez accéder à son code source sur [GitHub](https://github.com/videlalvaro/connstring_to_amqp).

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/converter.png" alt-text="Créer une ressource":::

Maintenant, ouvrez le plug-in de gestion de RabbitMQ dans nos navigateurs `http://localhost:15672/#/dynamic-shovels` et accédez à `Admin -> Shovel Management`, où vous pouvez ajouter Shovel qui prendra en charge l’envoi de messages d’une file d’attente RabbitMQ à votre file d’attente Azure Service Bus.

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/add-shovel.png" alt-text="Créer une ressource":::

Ici, nommez votre Shovel `azure` et choisissez `AMQP 0.9.1` comme protocole source. Dans la capture d’écran, nous avons `amqp://`, qui est l’URI par défaut qui se connecte à un serveur RabbitMQ local. Veillez à l’adapter à votre déploiement actuel.

Du côté de la file d’attente, vous pouvez utiliser `azure` comme nom de votre file d’attente. Si cette file d’attente n’existe pas, RabbitMQ la crée pour vous. Vous pouvez également choisir le nom d’une file d’attente qui existe déjà. Laissez les autres options telles quelles.

Ensuite, dans la `destination`, choisissez `AMQP 1.0` comme protocole. Dans le champ `URI`, entrez la chaîne de connexion que vous avez obtenue à l’étape précédente, étiez-vous convertie votre chaîne de connexion Azure au format RabbitMQ. Il doit se présenter comme suit :

```
amqps://rabbitmq-shovel:StringOfRandomChars@rabbitmq.servicebus.windows.net:5671/?sasl=plain
```

Dans le champ `Address`, nous allons entrer le nom de votre **file d’attente Azure Service Bus**, dans ce cas, il s’agit de `from-rabbitmq`. Cliquez sur `Add Shovel` et votre configuration sera prête à recevoir des messages.

## <a name="publishing-messages-from-rabbitmq-to-azure-service-bus"></a>Publication des messages de RabbitMQ vers Azure Service Bus

Dans l’interface de gestion RabbitMQ, nous pouvons accéder à `Queues`, sélectionner la file d’attente `azure` et rechercher le panneau `Publish message`. Un formulaire s’affiche et vous permet de publier des messages directement dans votre file d’attente. Pour notre exemple, nous allons simplement ajouter `fist message` comme `Payload` et appuyer sur `Publish Message` :

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/first-message.png" alt-text="Créer une ressource":::

Revenez à Azure et examinez votre file d’attente. Cliquez sur `Service Bus Explorer` dans le volet gauche, puis sur le bouton _Peek_. Si tout s’est bien passé, vous verrez que votre file d’attente contient désormais un message. Félicitations !

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/service-bus-queue.png" alt-text="Créer une ressource":::

Mais vérifions que le message est celui que vous avez envoyé à partir de RabbitMQ. Sélectionnez l’onglet `Peek`, puis cliquez sur le bouton `Peek` pour récupérer les derniers messages dans votre file d’attente. Cliquez sur le message pour inspecter son contenu. Vous devriez voir une image semblable à l’image ci-dessous où figure votre `first message`.

:::image type="content" source="./media/service-bus-integrate-with-rabbitmq/peek.png" alt-text="Créer une ressource":::

## <a name="lets-recap"></a>Récapitulatif

Félicitations ! Vous avez bien travaillé ! Vous avez réussi à récupérer vos messages de RabbitMQ à Azure Service Bus, récapitulons les étapes suivantes :

1. Créer un espace de noms Azure Service Bus
2. Ajouter une file d’attente à l’espace de noms
3. Ajouter une stratégie SAP à votre file d’attente
4. Obtenir la file d'attente de la chaîne de connexion
5. Activer le plug-in RabbitMQ Shovel et l’interface de gestion
6. Convertir la chaîne de connexion Azure Service Bus au format AMQP de RabbitMQ
7. Ajoutez un nouveau Shovel à RabbitMQ et le connecter à Azure Service Bus
8. Publier des messages

En suivant les étapes précédentes, vous avez intégré des zones de votre organisation qui étaient en dehors d’Azure. Le plug-in Shovel vous permettait d’expédier des messages de RabbitMQ à Azure Service Bus. Cela présente des avantages considérables, car vous pouvez désormais autoriser des tiers de confiance à connecter leurs applications à votre déploiement Azure.

En fin de compte, la messagerie est basée sur des connexions, et avec cette technique, nous venons d'en ouvrir une nouvelle.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur [Azure Service Bus](./service-bus-messaging-overview.md).
- En savoir plus sur la [prise en charge d’AMQP 1.0 dans Service Bus](./service-bus-amqp-overview.md)
