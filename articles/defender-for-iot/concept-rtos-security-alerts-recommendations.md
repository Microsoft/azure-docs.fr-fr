---
title: Alertes et suggestions personnalisables et intégrées au module de sécurité pour Azure RTOS
description: En savoir plus sur les alertes de sécurité et corrections suggérées lors de l’utilisation du module de sécurité IoT pour Azure RTOS.
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
ms.date: 09/07/2020
ms.author: mlottner
ms.openlocfilehash: cf4924f8a9b97487e64e12ab80df92f2b2a81de2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90930262"
---
# <a name="security-module-for-azure-rtos-security-alerts-and-recommendations-preview"></a>Alertes et suggestions relatives à la sécurité du module de sécurité pour Azure RTOS (préversion)

Le module de sécurité pour Azure RTOS bénéficie de capacités avancées d’analyse des données et de renseignements sur les menaces. Il les exploite pour veiller continuellement au bon fonctionnement de votre solution IoT. Ainsi, il vous avertira s’il détecte une activité potentiellement malveillante ou une modification suspecte du système. Il vous permet également de créer des alertes personnalisées basées sur vos références et vos connaissances sur le comportement prévu pour vos appareils.

Les alertes du module de sécurité pour Azure RTOS révèlent des compromissions potentielles. Vous devez systématiquement les examiner et résoudre les problèmes qu’elles révèlent. Les suggestions du module de sécurité pour Azure RTOS identifient les points faibles de votre posture de sécurité, pour vous permettre d’y répondre en apportant les mises à jour nécessaires. 

Cet article dresse une liste des alertes et des suggestions intégrées au module de sécurité et déclenchées en fonction des réglages par défaut. Vous pouvez les personnaliser avec vos propres valeurs, selon vos références ou le comportement prévu pour vos appareils. 

Si vous souhaitez en savoir plus sur la personnalisation des alertes dans le service Azure Defender pour IoT, consultez l’article [Alertes de sécurité personnalisables d’Azure Defender pour IoT](concept-customizable-security-alerts.md). Les alertes et suggestions spécifiques personnalisables lors de l’utilisation du module de sécurité pour Azure RTOS sont détaillées dans les tableaux suivants. 

## <a name="security-module-for-azure-rtos-supported-security-alerts"></a>Alertes de sécurité prises en charge par le module de sécurité pour Azure RTOS

### <a name="device-related-security-alerts"></a>Alertes de sécurité concernant un appareil

|Activité d’alerte de sécurité concernant un appareil  |Nom de l’alerte  |
|---------|---------|
|Adresse IP| Détection d’une communication avec une adresse IP suspecte|
|Empreinte du certificat de l’appareil X.509|Non-concordance de l’empreinte du certificat de l’appareil X.509|
|Certificat X.509| Le certificat X.509 a expiré|
|Jeton SAP| Jeton SAS expiré|
|Jeton SAP| Signature de jeton SAS non valide|

### <a name="iot-hub-related-security-alerts"></a>Alertes de sécurité concernant un hub IoT

|Activité d’alerte de sécurité concernant un hub IoT  |Nom de l’alerte  |
|---------|---------|
|Ajouter un certificat    |  Détection d’une tentative infructueuse d’ajout de certificat à un hub IoT       |
|Ajout ou modification d’un paramètre de diagnostic    | Détection d’une tentative d’ajout ou de modification d’un paramètre de diagnostic d’un hub IoT      |
|Supprimer un certificat    |  Détection d’une tentative infructueuse de suppression d’un certificat d’un hub IoT       |
|Supprimer un paramètre de diagnostic    |  Détection d’une tentative de suppression d’un paramètre de diagnostic d’un hub IoT      |
|Certificat supprimé    | Détection de la suppression d’un certificat d’un hub IoT        |
|Nouveau certificat     |  Détection de l’ajout d’un nouveau certificat à un hub IoT       |

## <a name="security-module-for-azure-rtos-supported-customizable-alerts"></a>Alertes personnalisables prises en charge par le module de sécurité Azure RTOS

### <a name="device-related-customizable-alerts"></a>Alertes personnalisables concernant un appareil

|Activité concernant un appareil |Nom de l’alerte  |
|---------|---------|
|Connexions actives|Le nombre de connexions actives ne correspond pas aux valeurs autorisées|
|Messages cloud-à-appareil dans le protocole **MQTT**|Le nombre de messages cloud-à-appareil dans le protocole **MQTT** ne correspond pas aux valeurs autorisées|
|Connexion sortante| Une connexion sortante vers une adresse IP qui n’est pas autorisée|

### <a name="hub-related-customizable-alerts"></a>Alertes personnalisables concernant un hub 

|Activité concernant un hub  |Nom de l’alerte  |
|---------|---------|
|Vidages de file d’attente de commandes     |  Le nombre de vidages de file d’attente de commandes ne correspond pas aux valeurs autorisées       |
|Messages cloud-à-appareil dans le protocole **MQTT**    |  Le nombre de messages cloud-à-appareil dans le protocole **MQTT** ne correspond pas aux valeurs autorisées       |
|Messages appareil-à-cloud dans le protocole **MQTT**    | Le nombre de messages appareil-à-cloud dans le protocole **MQTT** ne correspond pas aux valeurs autorisées        |
|Appel de méthode directe     |  Le nombre d’appels de méthode directe ne correspond pas aux valeurs autorisées       |
|Messages cloud-à-appareil rejetés dans le protocole **MQTT**     |   Le nombre de messages cloud-à-appareil rejetés dans le protocole **MQTT** ne correspond pas aux valeurs autorisées      |
|Mises à jour des modules jumeaux     |  Le nombre de mises à jour des modules jumeaux ne correspond pas aux valeurs autorisées       |
|Opérations non autorisées    |  Le nombre d’opérations non autorisées ne correspond pas aux valeurs autorisées       |

## <a name="security-module-for-azure-rtos-supported-recommendations"></a>Suggestions prises en charge par le module de sécurité pour Azure RTOS

### <a name="device-related-recommendations"></a>Suggestions concernant un appareil

|Activité concernant un appareil  |Nom de la recommandation |
|---------|---------|
|Informations d'authentification    |  Informations d’authentification identiques utilisées par plusieurs appareils       |

### <a name="hub-related-recommendations"></a>Suggestions concernant un hub

|Activité concernant un hub IoT  |Nom de la recommandation |
|---------|---------|
|Stratégie de filtre IP   |  La stratégie de filtre IP par défaut doit être définie sur **deny** (refuser)  |
|Règle de filtre IP| La règle de filtre IP inclut une grande plage d’adresses IP|
|Journaux de diagnostics|Suggestion d’activation des journaux de diagnostics dans un hub IoT|

### <a name="all-defender-for-iot-alerts-and-recommendations"></a>Toutes les alertes et suggestions de Defender pour IoT

Pour obtenir la liste des alertes et suggestions du service Defender pour IoT, consultez les pages [Alertes de sécurité de Defender pour IoT](concept-security-alerts.md) et [Suggestions de sécurité](concept-recommendations.md).

## <a name="next-steps"></a>Étapes suivantes

- [Démarrage rapide : Module de sécurité pour Azure RTOS](quickstart-azure-rtos-security-module.md)
- [Configurer et personnaliser le module de sécurité pour Azure RTOS](how-to-azure-rtos-security-module.md)
- Consultez la page [API Module de sécurité pour Azure RTOS](azure-rtos-security-module-api.md).