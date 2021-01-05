---
title: Introduction à la journalisation des flux pour les groupes de sécurité réseau
titleSuffix: Azure Network Watcher
description: Cet article explique comment utiliser les journaux de flux de groupe de sécurité réseau, une fonctionnalité d’Azure Network Watcher.
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: damendo
ms.openlocfilehash: 79f442c5ab7db92e69f5396f3f9205212bdf4d4d
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97399245"
---
# <a name="introduction-to-flow-logging-for-network-security-groups"></a>Présentation de la journalisation des flux pour les groupes de sécurité réseau

## <a name="introduction"></a>Introduction

Les journaux de flux de [groupe de sécurité réseau](../virtual-network/network-security-groups-overview.md#security-rules) (NSG) sont une fonctionnalité d’Azure Network Watcher qui vous permet de journaliser des informations sur le trafic IP circulant dans un NSG. Les données du flux sont envoyées aux comptes de stockage Azure à partir desquels vous pouvez y accéder et les exporter vers les outils de visualisation, applications de gestion des événements et informations de sécurité (SIEM, Security Information and Event Management) ou systèmes de détection d’intrusion (IDS, Intrusion Detection System) de votre choix.

![vue d’ensemble des journaux de flux](./media/network-watcher-nsg-flow-logging-overview/homepage.jpg)

## <a name="why-use-flow-logs"></a>Pourquoi utiliser des journaux de flux ?

Il est essentiel de surveiller, gérer et connaître votre propre réseau pour bénéficier de performances, d’une conformité et d’une sécurité optimales. Connaître votre propre environnement est d’une importance capitale si vous souhaitez le protéger et l’optimiser. Vous devez régulièrement connaître l’état actuel du réseau, savoir qui se connecte depuis quel emplacement, quels sont les ports ouverts sur Internet, quel est le comportement réseau attendu et quelles sont les hausses soudaines du trafic.

Les journaux de flux sont la source fidèle pour toute activité réseau au sein de votre environnement cloud. Que vous soyez une start-up cherchant à optimiser ses ressources ou une grande entreprise tentant de détecter une intrusion, les journaux de flux constituent votre meilleur atout. Ils vous permettent d’optimiser les flux réseau, de surveiller le débit, de vérifier la conformité, de détecter les intrusions et bien plus encore.

## <a name="common-use-cases"></a>Cas d’utilisation courants

**Supervision réseau** : Identifiez le trafic inconnu ou indésirable. Surveillez les niveaux de trafic et l’utilisation de la bande passante. Filtrer les journaux de flux par adresse IP et par port pour comprendre le comportement de l’application. Exportez les journaux de flux vers les outils d’analyse et de visualisation de votre choix pour configurer des tableaux de bord de surveillance.

**Supervision et optimisation de l’utilisation :** Identifiez les principaux acteurs de votre réseau. Exploitez les données de géolocalisation des adresses IP (GeoIP) pour identifier le trafic entre les régions. Acquérez une compréhension de la croissance du trafic pour la prévision de capacité. Utilisez les données pour supprimer les règles de trafic visiblement restrictives.

**Conformité** : Utilisez les données de flux pour vérifier l’isolement réseau et la conformité avec les règles d’accès de l’entreprise.

**Investigations sur le réseau et analyse de sécurité** : Analysez les flux réseau des adresses IP et des interfaces réseau compromises. Exportez les journaux de flux vers les outils de SIEM ou IDS de votre choix.

## <a name="how-logging-works"></a>Fonctionnement de la journalisation

**Propriétés clés**

- Les journaux de flux opèrent au niveau de la [couche 4](https://en.wikipedia.org/wiki/OSI_model#Layer_4:_Transport_Layer) et enregistrent tous les flux IP entrants et sortants d’un groupe de sécurité réseau.
- Les journaux sont collectés **à un intervalle d’une minute** via la plateforme Azure et n’affectent en aucune façon les ressources du client ou les performances réseau.
- Les journaux sont écrits au format JSON et affichent les flux entrants et sortants conformément à une règle de groupe de sécurité réseau.
- Chaque enregistrement de journal contient l’interface réseau (NIC) à laquelle le flux s’applique, les informations de quintuplet, ainsi que les informations sur le débit (version 2 uniquement) et les décisions de trafic. Pour plus d’informations, consultez _Format de journal_ ci-dessous.
- Les journaux de flux intègrent une fonctionnalité de rétention qui permet de supprimer automatiquement les journaux jusqu’à un an après leur création. 

> [!NOTE]
> La rétention n’est disponible que si vous utilisez des [comptes de stockage à usage général v2 (GPv2)](../storage/common/storage-account-overview.md#types-of-storage-accounts). 

**Concepts fondamentaux**

- Des réseaux définis par logiciel sont organisés autour de réseaux virtuels et de sous-réseaux. La sécurité de ces réseaux virtuels et sous-réseaux peut être gérée à l’aide d’un groupe de sécurité réseau.
- Un groupe de sécurité réseau contient une liste de _règles de sécurité_ qui autorisent ou refusent le trafic réseau dans les ressources auxquelles il est connecté. Des groupes de sécurité réseau peuvent être associés à des sous-réseaux, à des machines virtuelles spécifiques ou à des interfaces réseau (NIC) individuelles attachées à des machines virtuelles (Resource Manager). Pour plus d’informations, consultez [Sécurité du réseau](../virtual-network/network-security-groups-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json).
- Tous les flux de trafic au sein de votre réseau sont évalués à l’aide des règles dans le groupe de sécurité réseau applicable.
- Le résultat de ces évaluations est recueilli dans des journaux de flux de groupe de sécurité réseau. Les journaux de flux sont collectés via la plateforme Azure et ne nécessitent aucune modification des ressources du client.
- Remarque : Les règles sont de deux types : avec fin d’exécution et sans fin d’exécution, chacune avec des comportements de journalisation différents.
- - Les règles NSG de refus sont avec fin d’exécution. Le groupe de sécurité réseau qui refuse le trafic le consigne dans les journaux de flux, et dans ce cas le traitement s’arrête dès qu’un groupe de sécurité réseau refuse le trafic. 
- - Les règles NSG d’autorisation sont sans fin d’exécution, ce qui signifie que même si un groupe de sécurité réseau l’autorise, le traitement se poursuit jusqu’au groupe de sécurité réseau suivant. Le dernier groupe de sécurité réseau autorisant le trafic consigne le trafic dans les journaux de flux.
- Les journaux de flux de groupe de sécurité réseau sont écrits dans des comptes de stockage à partir desquels ils sont accessibles.
- Vous pouvez exporter, traiter, analyser et visualiser les journaux de flux à l’aide d’outils tels que TA, Splunk, Grafana, StealthWatch, etc.

## <a name="log-format"></a>Format de journal

Les flux de journaux d’activité incluent les propriétés suivantes :

* **time** - L’heure à laquelle l’événement a été journalisé.
* **systemId** - L’ID système du groupe de sécurité réseau.
* **category** - La catégorie de l'événement. La catégorie est toujours **NetworkSecurityGroupFlowEvent**
* **resourceid** - L’ID de ressource du groupe de sécurité réseau.
* **operationName** - Toujours NetworkSecurityGroupFlowEvents.
* **properties** - Une collection des propriétés du flux.
    * **Version** - Le numéro de version du schéma d’événement du journal de flux.
    * **flow** - Une collection de flux. Cette propriété a plusieurs entrées pour différentes règles.
        * **rule** - La règle pour laquelle les flux sont répertoriés.
            * **flows** - Une collection de flux.
                * **mac** - L’adresse MAC de la carte réseau de la machine virtuelle où le flux a été collecté.
                * **flowTuples** - Une chaîne qui contient plusieurs propriétés séparées par des virgules pour le tuple de flux.
                    * **Time Stamp** - Cette valeur indique la date et l’heure du flux au format UNIX EPOCH.
                    * **Source IP** - L’adresse IP source.
                    * **Destination IP** - L’adresse IP de destination.
                    * **Source Port** - Le port source.
                    * **Destination Port** - Le port de destination.
                    * **Protocol** - Le protocole du flux. Les valeurs valides sont **T** pour TCP et **U** pour UDP.
                    * **Traffic Flow** - La direction du flux de trafic. Les valeurs valides sont **I** pour le trafic entrant et **O** pour le trafic sortant.
                    * **Traffic Decision** - Indique si le trafic a été autorisé ou refusé. Les valeurs valides sont **A** pour autorisé et **D** pour refusé.
                    * **Flow State - Version 2 Only** - Capture l’état du flux. Les états possibles sont **B** : début, lors de la création d’un flux. Aucune statistique n’est fournie. **C** : continuation d’un flux en cours. Des statistiques sont fournies toutes les 5 minutes. **E** : fin, quand un flux est arrêté. Des statistiques sont fournies.
                    * **Packets - Source to destination - Version 2 Only** Nombre total de paquets TCP ou UDP envoyés de la source à la destination depuis la dernière mise à jour.
                    * **Bytes sent - Source to destination - Version 2 Only** Nombre total d’octets de paquets TCP ou UDP envoyés de la source à la destination depuis la dernière mise à jour. Les octets de paquets incluent l’en-tête et la charge utile du paquet.
                    * **Packets - Destination to source - Version 2 Only** Nombre total de paquets TCP ou UDP envoyés de la destination à la source depuis la dernière mise à jour.
                    * **Bytes sent - Destination to source - Version 2 Only** Nombre total d’octets de paquets TCP et UDP envoyés de la destination à la source depuis la dernière mise à jour. Les octets de paquets incluent l’en-tête et la charge utile du paquet.


**Journaux de flux de groupe de sécurité réseau version 2 (à distinguer de la version 1)** 

La version 2 des journaux introduit le concept d’état de flux. Vous pouvez configurer la version des journaux de flux que vous recevez.

L’état du flux _B_ est enregistré lorsqu’un flux est lancé. L’état du flux _C_ et l’état du flux _E_ sont les états qui marquent respectivement la continuation d’un flux et son arrêt. Les deux états _C_ et _E_ contiennent des informations sur la bande passante du trafic.

### <a name="sample-log-records"></a>Exemples d’enregistrements de journal

Le texte ci-dessous est un exemple de journal de flux. Comme vous pouvez le voir, il existe plusieurs enregistrements qui suivent la liste des propriétés décrite dans la section précédente.

> [!NOTE]
> Les valeurs de la propriété *flowTuples* correspondent à une liste de valeurs séparées par des virgules.
 
**Exemple de format de journal de flux de groupe de sécurité réseau version 1**
```json
{
    "records": [
        {
            "time": "2017-02-16T22:00:32.8950000Z",
            "systemId": "2c002c16-72f3-4dc5-b391-3444c3527434",
            "category": "NetworkSecurityGroupFlowEvent",
            "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/FABRIKAMRG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/FABRIAKMVM1-NSG",
            "operationName": "NetworkSecurityGroupFlowEvents",
            "properties": {
                "Version": 1,
                "flows": [
                    {
                        "rule": "DefaultRule_DenyAllInBound",
                        "flows": [
                            {
                                "mac": "000D3AF8801A",
                                "flowTuples": [
                                    "1487282421,42.119.146.95,10.1.0.4,51529,5358,T,I,D"
                                ]
                            }
                        ]
                    },
                    {
                        "rule": "UserRule_default-allow-rdp",
                        "flows": [
                            {
                                "mac": "000D3AF8801A",
                                "flowTuples": [
                                    "1487282370,163.28.66.17,10.1.0.4,61771,3389,T,I,A",
                                    "1487282393,5.39.218.34,10.1.0.4,58596,3389,T,I,A",
                                    "1487282393,91.224.160.154,10.1.0.4,61540,3389,T,I,A",
                                    "1487282423,13.76.89.229,10.1.0.4,53163,3389,T,I,A"
                                ]
                            }
                        ]
                    }
                ]
            }
        },
        {
            "time": "2017-02-16T22:01:32.8960000Z",
            "systemId": "2c002c16-72f3-4dc5-b391-3444c3527434",
            "category": "NetworkSecurityGroupFlowEvent",
            "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/FABRIKAMRG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/FABRIAKMVM1-NSG",
            "operationName": "NetworkSecurityGroupFlowEvents",
            "properties": {
                "Version": 1,
                "flows": [
                    {
                        "rule": "DefaultRule_DenyAllInBound",
                        "flows": [
                            {
                                "mac": "000D3AF8801A",
                                "flowTuples": [
                                    "1487282481,195.78.210.194,10.1.0.4,53,1732,U,I,D"
                                ]
                            }
                        ]
                    },
                    {
                        "rule": "UserRule_default-allow-rdp",
                        "flows": [
                            {
                                "mac": "000D3AF8801A",
                                "flowTuples": [
                                    "1487282435,61.129.251.68,10.1.0.4,57776,3389,T,I,A",
                                    "1487282454,84.25.174.170,10.1.0.4,59085,3389,T,I,A",
                                    "1487282477,77.68.9.50,10.1.0.4,65078,3389,T,I,A"
                                ]
                            }
                        ]
                    }
                ]
            }
        },
    "records":
    [
        
        {
             "time": "2017-02-16T22:00:32.8950000Z",
             "systemId": "2c002c16-72f3-4dc5-b391-3444c3527434",
             "category": "NetworkSecurityGroupFlowEvent",
             "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/FABRIKAMRG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/FABRIAKMVM1-NSG",
             "operationName": "NetworkSecurityGroupFlowEvents",
             "properties": {"Version":1,"flows":[{"rule":"DefaultRule_DenyAllInBound","flows":[{"mac":"000D3AF8801A","flowTuples":["1487282421,42.119.146.95,10.1.0.4,51529,5358,T,I,D"]}]},{"rule":"UserRule_default-allow-rdp","flows":[{"mac":"000D3AF8801A","flowTuples":["1487282370,163.28.66.17,10.1.0.4,61771,3389,T,I,A","1487282393,5.39.218.34,10.1.0.4,58596,3389,T,I,A","1487282393,91.224.160.154,10.1.0.4,61540,3389,T,I,A","1487282423,13.76.89.229,10.1.0.4,53163,3389,T,I,A"]}]}]}
        }
        ,
        {
             "time": "2017-02-16T22:01:32.8960000Z",
             "systemId": "2c002c16-72f3-4dc5-b391-3444c3527434",
             "category": "NetworkSecurityGroupFlowEvent",
             "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/FABRIKAMRG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/FABRIAKMVM1-NSG",
             "operationName": "NetworkSecurityGroupFlowEvents",
             "properties": {"Version":1,"flows":[{"rule":"DefaultRule_DenyAllInBound","flows":[{"mac":"000D3AF8801A","flowTuples":["1487282481,195.78.210.194,10.1.0.4,53,1732,U,I,D"]}]},{"rule":"UserRule_default-allow-rdp","flows":[{"mac":"000D3AF8801A","flowTuples":["1487282435,61.129.251.68,10.1.0.4,57776,3389,T,I,A","1487282454,84.25.174.170,10.1.0.4,59085,3389,T,I,A","1487282477,77.68.9.50,10.1.0.4,65078,3389,T,I,A"]}]}]}
        }
        ,
        {
             "time": "2017-02-16T22:02:32.9040000Z",
             "systemId": "2c002c16-72f3-4dc5-b391-3444c3527434",
             "category": "NetworkSecurityGroupFlowEvent",
             "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/FABRIKAMRG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/FABRIAKMVM1-NSG",
             "operationName": "NetworkSecurityGroupFlowEvents",
             "properties": {"Version":1,"flows":[{"rule":"DefaultRule_DenyAllInBound","flows":[{"mac":"000D3AF8801A","flowTuples":["1487282492,175.182.69.29,10.1.0.4,28918,5358,T,I,D","1487282505,71.6.216.55,10.1.0.4,8080,8080,T,I,D"]}]},{"rule":"UserRule_default-allow-rdp","flows":[{"mac":"000D3AF8801A","flowTuples":["1487282512,91.224.160.154,10.1.0.4,59046,3389,T,I,A"]}]}]}
        }
        
        
```
**Exemple de format de journal de flux de groupe de sécurité réseau version 2**
```json
 {
    "records": [
        {
            "time": "2018-11-13T12:00:35.3899262Z",
            "systemId": "a0fca5ce-022c-47b1-9735-89943b42f2fa",
            "category": "NetworkSecurityGroupFlowEvent",
            "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/FABRIKAMRG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/FABRIAKMVM1-NSG",
            "operationName": "NetworkSecurityGroupFlowEvents",
            "properties": {
                "Version": 2,
                "flows": [
                    {
                        "rule": "DefaultRule_DenyAllInBound",
                        "flows": [
                            {
                                "mac": "000D3AF87856",
                                "flowTuples": [
                                    "1542110402,94.102.49.190,10.5.16.4,28746,443,U,I,D,B,,,,",
                                    "1542110424,176.119.4.10,10.5.16.4,56509,59336,T,I,D,B,,,,",
                                    "1542110432,167.99.86.8,10.5.16.4,48495,8088,T,I,D,B,,,,"
                                ]
                            }
                        ]
                    },
                    {
                        "rule": "DefaultRule_AllowInternetOutBound",
                        "flows": [
                            {
                                "mac": "000D3AF87856",
                                "flowTuples": [
                                    "1542110377,10.5.16.4,13.67.143.118,59831,443,T,O,A,B,,,,",
                                    "1542110379,10.5.16.4,13.67.143.117,59932,443,T,O,A,E,1,66,1,66",
                                    "1542110379,10.5.16.4,13.67.143.115,44931,443,T,O,A,C,30,16978,24,14008",
                                    "1542110406,10.5.16.4,40.71.12.225,59929,443,T,O,A,E,15,8489,12,7054"
                                ]
                            }
                        ]
                    }
                ]
            }
        },
        {
            "time": "2018-11-13T12:01:35.3918317Z",
            "systemId": "a0fca5ce-022c-47b1-9735-89943b42f2fa",
            "category": "NetworkSecurityGroupFlowEvent",
            "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-000000000000/RESOURCEGROUPS/FABRIKAMRG/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/FABRIAKMVM1-NSG",
            "operationName": "NetworkSecurityGroupFlowEvents",
            "properties": {
                "Version": 2,
                "flows": [
                    {
                        "rule": "DefaultRule_DenyAllInBound",
                        "flows": [
                            {
                                "mac": "000D3AF87856",
                                "flowTuples": [
                                    "1542110437,125.64.94.197,10.5.16.4,59752,18264,T,I,D,B,,,,",
                                    "1542110475,80.211.72.221,10.5.16.4,37433,8088,T,I,D,B,,,,",
                                    "1542110487,46.101.199.124,10.5.16.4,60577,8088,T,I,D,B,,,,",
                                    "1542110490,176.119.4.30,10.5.16.4,57067,52801,T,I,D,B,,,,"
                                ]
                            }
                        ]
                    }
                ]
            }
        }
        
```
**Explication de multiplet de journal**

![tuple des journaux de flux](./media/network-watcher-nsg-flow-logging-overview/tuple.png)

**Exemple de calcul de la bande passante**

tuples de flux à partir d’une conversation TCP entre 185.170.185.105:35370 and 10.2.0.4:23 :

« 1493763938,185.170.185.105,10.2.0.4,35370,23,T,I,A,B,,,, » « 1493695838,185.170.185.105,10.2.0.4,35370,23,T,I,A,C,1021,588096,8005,4610880 » « 1493696138,185.170.185.105,10.2.0.4,35370,23,T,I,A,E,52,29952,47,27072 »

Pour la continuation _C_ et la fin _E_ d’états de flux, le nombre d’octets et de paquets est le nombre cumulé depuis l’enregistrement de tuple de flux précédent. Si l’on se réfère à l’exemple de conversation précédent, le nombre total de paquets transférés est de 1 021 + 52 + 8 005 + 47 = 9 125. Le nombre total d’octets transférés est de 588 096+29 952+4 610 880+27 072 = 5 256 000.


## <a name="enabling-nsg-flow-logs"></a>Activation des journaux de flux de groupe de sécurité réseau

Utilisez le lien approprié ci-dessous pour obtenir des guides sur l’activation des journaux de flux.

- [Azure portal](./network-watcher-nsg-flow-logging-portal.md)
- [PowerShell](./network-watcher-nsg-flow-logging-powershell.md)
- [INTERFACE DE LIGNE DE COMMANDE](./network-watcher-nsg-flow-logging-cli.md)
- [REST](./network-watcher-nsg-flow-logging-rest.md)
- [Azure Resource Manager](./network-watcher-nsg-flow-logging-azure-resource-manager.md)

## <a name="updating-parameters"></a>Mise à jour des paramètres

**Azure portal**

Sur le portail Azure, accédez à la section journaux de flux NSG dans Network Watcher. Cliquez ensuite sur le nom du groupe de sécurité réseau. Le volet des paramètres du journal de flux s’affiche. Modifiez les paramètres de votre choix, puis appuyez sur **Enregistrer** pour déployer les modifications.

**PS/CLI/REST/ARM**

Pour mettre à jour des paramètres à l’aide d’outils en ligne de commande, utilisez la même commande que celle utilisée pour activer les journaux de flux (ci-dessus), mais avec les paramètres mis à jour que vous souhaitez modifier.

## <a name="working-with-flow-logs"></a>Utilisation des journaux de flux

*Lire et exporter des journaux de flux*

- [Télécharger et afficher des journaux de flux à partir du portail](./network-watcher-nsg-flow-logging-portal.md#download-flow-log)
- [Lire des journaux de flux à l’aide de fonctions PowerShell](./network-watcher-read-nsg-flow-logs.md)
- [Exporter des journaux de flux de groupe de sécurité réseau vers Splunk](https://www.splunk.com/en_us/blog/tips-and-tricks/splunking-microsoft-azure-network-watcher-data.html)

Même si les journaux de flux ciblent les groupes de sécurité réseau, ils ne sont pas affichés de la même façon que les autres journaux d’activité. Les journaux de flux sont uniquement stockés dans un compte de stockage et suivent le chemin de journalisation comme indiqué dans l’exemple ci-après :

```
https://{storageAccountName}.blob.core.windows.net/insights-logs-networksecuritygroupflowevent/resourceId=/SUBSCRIPTIONS/{subscriptionID}/RESOURCEGROUPS/{resourceGroupName}/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/{nsgName}/y={year}/m={month}/d={day}/h={hour}/m=00/macAddress={macAddress}/PT1H.json
```

*Visualiser des journaux de flux*

- [Azure Traffic Analytics](./traffic-analytics.md) est un service Azure natif qui traite des journaux de flux, en extrait des aperçus et les visualise. 
- [[Didacticiel] Visualiser des journaux de flux de groupe de sécurité réseau avec Power BI](./network-watcher-visualize-nsg-flow-logs-power-bi.md)
- [[Didacticiel] Visualiser des journaux de flux de groupe de sécurité réseau avec Elastic Stack](./network-watcher-visualize-nsg-flow-logs-open-source-tools.md)
- [[Didacticiel] Gérer et analyser des journaux de flux de groupe de sécurité réseau à l’aide de Grafana](./network-watcher-nsg-grafana.md)
- [[Didacticiel] Gérer et analyser des journaux de flux de groupe de sécurité réseau à l’aide de Graylog](./network-watcher-analyze-nsg-flow-logs-graylog.md)


## <a name="nsg-flow-logging-considerations"></a>Considérations relatives à la journalisation de flux NSG

**Considérations relatives aux comptes de stockage** : 

- Localisation : le compte de stockage doit se trouver dans la même région que le groupe de sécurité réseau.
- Rotation des clés auto-gérée : si vous modifiez (ou effectuez la rotation) des clés d’accès à votre compte de stockage, les journaux de flux de groupe de sécurité réseau cessent de fonctionner. Pour corriger ce problème, vous devez désactiver puis réactiver les journaux de flux de groupe de sécurité réseau.

**Coûts de la journalisation de flux** : la journalisation de flux NSG est facturée selon le volume de journaux d’activité produits. Un volume de trafic élevé peut entraîner un volume important de journaux de flux avec les coûts associés. Les tarifs des journaux de flux NSG n’incluent pas les coûts de stockage afférents. L’utilisation de la fonctionnalité de stratégie de rétention avec la journalisation de flux NSG implique des coûts de stockage distincts pendant de longues périodes. Si vous n’avez pas besoin de la fonctionnalité de stratégie de conservation, nous vous recommandons de définir cette valeur sur 0. Pour en savoir plus, consultez [Tarifs Network Watcher](https://azure.microsoft.com/pricing/details/network-watcher/) et [Tarifs du stockage Azure](https://azure.microsoft.com/pricing/details/storage/) pour de plus amples informations.

**Problèmes liés aux règles TCP entrantes définies par l’utilisateur** : [Les groupes de sécurité réseau (NSG)](../virtual-network/network-security-groups-overview.md) sont implémentés en tant que [pare-feu avec état](https://en.wikipedia.org/wiki/Stateful_firewall?oldformat=true). Toutefois, en raison des limitations actuelles de la plateforme, les règles définies par l’utilisateur qui affectent les flux TCP entrants sont implémentées sans état. Pour cette raison, les flux affectés par les règles de trafic entrant définies par l’utilisateur n’ont pas de fin d’exécution. Par ailleurs, les nombres de paquets et d’octets ne sont pas enregistrés pour ces flux. Par conséquent, le nombre d’octets et de paquets signalé dans les journaux de flux NSG (et Traffic Analytics) peut différer du nombre réel. Un indicateur d’abonnement qui résout ces problèmes est planifié pour une disponibilité au plus tard en décembre 2020. En attendant, les clients confrontés à des problèmes graves en raison de ce comportement peuvent demander une assistance via le support technique, émettre une demande de support sous Network Watcher > Journaux de workflow NSG.  

**Flux entrants journalisés à partir d’adresses IP Internet dans des machines virtuelles sans IP publiques** : Les machines virtuelles qui n’ont pas d’IP publique attribuée via une IP publique associée à la carte d’interface réseau en tant qu’IP publique de niveau d’instance, ou qui font partie d’un pool principal équilibreur de charge de base, utilisent une [architecture de système en réseau par défaut](../load-balancer/load-balancer-outbound-connections.md) et ont une adresse IP affectée par Azure afin de faciliter la connectivité sortante. Par conséquent, vous pouvez observer des entrées de journal de flux pour les flux d’adresses IP Internet, si le flux est destiné à un port dans la plage de ports attribués à l’architecture de système en réseau. Bien qu’Azure n’autorise pas ces flux vers les machines virtuelles, la tentative est journalisée et apparaît par conception dans le journal de flux du Groupe de sécurité réseau Network Watcher. Nous recommandons que le trafic Internet entrant indésirable soit explicitement bloqué avec le Groupe de sécurité réseau.

**Problème avec le groupe de sécurité réseau du sous-réseau d’Application Gateway v2** : La journalisation de flux sur le groupe de sécurité réseau du sous-réseau d’Application Gateway v2 [n’est pas prise en charge](https://docs.microsoft.com/azure/application-gateway/application-gateway-faq#are-nsg-flow-logs-supported-on-nsgs-associated-to-application-gateway-v2-subnet) actuellement. Ce problème ne concerne pas Application Gateway v1.

**Services incompatibles** : En raison des limitations actuelles de la plateforme, un petit ensemble de services Azure n’est pas pris en charge par les journaux de flux NSG. La liste actuelle des services incompatibles est
- [Azure Kubernetes Services (AKS)](https://azure.microsoft.com/services/kubernetes-service/)
- [Logic Apps](https://azure.microsoft.com/services/logic-apps/) 

## <a name="best-practices"></a>Meilleures pratiques

**Activer sur des réseaux virtuels/sous-réseaux critiques** : Les journaux de flux doivent être activés sur tous les réseaux virtuels/sous-réseaux critiques de votre abonnement en guise de meilleures pratiques d’audit et de sécurité. 

**Activer la journalisation de flux NSG sur tous les groupes de sécurité réseau associés à une ressource** : la journalisation de flux dans Azure est configurée sur la ressource NSG. Un flux ne peut être associé qu’à une règle de groupe de sécurité réseau. Dans les scénarios où plusieurs groupes de sécurité réseau sont utilisés, nous recommandons d’activer les journaux de flux de groupe de sécurité réseau sur tous les groupes de sécurité réseau appliqués au sous-réseau ou à l’interface réseau de la ressource pour vous assurer que tout le trafic est enregistré. Pour en savoir plus, consultez la section [Évaluation du trafic](../virtual-network/network-security-group-how-it-works.md) dans Groupes de sécurité réseau. 

Quelques scénarios courants :
1. **Plusieurs groupes de sécurité réseau sur une carte réseau** : Si plusieurs groupes de sécurité réseau sont attachés à une carte réseau, la journalisation de flux doit être activée sur chacun d’eux.
1. **Groupe de sécurité réseau au niveau de la carte réseau et du sous-réseau** : Si le groupe de sécurité réseau est configuré au niveau de la carte réseau ainsi qu’au niveau du sous-réseau, la journalisation de flux doit être activée sur les deux groupes de sécurité réseau. 

**Approvisionnement du stockage** : Le stockage doit être approvisionné dans le paramétrage avec le volume de journal de flux attendu.

**Affectation de noms** : Le nom du groupe de sécurité réseau doit comporter jusqu’à 80 caractères et les noms de règle de groupe de sécurité réseau jusqu’à 65 caractères. Si les noms dépassent leur limite de caractères, ils peuvent être tronqués lors de la journalisation.

## <a name="troubleshooting-common-issues"></a>Résolution des problèmes courants

**Je n’ai pas pu activer les Journaux de flux NSG**

- Le fournisseur de ressources **Microsoft.Insights** n’est pas inscrit

Si vous avez reçu une erreur _AuthorizationFailed_ ou _GatewayAuthenticationFailed_, il se peut vous n’ayez pas activé le fournisseur de ressources Microsoft Insights sur votre abonnement. [Suivez les instructions](./network-watcher-nsg-flow-logging-portal.md#register-insights-provider) pour activer le fournisseur de Microsoft Insights.

**J’ai activé les Journaux de flux NSG, mais je ne vois pas de données dans mon compte de stockage**

- **Temps de préparation**

L’affichage des journaux de flux NSG dans votre compte de stockage (s’il est correctement configuré) peut prendre jusqu’à 5 minutes. Un fichier PT1H. json s’affiche, qui est accessible [comme expliqué ici](./network-watcher-nsg-flow-logging-portal.md#download-flow-log).

- **Aucun trafic sur votre groupes de sécurité réseau**

Il peut arriver que les journaux ne s’affichent pas parce que vos machines virtuelles ne sont pas actives ou parce que des filtres en amont au niveau d’App Gateway ou d’autres appareils bloquent le trafic vers vos groupes de sécurité réseau.

**Je souhaite automatiser les Journaux de flux NSG**

La prise en charge de l’automation via des modèles ARM n’est actuellement pas disponible pour les Journaux de flux NSG. Pour plus d’informations, lisez l’[annonce relative à la fonctionnalité](https://azure.microsoft.com/updates/arm-template-support-for-nsg-flow-logs/).

## <a name="faq"></a>Questions fréquentes (FAQ)

**Que font les journaux de flux de groupe de sécurité réseau ?**

Les ressources réseau Azure peuvent être combinées et gérées via les [groupes de sécurité réseau (NSG)](../virtual-network/network-security-groups-overview.md). Les journaux de flux NSG vous permettent de consigner les informations de flux à 5 tuples concernant tout le trafic via vos groupes de sécurité réseau. Les journaux de flux bruts sont écrits dans un compte de stockage Azure à partir duquel ils peuvent être traités, analysés, interrogés ou exportés en fonction des besoins.

**L’utilisation des journaux de flux a-t-elle un impact sur la latence ou les performances de mon réseau ?**

Les données des journaux de flux sont collectées en dehors du chemin d’accès de votre trafic réseau. Ils n’affectent donc pas le débit ou la latence du réseau. Vous pouvez créer ou supprimer des journaux de flux sans risque d’impact sur les performances du réseau.

**Comment utiliser des journaux de flux de groupe de sécurité réseau sur un compte de stockage derrière un pare-feu ?**

Pour utiliser un compte de stockage derrière un pare-feu, vous devez fournir une exception aux services Microsoft approuvés pour accéder à votre compte de stockage :

- Accédez au compte de stockage en tapant son nom dans la recherche globale sur le portail ou à partir de la [page Comptes de stockage](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.Storage%2FStorageAccounts)
- Dans la section **PARAMÈTRES**, sélectionnez **Pare-feu et réseaux virtuels**.
- Dans **Autoriser l’accès depuis**, sélectionnez **Réseaux sélectionnés**. Ensuite, sous **Exceptions**, cochez la case en regard de ****Autoriser les services Microsoft approuvés à accéder à ce compte de stockage****
- Si cette option est déjà sélectionnée, aucune modification n’est nécessaire.
- Localisez votre groupe de sécurité réseau cible dans la [page de vue d’ensemble des journaux de flux NSG](https://ms.portal.azure.com/#blade/Microsoft_Azure_Network/NetworkWatcherMenuBlade/flowLogs) et activez les journaux de flux NSG avec le compte de stockage ci-dessus sélectionné.

Vous pouvez consulter les journaux de stockage au bout de quelques minutes. Vous devriez y voir qu’un horodateur a été mis à jour ou qu’un fichier JSON a été créé.

**Comment utiliser des journaux de flux NSG sur un compte de stockage derrière un point de terminaison de service ?**

Les journaux de flux NSG sont compatibles avec les points de terminaison de service et ne nécessitent aucune configuration supplémentaire. Consultez le [tutoriel sur l’activation des points de terminaison de service](../virtual-network/tutorial-restrict-network-access-to-resources.md#enable-a-service-endpoint) dans votre réseau virtuel.

**Quelle est la différence entre les versions 1 et 2 des journaux de flux ?**

La version 2 des journaux de flux introduit le concept d’_état de flux_ et stocke des informations sur les octets et les paquets transmis. [En savoir plus](#log-format)

## <a name="pricing"></a>Tarifs

Les journaux de flux NSG sont facturés par Go de journaux collectés, et fournis avec un niveau gratuit de 5 Go/mois par abonnement. Pour connaître la tarification en vigueur dans votre région, consultez la [page de tarification de Network Watcher](https://azure.microsoft.com/pricing/details/network-watcher/).

Le stockage des journaux est facturé séparément. Pour connaître les prix en vigueur, consultez la [page de tarification des objet blob de blocs de Stockage Azure](https://azure.microsoft.com/pricing/details/storage/blobs/).
