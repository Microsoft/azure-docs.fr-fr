---
title: Exécuter Azure Stream Analytics sur Azure Stack (préversion)
description: Créez un travail de périphérie Azure Stream Analytics et déployez-le sur Azure Stack Hub par le biais du runtime IoT Edge.
ms.service: stream-analytics
author: raan
ms.author: raan
ms.reviewer: mamccrea
ms.topic: how-to
ms.date: 08/21/2020
ms.custom: seodec18
ms.openlocfilehash: 21cf432576829b575d70a94227f28df373a4d899
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93126156"
---
# <a name="run-azure-stream-analytics-on-azure-stack-preview"></a>Exécuter Azure Stream Analytics sur Azure Stack (préversion)

> [!IMPORTANT]
> Cette fonctionnalité est en préversion et il n’est pas recommandé de l’utiliser en production.

Vous pouvez exécuter Azure Stream Analytics sur Azure Stack Hub en tant que module IoT Edge. Des configurations ont été ajoutées au module IoT Edge qui lui permettent d’interagir avec le stockage d’objets blob, Event Hubs et les hubs IoT s’exécutant dans un abonnement Azure Stack Hub en autorisant les URL personnalisées trouvées dans chaque emploi d’ Azure Stack Hub.

Avec Stream Analytics sur Azure Stack, vous pouvez créer des architectures véritablement hybrides pour le traitement des flux dans votre propre cloud autonome privé, qui peut être connecté avec des applications natives cloud utilisant des services Azure cohérents en local, ou bien déconnecté de ces applications. 

Cet article vous montre comment effectuer un streaming de données à partir d’IoT Hub ou d’un hub d’événements vers un autre hub d’événements ou stockage d’objets blob dans un abonnement Azure Stack Hub et traiter ces données avec Stream Analytics.

## <a name="set-up-environments"></a>Configurer des environnements

Azure Stream Analytics est un service hybride sur Azure Stack Hub. Il s’agit d’un module IoT Edge qui est configuré dans Azure, mais qui peut être exécuté sur Azure Stack Hub.  

Si vous débutez avec Azure Stack Hub ou IoT Edge, suivez les instructions ci-dessous pour configurer des environnements.

### <a name="prepare-the-azure-stack-hub-environment"></a>Préparer l’environnement Azure Stack Hub

Créez un abonnement Azure Stack Hub. Pour plus d’informations, consultez le [tutoriel sur la création d’un abonnement Azure Stack Hub.](/azure-stack/user/azure-stack-subscribe-services/)

Si vous souhaitez évaluer Azure Stack Hub sur votre propre serveur, vous pouvez utiliser le Kit de développement Azure Stack (ASDK).  Pour plus d’informations sur le Kit ASDK, consultez la [présentation du Kit ASDK](/azure-stack/asdk/).

### <a name="install-the-iot-edge-runtime"></a>Installer le runtime IoT Edge

Pour exécuter Azure Stream Analytics sur Azure Stack Hub, votre appareil doit avoir le runtime IoT Edge et doit disposer d’une connectivité réseau au hub Azure Stack ou être une machine virtuelle s’exécutant sur ce hub. Le runtime IoT Edge permet aux travaux de périphérie Stream Analytics de s’intégrer au stockage Azure et à Azure Event Hubs qui s’exécutent sur votre hub Azure Stack. Pour plus d’informations, consultez [Azure Stream Analytics sur IoT Edge](stream-analytics-edge.md) 

En plus de disposer d’un accès réseau aux ressources Azure Stack Hub, l’appareil IoT Edge (ou machine virtuelle) doit accéder à Azure IoT Hub dans le cloud public Azure pour configurer le module Stream Analytics. 

Les guides suivants montrent comment configurer le runtime IoT Edge sur votre appareil ou machine virtuelle :

* [Installer le runtime Azure IoT Edge sur Windows](../iot-edge/how-to-install-iot-edge.md)
* [Installer le runtime Azure IoT Edge sur des systèmes Linux Debian](../iot-edge/how-to-install-iot-edge.md)


