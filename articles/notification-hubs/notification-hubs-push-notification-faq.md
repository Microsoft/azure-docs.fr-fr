---
title: Forum aux questions (FAQ) à propos d’Azure Notification Hubs | Microsoft Docs
description: Forum aux questions à propos de la conception et l’implémentation de solutions sur Azure Notification Hubs.
services: notification-hubs
documentationcenter: mobile
author: sethmanheim
manager: femila
keywords: notification push, notifications push, notifications push iOS, notifications push android, push ios, push android
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: multiple
ms.topic: article
ms.date: 11/13/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 11/13/2019
ms.openlocfilehash: 9d476b1db645ed1f91b62fcf11464f7077a8fb3c
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94491424"
---
# <a name="push-notifications-with-azure-notification-hubs-frequently-asked-questions"></a>Notifications Push avec Azure Notification Hubs : Forum aux questions

## <a name="general"></a>Général

### <a name="what-is-the-resource-structure-of-notification-hubs"></a>Quelle est la structure des ressources des Notification Hubs ?

Azure Notification Hubs a deux niveaux de ressources : les concentrateurs et les espaces de noms. Un concentrateur est une ressource push unique qui peut contenir les informations push interplateformes d’une application. Un espace de noms est une collection de concentrateurs dans une région. Le mappage recommandé consiste à mettre en correspondance un espace de noms et une application. Dans un espace de noms, vous pouvez avoir un concentrateur de production qui fonctionne avec votre application de production, un concentrateur de test qui fonctionne avec votre application de test, etc.

### <a name="what-is-the-price-model-for-notification-hubs"></a>Quel modèle de prix s’applique au service Notification Hubs ?

Pour les détails de tarification les plus récents, consultez la page [Tarification de Concentrateurs de notification]. Notification Hubs est facturé au niveau de l’espace de noms. (pour la définition d’un espace de noms, consultez « Quelle est la structure des ressources des Notification Hubs ? ») Notification Hubs propose trois niveaux :

* **Gratuit** : ce niveau est un bon point de départ pour explorer les fonctionnalités Push. Il n’est pas recommandé pour les applications de production. Vous obtenez 500 appareils et 1 million de notifications Push inclus par espace de noms par mois, sans garantie de contrat de niveau de service.
* **De base** : ce niveau, (ou le niveau Standard) est recommandé pour les applications de production plus petites. Vous obtenez 200 000 appareils et 10 millions de notifications Push inclus par espace de noms par mois comme point de départ.
* **Standard** : ce niveau est recommandé pour les applications de production de moyenne à grande taille. Vous obtenez 10 millions d’appareils et 10 millions de notifications Push inclus par espace de noms par mois comme point de départ. Inclut des données de télémétrie enrichies (données supplémentaires sur l’état push fourni).

Fonctionnalités du niveau standard :

* **Télémétrie enrichie** : vous pouvez utiliser la télémétrie par message de Notification Hubs pour effectuer le suivi des demandes d’opérations Push et les commentaires Platform Notification System pour le débogage.
* **Multilocation** : vous pouvez travailler avec les informations d’identification PNS (Platform Notification System) au niveau de l’espace de noms. Cette option vous permet de facilement répartir les locataires dans différents concentrateurs dans le même espace de noms.
* **Notification Push planifiée** : vous pouvez planifier l’envoi de notifications à tout moment.
* **Opérations en bloc** : active la fonctionnalité d’exportation/importation des inscriptions décrite dans le document [Exportation et modification d’inscriptions en bloc].

### <a name="what-is-the-notification-hubs-sla"></a>Qu’est-ce que le contrat SLA de Notification Hubs ?

