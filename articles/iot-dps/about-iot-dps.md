---
title: Vue d’ensemble du service Azure IoT Hub Device Provisioning | Microsoft Docs
description: Décrit le provisionnement d’appareils dans Azure avec le service IoT Hub Device Provisioning (DPS)
author: wesmc7777
ms.author: wesmc
ms.date: 04/04/2019
ms.topic: overview
ms.service: iot-dps
services: iot-dps
manager: eliotgra
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: 6b33b866a10ad4a44cef14f3c86d8ca1f40c4750
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94965372"
---
# <a name="provisioning-devices-with-azure-iot-hub-device-provisioning-service"></a>Approvisionner des appareils avec le service Azure IoT Hub Device Provisioning
Microsoft Azure fournit un ensemble complet de services cloud publics intégrés qui répondent à tous les besoins de votre solution IoT. Le service IoT Hub Device Provisioning (DPS) est un service d’assistance pour IoT Hub qui autorise un provisionnement sans contact, juste-à-temps sur le hub IoT approprié et sans aucune intervention humaine. DPS permet de provisionner plusieurs millions d’appareils de manière sécurisée et scalable.

## <a name="when-to-use-device-provisioning-service"></a>Quand utiliser le service Device Provisioning
Nombreux sont les scénarios de provisionnement dans lesquels DPS s’avère être un excellent choix pour connecter et configurer des appareils sur IoT Hub, notamment :

* Approvisionnement sans contact sur une solution IoT unique sans coder en dur en usine les informations de connexion IoT Hub (configuration initiale)
* Équilibrage de la charge d’appareils sur plusieurs hubs
* Connexion d’appareils à la solution IoT de leur propriétaire en fonction des données de transaction de vente (architecture multilocataire)
* Connexion d’appareils à une solution IoT particulière en fonction des cas d’usage (isolement de la solution)
* Connexion d’un appareil au hub IoT avec la latence la plus faible (géopartitionnement)
* Reprovisionnement basé sur un changement au niveau de l’appareil
* Restaurer les clés utilisées par l’appareil pour se connecter à IoT Hub (en cas de non-utilisation de certificats X.509 pour la connexion)

## <a name="behind-the-scenes"></a>Dans les coulisses
Tous les scénarios listés dans la section précédente peuvent être réalisés en utilisant le service DPS pour un provisionnement sans contact avec le même flux. La plupart des étapes manuelles que nécessite généralement le provisionnement sont automatisées avec DPS pour réduire le temps de déploiement des appareils IoT et diminuer le risque d’erreur manuelle. La section suivante décrit les tâches effectuées en arrière-plan pour obtenir un appareil provisionné. La première étape est manuelle, mais toutes les suivantes sont automatiques.

![Flux d’approvisionnement de base](./media/about-iot-dps/dps-provisioning-flow.png)