## <a name="create-an-azure-stream-analytics-edge-job"></a>Créer un travail de périphérie Azure Stream Analytics

Les tâches ASA Edge s’exécutent dans des conteneurs déployés sur des appareils Azure IoT Edge. Elles sont composées de deux parties :
* Une partie cloud qui est responsable de la définition de tâche : les utilisateurs définissent des entrées, des sorties, des requêtes et d’autres paramètres (événements en désordre, etc.) dans le cloud.
* Un module en cours d’exécution sur vos appareils IoT. Il contient le moteur ASA et reçoit la définition de tâche à partir du cloud.

### <a name="create-a-storage-account"></a>Créez un compte de stockage.

Lorsque vous créez un travail Azure Stream Analytics pour s’exécuter sur un appareil IoT Edge, il doit être stocké de manière à pouvoir être appelé à partir de l’appareil. Vous pouvez utiliser un compte de stockage Azure existant ou en créer un.
1. Dans le portail Azure, accédez à **Créer une ressource > Stockage > Compte de stockage - blob, fichier, table, file d’attente**.
2. Fournissez les valeurs suivantes pour créer votre compte de stockage :

   | Champ | Valeur |
   | --- | --- |
   | Nom | Fournissez un nom unique pour votre compte de stockage. |
   | Emplacement | Choisissez un emplacement proche de vous.|
   | Abonnement | Choisissez le même abonnement que votre IoT Hub.|
   | Groupe de ressources | Nous vous recommandons d’utiliser le même groupe de ressources pour toutes les ressources de test que vous créez dans le cadre des [guides de démarrage rapide](../iot-edge/quickstart.md) et tutoriels IoT Edge. Par exemple, utilisez **IoTEdgeResources**. |

3. Conservez les valeurs par défaut pour les autres champs et sélectionnez **Créer**.


### <a name="create-a-new-job"></a>Créer une nouvelle tâche

1. Dans le portail Azure, accédez à **Créer une ressource > Internet des objets > Tâche Stream Analytics**.
2. Fournissez les valeurs suivantes pour créer votre compte de stockage :

   | Champ | Valeur |
   | --- | --- |
   | Nom du travail | Fournissez un nom pour votre travail. Par exemple, **IoTEdgeJob** |
   | Abonnement | Choisissez le même abonnement que votre IoT Hub.|
   | Groupe de ressources | Nous vous recommandons d’utiliser le même groupe de ressources pour toutes les ressources de test que vous créez dans le cadre des [guides de démarrage rapide](../iot-edge/quickstart.md) et tutoriels IoT Edge. Par exemple, utilisez **IoTEdgeResources**. |
   | Emplacement | Choisissez un emplacement proche de vous. |
   | Environnement d'hébergement | Sélectionnez **Edge**. |

3. Sélectionnez **Create** (Créer).

### <a name="configure-your-job"></a>Configurer votre travail

Lorsque votre travail Stream Analytics est créé dans le portail Azure, vous pouvez le configurer avec une entrée, une sortie et une requête à exécuter sur les données qui le traverse. Vous pouvez spécifier manuellement des entrées à partir d’un hub IoT ou d’un hub d’événements dans un abonnement Azure Stack Hub.

