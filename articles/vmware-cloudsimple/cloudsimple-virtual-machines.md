---
title: Présentation des machines virtuelles
titleSuffix: Azure VMware Solution by CloudSimple
description: En savoir plus sur les machines virtuelles CloudSimple et sur leurs avantages. Vous pouvez gérer des machines virtuelles VMware depuis le portail Azure.
author: sharaths-cs
ms.author: dikamath
ms.date: 08/20/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 6f9be035978667287b8b88ec1bb64b3882b0f929
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88141974"
---
# <a name="cloudsimple-virtual-machines-overview"></a>Vue d’ensemble des machines virtuelles CloudSimple

CloudSimple vous permet de gérer des machines virtuelles VMware depuis le Portail Azure.  Un cluster ou une liste de ressources partagées à partir de votre cluster vSphere sont gérés via Azure, grâce à leur mappage à votre abonnement.

Pour créer une machine virtuelle CloudSimple depuis Azure, un modèle de machine virtuelle doit exister sur votre vCenter de cloud privé.  Le modèle est utilisé pour personnaliser le système d’exploitation et les applications.  Le modèle de machine virtuelle peut être renforcé pour répondre aux stratégies de sécurité d’entreprise.  Vous pouvez utiliser le modèle pour créer des machines virtuelles, puis les utiliser dans le portail Azure à l’aide d’un modèle en libre-service.

## <a name="benefits"></a>Avantages

Les machines virtuelles CloudSimple du Portail Microsoft Azure fournissent un mécanisme en libre-service pour que les utilisateurs puissent créer et gérer des machines virtuelles VMware.

* Créer une machine virtuelle CloudSimple sur votre vCenter de cloud privé
* Gérer les propriétés de la machine virtuelle
  * Ajouter/supprimer des disques
  * Ajouter/supprimer des cartes d’interface réseau
* Opérations d’alimentation de votre machine virtuelle CloudSimple
  * Mise sous tension et mise hors tension
  * Réinitialiser une machine virtuelle
* Supprimer une machine virtuelle

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [Utiliser des machines virtuelles VMware sur Azure](quickstart-create-vmware-virtual-machine.md)
* Découvrez comment [Mapper votre abonnement Azure](azure-subscription-mapping.md)
