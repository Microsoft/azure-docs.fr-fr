---
title: Configurer des agents de sécurité
description: Découvrez comment configurer des agents de sécurité Defender pour IoT en vue de les utiliser avec ce service.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/09/2020
ms.author: mlottner
ms.openlocfilehash: aa863ce556840903d16238d6afef32136ba2b80d
ms.sourcegitcommit: 30505c01d43ef71dac08138a960903c2b53f2499
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92090843"
---
# <a name="tutorial-configure-security-agents"></a>Tutoriel : Configurer des agents de sécurité

Cet article vous fournit des explications sur les agents de sécurité Defender pour IoT et vous indique la manière dont vous pouvez les modifier et les configurer.

> [!div class="checklist"]
> * Configurer des agents de sécurité
> * Modifier le comportement de l’agent en modifiant les propriétés de jumeaux
> * Découvrir la configuration par défaut

## <a name="agents"></a>Agents

Les agents de sécurité Defender pour IoT collectent des données à partir d’appareils IoT et effectuent des actions de sécurité pour atténuer les vulnérabilités détectées. La configuration de l’agent de sécurité peut être contrôlée à l’aide d’un ensemble de propriétés de jumeau de module que vous pouvez personnaliser. En règle générale, les mises à jour secondaires pour ces propriétés sont peu fréquentes.

L’objet de configuration des jumeaux d’agent de sécurité Defender pour IoT est un objet au format JSON. L’objet de configuration est un ensemble de propriétés contrôlables que vous pouvez définir pour contrôler le comportement de l’agent.

Ces configurations vous aident à personnaliser l’agent pour chaque scénario requis. Par exemple, l’exclusion automatique de certains événements, ou le maintien de la consommation d’énergie à un niveau minimal sont possibles en configurant ces propriétés.

