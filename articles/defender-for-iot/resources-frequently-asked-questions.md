---
title: Forum aux questions sur Defender for IoT
description: Trouvez des réponses à la plupart des questions fréquemment posées sur les fonctionnalités et le service Azure Defender pour IoT.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: rkarlin
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/07/2020
ms.author: rkarlin
ms.openlocfilehash: 5e7eabd44ea8c56fbb102f9e48812745a31de62a
ms.sourcegitcommit: 30505c01d43ef71dac08138a960903c2b53f2499
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92089194"
---
# <a name="azure-defender-for-iot-frequently-asked-questions"></a>Forum aux questions sur Azure Defender pour IoT

Cet article fournit une liste de questions fréquemment posées et les réponses correspondantes au sujet de Defender pour IoT.

## <a name="what-is-azures-unique-value-proposition-for-iot-security"></a>Quelle est la proposition de valeur unique d’Azure pour la sécurité IoT ?

Defender pour IoT permet aux entreprises d’étendre leur vue de la cybersécurité existante à la totalité des solutions IoT. Azure fournit une vue de bout en bout de votre solution d’entreprise, ce qui vous permet de prendre des mesures et décisions professionnelles en fonction de la posture de sécurité de votre entreprise et des données collectées. La sécurité combinée d’Azure IoT, Azure IoT Edge et Azure Security Center vous permet de créer la solution que vous souhaitez avec la sécurité dont vous avez besoin.

## <a name="our-organization-uses-proprietary-non-standard-industrial-protocols-are-they-supported"></a>Notre organisation utilise des protocoles industriels non standard propriétaires. Sont-ils pris en charge ? 

Azure Defender pour IoT offre une prise en charge complète des protocoles. En plus de la prise en charge des protocoles incorporés, vous pouvez sécuriser les appareils IoT et OT exécutant des protocoles propriétaires et personnalisés, ou des protocoles qui s’écartent de n’importe quel standard. À l’aide du kit de développement logiciel (SDK) horizon Open Development Environment (ODE), les développeurs peuvent créer des plug-ins de dissecteurs qui décodent le trafic réseau en fonction des protocoles définis. Le trafic est analysé par les services pour fournir une analyse complète, des alertes et des rapports. Utilisez horizon pour :
- Développez la visibilité et le contrôle sans avoir à effectuer une mise à niveau vers les nouvelles versions.
- Sécurisez les informations propriétaires en développant sur site en tant que plug-in externe. 
- Localisez le texte pour les alertes, les événements et les paramètres de protocole.

Cette solution unique pour le développement de protocoles en tant que plug-ins ne requiert pas d’équipes de développeurs dédiées ou de versions de version pour prendre en charge un nouveau protocole. Les développeurs, les partenaires et les clients peuvent développer des protocoles en toute sécurité et partager des Insights et des connaissances en utilisant Horizon. 

