---
title: Tutoriel - Configurer le cloud pour le service Azure IoT Hub Device Provisioning dans le portail
description: Ce tutoriel montre comment configurer les ressources cloud pour le provisionnement d’appareils dans le [portail Azure](https://portal.azure.com) à l’aide du service IoT Hub Device Provisioning
author: wesmc7777
ms.author: wesmc
ms.date: 11/12/2019
ms.topic: tutorial
ms.service: iot-dps
services: iot-dps
ms.custom: mvc
ms.openlocfilehash: f45c3def84c548ba12221efa59e9ebbd4699df71
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91316067"
---
# <a name="tutorial-configure-cloud-resources-for-device-provisioning-with-the-iot-hub-device-provisioning-service"></a>Tutoriel : Configurer les ressources de cloud pour l’approvisionnement d’appareils avec le service IoT Hub Device Provisioning

Ce didacticiel montre comment configurer le cloud pour l’approvisionnement d’appareils automatique à l’aide du service IoT Hub Device Provisioning. Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Utiliser le portail Azure pour créer un service IoT Hub Device Provisioning et obtenir l’étendue de l’ID
> * Créer un hub IoT
> * Lier le hub IoT au service Device Provisioning
> * Définir la stratégie d’allocation sur le service Device Provisioning

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Connectez-vous au [portail Azure](https://portal.azure.com/).

## <a name="create-a-device-provisioning-service-instance-and-get-the-id-scope"></a>Créer une instance de service Device Provisioning et obtenir l’étendue de l’ID

Suivez ces étapes pour créer une instance du service Device Provisioning.

1. En haut à gauche du portail Azure, cliquez sur **Créer une ressource**.

2. Dans la zone de recherche, tapez **device provisioning**. 

3. Cliquez sur **Service IoT Hub Device Provisioning**.

4. Remplissez le formulaire **Service IoT Hub Device Provisioning** avec les informations suivantes :
    
   | Paramètre       | Valeur suggérée | Description | 
   | ------------ | ------------------ | ------------------------------------------------- | 
   | **Nom** | Tout nom unique | -- | 
   | **Abonnement** | Votre abonnement  | Pour plus d’informations sur vos abonnements, consultez [Abonnements](https://account.windowsazure.com/Subscriptions). |
   | **Groupe de ressources** | myResourceGroup | Pour les noms de groupe de ressources valides, consultez [Naming conventions](/azure/architecture/best-practices/resource-naming) (Conventions d’affectation de nom). |
   | **Lieu** | Emplacement valide | Pour plus d’informations sur les régions, consultez [Régions Azure](https://azure.microsoft.com/regions/). |   

   ![Entrer les informations de base sur votre service Device Provisioning dans le portail](./media/tutorial-set-up-cloud/create-iot-dps-portal.png)

5. Cliquez sur **Créer**. Après quelques instants, l’instance de service Device Provisioning est créée et la page **Vue d’ensemble** s’affiche.

6. Sur la page **Vue d’ensemble** de la nouvelle instance de service, copiez la valeur de l’**étendue de l’ID** pour une utilisation ultérieure. Cette valeur permet d’identifier les ID d’enregistrement et de garantir leur unicité.

7. Copiez également la valeur de **point de terminaison de service** pour une utilisation ultérieure. 

## <a name="create-an-iot-hub"></a>Créer un hub IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

### <a name="retrieve-connection-string-for-iot-hub"></a>Récupérer la chaîne de connexion pour le hub IoT

[!INCLUDE [iot-hub-include-find-connection-string](../../includes/iot-hub-include-find-connection-string.md)]

Votre IoT Hub est maintenant créé et vous connaissez le nom d’hôte et la chaîne de connexion à IoT Hub dont vous avez besoin pour terminer ce qu’il reste du didacticiel.

## <a name="link-the-device-provisioning-service-to-an-iot-hub"></a>Lier le service Device Provisioning à un hub IoT

L’étape suivante consiste à lier le service Device Provisioning et le hub IoT afin que le service IoT Hub Device Provisioning puisse inscrire des appareils sur ce hub. Le service peut uniquement approvisionner des appareils sur des hubs IoT qui ont été liés au service Device Provisioning. Effectuez les opérations suivantes.

1. Dans la page **Toutes les ressources**, cliquez sur l’instance du service Device Provisioning que vous avez créée.

2. Dans la page Service Device Provisioning, cliquez sur **Hubs IoT liés**.

3. Cliquez sur **Add**.

4. Dans la page **Ajouter un lien au hub IoT**, fournissez les informations suivantes, puis cliquez sur **Enregistrer** :

    * **Abonnement :** Vérifiez que l’abonnement qui contient l’hub IoT est sélectionné. Vous pouvez établir un lien vers un hub IoT qui se trouve dans un autre abonnement.

    * **Hub IoT** : choisissez le nom du hub IoT que vous souhaitez lier à cette instance du service Device Provisioning.

    * **Stratégie d’accès :** sélectionnez **iothubowner** comme informations d’identification permettant d’établir le lien avec le hub IoT.

   ![Lier le nom du hub à lier au service Device Provisionning dans le portail](./media/tutorial-set-up-cloud/link-iot-hub-to-dps-portal.png)

## <a name="set-the-allocation-policy-on-the-device-provisioning-service"></a>Définir la stratégie d’allocation sur le service Device Provisioning

La stratégie d’allocation est un paramètre du service IoT Hub Device Provisioning qui détermine la façon dont les appareils sont affectés à un hub IoT. Trois stratégies d’allocation sont prises en charge : 

1. **Latence la plus faible** : les appareils sont provisionnés dans le hub IoT dont la latence est la plus faible.

2. **Distribution uniformément pondérée** (par défaut) : les hubs IoT liés ont tous la même probabilité d’être provisionnés. Il s’agit du paramètre par défaut. Si vous approvisionnez des appareils sur un seul hub IoT, vous pouvez conserver ce paramètre. 

3. **Configuration statique par le biais de la liste d’inscriptions** : le hub IoT spécifié dans la liste d’inscriptions a priorité sur la stratégie d’allocation au niveau du service Device Provisioning.

Pour définir la stratégie d’allocation, dans la page Service Device Provisioning, cliquez sur **Gérer la stratégie d’allocation**. Assurez-vous que la stratégie d’allocation est définie sur **Distribution uniformément pondérée** (valeur par défaut). Si vous apportez des modifications, cliquez sur **Enregistrer** quand vous avez terminé.

![Gérer la stratégie d’allocation](./media/tutorial-set-up-cloud/iot-dps-manage-allocation.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Les autres didacticiels de cette collection reposent sur ce didacticiel. Si vous souhaitez continuer à utiliser d’autres démarrages rapides ou les didacticiels, ne nettoyez pas les ressources créées dans ce didacticiel. Sinon, effectuez les opérations suivantes pour supprimer toutes les ressources créées au cours de ce didacticiel dans le portail Azure.

1. Dans le menu de gauche du portail Azure, cliquez sur **Toutes les ressources**, puis sélectionnez votre instance du service IoT Hub Device Provisioning. Dans la partie supérieure de la page **Toutes les ressources**, cliquez sur **Supprimer**.  

2. À partir du menu de gauche, dans le portail Azure, cliquez sur **Toutes les ressources**, puis sélectionnez votre IoT Hub. Dans la partie supérieure de la page **Toutes les ressources**, cliquez sur **Supprimer**.
 
## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Utiliser le portail Azure pour créer un service IoT Hub Device Provisioning et obtenir l’étendue de l’ID
> * Créer un hub IoT
> * Lier le hub IoT au service Device Provisioning
> * Définir la stratégie d’allocation sur le service Device Provisioning

Passez au didacticiel suivant pour apprendre à configurer votre appareil pour l’approvisionnement.

> [!div class="nextstepaction"]
> [Configurer l’appareil pour l’approvisionnement](tutorial-set-up-device.md)
