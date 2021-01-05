---
title: Azure Stream Analytics sur IoT Edge
description: Créez des tâches de périphérie dans Azure Stream Analytics et déployez-les sur des appareils exécutant Azure IoT Edge.
ms.service: stream-analytics
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.topic: how-to
ms.date: 10/29/2020
ms.custom: seodec18
ms.openlocfilehash: cba81b8415f0f9cf7253e674e90ae09718b94d54
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93130474"
---
# <a name="azure-stream-analytics-on-iot-edge"></a>Azure Stream Analytics sur IoT Edge
 
Azure Stream Analytics (ASA) sur IoT Edge encourage les développeurs à déployer une intelligence analytique quasiment en temps réel plus proche des appareils IoT pour leur permettre de déverrouiller la valeur complète des données générées par l’appareil. Azure Stream Analytics est conçu pour une latence faible, une résilience, une utilisation efficace de la bande passante et la conformité. Les entreprises peuvent désormais déployer la logique de contrôle proche des opérations industrielles et compléter l’analytique du Big Data effectuée dans le cloud.  

Azure Stream Analytics sur IoT Edge s’exécute dans le framework [Azure IoT Edge](https://azure.microsoft.com/campaigns/iot-edge/). Une fois que le travail est créé dans ASA, vous pouvez le déployer et le gérer à l’aide d’IoT Hub.

## <a name="scenarios"></a>Scénarios
![Diagramme IoT Edge haut niveau](media/stream-analytics-edge/ASAedge-highlevel-diagram.png)

* **Contrôle et commande de faible latence** : par exemple, les systèmes de sécurité de fabrication doivent répondre aux données opérationnelles avec une latence très faible. Avec ASA sur IoT Edge, vous pouvez analyser les données de capteur quasiment en temps réel et émettre des commandes lorsque vous détectez des anomalies pour arrêter une machine ou déclencher des alertes.
*   **Connectivité au cloud limitée** : les systèmes stratégiques, tels que les équipements miniers à distance, les navires connectés ou les installations de forage offshore, ont besoin d’analyser les données et d’y réagir, même lorsque la connectivité au cloud est intermittente. Avec ASA, votre logique de diffusion en continu s’exécute indépendamment de la connectivité réseau et vous pouvez choisir ce que vous envoyez sur le cloud pour un traitement ultérieur ou pour y être stocké.
* **Bande passante limitée** : le volume de données produites par les moteurs à réaction ou par les voitures connectées peut être tellement important que les données doivent être filtrées ou traitées au préalable avant d’être envoyées vers le cloud. À l’aide d’ASA, vous pouvez filtrer ou agréger les données qui doivent être envoyées vers le cloud.
* **Conformité** : pour obtenir une conformité réglementaire, certaines données peuvent être rendues anonymes localement ou agrégées avant d’être envoyés vers le cloud.

## <a name="edge-jobs-in-azure-stream-analytics"></a>Tâches de périphérie dans Azure Stream Analytics
### <a name="what-is-an-edge-job"></a>Qu’est-ce qu’une tâche de périphérie ?

Les tâches ASA Edge s’exécutent dans des conteneurs déployés sur des [appareils Azure IoT Edge](../iot-edge/about-iot-edge.md). Elles sont composées de deux parties :
1.  Une partie cloud qui est responsable de la définition de tâche : les utilisateurs définissent des entrées, des sorties, des requêtes et d’autres paramètres (événements en désordre, etc.) dans le cloud.
2.  Un module en cours d’exécution sur vos appareils IoT. Il contient le moteur ASA et reçoit la définition de tâche à partir du cloud. 

ASA utilise IoT Hub pour déployer des tâches de périphérie sur les périphériques. Vous obtiendrez plus d’informations sur le [déploiement IoT Edge ici](../iot-edge/module-deployment-monitoring.md).

![Travail de périphérie Azure Stream Analytics](media/stream-analytics-edge/stream-analytics-edge-job.png)


### <a name="installation-instructions"></a>Instructions d’installation
La procédure générale est décrite dans le tableau suivant. Vous obtiendrez plus de détails dans les sections suivantes.

| Étape | Notes |
| --- | --- |
| **Créer un conteneur de stockage** | Les conteneurs de stockage sont utilisés pour enregistrer votre définition de tâche, là où ils sont accessibles par vos appareils IoT. <br>  Vous pouvez réutiliser un conteneur de stockage existant. |
| **Créer une tâche ASA Edge** | Créez une nouvelle tâche, sélectionnez **Edge** en tant qu’ **environnement d’hébergement**. <br> Ces tâches sont créées/gérées à partir du cloud et s’exécutent sur vos propres appareils IoT Edge. |
| **Configurer votre environnement IoT Edge sur vos appareils** | Instructions pour [Windows](../iot-edge/quickstart.md) ou [Linux](../iot-edge/quickstart-linux.md).|
| **Déployer ASA sur vos appareils IoT Edge** | La définition de tâche ASA est exportée vers le conteneur de stockage créé précédemment. |

Vous pouvez suivre [ce didacticiel pas à pas](../iot-edge/tutorial-deploy-stream-analytics.md) pour déployer votre première tâche ASA sur IoT Edge. La vidéo suivante permet de comprendre le processus d’exécution d’une tâche Stream Analytics sur un appareil IoT Edge :  


> [!VIDEO https://channel9.msdn.com/Events/Connect/2017/T157/player]

#### <a name="create-a-storage-container"></a>Créer un conteneur de stockage
Un conteneur de stockage est nécessaire pour exporter la requête ASA compilée et la configuration de tâche. Il est utilisé pour configurer l’image ASA Docker avec votre requête spécifique. 
1. Suivez ces [instructions](../storage/common/storage-account-create.md) pour créer un compte de stockage à partir du portail Azure. Vous pouvez conserver toutes les options par défaut pour utiliser ce compte avec ASA.
2. Dans le compte de stockage nouvellement créé, créez un conteneur de stockage d’objets blob :
    1. Cliquez sur **BLOB** , puis  **+ Conteneur »** . 
    2. Entrez un nom et conservez le conteneur en tant que **Privé**.

#### <a name="create-an-asa-edge-job"></a>Créer une tâche ASA Edge
> [!Note]
> Ce didacticiel est consacré à la création d’un travail ASA à l’aide du portail Azure. Vous pouvez également [utiliser le plug-in Visual Studio pour créer un travail ASA Edge](./stream-analytics-tools-for-visual-studio-edge-jobs.md).

1. Depuis le portail Azure, créez une nouvelle tâche Stream Analytics. [Lien direct pour créer une nouvelle tâche ASA](https://ms.portal.azure.com/#create/Microsoft.StreamAnalyticsJob).

2. Dans l’écran de création, sélectionnez **Edge** en tant qu’ **environnement d’hébergement** (voir l’illustration suivante)

   ![Créer une tâche Stream Analytics sur Edge](media/stream-analytics-edge/create-asa-edge-job.png)
3. Définition de la tâche
    1. **Définir le(s) flux d’entrée**. Définissez un ou plusieurs flux d’entrée pour votre tâche.
    2. Définissez les données de référence (facultatif).
    3. **Définir le(s) flux de sortie**. Définissez un ou plusieurs flux de sortie pour votre tâche. 
    4. **Définir la requête**. Définissez la requête ASA dans le cloud à l’aide de l’éditeur inclus. Le compilateur vérifie automatiquement la syntaxe prise en charge par ASA Edge. Vous pouvez également tester votre requête en téléchargeant des exemples de données. 

4. Définissez les informations du conteneur de stockage dans le menu **Paramètres IoT Edge**.

5. Définir des paramètres facultatifs
    1. **Ordre des événements**. Vous pouvez configurer une stratégie d’arrivée en désordre dans le portail. La documentation est disponible [ici](/stream-analytics-query/time-skew-policies-azure-stream-analytics).
    2. **Paramètres régionaux**. Définissez le format d’internationalisation.



> [!Note]
> Lors de la création d’un déploiement, ASA exporte la définition de tâche dans un conteneur de stockage. Cette définition de tâche reste la même pendant la durée d’un déploiement. Par conséquent, si vous souhaitez mettre à jour une tâche en cours d’exécution en périphérie, vous devez modifier la tâche dans ASA, puis créer un nouveau déploiement dans IoT Hub.


#### <a name="set-up-your-iot-edge-environment-on-your-devices"></a>Configurer votre environnement IoT Edge sur vos appareils
Les tâches de périphérie peuvent être déployées sur les appareils exécutant Azure IoT Edge.
Pour ce faire, vous devez procéder comme suit :
- Créez un IoT Hub.
- Installez le runtime Docker et IoT Edge sur vos appareils de périphérie.
- Définissez vos appareils comme **Appareils IoT Edge** dans IoT Hub.

Ces étapes sont décrites dans la documentation IoT Edge pour [Windows](../iot-edge/quickstart.md) ou [Linux](../iot-edge/quickstart-linux.md).  


####  <a name="deployment-asa-on-your-iot-edge-devices"></a>Déployer ASA sur vos appareils IoT Edge
##### <a name="add-asa-to-your-deployment"></a>Ajouter ASA à votre déploiement
- Dans le portail Azure, ouvrez IoT Hub, accédez à **IoT Edge** , puis cliquez sur l’appareil que vous souhaitez cibler pour ce déploiement.
- Sélectionnez **Set modules** (Définir les modules), puis sélectionnez **+ Ajouter** et choisissez **Azure Stream Analytics Module** (Module Azure Stream Analytics).
- Sélectionnez l’abonnement et la tâche ASA Edge que vous avez créée. Cliquez sur Enregistrer.
![Ajouter un module ASA dans votre déploiement](media/stream-analytics-edge/add-stream-analytics-module.png)


> [!Note]
> Dans cette étape, ASA crée un dossier nommé « EdgeJobs » dans le conteneur de stockage (s’il n’existe pas déjà). Pour chaque déploiement, un nouveau sous-dossier est créé dans le dossier « EdgeJobs ».
> Lorsque vous déployez votre projet sur les appareils IoT Edge, ASA crée une signature d’accès partagé (SAP) pour le fichier de définition de tâche. La clé SAP est transmise de façon sécurisée aux appareils IoT Edge à l’aide de jumeaux d’appareil. La durée avant expiration de cette clé est de trois ans à partir du jour de sa création. Lorsque vous mettez à jour une tâche IoT Edge, la SAP change, mais la version de l’image reste la même. Lorsque vous **mettez à jour** , suivez le flux de travail du déploiement. Ensuite, une notification de mise à jour est enregistrée sur l’appareil.


Pour plus d’informations sur les déploiements IoT Edge, consultez [cette page](../iot-edge/module-deployment-monitoring.md).


##### <a name="configure-routes"></a>Configurer des itinéraires
IoT Edge offre un moyen de router les messages entre les modules et entre les modules et IoT Hub de façon déclarative. La syntaxe complète est décrite [ici](../iot-edge/module-composition.md).
Les noms des entrées et sorties créés dans la tâche ASA peuvent être utilisés en tant que points de terminaison pour le routage.  

###### <a name="example"></a>Exemple

```json
{
    "routes": {
        "sensorToAsa":   "FROM /messages/modules/tempSensor/* INTO BrokeredEndpoint(\"/modules/ASA/inputs/temperature\")",
        "alertsToCloud": "FROM /messages/modules/ASA/* INTO $upstream",
        "alertsToReset": "FROM /messages/modules/ASA/* INTO BrokeredEndpoint(\"/modules/tempSensor/inputs/control\")"
    }
}

```
Cet exemple montre les itinéraires pour le scénario décrit dans l’image suivante. Il contient une tâche de périphérie appelée **ASA** , une entrée nommée **temperature** et une sortie nommée **alert**.
![Exemple de diagramme de routage des messages](media/stream-analytics-edge/edge-message-routing-example.png)

Cet exemple définit les itinéraires suivants :
- Tous les messages du module **tempSensor** sont envoyés au module nommé **ASA** à l’entrée nommée **temperature**.
- Toutes les sorties du module **ASA** sont envoyées au IoT Hub lié à cet appareil ($upstream).
- Toutes les sorties du module **ASA** sont envoyées au point de terminaison **control** de **tempSensor**.


## <a name="technical-information"></a>Informations techniques
### <a name="current-limitations-for-iot-edge-jobs-compared-to-cloud-jobs"></a>Limitations actuelles pour les tâches IoT Edge par rapport aux tâches cloud
L’objectif est d’obtenir la parité entre les tâches IoT Edge et les tâches cloud. La plupart des fonctionnalités du langage de requête SQL sont prises en charge, ce qui permet d’exécuter la même logique dans le cloud et IoT Edge.
Cependant, les fonctionnalités suivantes ne sont pas encore prises en charge pour les tâches de périphérie :
* Fonctions définies par l’utilisateur en JavaScript. Des fonctions définies par l’utilisateur sont disponibles en [C# pour les tâches IoT Edge](./stream-analytics-edge-csharp-udf.md) (préversion).
* Agrégats définis par l’utilisateur – (UDA).
* Fonctions Azure ML.
* Utilisation de plus de 14 agrégats dans une seule étape.
* Format AVRO pour l’entrée/sortie. À ce stade, seuls les formats CSV et JSON sont pris en charge.
* Les opérateurs SQL suivants :
    * PARTITION BY
    * GetMetadataPropertyValue
* Stratégie d’arrivée tardive

### <a name="runtime-and-hardware-requirements"></a>Runtime et conditions matérielles requises
Pour exécuter ASA sur IoT Edge, vous avez besoin d’appareils pouvant exécuter [Azure IoT Edge](https://azure.microsoft.com/campaigns/iot-edge/). 

ASA et Azure IoT Edge utilisent des conteneurs **Docker** pour fournir une solution portable s’exécutant sur plusieurs systèmes d’exploitation hôtes (Windows, Linux).

ASA sur IoT Edge est mis à disposition sous forme d’images Windows et Linux, s’exécutant sur les architectures x86-64 ou ARM (Advanced RISC Machines). 


### <a name="input-and-output"></a>Entrée et sortie
#### <a name="input-and-output-streams"></a>Flux d’entrée et de sortie
Les tâches ASA Edge peuvent obtenir des entrées et sorties à partir d’autres modules qui s’exécutent sur des appareils IoT Edge. Pour vous connecter à partir de modules spécifiques et à ces derniers, vous pouvez définir la configuration de routage au moment du déploiement. Pour plus d’informations, consultez [la documentation de composition du module IoT Edge](../iot-edge/module-composition.md).

Les formats CSV et JSON sont pris en charge pour les entrées et sorties.

Pour chaque flux d’entrée et de sortie que vous créez dans votre tâche ASA, un point de terminaison correspondant est créé dans votre module déployé. Ces points de terminaison sont utilisables dans les itinéraires de votre déploiement.

Actuellement, les seuls types d’entrée et de sortie de flux pris en charge sont Edge Hub. L’entrée de référence prend en charge le type de fichier de référence. D’autres sorties peuvent être atteintes à l’aide d’un travail cloud en aval. Par exemple, un travail Stream Analytics hébergé dans Edge envoie la sortie à Edge Hub, qui peuvent ensuite envoyer la sortie à IoT Hub. Vous pouvez utiliser un deuxième travail Azure Stream Analytics Analytique hébergé sur le cloud avec une entrée d’IoT Hub et une sortie vers Power BI ou un autre type de sortie.



##### <a name="reference-data"></a>Données de référence
Les données de référence (également appelées « tables de choix ») sont un jeu de données finies, statiques ou variant lentement au fil du temps par nature. Elles sont utilisées pour effectuer des recherches ou pour effectuer des mises en corrélation avec votre flux de données. Pour utiliser des données de référence dans votre tâche Azure Stream Analytics, vous utiliserez généralement une [JOINTURE de données de référence](/stream-analytics-query/reference-data-join-azure-stream-analytics) dans votre requête. Pour plus d’informations, consultez [Utiliser des données de référence pour effectuer des recherches dans Stream Analytics](stream-analytics-use-reference-data.md).

Seules les données de référence locales sont prises en charge. Lorsqu'une tâche est déployée sur un appareil IoT Edge, il charge les données de référence à partir du chemin de fichier défini par l'utilisateur.

Pour créer une tâche avec des données de référence sur Edge :

1. Créez une nouvelle entrée pour votre tâche.

2. Choisissez **données de référence** pour le **type de source**.

3. Préparez un fichier de données de référence sur l'appareil. Pour un conteneur Windows, placez le fichier de données de référence sur le lecteur local et partagez ce lecteur avec le conteneur Docker. Pour un conteneur Linux, créez un volume Docker et renseignez le fichier de données sur le volume.

4. Définissez le chemin d’accès du fichier. Pour un système d’exploitation hôte Windows et un conteneur Windows, utilisez le chemin d’accès absolu : `E:\<PathToFile>\v1.csv`. Pour un système d’exploitation hôte Windows et un conteneur Linux ou un système d’exploitation Linux et un conteneur Linux, utilisez le chemin d’accès du volume : `<VolumeName>/file1.txt`.

![Nouvelle entrée de données de référence pour la tâche Azure Stream Analytics sur IoT Edge](./media/stream-analytics-edge/Reference-Data-New-Input.png)

Les données de référence situées sur la mise à jour d'IoT Edge sont déclenchées par un déploiement. Une fois déclenché, le module ASA sélectionne les données mises à jour sans arrêter la tâche en cours.

Deux méthodes sont disponibles pour mettre à jour les données de référence :
* En mettant à jour le chemin des données de référence de votre tâche ASA à partir du portail Azure
* En mettant à jour le déploiement IoT Edge

## <a name="license-and-third-party-notices"></a>Licence et mentions tierces
* [Licence Azure Stream Analytics sur IoT Edge](https://go.microsoft.com/fwlink/?linkid=862827). 
* [Mentions tierces pour Azure Stream Analytics sur IoT Edge, version](https://go.microsoft.com/fwlink/?linkid=862828).

## <a name="azure-stream-analytics-module-image-information"></a>Informations sur l’image de module Azure Stream Analytics 

La dernière mise à jour des informations de version a été effectuée le 27-06-2019 :

- Image : `mcr.microsoft.com/azure-stream-analytics/azureiotedge:1.0.9-linux-amd64`
   - image de base : mcr.microsoft.com/dotnet/core/runtime:2.1.13-alpine
   - plateforme :
      - architecture : amd64
      - système d’exploitation : linux
 
- Image : `mcr.microsoft.com/azure-stream-analytics/azureiotedge:1.0.9-linux-arm32v7`
   - image de base : mcr.microsoft.com/dotnet/core/runtime:2.1.13-bionic-arm32v7
   - plateforme :
      - architecture : arm
      - système d’exploitation : linux
 
- Image : `mcr.microsoft.com/azure-stream-analytics/azureiotedge:1.0.9-linux-arm64`
   - image de base : mcr.microsoft.com/dotnet/core/runtime:3.0-bionic-arm64v8
   - plateforme :
      - architecture : arm64
      - système d’exploitation : linux
      
      
## <a name="get-help"></a>Obtenir de l’aide
Pour obtenir de l’aide supplémentaire, consultez notre [page de questions Microsoft Q&R sur Azure Stream Analytics](/answers/topics/azure-stream-analytics.html).

## <a name="next-steps"></a>Étapes suivantes

* [Plus d’informations sur Azure Iot Edge](../iot-edge/about-iot-edge.md)
* [Didacticiel pour ASA sur IoT Edge](../iot-edge/tutorial-deploy-stream-analytics.md)
* [Développer des travaux Edge Stream Analytics avec les outils Visual Studio](./stream-analytics-tools-for-visual-studio-edge-jobs.md)
* [Implémenter CI/CD pour Stream Analytics à l’aide d’API](stream-analytics-cicd-api.md)

<!--Link references-->
[stream.analytics.developer.guide]: ../stream-analytics-developer-guide.md
[stream.analytics.scale.jobs]: stream-analytics-scale-jobs.md
[stream.analytics.introduction]: stream-analytics-introduction.md
[stream.analytics.get.started]: stream-analytics-real-time-fraud-detection.md
[stream.analytics.query.language.reference]: /stream-analytics-query/stream-analytics-query-language-reference
[stream.analytics.rest.api.reference]: /rest/api/streamanalytics/