## <a name="do-i-have-to-purchase-hardware-appliances-from-microsoft-partners"></a>Dois-je acheter des appliances matérielles auprès de partenaires Microsoft ?
Azure Defender pour le capteur IoT s’exécute sur des spécifications matérielles spécifiques, comme décrit dans le [Guide des spécifications matérielles](https://aka.ms/AzureDefenderforIoTBareMetalAppliance), les clients peuvent acheter du matériel certifié auprès de partenaires Microsoft ou utiliser la nomenclature (BOM) fournie et les acheter eux-mêmes. 

Le matériel certifié a été testé dans nos laboratoires pour la stabilité des pilotes, le rejet des paquets et le dimensionnement du réseau.


## <a name="regulation-does-not-allow-us-to-connect-our-system-to-the-internet-can-we-still-utilize-defender-for-iot"></a>La réglementation ne nous permet pas de connecter notre système à Internet. Pouvons-nous toujours utiliser Defender pour IoT ?

Oui, vous pouvez ! La solution locale Azure Defender pour la plateforme IoT est déployée en tant qu’appliance de capteur physique ou virtuel qui ingère passivement le trafic réseau (via SPAN, RSPAN ou TAP) pour analyser, détecter et surveiller en continu les réseaux IT, OT et IoT. Pour les grandes entreprises, plusieurs capteurs peuvent agréger leurs données dans une console de gestion locale.

## <a name="where-in-the-network-should-i-connect-monitoring-ports"></a>Où dois-je connecter les ports de surveillance dans le réseau ?

Le capteur Azure Defender pour IoT se connecte à un port SPAN ou un TAP réseau et commence immédiatement à collecter le trafic réseau ICS via une surveillance passive (sans agent). Il n’a aucun impact sur les réseaux OT, car il n’est pas placé dans le chemin d’accès aux données et n’analyse pas activement les appareils.

Par exemple :
- Une seule Appliance (virtuelle physique) peut se trouver dans la couche DMZ de l’atelier, ce qui a pour effet d’acheminer le trafic des cellules de l’atelier vers cette couche.
- Vous pouvez également localiser de petits mini-capteurs dans chaque cellule d’atelier avec l’administration Cloud ou locale qui résidera dans la couche DMZ d’atelier. Une autre appliance (virtuelle ou physique) peut surveiller le trafic dans la couche DMZ d’atelier (pour les environnements SCADA, historien ou MES).

## <a name="how-does-defender-for-iot-compare-to-the-competition"></a>Comment Defender pour IoT se compare-t-il par rapport à la concurrence ?

Là où les autres solutions fournissent un ensemble de fonctionnalités qui permettent aux clients de créer leurs propres solutions, Defender pour IoT fournit une solution de sécurité IoT de bout en bout unique qui offre une vue plus large sur la sécurité de toutes vos ressources liées à Azure. Azure permet un déploiement rapide et une intégration complète avec les jumeaux de module IoT Hub, pour une intégration aisée avec les outils de gestion des appareils existants.


## <a name="do-i-have-to-be-an-azure-iot-customer"></a>Dois-je être un client Azure IoT ?

Oui. Pour les déploiements connectés au Cloud, Azure Defender pour IoT s’appuie sur la connectivité et l’infrastructure Azure IoT.
## <a name="can-i-create-my-own-alerts"></a>Puis-je créer mes propres alertes ?

Oui. Vous pouvez définir une alerte personnalisée pour un ensemble prédéfini de comportements, comme l’adresse IP et les ports ouverts. Consultez [Créer des alertes personnalisées](quickstart-create-custom-alerts.md) pour en savoir plus sur les alertes personnalisées et la façon de les créer.

## <a name="where-can-i-see-logs-can-i-customize-logs"></a>Où puis-je voir les journaux ? Puis-je personnaliser les journaux ?

- Affichez les alertes et recommandations depuis votre espace de travail Log Analytics. Configurez la taille de stockage et la durée dans l’espace de travail.

- Les données brutes de votre agent de sécurité peuvent également être stockées dans votre compte Log Analytics. Envisagez les exigences de taille, durée et stockage ainsi que les coûts associés avant de modifier la configuration de cette option.



## <a name="what-happens-when-the-internet-connection-stops-working"></a>Que se passe-t-il lorsque la connexion Internet cesse de fonctionner ?

Les capteurs et les agents continuent à s’exécuter et à stocker les données tant que l’appareil est en cours d’exécution. Les données sont stockées dans le cache de messages de sécurité en fonction de la configuration de taille. Lorsque l’appareil retrouve sa connectivité, les messages de sécurité sont à nouveau envoyés.





## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la prise en main de Defender pour IoT, consultez la documentation suivante :

- Lire la [Vue d’ensemble de Defender pour IoT](overview.md)
- Vérifier les [conditions préalables du service](service-prerequisites.md)
- En savoir plus sur la [prise en main](getting-started.md)
- Comprendre les [Alertes de sécurité de Defender pour IoT](concept-security-alerts.md)
