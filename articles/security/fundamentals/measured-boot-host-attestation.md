---
title: Démarrage mesuré des microprogrammes et attestation de l’hôte – Sécurité Azure
description: Présentation technique du démarrage mesuré des microprogrammes et de l’attestation de l’hôte d’Azure.
author: yosharm
ms.service: security
ms.subservice: security-fundamentals
ms.topic: article
ms.author: terrylan
manager: rkarlin
ms.date: 11/10/2020
ms.openlocfilehash: f4906d4e0590df047bac4ee15cb0e996a59c3d5b
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498861"
---
# <a name="measured-boot-and-host-attestation"></a>Démarrage mesuré et attestation de l’hôte
Cet article explique la façon dont Microsoft garantit l’intégrité et la sécurité des hôtes par le biais du démarrage mesuré et de l’attestation de l’hôte.

## <a name="measured-boot"></a>Démarrage mesuré

Le [module de plateforme sécurisée](/windows/security/information-protection/tpm/trusted-platform-module-top-node) (TPM) est un composant d’audit sécurisé par chiffrement et inviolable avec un microprogramme fourni par un tiers de confiance. Le journal de configuration du démarrage contient des mesures chaînées par hachage enregistrées dans ses registres de configuration de plateforme (PCR) lorsque l’hôte a subi la dernière séquence de démarrage. Ce processus d’enregistrement est représenté dans la figure suivante. L’ajout incrémentiel d’une mesure précédemment hachée au hachage de la mesure suivante et l’exécution de l’algorithme de hachage sur l’union permet de réaliser le chaînage de hachage.

![Diagramme montrant le chaînage de hachage du service d’attestation d’hôte.](./media/measured-boot-host-attestation/hash-chaining.png)

L’attestation est effectuée lorsqu’un hôte fournit la preuve de son état de configuration à l’aide de son journal de configuration du démarrage (TCGLog). La falsification d’un journal de démarrage est difficile, car le module de plateforme sécurisée n’expose pas ses valeurs PCR autres que les opérations de lecture et d’extension. En outre, les informations d’identification fournies par le service d’attestation d’hôte sont scellées à des valeurs PCR spécifiques. L’utilisation du chaînage de hachage rend impossible l’usurpation ou le descellement des informations d’identification hors bande.

## <a name="host-attestation-service"></a>Service d’attestation d’hôte

Le service d’attestation d’hôte est une mesure préventive qui vérifie si les ordinateurs hôtes sont dignes de confiance avant de pouvoir interagir avec les données ou charges de travail des clients. Le service d’attestation d’hôte vérifie en validant un état de conformité (preuve vérifiable de la conformité de l’hôte) envoyé par chaque hôte par rapport à une stratégie d’attestation (définition de l’état sécurisé). L’intégrité de ce système est assurée par une [racine de confiance](https://www.uefi.org/sites/default/files/resources/UEFI%20RoT%20white%20paper_Final%208%208%2016%20%28003%29.pdf) fournie par un TPM.

Le service d’attestation d’hôte est présent dans chaque cluster Azure au sein d’un environnement spécialisé verrouillé. L’environnement verrouillé comprend d’autres services de contrôle qui participent au protocole de démarrage de l’ordinateur hôte. Une infrastructure à clé publique (PKI) agit comme intermédiaire pour valider la provenance des demandes d’attestation et comme émetteur d’identité (sous réserve de la réussite de l’attestation de l’hôte). Les informations d’identification postérieures à l’attestation et délivrées à l’hôte d’attestation sont scellées à son identité. Seul l’hôte demandeur peut désceller les informations d’identification et les exploiter pour obtenir des autorisations incrémentielles. Cela évite des attaques de l’intercepteur et d’usurpation d’identité.

Si un hôte Azure arrive d’usine avec une configuration de sécurité incompatible ou est altéré dans le centre de donnes, son TCGLog contient des indicateurs de compromission signalés par le service d’attestation d’hôte lors de l’attestation suivante, ce qui provoque un échec d’attestation. Les échecs d’attestation empêchent la flotte Azure d’approuver l’hôte incriminé. Cette prévention bloque efficacement toutes les communications vers et depuis l’hôte et déclenche un workflow d’incident. Une enquête et une analyse post-mortem détaillée sont effectuées pour déterminer les causes racines et les éventuelles indications de compromission. Une fois que l’analyse est terminée, un hôte est corrigé et a la possibilité de rejoindre la flotte Azure et d’assumer les charges de travail des clients.

Voici une architecture de haut niveau du service d’attestation d’hôte :

![Diagramme illustrant l’architecture du service d’attestation d’hôte.](./media/measured-boot-host-attestation/host-attestation-arch.png)

## <a name="attestation-measurements"></a>Mesures d’attestation

Voici quelques exemples des nombreuses mesures capturées aujourd’hui.

### <a name="secure-boot-and-secure-boot-keys"></a>Démarrage sécurisé et clés de démarrage sécurisé
En confirmant que les codes de hachage de bases de données de signatures et de signatures révoquées sont corrects, le service d’attestation d’hôte garantit que l’agent client considère les logiciels appropriés comme fiables. En validant les signatures de la base de données de clés d’inscription de clé publique et de la clé de plateforme publique, le service d’attestation d’hôte confirme que seules les parties approuvées sont autorisées à modifier les définitions des logiciels considérés comme étant dignes de confiance. Enfin, en veillant à ce que le démarrage sécurisé soit actif, le service d’attestation d’hôte confirme que ces définitions sont appliquées.

### <a name="debug-controls"></a>Contrôles de débogage
Les débogueurs sont des outils puissants pour les développeurs. Toutefois, le libre accès à la mémoire et aux autres commandes de débogage pourrait affaiblir la protection des données et l’intégrité du système s’il était accordé à un tiers non approuvé. Le service d’attestation d’hôte garantit que tout type de débogage est désactivé au démarrage sur les ordinateurs de production.

### <a name="code-integrity"></a>Intégrité du code
Le [démarrage sécurisé](secure-boot.md) d’UEFI garantit que seuls les logiciels de bas niveau approuvés peuvent s’exécuter pendant la séquence de démarrage. Toutefois, les mêmes vérifications doivent également être appliquées dans l’environnement après le démarrage aux pilotes et autres exécutables avec accès en mode noyau. À cette fin, une stratégie d’intégrité du code (CI) est utilisée pour définir les pilotes, les fichiers binaires et les autres exécutables considérés comme étant dignes de confiance en spécifiant des signatures valides et non valides. Ces stratégies sont appliquées. Les violations de stratégie génèrent des alertes qui sont adressées à l’équipe de réponse aux incidents de sécurité à des fins d’investigation.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur ce que nous faisons pour assurer l’intégrité et la sécurité de la plateforme, consultez :

- [Sécurité de microprogramme](firmware.md)
- [Démarrage sécurisé](secure-boot.md)
- [Project Cerberus](project-cerberus.md)
- [Chiffrement au repos](encryption-atrest.md)
- [Sécurité de l’hyperviseur](hypervisor.md)