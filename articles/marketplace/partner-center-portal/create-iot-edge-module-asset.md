---
title: Préparer vos ressources techniques de module IoT Edge - Place de marché Azure
description: Découvrez les exigences techniques et de configuration que vos ressources techniques de module IoT (Internet of Things) Edge doivent respecter pour que vous puissiez les publier sur la Place de marché Azure.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: keferna
ms.author: keferna
ms.date: 04/03/2020
ms.openlocfilehash: aca27b89a3b92b410fa560c8b4bd7eb3d4e0a935
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93346772"
---
# <a name="prepare-your-iot-edge-module-technical-assets"></a>Préparer vos ressources techniques de module IoT Edge

Cet article décrit les exigences que vos ressources techniques de module IoT Edge doivent remplir avant d’être publiées sur la Place de marché Microsoft Azure.

## <a name="get-started"></a>Bien démarrer

Un module IoT Edge est un conteneur compatible avec Docker qui s’exécute sur un appareil IoT Edge.

- Pour en savoir plus sur les modules IoT Edge, consultez [Présentation des modules Azure IoT Edge](../../iot-edge/iot-edge-modules.md).
- Pour commencer le développement de votre module IoT Edge, consultez [Développer vos propres modules IoT Edge](../../iot-edge/module-development.md).

## <a name="technical-requirements"></a>Exigences techniques

Votre module IoT Edge doit remplir les exigences techniques suivantes pour être certifié et publié sur la Place de marché Azure.

### <a name="platform-support"></a>Plateforme prise en charge

Votre module IoT Edge doit prendre en charge une des options de plateforme suivantes :

#### <a name="tier-1-platforms-supported-by-iot-edge"></a>Plateformes de niveau 1 prises en charge par IoT Edge

Votre module doit prendre en charge toutes les plateformes de niveau 1 prises en charge par IoT Edge (tel qu’enregistré dans [Prise en charge par Azure IoT Edge](../../iot-edge/support.md)). Nous recommandons cette option, car elle fournit une meilleure expérience client. Les modules répondant à ce critère sont présentés. Un module utilisant cette option de plateforme doit :

