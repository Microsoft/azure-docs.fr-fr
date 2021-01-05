---
title: Événements planifiés pour les machines virtuelles Linux dans Azure
description: Planifiez des événements en utilisant le service de métadonnées Azure pour vos machines virtuelles Linux.
author: EricRadzikowskiMSFT
ms.service: virtual-machines-linux
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 06/01/2020
ms.author: ericrad
ms.reviewer: mimckitt
ms.openlocfilehash: 99528d1575056917b68bcb38f41a24d065822827
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92792801"
---
# <a name="azure-metadata-service-scheduled-events-for-linux-vms"></a>Service de métadonnées Azure : événements planifiés pour les machines virtuelles Linux

Événements planifiés est un service de métadonnées Azure qui permet à votre application de disposer de suffisamment de temps pour se préparer à la maintenance des machines virtuelles. Il fournit des informations sur les événements de maintenance à venir (par exemple, les redémarrages), afin que votre application puisse s’y préparer et limiter les interruptions de service. Il est disponible pour tous les types de machines virtuelles Azure, notamment PaaS et IaaS sur Windows et Linux. 

Pour plus d’informations sur les événements planifiés sur Windows, consultez [Événements planifiés pour les machines virtuelles Windows](../windows/scheduled-events.md).

> [!Note] 
> Le service Événements planifiés est mis à la disposition générale dans toutes les régions Azure. Consultez la section [Version et disponibilité dans la région](#version-and-region-availability) pour obtenir des informations sur la version la plus récente.

## <a name="why-use-scheduled-events"></a>Pourquoi utiliser le service Événements planifiés ?

De nombreuses applications peuvent bénéficier d’un délai pour se préparer à la maintenance des machines virtuelles. Ce délai peut servir à effectuer des tâches propres aux applications qui améliorent la disponibilité, la fiabilité et la facilité de maintenance, notamment : 

- Point de contrôle et restauration
- Vidage des connexions
- Basculement du réplica principal
- Suppression à partir du pool d’équilibreur de charge
- Journalisation des événements
- Arrêt approprié

Avec le service Événements planifiés, votre application peut savoir quand une maintenance a lieu et déclencher des tâches pour limiter son impact.  

Le service Événements planifiés fournit des événements dans les cas d’usage suivants :

- [Maintenance lancée par la plateforme](../maintenance-and-updates.md?bc=/azure/virtual-machines/linux/breadcrumb/toc.json&toc=/azure/virtual-machines/linux/toc.json) (par exemple, redémarrage de machine virtuelle, migration dynamique ou mémoire conservant les mises à jour pour l’hôte)
- La machine virtuelle est en cours d’exécution sur le [matériel hôte détérioré](https://azure.microsoft.com/blog/find-out-when-your-virtual-machine-hardware-is-degraded-with-scheduled-events) dont la défaillance prochaine est prédite
- L’utilisateur a lancé une maintenance (par exemple, un utilisateur redémarre ou redéploie une machine virtuelle).
- Évictions d’instances de [machine virtuelle Spot](../spot-vms.md) et de [groupe de machines virtuelles identiques Spot](../../virtual-machine-scale-sets/use-spot.md).

## <a name="the-basics"></a>Concepts de base  

  Le service de métadonnées expose des informations sur les machines virtuelles en cours d’exécution en utilisant un point de terminaison REST accessible depuis la machine virtuelle. Ces informations sont disponibles via une adresse IP non routable, de façon à ce qu’elles ne soient pas exposées en dehors de la machine virtuelle.

### <a name="scope"></a>Étendue
Les événements planifiés sont remis à :

- Machines virtuelles autonomes.
- Toutes les machines virtuelles d’un service cloud
- Toutes les machines virtuelles d’un groupe à haute disponibilité
- Toutes les machines virtuelles d’une zone à haute disponibilité. 
- Toutes les machines virtuelles d’un groupe de placement de groupe identique 

> [!NOTE]
> Les événements planifiés, spécifiques aux machines virtuelles dans une zone de disponibilité, sont dirigés vers des machines virtuelles uniques d’une zone.
> Par exemple, si vous avez 100 machines virtuelles dans un groupe à haute disponibilité et que l’une d’elles doit être mise à jour, l’événement planifié les concernera toutes, tandis que s’il y a 100 machines virtuelles uniques dans une zone, l’événement concernera uniquement la machine virtuelle impactée.

Par conséquent, vérifiez le champ `Resources` de l’événement pour identifier les machines virtuelles concernées.

### <a name="endpoint-discovery"></a>Découverte de point de terminaison
Pour les machines virtuelles compatibles avec le réseau virtuel, le service de métadonnées est disponible à partir d’une adresse IP non routable statique, `169.254.169.254`. Le point de terminaison complet de la dernière version des événements planifiés est : 

 > `http://169.254.169.254/metadata/scheduledevents?api-version=2019-08-01`

Si la machine virtuelle n’est pas créée au sein d’un réseau virtuel, ce qui est habituellement le cas pour les services cloud et les machines virtuelles classiques, une logique supplémentaire est nécessaire pour découvrir l’adresse IP à utiliser. Reportez-vous à cet exemple pour savoir comment [découvrir le point de terminaison hôte](https://github.com/azure-samples/virtual-machines-python-scheduled-events-discover-endpoint-for-non-vnet-vm).

### <a name="version-and-region-availability"></a>Version et disponibilité dans la région
Les versions du service Événements planifiés sont gérées. Ces versions sont obligatoires et la version actuelle est `2019-01-01`.

| Version | Type de version | Régions | Notes de publication | 
| - | - | - | - | 
| 2019-08-01 | Disponibilité générale | Tous | <li> Ajout de la prise en charge pour EventSource |
| 2019-04-01 | Disponibilité générale | Tous | <li> Ajout de la prise en charge pour Description de l’événement |
| 2019-01-01 | Disponibilité générale | Tous | <li> Ajout de la prise en charge des groupes de machines virtuelles identiques, EventType « Terminate » |
| 2017-11-01 | Disponibilité générale | Tous | <li> Ajout de la prise en charge de l’éviction de machine virtuelle Spot, EventType « Preempt »<br> | 
| 2017-08-01 | Disponibilité générale | Tous | <li> Suppression du trait de soulignement ajouté au début des noms de ressources pour les machines virtuelles IaaS<br><li>Spécification d’en-tête de métadonnées appliquée à toutes les requêtes | 
| 2017-03-01 | PRÉVERSION | Tous | <li>Version initiale |


> [!NOTE] 
> Les préversions précédentes du service Evénements planifiés prenaient en charge {latest} en tant que version de l’api. Ce format n’est plus pris en charge et sera déconseillé à l’avenir.

### <a name="enabling-and-disabling-scheduled-events"></a>Activation et désactivation du service Événements planifiés
Le service Événements planifiés est activé pour votre service la première fois que vous faites une requête d’événements. Attendez-vous à ce que la réponse à votre première demande ait un retard pouvant atteindre deux minutes.

Le service Événements planifiés est désactivé pour votre service, s’il ne fait aucune requête pendant 24 heures.

### <a name="user-initiated-maintenance"></a>Maintenance lancée par l’utilisateur
La maintenance de machine virtuelle lancée par l’utilisateur via le portail Azure, l’API, l’interface de ligne de commande ou PowerShell entraîne un événement planifié. Cela vous permet de tester la logique de préparation de la maintenance dans votre application et à cette dernière de se préparer à la maintenance lancée par l’utilisateur.

Si vous redémarrez une machine virtuelle, un événement de type `Reboot` est planifié. Si vous redéployez une machine virtuelle, un événement de type `Redeploy` est planifié.

## <a name="use-the-api"></a>Utilisation de l’API

### <a name="headers"></a>headers
Quand vous interrogez le service de métadonnées, vous devez fournir l’en-tête `Metadata:true` pour garantir que la requête n’a pas été redirigée involontairement. L’en-tête `Metadata:true` est obligatoire pour toutes les requêtes d’événements planifiés. L’absence d’en-tête dans la requête génère une réponse « Requête incorrecte » du service de métadonnées.

### <a name="query-for-events"></a>Rechercher des événements
Vous pouvez rechercher des événements planifiés en effectuant l’appel suivant :

#### <a name="bash"></a>Bash
```
curl -H Metadata:true http://169.254.169.254/metadata/scheduledevents?api-version=2019-08-01
```

Une réponse contient un tableau d’événements planifiés. Un tableau vide signifie qu’il n’y a actuellement aucun événement planifié.
S’il existe des événements planifiés, la réponse contient un tableau d’événements. 
```
{
    "DocumentIncarnation": {IncarnationID},
    "Events": [
        {
            "EventId": {eventID},
            "EventType": "Reboot" | "Redeploy" | "Freeze" | "Preempt" | "Terminate",
            "ResourceType": "VirtualMachine",
            "Resources": [{resourceName}],
            "EventStatus": "Scheduled" | "Started",
            "NotBefore": {timeInUTC},       
            "Description": {eventDescription},
            "EventSource" : "Platform" | "User",
        }
    ]
}
```

### <a name="event-properties"></a>Propriétés de l’événement
|Propriété  |  Description |
| - | - |
| EventId | GUID pour cet événement. <br><br> Exemple : <br><ul><li>602d9444-d2cd-49c7-8624-8643e7171297  |
| Type d’événement | Impact provoqué par cet événement. <br><br> Valeurs : <br><ul><li> `Freeze`: une pause de quelques secondes est planifiée pour la machine virtuelle. L’UC et la connectivité réseau peuvent être suspendus, mais cela n’a aucun impact sur la mémoire ni sur les fichiers ouverts.<li>`Reboot`: un redémarrage est planifié pour la machine virtuelle (la mémoire non persistante est effacée). <li>`Redeploy`: un déplacement vers un autre nœud est planifié pour la machine virtuelle (le contenu des disques éphémères est perdu). <li>`Preempt`: la machine virtuelle Spot est supprimée (le contenu des disques éphémères est perdu). <li> `Terminate`: La suppression de la machine virtuelle est planifiée. |
| ResourceType | Type de ressource affecté par cet événement. <br><br> Valeurs : <ul><li>`VirtualMachine`|
| Ressources| Liste de ressources affectée par cet événement. Elle contient à coup sûr des machines d’au plus un [domaine de mise à jour](../manage-availability.md), mais elle peut tout aussi bien ne pas contenir toutes les machines de ce domaine. <br><br> Exemple : <br><ul><li> ["FrontEnd_IN_0", "BackEnd_IN_0"] |
| EventStatus | État de cet événement. <br><br> Valeurs : <ul><li>`Scheduled`: cet événement est planifié pour démarrer après l’heure spécifiée dans la propriété `NotBefore`.<li>`Started`: cet événement a démarré.</ul> Aucun état `Completed` ou similaire n’est fourni. L’événement n’est plus renvoyé lorsqu’il est terminé.
| NotBefore| Heure après laquelle cet événement peut démarrer. <br><br> Exemple : <br><ul><li> Lundi 19 septembre 2016 18:29:47 GMT  |
| Description | Description de cet événement. <br><br> Exemple : <br><ul><li> Le serveur hôte est en cours de maintenance. |
| EventSource | Initiateur de l’événement. <br><br> Exemple : <br><ul><li> `Platform`: Cet événement est déclenché par la plateforme. <li>`User`: Cet événement est déclenché par l’utilisateur. |

### <a name="event-scheduling"></a>Planification d’événement
Chaque événement est planifié à un moment donné dans le futur (délai minimum), en fonction de son type. Cette heure est reflétée dans la propriété `NotBefore` d’un événement. 

|Type d’événement  | Préavis minimal |
| - | - |
| Freeze| 15 minutes |
| Reboot | 15 minutes |
| Redeploy | 10 minutes |
| Preempt | 30 secondes |
| Terminate | [Configurable par l’utilisateur](../../virtual-machine-scale-sets/virtual-machine-scale-sets-terminate-notification.md#enable-terminate-notifications) : 5 à 15 minutes |

> [!NOTE] 
> Dans certains cas, Azure est en mesure de prédire une défaillance de l’hôte en raison d’un matériel détérioré, et tente d’atténuer l’interruption du service en planifiant une migration. Les machines virtuelles affectées recevront un événement planifié avec une valeur de temps `NotBefore` généralement définie sur quelques jours dans le futur. La valeur de temps réelle varie en fonction de l’évaluation du risque de la défaillance prédite. Autant que possible, Azure tente de donner un préavis de 7 jours, mais le temps réel varie et peut être inférieur si la prédiction indique qu’il y a de fortes chances que la défaillance du matériel soit imminente. Pour minimiser les risques auxquels votre service serait exposé en cas de défaillance du matériel avant la migration lancée par le système, nous vous recommandons d’auto-redéployer votre machine virtuelle dès que possible.

### <a name="polling-frequency"></a>Fréquence d’interrogation

Vous pouvez rechercher des mises à jour sur le point de terminaison à la fréquence de votre choix. Toutefois, plus l’intervalle entre deux requêtes est long, plus vous perdez potentiellement du temps pour réagir à un événement à venir. La plupart des événements préviennent de 5 à 15 minutes à l’avance, bien que dans certains cas, 30 secondes suffisent. Pour être sûr de disposer du maximum de temps pour prendre des mesures d’atténuation, nous vous recommandons d’interroger le service une fois pas seconde.

### <a name="start-an-event"></a>Démarrer un événement 

Une fois que vous avez pris connaissance d’un événement à venir et que vous avez mis en place une logique d’arrêt appropriée, vous pouvez approuver l’événement en suspens en effectuant un appel `POST` au service de métadonnées avec `EventId`. Cet appel indique à Azure qu’il peut raccourcir l’heure de notification minimale (quand c’est possible). 

Voici un exemple de code JSON attendu dans le corps de la requête `POST`. La requête doit contenir une liste de `StartRequests`. Chaque `StartRequest` contient le paramètre `EventId` correspondant à l’événement que vous souhaitez envoyer :
```
{
    "StartRequests" : [
        {
            "EventId": {EventId}
        }
    ]
}
```

#### <a name="bash-sample"></a>Exemple Bash
```
curl -H Metadata:true -X POST -d '{"StartRequests": [{"EventId": "f020ba2e-3bc0-4c40-a10b-86575a9eabd5"}]}' http://169.254.169.254/metadata/scheduledevents?api-version=2019-01-01
```

> [!NOTE] 
> Le fait d’accuser réception d’un événement lui permet de poursuivre pour tous les éléments `Resources` qu’il contient, et pas seulement pour la machine virtuelle qui en accuse réception. Vous pouvez donc choisir de désigner une amorce de début pour coordonner l’accusé de réception, qui peut être simplement la première machine du champ `Resources`.

## <a name="python-sample"></a>Exemple de code Python 

L’exemple suivant interroge le service de métadonnées pour obtenir les événements planifiés et approuve chaque événement en suspens :

```python
#!/usr/bin/python

import json
import socket
import urllib2

metadata_url = "http://169.254.169.254/metadata/scheduledevents?api-version=2019-08-01"
this_host = socket.gethostname()


def get_scheduled_events():
    req = urllib2.Request(metadata_url)
    req.add_header('Metadata', 'true')
    resp = urllib2.urlopen(req)
    data = json.loads(resp.read())
    return data


def handle_scheduled_events(data):
    for evt in data['Events']:
        eventid = evt['EventId']
        status = evt['EventStatus']
        resources = evt['Resources']
        eventtype = evt['EventType']
        resourcetype = evt['ResourceType']
        notbefore = evt['NotBefore'].replace(" ", "_")
    description = evt['Description']
    eventSource = evt['EventSource']
        if this_host in resources:
            print("+ Scheduled Event. This host " + this_host +
                " is scheduled for " + eventtype + 
        " by " + eventSource + 
        " with description " + description +
        " not before " + notbefore)
            # Add logic for handling events here


def main():
    data = get_scheduled_events()
    handle_scheduled_events(data)


if __name__ == '__main__':
    main()
```

## <a name="next-steps"></a>Étapes suivantes 
- Regardez la vidéo sur le service [Événements planifiés sur Azure Friday](https://channel9.msdn.com/Shows/Azure-Friday/Using-Azure-Scheduled-Events-to-Prepare-for-VM-Maintenance) pour voir une démonstration. 
- Passez en revue les exemples de code d’événements planifiés disponibles dans le [dépôt GitHub d’événements planifiés de métadonnées d’instance Azure](https://github.com/Azure-Samples/virtual-machines-scheduled-events-discover-endpoint-for-non-vnet-vm).
- Apprenez-en davantage sur les API disponibles dans le [service de métadonnées d’instance](instance-metadata-service.md).
- Découvrez plus d’informations sur la [maintenance planifiée pour les machines virtuelles Linux dans Azure](../maintenance-and-updates.md?bc=/azure/virtual-machines/linux/breadcrumb/toc.json&toc=/azure/virtual-machines/linux/toc.json).