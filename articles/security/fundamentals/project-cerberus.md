---
title: Intégrité des microprogrammes – Sécurité Azure
description: En savoir plus sur les mesures de chiffrement pour garantir l’intégrité des microprogrammes.
author: yosharm
ms.service: security
ms.subservice: security-fundamentals
ms.topic: article
ms.author: terrylan
manager: rkarlin
ms.date: 11/10/2020
ms.openlocfilehash: f085858a9d550623704efd4f051ed525e55a37e0
ms.sourcegitcommit: dc342bef86e822358efe2d363958f6075bcfc22a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94557350"
---
# <a name="project-cerberus"></a>Project Cerberus

Cerberus est une racine de confiance matérielle conforme à la norme NIST 800-193 avec une identité qui ne peut pas être clonée. Cerberus est conçu pour améliorer encore davantage la sécurité de l’infrastructure Azure en fournissant un point d’ancrage approuvé et solide pour l’intégrité des microprogrammes.

## <a name="enabling-an-anchor-of-trust"></a>Activation d’une ancre de confiance
Chaque processeur Cerberus a une identité de chiffrement unique qui est établie à l’aide d’une chaîne de certificats signée enracinée dans une autorité de certification Microsoft. Les mesures obtenues à partir de Cerberus peuvent être utilisées pour valider l’intégrité des composants tels que :

- Host
- Baseboard Management Controller (BMC)
- Tous les périphériques, y compris les cartes d’interface réseau et le [système sur une puce](https://en.wikipedia.org/wiki/System_on_a_chip) (SoC)

Cette ancre de confiance permet de défendre les microprogrammes de la plateforme contre les éléments suivants :

- Binaires de microprogramme compromis s’exécutant sur la plateforme
- Programme malveillant et pirates informatiques qui exploitent les bogues dans le système d’exploitation, l’application ou l’hyperviseur
- Certains types d’attaques par chaîne d’approvisionnement (fabrication, assembly, transit)
- Employés malveillants avec des privilèges Administrateur ou l’accès au matériel

## <a name="cerberus-attestation"></a>Attestation Cerberus
Cerberus authentifie l’intégrité des microprogrammes pour les composants du serveur à l’aide d’un manifeste de microprogramme de plateforme (PFM). Le PFM définit une liste de versions de microprogramme autorisées et fournit une mesure de plateforme au [service d’attestation d’hôte](measured-boot-host-attestation.md) Azure. Le service d’attestation d’hôte valide les mesures et prend la décision de n’autoriser que les hôtes de confiance à rejoindre la flotte Azure et à accueillir les charges de travail des clients.

Conjointement avec le service d’attestation d’hôte, les capacités de Cerberus améliorent et favorisent une infrastructure de production Azure hautement sécurisée.

> [!NOTE]
> Pour en savoir plus, consultez les informations relatives au [projet Cerberus](https://github.com/opencomputeproject/Project_Olympus/tree/master/Project_Cerberus) sur GitHub.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur ce que nous faisons pour assurer l’intégrité et la sécurité de la plateforme, consultez :

- [Sécurité de microprogramme](firmware.md)
- [Démarrage sécurisé](secure-boot.md)
- [Démarrage mesuré et attestation de l’hôte](measured-boot-host-attestation.md)
- [Chiffrement au repos](encryption-atrest.md)
- [Sécurité de l’hyperviseur](hypervisor.md)