1. Accédez à votre travail Stream Analytics dans le portail Azure.
2. Sous **Configurer** , sélectionnez **Paramètres du compte de stockage** , puis choisissez le compte de stockage que vous avez créé à l’étape précédente.
   > [!div class="mx-imgBorder"]
   > [ ![Paramétrage du compte de stockage de travail](media/on-azure-stack/storage-account-settings.png) ](media/on-azure-stack/storage-account-settings.png#lightbox)
3. Sous **Topologie de la tâche** , sélectionnez **Entrées** puis **Ajouter une entrée de flux**.
4. Dans la liste déroulante, choisissez **Hub IoT** , **Hub d’événements** ou **Edge Hub**. 
5. Si l’entrée est un hub d’événements ou un hub IoT dans un abonnement Azure Stack Hub, fournissez les informations manuellement, comme indiqué ci-dessous.

   #### <a name="event-hub"></a>Event Hub

   | Champ | Valeur |
   | --- | --- |
   | Alias d’entrée | Nom convivial que vous utilisez dans la requête du travail pour faire référence à cette entrée. |
   | Espace de noms Service Bus | Conteneur pour un ensemble d’entités de messagerie. Lorsque vous créez un concentrateur Event Hub, vous créez également l’espace de noms. (Exemple : *sb://<Event Hub Name>.eventhub.shanghai.azurestack.corp.microsoft.com* ) |
   | Nom du hub d’événements | Nom du concentrateur Event Hub à utiliser comme entrée. |
   | Nom de la stratégie du hub d’événements | La stratégie d’accès partagé qui fournit l’accès au concentrateur Event Hub. Chaque stratégie d’accès partagé a un nom, les autorisations que vous définissez ainsi que des clés d’accès. Cette option est automatiquement renseignée, sauf si vous sélectionnez l’option pour indiquer manuellement les paramètres de l’Event Hub. |
   | Clé de stratégie Event Hub | Clé d’accès partagé utilisée pour autoriser l’accès au hub d’événements. Cette option est automatiquement renseignée, sauf si vous sélectionnez l’option pour indiquer manuellement les paramètres du concentrateur Event Hub. Vous pouvez la retrouver dans les paramètres du hub d’événements. |
   | Groupe de consommateurs du hub d’événements (facultatif) | Il est vivement recommandé d’utiliser un groupe de consommateurs différent pour chaque travail Stream Analytics. Cette chaîne identifie le groupe de consommateurs à utiliser pour ingérer les données du concentrateur Event Hub. Si aucun groupe de consommateurs n’est spécifié, le travail Stream Analytics utilise le groupe de consommateurs $Default. |
   | Nombre de partitions | Nombre de partitions dans un hub d’événements. |

   > [!div class="mx-imgBorder"]
   > [ ![Entrée du hub d’événements](media/on-azure-stack/event-hub-input.png) ](media/on-azure-stack/event-hub-input.png#lightbox)

   #### <a name="iot-hub"></a>IoT Hub

   | Champ | Valeur |
   | --- | --- |
   | Alias d’entrée | Nom convivial que vous utilisez dans la requête du travail pour faire référence à cette entrée. |
   | IoT Hub | Le nom du concentrateur IoT Hub à utiliser comme entrée. (Exemple : *<IoT Hub Name>.shanghai.azurestack.corp.microsoft.com* ) |
   | Nom de la stratégie d’accès partagé | La stratégie d’accès partagé qui fournit l’accès au concentrateur IoT Hub. Chaque stratégie d’accès partagé a un nom, les autorisations que vous définissez ainsi que des clés d’accès. |
   | Clé de la stratégie d’accès partagé | La clé d’accès partagé utilisée pour autoriser l’accès au concentrateur IoT Hub. Cette option est automatiquement renseignée, sauf si vous sélectionnez l’option pour indiquer manuellement les paramètres du concentrateur IoT Hub. |
   | Groupe de consommateurs (facultatif) | Il est vivement recommandé d’utiliser un groupe de consommateurs différent pour chaque travail Stream Analytics. Le groupe de consommateurs à utiliser pour ingérer les données du concentrateur IoT Hub. Stream Analytics utilise le groupe de consommateurs $Default, sauf si vous spécifiez autre chose. |
   | Nombre de partitions | Nombre de partitions dans un hub d’événements. |

   > [!div class="mx-imgBorder"]
   > [ ![Entrée du hub IoT](media/on-azure-stack/iot-hub-input.png) ](media/on-azure-stack/iot-hub-input.png#lightbox)

6. Conservez les valeurs par défaut pour les autres champs et sélectionnez Enregistrer.
7. Sous Topologie de la tâche, ouvrez Sorties, puis sélectionnez Ajouter.
8. Dans la liste déroulante, choisissez Stockage Blob, Hub d’événements ou Hub Edge.
9. Si la sortie est un hub d’événements ou un stockage Blob dans un abonnement Azure Stack Hub, fournissez les informations manuellement, comme indiqué ci-dessous.

   #### <a name="event-hub"></a>Event Hub

   | Champ | Valeur |
   | --- | --- |
   | Alias de sortie | Nom convivial utilisé dans les requêtes pour diriger la sortie de requête vers cet Event Hub. |
   | Espace de noms Service Bus | Conteneur pour un ensemble d’entités de messagerie. En créant un hub d’événements, vous avez également créé un espace de noms Service Bus. (Exemple : *sb://<Event Hub Name>.eventhub.shanghai.azurestack.corp.microsoft.com* ) |
   | Nom du hub d’événements | Nom de votre sortie Event Hub. |
   | Nom de la stratégie du hub d’événements | Stratégie d’accès partagé que vous pouvez créer dans l’onglet Configurer de l’Event Hub. Chaque stratégie d’accès partagé a un nom, les autorisations que vous définissez ainsi que des clés d’accès. |
   | Clé de stratégie Event Hub | Clé d’accès partagé utilisée pour authentifier l’accès à l’espace de noms Event Hub. |

   > [!div class="mx-imgBorder"]
   > [ ![Sortie du hub d’événements](media/on-azure-stack/event-hub-output.png) ](media/on-azure-stack/event-hub-output.png#lightbox)

   #### <a name="blob-storage"></a>Stockage Blob 

   | Champ | Valeur |
   | --- | --- |
   | Alias de sortie | Nom convivial utilisé dans les requêtes pour diriger la sortie de requête vers ce stockage d’objets blob. |
   | Compte de stockage | Nom du compte de stockage dans lequel vous envoyez votre sortie. (Exemple : *<Storage Account Name>.blob.shanghai.azurestack.corp.microsoft.com* ) |
   | Clé du compte de stockage | Clé secrète associée au compte de stockage. Cette option est automatiquement renseignée, sauf si vous sélectionnez l’option pour indiquer manuellement les paramètres du stockage d’objets blob. |

> [!NOTE]
> Le format Parquet n’est pas pris en charge pour les travaux de périphérie sur Azure Stack Hub. Pour le Nombre minimal de lignes et la Durée maximale, utilisez 0 ou laissez-les vides.


## <a name="deploy-stream-analytics-on-a-vm-or-device-connected-to-azure-stack"></a>Déployer Stream Analytics sur une machine virtuelle ou un appareil connecté à Azure Stack

1. Dans le portail Azure, ouvrez IoT Hub. Accédez à **IoT Edge** , puis cliquez sur l’appareil (machine virtuelle) que vous souhaitez cibler pour ce déploiement.
2. Sélectionnez **Définir des modules**. Ensuite, sélectionnez **+ Ajouter** et choisissez **Module Azure Stream Analytics**. 
3. Sélectionnez l’abonnement et le travail de périphérie Stream Analytics que vous avez créé. Cliquez sur **Enregistrer** et sélectionnez **Suivant : Routes**.

   > [!div class="mx-imgBorder"]
   > [ ![Ajouter des modules](media/on-azure-stack/edge-modules.png) ](media/on-azure-stack/edge-modules.png#lightbox)

4. Cliquez sur **Vérifier + créer >** .
5. À l’étape **Vérifier + créer** , sélectionnez **Créer**. 
   > [!div class="mx-imgBorder"]
   > [ ![Manifeste](media/on-azure-stack/module-content.png) ](media/on-azure-stack/module-content.png#lightbox)
6. Vérifiez que le module est ajouté à la liste.
   > [!div class="mx-imgBorder"]
   > [ ![Page de déploiement](media/on-azure-stack/edge-deployment.png) ](media/on-azure-stack/edge-deployment.png#lightbox)

## <a name="next-steps"></a>Étapes suivantes
- [Azure Stream Analytics sur IoT Edge](./stream-analytics-edge.md)
- [Développer des travaux de périphérie Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference)