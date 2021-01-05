---
title: 'Concepts : Flux de données dans les Connecteur Azure IoT pour FHIR (préversion) de l’API Azure pour FHIR'
description: Comprenez le flux de données du connecteur Azure IoT pour FHIR (préversion). Le connecteur Azure IoT pour FHIR (préversion) ingère, normalise, regroupe, transforme et conserve des données IoMT sur l’API Azure pour FHIR.
services: healthcare-apis
author: ms-puneet-nagpal
ms.service: healthcare-apis
ms.subservice: iomt
ms.topic: conceptual
ms.date: 11/13/2020
ms.author: punagpal
ms.openlocfilehash: 310be095af10f5ed5860c4f627caa0373be55835
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94629663"
---
# <a name="azure-iot-connector-for-fhir-preview-data-flow"></a>Flux de données du Connecteur Azure IoT pour FHIR (préversion)

Cet article fournit une vue d’ensemble du flux de données dans le connecteur Azure IoT pour Fast Healthcare Interoperability Resources (FHIR&#174;)*. Vous en apprendrez davantage sur les différentes étapes du traitement des données dans le Connecteur IoT pour FHIR, qui transforment les données d’appareils en ressources d’[Observation](https://www.hl7.org/fhir/observation.html) basées sur FHIR.

![Flux de données du Connecteur Azure IoT pour FHIR](media/concepts-iot-data-flow/iot-connector-data-flow.png)

Le diagramme ci-dessus illustre les flux de données courants utilisant le Connecteur Azure IoT pour FHIR. 

Voici les différentes étapes par lesquelles les données passent une fois que le Connecteur Azure IoT pour FHIR les a reçues.

## <a name="ingest"></a>Ingérer
L’ingestion est la première étape où les données d’appareils sont reçues dans le Connecteur Azure IoT pour FHIR. Le point de terminaison d’ingestion pour les données d’appareils est hébergé sur un [Azure Event Hub](../event-hubs/index.yml). La plateforme Azure Event Hub prend en charge une échelle et un débit élevés, avec la possibilité de recevoir et de traiter des millions de messages par seconde. Elle permet également au Connecteur Azure IoT pour FHIR de consommer des messages de manière asynchrone, ce qui évite aux appareils d’attendre que les données soient traitées.

> [!NOTE]
> JSON est le seul format actuellement pris en charge pour les données d’appareils.

## <a name="normalize"></a>Normaliser
La normalisation est l’étape suivante dans laquelle les données d’appareils sont récupérées à partir de l’Azure Event Hub ci-dessus et traitées à l’aide de modèles de mappage d’appareil. Ce processus de mappage entraîne une transformation des données d’appareils en un schéma normalisé. 

Le processus de normalisation non seulement simplifie le traitement des données à des étapes ultérieures, mais aussi permet de projeter un message d’entrée vers plusieurs messages normalisés. Par exemple, un appareil pourrait envoyer plusieurs signes vitaux pour la température du corps, la fréquence cardiaque, la pression artérielle et la fréquence respiratoire dans un message unique. Ce message d’entrée crée quatre ressources FHIR distinctes. Chaque ressource représente un signe vital différent, avec le message d’entrée projeté dans quatre messages normalisés différents.

## <a name="group"></a>Grouper
Le groupement est l’étape suivante dans laquelle les messages normalisés disponibles de l’étape précédente sont groupés sur la base de trois paramètres différents : identité de l’appareil, type de mesure et période.

Lee groupements par identité de l’appareil et types de mesure permettent d’utiliser le type de mesure [SampledData](https://www.hl7.org/fhir/datatypes.html#SampledData). Ce type fournit un moyen concis de représenter une série de mesures basées sur le temps d’un appareil dans FHIR. Et la période la latence à laquelle les ressources d’observation générées par Connecteur Azure IoT pour FHIR sont écrites dans l’API Azure pour FHIR.

> [!NOTE]
> La valeur de la période est définie par défaut sur 15 minutes et ne peut pas être configurée pour la préversion.

## <a name="transform"></a>Transformer
Dans l’étape de transformation, les messages normalisés groupés sont traités via des modèles de mappage FHIR. Les messages correspondant à un type de modèle sont transformés en ressources d’observation basées sur FHIR, comme spécifié via le mappage.

À ce stade, une ressource [Appareil](https://www.hl7.org/fhir/device.html), ainsi que sa ressource [Patient](https://www.hl7.org/fhir/patient.html) associée, sont également récupérées du serveur FHIR à l’aide de l’identificateur d’appareil présent dans le message. Ces ressources sont ajoutées en tant que référence à la ressource Observation en cours de création.

> [!NOTE]
> Toutes les recherches d’identité sont mises en cache une fois résolues pour réduire la charge sur le serveur FHIR. Si vous prévoyez de réutiliser des appareils avec plusieurs patients, il est recommandé de créer une ressource appareil virtuel spécifique du patient et d’envoyer l’identificateur d’appareil virtuel dans la charge utile du message. L’appareil virtuel peut être lié à la ressource appareil réelle en tant que parent.

S’il n’existe aucune ressource appareil pour un identificateur d’appareil donné dans le serveur FHIR, le résultat dépend de la valeur de `Resolution Type` définie au moment de la création. Lorsque la valeur est définie sur `Lookup`, le message spécifique est ignoré et le pipeline continue de traiter d’autres messages entrants. Si la valeur est définie sur `Create`, le Connecteur Azure IoT pour FHIR crée un appareil nu et des ressources patient sur le serveur FHIR.  

## <a name="persist"></a>Conserver
Une fois la ressource FHIR Observation générée à l’étape de transformation, elle est enregistrée dans l’API Azure pour FHIR. Si la ressource FHIR est nouvelle, elle est créée sur le serveur. Si la ressource FHIR existe déjà, elle est mise à jour.

## <a name="next-steps"></a>Étapes suivantes

Cliquez ci-dessous sur l’étape suivante pour apprendre à créer des modèles de mappage d’appareil et FHIR.

>[!div class="nextstepaction"]
>[Modèles de mappage du Connecteur Azure IoT pour FHIR](iot-mapping-templates.md)

*Sur le Portail Azure, le Connecteur Azure IoT pour FHIR est appelé Connecteur IoT (préversion). Le sigle FHIR est une marque déposée de HL7 et est utilisé avec l’autorisation de HL7. 