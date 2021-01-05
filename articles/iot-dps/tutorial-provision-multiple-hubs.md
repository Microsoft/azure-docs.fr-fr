---
title: Tutoriel - Provisionner des appareils sur des hubs à charge équilibrée à l’aide du service Azure IoT Hub Device Provisioning
description: Ce tutoriel montre comment le service Device Provisioning permet le provisionnement automatique d’appareils sur des hubs IoT à charge équilibrée dans le portail Azure
author: wesmc7777
ms.author: wesmc
ms.date: 11/12/2019
ms.topic: tutorial
ms.service: iot-dps
services: iot-dps
ms.custom: mvc
ms.openlocfilehash: 6cca34774038156cfb01e872e751338c1c96efaa
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91315998"
---
# <a name="tutorial-provision-devices-across-load-balanced-iot-hubs"></a>Tutoriel : Approvisionner des appareils sur des hubs IoT à charge équilibrée

Ce didacticiel montre comment approvisionner des appareils sur plusieurs hubs IoT à charge équilibrée en utilisant le service Device Provisioning. Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Utiliser le portail Azure pour approvisionner un deuxième appareil sur un deuxième hub IoT 
> * Ajouter une entrée de liste d’inscriptions au deuxième appareil
> * Définir la stratégie d’allocation du service Device Provisioning sur une **distribution égale**
> * Lier le nouveau hub IoT au service Device Provisioning

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Ce didacticiel s’appuie sur le précédent didacticiel [Approvisionner un appareil sur un hub](tutorial-provision-device-to-hub.md).

## <a name="use-the-azure-portal-to-provision-a-second-device-to-a-second-iot-hub"></a>Utiliser le portail Azure pour approvisionner un deuxième appareil sur un deuxième hub IoT

Suivez les étapes du didacticiel [Approvisionner un appareil sur un hub](tutorial-provision-device-to-hub.md) afin d’approvisionner un deuxième appareil sur un autre hub IoT.

## <a name="add-an-enrollment-list-entry-to-the-second-device"></a>Ajouter une entrée de liste d’inscriptions au deuxième appareil

La liste d’inscriptions indique au service Device Provisioning la méthode d’attestation (méthode de confirmation d’une identité d’appareil) qu’il utilise avec l’appareil. L’étape suivante consiste à ajouter une entrée de liste d’inscriptions pour le deuxième appareil. 

1. Dans la page de votre service Device Provisioning, cliquez sur **Gérer les inscriptions**. La page **Ajouter une entrée de liste d’inscriptions** s’affiche. 
2. En haut de la page, cliquez sur **Ajouter**.
2. Renseignez les champs et cliquez sur **Enregistrer**.

## <a name="set-the-device-provisioning-service-allocation-policy"></a>Définir la stratégie d’allocation du service Device Provisioning

La stratégie d’allocation est un paramètre du service Device Provisioning qui détermine la façon dont les appareils sont assignés à un hub IoT. Trois stratégies d’allocation sont prises en charge : 

1. **Latence la plus faible** : les appareils sont provisionnés dans le hub IoT dont la latence est la plus faible.
2. **Distribution uniformément pondérée** (par défaut) : les hubs IoT liés ont tous la même probabilité d’être provisionnés. Il s'agit du paramètre par défaut. Si vous approvisionnez des appareils sur un seul hub IoT, vous pouvez conserver ce paramètre. 
3. **Configuration statique par le biais de la liste d’inscriptions** : le hub IoT spécifié dans la liste d’inscriptions a priorité sur la stratégie d’allocation au niveau du service Device Provisioning.

Effectuez les étapes suivantes pour définir la stratégie d’allocation :

1. Pour définir la stratégie d’allocation, dans la page Service Device Provisioning, cliquez sur **Gérer la stratégie d’allocation**.
2. Définissez la stratégie d’allocation sur **Distribution uniformément pondérée**.
3. Cliquez sur **Enregistrer**.

## <a name="link-the-new-iot-hub-to-the-device-provisioning-service"></a>Lier le nouveau hub IoT au service Device Provisioning

Liez le service Device Provisioning et le hub IoT pour que le service Device Provisioning puisse inscrire des appareils dans ce hub.

1. Dans la page **Toutes les ressources**, cliquez sur le service Device Provisioning que vous avez créé.
2. Dans la page Service Device Provisioning, cliquez sur **Hubs IoT liés**.
3. Cliquez sur **Add**.
4. Dans la page **Ajouter un lien au hub IoT**, utilisez les cases d’option pour spécifier si le hub IoT lié se trouve dans l’abonnement actuel ou dans un autre abonnement. Ensuite, choisissez le nom du hub IoT dans la zone **Hub IoT**.
5. Cliquez sur **Enregistrer**.

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Utiliser le portail Azure pour approvisionner un deuxième appareil sur un deuxième hub IoT 
> * Ajouter une entrée de liste d’inscriptions au deuxième appareil
> * Définir la stratégie d’allocation du service Device Provisioning sur une **distribution égale**
> * Lier le nouveau hub IoT au service Device Provisioning

## <a name="next-steps"></a>Étapes suivantes

<!-- Advance to the next tutorial to learn how to 
 Replace this .md
> [!div class="nextstepaction"]
> [Bind an existing custom SSL certificate to Azure Web Apps]()
-->
