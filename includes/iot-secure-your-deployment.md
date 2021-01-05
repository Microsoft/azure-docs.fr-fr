---
title: Fichier Include
description: Fichier Include
services: iot-fundamentals
author: robinsh
ms.service: iot-fundamentals
ms.topic: include
ms.date: 08/07/2018
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: 08cca67455df4b2d28bba0a7410fccc11446fcdc
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010694"
---
Cet article fournit davantage de détails sur la sécurisation de l’infrastructure Internet des objets (IoT) basée sur Azure IoT. Il est lié à la procédure au niveau de l’implémentation pour la configuration et le déploiement de chaque composant. Il fournit également des comparaisons et des choix entre les différentes méthodes.

La sécurisation du déploiement Azure IoT peut être divisée selon les trois zones de sécurité suivantes :

* **Sécurité de l’appareil**: sécurisation de l’appareil IoT alors que celui-ci est déployé dans la nature.

* **Sécurité de la connexion**: s’assurer que toutes les données transmises entre l’appareil IoT et IoT Hub sont confidentielles et protégées contre la falsification.

* **Sécurité du cloud**: fournir un moyen de sécuriser les données lorsque celles-ci transitent par le cloud ou sont stockées sur celui-ci.

![Trois zones de sécurité](./media/iot-secure-your-deployment/overview.png)

## <a name="secure-device-provisioning-and-authentication"></a>Approvisionnement et authentification sécurisés des appareils

Les accélérateurs de solution IoT sécurisent les appareils IoT de deux façons :

* En fournissant une clé d’identité unique (jetons de sécurité) pour chaque appareil, qui peut être utilisée par l’appareil pour communiquer avec IoT Hub.

