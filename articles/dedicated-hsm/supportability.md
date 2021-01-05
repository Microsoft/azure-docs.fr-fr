---
title: Prise en charge - Azure HSM dédié | Microsoft Docs
description: Options de prise en charge et domaines de responsabilité pour un module de sécurité matériel (HSM) dédié Azure dans différents scénarios
services: dedicated-hsm
author: johndaw
manager: rkarlin
ms.service: key-vault
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.custom: seodec18
ms.date: 03/27/2019
ms.author: mbaldwin
ms.openlocfilehash: d83d688707baf6098d63dfde9b4181eb04fb9729
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "70881005"
---
# <a name="azure-dedicated-hsm-supportability"></a>Prise en charge Azure HSM dédié

Le service HSM dédié d’Azure fournit un appareil physique à usage exclusif du client, avec un contrôle administratif complet et une responsabilité de la gestion. L’appareil mis à disposition est un [modèle Gemalto SafeNet Luna 7 HSM A790](https://safenet.gemalto.com/data-encryption/hardware-security-modules-hsms/safenet-network-hsm/). Une fois l’approvisionnement effectué par un client, Microsoft n’aura aucun accès d’administration au-delà d’un attachement de port sériel physique dans un rôle de surveillance.  Sans accès, Microsoft ne peut ni assurer une maintenance au niveau logiciel, ni assumer des responsabilités d’administration de système. Par conséquent, les clients sont responsables des activités opérationnelles classiques.
Ils sont entièrement responsables des applications qui utilisent les HSM et doivent utiliser Gemalto pour bénéficier d’un support ou d’une assistance basée sur du conseil . En raison de l’étendue de la propriété du client en matière d’hygiène opérationnelle, Microsoft ne peut offrir aucun type de garantie de haute disponibilité pour ce service. Il est la responsabilité du client de s’assurer que ses applications sont correctement configurées pour atteindre une haute disponibilité. Microsoft surveille l’intégrité de l’appareil et la connectivité réseau et assure leur maintenance.

## <a name="getting-support"></a>Obtention d’une assistance

Le support technique pour Dedicated HSM est un effort conjoint entre Microsoft et Gemalto. Tous les problèmes de matériel ou de chemin réseau seront résolus par Microsoft, et tout ce qui concerne le module HSM proprement dit, comme la configuration, les logiciel, le microprogramme et le développement d’application, sera traité par Gemalto. Ce modèle de prise en charge garantit l’accès le plus rapide au support le plus efficace. En cas de doute sur un problème particulier, envoyez une demande de support à Microsoft, et nous veillerons à vous diriger correctement. Microsoft restera impliqué dans tous les scénarios de support et s’efforcera de procurer à ses clients la meilleure expérience de support possible.

## <a name="gemalto-support"></a>Support de Gemalto

Les clients qui utilisent le service Dedicated HSM sont éligibles au support technique de Gemalto conformément à leur Plan de support Plus. Cela nécessite simplement un processus d’inscription par le biais du portail de support Gemalto. Un ID de client et des instructions vous seront fournis dans le cadre de l’engagement initial auprès de Microsoft afin d’accéder au service Dedicated HSM. Pour bénéficier du support de Gemalto, ils doivent passer par leur [portail de support client](https://supportportal.gemalto.com/csm/).
Un aspect important à noter est que Gemalto fournit tous les logiciels et la documentation nécessaires pour utiliser le HSM (par exemple le logiciel d’accès client et les kits SDK) par le biais du téléchargement à partir du portail de support client.

### <a name="software-components"></a>Composants logiciels

Différents composants logiciels sont utilisés dans la configuration de périphériques HSM :

* Logiciel client
* Kit SDK
* Outils

### <a name="guidance"></a>Assistance

Gemalto met à disposition des instructions d’administration et de configuration via le [portail de support client](https://supportportal.gemalto.com/csm/). Après la connexion à l’aide d’un ID client valide, ces documents sont disponibles au téléchargement. Gemalto fournit également une série de guides d’intégration pour aider les clients avec différents scénarios et intégrations de logiciels. Pour plus d’informations, consultez le [site partenaire de Gemalto pour Microsoft](https://safenet.gemalto.com/partners/microsoft/).

### <a name="support"></a>Support

Tout problème ou toute question relatifs au logiciel et liés à l’utilisation des HSM dans le cadre du service HSM dédié doivent être soumis directement à Gemalto. Tous les composants logiciels répertoriés ci-dessus et toute configuration HSM personnalisée après le déploiement seront traités par Gemalto. Pour plus d’informations, consultez le [portail de support client Gemalto](https://supportportal.gemalto.com/csm/).

### <a name="consulting-services"></a>Services de conseil

Pour toute assistance lors de la conception, du développement et du déploiement d’applications personnalisées qui utilisent HSM, contactez votre responsable de compte Gemalto.

## <a name="microsoft-support"></a>Support Microsoft

Microsoft veillera à ce que les appareils HSM physiques soient accessibles par le biais du réseau et en état de fonctionner pour une utilisation exclusive par un seul client. Les clients sont responsables de la configuration, de l’administration et de la gestion de l’appareil. Les responsabilités suivantes incombent à Microsoft :

* S’assurer que l’appareil est sous tension et refroidi
* Maintenir un état opérationnel du HSM (par exemple, scénarios d’arrêt/de réparation)
* L’appareil est accessible via le réseau.

Les problèmes tels que les suivants doivent être signalés à Microsoft :

* Défaillances de composants
* Défaillance complète de l’appareil
* Problèmes d’accès au réseau
* Problèmes d’approvisionnement et de suppression de privilèges d’accès.

Microsoft a accès à l’appareil par le biais d’un port sériel physique via un rôle de surveillance (autrement dit, pas un rôle d’administration) qui permet une télémétrie d’intégrité de base.  Cela permet à Microsoft fournir une notification proactive des problèmes au client, sauf si ce dernier choisit de limiter cette autorisation. 

### <a name="provisioning-and-decommissioning"></a>Approvisionnement et désaffectation

Une fois un client dispose d’une inscription approuvée pour le service HSM dédié, il peut créer des ressources HSM (actuellement par le biais de PowerShell ou de l’interface de ligne de commande et non du portail Azure). La ressource passe par un processus d’allocation qui mappe un appareil physique dans une région spécifiée au réseau virtuel prédéfini d’un client (VNet). Une fois visible sur un VNet, le client peut accéder à l'appareil et le configurer plus en détail selon ses besoins. Les clients accèdent à leurs HSM dédiés à l’aide des outils et logiciels clients Gemalto. Le processus de création de ressources est pris en charge par Microsoft. Le processus de configuration personnalisé et les autres processus sont pris en charge par Gemalto. (voir le support Gemalto ci-dessus). Lorsqu’un client a terminé d’utiliser un HSM, il doit être réinitialisé (ou remis à zéro) pour s’assurer que les données ne sont pas persistantes. Le processus de réinitialisation de l’appareil supprime toutes les données et toute la configuration personnalisées. Microsoft libère l’appareil et le renvoie vide au pool. Cela signifie que lorsque l’appareil est renvoyé au pool, il n’y aucune preuve de l’activité du client précédent. 

### <a name="hardware-issues"></a>Problèmes matériels

L’appareil HSM a des dispositifs d’alimentation et des ventilateurs redondants et remplaçables.  Toutefois, l’enlèvement des ventilateurs génèrera quand même un événement de falsification. En cas de défaillance d’un composant, Microsoft utilise le processus le plus approprié pour résoudre le problème au niveau du composant d’une manière qui provoque une interruption minimale et le plus faible risque en termes de disponibilité des services pour nos clients.
N’importe quelle défaillance plus grave de l’appareil entraîne son remplacement par un appareil neuf du pool libre. Le client inclut simplement le nouvel appareil dans la paire à haute disponibilité existante pour qu’il soit synchronisé et retrouve un état opérationnel complet. Les dispositifs de support des données de l’appareil défaillant sont enlevés et détruits dans le centre de données. Seul le châssis est renvoyé à Gemalto pour être recyclé.


### <a name="networking-issues"></a>Problèmes de mise en réseau

Si les clients rencontrent des problèmes d’accès réseau à l’appareil HSM, ils doivent contacter le support Microsoft. Un test simple de l’accès réseau consiste à utiliser le protocole SSH pour se connecter à l’appareil HSM. Si cela échoue, contactez le support Microsoft.

## <a name="service-level-expectations-for-support"></a>Attentes de niveau de service pour le support

Pour les niveaux de service du support Microsoft, reportez-vous au [plan de support Azure](https://azure.microsoft.com/support/plans/).
Pour les niveaux de service de support Gemalto, reportez-vous à [Éléments principaux du support Gemalto](https://azure.microsoft.com/support/plans/).

## <a name="next-steps"></a>Étapes suivantes

Il est recommandé de bien comprendre les concepts clés, comme la haute disponibilité et la sécurité, avant le provisionnement des appareils et la conception ou le déploiement des applications.

* [Architecture de déploiement](deployment-architecture.md)
* [Haute disponibilité](high-availability.md)
* [Sécurité physique](physical-security.md)
* [Mise en réseau](networking.md)

