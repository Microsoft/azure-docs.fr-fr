---
title: Options de déploiement
description: Découvrez le flux de travail de base des fonctionnalités et du service Defender pour IoT.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/09/2020
ms.author: mlottner
ms.openlocfilehash: 6aa525fd7f2d82194baa2e2a0c910cb71509c2d5
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96340014"
---
# <a name="getting-started-with-azure-defender-for-iot"></a>Bien démarrer avec Azure Defender pour IoT

Cet article décrit les processus de déploiement et d’intégration nécessaires pour exécuter Azure Defender pour IoT. Des étapes supplémentaires sont nécessaires. Nous vous recommandons de bien comprendre ces étapes et de vous familiariser avec les informations des documents associés.

Une fois que vous avez terminé toutes les étapes, les capteurs Azure Defender pour IoT surveillent votre réseau. Selon la configuration que vous avez choisie pour votre solution, les détections peuvent également être envoyées à la console de gestion locale ou à l’IoT Hub.

Procédez comme suit pour qu’Azure Defender pour IoT soit opérationnel.

## <a name="1-set-up-azure"></a>1. Configurer Azure

- Configurer un compte Azure. Pour plus d’informations, consultez [Créer un compte Azure](/learn/modules/create-an-azure-account/).

- Pare-feu ou proxy : Si vous avez un pare-feu ou un périphérique réseau intervenant similaire qui est configuré pour autoriser des connexions spécifiques, vérifiez que *.azure-devices.net:443 est ouvert sur le pare-feu ou le proxy. Si les caractères génériques ne sont pas pris en charge ou si vous souhaitez davantage de contrôle, le nom de domaine complet IoT Hub spécifique doit être ouvert dans votre pare-feu ou proxy. Pour plus d’informations, consultez l’article [Référence - Points de terminaison IoT Hub](../iot-hub/iot-hub-devguide-endpoints.md).

## <a name="2-deploy-hardware-software-and-onboard-to-sensor"></a>2. Déployer du matériel, des logiciels et les intégrer au capteur

- Achetez le matériel de capteur et installez le logiciel. Suivez les étapes décrites ici. Pour plus d’informations, consultez cet article et le [Guide du matériel Defender pour IoT](https://aka.ms/AzureDefenderforIoTBareMetalAppliance) ainsi que le [Guide d’installation](https://aka.ms/AzureDefenderforIoTInstallSensorISO).

  - Après avoir installé votre capteur, enregistrez en toute sécurité les informations d’identification de connexion du capteur. Vous aurez besoin des informations d’identification pour charger le fichier d’activation vers le capteur.

  - Si vous travaillez avec des capteurs gérés localement, enregistrez en toute sécurité l’adresse IP du capteur ou le nom du capteur défini lors de l’installation. Vous pouvez en avoir besoin lors de la création d’un nom de capteur pendant l’inscription du capteur dans le portail Defender pour IoT. Plus tard, vous pouvez les utiliser pour faciliter le suivi et la cohérence du nommage entre le nom d’inscription sur le portail Azure Defender pour IoT et l’adresse IP du capteur déployé affiché dans la console de capteur.

- Inscrivez le capteur sur le portail Defender pour IoT et téléchargez un fichier d’activation de capteur.

- Charger le fichier d’activation sur votre capteur.

## <a name="3-perform-network-setup-for-sensor-monitoring-and-management"></a>3. Configurer le réseau pour la surveillance et la gestion du capteur

- Connectez votre capteur au réseau. Décrit dans le [Guide de configuration du réseau](https://aka.ms/AzureDefenderForIoTNetworkSetup).

## <a name="4-start-discovering-your-network"></a>4. Découvrir votre réseau

- Ajustez les paramètres système dans la console du capteur.

- Connectez les capteurs à une console de gestion locale.

Pour plus d’informations, reportez-vous au [Guide de l’utilisateur des capteurs Azure Defender pour IoT](https://aka.ms/AzureDefenderforIoTUserGuide) ainsi qu’au [Guide de l’utilisateur de la console de gestion locale Defender pour IoT](https://aka.ms/DefenderForIoTManagementConsole).

## <a name="5-populate-azure-sentinel-with-alert-information"></a>5. Remplir Azure Sentinel avec les informations d’alerte

- Pour envoyer des informations d’alerte à Azure Sentinel, configurez Azure Sentinel : [Connectez vos données Defender pour IoT à Azure Sentinel](how-to-configure-with-sentinel.md).
 

## <a name="next-steps"></a>Étapes suivantes

- Activer [Defender pour IoT](quickstart-onboard-iot-hub.md)
- Configurer une [solution](quickstart-configure-your-solution.md)
- [Créer des modules de sécurité](quickstart-create-security-twin.md)
- Configurer des [alertes personnalisées](quickstart-create-custom-alerts.md)
- [Déployer un agent de sécurité](how-to-deploy-agent.md)