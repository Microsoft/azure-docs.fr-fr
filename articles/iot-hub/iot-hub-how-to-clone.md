---
title: Comment cloner un hub IoT Azure
description: Comment cloner un hub IoT Azure
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 12/09/2019
ms.author: robinsh
ms.openlocfilehash: 370ea2f16632ae18142f0770742e5a52d3cabae0
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92151660"
---
# <a name="how-to-clone-an-azure-iot-hub-to-another-region"></a>Comment cloner un hub IoT Azure dans une autre région

Cet article explore les différentes façons de cloner un IoT Hub et fournit des questions auxquelles vous devez répondre avant de commencer. Voici plusieurs raisons pour lesquelles vous pouvez souhaiter cloner un hub IoT :
 
* Vous déplacez votre société d’une région vers une autre, par exemple d’Europe vers l’Amérique du Nord (ou vice versa) et vous souhaitez que vos ressources et données soient géographiquement proches de votre nouvel emplacement. Vous devez donc déplacer votre hub.

* Vous configurez un hub pour un environnement de développement ou de production.

* Vous souhaitez effectuer une implémentation personnalisée de la haute disponibilité à plusieurs hubs. Pour plus d’informations, consultez[Comment obtenir une section de haute disponibilité inter-régions de la haute disponibilité et récupération d'urgence IoT Hub](iot-hub-ha-dr.md#achieve-cross-region-ha).

* Vous souhaitez augmenter le nombre de [partitions](iot-hub-scaling.md#partitions) configurées pour votre hub. Cette valeur est définie lorsque vous créez votre hub pour la première fois et ne peut pas être modifiée. Vous pouvez utiliser les informations de cet article pour cloner votre hub et, lors de la création du clone, augmenter le nombre de partitions.

Pour cloner un hub, vous avez besoin d’un abonnement avec un accès administratif au hub d’origine. Vous pouvez placer le nouveau hub dans un nouveau groupe de ressources et une nouvelle région, dans le même abonnement que le hub d’origine ou même dans un nouvel abonnement. Vous ne pouvez pas utiliser le même nom, car le nom du hub doit être globalement unique.

> [!NOTE]
> À l’heure actuelle, il n’existe pas de fonctionnalité permettant de cloner automatiquement un hub IoT. Il s’agit principalement d’un processus manuel, qui est donc souvent sujet aux erreurs. La complexité du clonage d’un hub est directement proportionnelle à la complexité du hub. Par exemple, le clonage d’un hub IoT sans routage de messages est relativement simple. Si vous ajoutez le routage des messages comme une seule complexité, le clonage du hub devient au moins un ordre de magnitude plus compliqué. Si vous déplacez également les ressources utilisées pour les points de terminaison de routage, il s’agit d’un autre ordre de magnitude plus compliqué. 

## <a name="things-to-consider"></a>Points importants à prendre en compte

Il y a plusieurs points à prendre en compte avant de cloner un hub IoT.

* Assurez-vous que toutes les fonctionnalités disponibles à l’emplacement d’origine sont également disponibles dans le nouvel emplacement. Certains services sont en préversion et toutes les fonctionnalités ne sont pas disponibles partout.

* Ne supprimez pas les ressources d’origine avant de créer et de vérifier la version clonée. Une fois que vous avez supprimé un hub, il n’a plus aucun moyen de le récupérer pour vérifier les paramètres ou les données afin de s’assurer que le hub est correctement répliqué.

* De nombreuses ressources nécessitent des noms globaux uniques. Vous devez donc utiliser des noms différents pour les versions clonées. Vous devez également utiliser un nom différent pour le groupe de ressources auquel appartient le hub cloné. 

* Les données de l’hub IoT d’origine ne sont pas migrées. Ceci inclus les messages de télémétrie, commandes cloud-à-appareil (C2D) et informations liés aux travaux tels que les planifications et historiques. Les résultats de mesures et de journalisation ne sont pas non plus migrés. 

* Pour les données ou les messages acheminés vers le stockage Azure, vous pouvez laisser les données dans le compte de stockage d’origine, transférer ces données vers un nouveau compte de stockage dans la nouvelle région ou laisser les anciennes données sur place et créer un nouveau compte de stockage dans le nouvel emplacement pour les nouvelles données. Pour plus d’informations sur le déplacement des données dans le stockage blob, consultez [Prise en main d’AzCopy](../storage/common/storage-use-azcopy-v10.md).

* Il est impossible de migrer les données des rubriques et files d’attente Event Hubs et Service Bus. Il s’agit de données à un point dans le temps qui ne sont pas stockées après le traitement des messages.

* Vous devez planifier un temps d’arrêt pour la migration. Le clonage des appareils sur le nouveau hub prend du temps. Si vous utilisez la méthode Importer/Exporter, le test d’évaluation a révélé qu’il pouvait prendre environ deux heures pour déplacer 500 000 appareils et quatre heures pour déplacer un million d’appareils. 

* Vous pouvez copier les appareils sur le nouveau hub sans arrêter ou modifier les appareils. 

    * Si les appareils ont été approvisionnés à l’origine à l’aide de DPS, leur réapprovisionnement met à jour les informations de connexion stockées sur chaque appareil. 
    
    * Dans le cas contraire, vous devez utiliser la méthode Importer/Exporter pour déplacer les appareils, puis les appareils doivent être modifiés pour utiliser le nouveau hub. Par exemple, vous pouvez configurer votre appareil pour utiliser le nom d’hôte IoT Hub à partir des propriétés souhaitées du jumeau. L’appareil prend le nom d’hôte d’IoT Hub, déconnecte l’appareil de l’ancien hub et le reconnecte au nouveau.
    
* Vous devez mettre à jour tous les certificats que vous utilisez pour pouvoir les utiliser avec les nouvelles ressources. En outre, vous avez probablement défini un hub dans une table DNS quelque part, vous devrez mettre à jour ces informations DNS.

## <a name="methodology"></a>Méthodologie

Il s’agit de la méthode générale recommandée pour déplacer un hub IoT d’une région à une autre. Pour le routage des messages, cela suppose que les ressources ne sont pas déplacées vers la nouvelle région. Pour plus d’informations, consultez la [section sur le routage des messages](#how-to-handle-message-routing).

   1. Exportez le hub et ses paramètres dans un modèle Resource Manager. 
   
   1. Apportez les modifications nécessaires au modèle, telles que la mise à jour de toutes les occurrences du nom et de l’emplacement du hub cloné. Pour toutes les ressources du modèle utilisé pour les points de terminaison de routage des messages, mettez à jour la clé dans le modèle pour cette ressource.
   
   1. Importez le modèle dans un nouveau groupe de ressources dans le nouvel emplacement. Cela crée le clone.

   1. Déboguez si nécessaire. 
   
   1. Ajoutez tout ce qui n’a pas été exporté dans le modèle. 
   
       Par exemple, les groupes de consommateurs ne sont pas exportés vers le modèle. Vous devez ajouter manuellement les groupes de consommateurs au modèle ou utiliser le [Portail Azure](https://portal.azure.com) une fois le hub créé. Voici un exemple d’ajout d’un groupe de consommateurs à un modèle dans l’article [Utiliser un modèle Azure Resource Manager pour configurer le routage des messages IoT Hub](tutorial-routing-config-message-routing-rm-template.md).
       
   1. Copiez les appareils à partir du hub d’origine vers le clone. Ce sujet est abordé dans la section [Gestion des appareils inscrits auprès de l’hub IoT](#managing-the-devices-registered-to-the-iot-hub).

## <a name="how-to-handle-message-routing"></a>Comment gérer le routage des messages

Si votre hub utilise le [routage personnalisé](iot-hub-devguide-messages-read-custom.md), l’exportation du modèle pour le hub inclut la configuration de routage, mais elle n’inclut pas les ressources elles-mêmes. Vous devez choisir s’il faut déplacer les ressources de routage vers le nouvel emplacement ou les laisser sur place et continuer à les utiliser « tel quel ». 

Par exemple, imaginons que vous avez un hub dans l’ouest des États-Unis qui achemine les messages vers un compte de stockage (également dans l’ouest des États-Unis) et que vous souhaitez déplacer le hub vers l’est des États-Unis. Vous pouvez déplacer le hub et le faire toujours acheminer des messages vers le compte de stockage dans l’ouest des États-Unis ou vous pouvez déplacer le hub et déplacer également le compte de stockage. Il peut y avoir un faible impact sur les performances du routage des messages vers les ressources de point de terminaison dans une autre région.

Vous pouvez déplacer un hub qui utilise le routage des messages assez facilement si vous ne déplacez pas également les ressources utilisées pour les points de terminaison du routage. 

Si le hub utilise le routage des messages, vous avez deux possibilités. 

1. Déplacez les ressources utilisées pour les points de terminaison du routage vers le nouvel emplacement.

    * Vous devez créer vous-même les nouvelles ressources manuellement dans le [Portail Azure](https://portal.azure.com) ou à l’aide de modèles Resource Manager. 

    * Vous devez renommer toutes les ressources lorsque vous les créez dans le nouvel emplacement, car elles ont des noms globaux uniques. 
     
    * Vous devez mettre à jour les noms de ressources et les clés de ressource dans le modèle du nouveau hub, avant de créer le nouveau hub. Les ressources doivent être présentes lors de la création du nouveau hub.

1. Ne déplacez pas les ressources utilisées pour les points de terminaison du routage. Utilisez-les « sur place ».

   * À l’étape où vous modifiez le modèle, vous devrez récupérer les clés pour chaque ressource de routage et les placer dans le modèle avant de créer le nouveau hub. 

   * Le hub fait toujours référence aux ressources de routage d’origine et achemine les messages en fonction de la configuration.

   * Vous aurez une faible baisse des performances, car les ressources du hub et du point de terminaison de routage ne se trouvent pas au même emplacement.

## <a name="prepare-to-migrate-the-hub-to-another-region"></a>Préparer la migration du hub vers une autre région

Cette section fournit des instructions spécifiques pour la migration du hub.

### <a name="find-the-original-hub-and-export-it-to-a-resource-template"></a>Recherchez le hub d’origine et exportez-le dans un modèle de ressource.

1. Connectez-vous au [portail Azure](https://portal.azure.com). 

1. Accédez à **Groupes de ressources** et sélectionnez le groupe de ressources qui contient le hub que vous souhaitez déplacer. Vous pouvez également accéder aux **Ressources** et rechercher le hub de cette façon. Sélectionnez le hub.

1. Sélectionnez **Exporter le modèle** dans la liste des propriétés et des paramètres du hub. 

   ![Capture d’écran montrant la commande permettant d’exporter le modèle pour l’IoT Hub.](./media/iot-hub-how-to-clone/iot-hub-export-template.png)

1. Sélectionnez **Télécharger** pour télécharger le modèle. Enregistrez le fichier quelque part pour le retrouver. 

   ![Capture d’écran montrant la commande permettant de télécharger le modèle pour l’IoT Hub.](./media/iot-hub-how-to-clone/iot-hub-download-template.png)

### <a name="view-the-template"></a>Afficher le modèle 

1. Accédez au dossier Téléchargements (ou au dossier que vous avez utilisé lors de l’exportation du modèle) et recherchez le fichier zip. Ouvrez le fichier zip et recherchez le fichier appelé `template.json`. Sélectionnez-le, puis sélectionnez Ctrl + C pour copier le modèle. Accédez à un autre dossier qui ne se trouve pas dans le fichier zip et collez le fichier (Ctrl + V). Vous pouvez maintenant le modifier.
 
    L’exemple suivant concerne un hub générique sans configuration de routage. Il s’agit d’un hub de niveau S1 (avec 1 unité) appelé **ContosoTestHub29358** dans la région **westus**. Voici le modèle exporté.

    ``` json
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "IotHubs_ContosoTestHub29358_name": {
                "defaultValue": "ContosoTestHub29358",
                "type": "String"
            }
        },
        "variables": {},
        "resources": [
            {
                "type": "Microsoft.Devices/IotHubs",
                "apiVersion": "2018-04-01",
                "name": "[parameters('IotHubs_ContosoTestHub29358_name')]",
                "location": "westus",
                "sku": {
                    "name": "S1",
                    "tier": "Standard",
                    "capacity": 1
                },
                "properties": {
                    "operationsMonitoringProperties": {
                        "events": {
                            "None": "None",
                            "Connections": "None",
                            "DeviceTelemetry": "None",
                            "C2DCommands": "None",
                            "DeviceIdentityOperations": "None",
                            "FileUploadOperations": "None",
                            "Routes": "None"
                        }
                    },
                    "ipFilterRules": [],
                    "eventHubEndpoints": {
                        "events": {
                            "retentionTimeInDays": 1,
                            "partitionCount": 2,
                            "partitionIds": [
                                "0",
                                "1"
                            ],
                            "path": "contosotesthub29358",
                            "endpoint": "sb://iothub-ns-contosotes-2227755-92aefc8b73.servicebus.windows.net/"
                        },
                        "operationsMonitoringEvents": {
                            "retentionTimeInDays": 1,
                            "partitionCount": 2,
                            "partitionIds": [
                                "0",
                                "1"
                            ],
                            "path": "contosotesthub29358-operationmonitoring",
                            "endpoint": "sb://iothub-ns-contosotes-2227755-92aefc8b73.servicebus.windows.net/"
                        }
                    },
                    "routing": {
                        "endpoints": {
                            "serviceBusQueues": [],
                            "serviceBusTopics": [],
                            "eventHubs": [],
                            "storageContainers": []
                        },
                        "routes": [],
                        "fallbackRoute": {
                            "name": "$fallback",
                            "source": "DeviceMessages",
                            "condition": "true",
                            "endpointNames": [
                                "events"
                            ],
                            "isEnabled": true
                        }
                    },
                    "storageEndpoints": {
                        "$default": {
                            "sasTtlAsIso8601": "PT1H",
                            "connectionString": "",
                            "containerName": ""
                        }
                    },
                    "messagingEndpoints": {
                        "fileNotifications": {
                            "lockDurationAsIso8601": "PT1M",
                            "ttlAsIso8601": "PT1H",
                            "maxDeliveryCount": 10
                        }
                    },
                    "enableFileUploadNotifications": false,
                    "cloudToDevice": {
                        "maxDeliveryCount": 10,
                        "defaultTtlAsIso8601": "PT1H",
                        "feedback": {
                            "lockDurationAsIso8601": "PT1M",
                            "ttlAsIso8601": "PT1H",
                            "maxDeliveryCount": 10
                        }
                    },
                    "features": "None"
                }
            }
        ]
    }
    ```

### <a name="edit-the-template"></a>Modifier le modèle 

Vous devez apporter des modifications avant de pouvoir utiliser le modèle pour créer le nouveau hub dans la nouvelle région. Utilisez [VS Code](https://code.visualstudio.com) ou un éditeur de texte pour modifier le modèle.

#### <a name="edit-the-hub-name-and-location"></a>Modifier le nom et l’emplacement du hub

1. Supprimez la section Paramètres en haut, il est bien plus simple d’utiliser simplement le nom du hub, car nous n’allons pas disposer de plusieurs paramètres. 

    ``` json
        "parameters": {
            "IotHubs_ContosoTestHub29358_name": {
                "defaultValue": "ContosoTestHub29358",
                "type": "String"
            }
        },
    ```

1. Modifiez le nom pour qu’il utilise le nom réel (nouveau) au lieu de le récupérer à partir d’un paramètre (que vous avez supprimé à l’étape précédente). 

    Pour le nouveau hub, utilisez le nom du hub d’origine plus le *clone* de la chaîne pour créer le nouveau nom. Commencez par nettoyer le nom et l’emplacement du hub.
    
    Ancienne version :

    ``` json 
    "name": "[parameters('IotHubs_ContosoTestHub29358_name')]",
    "location": "westus",
    ```
    
    Nouvelle version : 

    ``` json 
    "name": "ContosoTestHub29358clone",
    "location": "eastus",
    ```

    Ensuite, vous découvrirez que les valeurs du **chemin d’accès** contiennent l’ancien nom du hub. Modifiez-les pour utiliser le nouveau. Il s’agit des valeurs de chemin d’accès sous **eventHubEndpoints** appelées **événements** et **OperationsMonitoringEvents**.

    Lorsque vous avez terminé, votre section points terminaux de Event Hub doit ressembler à ceci :

    ``` json
    "eventHubEndpoints": {
        "events": {
            "retentionTimeInDays": 1,
            "partitionCount": 2,
            "partitionIds": [
                "0",
                "1"
            ],
            "path": "contosotesthub29358clone",
            "endpoint": "sb://iothub-ns-contosotes-2227755-92aefc8b73.servicebus.windows.net/"
        },
        "operationsMonitoringEvents": {
            "retentionTimeInDays": 1,
            "partitionCount": 2,
            "partitionIds": [
                "0",
                "1"
            ],
            "path": "contosotesthub29358clone-operationmonitoring",
            "endpoint": "sb://iothub-ns-contosotes-2227755-92aefc8b73.servicebus.windows.net/"
        }
    ```

#### <a name="update-the-keys-for-the-routing-resources-that-are-not-being-moved"></a>Mettre à jour les clés pour les ressources de routage qui ne sont pas déplacées

Lorsque vous exportez le modèle Resource Manager pour un hub pour lequel le routage est configuré, vous constatez que les clés de ces ressources ne sont pas fournies dans le modèle exporté, leur placement est indiqué par des astérisques. Vous devez les remplir en accédant à ces ressources dans le portail et en récupérant les clés **avant** d’importer le modèle du nouveau modèle de hub et de créer le hub. 

1. Récupérez les clés requises pour toutes les ressources de routage et placez-les dans le modèle. Vous pouvez récupérer la ou les clés de la ressource dans le [Portail Azure ](https://portal.azure.com). 

   Par exemple, si vous acheminez des messages vers un conteneur de stockage, recherchez le compte de stockage dans le portail. Dans la section Paramètres, sélectionnez **clés d’accès**, puis copiez l’une des clés. Voici à quoi ressemble la clé lorsque vous exportez le modèle pour la première fois :

   ```json
   "connectionString": "DefaultEndpointsProtocol=https;
   AccountName=fabrikamstorage1234;AccountKey=****",
   "containerName": "fabrikamresults",
   ```

1. Après avoir récupéré la clé de compte pour le compte de stockage, placez-la dans le modèle dans la clause `AccountKey=****` à la place des astérisques. 

1. Pour les files d’attente Service bus, récupérez la clé d’accès partagé correspondant à SharedAccessKeyName. Voici la clé et le `SharedAccessKeyName` dans le fichier JSON :

   ```json
   "connectionString": "Endpoint=sb://fabrikamsbnamespace1234.servicebus.windows.net:5671/;
   SharedAccessKeyName=iothubroutes_FabrikamResources;
   SharedAccessKey=****;
   EntityPath=fabrikamsbqueue1234",
   ```

1. Il en va de même pour les rubriques Service Bus et les connexions Event Hub.

#### <a name="create-the-new-routing-resources-in-the-new-location"></a>Créer les nouvelles ressources de routage dans le nouvel emplacement

Cette section s’applique uniquement si vous déplacez les ressources utilisées par le hub pour les points de terminaison de routage.

Si vous souhaitez déplacer les ressources de routage, vous devez configurer manuellement les ressources dans le nouvel emplacement. Vous pouvez créer les ressources de routage à l’aide du [Portail Azure](https://portal.azure.com) ou en exportant le modèle Resource Manager pour chacune des ressources utilisées par le routage des messages, en les modifiant et en les important. Une fois les ressources configurées, vous pouvez importer le modèle de hub (qui comprend la configuration du routage).

1. Créez chaque ressource utilisée par le routage. Vous pouvez effectuer cette opération manuellement à l’aide du [Portail Azure](https://portal.azure.com) ou créer les ressources à l’aide de modèles Resource Manager. Si vous souhaitez utiliser des modèles, procédez comme suit :

    1. Pour chaque ressource utilisée par le routage, exportez-la vers un modèle Resource Manager.
    
    1. Mettez à jour le nom et l’emplacement de la ressource. 

    1. Mettez à jour les références croisées entre les ressources. Par exemple, si vous créez un modèle pour un nouveau compte de stockage, vous devez mettre à jour le nom du compte de stockage dans ce modèle et tous les autres modèles qui y font référence. Dans la plupart des cas, la section routage du modèle pour le hub est le seul autre modèle qui fait référence à la ressource. 

    1. Importez chacun des modèles, qui déploie chaque ressource.

    Une fois les ressources utilisées par le routage configurées et en cours d’exécution, vous pouvez continuer.

1. Dans le modèle pour le hub IoT, remplacez le nom de chacune des ressources de routage par son nouveau nom et mettez à jour l’emplacement si nécessaire. 

Vous disposez maintenant d’un modèle qui crée un nouveau hub qui ressemble presque exactement à l’ancien hub, en fonction de la façon dont vous avez décidé de gérer le routage.

## <a name="move----create-the-new-hub-in-the-new-region-by-loading-the-template"></a>Déplacer--créer le nouveau hub dans la nouvelle région en chargeant le modèle

Créer le nouveau hub dans le nouvel emplacement à l’aide du modèle. Si vous avez des ressources de routage qui vont être déplacées, les ressources doivent être configurées dans le nouvel emplacement et les références du modèle mises à jour pour correspondre. Si vous ne déplacez pas les ressources de routage, elles doivent se trouver dans le modèle avec les clés mises à jour.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Sélectionnez **Créer une ressource**. 

1. Dans la zone de recherche, entrez « déploiement de modèle » et sélectionnez Entrée.

1. Sélectionnez **déploiement de modèle (déployer à l’aide de modèles personnalisés)** . Vous accédez à un écran Template deployment. Sélectionnez **Create** (Créer). L'écran suivant s'affiche :

   ![Capture d’écran montrant la commande de génération de votre propre modèle](./media/iot-hub-how-to-clone/iot-hub-custom-deployment.png)

1. Sélectionnez **Générer votre propre modèle dans l’éditeur**, ce qui vous permet de télécharger votre modèle à partir d’un fichier. 

1. Sélectionnez **Charger le fichier**. 

   ![Capture d’écran montrant la commande de chargement d’un modèle de fichier](./media/iot-hub-how-to-clone/iot-hub-upload-file.png)

1. Recherchez le nouveau modèle que vous avez modifié et sélectionnez-le, puis sélectionnez **Ouvrir**. Cela permet de charger votre modèle dans la fenêtre d’édition. Sélectionnez **Enregistrer**. 

   ![Capture d’écran montrant le chargement du modèle](./media/iot-hub-how-to-clone/iot-hub-loading-template.png)

1. Complétez les champs suivants.

   **Abonnement** : sélectionnez l'abonnement à utiliser.

   **Groupe de ressources** : créez un nouveau groupe de ressources dans un nouvel emplacement. Si vous en avez déjà configuré un nouveau, vous pouvez le sélectionner au lieu d’en créer un autre.

   **Emplacement** : Si vous avez sélectionné un groupe de ressources existant, il est renseigné pour correspondre à l’emplacement du groupe de ressources. Si vous avez créé un nouveau groupe de ressources, il s’agit de son emplacement.

   **Case J’accepte** : cela signifie que vous acceptez de payer pour la ou les ressources que vous créez.

1. Sélectionnez le bouton **Acheter**.

Le portail valide désormais votre modèle et déploie votre hub cloné. Si vous avez des données de configuration de routage, elles sont incluses dans le nouveau hub, mais elles pointent vers les ressources de l’emplacement précédent.

## <a name="managing-the-devices-registered-to-the-iot-hub"></a>Gestion des appareils enregistrés auprès du hub IoT

Maintenant que votre clone est opérationnel, vous devez copier tous les appareils du hub d’origine vers le clone. 

Pour ce faire, plusieurs options s’offrent à vous. À l’origine, vous avez utilisé [Service de provisionnement des appareils (DPS)](../iot-dps/about-iot-dps.md)pour approvisionner les appareils, ou vous ne l’avez pas fait. Si vous l’avez fait, cela n’est pas difficile. Si ce n’est pas le cas, cela peut s’avérer très compliqué. 

Si vous n’avez pas utilisé DPS pour approvisionner vos appareils, vous pouvez ignorer la section suivante et commencer par [Utilisation de Importer/Exporter pour déplacer les appareils vers le nouveau hub](#using-import-export-to-move-the-devices-to-the-new-hub).

## <a name="using-dps-to-re-provision-the-devices-in-the-new-hub"></a>Utilisation de DPS pour réapprovisionner les appareils dans le nouveau hub

Pour utiliser DPS pour déplacer les appareils vers le nouvel emplacement, consultez [Comment réapprovisionner des appareils](../iot-dps/how-to-reprovision.md). Lorsque vous avez terminé, vous pouvez afficher les appareils dans le [Portail Azure](https://portal.azure.com) et vérifier qu’ils se trouvent dans le nouvel emplacement.

Accédez au nouveau hub à l’aide du [Portail Azure](https://portal.azure.com). Sélectionnez votre hub, puis sélectionnez **Appareils IoT**. Vous voyez les appareils qui ont été réapprovisionnés sur le hub cloné. Vous pouvez également afficher les propriétés du hub cloné. 

Si vous avez implémenté le routage, testez et assurez-vous que vos messages sont acheminés correctement vers les ressources.

### <a name="committing-the-changes-after-using-dps"></a>Validation des modifications après utilisation de DPS

Cette modification a été validée par le service DPS.

### <a name="rolling-back-the-changes-after-using-dps"></a>Restauration des modifications après utilisation de DPS. 

Si vous souhaitez restaurer les modifications, réapprovisionnez les appareils du nouveau hub vers l’ancien.

Vous avez maintenant terminé la migration de votre hub et de ses appareils. Vous pouvez passer à [Nettoyage](#clean-up).

## <a name="using-import-export-to-move-the-devices-to-the-new-hub"></a>Utilisation de l’option Import-Export pour déplacer les appareils vers le nouveau hub

L’application cible .NET Core, ce qui vous permet de l’exécuter sur Windows ou Linux. Vous pouvez télécharger l’exemple, récupérer vos chaînes de connexion, définir les indicateurs pour lesquels vous souhaitez exécuter, puis l’exécuter. Vous pouvez le faire sans jamais ouvrir le code.

### <a name="downloading-the-sample"></a>Téléchargement de l’exemple

1. Utilisez les exemples C# IoT à partir de cette page : [Exemples Azure IoT pour C#](https://azure.microsoft.com/resources/samples/azure-iot-samples-csharp/). Téléchargez le fichier zip et décompressez-le sur votre ordinateur. 

1. Le code pertinent se trouve dans ./iot-hub/Samples/service/ImportExportDevicesSample. Vous n’avez pas besoin d’afficher ni de modifier le code pour exécuter l’application.

1. Pour exécuter l’application, spécifiez trois chaînes de connexion et cinq options. Vous transmettez ces données en tant qu’arguments de ligne de commande, utilisez des variables d’environnement ou utilisez une combinaison des deux. Nous allons passer les options dans en tant qu’arguments de ligne de commande et les chaînes de connexion en tant que variables d’environnement. 

   Cela est dû au fait que les chaînes de connexion sont longues et incomplètes et peu susceptibles de changer, mais vous pouvez modifier les options et exécuter l’application plusieurs fois. Pour modifier la valeur d’une variable d’environnement, vous devez fermer la fenêtre de commande et Visual Studio ou VS Code, selon votre utilisation. 

### <a name="options"></a>Options

Voici les cinq options que vous spécifiez lorsque vous exécutez l’application. Nous allons les placer sur la ligne de commande en une minute.

*   **addDevices** (argument 1) : définissez ce paramètre sur true si vous souhaitez ajouter des périphériques virtuels qui sont générés pour vous. Ils sont ajoutés au hub source. Par ailleurs, définissez **numToAdd** (argument 2) pour spécifier le nombre d’appareils que vous souhaitez ajouter. Le nombre maximal d’appareils que vous pouvez inscrire auprès d’un hub est de 1 million. L’objectif de cette option est à des fins de test : vous pouvez générer un nombre spécifique d’appareils, puis les copier sur un autre hub.

*   **copyDevices** (argument 3) : définissez cette valeur sur true pour copier les appareils d’un hub à un autre. 

*   **deleteSourceDevices** (argument 4) : définissez cette valeur sur true pour supprimer tous les appareils inscrits auprès du hub source. Nous vous recommandons de patienter jusqu’à ce que vous soyez certain que tous les appareils ont été transférés avant l’exécution. Une fois que vous avez supprimé les appareils, vous ne pouvez pas les récupérer.

*   **deleteSourceDevices** (argument 5) : définissez cette valeur sur true pour supprimer tous les appareils inscrits auprès du hub de destination (le clone). Vous souhaiterez peut-être le faire si vous souhaitez copier plusieurs fois les appareils. 

La commande de base sera *dotnet run* : cela indique à .NET de générer le fichier csproj local, puis de l’exécuter. Vous ajoutez vos arguments de ligne de commande à la fin, avant de l’exécuter. 

La ligne de commande ressemblera à ces exemples :

``` console 
    // Format: dotnet run add-devices num-to-add copy-devices delete-source-devices delete-destination-devices

    // Add 1000 devices, don't copy them to the other hub, or delete them. 
    // The first argument is true, numToAdd is 50, and the other arguments are false.
    dotnet run true 1000 false false false 

    // Copy the devices you just added to the other hub; don't delete anything.
    // The first argument is false, numToAdd is 0, copy-devices is true, and the delete arguments are both false
    dotnet run false 0 true false false 
```

### <a name="using-environment-variables-for-the-connection-strings"></a>Utilisation des variables d’environnement pour les chaînes de connexion

1. Pour exécuter l’exemple, vous avez besoin des chaînes de connexion aux anciens et nouveaux hubs IoT, ainsi qu’à un compte de stockage que vous pouvez utiliser pour les fichiers de travail temporaires. Nous allons stocker les valeurs pour ces variables d’environnement.

1. Pour obtenir les valeurs de chaîne de connexion; connectez-vous au [Portail Azure](https://portal.azure.com). 

1. Placez les chaînes de connexion dans un endroit où vous pouvez les récupérer, par exemple le bloc-notes. Si vous copiez les éléments suivants, vous pouvez coller les chaînes de connexion directement là où elles se trouvent. N’ajoutez pas d’espaces autour du signe égal ou cela modifie le nom de la variable. En outre, vous n’avez pas besoin de guillemets doubles autour des chaînes de connexion. Si vous placez des guillemets autour de la chaîne de connexion du compte de stockage, cela ne fonctionnera pas.

   Pour Windows, il s’agit de la façon de définir les variables d’environnement :

   ``` console  
   SET IOTHUB_CONN_STRING=<put connection string to original IoT Hub here>
   SET DEST_IOTHUB_CONN_STRING=<put connection string to destination or clone IoT Hub here>
   SET STORAGE_ACCT_CONN_STRING=<put connection string to the storage account here>
   ```
 
   Pour Linux, il s’agit de la façon de définir les variables d’environnement :

   ``` console  
   export IOTHUB_CONN_STRING="<put connection string to original IoT Hub here>"
   export DEST_IOTHUB_CONN_STRING="<put connection string to destination or clone IoT Hub here>"
   export STORAGE_ACCT_CONN_STRING="<put connection string to the storage account here>"
   ```

1. Pour les chaînes de connexion du hub IoT, accédez à chaque hub dans le portail. Vous pouvez rechercher dans **Ressources** pour le hub. Si vous connaissez le groupe de ressources, vous pouvez accéder aux **Groupes de ressources**, sélectionner votre groupe de ressources, puis sélectionner le hub dans la liste des ressources de ce groupe de ressources. 

1. Sélectionnez **Stratégies d’accès partagé** à partir de Paramètres du hub, puis sélectionnez **iothubowner** et copiez l’une des chaînes de connexion. Procédez de la même façon pour le hub de destination. Ajoutez-les aux commandes SET appropriées.

1. Pour la chaîne de connexion du compte de stockage, recherchez le compte de stockage dans **Ressources** ou sous son **Groupe de ressources** et ouvrez-le. 
   
1. Dans la section Paramètres, sélectionnez **Clés d’accès** et copiez l’une des chaînes de connexion. Placez la chaîne de connexion dans votre fichier texte pour la commande SET appropriée. 

Vous avez maintenant les variables d’environnement dans un fichier avec les commandes SET et vous savez ce que sont vos arguments de ligne de commande. Exécutons l’exemple.

### <a name="running-the-sample-application-and-using-command-line-arguments"></a>Exécution de l’exemple d’application et utilisation des arguments de ligne de commande

1. Ouvrir une fenêtre d’invite de commandes. Sélectionnez Windows et saisissez `command prompt` pour accéder à la fenêtre d’invite de commandes.

1. Copiez les commandes qui définissent les variables d’environnement, une à la fois, puis collez-les dans la fenêtre d’invite de commandes et sélectionnez Entrée. Lorsque vous avez terminé, saisissez `SET` dans la fenêtre d’invite de commandes pour afficher vos variables d’environnement et leurs valeurs. Une fois que vous les avez copiées dans la fenêtre d’invite de commandes, vous n’êtes pas obligé de les copier à nouveau, sauf si vous ouvrez une nouvelle fenêtre d’invite de commandes.

1. Dans la fenêtre d’invite de commandes, modifiez les répertoires jusqu’à ce que vous soyez dans ./ImportExportDevicesSample (où se trouve le fichier ImportExportDevicesSample.csproj). Saisissez ensuite ce qui suit et incluez vos arguments de ligne de commande.

    ``` console
    // Format: dotnet run add-devices num-to-add copy-devices delete-source-devices delete-destination-devices
    dotnet run arg1 arg2 arg3 arg4 arg5
    ```

    La commande dotnet génère et exécute l’application. Étant donné que vous passez les options lors de l’exécution de l’application, vous pouvez modifier leurs valeurs chaque fois que vous exécutez l’application. Par exemple, vous souhaiterez peut-être l’exécuter une fois et créer de nouveaux appareils, puis l’exécuter à nouveau et copier ces appareils sur un nouveau hub, et ainsi de suite. Vous pouvez également suivre toutes les étapes de la même exécution, même si nous vous recommandons de ne pas supprimer les appareils tant que vous n’êtes pas certain de la fin du clonage. Voici un exemple qui crée 1 000 appareils, puis les copie sur l’autre hub.

    ``` console
    // Format: dotnet run add-devices num-to-add copy-devices delete-source-devices delete-destination-devices

    // Add 1000 devices, don't copy them to the other hub or delete them. 
    dotnet run true 1000 false false false 

    // Do not add any devices. Copy the ones you just created to the other hub; don't delete anything.
    dotnet run false 0 true false false 
    ```

    Une fois que vous avez vérifié que les appareils ont été correctement copiés, vous pouvez supprimer les appareils du hub source comme suit :

   ``` console
   // Format: dotnet run add-devices num-to-add copy-devices delete-source-devices delete-destination-devices
   // Delete the devices from the source hub.
   dotnet run false 0 false true false 
   ```

### <a name="running-the-sample-application-using-visual-studio"></a>Exécution de l’exemple d’application à l’aide de Visual Studio

1. Si vous souhaitez exécuter l’application dans Visual Studio, remplacez votre répertoire actif par le dossier dans lequel réside le fichier IoTHubServiceSamples.sln. Exécutez ensuite cette commande dans la fenêtre d’invite de commandes pour ouvrir la solution dans Visual Studio. Vous devez le faire dans la même fenêtre de commande que celle où vous définissez les variables d’environnement, afin que ces variables soient connues.

   ``` console       
   IoTHubServiceSamples.sln
   ```
    
1. Cliquez avec le bouton de droite sur *ImportExportDevicesSample* et sélectionnez **Définir comme projet de démarrage**.    
    
1. Définissez les variables en haut de Program.cs dans le dossier ImportExportDevicesSample pour les cinq options.

   ``` csharp
   // Add randomly created devices to the source hub.
   private static bool addDevices = true;
   //If you ask to add devices, this will be the number added.
   private static int numToAdd = 0; 
   // Copy the devices from the source hub to the destination hub.
   private static bool copyDevices = false;
   // Delete all of the devices from the source hub. (It uses the IoTHubConnectionString).
   private static bool deleteSourceDevices = false;
   // Delete all of the devices from the destination hub. (Uses the DestIotHubConnectionString).
   private static bool deleteDestDevices = false;
   ```

1. Sélectionnez F5 pour exécuter l'application. Une fois l’exécution terminée, vous pouvez afficher les résultats.

### <a name="view-the-results"></a>View the results 

Vous pouvez afficher les appareils dans le [Portail Azure](https://portal.azure.com) et vérifier qu’ils se trouvent dans le nouvel emplacement.

1. Accédez au nouveau hub à l’aide du [Portail Azure](https://portal.azure.com). Sélectionnez votre hub, puis sélectionnez **Appareils IoT**. Les appareils que vous venez de copier de l’ancien hub vers le hub cloné s’affichent. Vous pouvez également afficher les propriétés du hub cloné. 

1. Recherchez les erreurs d’importation/exportation en accédant au compte de stockage Azure dans le [Portail Azure](https://portal.azure.com) et en recherchant dans le conteneur `devicefiles` le `ImportErrors.log`. Si ce fichier est vide (la taille est 0), aucune erreur n’est survenue. Si vous essayez d’importer le même appareil plusieurs fois, il rejette l’appareil la deuxième fois et ajoute un message d’erreur dans le fichier journal.

### <a name="committing-the-changes"></a>Validation des modifications 

À ce stade, vous avez copié votre hub sur le nouvel emplacement et migré les appareils vers le nouveau clone. À présent, vous devez apporter des modifications afin que les appareils fonctionnent avec le hub cloné.

Pour valider les modifications, voici les étapes que vous devez effectuer : 

* Mettez à jour chaque appareil pour modifier le nom d’hôte IoT Hub afin que celui-ci pointe vers le nouveau hub. Pour ce faire, vous devez utiliser la même méthode que celle que vous avez utilisée lors du premier approvisionnement de l’appareil.

* Modifiez les applications dont vous disposez, qui font référence à l’ancien hub pour pointer vers le nouveau hub.

* Une fois que vous avez terminé, le nouveau hub doit être opérationnel. L’ancien hub ne doit pas avoir d’appareils actifs ni se trouver dans un état déconnecté. 

### <a name="rolling-back-the-changes"></a>Restauration des modifications

Si vous décidez de restaurer les modifications, voici les étapes à suivre :

* Mettez à jour chaque appareil pour modifier le nom d’hôte IoT Hub afin que celui-ci pointe vers l’ancien hub. Pour ce faire, vous devez utiliser la même méthode que celle que vous avez utilisée lors du premier approvisionnement de l’appareil.

* Modifiez les applications dont vous disposez, qui font référence au nouveau hub pour pointer vers l’ancien hub. Par exemple, si vous utilisez Azure Analytics, vous devrez peut-être reconfigurer votre [entrée Azure Stream Analytics](../stream-analytics/stream-analytics-define-inputs.md#stream-data-from-iot-hub).

* Supprimez le nouveau hub. 

* Si vous avez des ressources de routage, la configuration de l’ancien hub doit toujours pointer vers la configuration du routage correcte et doit fonctionner avec ces ressources une fois le hub redémarré.

### <a name="checking-the-results"></a>Vérification des résultats 

Pour vérifier les résultats, modifiez votre solution IoT pour qu’elle pointe vers votre hub sur le nouvel emplacement et exécutez-la. En d’autres termes, effectuez les mêmes actions avec le nouveau hub que vous avez effectué avec le hub précédent et assurez-vous qu’ils fonctionnent correctement. 

Si vous avez implémenté le routage, testez et assurez-vous que vos messages sont acheminés correctement vers les ressources.

## <a name="clean-up"></a>Nettoyage

Ne nettoyez pas tant que vous n’êtes pas certain que le nouveau hub est en cours d’exécution et que les appareils fonctionnent correctement. Veillez également à tester le routage si vous utilisez cette fonctionnalité. Lorsque vous êtes prêt, nettoyez les anciennes ressources en effectuant les étapes suivantes :

* Si vous ne l’avez pas encore fait, supprimez l’ancien hub. Cela supprime tous les appareils actifs du hub.

* Si vous avez déplacé les ressources de routage vers le nouvel emplacement, vous pouvez supprimer les anciennes ressources de routage.

## <a name="next-steps"></a>Étapes suivantes

Vous avez cloné un hub IoT dans un nouveau hub d’une nouvelle région, terminez avec les appareils. Pour plus d’informations sur l’exécution d’opérations en bloc sur le registre des identités dans un IoT Hub, consultez [Importer et exporter des identités d’appareil IoT Hub en bloc](iot-hub-bulk-identity-mgmt.md).

Pour plus d’informations sur IoT Hub et le développement du hub, consultez les articles suivants.

* [Guide du développeur IoT Hub](iot-hub-devguide.md)

* [Didacticiel de routage IoT Hub](tutorial-routing.md)

* [Aperçu de gestion des appareils IoT Hub](iot-hub-device-management-overview.md)

* Si vous souhaitez déployer l’exemple d’application, consultez [Déploiement d’applications .NET Core](/dotnet/core/deploying/index).