- Fournir une étiquette « latest » et une étiquette de version (par exemple, 1.0.1), qui sont des étiquettes manifeste générées avec [Manifest-tool de GitHub](https://github.com/estesp/manifest-tool).

- Utilisez l’onglet Description de l’offre dans l’ [Espace partenaires](https://partner.microsoft.com/dashboard/commercial-marketplace) pour ajouter un lien sous la section **Liens utiles** vers le [catalogue d’appareils Azure IoT Edge Certified](https://catalog.azureiotsolutions.com/alldevices?filters={%2218%22:[%221%22]}/).

#### <a name="a-subset-of-tier-1-platforms-supported-by-iot-edge"></a>Un sous-ensemble de plateformes de niveau 1 prises en charge par IoT Edge

Votre module doit prendre en charge un sous-ensemble (au moins un) de plateformes de niveau 1 prises en charge par IoT Edge (tel qu’enregistré dans [Prise en charge par Azure IoT Edge](../../iot-edge/support.md)). Un module utilisant cette option de plateforme doit :

- Fournir une étiquette « latest » et une étiquette de version (par exemple, 1.0.1), qui sont des étiquettes manifeste générées avec [manifest-tool](https://github.com/estesp/manifest-tool) de GitHub si plusieurs plateformes sont prises en charge. Les balises manifeste ne sont facultatives que si une seule plateforme est prise en charge.
- Utilisez l’onglet Description de l’offre dans l’ [Espace partenaires](https://partner.microsoft.com/dashboard/commercial-marketplace) pour ajouter un lien sous la section **Liens utiles** vers au moins un appareil IoT Edge à partir du [catalogue d’appareils Azure IoT Edge Certified](https://catalog.azureiotsolutions.com/).

:::image type="content" source="media/iot-edge-module-technical-assets-offer-listing.png" alt-text="Image de la section Description de l’offre dans l’Espace partenaires":::

### <a name="device-dimensions"></a>Dimensions de l’appareil

Les dimensions de module IoT Edge (telles que processeur, RAM, stockage et GPU) sur les appareils IoT Edge ciblés doivent remplir les exigences suivantes :

- Le module doit fonctionner avec au moins un appareil IoT Edge du [catalogue d’appareils Azure IoT Edge Certified](https://catalog.azureiotsolutions.com/).

- La configuration matérielle minimale doit correspondre au dernier paragraphe de la description de l’offre (sous l’onglet Description de l’offre dans l’[Espace partenaires](https://partner.microsoft.com/dashboard/commercial-marketplace)). Si vous le souhaitez, vous pouvez également répertorier la configuration matérielle recommandée si elles diffèrent considérablement. Par exemple, ajoutez la section suivante à la fin de la description de votre offre :

Copiez ce texte HTML ou utilisez les fonctions de texte enrichi correspondantes dans la fenêtre d’édition.

```html
<p><u>Minimum hardware requirements:</u> Linux x64 and arm32 OS, 1GB of RAM, 500 Mb of storage</p>
```

### <a name="configuration"></a>Configuration

Votre module doit inclure des paramètres de configuration par défaut pour rendre le déploiement sur un appareil IoT Edge aussi simple que possible. Ces informations peuvent être fournies dans la page **Configuration technique** du plan dans l’ [Espace partenaires](https://partner.microsoft.com/dashboard/commercial-marketplace). Le conteneur peut également inclure le SDK IoT Edge pour permettre la communication avec le hub de périphérie et IoT Hub.

#### <a name="default-configuration"></a>Configuration par défaut

Les modules IoT Edge doivent pouvoir démarrer avec les paramètres par défaut fournis dans la page **Configuration technique** du plan dans l’ [Espace partenaires](https://partner.microsoft.com/dashboard/commercial-marketplace). Les paramètres par défaut suivants sont disponibles :

- **Itinéraires** par défaut
- **Propriétés souhaitées du jumeau de module** par défaut
- **Variables d’environnement** par défaut
- **Options de création de conteneur** par défaut

Dans un scénario où un paramètre requis pour une valeur par défaut n’est pas judicieux (par exemple, l’adresse IP du serveur d’un client), ajoutez un paramètre comme valeur par défaut. Cette valeur est en majuscules et est placée entre crochets. Pour cet exemple, vous définissez la variable d’environnement par défaut suivante :

```
ServerIPAddress = <MY_SERVER_IP_ADDRESS>
```

#### <a name="configuration-documentation"></a>Documentation de configuration

Tous les paramètres de configuration d’un module IoT Edge doivent être clairement documentés. Par exemple, vous devez documenter l’utilisation de ses routes, propriétés souhaitées du jumeau, variables d’environnement, options de création, etc. Vous devez fournir un lien vers votre documentation ou intégrer celle-ci à la description de votre offre ou plan. Vous pouvez fournir ces informations dans les pages **Description de l’offre** et **Liste des plans** de l’ [Espace partenaires](https://partner.microsoft.com/dashboard/commercial-marketplace).

#### <a name="tags-and-versioning"></a>Balises et contrôle de version

Les clients doivent être en mesure de déployer facilement un module et d’obtenir automatiquement les mises à jour de la Place de marché (dans un scénario pour développeur). Ils doivent également être en mesure d’utiliser et de figer une version exacte qu’ils ont testée (dans un scénario de production).

Pour répondre à ces attentes des clients et être publiés dans la Place de marché, les modules IoT Edge doivent remplir les exigences suivantes :

- Inclure une étiquette « latest » de manifeste qui pointe vers la dernière version sur toutes les plateformes prises en charge.
- Créer les étiquettes de version au format X.Y.Z, où X, Y et Z sont des entiers.
- Inclure une étiquette « version », telle que 1.0.1, qui pointe vers une version spécifique sur toutes les plateformes prises en charge.
- Ne mettez pas à jour les étiquettes « version », telles que 1.0.1, car elles ne doivent pas être changées.

> [!NOTE]
> Si vous le souhaitez, le contrôle de version peut inclure des étiquettes « déploiement de version », telles que 2.0 et 1.0. Cela permet la prise en charge de la gestion de plusieurs versions principales en parallèle.

### <a name="telemetry"></a>Télémétrie

Les modules utilisant le SDK de module IoT doivent définir l’identificateur sur PublisherId.OfferId.SkuId à des fins de télémétrie. Un identificateur unique aide la Place de marché Microsoft Azure à identifier le nombre d’instances de module en cours d’exécution.

Utilisez l’une des méthodes suivantes des SDK de module IoT pour définir ProductInfo sur cet identificateur :

- [C#](/dotnet/api/microsoft.azure.devices.client.deviceclient.productinfo#Microsoft_Azure_Devices_Client_DeviceClient_ProductInfo)
- [C](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/Iothub_sdk_options.md)
- [Python](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/Iothub_sdk_options.md)
- [Java](/java/api/com.microsoft.azure.sdk.iot.device.productinfo)

Pour les modules qui n’utilisent pas le SDK de module IoT, des insights moins précis sont disponibles sur l’Espace partenaires, comme le nombre de téléchargements.

### <a name="security"></a>Sécurité

Les modules IoT Edge doivent éviter les [modules privilégiés](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities). Demandez à la place l’accès à l’hôte avec le moins de privilèges possible.

### <a name="module-iot-sdk"></a>Kit de développement logiciel (SDK) de module IoT

Si la certification ne passe pas nécessairement par l’inclusion du Kit de développement logiciel (SDK) de module IoT, celle-ci peut offrir une meilleure expérience utilisateur avec, par exemple, la prise en charge du routage, l’envoi de message au cloud.

Le SDK de module IoT est nécessaire pour obtenir des données de télémétrie sur le nombre d’instances de module en cours d’exécution.

## <a name="recertification-process"></a>Processus de renouvellement de certification

Les partenaires reçoivent une notification à chaque changement cassant qui affecte leurs modules, tel que :

- Matrice de prise en charge du système d’exploitation/arch de niveau 1 prise en charge par IoT Edge
- Kit de développement logiciel (SDK) de module IoT
- Runtime IoT Edge
- Directives de certification de module IoT Edge

Les partenaires doivent mettre à jour et recertifier leurs offres en les republiant sur l’[Espace partenaires](https://partner.microsoft.com/dashboard/commercial-marketplace).

Votre offre est également recertifiée si vous la mettez à jour, par exemple, en ajoutant de nouvelles étiquettes d’image.

## <a name="host-module-in-azure-container-registry"></a>Héberger un module dans un registre de containeurs Azure

Pour charger votre module IoT Edge sur la Place de marché Azure, vous devez tout d’abord l’héberger dans un [registre de containeurs Azure](https://azure.microsoft.com/services/container-registry/). Le module doit inclure toutes les étiquettes que vous souhaitez publier, y compris les étiquettes d’image qui sont référencées par une étiquette de manifeste. Pour plus d’informations, consultez le tutoriel [Créer un registre de conteneurs Azure et envoyer une image conteneur](../../container-instances/container-instances-tutorial-prepare-acr.md).

## <a name="next-steps"></a>Étapes suivantes

- [Créer une offre de module IoT Edge](azure-iot-edge-module-creation.md)