---
title: Fichier include
description: Fichier include
services: event-grid
author: spelluru
ms.service: event-grid
ms.topic: include
ms.date: 10/10/2019
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 0fc6a62a94f31617209bcc60bfaa95bc8927551a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86050464"
---
## <a name="deploy-event-grid-iot-edge-module"></a>Déployer le module Event Grid sur IoT Edge

Il existe plusieurs moyens de déployer des modules sur un appareil IoT Edge ; tous fonctionnent pour Azure Event Grid sur IoT Edge. Cet article décrit les étapes à suivre pour déployer Event Grid sur IoT Edge sur le Portail Azure.

>[!NOTE]
> Dans ce tutoriel, vous allez déployer le module Event Grid sans persistance. Cela signifie que toutes les rubriques et tous les abonnements créés dans ce tutoriel seront supprimés lors du redéploiement du module. Pour savoir comment configurer la persistance, voir les articles suivants : [Conserver l’état dans Linux](../articles/event-grid/edge/persist-state-linux.md) ou [Conserver l’état dans Windows](../articles/event-grid/edge/persist-state-windows.md). Pour les charges de travail de production, nous vous recommandons d’installer le module Event Grid avec la persistance.

>[!IMPORTANT]
> Dans ce tutoriel, le module Event Grid sera déployé sans authentification client et autorisera les abonnés HTTP. Dans le cas de charges de travail de production, nous vous recommandons d’autoriser uniquement les requêtes HTTPS et les abonnés pour lesquels l’authentification client est activée. Pour savoir comment configurer le module Event Grid de manière sécurisée, voir [Sécurité et authentification](../articles/event-grid/edge/security-authentication.md).
 
### <a name="select-your-iot-edge-device"></a>Sélectionnez votre appareil IoT Edge

1. Connectez-vous au [portail Azure](https://portal.azure.com)
1. Accédez à votre IoT Hub.
1. Sélectionnez **IoT Edge** dans le menu de la section **Gestion automatique des appareils**. 
1. Cliquez sur l’ID de l’appareil cible dans la liste des appareils
1. Sélectionnez **Définir modules**. Laissez la page ouverte. Vous continuerez avec les étapes de la prochaine section.

### <a name="configure-a-deployment-manifest"></a>Configurer un manifeste de déploiement

Un manifeste de déploiement est un document JSON qui décrit les modules à déployer, le flux des données entre les modules et les propriétés souhaitées des jumeaux de module. Le Portail Azure comprend un Assistant qui vous guide à travers la création du manifeste de déploiement, vous évitant de générer le document JSON manuellement.  Il comporte trois étapes : **Ajouter des modules**, **Spécifier des routes** et **Vérifier le déploiement**.

### <a name="add-modules"></a>Ajouter des modules

1. Dans la section **Modules de déploiement**, sélectionnez **Ajouter**
1. Dans la liste déroulante des types de modules, sélectionnez **Module IoT Edge**
1. Indiquez le nom, l’image et les options de création du conteneur :

[!INCLUDE [event-grid-edge-module-version-update](event-grid-edge-module-version-update.md)]

   * **Nom** : eventgridmodule
   * **URI de l’image**  : `mcr.microsoft.com/azure-event-grid/iotedge:latest`
   * **Options de création de conteneur** :

```json
    {
      "Env": [
        "inbound__clientAuth:clientCert__enabled=false",
        "outbound__webhook__httpsOnly=false"
      ],
      "HostConfig": {
        "PortBindings": {
          "4438/tcp": [
            {
              "HostPort": "4438"
            }
          ]
        }
      }
    }
```

 1. Cliquez sur **Enregistrer**.
 1. Cliquez sur **Suivant** pour passer à la section des itinéraires

    > [!NOTE]
    > Si vous utilisez une machine virtuelle Azure comme appareil Edge, ajoutez une règle de port entrant pour autoriser le trafic entrant sur le port 4438. Pour savoir comment ajouter la règle, voir [Guide pratique pour ouvrir des ports sur une machine virtuelle](../articles/virtual-machines/windows/nsg-quickstart-portal.md).


### <a name="setup-routes"></a>Configurer les itinéraires

 Conservez les itinéraires par défaut, puis sélectionnez **Suivant** pour passer à la section de vérification

### <a name="review-deployment"></a>Vérifier le déploiement

1. La section de vérification vous montre le manifeste de déploiement JSON qui a été créé en fonction de vos sélections dans les deux sections précédentes. Vérifiez que les deux modules apparaissent dans la liste : **$edgeAgent** et **$edgeHub**. Ils composent le runtime IoT Edge et sont les valeurs par défaut requises pour tous les déploiements.
1. Passez en revue les informations de votre déploiement, puis sélectionnez **Envoyer**.

### <a name="verify-your-deployment"></a>Vérifier votre déploiement

1. Une fois le déploiement envoyé, la page IoT Edge du hub IoT s’affiche à nouveau.
1. Sélectionnez l’**appareil IoT Edge** que vous avez ciblé avec le déploiement pour accéder à ses informations détaillées.
1. Dans les détails de l’appareil, vérifiez que le module Event Grid apparaît à la fois **Spécifié dans le déploiement** et **Signalé par l’appareil**.

Il peut s’écouler quelques instants avant que le module ne démarre sur l’appareil et qu’il ne soit renvoyé à IoT Hub. Actualisez la page pour afficher un état mis à jour.