1. Le fabricant de l’appareil ajoute les informations d’enregistrement de l’appareil à la liste d’inscriptions dans le portail Azure.
2. L’appareil contacte le point de terminaison DPS défini en usine. L’appareil passe les informations d’identification à DPS pour prouver son identité.
3. DPS valide l’identité de l’appareil en comparant l’ID et la clé d’inscription à l’entrée de liste d’inscriptions au moyen d’un test nonce ([Module de plateforme sécurisée](https://trustedcomputinggroup.org/work-groups/trusted-platform-module/)) ou d’une vérification X.509 standard (X.509).
4. DPS inscrit l’appareil sur un hub IoT et renseigne l’[état souhaité du jumeau](../iot-hub/iot-hub-devguide-device-twins.md) de l’appareil.
5. Le hub IoT retourne l’ID de l’appareil à DPS.
6. DPS retourne les informations de connexion du hub IoT à l’appareil. L’appareil peut maintenant commencer à envoyer des données directement au hub IoT.
7. L’appareil se connecte au hub IoT.
8. L’appareil obtient l’état souhaité de son jumeau dans le hub IoT.

## <a name="provisioning-process"></a>Processus de mise en service
Le processus de déploiement d’un appareil, dans lequel DPS joue un rôle qui peut être effectué indépendamment, se déroule en deux étapes distinctes :

* L’**étape de fabrication** qui permet de créer et préparer l’appareil en usine.
* L’**étape de configuration du cloud** qui configure le service Device Provisioning pour l’approvisionnement automatique.

Ces deux étapes s’intègrent parfaitement aux processus de fabrication et de déploiement existants. DPS simplifie même certains processus de déploiement qui nécessitent des tâches manuelles pour obtenir les informations de connexion sur l’appareil.

### <a name="manufacturing-step"></a>Étape de fabrication
Cette étape englobe toutes les tâches de la ligne de fabrication. Les rôles impliqués dans cette étape sont notamment le concepteur de silicium, le fabricant de silicium, l’intégrateur et/ou le fabricant final de l’appareil. Cette étape concerne la création du matériel.

DPS n’introduit pas une nouvelle étape dans le processus de fabrication ; il se lie plutôt à l’étape d’installation existante du logiciel initial et (dans l’idéal) du module HSM sur l’appareil. Dans cette étape, plutôt que de créer un ID d’appareil, l’appareil est programmé avec les informations du service de provisionnement pour qu’il puisse l’appeler afin d’obtenir son attribution de solution IoT/d’informations de connexion quand il est allumé.

Dans cette étape également, le fabricant fournit à l’opérateur ou au système de déploiement de l’appareil les informations de clé d’identification. Pour fournir ces informations, vous pouvez simplement confirmer que tous les appareils ont un certificat X.509 généré par une autorité de certification fournie par l’opérateur ou le responsable de déploiement de l’appareil, ou bien, ce qui est plus compliqué, extraire la partie publique d’une paire de clés de type EK TPM sur chaque appareil TPM. Ces services sont proposés par de nombreux fabricants de silicium aujourd'hui.

### <a name="cloud-setup-step"></a>Étape de configuration du cloud
Cette étape configure le cloud pour l’approvisionnement automatique approprié. Généralement, deux types d’utilisateurs sont impliqués dans l’étape de configuration du cloud : une personne qui sait comment les appareils doivent être configurés initialement (un opérateur d’appareil) et une autre personne qui sait comment les appareils doivent être répartis dans les hubs IoT (un opérateur de solution).

Le provisionnement doit passer par une configuration initiale unique qui est généralement gérée par l’opérateur de solution. Une fois le service d’approvisionnement configuré, il n’a pas besoin d’être modifié, sauf si le cas d’utilisation change.

Une fois que le service a été configuré pour la configuration automatique, il doit être préparé pour inscrire les appareils. Cette étape est effectuée par l’opérateur d’appareil, qui connaît la configuration souhaitée du ou des appareils et doit vérifier que le service d’approvisionnement peut correctement attester de l’identité de l’appareil au moment de rechercher son hub IoT. L’opérateur d’appareil récupère les informations de la clé d’identification auprès du fabricant et les ajoute à la liste d’inscriptions. La liste d’inscriptions peut être mise à jour par la suite, à mesure que de nouvelles entrées sont ajoutées ou que des entrées existantes sont mises à jour avec les dernières informations des appareils.

## <a name="registration-and-provisioning"></a>Enregistrement et approvisionnement
L’*approvisionnement* désigne différentes choses, selon le secteur dans lequel est utilisé le terme. Dans le contexte de l’approvisionnement d’appareils IoT dans une solution cloud, il désigne un processus en deux parties :

1. La première partie établit la connexion initiale entre l’appareil et la solution IoT en enregistrant l’appareil.
2. La deuxième partie applique la configuration appropriée à l’appareil selon les exigences spécifiques de la solution dans laquelle il est enregistré.

Une fois ces deux étapes terminées, on peut dire que l’appareil est entièrement provisionné. Certains services cloud fournissent uniquement la première étape du processus d’approvisionnement (l’enregistrement des appareils au point de terminaison de la solution IoT) et ne fournissent pas la configuration initiale. DPS automatise les deux étapes pour offrir une expérience de provisionnement sans interruption pour l’appareil.

## <a name="features-of-the-device-provisioning-service"></a>Fonctionnalités du service Device Provisioning
DPS propose de nombreuses fonctionnalités qui le rendent idéal pour le provisionnement d’appareils.

* Prise en charge de l’**attestation sécurisée** pour les identités X.509 et TPM.
* **Liste d’inscriptions** contenant l’enregistrement complet des appareils/groupes d’appareils susceptibles d’être enregistrés à un moment donné. La liste d’inscriptions contient des informations sur la configuration souhaitée de l’appareil après son enregistrement et peut être mise à jour à tout moment.
* **Plusieurs stratégies d’allocation** pour contrôler la façon dont DPS affecte les appareils aux hubs IoT en fonction de vos scénarios : Latence la plus faible, distribution uniformément pondérée (par défaut) et configuration statique par le biais de la liste d’inscriptions. La latence est déterminée selon la même méthode que celle utilisée par [Traffic Manager](../traffic-manager/traffic-manager-routing-methods.md#performance).
* **Monitoring et journalisation des diagnostics** pour vérifier que tout fonctionne correctement.
* La **prise en charge multihub** permet à DPM d’affecter des appareils à plusieurs hubs IoT. DPS peut communiquer avec les hubs de plusieurs abonnements Azure.
* La **prise en charge inter-régions** permet à DPS d’affecter des appareils à des hubs IoT d’autres régions.
* Le **chiffrement des données au repos** est conforme à la norme FIPS 140-2 et permet de chiffrer et déchiffrer les données dans DPM de façon transparente avec le chiffrement AES 256 bits, un des chiffrements par blocs les plus puissants actuellement disponibles.


Pour plus d’informations sur les concepts et les fonctionnalités impliqués dans le provisionnement des appareils, consultez la rubrique [Terminologie DPS](concepts-service.md) et les autres rubriques conceptuelles de la même section.

## <a name="cross-platform-support"></a>Prise en charge multiplateforme
Comme tous les services Azure IoT, DPS fonctionne sur plusieurs plateformes avec une variété de systèmes d’exploitation. Azure propose des kits SDK open source dans différents [langages](https://github.com/Azure/azure-iot-sdks), ce qui facilite la connexion des appareils et la gestion du service. DPS prend en charge les protocoles suivants pour la connexion des appareils :

* HTTPS
* AMQP
* AMQP sur WebSockets
* MQTT
* MQTT sur WebSockets

DPS prend uniquement en charge les connexions HTTPS pour les opérations de service.

## <a name="regions"></a>Régions
DPS est disponible dans de nombreuses régions. La liste des régions existantes et récemment annoncées pour tous les services est disponible dans [Régions Azure](https://azure.microsoft.com/regions/). Pour connaître la disponibilité du service Device Provisioning, accédez à la page [Statut Azure](https://azure.microsoft.com/status/).

> [!NOTE]
> DPS est mondial et n’est lié à aucune zone géographique. Cependant, vous devez spécifier la région dans laquelle les métadonnées associées à votre profil DPS résideront.

## <a name="availability"></a>Disponibilité
Le contrat de niveau de service (SLA) pour DPS offre une disponibilité de 99,9 %, et vous pouvez [lire le contrat SLA](https://azure.microsoft.com/support/legal/sla/iot-hub/). La version complète du [contrat SLA Azure](https://azure.microsoft.com/support/legal/sla/) explique la disponibilité garantie d’Azure dans son ensemble.

## <a name="quotas"></a>Quotas
Chaque abonnement Azure a des limites de quota par défaut qui peuvent impacter l’étendue de votre solution IoT. La limite actuelle par abonnement est de 10 services Device Provisioning par abonnement.

[!INCLUDE [azure-iotdps-limits](../../includes/iot-dps-limits.md)]

Pour plus d’informations sur les limites de quota :
* [Limites du service d’abonnement Azure](../azure-resource-manager/management/azure-subscription-service-limits.md)

## <a name="related-azure-components"></a>Composants Azure connexes
DPS automatise le provisionnement d’appareils avec Azure IoT Hub. Découvrez plus d’informations sur [IoT Hub](../iot-hub/index.yml).

## <a name="next-steps"></a>Étapes suivantes
Vous avez maintenant une vue d’ensemble de l’approvisionnement des appareils IoT dans Azure. L’étape suivante consiste à tester un scénario IoT de bout en bout.

[Configurer le service IoT Hub Device Provisioning avec le portail Azure](quick-setup-auto-provision.md)

[Créer et provisionner un appareil simulé](quick-create-simulated-device.md)

[Configurer l’appareil pour l’approvisionnement](tutorial-set-up-device.md)