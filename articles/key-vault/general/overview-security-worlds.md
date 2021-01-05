---
title: Mondes de sécurité Azure Key Vault | Microsoft Docs
description: Azure Key Vault est un service mutualisé. Il utilise un pool de modules HSM dans chaque emplacement Azure. Tous les emplacements au sein d’une région géographique partagent une limite de chiffrement.
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
author: msmbaldwin
ms.author: mbaldwin
manager: rkarlin
ms.date: 07/03/2017
ms.openlocfilehash: 3584f83c5e1a5e83d069373395227b70c084eae9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81426745"
---
# <a name="azure-key-vault-security-worlds-and-geographic-boundaries"></a>Mondes de sécurité Azure Key Vault et limites géographiques

Azure Key Vault est un service mutualisé qui utilise un pool de modules de sécurité matériel (HSM) dans chaque emplacement Azure. 

Tous les modules de sécurité matériel présents dans les emplacements Azure au sein d’une même région géographique partagent la même limite de chiffrement (monde de sécurité Thales). Par exemple, les régions USA Est et USA Ouest partagent le même monde de sécurité car elles appartiennent à l’emplacement géographique USA. De même, tous les emplacements Azure au Japon partagent le même monde de sécurité, tout comme les emplacements Azure en Australie, en Inde, etc. 

## <a name="backup-and-restore-behavior"></a>Comportement de sauvegarde et de restauration

La sauvegarde d’une clé d’un coffre de clés dans un emplacement Azure peut être restaurée dans un coffre de clés situé dans un autre emplacement Azure, tant que les deux conditions suivantes sont remplies :

- Les deux emplacements Azure appartiennent à la même région géographique
- Les deux coffres de clés appartiennent au même abonnement Azure

Par exemple, la sauvegarde d’une clé effectuée par un abonnement spécifique dans un coffre de clés dans la région Inde Ouest peut uniquement être restaurée dans un coffre de clés appartenant aux mêmes abonnement et emplacement géographique (Inde Ouest, Inde Centre ou Inde Sud).

## <a name="regions-and-products"></a>Régions et produits

- [Régions Azure](https://azure.microsoft.com/regions/)
- [Produits Microsoft par région](https://azure.microsoft.com/regions/services/)

Les régions sont mappées aux mondes de sécurité, illustrés sous forme d’en-têtes principaux dans les tableaux :

Dans l’article « Produits par région », par exemple, l’onglet **États-Unis** contient les éléments USA EST, USA CENTRE, USA OUEST, tous mappés à la région États-Unis. 

>[!NOTE]
>US DOD EAST et US DOD CENTRAL sont des exceptions, dans la mesure où ils possèdent leurs propres mondes de sécurité. 

De même, dans l’onglet **Europe**, NORTH EUROPE et WEST EUROPE sont tous deux mappés à la région Europe. Cela vaut également pour l’onglet **Asie Pacifique**.



