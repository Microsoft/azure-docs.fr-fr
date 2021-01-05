---
author: rothja
ms.service: iot-dps
ms.topic: include
ms.date: 11/09/2018
ms.author: jroth
ms.openlocfilehash: bbe928084a758335e5fa3b0531726c77a6d599b8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85839062"
---
Le tableau suivant répertorie les limites qui s’appliquent aux ressources du 	Service Azure IoT Hub Device Provisioning.

| Ressource | Limite |
| --- | --- |
| Nombre maximal de services d’approvisionnement d’appareil par abonnement Azure | 10 |
| Nombre maximal d’inscriptions | 1 000 000 |
| Nombre maximal d’inscriptions | 1 000 000 |
| Nombre maximal de groupes d’inscription | 100 |
| Nombre maximal d’autorités de certification | 25 |
| Nombre maximal de hubs IoT liés | 50 |
| Taille maximale du message | 96 Ko|

> [!NOTE]
> Pour augmenter le nombre d’abonnements et d’inscriptions sur votre service de provisionnement, contactez le [Support Microsoft](https://azure.microsoft.com/support/options/).

> [!NOTE]
> L’augmentation du nombre maximal d’autorités de certification n’est pas prise en charge.

Le Service Device Provisioning limite les demandes lorsque les quotas suivants sont dépassés.

| Limitation | Valeur par unité |
| --- | --- |
| Opérations | 200/min/service |
| Inscriptions d’appareil | 200/min/service |
| Opération d’interrogation des appareils | 5/10 s/appareil |
