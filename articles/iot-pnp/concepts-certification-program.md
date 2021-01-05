---
title: Programme Azure Certified Device | Microsoft Docs
description: Découvrez ce qu’est le programme Azure Certified Device.
author: koichih
ms.author: koichih
ms.date: 09/25/2020
ms.topic: conceptual
ms.custom: mvc
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 1fcfc7a9e632e5db1fb809dba7a938c8641c9ddc
ms.sourcegitcommit: 2e72661f4853cd42bb4f0b2ded4271b22dc10a52
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92048098"
---
# <a name="what-is-the-azure-certified-device-program"></a>En quoi consiste le programme Azure Certified Device ?

Le programme Azure Certified Device garantit que les solutions des clients fonctionnent parfaitement avec vos services Azure. Le programme utilise des outils, des services et une place de marché pour partager les connaissances et les bonnes pratiques du secteur avec une communauté de fabricants d’appareils et de créateurs de solutions.

Ce programme est conçu pour :

- **Donner confiance aux clients :** les clients peuvent acheter en toute confiance des appareils certifiés par Microsoft pour une utilisation avec les services Azure.

- **Aider les clients à trouver les appareils adaptés à leur solution :** les fabricants d’appareils peuvent publier les fonctionnalités uniques de leurs appareils dans le cadre du processus de certification. Les clients trouvent ainsi facilement les produits qui répondent à leurs besoins.

- **Promouvoir les appareils certifiés :** les fabricants d’appareils gagnent en visibilité, ont plus de contact avec les clients et tirent davantage profit de la marque Azure Certified Device.

Cet article explique comment :

- Intégrer votre entreprise au programme Azure Certified Device.
- Identifier la certification applicable à votre appareil.
- Trouver les conditions du programme et d’autres ressources pour vous aider à démarrer les tests.
- Utiliser le portail Azure Certified Device pour valider votre appareil.

## <a name="onboarding"></a>Intégration

