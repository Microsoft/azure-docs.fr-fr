---
title: Foire aux questions Azure IoT Central | Microsoft Docs
description: Foire aux questions (FAQ) Azure IoT Central et réponses
author: dominicbetts
ms.author: dobett
ms.date: 12/20/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.openlocfilehash: acabaf843f8acfe7bc0b5e9456dee09bde74bef7
ms.sourcegitcommit: ab829133ee7f024f9364cd731e9b14edbe96b496
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/28/2020
ms.locfileid: "97796038"
---
# <a name="frequently-asked-questions-for-iot-central"></a>Foire aux questions pour IoT Central

**Comment vérifier les problèmes d’informations d’identification si un appareil ne se connecte pas à mon application IoT Central ?**

L’article [Déterminer la raison pour laquelle les données de vos appareils ne s’affichent pas dans Azure IoT Central](troubleshoot-connection.md) comprend des étapes pour diagnostiquer les problèmes de connectivité des appareils.

**Comment soumettre un ticket auprès du support technique ?**

Si vous avez besoin d’aide, vous pouvez soumettre un [ticket de support Azure](https://portal.azure.com/#create/Microsoft.Support).

Pour plus d’informations, notamment sur les autres options de support, consultez [Options d’aide et de support Azure IoT](../../iot-fundamentals/iot-support-help.md).

**Comment débloquer un appareil ?**

Quand un appareil est bloqué, il ne peut pas envoyer de données à votre application IoT Central. L’état des appareils bloqués est **Bloqué** sur la page **Appareils** de votre application. Un opérateur doit débloquer l’appareil avant que ce dernier puisse recommencer à envoyer des données :

:::image type="content" source="media/howto-faq/blocked.png" alt-text="Capture d’écran montrant un appareil bloqué":::

Quand un opérateur débloque un appareil, l’état revient à sa valeur précédente, à savoir **Inscrit** ou **Approvisionné**.

**Comment approuver un appareil ?**

Si l’état de l’appareil est **En attente d’approbation** sur la page **Appareils**, cela signifie que l’option **Approbation automatique** est désactivée :

:::image type="content" source="media/howto-faq/auto-approve.png" alt-text="Capture d’écran montrant une application avec l’option d’approbation automatique désactivée.":::

Un opérateur doit approuver explicitement un appareil avant qu’il ne commence à envoyer des données. Les appareils non enregistrés manuellement sur la page **Appareils**, mais connectés avec des informations d'identification valides afficheront l’état **En attente d'approbation**. Les opérateurs peuvent approuver ces appareils sur la page **Appareils** en utilisant le bouton **Approuver** :

:::image type="content" source="media/howto-faq/approve-device.png" alt-text="Capture d’écran montrant comment approuver un appareil":::

**Comment associer un appareil à un modèle d’appareil ?**

Si l’état de l’appareil est **Non associé**, cela signifie que les appareils se connectant à IoT Central n’ont pas de modèle d’appareil associé. C’est généralement le cas dans les scénarios suivants :

- Un ensemble d’appareils est ajouté en utilisant **Importer** sur la page **Appareils**, sans spécifier le modèle d’appareil.
- Un appareil a été inscrit manuellement sur la page **Appareils** sans spécifier le modèle d’appareil. L’appareil est alors connecté avec des informations d’identification valides.  

L’opérateur peut associer un appareil à un modèle d’appareil sur la page **Appareils** en utilisant le bouton **Migrer**. Pour plus d’informations, consultez [Gérer les appareils dans votre application Azure IoT Central > Migration d’appareils vers un modèle](howto-manage-devices.md).

**Où puis-je en savoir plus sur IoT Hub ?**

Azure IoT Central utilise Azure IoT Hub comme passerelle cloud pour la connectivité des appareils. IoT Hub permet :

- L’ingestion de données à grande échelle dans le cloud.
- La gestion des appareils.
- La connectivité sécurisée des appareils.

Pour plus d’informations sur IoT Hub, consultez [Azure IoT Hub](../../iot-hub/index.yml).

**Où puis-je en savoir plus sur le service Device Provisioning (DPS) ?**

Azure IoT Central utilise DPS pour permettre aux appareils de se connecter à votre application. Pour en savoir plus sur le rôle de DPS dans la connexion des appareils à IoT Central, consultez [Se connecter à Azure IoT Central](concepts-get-connected.md). Pour en savoir plus sur DPS, consultez [Approvisionner des appareils avec le service Azure IoT Hub Device Provisioning](../../iot-dps/about-iot-dps.md).