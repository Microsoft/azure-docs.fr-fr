---
title: Chiffrement de données du Service Azure Device Provisioning au repos à l’aide de clés gérées par le client | Microsoft Docs
description: Chiffrement de données au repos avec des clés gérées par le client pour le Service Device Provisioning
author: chrissie926
manager: nberdy
ms.service: iot-dps
services: iot-dps
ms.topic: conceptual
ms.date: 02/24/2020
ms.author: menchi
ms.openlocfilehash: d22a01bab81fc330484e7715a65c89a1cfd7802c
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967174"
---
# <a name="encryption-of-data-at-rest-with-customer-managed-keys-for-device-provisioning-service"></a>Chiffrement de données au repos avec des clés gérées par le client pour le Service Device Provisioning

## <a name="overview"></a>Vue d’ensemble

Le Service Device Provisioning (DPS) prend en charge le chiffrement des données au repos avec des clés gérées par le client (CMK). Cette méthode est également appelée BYOK (Bring Your Own Key, ou apportez votre propre clé). DPS fournit un chiffrement des données au repos et en transit, à mesure qu’elles sont écrites dans nos centres de données, et les déchiffre quand vous y accédez. Par défaut, il utilise des clés gérées par Microsoft pour chiffrer les données au repos. Avec CMK, vous pouvez obtenir une couche de chiffrement en plus du chiffrement de plateforme par défaut, en choisissant de chiffrer les données au repos avec une clé de chiffrement à clé gérée via votre [Azure Key Vault](https://azure.microsoft.com/services/key-vault/). Cela vous donne la flexibilité nécessaire pour la création, la rotation, la désactivation et la révocation des clés. Si CMK est configuré pour votre DPS, cela implique que le double chiffrement est activé avec deux couches de protection protégeant activement vos données. 

Cette fonctionnalité requiert la création d’un service DPS. Pour essayer cette fonctionnalité, contactez-nous par le biais du [support technique Microsoft](https://azure.microsoft.com/support/create-ticket/). Partagez le nom de votre société et votre ID d’abonnement pour contacter le support Microsoft.


## <a name="next-steps"></a>Étapes suivantes

* [En savoir plus sur le service Device Provisioning](./index.yml)

* [En savoir plus sur Azure Key Vault](../key-vault/general/overview.md)