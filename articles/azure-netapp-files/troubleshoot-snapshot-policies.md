---
title: Résoudre les problèmes liés aux stratégies d’instantanés pour Azure NetApp Files | Microsoft Docs
description: Décrit les messages d’erreur et les résolutions qui peuvent vous aider à résoudre les problèmes de gestion des stratégies d’instantanés pour Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 09/23/2020
ms.author: b-juche
ms.openlocfilehash: 6ba8b18876bdae2754a6a772ce3909ff2f5a71b7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91651002"
---
# <a name="troubleshoot-snapshot-policies"></a>Résoudre les problèmes de stratégies d’instantanés

Cet article décrit les scénarios d’erreur que vous pouvez rencontrer lors de la gestion des stratégies d’instantanés Azure NetApp Files. Il fournit également des solutions qui peuvent vous aider à résoudre les problèmes.

## <a name="error-conditions-and-resolutions"></a>Conditions d’erreur et résolutions 

|     État d’erreur    |     Résolution    |
|-|-|
| La création de la stratégie d’instantané échoue avec un nom de stratégie d’instantané non valide. | Une erreur se produit lors de la création de la stratégie d’instantané si le nom de votre stratégie d’instantané n’est pas valide. Les indications suivantes s’appliquent aux noms de stratégie d’instantané :  <ul><li> Le nom de la stratégie d’instantané ne peut pas contenir de caractères non ASCII ni de caractères spéciaux. </li> <li> Le nom de la stratégie d’instantané doit commencer par une lettre ou un chiffre, et peut contenir uniquement des lettres, des chiffres, des traits de soulignement (« _ ») et des traits d’union (« - »). </li> <li> Le nom de la stratégie d’instantané doit compter entre 1 et 64 caractères.  </li></ul> Modifiez le nom de la stratégie d’instantané conformément aux instructions.  |
| La création de la stratégie d’instantané échoue avec des valeurs non valides. | Azure NetApp Files ne parvient pas à créer une stratégie d’instantané si vous entrez une valeur non valide pour un champ comme `Number of snapshots to keep` ou `Minute on the hour to take snapshot`. Les valeurs valides sont les suivantes :  <ul><li>La valeur doit être un nombre valide.</li> <li>La valeur doit être comprise entre 0 et 59.</li></ul> Assurez-vous qu’une valeur valide est fournie pour les champs. | 
| La création de la stratégie d’instantané échoue avec l’erreur `Total number of snapshots to keep exceeds 255`. | Chaque volume peut disposer d’un [maximum de 255 instantanés](azure-netapp-files-resource-limits.md). La valeur maximale comprend la somme de tous les instantanés horaires, quotidiens, hebdomadaires et mensuels. <br> Diminuez la valeur de `Snapshots to keep`, puis réessayez. |
| L’attribution d’une stratégie à un volume échoue avec l’erreur `Total snapshot policy is over the max '255'`. | Chaque volume peut disposer d’un [maximum de 255 instantanés](azure-netapp-files-resource-limits.md). Lorsque la somme de tous les instantanés à la demande, horaires, quotidiens, hebdomadaires et mensuels dépasse la limite maximale, une erreur se produit. <br> Réduisez la valeur de `snapshots to keep` ou supprimez des instantanés à la demande, puis réessayez. | 

## <a name="next-steps"></a>Étapes suivantes  

* [Gérer les stratégies de capture instantanée](azure-netapp-files-manage-snapshots.md#manage-snapshot-policies)