Pour les niveaux De base et Standard de Notification Hubs, les applications correctement configurées peuvent envoyer des notifications push ou effectuer des opérations de gestion d’inscription au moins 99,9 % du temps. Pour plus d’informations sur le contrat de niveau de service, voir la page [Contrat SLA de Notification Hubs](https://azure.microsoft.com/support/legal/sla/notification-hubs/) .

> [!NOTE]
> Étant donné que les notifications Push dépendent de Platform Notification System tiers, tels que Push Notification Service (APNs) d’Apple et Firebase Cloud Messaging (FCM) de Google, il n’existe aucun contrat de niveau de service pour la remise de ces messages. Une fois que Notification Hubs a envoyé les lots aux Platform Notification Systems (contrat de niveau de service garanti), il est de la responsabilité de ces derniers de remettre les notifications Push (aucun contrat de niveau de service garanti).

### <a name="how-do-i-upgrade-or-downgrade-my-hub-or-namespace-to-a-different-tier"></a>Comment mettre à niveau ou rétrograder mon concentrateur ou mon espace de noms pour modifier son niveau ?

Accédez au **[Azure portal]**  > **Espaces de noms Notification Hubs** ou **Notification Hubs**. Sélectionnez la ressource que vous souhaitez mettre à jour et accédez au **Niveau de tarification**. Notez les exigences suivantes :

* Le nouveau niveau de tarification s’applique à *tous* les concentrateurs de l’espace de noms que vous utilisez.
* Si votre nombre d’appareils dépasse la limite du niveau inférieur vers lequel vous basculez, vous devez supprimer des appareils avant de procéder à cette opération.

## <a name="design-and-development"></a>Conception et développement

### <a name="which-server-side-platforms-do-you-support"></a>Quelles plateformes côté service prenez-vous en charge ?

Des kits de développement logiciel (SDK) sur serveur pour .NET, Java, Node.js, PHP et Python sont disponibles. En outre, les API de Notification Hubs reposent sur les interfaces REST. Vous pouvez donc choisir de travailler directement avec les API REST si vous utilisez différentes plateformes ou si vous ne souhaitez pas de dépendance supplémentaire. Pour plus d’informations, voir la page [API REST de Notification Hubs].

### <a name="which-client-platforms-do-you-support"></a>Quelles plateformes clientes prenez-vous en charge ?

Les notifications Push sont prises en charge pour [iOS](ios-sdk-get-started.md), [Android](notification-hubs-android-push-notification-google-fcm-get-started.md), [Windows Universal](notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md), [Windows Phone](notification-hubs-windows-mobile-push-notifications-mpns.md), [Android China (par Baidu)](notification-hubs-baidu-china-android-notifications-get-started.md), Xamarin [iOS](xamarin-notification-hubs-ios-push-notification-apns-get-started.md) et [Android](xamarin-notification-hubs-push-notifications-android-gcm.md), et [Safari](https://github.com/Azure/azure-notificationhubs-samples/tree/master/PushToSafari). Pour plus d’informations, voir la page [Notification Hubs Getting Started tutorials](ios-sdk-get-started.md) (Tutoriels de prise en main de Notification Hubs).

### <a name="do-you-support-text-message-email-or-web-notifications"></a>Prenez-vous en charge les SMS, les emails ou les notifications web ?

Notification Hubs envoie des notifications aux appareils utilisant des applications mobiles. Il ne propose pas de fonctionnalités de courrier électronique ou de SMS. Notification Hubs ne fournit pas non plus de service prêt à l’emploi de remise de notification Push dans le navigateur. Les clients peuvent implémenter cette fonctionnalité en utilisant SignalR par-dessus les plateformes prises en charge côté serveur. 

### <a name="how-many-devices-can-i-support-if-i-send-push-notifications-via-notification-hubs"></a>Combien d’appareils puis-je prendre en charge si j’envoie des notifications Push via Notification Hubs ?

Pour plus d’informations sur le nombre d’appareils pris en charge, voir la [Tarification de Concentrateurs de notification].

Si vous devez prendre en charge plus de 10 millions d’appareils inscrits, vous devez partitionner vos appareils sur plusieurs espaces de noms.

### <a name="how-many-push-notifications-can-i-send-out"></a>Combien de notifications Push puis-je envoyer ?

Selon le niveau sélectionné, Azure Notification Hubs est automatiquement mis à l’échelle en fonction du nombre de notifications transitant par le système.

> [!NOTE]
> Le coût d’utilisation global peut augmenter en fonction du nombre de notifications Push envoyées. Assurez-vous que vous comprenez les limites de niveau décrites dans la page [Tarification de Concentrateurs de notification] .

Nos clients utilisent Notification Hubs pour envoyer des millions de notifications Push quotidiennement. Vous n’avez rien de spécial à faire pour mettre à l’échelle la portée de vos notifications Push tant que vous utilisez Azure Notification Hubs.

### <a name="how-long-does-it-take-for-sent-push-notifications-to-reach-my-device"></a>Combien de temps faut-il pour que des notifications Push atteignent mon appareil ?

Azure Notification Hubs peut traiter au moins *un million d’envois de notifications Push par minute* dans un scénario d’utilisation normale, où la charge entrante est relativement homogène et régulière. Ce taux peut varier en fonction du nombre de balises, de la nature des envois entrants et d’autres facteurs externes.

Pendant le délai de livraison estimé, le service calcule les cibles par plateforme et achemine des messages vers le service de notification Push sur la base des balises/expressions de balises inscrites. Il est de la responsabilité du service de notification Push d’envoyer des notifications à l’appareil.

Le service de notification Push ne garantit aucun contrat de niveau de service pour la remise des notifications. Toutefois, la plupart des notifications Push sont remises aux appareils cibles en quelques minutes (généralement dans les 10 minutes) depuis le moment où elles sont envoyées aux hubs de notification. Certaines notifications peuvent prendre plus de temps.

> [!NOTE]
> Azure Notification Hubs applique une stratégie consistant à supprimer toutes les notifications Push qui ne peuvent pas être remises au PNS dans un délai de 30 minutes. Cela peut se produire pour plusieurs raisons, la plus courante étant une limitation imposée par le PNS sur votre application.

### <a name="is-there-any-latency-guarantee"></a>Y a-t-il une garantie de latence ?

En raison de la nature même des notifications Push, qui sont remises par un service de notifications Push spécifique d’une plateforme externe, il n’y a aucune garantie de latence. En règle générale, la majorité des notifications Push sont remises après quelques minutes.

### <a name="what-do-i-need-to-consider-when-designing-a-solution-with-namespaces-and-notification-hubs"></a>Quels sont les points à prendre en compte lors de la conception d’une solution avec des espaces de noms et hubs de notification ?

#### <a name="mobile-appenvironment"></a>Application mobile/Environnement

* Utilisez un hub de notification par application mobile et par environnement.
* Dans un scénario d’architecture mutualisée, chaque locataire doit disposer d’un concentrateur distinct.
* Ne partagez jamais le même hub de notification pour les environnements de production et de test. Cela peut entraîner des problèmes lors de l’envoi de notifications. Apple fournit des points de terminaison Push pour les environnements de bac à sable (sandbox) et de production, chaque environnement étant défini avec des informations d’identification distinctes.
* Par défaut, vous pouvez envoyer des notifications de test à vos appareils inscrits via le portail Azure ou le composant intégré Azure dans Visual Studio. Le seuil est défini sur 10 appareils sélectionnés au hasard dans le pool d’inscription.

> [!NOTE]
> Si votre hub a été initialement configuré avec un certificat de bac à sable Apple et a été reconfiguré pour utiliser un certificat de production d’Apple, les jetons de l’appareil d’origine ne sont pas valides. Des jetons non valides entraînent l’échec des notifications Push. Séparez vos environnements de test et de production et utilisez des hubs différents pour chaque environnement.

#### <a name="pns-credentials"></a>Informations d’identification du PNS

Lorsqu’une application mobile est inscrite via un portail des développeurs de plate-forme (par exemple, Apple ou Google), un identificateur d’application et des jetons de sécurité sont envoyés. Le serveur principal d’applications fournit ces jetons au PNS de la plateforme afin que des notifications Push puissent être envoyées aux appareils. Les jetons de sécurité peuvent être des certificats (par exemple, Apple iOS ou Windows Phone) ou des clés de sécurité (par exemple, Google Android ou Windows). Ils doivent être configurés dans hubs de notification. Leur configuration s’effectue généralement au niveau du hub de notification, mais peut aussi être réalisée au niveau de l’espace de noms dans un scénario d’architecture mutualisée.

#### <a name="namespaces"></a>Espaces de noms

Des espaces de noms peuvent être utilisés dans le cadre du regroupement de déploiement. Dans un scénario d’architecture mutualisée, ils permettent aussi de représenter tous les hubs de notification de tous les locataires de la même application.

#### <a name="geo-distribution"></a>Géo-distribution

La géo-distribution n’est pas toujours indispensable dans les scénarios de notification Push. Les divers PNS (par exemple, APNs ou FCM) qui envoient des notifications Push aux appareils ne sont pas distribués de manière uniforme.

Si vous disposez d’une application utilisée de manière globale, vous pouvez créer des hubs dans différents espaces de noms en utilisant le service Notification Hubs dans différentes régions Azure à travers le monde.

> [!NOTE]
> Cette disposition est déconseillée car elle augmente le coût de gestion, en particulier pour les inscriptions. Elle doit être utilisée uniquement en cas de besoin.

### <a name="should-i-do-registrations-from-the-app-backend-or-directly-through-client-devices"></a>Dois-je effectuer les inscriptions à partir du service principal d’application ou directement via des appareils clients ?

Les inscriptions à partir du serveur principal d’applications sont utiles lorsque vous devez authentifier les clients avant de créer l’inscription. Elles sont également utiles lorsque vous avez des balises qui doivent être créées ou modifiées par serveur principal d’applications en se basant sur la logique de l’application. Pour plus d’informations, consultez les pages [Aide sur l’inscription auprès du serveur principal] et [Aide sur l’inscription auprès du serveur principal - 2].

### <a name="what-is-the-push-notification-delivery-security-model"></a>Quel est le modèle de sécurité de remise de notification Push ?

Azure Notification Hubs utilise un modèle de sécurité basé sur une [signature d’accès partagé](../storage/common/storage-sas-overview.md). Vous pouvez utiliser les jetons SAP au niveau de l’espace de noms racine ou au niveau granulaire du hub de notification. Les jetons SAP peuvent être définis pour respecter différentes règles d’autorisation, par exemple, pour envoyer des autorisations de messages ou pour écouter des autorisations de notification. Pour plus d’informations, consultez le document [Modèle de sécurité de Notification Hubs] (Modèle de sécurité de Notification Hubs).

### <a name="how-should-i-handle-sensitive-payload-in-push-notifications"></a>Comment gérer une charge utile sensible dans des notifications Push ?

Toutes les notifications sont remises à des appareils cibles par les services de notification Push des plateformes. Lorsqu’une notification est envoyée à Azure Notification Hubs, celle-ci est traitée et transmise au PNS respectif.

Toutes les connexions entre l’expéditeur, Azure Notifications Hubs et le PNS utilisent le protocole HTTPS.

> [!NOTE]
> Azure Notifications Hubs n’enregistre pas la charge utile des messages.

Pour envoyer des charges utiles sensibles, nous recommandons l’utilisation d’un modèle de notifications Push sécurisées. L’expéditeur remet une notification Ping avec un identificateur de message à l’appareil sans la charge utile sensible. Lorsque l’application de l’appareil reçoit la charge utile, l’application appelle directement une API sécurisée pour extraire les détails du message. Pour savoir comment implémenter ce modèle, accédez à la page [didacticiel sur les notifications Push sécurisées avec Azure Notification Hubs].

## <a name="operations"></a>Opérations

### <a name="what-support-is-provided-for-disaster-recovery"></a>Quelle prise en charge est fournie pour la récupération d’urgence ?

Nous assurons la couverture de la récupération d’urgence des métadonnées (nom de hub de notification, chaîne de connexion et autres informations critiques). Quand un scénario de récupération d’urgence est déclenché, les données d’inscription sont le *seul segment* de l’infrastructure de Notification Hubs qui est perdu. Vous devez implémenter une solution pour réinsérer ces données dans votre nouveau hub après la récupération :

1. Créez un hub de notification secondaire dans un autre centre de données. Nous vous recommandons d’en créer un dès le début pour vous protéger d’une reprise après sinistre qui peut affecter vos capacités de gestion. Vous pouvez également créer un hub au moment de l’événement de récupération d’urgence.

2. Veillez à ce que le hub de notification secondaire soit synchronisé avec le hub de notification principal en utilisant l’une des options suivantes :

   * Utilisez un serveur principal d’application qui crée et met à jour simultanément des installations dans les deux hubs de notification. Ces installations vous permettent de spécifier votre propre identificateur d’appareil unique, ce qui le rend plus adapté au scénario de réplication. Pour plus d’informations, consultez [cet exemple de code](https://github.com/Azure/azure-notificationhubs-dotnet/tree/main/Samples/RedundantHubSample).
   * Utilisez un serveur principal d’applications qui obtient un vidage régulier des inscriptions à partir du hub de notification principal sous forme de sauvegarde. Il peut alors effectuer une insertion en bloc dans le hub de notification secondaire.

Le hub de notification secondaire peut se retrouver avec des installations/inscriptions ayant expiré. Lorsque la notification Push est effectuée vers un handle expiré, Notification Hubs nettoie automatiquement l’enregistrement d’installation/inscription associé en fonction de la réponse reçue du serveur PNS. Pour nettoyer les enregistrements expirés d’un hub de notification secondaire, ajoutez une logique personnalisée qui traite les informations reçues de chaque envoi. Ensuite, faites expirer l’installation/l’inscription dans le hub de notification secondaire.

Si vous n’avez de serveur principal d’applications, lorsque l’application démarre sur les appareils cibles, ces derniers effectuent une nouvelle inscription dans le hub de notification secondaire. Finalement, tous les appareils actifs sont inscrits dans le hub de notification secondaire.

Pendant une certaine période, les appareils sur lesquels aucune application n’est ouverte ne reçoivent pas de notifications.

### <a name="is-all-of-my-data-stored-in-encrypted-form"></a>Toutes mes données sont-elles stockées sous une forme chiffrée ?

Azure Notification Hubs chiffre toutes les données client au repos à l’exception des balises d’inscription. C’est pourquoi vous ne devez pas stocker de données personnelles ou confidentielles avec des balises.

### <a name="is-there-audit-log-capability"></a>Existe-t-il une fonctionnalité de journal d’audit ?

Oui. Toutes les opérations de gestion de Notification Hubs mettent à jour le journal d’activité Azure dans les journaux des opérations qui est exposé dans le [Azure portal]. Le journal d’activité Azure fournit des insights sur les opérations effectuées sur des ressources de vos abonnements. Avec le journal d’activité, vous pouvez déterminer qui, quand et quoi pour toutes les opérations d’écriture (PUT, POST, DELETE) effectuées sur les ressources de votre abonnement. Vous pouvez également comprendre l’état des opérations et d’autres propriétés pertinentes. Toutefois, le journal d’activité n’inclut pas l’opération de lecture (GET).

## <a name="monitoring-and-troubleshooting"></a>Surveillance et dépannage

### <a name="what-troubleshooting-capabilities-are-available"></a>Quelles sont les fonctionnalités proposées pour faciliter la résolution des problèmes ?

Azure Notification Hubs fournit plusieurs fonctionnalités de dépannage, en particulier pour la suppression de notifications, qui est le scénario le plus fréquent. Pour plus d’informations, consultez le livre blanc [Dépannage de Notification Hubs].

### <a name="what-telemetry-features-are-available"></a>Quelles sont les fonctionnalités de télémétrie proposées ?

Azure Notification Hubs permet d’afficher des données de télémétrie sur le [Azure portal]. Pour plus de détails sur les mesures disponibles, voir la page [Mesures de Notification Hubs].

Vous pouvez également accéder aux mesures de manière programmatique. Pour plus d’informations, consultez les articles suivants :

- [Récupérer les métriques Azure Monitor avec .NET](https://azure.microsoft.com/resources/samples/monitor-dotnet-metrics-api/). Cet exemple utilise le nom d’utilisateur et le mot de passe. Pour utiliser un certificat, surchargez la méthode FromServicePrincipal pour fournir un certificat, comme illustré dans [cet exemple](https://github.com/Azure/azure-libraries-for-net/blob/master/src/ResourceManagement/ResourceManager/Authentication/AzureCredentialsFactory.cs). 
- [Obtention de métriques et journaux d’activité pour une ressource](https://azure.microsoft.com/resources/samples/monitor-dotnet-query-metrics-activitylogs/)
- [Procédure pas à pas d’utilisation de l’API REST d’Azure Monitor](../azure-monitor/platform/rest-api-walkthrough.md)

> [!NOTE]
> Les notifications réussies signifient simplement que les notifications Push ont été remises au PNS externe (par exemple, APNs pour iOS et macOS ou FCM pour les appareils Android). Il est de la responsabilité du service de notification Push d’envoyer des notifications aux appareils cibles. En règle générale, le PNS n’expose pas les métriques de remise aux tiers.  

[Azure portal]: https://portal.azure.com
[Tarification de Concentrateurs de notification]: https://azure.microsoft.com/pricing/details/notification-hubs/
[Notification Hubs SLA]: https://azure.microsoft.com/support/legal/sla/
[API REST de Notification Hubs]: /previous-versions/azure/reference/dn530746(v=azure.100)
[Mobile Services Pricing]: https://azure.microsoft.com/pricing/details/mobile-services/
[Aide sur l’inscription auprès du serveur principal]: /previous-versions/azure/azure-services/dn743807(v=azure.100)
[Aide sur l’inscription auprès du serveur principal - 2]: /previous-versions/azure/azure-services/dn530747(v=azure.100)
[Modèle de sécurité de Notification Hubs]: /previous-versions/azure/azure-services/dn495373(v=azure.100)
[didacticiel sur les notifications Push sécurisées avec Azure Notification Hubs]: ./notification-hubs-aspnet-backend-ios-push-apple-apns-secure-notification.md
[Dépannage de Notification Hubs]: ./notification-hubs-push-notification-fixer.md
[Mesures de Notification Hubs]: ../azure-monitor/platform/metrics-supported.md#microsoftnotificationhubsnamespacesnotificationhubs
[Exportation et modification d’inscriptions en bloc]: ./export-modify-registrations-bulk.md
[Azure portal]: https://portal.azure.com
[complete samples]: https://github.com/Azure/azure-notificationhubs-samples
[App Service Pricing]: https://azure.microsoft.com/pricing/details/app-service/
