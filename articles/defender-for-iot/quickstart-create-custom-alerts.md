---
title: 'Démarrage rapide : Créer des alertes personnalisées'
description: Étudiez, créez et affectez des alertes d’appareil personnalisées destinées au service de sécurité Azure Defender pour IoT.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/04/2020
ms.author: mlottner
ms.openlocfilehash: 48682e465374c1a0e1fb74fc6627016696ff6d2c
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "90945082"
---
# <a name="quickstart-create-custom-alerts"></a>Démarrage rapide : Créer des alertes personnalisées

Les alertes et groupes de sécurité personnalisés vous permettent de tirer pleinement parti des informations de sécurité de bout en bout et des connaissances catégorisées sur les appareils pour assurer une plus grande sécurité dans votre solution IoT.

## <a name="why-use-custom-alerts"></a>Pourquoi utiliser des alertes personnalisées ?

Vous connaissez vos appareils IoT mieux que personne.

Pour les clients qui connaissent très bien le comportement attendu de leurs appareils, Defender pour IoT permet de traduire cette connaissance en stratégie de comportement des appareils et de générer des alertes sur les écarts enregistrés par rapport au comportement normal et attendu.

## <a name="security-groups"></a>Groupes de sécurité

Les groupes de sécurité vous permettent de définir des groupes logiques d’appareils et de gérer leur état de sécurité de façon centralisée.

Ces groupes peuvent représenter des appareils équipés d’un matériel particulier, des appareils déployés dans un emplacement donné, en fait tout groupe répondant à des besoins spécifiques.

Les groupes de sécurité sont définis par une propriété d’étiquette de jumeau d'appareil nommée **SecurityGroup**. Par défaut, chaque solution IoT sur IoT Hub dispose d’un groupe de sécurité nommé **par défaut**. Modifiez la valeur de la propriété **SecurityGroup** pour modifier le groupe de sécurité d’un appareil.

Par exemple :

```
{
  "deviceId": "VM-Contoso12",
  "etag": "AAAAAAAAAAM=",
  "deviceEtag": "ODA1BzA5QjM2",
  "status": "enabled",
  "statusUpdateTime": "0001-01-01T00:00:00",
  "connectionState": "Disconnected",
  "lastActivityTime": "0001-01-01T00:00:00",
  "cloudToDeviceMessageCount": 0,
  "authenticationType": "sas",
  "x509Thumbprint": {
    "primaryThumbprint": null,
    "secondaryThumbprint": null
  },
  "version": 4,
  "tags": {
    "SecurityGroup": "default"
  },
```

Utilisez des groupes de sécurité pour regrouper vos appareils par catégories logiques. Après avoir créé les groupes, affectez-les aux alertes personnalisées de votre choix, pour utiliser la solution de sécurité IoT de bout en bout la plus efficace.

## <a name="customize-an-alert"></a>Personnaliser une alerte

1. Ouvrez votre hub IoT et sélectionnez **Paramètres** dans le menu **Sécurité**. 
1. Cliquez sur **Alertes personnalisées**.
1. Choisissez un groupe de sécurité auquel vous souhaitez appliquer la personnalisation.
1. Cliquez sur **Ajouter une alerte personnalisée**.
1. Sélectionnez une alerte personnalisée dans la liste déroulante.
1. Modifiez les propriétés nécessaires, puis cliquez sur **OK**.
1. N’oubliez pas de cliquer sur **ENREGISTRER**. Si la nouvelle alerte n’est pas enregistrée, elle est supprimée la prochaine fois que vous fermez IoT Hub.

## <a name="alerts-available-for-customization"></a>Alertes disponibles pour la personnalisation

Defender pour IoT propose un grand nombre d’alertes qui peuvent être personnalisées en fonction de vos besoins spécifiques. Consultez le [tableau d’alertes personnalisables](concept-customizable-security-alerts.md) pour connaître la gravité des alertes, la source de données et la description ainsi que les étapes de correction que nous suggérons, si et quand chaque alerte est reçue.

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant pour savoir comment déployer un agent de sécurité...

> [!div class="nextstepaction"]
> [Déployer un agent de sécurité](how-to-deploy-agent.md)