* À l’aide d’un [certificat X.509](https://www.itu.int/rec/T-REC-X.509-201210-S) et d’une clé privée sur l’appareil en vue d’authentifier l’appareil auprès de l’IoT Hub. Cette méthode d’authentification permet de s’assurer que la clé privée sur l’appareil reste confidentielle à tout moment, offrant ainsi un niveau de sécurité plus élevé.

La méthode du jeton de sécurité fournit une authentification pour chaque appel effectué par l’appareil à IoT Hub en associant la clé symétrique à chaque appel. L’authentification basée sur le certificat X.509 permet l’authentification d’un appareil IoT au niveau physique dans le cadre de l’établissement d’une connexion TLS. La méthode basée sur le jeton de sécurité peut être utilisée sans l’authentification X.509 qui constitue un modèle moins sécurisé. Le choix entre les deux méthodes dépend principalement du niveau requis pour l’authentification de l’appareil et de la disponibilité d’un stockage sécurisé sur l’appareil (pour stocker la clé privée en toute sécurité).

## <a name="iot-hub-security-tokens"></a>Jetons de sécurité IoT Hub

IoT Hub utilise des jetons de sécurité pour authentifier les appareils et les services afin d’éviter d’envoyer des clés sur le réseau. En outre, la validité et la portée des jetons sont limitées dans le temps. Les Kits de développement logiciel (SDK) Azure IoT Hub génèrent automatiquement les jetons sans configuration spéciale. Certains scénarios requièrent toutefois que l’utilisateur génère et utilise directement des jetons de sécurité. Ces scénarios peuvent comprendre l’utilisation directe des surfaces MQTT, AMQP ou HTTP, ou de l’implémentation du modèle de service de jeton.

Vous trouverez plus d’informations sur la structure du jeton de sécurité et son utilisation dans les articles suivants :

* [Structure du jeton de sécurité](../articles/iot-hub/iot-hub-devguide-security.md#security-token-structure)

* [Utilisation de jetons SAS comme appareil](../articles/iot-hub/iot-hub-devguide-security.md#use-sas-tokens-in-a-device-app)

Chaque IoT Hub dispose d’un [registre d’identité](../articles/iot-hub/iot-hub-devguide-identity-registry.md) servant à créer des ressources par appareil dans le service, par exemple, une file d’attente contenant des messages actuels envoyés du cloud vers l’appareil. En outre, il autorise l’accès aux points de terminaison des appareils. Le registre des identités IoT Hub stocke de manière sécurisée les identités des appareils et les clés de sécurité d’une solution. Des identités d’appareil peuvent être ajoutées à une liste verte ou une liste rouge, individuellement ou en groupe, permettant un contrôle total de l’accès à l’appareil. Les articles suivants fournissent plus de détails sur la structure du registre d’identité et les opérations prises en charge.

[IoT Hub prend en charge les protocoles tels que MQTT, AMQP et HTTP](../articles//iot-hub/iot-hub-devguide-security.md). Chacun de ces protocoles utilise des jetons de sécurité à partir de l’appareil IoT vers IoT Hub différemment :

* AMQP : sécurité basée sur des revendications AMQP et SASL PLAIN (`{policyName}@sas.root.{iothubName}` avec des jetons au niveau d’IoT Hub ; `{deviceId}` dans le cas de jetons à l’échelle de l’appareil).

* MQTT : le paquet CONNECT utilise `{deviceId}` en tant que `{ClientId}`, `{IoThubhostname}/{deviceId}` dans le champ **Nom d’utilisateur** et un jeton SAS dans le champ **Mot de passe**.

* HTTP : le jeton valide se trouve dans l’en-tête de la requête d’autorisation.

Vous pouvez utiliser le registre d’identité IoT Hub pour configurer les informations d’identification de sécurité et le contrôle d’accès par appareil. Cependant, si une solution IoT représente déjà un investissement significatif dans un [registre d’identité des appareils personnalisé et/ou un schéma d’authentification](../articles/iot-hub/iot-hub-devguide-security.md#custom-device-and-module-authentication), vous pouvez intégrer cette infrastructure existante à IoT Hub en créant un service de jeton.

### <a name="x509-certificate-based-device-authentication"></a>Authentification des appareils basée sur un certificat X.509

L’utilisation d’un [certificat X.509 basé sur un appareil](../articles/iot-hub/iot-hub-devguide-security.md) et sa paire de clés privée et publique associée permet d’obtenir une authentification supplémentaire au niveau de la couche physique. La clé privée est stockée en toute sécurité dans l’appareil et n’est pas détectable en dehors de l’appareil. Le certificat X.509 contient des informations sur l’appareil, telles que l’identifiant de l’appareil et d’autres détails organisationnels. Une signature du certificat est générée à l’aide de la clé privée.

Flux d’approvisionnement des appareils de haut niveau :

* Associer un identificateur à un appareil physique : identité de l’appareil et/ou certificat X.509 associés à l’appareil lors de la fabrication ou de la mise en service de l’appareil.

* Créer une entrée d’identité correspondante dans IoT Hub : identité de l’appareil et informations connexes sur l’appareil dans le Registre d’identité IoT Hub.

* Stocker en toute sécurité l’empreinte numérique du certificat X.509 dans le Registre d’identité IoT Hub.

### <a name="root-certificate-on-device"></a>Certificat racine sur un appareil

Lors de l’établissement d’une connexion TLS sécurisée avec IoT Hub, l’appareil IoT authentifie IoT Hub à l’aide d’un certificat racine qui fait partie du Kit de développement logiciel (SDK) de l’appareil. Pour le Kit de développement logiciel (SDK) du client C, le certificat se trouve dans le dossier « \\c\\certs » sous la racine du référentiel. Bien que ces certificats racine soient de longue durée, ils peuvent malgré tout expirer ou être révoqués. S’il n’existe aucun moyen de mettre à jour le certificat sur l’appareil, il est possible que celui-ci ne puisse pas se connecter ultérieurement à IoT Hub (ou tout autre service cloud). Pour minimiser ce risque, nous vous conseillons de disposer d’un moyen de mettre à jour le certificat racine une fois l’appareil IoT déployé.

## <a name="securing-the-connection"></a>Sécurisation de la connexion

La connexion Internet entre l’appareil IoT et IoT Hub est sécurisée à l’aide de la norme TLS (Transport Layer Security). Azure IoT prend en charge [TLS 1.2](https://tools.ietf.org/html/rfc5246), TLS 1.1 et TLS 1.0, dans cet ordre. La prise en charge de TLS 1.0 est fournie uniquement à des fins de compatibilité descendante. Consultez [Prise en charge de TLS dans Iot Hub](../articles/iot-hub/iot-hub-tls-support.md) pour savoir comment configurer votre hub de manière à utiliser TLS 1.2, car il offre le plus de sécurité.

## <a name="securing-the-cloud"></a>Sécurisation du cloud

Azure IoT Hub permet la définition de [stratégies de contrôle d’accès](../articles/iot-hub/iot-hub-devguide-security.md) pour chaque clé de sécurité. Azure IoT Hub utilise l’ensemble d’autorisations qui suit pour accorder l’accès à chaque point de terminaison IoT Hub. Les autorisations limitent l’accès à un IoT Hub selon la fonctionnalité.

* **RegistryRead**. Accorde l’accès en lecture au registre des identités. Pour plus d’informations, consultez [Registre des identités](../articles/iot-hub/iot-hub-devguide-identity-registry.md).

* **RegistryReadWrite**. Accorde l’accès en lecture et en écriture au registre des identités. Pour plus d’informations, consultez [Registre des identités](../articles/iot-hub/iot-hub-devguide-identity-registry.md).

* **ServiceConnect**. Accorde l’accès à la communication de services cloud et à la surveillance des points de terminaison. Par exemple, elle permet aux services cloud principaux de recevoir des messages appareil-à-cloud, d’envoyer des messages cloud-à-appareil et de récupérer les accusés de remise correspondants.

* **DeviceConnect**. Accorde l’accès aux points de terminaison côté appareil. Par exemple, elle permet d’envoyer des messages appareil-à-cloud et de recevoir des messages cloud-à-appareil. Cette autorisation est utilisée par les appareils.

Il existe deux façons d’obtenir des autorisations **DeviceConnect** avec IoT Hub avec des [jetons de sécurité](../articles/iot-hub/iot-hub-devguide-security.md#use-sas-tokens-in-a-device-app) : avec une clé d’identité d’appareil ou une clé d’accès partagé. En outre, il est important de noter que toutes les fonctionnalités accessibles à partir des appareils sont exposées dès le départ sur les points de terminaison avec le préfixe `/devices/{deviceId}`.

Les [composants de service peuvent uniquement créer des jetons de sécurité](../articles/iot-hub/iot-hub-devguide-security.md#use-security-tokens-from-service-components) utilisant des stratégies d’accès partagé pour accorder les autorisations adaptées.

Azure IoT Hub et d’autres services pouvant faire partie de la solution autorisent la gestion des utilisateurs à l’aide d’Azure Active Directory.

Les données reçues par Azure IoT Hub peuvent être utilisées par une variété de services comme Azure Stream Analytics et le stockage d’objets blob. Ces services permettent un accès en gestion. En savoir plus sur ces services et les options disponibles :

* [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) : service de base de données évolutif et entièrement indexé pour données semi-structurées, qui gère les métadonnées (attributs, configuration, propriétés de sécurité, etc.) des appareils que vous provisionnez. Azure Cosmos DB assure un traitement hautes performances et à débit élevé, ainsi qu’une indexation des données indépendante du schéma. Ce service offre également une interface de requête SQL enrichie.

* [Azure Stream Analytics](https://azure.microsoft.com/services/stream-analytics/) : traitement des flux en temps réel dans le cloud, permettant de développer et de déployer rapidement une solution d’analyse à faible coût pour obtenir des informations en temps réel de la part des appareils, capteurs, infrastructures et applications. Les données de ce service entièrement géré peuvent être mises à l’échelle selon n’importe quel volume. Vous continuez à bénéficier d’un débit élevé, d’une faible latence et de la résilience.

* [Azure App Services](https://azure.microsoft.com/services/app-service/) : plateforme cloud permettant de créer de puissantes applications web et mobiles qui se connectent aux données n’importe où, que ce soit localement ou dans le cloud. Créez des applications mobiles attrayantes pour iOS, Android et Windows. Assurez l’intégration avec vos applications SaaS (Software as a Service) et d’entreprise grâce à une connectivité prête à l’emploi à des dizaines d’applications et services basés sur le cloud. Encodez dans votre environnement de développement intégré et votre langage favoris (.NET, Node.js, PHP, Python ou Java) pour créer rapidement des applications web et des API.

* [Logic Apps](https://azure.microsoft.com/services/app-service/logic/) : fonctionnalité Azure App Service qui vous aide à intégrer votre solution IoT dans vos systèmes métier existants et à automatiser les processus de flux de travail. Logic Apps permet aux développeurs de concevoir des flux de travail qui démarrent à partir d’un déclencheur et exécutent une série d’étapes — des règles et actions utilisant des connecteurs puissants pour l’intégration à vos processus d’entreprise. Logic Apps offre une connectivité prête à l’emploi vers un vaste écosystème d’applications SaaS, basées sur le cloud et locales.

* [Stockage Blob Azure](https://azure.microsoft.com/services/storage/) : stockage cloud fiable et économique pour les données que vos appareils envoient au cloud.

## <a name="conclusion"></a>Conclusion

Cet article fournit une vue d’ensemble des détails au niveau de l’implémentation pour concevoir et déployer une infrastructure IoT à l’aide d’Azure IoT. La configuration de chaque composant afin qu’il soit sécurisé est essentielle pour sécuriser l’infrastructure IoT dans son ensemble. Les choix de conception disponibles dans Azure IoT offrent un certain degré de souplesse et de choix. Toutefois, chaque choix peuvent avoir des implications en matière de sécurité. Nous vous recommandons d’évaluer chacun de ces choix à l’aide d’une évaluation des risques et du coût.
