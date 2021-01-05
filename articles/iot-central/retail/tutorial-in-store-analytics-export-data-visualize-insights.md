---
title: Tutoriel - Exporter des données et visualiser des insights dans Azure IoT Central
description: Dans ce didacticiel, découvrez comment exporter des données à partir de IoT Central et visualiser des insights dans un tableau de bord Power BI.
services: iot-central
ms.service: iot-central
ms.subservice: iot-central-retail
ms.topic: tutorial
ms.custom:
- iot-storeAnalytics-checkout
- iot-p0-scenario
ms.author: dobett
author: dominicbetts
ms.date: 11/12/2019
ms.openlocfilehash: fca9a72fa237a11a02b6d4fbe30ebba5857384cf
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97346365"
---
# <a name="tutorial-export-data-from-azure-iot-central-and-visualize-insights-in-power-bi"></a>Tutoriel : Exporter des données à partir d’Azure IoT Central et visualiser des insights dans Power BI


Dans les deux didacticiels précédents, vous avez créé et personnalisé une application IoT Central à l’aide du modèle d’application **in-Store Analytics-Checkout**. Dans ce didacticiel, vous configurez votre application IoT Central pour exporter les données de télémétrie collectées à partir des appareils. Vous utilisez ensuite Power BI pour créer un tableau de bord personnalisé pour le responsable du magasin afin de visualiser les insights dérivés des données de télémétrie.

Dans ce didacticiel, vous apprendrez à :
> [!div class="checklist"]
> * Configurer une application IoT Central pour exporter des données de télémétrie vers un Event Hub.
> * Utiliser Logic Apps pour envoyer des données d’un Event Hub vers un jeu de données de streaming Power BI.
> * Créer un tableau de bord Power BI pour visualiser les données dans le jeu de données de streaming.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous avez besoin des éléments suivants :

