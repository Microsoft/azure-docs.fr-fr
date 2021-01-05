---
title: Segments réseau VMware HCX
description: Vous avez besoin de quatre réseaux pour VMware HCX.
ms.topic: include
ms.date: 11/23/2020
ms.openlocfilehash: 48894c532c97b70cde1473fb8b81f406ded70343
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95999624"
---
<!-- Used in avs-production-ready-deployment.md and tutorial-deploy-vmware-hcx.md -->

Quatre réseaux sont nécessaires pour VMware HCX :

- **Réseau de gestion :** En règle générale, il s’agit du même réseau de gestion que celui utilisé sur le cluster vSphere. Au minimum, identifiez deux adresses IP sur ce segment réseau pour VMware HCX. Selon votre déploiement, vous devrez peut-être utiliser des nombres plus importants.

   > [!NOTE]
   > La méthode que nous recommandons consiste à créer un réseau /26. Sur un réseau /26, vous pouvez utiliser jusqu’à 10 maillages de service et 60 contrôles d’extendeur réseau (-1 par maillage de service). Vous pouvez étirer huit réseaux par contrôle d’extendeur réseau en utilisant des clouds privés Azure VMware Solution.
   >
   
- **Réseau vMotion :** En règle générale, il s’agit du même réseau que celui utilisé pour vMotion sur le cluster vSphere.  Au minimum, identifiez deux adresses IP sur ce segment réseau pour VMware HCX. Selon votre déploiement, vous devrez peut-être utiliser des nombres plus importants.  

   Le réseau vMotion doit être exposé sur un commutateur virtuel distribué ou vSwitch0. Dans le cas contraire, modifiez l’environnement.

   > [!NOTE]
   > Ce réseau peut être privé (non routé).

- **Réseau de liaison montante :** Vous souhaitez créer un réseau pour la liaison montante VMware HCX et l’étendre à votre cluster vSphere via un groupe de ports. Au minimum, identifiez deux adresses IP sur ce segment réseau pour VMware HCX. Selon votre déploiement, vous devrez peut-être utiliser des nombres plus importants.  

   > [!NOTE]
   > La méthode que nous recommandons consiste à créer un réseau /26. Sur un réseau /26, vous pouvez utiliser jusqu’à 10 maillages de service et 60 contrôles d’extendeur réseau (-1 par maillage de service). Vous pouvez étirer huit réseaux par contrôle d’extendeur réseau en utilisant des clouds privés Azure VMware Solution.
   >
   
- **Réseau de réplication :** Ce paramètre est facultatif. Vous souhaitez créer un réseau pour la réplication VMware HCX et étendre ce réseau à votre cluster vSphere via un groupe de ports. Au minimum, identifiez deux adresses IP sur ce segment réseau pour VMware HCX. Selon votre déploiement, vous devrez peut-être utiliser des nombres plus importants.

   > [!NOTE]
   > Cette configuration n’est possible que si les hôtes du cluster local utilisent un réseau Replication VMkernel dédié.  Si aucun réseau Replication VMkernel dédié n’est défini pour votre cluster local, vous n’avez pas besoin de créer ce réseau.
