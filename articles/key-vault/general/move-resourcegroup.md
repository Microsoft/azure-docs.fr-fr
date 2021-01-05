---
title: Déplacer un coffre Azure Key Vault vers un autre groupe de ressources | Microsoft Docs
description: Conseils pour déplacer un coffre de clés vers un autre groupe de ressources.
services: key-vault
author: ShaneBala-keyvault
manager: ravijan
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 04/29/2020
ms.author: sudbalas
Customer intent: As a key vault administrator, I want to move my vault to another resource group.
ms.openlocfilehash: ea152e1d78ab1ea610eee5420394c89524673993
ms.sourcegitcommit: 2e72661f4853cd42bb4f0b2ded4271b22dc10a52
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92042380"
---
# <a name="moving-an-azure-key-vault-across-resource-groups"></a>Déplacer un coffre Azure Key Vault d’un groupe de ressources à un autre

## <a name="overview"></a>Vue d’ensemble

Le déplacement d’un coffre de clés entre des groupes de ressources est une fonctionnalité Key Vault prise en charge. Ce déplacement n’impacte pas les configurations de stratégie d’accès ou de pare-feu Key Vault. Les applications connectées et les principaux de service continuent en principe à fonctionner comme prévu.

> [!IMPORTANT]
> **Les coffres de clés utilisés pour le chiffrement de disque ne peuvent pas être déplacés.**
> Si vous utilisez un coffre de clés avec le chiffrement de disque pour une machine virtuelle, le coffre de clés ne peut pas être déplacé vers un autre groupe de ressources ou un abonnement lorsque le chiffrement de disque est activé. Vous devez désactiver le chiffrement de disque avant de déplacer le coffre de clés vers un nouveau groupe de ressources ou un nouvel abonnement. 

## <a name="design-considerations"></a>Remarques sur la conception

Votre organisation a peut-être implémenté Azure Policy avec une application ou des exclusions au niveau du groupe de ressources. Il peut y avoir un ensemble d’attributions de stratégies différent dans le groupe de ressources où se trouve actuellement votre coffre de clés et le groupe de ressources vers lequel vous voulez déplacer votre coffre de clés. Un conflit dans les exigences de stratégie peut entraîner un dysfonctionnement de vos applications.

### <a name="example"></a>Exemple

Vous avez une application connectée à un coffre de clés qui crée des certificats valides pendant deux ans. Le groupe de ressources vers lequel vous voulez déplacer votre coffre de clés a une attribution de stratégie qui empêche la création de certificats d’une durée de validité de plus d’un an. Après avoir déplacé votre coffre de clés vers le nouveau groupe de ressources, l’opération de création d’un certificat d’une validité de deux ans sera bloquée par une attribution de stratégie Azure.

### <a name="solution"></a>Solution

Accédez à la page Azure Policy dans le Portail Azure et vérifiez que les attributions de stratégie sont les mêmes pour votre groupe de ressources actuel et le groupe de ressources cible.

## <a name="procedure"></a>Procédure

1. Se connecter au portail Azure.
2. Accédez à votre coffre de clés.
3. Cliquer sur l’onglet « Vue d’ensemble »
4. Sélectionner le bouton « Déplacer »
5. Sélectionner « Déplacer vers un autre groupe de ressources » dans les options de la liste déroulante
6. Sélectionner le groupe de ressources vers lequel vous souhaitez déplacer votre coffre de clés
7. Confirmer la lecture de l’avertissement sur le déplacement des ressources
8. Sélectionner « OK »

Key Vault évalue à présent la validité du déplacement des ressources et vous alerte en cas d’erreur. Si aucune erreur n’est trouvée, les ressources sont déplacées. 