* Pour terminer les deux didacticiels précédents, [Créer une application Analytique dans le magasin dans Azure IoT Central](./tutorial-in-store-analytics-create-app.md) et [Personnaliser le tableau de bord de l’opérateur et gérer les appareils dans Azure IoT Central](./tutorial-in-store-analytics-customize-dashboard.md).
* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.
* Un compte Microsoft Power BI. Si vous n’avez pas de compte Power BI, inscrivez-vous pour bénéficier d’une version d’évaluation gratuite de [Power BI Pro](https://app.powerbi.com/signupredirect?pbi_source=web) avant de commencer.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Avant de créer votre Event Hub et votre application logique, vous devez créer un groupe de ressources pour les gérer. Le groupe de ressources doit se trouver au même emplacement que votre application IoT Central **Analytique dans le magasin - Paiement**. Pour créer un groupe de ressources :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans le volet de navigation de gauche, sélectionnez **Groupes de ressources**. Sélectionnez ensuite **Ajouter**.
1. Sous **Abonnement**, sélectionnez le nom de l’abonnement Azure que vous avez utilisé pour créer votre application IoT Central.
1. Pour le nom du **groupe de ressources**, entrez _retail-store-analysis_*.
1. Pour la **région**, sélectionnez la région que vous avez choisie pour l’application IoT Central.
1. Sélectionnez **Vérifier + créer**.
1. Dans la page **Vérifier + créer**, sélectionnez **Créer**.

Vous disposez maintenant d’un groupe de ressources appelé **retail-store-analysis** dans votre abonnement.

## <a name="create-an-event-hub"></a>Créer un hub d’événements

Avant de pouvoir configurer l’application de surveillance de la vente au détail pour exporter les données de télémétrie, vous devez créer un Event Hub pour recevoir les données exportées. Les étapes suivantes détaillent la création de votre Event Hub :

1. Dans le portail Azure, sélectionnez **Créer une ressource** en haut à gauche de l’écran.
1. Dans **Rechercher dans la Place de marché**, entrez _Event Hubs_, puis appuyez sur **Entrée**.
1. Dans la page **Event Hubs**, sélectionnez **Créer**.
1. Dans la page **Créer un espace de noms**, effectuez les étapes suivantes :
    * Entrez un nom unique pour l’espace de noms, par exemple _votrenom-Retail-Store-Analysis_. Le système vérifie si le nom est disponible.
    * Choisir le niveau tarifaire **De base**.
    * Sélectionnez le même **abonnement** que vous avez utilisé pour créer votre application IoT Central.
    * Sélectionnez le groupe de ressources **retail-store-analysis**.
    * Sélectionnez le même emplacement que vous avez utilisé pour votre application IoT Central.
    * Sélectionnez **Create** (Créer). Vous devrez peut-être attendre quelques minutes pour que le système approvisionne les ressources.
1. Dans le portail, accédez au groupe de ressources **retail-store-analysis**. Attendez la fin du déploiement. Vous devrez peut-être sélectionner **Actualiser** pour mettre à jour l’état du déploiement. Vous pouvez également vérifier l’état de la création de l’espace de noms Event Hub dans les **notifications**.
1. Dans le groupe de ressources **retail-store-analysis**, sélectionnez **l’espace de noms Event Hubs**. La page d’accueil de votre **espace de noms Event Hubs** apparaît dans le portail.

Maintenant que vous disposez d’un espace de noms **Event Hubs**, vous pouvez créer un **Event Hub** à utiliser avec votre application IoT Central :

1. Sur la page d’accueil de votre **espace de noms Event Hubs** dans le portail, sélectionnez **+ Event Hub**.
1. Sur la page **Créer un Event Hub**, entrez _store-telemetry_ comme nom, puis sélectionnez **Créer**.

Vous disposez maintenant d’un Event Hub que vous pouvez utiliser lorsque vous configurez l’exportation de données à partir de votre application IoT Central :

![Event Hub](./media/tutorial-in-store-analytics-visualize-insights/event-hub.png)

## <a name="configure-data-export"></a>Configurer une exportation de données

Maintenant que vous avez un Event Hub, vous pouvez configurer votre application **Analytique dans le magasin - Paiement** pour exporter les données de télémétrie à partir des appareils connectés. Les étapes suivantes montrent comment configurer l’exportation :

1. Connectez-vous à votre application IoT Central **Analytique dans le magasin - paiement**.
1. Dans le menu de gauche, sélectionnez **Exportation de données**.
1. Sélectionnez **Nouveau > Azure Event Hubs**.
1. Entrez _Exportation des données de télémétrie_ comme **Nom d’affichage**.
1. Sélectionnez votre **espace de noms Event Hubs**.
1. Sélectionnez l’Event Hub **store-telemetry**.
1. Désactivez **Appareils** et **Modèles d’appareil** dans la section **Données à exporter**.
1. Sélectionnez **Enregistrer**.

L’exportation de données peut prendre quelques minutes pour commencer à envoyer des données de télémétrie à votre Event Hub. Vous pouvez voir l’état de l’exportation sur la page **Exportations de données** :

![Configuration de l’exportation de données continue](./media/tutorial-in-store-analytics-visualize-insights/export-configuration.png)

## <a name="create-the-power-bi-datasets"></a>Créer les jeux de données Power BI

Votre tableau de bord Power BI affiche des données de votre application de surveillance de la vente au détail. Dans cette solution, vous utilisez des jeux de données de streaming Power BI comme source de données pour le tableau de bord Power BI. Dans cette section, vous allez définir le schéma des jeux de données de streaming afin que l’application logique puisse transférer des données à partir de l’Event Hub. Les étapes suivantes vous montrent comment créer deux jeux de données de streaming pour les capteurs environnementaux et un jeu de données de diffusion en continu pour le capteur d’occupation :

1. Connectez-vous à votre compte **Power BI**.
1. Sélectionnez **Espaces de travail**, puis sélectionnez **Créer un espace de travail**.
1. Dans la page **Créer un espace de travail**, entrez _Analytique dans le magasin - paiement_ comme **nom de l’espace de travail**.
1. Faites défiler l’écran jusqu’au bas de la page **Bienvenue dans l’analytique dans le magasin - paiement**, puis sélectionnez **Ignorer**.
1. Sur la page espace de travail, sélectionnez **Créer > Jeu de données de streaming**.
1. Sur la page **nouveau jeu de données de streaming**, choisissez **API**, puis sélectionnez **Suivant**.
1. Entrez _Détecteur de zone 1_ comme **nom de jeu de données**.
1. Entrez les trois **valeurs provenant du flux** dans le tableau suivant :

    | Nom de la valeur  | Type de valeur |
    | ----------- | ---------- |
    | Timestamp   | DateTime   |
    | Humidité    | Number     |
    | Température | Number     |

1. Activez **Analyse des données d’historique**.
1. Sélectionnez **Créer** puis **Terminé**.
1. Créez un autre jeu de données de streaming appelé **Détecteur de zone 2** avec le même schéma et les mêmes paramètres que le jeu de données de streaming de **Détecteur de zone 1**.

Vous avez maintenant deux jeux de données de streaming. L’application logique achemine les données de télémétrie à partir des deux capteurs environnementaux connectés à votre application **Analytique dans le magasin - paiement** à ces deux jeux de données :

![Jeux de données de zone](./media/tutorial-in-store-analytics-visualize-insights/dataset-1.png)

Cette solution utilise un jeu de données de streaming pour chaque capteur, car il n’est pas possible d’appliquer des filtres pour la diffusion en continu des données dans Power BI.

Vous avez également besoin d’un jeu de données de streaming pour la télémétrie d’occupation :

1. Sur la page espace de travail, sélectionnez **Créer > Jeu de données de streaming**.
1. Sur la page **nouveau jeu de données de streaming**, choisissez **API**, puis sélectionnez **Suivant**.
1. Entrez _Capteur d’occupation_ comme **nom de jeu de données**.
1. Entrez les cinq **valeurs provenant du flux** dans le tableau suivant :

    | Nom de la valeur     | Type de valeur |
    | -------------- | ---------- |
    | Timestamp      | DateTime   |
    | Longueur de file d’attente 1 | Number     |
    | Longueur de file d’attente 2 | Number     |
    | Durée de maintien 1   | Number     |
    | Durée de maintien 2   | Number     |

1. Activez **Analyse des données d’historique**.
1. Sélectionnez **Créer** puis **Terminé**.

Vous disposez maintenant d’un troisième jeu de données de streaming qui stocke les valeurs du capteur d’occupation simulé. Ce capteur signale la longueur de la file d’attente lors des deux extractions du magasin, ainsi que la durée d’attente des clients dans ces files d’attente :

![Jeu de données d’occupation](./media/tutorial-in-store-analytics-visualize-insights/dataset-2.png)

## <a name="create-a-logic-app"></a>Créer une application logique

Dans cette solution, l’application logique lit les données de télémétrie à partir de l’Event Hub, les analyse, puis les envoie aux jeux de données de streaming Power BI que vous avez créés.

Avant de créer l’application logique, vous avez besoin des ID d’appareil des deux capteurs RuuviTag que vous avez connectés à votre application IoT Central dans le didacticiel [Créer une application d’analytique en magasin dans Azure IoT Central](./tutorial-in-store-analytics-create-app.md) :

1. Connectez-vous à votre application IoT Central **Analytique dans le magasin - paiement**.
1. Sélectionnez **Appareils** dans le panneau de gauche. Puis sélectionnez **RuuviTag**.
1. Prenez note des **ID de l’appareil**. Dans la capture d’écran suivante, les ID sont **f5dcf4ac32e8** et **e29ffc8d5326** :

    ![ID des appareils](./media/tutorial-in-store-analytics-visualize-insights/device-ids.png)

Les étapes suivantes montrent comment créer l’application logique à partir du portail Azure :

1. Connectez-vous au [portail Azure](https://portal.azure.com), sélectionnez **Créer une ressource** en haut à gauche de l’écran.
1. Dans **Rechercher dans la Place de marché**, entrez _Logic App_, puis appuyez sur **Entrée**.
1. Dans la page **Application logique**, sélectionnez **Créer**.
1. Sur la page de création d’**application logique** :
    * Entrez un nom unique pour votre application logique, par exemple _votrenom-Retail-Store-Analysis_.
    * Sélectionnez le même **abonnement** que vous avez utilisé pour créer votre application IoT Central.
    * Sélectionnez le groupe de ressources **retail-store-analysis**.
    * Sélectionnez le même emplacement que vous avez utilisé pour votre application IoT Central.
    * Sélectionnez **Create** (Créer). Vous devrez peut-être attendre quelques minutes pour que le système approvisionne les ressources.
1. Dans le portail Azure, accédez à votre nouvelle application logique.
1. Dans la page **Concepteur Logic Apps**, faites défiler la page et sélectionnez **Application logique vide**.
1. Dans **Rechercher parmi les connecteurs et les déclencheurs**, entrez _Event Hubs_.
1. Dans **Déclencheurs**, sélectionnez **Lorsque les événements sont disponibles dans un hub Event Hub**.
1. Entrez _Stocker les données de télémétrie_ comme le **nom de connexion**, puis sélectionnez votre **espace de noms Event Hubs**.
1. Sélectionnez la stratégie **RootManageSharedAccess**, puis sélectionnez **Créer**.
1. Dans l’action **Lorsque les événements sont disponibles dans un hub Event Hubs** :
    * Dans **Nom de l’Event Hub**, sélectionnez **store-telemetry**.
    * Dans **Type de contenu**, sélectionnez **application/json**.
    * Définissez l’**intervalle** sur trois et la **fréquence** sur secondes
1. Sélectionnez **Enregistrer** pour enregistrer votre application logique.

Pour ajouter la logique à la conception de votre application logique, sélectionnez **mode Code** :

1. Remplacez `"actions": {},` par le code JSON suivant. Remplacez les deux espaces réservés `[YOUR RUUVITAG DEVICE ID 1]` et `[YOUR RUUVITAG DEVICE ID 2]` par les ID que vous avez notés pour vos deux appareils RuuviTag :

    ```json
    "actions": {
        "Initialize_Device_ID_variable": {
            "inputs": {
                "variables": [
                    {
                        "name": "DeviceID",
                        "type": "String"
                    }
                ]
            },
            "runAfter": {},
            "type": "InitializeVariable"
        },
        "Initialize_Interface_ID_variable": {
            "inputs": {
                "variables": [
                    {
                        "name": "InterfaceID",
                        "type": "String",
                        "value": "Other"
                    }
                ]
            },
            "runAfter": {
                "Initialize_Device_ID_variable": [
                    "Succeeded"
                ]
            },
            "type": "InitializeVariable"
        },
        "Parse_Properties": {
            "inputs": {
                "content": "@triggerBody()?['Properties']",
                "schema": {
                    "properties": {
                        "iothub-connection-auth-generation-id": {
                            "type": "string"
                        },
                        "iothub-connection-auth-method": {
                            "type": "string"
                        },
                        "iothub-connection-device-id": {
                            "type": "string"
                        },
                        "iothub-enqueuedtime": {
                            "type": "string"
                        },
                        "iothub-interface-name": {
                            "type": "string"
                        },
                        "iothub-message-source": {
                            "type": "string"
                        },
                        "x-opt-enqueued-time": {
                            "type": "string"
                        },
                        "x-opt-offset": {
                            "type": "string"
                        },
                        "x-opt-sequence-number": {
                            "type": "integer"
                        }
                    },
                    "type": "object"
                }
            },
            "runAfter": {
                "Initialize_Interface_ID_variable": [
                    "Succeeded"
                ]
            },
            "type": "ParseJson"
        },
        "Parse_Telemetry": {
            "inputs": {
                "content": "@triggerBody()?['ContentData']",
                "schema": {
                    "properties": {
                        "DwellTime1": {
                            "type": "number"
                        },
                        "DwellTime2": {
                            "type": "number"
                        },
                        "count1": {
                            "type": "number"
                        },
                        "count2": {
                            "type": "number"
                        },
                        "humidity": {
                            "type": "number"
                        },
                        "temperature": {
                            "type": "number"
                        }
                    },
                    "type": "object"
                }
            },
            "runAfter": {
                "Initialize_Interface_ID_variable": [
                    "Succeeded"
                ]
            },
            "type": "ParseJson"
        },
        "Set_Device_ID_variable": {
            "inputs": {
                "name": "DeviceID",
                "value": "@body('Parse_Properties')?['iothub-connection-device-id']"
            },
            "runAfter": {
                "Parse_Properties": [
                    "Succeeded"
                ]
            },
            "type": "SetVariable"
        },
        "Set_Interface_ID_variable": {
            "inputs": {
                "name": "InterfaceID",
                "value": "@body('Parse_Properties')?['iothub-interface-name']"
            },
            "runAfter": {
                "Set_Device_ID_variable": [
                    "Succeeded"
                ]
            },
            "type": "SetVariable"
        },
        "Switch_by_DeviceID": {
            "cases": {
                "Occupancy": {
                    "actions": {
                        "Switch_by_InterfaceID": {
                            "cases": {
                                "Dwell_Time_interface": {
                                    "actions": {},
                                    "case": "RS40_Occupancy_Sensor_v2_1l0"
                                },
                                "People_Count_interface": {
                                    "actions": {},
                                    "case": "RS40_Occupancy_Sensor_iv"
                                }
                            },
                            "default": {
                                "actions": {}
                            },
                            "expression": "@variables('InterfaceID')",
                            "runAfter": {},
                            "type": "Switch"
                        }
                    },
                    "case": "Occupancy"
                },
                "Zone 2 environment": {
                    "actions": {},
                    "case": "[YOUR RUUVITAG DEVICE ID 2]"
                },
                "Zone_1_environment": {
                    "actions": {},
                    "case": "[YOUR RUUVITAG DEVICE ID 1]"
                }
            },
            "default": {
                "actions": {}
            },
            "expression": "@variables('DeviceID')",
            "runAfter": {
                "Parse_Telemetry": [
                    "Succeeded"
                ],
                "Set_Interface_ID_variable": [
                    "Succeeded"
                ]
            },
            "type": "Switch"
        }
    },
    ```

1. Sélectionnez **Enregistrer** puis sélectionnez **Concepteur** pour afficher la version visuelle de la logique que vous avez ajoutée :

    ![Conception d’application logique](./media/tutorial-in-store-analytics-visualize-insights/logic-app.png)

1. Sélectionnez **Basculer par DeviceID** pour développer l’action. Sélectionnez ensuite **Environnement de zone 1**, puis sélectionnez **Ajouter une action**.
1. Dans **Rechercher parmi les actions et les connecteurs**, entrez **Power BI**, puis appuyez sur **Entrer**.
1. Sélectionnez l’action **Ajouter des lignes à un jeu de données (préversion)** .
1. Sélectionnez **Se connecter** et suivez les invites pour vous connecter à votre compte Power BI.
1. Une fois le processus de connexion terminé, dans l’action **Ajouter des lignes à un jeu de données** :
    * Sélectionnez **Analyses dans le magasin - paiement** en tant qu’espace de travail.
    * Sélectionnez **Détecteur de zone 1** comme jeu de données.
    * Sélectionnez **RealTimeData** comme table.
    * Sélectionnez **Ajouter un nouveau paramètre** puis sélectionnez les champs **Horodateur**, **Humidité** et **Température**.
    * Sélectionnez le champ **Horodateur**, puis sélectionnez **x-opt-enqueuedtime** dans la liste **Contenu dynamique**.
    * Sélectionnez le champ **Humidité**, puis sélectionnez **Afficher plus** près de **Analyser la télémétrie**. Sélectionnez ensuite **Humidité**.
    * Sélectionnez le champ **Température**, puis sélectionnez **Afficher plus** près de **Analyser la télémétrie**. Sélectionnez ensuite **Température**.
    * Cliquez sur **Enregistrer** pour enregistrer vos modifications. L’action **Environnement de zone 1** ressemble à la capture d’écran suivante : ![Environnement de zone 1](./media/tutorial-in-store-analytics-visualize-insights/zone-1-action.png)
1. Sélectionnez l’action **Environnement de zone 2**, puis sélectionnez **Ajouter une action**.
1. Dans **Rechercher parmi les actions et les connecteurs**, entrez **Power BI**, puis appuyez sur **Entrer**.
1. Sélectionnez l’action **Ajouter des lignes à un jeu de données (préversion)** .
1. Dans l’action **Ajouter des lignes à un jeu de données 2** :
    * Sélectionnez **Analyses dans le magasin - paiement** en tant qu’espace de travail.
    * Sélectionnez **Détecteur de zone 2** comme jeu de données.
    * Sélectionnez **RealTimeData** comme table.
    * Sélectionnez **Ajouter un nouveau paramètre** puis sélectionnez les champs **Horodateur**, **Humidité** et **Température**.
    * Sélectionnez le champ **Horodateur**, puis sélectionnez **x-opt-enqueuedtime** dans la liste **Contenu dynamique**.
    * Sélectionnez le champ **Humidité**, puis sélectionnez **Afficher plus** près de **Analyser la télémétrie**. Sélectionnez ensuite **Humidité**.
    * Sélectionnez le champ **Température**, puis sélectionnez **Afficher plus** près de **Analyser la télémétrie**. Sélectionnez ensuite **Température**.
    Cliquez sur **Enregistrer** pour enregistrer vos modifications.  L’action **Environnement de zone 2** ressemble à la capture d’écran suivante : ![Environnement de zone 2](./media/tutorial-in-store-analytics-visualize-insights/zone-2-action.png)
1. Sélectionnez l’action **Occupation**, puis sélectionnez l’action **Basculer par ID d’interface**.
1. Sélectionnez l’action **Interface de durée de maintien**, puis sélectionnez **Ajouter une action**.
1. Dans **Rechercher parmi les actions et les connecteurs**, entrez **Power BI**, puis appuyez sur **Entrer**.
1. Sélectionnez l’action **Ajouter des lignes à un jeu de données (préversion)** .
1. Dans l’action **Ajouter des lignes à un jeu de données** :
    * Sélectionnez **Analyses dans le magasin - paiement** en tant qu’espace de travail.
    * Sélectionnez **Capteur d’occupation** comme jeu de données.
    * Sélectionnez **RealTimeData** comme table.
    * Sélectionnez **Ajouter un nouveau paramètre** puis sélectionnez les champs **Horodateur**, **Durée de maintien 1** et **Durée de maintien 2**.
    * Sélectionnez le champ **Horodateur**, puis sélectionnez **x-opt-enqueuedtime** dans la liste **Contenu dynamique**.
    * Sélectionnez le champ **Durée de maintien 1**, puis sélectionnez **Afficher plus** près de **Analyser la télémétrie**. Sélectionnez ensuite **Durée de maintien 1**.
    * Sélectionnez le champ **Durée de maintien 2**, puis sélectionnez **Afficher plus** près de **Analyser la télémétrie**. Sélectionnez ensuite **Durée de maintien 2**.
    * Cliquez sur **Enregistrer** pour enregistrer vos modifications. L’action **Interface de durée de maintien** ressemble à la capture d’écran suivante : ![Capture d’écran illustrant l’action « Interface de durée de maintien ».](./media/tutorial-in-store-analytics-visualize-insights/occupancy-action-1.png)
1. Sélectionnez l’action **Interface du nombre de personnes**, puis sélectionnez **Ajouter une action**.
1. Dans **Rechercher parmi les actions et les connecteurs**, entrez **Power BI**, puis appuyez sur **Entrer**.
1. Sélectionnez l’action **Ajouter des lignes à un jeu de données (préversion)** .
1. Dans l’action **Ajouter des lignes à un jeu de données** :
    * Sélectionnez **Analyses dans le magasin - paiement** en tant qu’espace de travail.
    * Sélectionnez **Capteur d’occupation** comme jeu de données.
    * Sélectionnez **RealTimeData** comme table.
    * Sélectionnez **Ajouter un nouveau paramètre** puis sélectionnez les champs **Horodateur**, **Longueur de file d’attente 1** et **Longueur de file d’attente 2**.
    * Sélectionnez le champ **Horodateur**, puis sélectionnez **x-opt-enqueuedtime** dans la liste **Contenu dynamique**.
    * Sélectionnez le champ **Longueur de file d’attente 1**, puis sélectionnez **Afficher plus** près de **Analyser la télémétrie**. Sélectionnez ensuite **count1**.
    * Sélectionnez le champ **Longueur de file d’attente 2**, puis sélectionnez **Afficher plus** près de **Analyser la télémétrie**. Sélectionnez ensuite **count2**.
    * Cliquez sur **Enregistrer** pour enregistrer vos modifications. L’action **Interface de comptage du nombre de personnes** ressemble à la capture d’écran suivante : ![Action d’occupation](./media/tutorial-in-store-analytics-visualize-insights/occupancy-action-2.png)

L’application logique s’exécute automatiquement. Pour afficher l’état de chaque exécution, accédez à la page **Vue d’ensemble** de l’application logique dans la Portail Azure :

## <a name="create-a-power-bi-dashboard"></a>Créer un tableau de bord Power BI

À présent, vous avez des données de télémétrie provenant de votre application IoT Central par le biais de votre Event Hub. Ensuite, votre application logique analyse les messages Event Hub et les ajoute à un jeu de données de streaming Power BI. À présent, vous pouvez créer un tableau de bord Power BI pour visualiser les données de télémétrie :

1. Connectez-vous à votre compte **Power BI**.
1. Sélectionnez **Espace de travail > Analyses dans le magasin - paiement**.
1. Sélectionnez **Créer > Tableau de bord**.
1. Entrez **Store analytic** comme nom du tableau de bord, puis sélectionnez **Créer**.

### <a name="add-line-charts"></a>Ajouter des graphiques en courbes

Ajoutez quatre vignettes de graphiques en courbes pour afficher la température et l’humidité des deux capteurs environnementaux. Utilisez les informations du tableau suivant pour créer les vignettes. Pour ajouter chaque vignette, commencez par sélectionner **... (Plus d’options) > Ajouter une vignette**. Sélectionnez **Données de streaming personnalisées**, puis **Suivant** :

| Paramètre | Graphique #1 | Graphique #2 | Graphique #3 | Graphique #4 |
| ------- | -------- | -------- | -------- | -------- |
| Dataset | Capteur de zone 1 | Capteur de zone 1 | Capteur de zone 2 | Capteur de zone 2 |
| Type de visualisation | Graphique en courbes | Graphique en courbes | Graphique en courbes | Graphique en courbes |
| Axe | Timestamp | Timestamp | Timestamp | Timestamp |
| Valeurs | Température | Humidité | Température | Humidité |
| Fenêtre de temps | 60 minutes | 60 minutes | 60 minutes | 60 minutes |
| Intitulé | Température (1 heure) | Humidité (1 heure) | Température (1 heure) | Humidité (1 heure) |
| Sous-titre | Zone 1 | Zone 1 | Zone 2 | Zone 2 |

La capture d’écran suivante montre les paramètres pour le premier graphique :

![Paramètres de graphique en courbes](./media/tutorial-in-store-analytics-visualize-insights/line-chart.png)

### <a name="add-cards-to-show-environmental-data"></a>Ajouter des cartes pour afficher les données environnementales

Ajoutez quatre vignettes de carte pour afficher les valeurs de température et d’humidité les plus récentes des deux capteurs environnementaux. Utilisez les informations du tableau suivant pour créer les vignettes. Pour ajouter chaque vignette, commencez par sélectionner **... (Plus d’options) > Ajouter une vignette**. Sélectionnez **Données de streaming personnalisées**, puis **Suivant** :

| Paramètre | Carte #1 | Carte #2 | Carte #3 | Carte #4 |
| ------- | ------- | ------- | ------- | ------- |
| Dataset | Capteur de zone 1 | Capteur de zone 1 | Capteur de zone 2 | Capteur de zone 2 |
| Type de visualisation | Card | Card | Card | Card |
| Champs | Température | Humidité | Température | Humidité |
| Intitulé | Température (F) | Humidité (%) | Température (F) | Humidité (%) |
| Sous-titre | Zone 1 | Zone 1 | Zone 2 | Zone 2 |

(La capture d’écran suivante montre les paramètres pour le premier graphique :

![Paramètres de la carte](./media/tutorial-in-store-analytics-visualize-insights/card-settings.png)

### <a name="add-tiles-to-show-checkout-occupancy-data"></a>Ajouter des vignettes pour afficher les données de l’occupation d’extraction

Ajoutez quatre vignettes de carte pour afficher la longueur de la file d’attente et la durée de maintien des deux extractions dans le magasin. Utilisez les informations du tableau suivant pour créer les vignettes. Pour ajouter chaque vignette, commencez par sélectionner **... (Plus d’options) > Ajouter une vignette**. Sélectionnez **Données de streaming personnalisées**, puis **Suivant** :

| Paramètre | Carte #1 | Carte #2 | Carte #3 | Carte #4 |
| ------- | ------- | ------- | ------- | ------- |
| Dataset | Capteur d’occupation | Capteur d’occupation | Capteur d’occupation | Capteur d’occupation |
| Type de visualisation | Histogramme groupé | Histogramme groupé | Jauge | Jauge |
| Axe    | Timestamp | Timestamp | N/A | N/A |
| Valeur | Durée de maintien 1 | Durée de maintien 2 | Longueur de file d’attente 1 | Longueur de file d’attente 2 |
| Fenêtre de temps | 60 minutes | 60 minutes |  N/A | N/A |
| Intitulé | Durée de maintien | Durée de maintien | Longueur de la file d’attente | Longueur de la file d’attente |
| Sous-titre | Validation 1 | Validation 2 | Validation 1 | Validation 2 |

Redimensionnez et réorganisez les vignettes de votre tableau de bord pour qu’elles ressemblent à la capture d’écran suivante :

![Capture d’écran montrant le tableau de bord Power BI avec des vignettes redimensionnées et réorganisées.](./media/tutorial-in-store-analytics-visualize-insights/pbi-dashboard.png)

Vous pouvez ajouter des ressources graphiques supplémentaires pour personnaliser davantage le tableau de bord :

![Tableau de bord Power BI](./media/tutorial-in-store-analytics-visualize-insights/pbi-dashboard-graphics.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous avez terminé avec votre application IoT Central, vous pouvez la supprimer en vous connectant à l’application et en accédant à la page **Paramètres de l’application** dans la section **Administration**.

Si vous souhaitez conserver l’application tout en réduisant les coûts associés, désactivez l’exportation de données qui envoie les données de télémétrie à votre Event Hub.

Vous pouvez supprimer l’Event Hub et l’application logique dans le portail Azure en supprimant le groupe de ressources appelé **retail-store-analysis**.

Vous pouvez supprimer vos jeux de données Power BI et votre tableau de bord en supprimant l’espace de travail à partir de la page Paramètres Power BI de l’espace de travail.

## <a name="next-steps"></a>Étapes suivantes

Ces trois didacticiels vous ont présenté une solution de bout en bout qui utilise le modèle d’application IoT Central **Analytique dans le magasin - paiement**. Vous avez connecté des appareils à l’application, utilisé IoT Central pour surveiller les appareils et utilisé Power BI pour créer un tableau de bord afin d’afficher des insights à partir des données de télémétrie de l’appareil. L’étape suivante recommandée consiste à explorer l’un des autres modèles d’application IoT Central :

> [!div class="nextstepaction"]
> * [Créer des solutions énergétiques avec IoT Central](../energy/overview-iot-central-energy.md)
> * [Créer des solutions pour le secteur public avec IoT Central](../government/overview-iot-central-government.md)
> * [Créer des solutions de santé avec IoT Central](../healthcare/overview-iot-central-healthcare.md)
