---
title: Gérer la connectivité IoT Hub et la messagerie fiable avec des Kits de développement logiciel (SDK) d’appareils
description: Découvrez comment améliorer la connectivité et la messagerie de vos appareils lorsque vous utilisez les kits Azure IoT Hub device SDK
services: iot-hub
author: robinsh
ms.author: robinsh
ms.date: 07/07/2018
ms.topic: article
ms.service: iot-hub
ms.custom:
- amqp
- mqtt
- devx-track-csharp
ms.openlocfilehash: bccad7774a8ec4ab415718f28a2adff0bc58863d
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92144477"
---
# <a name="manage-connectivity-and-reliable-messaging-by-using-azure-iot-hub-device-sdks"></a>Gérer la connectivité et la messagerie fiable à l’aide des kits SDK d’appareil Azure IoT Hub

Cet article fournit des instructions détaillées pour vous aider à concevoir des applications d’appareil plus résilientes. Il vous montre comment tirer parti des fonctionnalités de connectivité et de messagerie fiable fournies dans les kits Azure IoT device SDK. L’objectif de ce guide est de vous aider à gérer les scénarios suivants :

* Correction d’une connexion réseau supprimée

* Basculement entre différentes connexions réseau

* Reconnexion en raison d’erreurs de connexion temporaires aux services

Les détails d’implémentation peuvent varier selon le langage. Pour plus d’informations, consultez la documentation de l’API ou le kit SDK spécifique :

