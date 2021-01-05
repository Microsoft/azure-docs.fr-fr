---
title: 'Tutoriel : Configurer un environnement Gen2 - Azure Time Series Insights Gen2 | Microsoft Docs'
description: 'Tutoriel : Découvrez comment configurer un environnement dans Azure Time Series Insights Gen2.'
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: tutorial
ms.date: 09/30/2020
ms.custom: seodec18
ms.openlocfilehash: eeb3de2fc3f0e3e0be9c98002f11e470eaf04f8c
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95020927"
---
# <a name="tutorial-set-up-an-azure-time-series-insights-gen2-environment"></a>Tutoriel : Configurer un environnement Azure Time Series Insights Gen2

Ce tutoriel vous guide tout au long du processus de création d’un environnement Azure Time Series Insights Gen2 avec *paiement à l’utilisation*.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Créer un environnement Azure Time Series Insights Gen2.
> * Connecter l’environnement Azure Time Series Insights Gen2 à un hub IoT.
> * Exécuter un exemple d’accélérateur de solution pour transmettre des données à l’environnement Azure Time Series Insights Gen2.
> * Effectuer une analyse de base des données.
> * Définir une hiérarchie et un type de modèle Time Series, et les associer à vos instances

>[!TIP]
> [Les accélérateurs de solution IoT](https://www.azureiotsolutions.com/Accelerators) fournissent des solutions préconfigurées de classe entreprise, qui vous permettent d’accélérer le développement de solutions IoT personnalisées.

Souscrivez un [abonnement Azure gratuit](https://azure.microsoft.com/free/) si vous n’en avez pas déjà un.

## <a name="prerequisites"></a>Prérequis

* Au minimum, vous devez disposer du rôle **Contributeur** pour l’abonnement Azure. Pour plus d’informations, consultez [Ajouter ou supprimer des attributions de rôles Azure à l’aide du portail Azure](../role-based-access-control/role-assignments-portal.md).

## <a name="create-a-device-simulation"></a>Créer une simulation d’appareil

Dans cette section, vous allez créer trois appareils simulés qui envoient des données à une instance Azure IoT Hub.

1. Accédez à la [page des accélérateurs de solution Azure IoT](https://www.azureiotsolutions.com/Accelerators). La page affiche plusieurs exemples prédéfinis. Connectez-vous à l’aide de votre compte Azure. Puis sélectionnez **Simulation d'appareil**.

   [![Page des accélérateurs de solution Azure IoT](media/v2-update-provision/iot-solution-accelerators-landing-page.png)](media/v2-update-provision/iot-solution-accelerators-landing-page.png#lightbox)

1. Entrez l’abonnement dans lequel vous souhaitez créer la simulation d’appareil.

   Paramètre|Description
   ---|---
   **Nom du déploiement** | Cette valeur unique sert à créer un groupe de ressources. Les ressources Azure répertoriées sont créées et affectées au groupe de ressources.
   **Abonnement Azure** | Spécifiez le même abonnement que celui utilisé pour la création de votre environnement Azure Time Series Insights Gen2 dans la section précédente.
   **Options de déploiement** | Sélectionnez **Approvisionner nouveau IoT Hub** pour créer un nouvel IoT hub spécifique pour ce didacticiel.
   **Emplacement Azure** | Spécifiez la même région que celle utilisée pour la création de votre environnement Azure Time Series Insights Gen2 dans la section précédente.

   Quand vous avez terminé, sélectionnez **Créer** pour provisionner les ressources Azure de la solution. Ce processus peut prendre jusqu’à 20 minutes.

   [![Provisionner la solution de simulation d’appareil](media/v2-update-provision/iot-solution-accelerators-configuration.png)](media/v2-update-provision/iot-solution-accelerators-configuration.png#lightbox)

1. Une fois le provisionnement terminé, deux notifications s’affichent pour indiquer que l’état du déploiement est passé de **Provisionnement** à **Prêt**.

   >[!IMPORTANT]
   > N’entrez pas encore votre accélérateur de solution ! Gardez cette page web ouverte car vous aurez besoin d’y revenir plus tard.

   [![Provisionnement de la solution de simulation d’appareil terminé.](media/v2-update-provision/iot-solution-accelerator-ready.png)](media/v2-update-provision/iot-solution-accelerator-ready.png#lightbox)

1. Observons à présent les ressources nouvellement créées dans le portail Azure. Sur la page **Groupes de ressources**, vous remarquerez qu’un nouveau groupe de ressources a été créé à l’aide du **nom de la solution** indiqué à la dernière étape. Prenez note des ressources qui ont été créées pour la simulation d’appareil.

   [![Ressources de la simulation d’appareil](media/v2-update-provision/tsi-device-sim-solution-resources.png)](media/v2-update-provision/tsi-device-sim-solution-resources.png#lightbox)

## <a name="create-an-azure-time-series-insights-gen2-environment"></a>Créer un environnement Azure Time Series Insights Gen2

Cette section décrit comment créer un environnement Azure Time Series Insights Gen2 et le connecter au hub IoT créé par l’accélérateur de solution IoT à l’aide du [portail Azure](https://portal.azure.com/).

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide de votre compte d’abonnement Azure.
1. Cliquez sur **+ Créer une ressource** dans le coin supérieur gauche.
1. Sélectionnez la catégorie **Internet des objets**, puis **Time Series Insights**.

   [![Sélectionner la ressource d’environnement Time Series Insights](media/v2-update-provision/tsi-create-new-environment.png)](media/v2-update-provision/tsi-create-new-environment.png#lightbox)

1. Dans le volet **Créer un environnement Time Series Insights**, dans l’onglet **De base**, définissez les paramètres suivants :

    | Paramètre | Action |
    | --- | ---|
    | **Nom de l’environnement** | Entrez un nom unique pour l’environnement Azure Time Series Insights Gen2. |
    | **Abonnement** | Entrez l’abonnement pour lequel vous souhaitez créer l’environnement Azure Time Series Insights Gen2. Nous vous recommandons d'utiliser le même abonnement que pour les autres ressources IoT créées par le simulateur d'appareil. |
    | **Groupe de ressources** | Sélectionnez un groupe de ressources existant, ou créez-en un pour la ressource d’environnement Azure Time Series Insights Gen2. Un groupe de ressources est un conteneur pour les ressources Azure. Une bonne pratique consiste à utiliser le même groupe de ressources que pour les autres ressources IoT créées par le simulateur d’appareil. |
    | **Lieu** | Sélectionnez une région de centre de données pour votre environnement Azure Time Series Insights Gen2. Pour éviter toute latence supplémentaire, il est préférable de créer votre environnement Azure Time Series Insights Gen2 dans la même région que le hub IoT créé par le simulateur d’appareil. |
    | **Niveau** |  Sélectionnez **Gen2(L1)** . Il s’agit de la référence SKU du produit Azure Time Series Insights Gen2. |
    | **Nom de propriété d’ID Time Series** | Entrez le nom d’une propriété qui contient des valeurs qui identifient de façon unique les instances de la série chronologique. La valeur que vous entrez dans la zone **Nom de la propriété** en tant qu’ID Time Series ne peut pas être modifiée ultérieurement. Pour ce tutoriel, utilisez l’identificateur **_iothub-connection-device-id_* _. Pour en savoir plus sur l’ID Time Series, notamment sur l’ID Time Series composite, consultez [Bonnes pratiques pour le choix d’un ID Time Series](./how-to-select-tsid.md). |
    | _ *Nom du compte de stockage** | Entrez un nom global unique pour le nouveau compte de stockage.|
    | **Type de compte de stockage** | Sélectionnez le type de stockage pour un nouveau compte de stockage. Nous vous recommandons StorageV2.|
    | **Réplication de compte de stockage** | Sélectionnez le type de stockage pour un nouveau compte de stockage. En fonction de l’emplacement sélectionné, vous pouvez choisir parmi LRS, GRS et ZRS. Pour ce tutoriel, vous pouvez sélectionner LRS.|
    | **Espace de noms hiérarchique** |Cette option peut être sélectionnée, une fois que vous avez sélectionné le type de stockage StorageV2r. Elle est désactivée par défaut. Pour ce tutoriel, vous pouvez la conserver dans son état par défaut *désactivé*.|
    |**Activer le magasin chaud**|Sélectionnez **Oui** pour activer le magasin chaud. Ce paramètre peut être désactivé et réactivé une fois que l’environnement a également été créé. |
    |**Conservation des données (en jours)**|Choisissez l’option par défaut de 7 jours. |

    [![Configuration du nouvel environnement Time Series Insights](media/v2-update-provision/tsi-environment-configuration.png)](media/v2-update-provision/tsi-environment-configuration.png#lightbox)

1. Sélectionnez **Suivant : Source de l'événement**.

   [![Configurez l’ID de la série chronologique de l’environnement](media/v2-update-provision/tsi-time-series-id-selection.png)](media/v2-update-provision/tsi-time-series-id-selection.png#lightbox)

1. Dans l’onglet **Source de l'événement**, définissez les paramètres suivants :

   | Paramètre | Action |
   | --- | --- |
   | **Créer une source d’événement ?** | Sélectionnez **Oui**.|
   | **Nom** | Entrez une valeur unique pour le nom de la source d’événement. |
   | **Type de source** | Sélectionnez **IoT Hub**. |
   | **Sélectionner un hub** | Choisissez **Sélection d’un élément existant**. |
   | **Abonnement** | Sélectionnez l’abonnement que vous avez utilisé pour le simulateur d’appareil. |
   | **Nom du hub IoT** | Entrez le nom du hub IoT que vous avez créé pour le simulateur d’appareil. |
   | **Stratégie d'accès IoT Hub** | sélectionnez **iothubowner**. |
   | **Groupe de consommateurs IoT Hub** | Sélectionnez **Nouveau**, entrez un nom unique, puis sélectionnez **+ Ajouter**. Le groupe de consommateurs doit représenter une valeur unique dans Azure Time Series Insights Gen2. |
   | **Propriété timestamp** | Cette valeur utilisée pour identifier la propriété d'**horodatage** dans vos données de télémétrie entrantes. Pour ce didacticiel, laissez cette zone vide. Ce simulateur utilise le timestamp entrant du hub IoT auquel Azure Time Series Insights Gen2 est associé par défaut. |

1. Sélectionnez **Vérifier + créer**.

   [![Configurer le hub IoT créé comme une source d’événement](media/v2-update-provision/tsi-configure-event-source.png)](media/v2-update-provision/tsi-configure-event-source.png#lightbox)

1. Sélectionnez **Create** (Créer).

    [![Page Vérifier + Créer, avec le bouton Créer](media/v2-update-provision/tsi-environment-confirmation.png)](media/v2-update-provision/tsi-environment-confirmation.png#lightbox)

    Vous pouvez examiner l’état de votre déploiement :

    [![Notification de la fin du déploiement](media/v2-update-provision/tsi-deployment-notification.png)](media/v2-update-provision/tsi-deployment-notification.png#lightbox)

1. Vous avez accès à votre environnement Azure Time Series Insights Gen2 par défaut si vous êtes propriétaire de l’abonnement Azure. Pour vérifier votre accès :

   1. Recherchez votre groupe de ressources et sélectionnez votre environnement Azure Time Series Insights Gen2.

      [![Sélectionner et afficher l’environnement](media/v2-update-provision/verify-tsi-resource-in-group.png)](media/v2-update-provision/verify-tsi-resource-in-group.png#lightbox)

   1. Dans la page Azure Time Series Insights Gen2, sélectionnez **Stratégies d’accès aux données** :

      [![Vérifier les stratégies d’accès aux données](media/v2-update-provision/tsi-data-access-panel.png)](media/v2-update-provision/tsi-data-access-panel.png#lightbox)

   1. Vérifiez que vos informations d'identification sont répertoriées :

      Si vos informations d’identification ne sont pas listées, vous devez vous octroyer l’autorisation d’accéder à l’environnement en sélectionnant Ajouter et en recherchant vos informations d'identification. Pour en savoir plus sur la définition des autorisations, consultez [Octroyer l’accès aux données](./concepts-access-policies.md).

## <a name="stream-data"></a>Transmettre des données en continu

Maintenant que vous avez déployé votre environnement Azure Time Series Insights Gen2, commencez à lui transmettre des données en continu à des fins d’analyse.

1. Revenez à votre [tableau de bord Accélérateurs de solution](https://www.azureiotsolutions.com/Accelerators#dashboard). Connectez-vous à nouveau si nécessaire, à l’aide du même compte Azure que celui utilisé dans ce didacticiel. Sélectionnez votre « solution d’appareil », puis **accédez à votre accélérateur de solution** pour lancer votre solution déployée.

   [![Tableau de bord des accélérateurs de solution](media/v2-update-provision/iot-solution-accelerator-ready.png)](media/v2-update-provision/iot-solution-accelerator-ready.png#lightbox)

1. L’application web de simulation d’appareil commence par vous inviter à lui accorder l’autorisation **Vous connecter et lire votre profil**. Cette autorisation permet à l’application de récupérer les informations des profils utilisateur dont l’application a besoin pour fonctionner.

   [![Autorisation de l’application web de simulation d’appareil](media/v2-update-provision/sawa-signin-consent.png)](media/v2-update-provision/sawa-signin-consent.png#lightbox)

1. Sélectionnez **+ Nouvelle simulation**.

    1. Une fois la page **Configuration de la simulation** chargée, entrez les paramètres requis.

        | Paramètre | Action |
        | --- | --- |
        | **Nom** | Entrez un nom unique pour le simulateur. |
        | **Description** | Entrez une définition. |
        | **Durée de la simulation** | Définissez ce paramètre sur **Run indefinitely** (Exécuter indéfiniment). |
        | **Modèle de l’appareil** | Cliquez sur + **Ajouter un type d’appareil** <br />**Name** : Entrez **Ascenseur**. <br />**Quantité** : Entrez **3**. <br /> Conservez les valeurs par défaut restantes |
        | **IoT Hub cible** | Définissez ce paramètre sur **Use pre-provisioned IoT Hub** (Utiliser le hub IoT préprovisionné). |

        [![Configurer les paramètres et lancer la simulation](media/v2-update-provision/tsi-launch-solution-accelerator.png)](media/v2-update-provision/tsi-launch-solution-accelerator.png#lightbox)

    1. Sélectionnez **Démarrer la simulation**. Dans le tableau de bord de simulation d’appareil, les **Appareils actifs** et le **Nombre total de messages** sont affichés.

        [![Tableau de bord de la simulation Azure IoT](media/v2-update-provision/tsi-see-active-devices-and-messages.png)](media/v2-update-provision/tsi-see-active-devices-and-messages.png#lightbox)

## <a name="analyze-data"></a>Analyser des données

Dans cette section, vous procédez à l’analyse de base de vos données de séries chronologiques à l’aide de [l’explorateur Azure Time Series Insights Gen2](./concepts-ux-panels.md).

1. Accédez à l’explorateur Azure Time Series Insights Gen2 en sélectionnant l’URL dans la page des ressources sur le [portail Azure](https://portal.azure.com/).

    [![URL de l’Explorateur Azure Time Series Insights Gen2.](media/v2-update-provision/tsi-select-explorer-url.png)](media/v2-update-provision/tsi-select-explorer-url.png#lightbox)

1. Dans l’explorateur Azure Time Series Insights Gen2, une barre s’affiche en haut de l’écran. Il s’agit de votre sélecteur de disponibilité. Vérifiez que vous avez sélectionné au moins 2 m et, si nécessaire, développez le délai d’exécution en sélectionnant et en faisant glisser les poignées du sélecteur vers la gauche et vers la droite.

1. **Instances Time Series** sera affiché sur le côté gauche.

    [![Liste des instances non apparentées](media/v2-update-provision/tsi-explorer-unparented-instances.png)](media/v2-update-provision/tsi-explorer-unparented-instances.png#lightbox)

1. Sélectionnez la première instance de série chronologique. Sélectionnez ensuite **Afficher la température**.

    [![Instance de série chronologique sélectionnée avec la commande de menu pour afficher la température moyenne](media/v2-update-provision/select-instance-and-temperature.png)](media/v2-update-provision/select-instance-and-temperature.png#lightbox)

    Un graphique des séries chronologiques apparaît. Modifiez l'**Intervalle** sur **30s**.

1. Répétez l’étape précédente avec les deux autres instances de séries chronologiques pour afficher les trois, comme indiqué dans le graphique suivant :

    [![Graphique affichant toutes les séries chronologiques](media/v2-update-provision/tsi-explorer-add-three-instances.png)](media/v2-update-provision/tsi-explorer-add-three-instances.png#lightbox)

1. Sélectionnez le sélecteur d'intervalle de temps dans le coin supérieur droit. Ici, vous pouvez sélectionner des heures de début et de fin à la milliseconde près, ou choisir parmi les options préconfigurées telles que **30 dernières minutes**. Vous avez également la possibilité de modifier le fuseau horaire par défaut.

    [![Définir l’intervalle de temps sur les 30 dernières minutes](media/v2-update-provision/tsi-explorer-thirty-minute-time-range.png)](media/v2-update-provision/tsi-explorer-thirty-minute-time-range.png#lightbox)

    La progression de l’accélérateur de solution au cours des **30 dernières minutes** s’affiche à présent dans l’explorateur Azure Time Series Insights Gen2.

## <a name="define-and-apply-a-model"></a>Définir et appliquer un modèle

Dans cette section, vous appliquez un modèle pour structurer vos données. Pour finaliser le modèle, vous définissez les types, les hiérarchies et les instances. Pour en savoir plus sur la modélisation des données, consultez [Modèle Time Series](./concepts-model-overview.md).

1. Dans l’explorateur, sélectionnez l’onglet **Modèle** :

   [![Afficher l’onglet Modèle dans l’Explorateur](media/v2-update-provision/tsi-select-model-view.png)](media/v2-update-provision/tsi-select-model-view.png#lightbox).

   Sous l’onglet **Types**, sélectionnez **+ Ajouter**.

1. Entrez les paramètres suivants :

    | Paramètre | Action |
    | --- | ---|
    | **Nom** | Entrez **Ascenseur** |
    | **Description** | Entrez **Définition du type d'ascenseur** |

1. Sélectionnez ensuite l'onglet **Variables**.

    1. Sélectionnez **+ Ajouter une variable** et renseignez les valeurs suivantes pour la première variable du type d'ascenseur. Vous créerez trois variables en tout.

        | Paramètre | Action |
        | --- | --- |
        | **Nom** | Entrez **Avg Temperature** (Température moyenne). |
        | **Type** | Sélectionner **Numérique** |
        | **Valeur** | Sélectionner à partir d'une présélection : Sélectionnez **Température (Double)** . <br /> Remarque : Le remplissage automatique du champ **Valeur** peut prendre plusieurs minutes après qu’Azure Time Series Insights Gen2 a commencé à recevoir des événements.|
        | **Opération d'agrégation** | Développez **Options avancées**. <br /> Sélectionnez **AVG**. |

    1. Sélectionnez **Appliquer**. Ensuite, sélectionnez à nouveau **+ Ajouter une variable**, puis définissez les valeurs suivantes :

        | Paramètre | Action |
        | --- | --- |
        | **Nom** | Entrez **Vibration moyenne**. |
        | **Type** | Sélectionner **Numérique** |
        | **Valeur** | Sélectionner à partir d'une présélection : Sélectionnez **vibration (double)** . <br /> Remarque : Le remplissage automatique du champ **Valeur** peut prendre plusieurs minutes après qu’Azure Time Series Insights Gen2 a commencé à recevoir des événements.|
        | **Opération d'agrégation** | Développez **Options avancées**. <br /> Sélectionnez **AVG**. |

    1. Sélectionnez **Appliquer**. Ensuite, sélectionnez à nouveau **+ Ajouter une variable**, puis définissez les valeurs suivantes pour la troisième et dernière variable :

        | Paramètre | Action |
        | --- | --- |
        | **Nom** | Entrez **Étage**. |
        | **Type** | Sélectionner **Par catégorie** |
        | **Valeur** | Sélectionner à partir d'une présélection : Sélectionnez **Étage (double)** . <br /> Remarque : Le remplissage automatique du champ **Valeur** peut prendre plusieurs minutes après qu’Azure Time Series Insights Gen2 a commencé à recevoir des événements.|
        | **Catégories** | <span style="text-decoration: underline">Valeurs d'</span>  - <span style="text-decoration: underline">étiquette</span> <br /> Inférieur : 1,2,3,4 <br /> Intermédiaire : 5,6,7,8,9 <br /> Supérieur : 10,11,12,13,14,15 |
        | **Catégorie par défaut** | Entrer **Inconnu** |

        [![Ajouter les variables de type](media/v2-update-provision/tsi-add-type-variables.png)](media/v2-update-provision/tsi-add-type-variables.png#lightbox)

    1. Sélectionnez **Appliquer**. 
    1. Sélectionnez **Enregistrer**. Trois variables sont créées et affichées.

        [![Après avoir ajouté le type, consultez-le dans la vue Modèle.](media/v2-update-provision/tsi-add-type-and-view.png)](media/v2-update-provision/tsi-add-type-and-view.png#lightbox)

1. Sélectionnez l’onglet **Hiérarchies**. Sélectionnez ensuite **+ Ajouter**.

   1. Dans le volet **Modifier la hiérarchie**, définissez les paramètres suivants :

        | Paramètre | Action |
        | --- | ---|
        | **Nom** | Entrez **Location Hierarchy** (Hiérarchie d’emplacement). |
        |**Niveaux**| Entrez **Pays** comme nom du premier niveau <br /> Sélectionnez **+ Ajouter un niveau** <br /> Entrez **Ville** pour le deuxième niveau, puis sélectionnez **+ Ajouter un niveau** <br /> Entrez **Immeuble** comme nom du troisième et dernier niveau |

   1. Sélectionnez **Enregistrer**.

        [![Afficher votre nouvelle hiérarchie dans la vue Modèle](media/v2-update-provision/tsi-add-hierarchy-and-view.png)](media/v2-update-provision/tsi-add-hierarchy-and-view.png#lightbox)

1. Accédez à **Instances**.

    1. Sous **Actions** à l’extrême droite, sélectionnez l’icône de crayon pour modifier la première instance avec les valeurs suivantes :

        | Paramètre | Action |
        | --- | --- |
        | **Type** | Sélectionnez **Ascenseur**. |
        | **Nom** | Entrez **Ascenseur 1**|
        | **Description** | Entrez **Instance pour ascenseur 1** |

    1. Accédez à **Champs d’instance** et entrez les valeurs suivantes :

        | Paramètre | Action |
        | --- | --- |
        | **Hiérarchies** | Sélectionnez **Hiérarchie d'emplacement** |
        | **Pays** | Entrez **USA** |
        | **Ville** | Entrez **Seattle** |
        | **Immeuble** | Entrez **Space Needle** |

    1. Sélectionnez **Enregistrer**.

1. Répétez l’étape précédente pour les deux autres instances en utilisant les valeurs suivantes :

    **Pour l’ascenseur 2** :

    | Paramètre | Action |
    | --- | --- |
    | **Type** | Sélectionnez **Ascenseur**. |
    | **Nom** | Entrez **Ascenseur 2**|
    | **Description** | Entrez **Instance for ascenseur 2** |
    | **Hiérarchies** | Sélectionnez **Hiérarchie d'emplacement** |
    | **Pays** | Entrez **USA** |
    | **Ville** | Entrez **Seattle** |
    | **Immeuble** | Entrez **Pacific Science Center** |

    **Pour l’ascenseur 3** :

    | Paramètre | Action |
    | --- | --- |
    | **Type** | Sélectionnez **Ascenseur**. |
    | **Nom** | Entrez **Ascenseur 3**|
    | **Description** | Entrez **Instance pour ascenseur 3** |
    | **Hiérarchies** | Sélectionnez **Hiérarchie d'emplacement** |
    | **Pays** | Entrez **USA** |
    | **Ville** | Entrez **New York** |
    | **Immeuble** | Entrez **Empire State Building** |

    [![Afficher les instances mises à jour](media/v2-update-provision/iot-solution-accelerator-instances.png)](media/v2-update-provision/iot-solution-accelerator-instances.png#lightbox)

1. Revenez à l’onglet **Analyser** pour afficher le volet graphique. Sous **Location Hierarchy** (Hiérarchie d’emplacement), développez tous les niveaux de hiérarchie pour afficher les instances de séries chronologiques :

    [![Afficher toutes les hiérarchies dans la vue Graphique](media/v2-update-provision/iot-solution-accelerator-view-hierarchies.png)](media/v2-update-provision/iot-solution-accelerator-view-hierarchies.png#lightbox)

1. Sous **Pacific Science Center**, sélectionnez l’instance de série chronologique **Ascenseur 2**, puis **Afficher la température moyenne**.

1. Pour la même instance, **Ascenseur 2**, sélectionnez **Afficher l'étage**.

    Avec votre variable par catégorie, vous pouvez déterminer le temps que passe l'ascenseur aux étages inférieurs, supérieurs et intermédiaires.

    [![Visualiser l’ascenseur 2 avec la hiérarchie et les données](media/v2-update-provision/iot-solution-accelerator-elevator-two.png)](media/v2-update-provision/iot-solution-accelerator-elevator-two.png#lightbox)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Maintenant que vous avez terminé le didacticiel, nettoyez les ressources que vous avez créées :

1. Dans le menu de gauche du [portail Azure](https://portal.azure.com), sélectionnez **Toutes les ressources**, puis recherchez votre groupe de ressources Azure Time Series Insights Gen2.
1. Vous pouvez supprimer l’intégralité du groupe de ressources (et toutes les ressources qu’il contient) en sélectionnant **Supprimer** ou supprimer chaque ressource individuellement.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :  

* Créer et utiliser un accélérateur de simulation d'appareil
* Créer un environnement Azure Time Series Insights Gen2 avec paiement à l’utilisation
* Connecter l’environnement Azure Time Series Insights Gen2 à un hub IoT
* Exécuter un exemple d’accélérateur de solution pour transmettre des données à l’environnement Azure Time Series Insights Gen2
* Effectuer une analyse de base des données
* Définir une hiérarchie et un type de modèle Time Series, et les associer à vos instances

Maintenant que vous savez créer votre propre environnement Azure Time Series Insights Gen2, penchez-vous sur les concepts clés d’Azure Time Series Insights Gen2.

En savoir plus sur l’ingestion Azure Time Series Insights Gen2 :

> [!div class="nextstepaction"]
> [Présentation de l’ingestion de données Azure Time Series Insights Gen2](./concepts-ingestion-overview.md)

En savoir plus sur le stockage Azure Time Series Insights Gen2 :

> [!div class="nextstepaction"]
> [Stockage de données Azure Time Series Insights Gen2](./concepts-storage.md)

En savoir plus sur les modèles Time Series :

> [!div class="nextstepaction"]
> [Modélisation de données Azure Time Series Insights Gen2](./concepts-model-overview.md)

En savoir plus sur la connexion de votre environnement à Power BI :

> [!div class="nextstepaction"]
> [Visualiser des données à partir d’Azure Time Series Insights Gen2 dans Power BI](./how-to-connect-power-bi.md)