Pour certifier vos appareils sur le [portail Azure Certified Device](https://aka.ms/acdp), vous devez effectuer les étapes suivantes :

1. Vérifier que votre entreprise dispose d’un compte Azure Active Directory avec un locataire professionnel ou scolaire.
2. Rejoindre le réseau [Microsoft Partner Network (MPN)](https://partner.microsoft.com/) avec votre compte.
3. Vous connecter au [portail Azure Certified Device](https://aka.ms/acdp) après avoir rejoint le réseau MPN.
4. Lire et signer le [contrat du programme](https://aka.ms/acdagreement) dans la page du profil de l’entreprise.

### <a name="company-profile"></a>Profil de l’entreprise

Pour gérer le profil de votre entreprise dans le portail Azure Certified Device, sélectionnez **Profil de l’entreprise**. Le profil d’entreprise comprend l’URL, l’adresse e-mail et le logo de l’entreprise. Vous devez accepter le contrat du programme sur cette page avant de commencer toute opération de certification.

La page de description de l’appareil dans le catalogue Azure Certified Device reprend les informations du profil de l’entreprise.

## <a name="choose-the-certification"></a>Choix de la certification

Il existe trois certifications différentes, chacune apportant une valeur différente aux clients. Selon le type d’appareil que vous fabriquez et votre public cible, vous choisissez la ou les certifications les plus appropriées :

### <a name="azure-certified-device"></a>Azure Certified Device

La _certification Azure Certified Device_ valide la possibilité de connecter un appareil à Azure IoT Hub et de le provisionner par le biais du service Device Provisioning (DPS). Cette certification reflète le niveau de fonctionnalité et d’interopérabilité de base qu’un appareil doit présenter pour prétendre à des certifications plus avancées.

- Pour en savoir plus, consultez les [conditions de certification](https://aka.ms/acdrequirements).
- Pour en savoir plus sur l’utilisation du service Device Provisioning pour connecter votre appareil à Azure IoT Hub, consultez la [vue d’ensemble du provisionnement d’appareils](../iot-dps/about-iot-dps.md).

### <a name="iot-plug-and-play"></a>IoT Plug-and-Play

La _certification IoT Plug-and-Play_ est une certification plus avancée que la certification Azure Certified Device ; elle simplifie le processus de fabrication d’appareils sans code d’appareil personnalisé. IoT Plug-and-Play permet aux fournisseurs de matériel partenaires de fabriquer des appareils qui s’intègrent facilement aux solutions cloud basées sur des solutions IoT Central Azure et de tiers.

- Pour en savoir plus, consultez les [conditions de certification](https://aka.ms/acdiotpnprequirements).
- Pour en savoir plus sur la préparation d’un appareil en vue des tests IoT Plug-and-Play, consultez [Certifier des appareils IoT Plug-and-Play](howto-certify-device.md).

### <a name="edge-managed"></a>Edge Managed

La _certification Edge Managed_ est une certification plus avancée que la certification Azure Certified Device ; elle porte essentiellement sur les standards de gestion des appareils Azure IoT exécutant Windows, Linux ou RTOS. Actuellement, cette certification de programme est axée sur la compatibilité du runtime Edge pour le déploiement et la gestion des modules.

> [!TIP]
> Avant, ce programme s’appelait _programme de certification IoT Edge_.

- Pour en savoir plus, consultez les [conditions de certification](https://aka.ms/acdedgemanagedrequirements).
- Pour en savoir plus sur IoT Edge, consultez la [vue d’ensemble d’IoT Edge](../iot-edge/about-iot-edge.md).
- Pour en savoir plus sur les systèmes d’exploitation et les conteneurs pris en charge, consultez [Systèmes pris en charge par Azure IoT Edge](../iot-edge/support.md).

## <a name="use-the-azure-certified-device-portal"></a>Utiliser le portail Azure Certified Device

Cette section récapitule comment utiliser le [portail Azure Certified Device](https://certify.azure.com). Pour en savoir plus, consultez le [guide Bien démarrer avec le portail](https://aka.ms/acdhelp).

Pour certifier un appareil dans le programme Azure Certified Device, effectuez les quatre étapes suivantes :

1. Fournissez les détails de l’appareil
2. Testez l’appareil
3. Soumettez et effectuez la revue
4. Publiez sur le catalogue Azure Certified Device (facultatif)

### <a name="provide-device-details"></a>Fournir les détails de l’appareil

Pour chaque appareil que vous souhaitez certifier, renseignez les détails de l’appareil (les caractéristiques matérielles, les instructions d’installation et les supports marketing) dans les formulaires appropriés du portail de certification :

- **Informations sur l’appareil :** collecte des informations sur l’appareil, telles que son nom, sa description, ses caractéristiques matérielles et son système d’exploitation.
- **Guide de démarrage :** document PDF qu’un client peut utiliser pour démarrer rapidement avec votre produit. Des [exemples de modèles](https://aka.ms/GSTemplate) sont disponibles.
- **Détails marketing :** contient des données marketing pour votre appareil, comme une photo et des informations de distribution, que le client peut utiliser directement.
- **Autres certifications du secteur :** section facultative dans laquelle vous pouvez donner des informations sur les autres certifications de l’appareil.

### <a name="test-the-device"></a>Tester l’appareil

Cette phase interagit avec votre appareil et exécute une série de tests après que l’appareil se soit connecté à IoT Hub par le biais du service Device Provisioning. À la fin de l’opération, vous pouvez examiner les différents fichiers journaux qui contiennent les résultats des tests de votre appareil.

Le portail de certification fournit des instructions sur la façon de se connecter à l’instance IoT Hub utilisée pour les tests. Vous pouvez établir la connexion au service Device Provisioning au moyen de l’une des [méthodes d’attestation prises en charge](../iot-dps/concepts-service.md#attestation-mechanism).

L’équipe Azure Certified Device peut être amenée à contacter le fabricant de l’appareil pour demander une validation manuelle supplémentaire de l’appareil.

### <a name="submit-and-publish"></a>Soumettre et publier

La dernière étape requise consiste à soumettre le projet pour revue. Cette étape indique à un membre de l’équipe Azure Certified Device de vérifier que votre projet contient toutes les informations requises, notamment les détails de l’appareil, les données marketing et le guide de démarrage. Il est possible qu’un membre de l’équipe vous envoie des questions à l’adresse e-mail professionnelle précédemment fournie ou modifie les requêtes avant l’approbation.

Si votre appareil nécessite une validation manuelle supplémentaire dans le cadre de la certification, vous recevrez une notification au moment opportun.

Quand un appareil est certifié, vous pouvez choisir de publier les détails de votre produit dans le catalogue Azure Certified Device à l’aide de la fonctionnalité **Publier dans le catalogue**, qui est disponible dans la page de résumé du produit.

## <a name="if-you-have-questions"></a>Si vous avez des questions

Contactez l’équipe sur [iotcert@microsoft.com](mailto:iotcert@microsoft.com?subject=Azure%20Certified%20Device%20question) si vous avez des questions à poser sur les programmes de certification, le portail de certification ou le catalogue Azure Certified Device.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez une vue d’ensemble du programme Azure Certified Device, nous vous suggérons de découvrir comment [certifier des appareils IoT Plug-and-Play](howto-certify-device.md).