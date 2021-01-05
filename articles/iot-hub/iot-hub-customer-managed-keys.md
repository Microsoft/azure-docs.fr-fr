---
title: Chiffrement de données Azure IoT Hub au repos par le biais de clés gérées par le client | Microsoft Docs
description: Chiffrement de données au repos avec des clés gérées par le client pour IoT Hub
author: ash2017
manager: briz
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 06/17/2020
ms.author: asrastog
ms.openlocfilehash: 352da24b36124ff0446a81c1ecbc584da545bb16
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92142204"
---
# <a name="encryption-of-data-at-rest-with-customer-managed-keys-for-iot-hub"></a>Chiffrement de données au repos avec des clés gérées par le client pour IoT Hub

IoT Hub prend en charge le chiffrement des données au repos avec des clés gérées par le client (CMK). On appelle également ce comportement BYOK. Azure IoT Hub fournit un chiffrement des données au repos et en transit, à mesure qu’elles sont écrites dans nos centres de données, et les déchiffre quand vous y accédez. Par défaut, IoT Hub utilise des clés gérées par Microsoft pour chiffrer les données au repos. Avec CMK, vous pouvez obtenir une autre couche de chiffrement en plus du chiffrement par défaut, ainsi que choisir de chiffrer les données au repos avec une clé de chiffrement à clé gérée via votre [Azure Key Vault](https://azure.microsoft.com/services/key-vault/). Cela vous donne la flexibilité nécessaire pour la création, la rotation, la désactivation et la révocation des contrôles d’accès. Si le service BYOK est configuré pour votre IoT Hub, nous assurons également un double chiffrement qui ajoute une deuxième couche de protection, tout en vous permettant de contrôler la clé de chiffrement via votre Azure Key Vault.

Cette fonctionnalité requiert la création d’un nouveau IoT Hub (niveau de base ou standard). Pour essayer cette fonctionnalité, contactez-nous par le biais du [support technique Microsoft](https://azure.microsoft.com/support/create-ticket/). Partagez le nom de votre société et votre ID d’abonnement pour contacter le support Microsoft.


## <a name="next-steps"></a>Étapes suivantes

* [En savoir plus sur IoT Hub](./about-iot-hub.md)

* [En savoir plus sur Azure Key Vault](../key-vault/general/overview.md)