---
title: Sécurité des microprogrammes - Sécurité Azure
description: Découvrez comment Microsoft sécurise le matériel et les microprogrammes d’Azure.
author: yosharm
ms.service: security
ms.subservice: security-fundamentals
ms.topic: article
ms.author: terrylan
manager: rkarlin
ms.date: 11/10/2020
ms.openlocfilehash: 8233cc714d977083f4d55716d35c0b7094a069ea
ms.sourcegitcommit: dc342bef86e822358efe2d363958f6075bcfc22a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94557387"
---
# <a name="firmware-security"></a>Sécurité de microprogramme
Cet article décrit comment Microsoft sécurise l’écosystème matériel du cloud et les chaînes d’approvisionnement.

## <a name="securing-the-cloud-hardware-ecosystem"></a>Sécurisation de l’écosystème matériel du cloud
Microsoft travaille activement en partenariat dans l’écosystème matériel du cloud pour améliorer la sécurité de façon continue :

- Collaboration avec des partenaires pour le matériel et les microprogrammes d’Azure (comme des fabricants de composants et des intégrateurs de systèmes) pour répondre aux exigences de sécurité du matériel et des microprogrammes d’Azure.

- Possibilité pour les partenaires d’effectuer l’évaluation et l’amélioration continues de la posture de sécurité de leurs produits en utilisant les exigences définies par Microsoft dans des domaines comme :

  - Démarrage sécurisé des microprogrammes
  - Récupération sécurisée des microprogrammes
  - Mise à jour sécurisée des microprogrammes
  - Chiffrement des microprogrammes
  - Matériel verrouillé
  - Télémétrie de débogage détaillée
  - Prise en charge système du matériel TPM 2.0 pour activer le démarrage mesuré

- Implication et contribution au projet de sécurité [Open Compute Project (OCP)](https://www.opencompute.org/wiki/Security) via le développement de spécifications. Les spécifications favorisent la cohérence et la clarté pour la conception et l’architecture sécurisées dans l’écosystème.

   > [!NOTE]
   > La spécification [Hardware Secure Boot](https://docs.google.com/document/d/1Se1Dd-raIZhl_xV3MnECeuu_I0nF-keg4kqXyK4k4Wc/edit#heading=h.5z2d7x9gbhk0) (Démarrage sécurisé du matériel) est un exemple de notre contribution au projet de sécurité OCP.

## <a name="securing-hardware-and-firmware-supply-chains"></a>Sécurisation des chaînes d’approvisionnement du matériel et des microprogrammes
Les fournisseurs et les vendeurs de matériel cloud pour Azure doivent également adhérer aux processus et exigences de sécurité de la chaîne logistique développés par Microsoft. Les processus de développement et de déploiement du matériel et des microprogrammes doivent suivre les processus [SDL (Security Development Lifecycle)](https://www.microsoft.com/securityengineering/sdl) de Microsoft, comme :

- Modélisation des menaces
- Révisions de conception sécurisée
- Révisions des microprogrammes et tests de pénétration
- Environnements de génération et de test sécurisés
- Gestion des vulnérabilités de sécurité et réponse aux incidents

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur ce que nous faisons pour assurer l’intégrité et la sécurité de la plateforme, consultez :

- [Démarrage sécurisé](secure-boot.md)
- [Démarrage mesuré et attestation de l’hôte](measured-boot-host-attestation.md)
- [Project Cerberus](project-cerberus.md)
- [Chiffrement au repos](encryption-atrest.md)
- [Sécurité de l’hyperviseur](hypervisor.md)