* [Kit de développement logiciel (SDK) C/iOS](https://github.com/azure/azure-iot-sdk-c)

* [Kit de développement logiciel (SDK) .NET](https://github.com/Azure/azure-iot-sdk-csharp/blob/master/iothub/device/devdoc/retrypolicy.md)

* [Kit SDK Java](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/devdoc/requirement_docs/com/microsoft/azure/iothub/retryPolicy.md)

* [Node SDK](https://github.com/Azure/azure-iot-sdk-node/wiki/Connectivity-and-Retries#types-of-errors-and-how-to-detect-them)

* [Kit de développement logiciel (SDK) Python](https://github.com/Azure/azure-iot-sdk-python) (Fiabilité encore non implémentée)

## <a name="designing-for-resiliency"></a>Conception pour la résilience

Les appareils IoT s’appuient souvent sur des connexions réseau non continues ou instables telles que des connexions GSM ou satellite. Des erreurs peuvent se produire quand les appareils interagissent avec les services cloud à cause d’une disponibilité intermittente des services et d’erreurs d’infrastructure ou temporaires. Une application qui s’exécute sur un appareil doit gérer les mécanismes de connexion et de reconnexion ainsi que la logique de nouvelle tentative pour l’envoi et la réception de messages. De plus, les exigences de stratégie de nouvelles tentatives dépendent fortement du scénario IoT, du contexte et des fonctionnalités de l’appareil.

Les kits SDK d’appareil Azure IoT Hub ont pour but de simplifier la connexion et la communication de cloud-à-appareil et d’appareil-à-cloud. Ces kits fournissent un moyen fiable de se connecter à Azure IoT Hub et un ensemble complet d’options pour envoyer et recevoir des messages. Les développeurs peuvent également modifier l’implémentation existante pour personnaliser une meilleure stratégie de nouvelles tentatives pour un scénario donné.

Les fonctionnalités SDK pertinentes qui prennent en charge la connectivité et la messagerie fiable sont décrites dans les sections suivantes.

## <a name="connection-and-retry"></a>Connexion et nouvelle tentative

Cette section fournit une vue d’ensemble des modèles de reconnexion et de nouvelle tentative disponibles lors de la gestion des connexions. Elle détaille les conseils d’implémentation pour l’utilisation d’une stratégie de nouvelles tentatives différente dans votre application d’appareil et répertorie les API appropriées des kits SDK d’appareil.

### <a name="error-patterns"></a>Modèles d’erreur

Des échecs de connexion peuvent se produire à de nombreux niveaux :

* Erreurs réseau : socket déconnecté et erreurs de résolution de nom

* Erreurs au niveau du protocole pour le transport HTTP, AMQP et MQTT : liens détachés ou sessions expirées

* Erreurs au niveau de l’application qui résultent d’erreurs locales, telles que des informations d’identification non valides, ou du comportement des services, tel que le dépassement du quota ou de la limitation

Les kits SDK d’appareil détectent les erreurs aux trois niveaux. Les erreurs liées au système d’exploitation et les erreurs matérielles ne sont pas détectées ni gérées par les kits SDK d’appareil. La conception des kits SDK est basée sur le [Guide de gestion des erreurs temporaires](/azure/architecture/best-practices/transient-faults#general-guidelines) fourni dans le Centre des architectures Azure.

### <a name="retry-patterns"></a>Modèles de nouvelle tentative

Les étapes suivantes décrivent le processus de nouvelle tentative lors de la détection d’erreurs de connexion :

1. Le kit SDK détecte l’erreur et l’erreur associée dans le réseau, le protocole ou l’application.

2. Le kit SDK utilise le filtre d’erreur pour déterminer le type d’erreur et décider si une nouvelle tentative est nécessaire.

3. Si le kit SDK identifie une **erreur irrécupérable**, les opérations telles que la connexion, l’envoi et la réception sont arrêtées. Le kit SDK en informe l’utilisateur. Une erreur d’authentification et une erreur de point de terminaison défectueux sont des exemples d’erreurs irrécupérables.

4. Si le kit SDK identifie une **erreur irrécupérable**, il effectue une nouvelle tentative en fonction de la stratégie de nouvelles tentatives spécifiée jusqu’à l’expiration du délai donné.  Notez que le Kit de développement logiciel (SDK) utilise par défaut une stratégie de nouvelles tentatives de type **interruption exponentielle avec instabilité**.
5. Quand le délai imparti expire, le kit SDK arrête d’essayer d’établir la connexion ou l’envoi. Il en informe l’utilisateur.

6. Le kit SDK permet à l’utilisateur de joindre un rappel pour recevoir les modifications de l’état de la connexion.

Les kits SDK fournissent trois stratégies de nouvelles tentatives :

* **Interruption exponentielle avec instabilité** : cette stratégie de nouvelles tentatives par défaut a tendance à être agressive au début, puis ralentit progressivement avant d’atteindre un délai maximal. La conception est basée sur le [Guide sur les nouvelles tentatives fourni dans le Centre des architectures Azure](/azure/architecture/best-practices/retry-service-specific). 

* **Nouvelle tentative personnalisée** : pour certains langages de SDK, vous pouvez concevoir une stratégie de nouvelles tentatives personnalisée qui est mieux adaptée à votre scénario, puis l’injecter dans la stratégie de nouvelles tentatives. La nouvelle tentative personnalisée n’est pas disponible sur le kit SDK C, et n’est actuellement pas prise en charge sur le kit SDK Python. Le kit SDK Python se reconnecte en fonction des besoins.

* **Aucune nouvelle tentative** : vous pouvez définir la stratégie de nouvelles tentatives sur « aucune nouvelle tentative », ce qui désactive la logique de nouvelle tentative. Le kit SDK tente de se connecter une fois et d’envoyer un message une fois, en supposant que la connexion est établie. Cette stratégie est généralement utilisée dans des scénarios avec des problèmes de bande passante ou de coût. Si vous choisissez cette option, les messages qui ne peuvent pas être envoyés sont perdus et ne peuvent pas être récupérés.

### <a name="retry-policy-apis"></a>API de stratégie de nouvelles tentatives

   | Kit SDK | Méthode SetRetryPolicy | Implémentations de stratégie | Conseils d’implémentation |
   |-----|----------------------|--|--|
   |  C/iOS  | [IOTHUB_CLIENT_RESULT IoTHubClient_SetRetryPolicy](https://github.com/Azure/azure-iot-sdk-c/blob/2018-05-04/iothub_client/inc/iothub_client.h#L188)        | **Par défaut** : [IOTHUB_CLIENT_RETRY_EXPONENTIAL_BACKOFF](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/connection_and_messaging_reliability.md#connection-retry-policies)<BR>**Personnalisée :** utiliser la stratégie [retryPolicy](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/connection_and_messaging_reliability.md#connection-retry-policies) disponible<BR>**Aucune nouvelle tentative :** [IOTHUB_CLIENT_RETRY_NONE](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/connection_and_messaging_reliability.md#connection-retry-policies)  | [Implémentation C/iOS](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/connection_and_messaging_reliability.md#)  |
   | Java| [SetRetryPolicy](/java/api/com.microsoft.azure.sdk.iot.device.deviceclientconfig.setretrypolicy?view=azure-java-stable)        | **Par défaut** : [classe ExponentialBackoffWithJitter](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/src/main/java/com/microsoft/azure/sdk/iot/device/transport/NoRetry.java)<BR>**Personnalisée :** implémenter l’[interface RetryPolicy](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/src/main/java/com/microsoft/azure/sdk/iot/device/transport/RetryPolicy.java)<BR>**Aucune nouvelle tentative :** [classe NoRetry](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/src/main/java/com/microsoft/azure/sdk/iot/device/transport/NoRetry.java)  | [Implémentation Java](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/devdoc/requirement_docs/com/microsoft/azure/iothub/retryPolicy.md) |
   | .NET| [DeviceClient.SetRetryPolicy](/dotnet/api/microsoft.azure.devices.client.deviceclient.setretrypolicy?view=azure-dotnet) | **Par défaut** : [classe ExponentialBackoff](/dotnet/api/microsoft.azure.devices.client.exponentialbackoff?view=azure-dotnet)<BR>**Personnalisée :** implémenter l’[interface IRetryPolicy](/dotnet/api/microsoft.azure.devices.client.iretrypolicy?view=azure-dotnet)<BR>**Aucune nouvelle tentative :** [classe NoRetry](/dotnet/api/microsoft.azure.devices.client.noretry?view=azure-dotnet) | [Implémentation C#](https://github.com/Azure/azure-iot-sdk-csharp) | |
   | Nœud| [setRetryPolicy](/javascript/api/azure-iot-device/client?view=azure-iot-typescript-latest) | **Par défaut** : [classe ExponentialBackoffWithJitter](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/src/main/java/com/microsoft/azure/sdk/iot/device/transport/NoRetry.java) | [Implémentation Node](https://github.com/Azure/azure-iot-sdk-node/wiki/Connectivity-and-Retries#types-of-errors-and-how-to-detect-them) |
   | Python| Non prise en charge pour le moment | Non prise en charge pour le moment | Non prise en charge pour le moment |

Les exemples de code ci-dessous illustrent ce flux :

#### <a name="net-implementation-guidance"></a>Conseils d’implémentation .NET

L’exemple de code ci-dessous montre comment définir et paramétrer la stratégie de nouvelles tentatives par défaut :

   ```csharp
   // define/set default retry policy
   IRetryPolicy retryPolicy = new ExponentialBackoff(int.MaxValue, TimeSpan.FromMilliseconds(100), TimeSpan.FromSeconds(10), TimeSpan.FromMilliseconds(100));
   SetRetryPolicy(retryPolicy);
   ```

Pour éviter une utilisation élevée du processeur, les nouvelles tentatives sont limitées si le code échoue immédiatement, par exemple quand il n’existe aucun réseau ni aucune route vers la destination. Le temps minimal pour exécuter la nouvelle tentative suivante est de 1 seconde.

Si le service répond avec une erreur de limitation, la stratégie de nouvelles tentatives est différente et ne peut pas être modifiée via l’API publique :

   ```csharp
   // throttled retry policy
   IRetryPolicy retryPolicy = new ExponentialBackoff(RetryCount, TimeSpan.FromSeconds(10), 
     TimeSpan.FromSeconds(60), TimeSpan.FromSeconds(5)); SetRetryPolicy(retryPolicy);
   ```

Le mécanisme de nouvelle tentative s’arrête après `DefaultOperationTimeoutInMilliseconds`, qui est actuellement défini sur 4 minutes.

#### <a name="other-languages-implementation-guidance"></a>Conseils d’implémentation pour d’autres langages

Pour obtenir des exemples de code dans d’autres langages, consultez les documents d’implémentation suivants. Le dépôt contient des exemples qui illustrent l’utilisation des API de stratégie de nouvelles tentatives.

* [Kit de développement logiciel (SDK) C/iOS](https://github.com/azure/azure-iot-sdk-c)

* [Kit de développement logiciel (SDK) .NET](https://github.com/Azure/azure-iot-sdk-csharp/blob/master/iothub/device/devdoc/retrypolicy.md)

* [Kit SDK Java](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/devdoc/requirement_docs/com/microsoft/azure/iothub/retryPolicy.md)

* [Node SDK](https://github.com/Azure/azure-iot-sdk-node/wiki/Connectivity-and-Retries#types-of-errors-and-how-to-detect-them)

* [Kit de développement logiciel (SDK) Python](https://github.com/Azure/azure-iot-sdk-python) (Fiabilité encore non implémentée)

## <a name="next-steps"></a>Étapes suivantes

* [Utiliser les Kits de développement logiciel (SDK) de services et d’appareils](./iot-hub-devguide-sdks.md)

* [Utiliser le Kit de développement logiciel (SDK) Azure IoT device pour C](./iot-hub-device-sdk-c-intro.md)

* [Développer pour des appareils avec contraintes](./iot-hub-devguide-develop-for-constrained-devices.md)

* [Développer pour appareils mobiles](./iot-hub-how-to-develop-for-mobile-devices.md)

* [Résoudre les problèmes de déconnexion des appareils](iot-hub-troubleshoot-connectivity.md)