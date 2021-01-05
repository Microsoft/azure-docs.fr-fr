---
title: Qu’est-ce qu’Azure FarmBeats ?
description: Offre une vue d’ensemble d’Azure FarmBeats
author: uhabiba04
ms.topic: overview
ms.date: 11/04/2019
ms.author: v-umha
ms.openlocfilehash: b08013e9d015d3c987c0032761bebccd4b57ec72
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92678023"
---
# <a name="overview-of-azure-farmbeats-preview"></a>Vue d’ensemble d’Azure FarmBeats (préversion)

Azure FarmBeats est une offre interentreprise disponible sur la Place de marché Azure. Cette solution permet d’agréger des jeux de données agricoles provenant de différents fournisseurs. Avec Azure FarmBeats, vous pouvez créer des modèles d’intelligence artificielle (IA) ou des modèles Machine Learning (ML) basés sur les jeux de données fusionnés. En utilisant Azure FarmBeats, les entreprises du secteur agricole peuvent se concentrer sur leur valeur ajoutée principale plutôt que passer du temps sur des tâches indifférenciées d’ingénierie des données.

> [!NOTE]
> Azure FarmBeats est actuellement disponible en préversion publique. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Azure FarmBeats est fourni sans contrat de niveau de service. Utilisez le [Forum de support Azure FarmBeats](/answers/topics/azure-farmbeats.html) pour obtenir du support.

![Projet FarmBeats](./media/architecture-for-farmbeats/farmbeats-architecture-1.png)

Avec la préversion d’Azure FarmBeats, vous pouvez :

- Évaluer l’intégrité de l’exploitation à l’aide d’un indice de la végétation et de l’eau basé sur des images satellite.
- Obtenir des suggestions sur le nombre de capteurs d’humidité du sol à utiliser et sur leur implantation.
- Effectuer le suivi de l’état de l’exploitation agricole en visualisant les données de sol collectées par les capteurs de différents fournisseurs.
- Obtenir une carte du taux d’humidité du sol basé sur la fusion de données satellite et de capteurs.
- Obtenir des insights exploitables en générant des modèles AI/ML à partir de jeux de données agrégés.
- Créer ou enrichir votre solution agricole numérique en fournissant des conseils pour favoriser l’intégrité de l’exploitation.

## <a name="datahub"></a>Datahub

Azure FarmBeats Datahub est une couche d’API qui permet l’agrégation, la normalisation et la contextualisation de jeux de données agricoles issus de différents fournisseurs. Vous pouvez utiliser Azure FarmBeats pour obtenir :
- Des **données de capteur** de fournisseurs de capteurs : [Davis Instruments](https://www.davisinstruments.com/product/enviromonitor-gateway/), [Teralytic](https://teralytic.com/), [Pessl Instruments](https://metos.at/)
- Des **images satellite** transmises par la mission du satellite [Sentinel-2](https://sentinel.esa.int/web/sentinel/home) de l’Agence spatiale européenne
- Des **images par drone** de trois fournisseurs d’images prises par des drones : [senseFly](https://www.sensefly.com/), [SlantRange](https://slantrange.com/) et [DJI](https://dji.com/)

Datahub est conçu comme une plateforme d’API extensible. Nous cherchons à intégrer d’autres fournisseurs à Azure FarmBeats pour vous proposer davantage d’options lors de la création de votre solution.

## <a name="accelerator"></a>Accélérateur

Azure FarmBeats Accelerator est un exemple d’application web qui repose sur DataHub. Cette solution accélère le développement de votre interface utilisateur et de votre modèle. Elle s’appuie sur les API d’Azure FarmBeats. Elle représente les données de capteurs ingérées sous forme de graphiques et illustre les sorties du modèle sur des cartes. Par exemple, vous pouvez utiliser l’accélérateur pour créer rapidement une exploitation agricole et obtenir facilement une carte de l’indice de végétation ou de l’implantation des capteurs de cette exploitation.

## <a name="azure-role-based-access-control-azure-rbac"></a>Contrôle d’accès en fonction du rôle Azure (Azure RBAC)

Un administrateur peut définir des règles d’accès pour Azure FarmBeats en utilisant l’un des rôles prédéfinis. Les rôles déterminent les zones de l’application auxquelles un utilisateur a accès ainsi que les actions qu’il peut effectuer. Il existe deux types de rôles dans Azure FarmBeats : pour les utilisateurs et pour les partenaires.

### <a name="user-roles"></a>Rôles d'utilisateur

Un [administrateur peut ajouter et gérer des utilisateurs](manage-users-in-azure-farmbeats.md) et définir leurs niveaux d’accès en fonction de deux rôles d’utilisateur : administrateur et lecture seule.

### <a name="partner-roles"></a>Partner Roles

Un administrateur peut ajouter plusieurs partenaires comme fournisseurs de données à Azure FarmBeats. Voici un résumé des rôles de partenaire disponibles dans FarmBeats et de leurs autorisations :

| Type de partenaire    |   Actions  | Étendue |
| ---- | -------- | -------- |
| Partenaire de capteur  |   Créer, lire, mettre à jour <br/> <br/> Lire, mettre à jour | DeviceModel, Device, SensorModel, Sensor <br/> <br/> ExtendedType |
| Partenaire d’imagerie  |   Créer, lire, mettre à jour <br/> <br/> Lire, mettre à jour <br/> <br/> Lire | Scene, SceneFile <br/> <br/> ExtendedType <br/> <br/> Farm |
| Partenaire météo* <br/> <br/>  (* Bientôt disponible) |   Créer, lire, mettre à jour <br/> <br/> Lire, mettre à jour <br/> <br/> Lire | WeatherDataModel, WeatherDataLocation, JobType <br/> <br/> ExtendedType <br/> <br/> Farm |

## <a name="resources"></a>Ressources

Azure FarmBeats est fourni sans frais supplémentaires. Vous payez uniquement les ressources Azure que vous consommez. Pour en savoir plus sur l’offre, vous pouvez :

- Lire les toutes dernières informations relatives à Azure FarmBeats en visitant notre [blog Azure FarmBeats](https://aka.ms/farmbeatsblog).
- Demander de l’aide en postant une question sur notre [forum de support Azure FarmBeats](/answers/topics/azure-farmbeats.html).
- Envoyer vos commentaires en postant une idée de fonctionnalité ou en votant pour une idée existante sur notre [forum de commentaires Azure FarmBeats](https://aka.ms/farmbeatsfeedback).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Installer Azure FarmBeats](install-azure-farmbeats.md)