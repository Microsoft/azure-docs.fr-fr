---
title: Sécurité de l’hyperviseur sur la flotte Azure – Sécurité Azure
description: Présentation technique de la sécurité de l’hyperviseur sur la flotte Azure.
author: yosharm
ms.service: security
ms.subservice: security-fundamentals
ms.topic: article
ms.author: terrylan
manager: rkarlin
ms.date: 11/10/2020
ms.openlocfilehash: 766266edd663b75b893a5883e30bb48eed7bbfdf
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94696113"
---
# <a name="hypervisor-security-on-the-azure-fleet"></a>Sécurité de l’hyperviseur sur la flotte Azure

Le système de l’hyperviseur Azure repose sur Windows Hyper-V. Le système de l’hyperviseur permet à l’administrateur de l’ordinateur de spécifier des partitions invitées avec des espaces d’adressage distincts. Les espaces d’adressage distincts vous permettent de charger un système d’exploitation et des applications fonctionnant en parallèle du système d’exploitation (hôte) qui s’exécute dans la partition racine de l’ordinateur. Le système d’exploitation hôte (également appelé partition racine privilégiée) a un accès direct à tous les appareils physiques et périphériques du système (contrôleurs de stockage, cartes réseau). Le système d’exploitation hôte permet aux partitions invitées de partager l’utilisation de ces appareils physiques en exposant des « appareils virtuels » à chaque partition invitée. Ainsi, un système d’exploitation qui s’exécute dans une partition invitée a accès aux périphériques virtualisés qui sont fournis par les services de virtualisation s’exécutant dans la partition racine.

L’hyperviseur Azure est conçu pour tenir compte des objectifs de sécurité suivants :

| Objectif | Source |
|--|--|
| Isolation | Une stratégie de sécurité n’impose aucun transfert d’informations entre les machines virtuelles. Cette contrainte requiert des capacités dans Virtual Machine Manager (VMM) et du matériel pour l’isolation de la mémoire, des appareils, du réseau et des ressources managées telles que les données persistantes. |
| Intégrité de VMM | Pour garantir l’intégrité globale du système, l’intégrité des composants individuels de l’hyperviseur est établie et entretenue. |
| Intégrité de la plateforme | L’intégrité de l’hyperviseur dépend de l’intégrité du matériel et des logiciels sur lesquels il s’appuie. Bien que l’hyperviseur n’ait pas de contrôle direct sur l’intégrité de la plateforme, Azure s’appuie sur des mécanismes matériels et logiciels, tels que le processeur [Cerberus](project-cerberus.md), pour protéger et détecter l’intégrité de la plateforme sous-jacente. Les invités et VMM ne peuvent pas s’exécuter si l’intégrité de la plateforme est compromise. |
| ACCÈS RESTREINT | Les fonctions de gestion ne sont exercées que par les administrateurs autorisés connectés via des connexions sécurisées. Un principe de moindre privilège est appliqué par les mécanismes Azure RBAC (contrôle d’accès en fonction du rôle Azure). |
| Audit | Azure permet à la capacité d’audit de capturer et de protéger les données sur ce qui se passe sur un système afin qu’elles puissent être inspectées ultérieurement. |

L’approche de Microsoft pour renforcer l’hyperviseur Azure et le sous-système de virtualisation peut être divisée en trois catégories.

## <a name="strongly-defined-security-boundaries-enforced-by-the-hypervisor"></a>Des limites de sécurité fortement définies et appliquées par l’hyperviseur

L’hyperviseur Azure applique plusieurs limites de sécurité entre :

- Les partitions « invité » virtualisées et la partition privilégiée (« hôte »)
- Plusieurs invités
- Lui-même et l’hôte
- Lui-même et tous les invités

La confidentialité, l’intégrité et la disponibilité sont assurées pour les limites de sécurité de l’hyperviseur. Ces limites permettent de se défendre contre toute une série d’attaques, notamment les fuites d’informations sur les canaux secondaires, le déni de service et l’élévation des privilèges.

La limite de sécurité de l’hyperviseur fournit également une segmentation entre les locataires pour le trafic, les appareils virtuels, le stockage, les ressources de calcul et toutes les autres ressources de machine virtuelle.

## <a name="defense-in-depth-exploit-mitigations"></a>Atténuations de défense en profondeur de code malveillant exploitant une faille de sécurité

Dans le cas improbable où une limite de sécurité présente une vulnérabilité, l’hyperviseur Azure inclut plusieurs couches d’atténuations, notamment :

- Isolation du processus basé sur l’hôte hébergeant des composants entre machines virtuelles
- Sécurité basée sur la virtualisation (VBS) pour garantir l’intégrité des composants utilisateur et en mode noyau à partir d’un monde sécurisé
- Plusieurs niveaux d’atténuations de code malveillant exploitant une faille de sécurité. Les atténuations incluent la randomisation du layout d’espace d’adresse (ASLR), la prévention de l’exécution des données (DEP), la protection de code arbitraire, l’intégrité du flux de contrôle et la prévention de l’altération des données
- Initialisation automatique des variables de pile au niveau du compilateur
- API de noyau qui initialisent automatiquement les allocations de tas de noyau effectuées par Hyper-V

Ces atténuations sont conçues pour rendre irréalisable le développement d’un code malveillant exploitant une faille de sécurité pour introduire une vulnérabilité sur toutes les machines virtuelles.

## <a name="strong-security-assurance-processes"></a>De solides processus d’assurance de la sécurité

La surface d’attaque liée à l’hyperviseur comprend la mise en réseau logicielle, les appareils virtuels et toutes les surfaces de machines virtuelles. La surface d’attaque est suivie grâce à l’intégration automatisée de build, qui déclenche des révisions de sécurité périodiques.

Toutes les surfaces d’attaque de machine virtuelle sont modélisées d’après les menaces et le code est revu, mis à l’épreuve de tests à données aléatoires et testé par notre équipe RED pour rechercher les violations des limites de sécurité. Microsoft dispose d’un [programme de primes pour les bogues](https://www.microsoft.com/msrc/bounty-hyper-v) qui récompense la découverte de vulnérabilités pertinentes dans les versions de produit éligibles pour Microsoft Hyper-V.

> [!NOTE]
> En savoir plus sur [les processus d’assurance de la sécurité](../../azure-government/azure-secure-isolation-guidance.md#strong-security-assurance-processes) dans Hyper-V.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur ce que nous faisons pour assurer l’intégrité et la sécurité de la plateforme, consultez :

- [Sécurité de microprogramme](firmware.md)
- [Démarrage sécurisé](secure-boot.md)
- [Démarrage mesuré et attestation de l’hôte](measured-boot-host-attestation.md)
- [Project Cerberus](project-cerberus.md)
- [Chiffrement au repos](encryption-atrest.md)