Utilisez le [schéma](https://aka.ms/iot-security-github-module-schema) de configuration de l’agent de sécurité Defender pour IoT pour apporter des modifications.

## <a name="configuration-objects"></a>Objets de configuration

Chaque propriété liée à l’agent de sécurité Defender pour IoT se trouve dans l’objet de configuration de l’agent, dans la section Propriétés souhaitées du module **azureiotsecurity**.

Pour modifier la configuration, créez et modifiez cet objet à l’intérieur de l’identité de jumeau de module **azureiotsecurity**.

Si l’objet de configuration de l’agent n’existe pas dans le jumeau de module **azureiotsecurity**, toutes les valeurs de propriété de l’agent de sécurité sont définies aux valeurs par défaut.

```json
"desired": {
  "ms_iotn:urn_azureiot_Security_SecurityAgentConfiguration": {
  }
}
```

## <a name="configuration-schema-and-validation"></a>Schéma de configuration et validation

Veillez à valider votre configuration d’agent par rapport à ce [schéma](https://aka.ms/iot-security-github-module-schema). Un agent ne se lancera pas si l’objet de configuration ne correspond pas au schéma.

Si, lorsque l’agent est en cours d’exécution, l’objet de configuration est modifié en une configuration non valide (la configuration ne correspond pas au schéma), l’agent ignore la configuration non valide et continue avec la configuration actuelle.

### <a name="configuration-validation"></a>Validation de configuration

L’agent de sécurité Defender pour IoT signale sa configuration actuelle dans la section Propriétés signalées de l’identité du jumeau de module **azureiotsecurity**.
L’agent fait état de toutes les propriétés disponibles. Si une propriété n’a pas été définie par l’utilisateur, l’agent indique la configuration par défaut.

Pour valider votre configuration, comparez les valeurs définies dans la section souhaitée avec celles indiquées dans la section signalée.

En cas d’incohérence entre les propriétés souhaitées et les propriétés signalées, l’agent ne peut pas analyser la configuration.

Validez les propriétés souhaitées par rapport au [schéma](https://aka.ms/iot-security-github-module-schema), corrigez les erreurs et définissez à nouveau les propriétés souhaitées.

> [!NOTE]
> L’agent envoie une alerte d’erreur de configuration s’il ne peut pas analyser la configuration souhaitée.
> Comparez les sections signalées et souhaitées pour déterminer si l’alerte est toujours valable

## <a name="editing-a-property"></a>Modification d’une propriété

Toutes les propriétés personnalisées doivent être définies à l’intérieur de l’objet de configuration de l’agent dans le jumeau de module **azureiotsecurity**.
Pour utiliser une valeur de propriété par défaut, supprimez la propriété de l’objet de configuration.

### <a name="setting-a-property"></a>Définition d’une propriété

1. Dans votre IoT Hub, recherchez et sélectionnez l’appareil que vous souhaitez modifier.

1. Cliquez sur votre appareil, puis sur le module **azureiotsecurity**.

1. Cliquez sur **Jumeau d’identité de module**.

1. Modifiez les propriétés que vous souhaitez changer dans le module sécurité.

   Par exemple, pour configurer les événements de connexion sur une priorité élevée et collecter les événements de haute priorité toutes les 7 minutes, utilisez la configuration suivante.

    ```json
    "desired": {
        "ms_iotn:urn_azureiot_Security_SecurityAgentConfiguration": {
            "highPriorityMessageFrequency": {
                "value": "PT7M"
            },
            "eventPriorityConnectionCreate": {
                "value": "High"
            }
        }
    }
    ```

1. Cliquez sur **Enregistrer**.

### <a name="using-a-default-value"></a>Avec une valeur par défaut

Pour utiliser une valeur de propriété par défaut, supprimez la propriété de l’objet de configuration.

## <a name="default-properties"></a>Propriétés par défaut

Le tableau suivant contient les propriétés contrôlables des agents de sécurité Defender pour IoT.

Les valeurs par défaut sont disponibles dans le schéma approprié dans [Github](https\://aka.ms/iot-security-module-default).

| Nom| Statut | Valeurs valides| Valeurs par défaut| Description |
|----------|--------|--|-------|----|
|highPriorityMessageFrequency|Requis : false |Valeurs valides : Durée au format ISO 8601 |Valeur par défaut : PT7M |Intervalle de temps maximale avant l’envoi de messages de priorité élevée.|
|lowPriorityMessageFrequency |Requis : false|Valeurs valides : Durée au format ISO 8601 |Valeur par défaut : PT5H |Durée maximale avant l’envoi de message de basse priorité.|
|snapshotFrequency |Requis : false|Valeurs valides : Durée au format ISO 8601 |La valeur par défaut est PT13H |Intervalle de temps pour la création d’instantanés d’état d’appareil.|
|maxLocalCacheSizeInBytes |Requis : false |Valeurs valides : |Valeur par défaut : 2560000, supérieur à 8192 | Stockage maximal (en octets) autorisé pour le cache de messages d’un agent. Quantité maximale d’espace pour le stockage des messages sur l’appareil avant leur envoi.|
|maxMessageSizeInBytes |Requis : false |Valeurs valides : Un nombre positif, supérieur à 8192 et inférieur à 262144 |Valeur par défaut : 204800 |Taille maximale autorisée d’un message d’agent au cloud. Ce paramètre contrôle la quantité maximale de données envoyées dans chaque message. |
|eventPriority${EventName} |Requis : false |Valeurs valides : High, Low, Off |Valeurs par défaut : |Priorité de chaque événement généré par l’agent |

### <a name="supported-security-events"></a>Événements de sécurité pris en charge

|Nom d'événement| PropertyName | Valeur par défaut| Événement de capture instantanée| État des détails  |
|----------|-|---------|----|----|
|Événement de diagnostic|eventPriorityDiagnostic| Off| False| Événements de diagnostic liés à l’agent. Utilisez cet événement pour la journalisation détaillée.|
|Erreur de configuration |eventPriorityConfigurationError |Faible |False |L’agent a échoué à analyser la configuration. Vérifiez la configuration par rapport au schéma.|
|Statistiques des événements annulés |eventPriorityDroppedEventsStatistics |Faible |True|Statistiques des événements liés à l’agent. |
|Matériel connecté|eventPriorityConnectedHardware |Faible |True |Instantané de tout le matériel connecté à l’appareil.|
|Ports d’écoute|eventPriorityListeningPorts |Élevé |True |Instantané de tous les ports d’écoute sur l’appareil.|
|Création de processus |eventPriorityProcessCreate |Faible |False |Vérifie la création de processus sur l’appareil.|
|Arrêt de processus|eventPriorityProcessTerminate |Faible |False |Vérifie l’arrêt de processus sur l’appareil.|
|Informations système |eventPrioritySystemInformation |Faible |True |Un instantané des informations système (par exemple : Système d’exploitation ou processeur).|
|Utilisateurs locaux| eventPriorityLocalUsers |Élevé |True|Une capture instantanée des utilisateurs locaux inscrits sur le système. |
|Connexion|  eventPriorityLogin |Élevé|False|Vérification des événements de connexion à l’appareil (accès locaux et distants).|
|Création de connexion |eventPriorityConnectionCreate|Faible|False|Vérification des connexions TCP créées de et vers l’appareil. |
|Configuration du pare-feu| eventPriorityFirewallConfiguration|Faible|True|Instantané de la configuration du pare-feu de l’appareil (règles de pare-feu). |
|Base de référence système d’exploitation| eventPriorityOSBaseline| Faible|True|Instantané de vérification de ligne de base du système d’exploitation de l’appareil.|
|

## <a name="next-steps"></a>Étapes suivantes

- [Comprendre les recommandations Defender pour IoT](concept-recommendations.md)
- [Explorer les alertes Defender pour IoT](concept-security-alerts.md)
- [Accéder aux données de sécurité brutes](how-to-security-data-access.md)
