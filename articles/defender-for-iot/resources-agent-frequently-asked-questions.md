---
title: Forum aux questions sur Azure Defender pour IoT
description: Trouvez les réponses aux questions les plus fréquemment posées sur Azure Defender pour IoT.
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
ms.openlocfilehash: 9862519a2003eb373c43fef1b660986a8d830327
ms.sourcegitcommit: 30505c01d43ef71dac08138a960903c2b53f2499
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92094129"
---
# <a name="azure-defender-for-iot-agent-frequently-asked-questions"></a>Forum aux questions sur Azure Defender pour IoT

Cet article fournit la liste des questions fréquemment posées et des réponses sur l’agent Defender for IoT.


## <a name="do-i-have-to-install-an-embedded-security-agent"></a>Dois-je installer un agent de sécurité incorporé ?

L’installation de l’agent sur vos appareils IoT n’est pas obligatoire pour activer Defender pour IoT. Vous pouvez choisir entre les trois options suivantes, pour des niveaux de sécurité et des fonctionnalités de surveillance différentes selon votre choix :

- Un déploiement passif, non invasif (sans agent) à l’aide de capteurs NTA (Network Traffic Analysis) pour surveiller et fournir une visibilité complète du risque sur l’IoT/le, sans aucun impact sur les performances du réseau et des appareils
- Installez l’agent Defender pour la sécurité intégrée IoT avec ou sans modifications. Cette option fournit le plus haut niveau d’informations de sécurité améliorées pour le comportement et l’accès à l’appareil.

- Créez votre propre agent et implémentez le schéma de message de sécurité Defender pour IoT. Cette option permet d’utiliser les outils d’analyse de Defender pour IoT en plus de l’agent de sécurité de votre appareil.

- Aucune installation de l’agent de sécurité sur vos appareils IoT. Cette option permet de surveiller les communications d’IoT Hub, avec des fonctionnalités de surveillance et de gestion de la sécurité limitées.

## <a name="what-does-the-defender-for-iot-agent-do"></a>Que fait l’agent Defender pour IoT ?

L’agent Defender pour IoT couvre les menaces au niveau de l’appareil en matière de configuration, de comportement, d’accès (en analysant la configuration), de traitement et de connectivité. L’agent de sécurité Defender pour IoT n’analyse pas les données ou l’activité de l’entreprise.

L’agent de sécurité Defender pour IoT est open source et disponible sur GitHub en versions 32 et 64 bits pour Windows et Linux : https://github.com/Azure/Azure-IoT-Security.


## <a name="what-are-the-dependencies-and-prerequisites-of-the-agent"></a>Quelles sont les dépendances et les conditions requises de l’agent ?

Defender pour IoT prend en charge un vaste éventail de plateformes. Consultez les [plateformes d’appareils pris en charge](how-to-deploy-agent.md) pour vérifier la prise en charge de vos appareils spécifiques.

## <a name="which-data-is-collected-by-the-agent"></a>Quelles données sont collectées par l’agent ?

Des données de connectivité, d’accès, de configuration du pare-feu, de liste de processus et de ligne de base du système d’exploitation sont collectées par l’agent.

## <a name="how-much-data-will-the-agent-generate"></a>Quelle quantité de données l’agent générera-t-il ?

La génération de données de l’agent est déterminée par l’appareil, l’application, le type de connectivité et la configuration de l’agent client. En raison d’une forte variabilité entre les appareils et solutions IoT, nous vous recommandons de commencer par déployer l’agent dans un laboratoire ou un environnement de test pour observer, apprendre et déterminer la configuration spécifique qui répond à vos besoins en mesurant la quantité de données générées. Après le démarrage du service, l’agent Defender pour IoT fournit des recommandations opérationnelles permettant d’optimiser le débit de l’agent pour vous aider dans le processus de configuration et de personnalisation.

## <a name="do-agent-messages-use-up-quota-from-iot-hub"></a>Les messages de l’agent exploitent-ils le quota d’IoT Hub ?

Oui. Les données transmises par l’agent sont comptabilisées dans le quota de votre IoT Hub.

## <a name="what-next-ive-installed-an-agent-and-dont-see-any-activities-or-logs"></a>Étapes suivantes J’ai installé un agent et je ne vois pas d’activités ou de journaux...

1. Vérifiez que le [type d’agent correspond à la plateforme de système d’exploitation désignée de votre appareil](how-to-deploy-agent.md)

1. Confirmez que [l’agent est en cours d’exécution sur l’appareil](how-to-agent-configuration.md).

1. Vérifiez que le [service a été activé avec succès](quickstart-onboard-iot-hub.md) sur **Sécurité** dans votre IoT Hub.

1. Vérifiez que l’appareil est [configuré dans IoT Hub avec le module Defender pour IoT](quickstart-create-security-twin.md).

Si les activités ou les journaux ne sont toujours pas disponibles, contactez votre partenaire Defender pour IoT pour lui demander une aide supplémentaire.

## <a name="what-happens-when-the-internet-connection-stops-working"></a>Que se passe-t-il lorsque la connexion Internet cesse de fonctionner ?

Les capteurs et les agents continuent à s’exécuter et à stocker les données tant que l’appareil est en cours d’exécution. Les données sont stockées dans le cache de messages de sécurité en fonction de la configuration de taille. Lorsque l’appareil retrouve sa connectivité, les messages de sécurité sont à nouveau envoyés.

## <a name="can-the-agent-affect-the-performance-of-the-device-or-other-installed-software"></a>L’agent peut-il affecter les performances de l’appareil ou des autres logiciels installés ?

L’agent consomme des ressources informatiques comme n’importe quel autre processus/application et ne devrait pas perturber l’activité normale de l’appareil. La consommation de ressources sur l’appareil sur lequel l’agent s’exécute dépend de son installation et de sa configuration. Nous vous recommandons de tester votre configuration d’agent dans un environnement contenu, ainsi que son interopérabilité avec les autres applications et fonctionnalités IoT avant de tenter de la déployer dans un environnement de production.

## <a name="im-making-some-maintenance-on-the-device-can-i-turn-off-the-agent"></a>J’effectue une maintenance sur l’appareil. Puis-je désactiver l’agent ?

L’agent ne peut pas être désactivé.

## <a name="is-there-a-way-to-test-if-the-agent-is-working-correctly"></a>Existe-t-il un moyen de vérifier que l’agent fonctionne correctement ?

Si l’agent cesse de communiquer ou ne parvient pas à envoyer des messages de sécurité, une alerte **L’appareil est silencieux** est générée.



## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la prise en main de Defender pour IoT, consultez la documentation suivante :

- Lire la [Vue d’ensemble de Defender pour IoT](overview.md)
- Vérifier les [conditions préalables du service](service-prerequisites.md)
- En savoir plus sur la [prise en main](getting-started.md)
- Comprendre les [Alertes de sécurité de Defender pour IoT](concept-security-alerts.md)
