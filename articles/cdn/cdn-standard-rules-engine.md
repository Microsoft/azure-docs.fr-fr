---
title: Utiliser un moteur de règles pour appliquer le protocole HTTPS dans Azure CDN Standard | Microsoft Docs
description: Utilisez le moteur de règles de Microsoft Standard Azure Content Delivery Network (Azure CDN) pour personnaliser la façon dont Azure CDN gère les requêtes HTTP, notamment le blocage de la livraison de certains types de contenu, la définition d’une stratégie de mise en cache et la modification des en-têtes HTTP. Dans cet article, vous allez apprendre à créer une règle pour rediriger les utilisateurs vers HTTPS.
services: cdn
author: asudbring
ms.service: azure-cdn
ms.topic: how-to
ms.date: 11/01/2019
ms.author: allensu
ms.openlocfilehash: 1c596bf10d8afbce504b5abf04eacb877989521a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84887532"
---
# <a name="set-up-the-standard-rules-engine-for-azure-cdn"></a>Configurer le moteur de règles Standard pour Azure CDN

Cet article explique comment configurer et utiliser le moteur de règles Standard pour Azure Content Delivery Network (Azure CDN).

## <a name="standard-rules-engine"></a>Moteur de règles Standard

Vous pouvez utiliser le moteur de règles Standard pour Azure CDN afin de personnaliser la façon dont les requêtes HTTP sont gérées. Par exemple, vous pouvez utiliser le moteur de règles afin que la livraison de contenu utilise des protocoles spécifiques, de définir une stratégie de mise en cache ou de modifier un en-tête HTTP. Cet article montre comment créer une règle qui redirige automatiquement les utilisateurs vers HTTPS. 

> [!NOTE]
> Le moteur de règles décrit dans cet article est uniquement disponible pour Azure CDN Standard de Microsoft. 

## <a name="redirect-users-to-https"></a>Rediriger les utilisateurs vers HTTPS

1. Dans vos profils Microsoft, accédez à Azure Content Delivery Network.

1. Dans la page **Profil CDN**, sélectionnez le point de terminaison pour lequel vous souhaitez créer des règles.
  
1. Sélectionnez l’onglet **Moteur de règles**.
   
    Le volet **Moteur de règles** s’ouvre et affiche la liste des règles globales disponibles. 
   
    [![Page de nouvelles règles Azure CDN](./media/cdn-standard-rules-engine/cdn-new-rule.png)](./media/cdn-standard-rules-engine/cdn-new-rule.png#lightbox)
   
   > [!IMPORTANT]
   > L’ordre dans lequel plusieurs règles sont listées affecte la façon dont elles sont gérées. Les actions spécifiées dans une règle peuvent être remplacées par une règle suivante.
   >

1. Sélectionnez **Ajouter une règle** et entrez un nom de règle. Les noms de règle doivent commencer par une lettre et peuvent contenir uniquement des chiffres et des lettres.

1. Pour identifier le type de requêtes auxquelles la règle s’applique, créez une condition de correspondance :
    1. Sélectionnez **Ajouter une condition**, puis sélectionnez la condition de correspondance **Protocole de demande**.
    1. Pour **Opérateur**, sélectionnez **Est égal à**.
    1. Pour **Valeur**, sélectionnez **HTTP**.
   
   [![Condition de correspondance de règle Azure CDN](./media/cdn-standard-rules-engine/cdn-match-condition.png)](./media/cdn-standard-rules-engine/cdn-match-condition.png#lightbox)
   
   > [!NOTE]
   > Vous pouvez sélectionner plusieurs conditions de correspondance dans la liste déroulante **Ajouter une condition**. Pour obtenir la liste détaillée des conditions de correspondance, consultez [Conditions de correspondance du moteur de règles Standard](cdn-standard-rules-engine-match-conditions.md).
   
1. Sélectionnez l’action à appliquer aux requêtes qui répondent à la condition de correspondance :
   1. Sélectionnez **Ajouter une action**, puis sélectionnez **Redirection de l’URL**.
   1. Pour **Type**, sélectionnez **Trouvé (302)** .
   1. Pour **Protocole**, sélectionnez **HTTPS**.
   1. Laissez tous les autres champs vides pour utiliser les valeurs entrantes.
   
   [![Action de règle Azure CDN](./media/cdn-standard-rules-engine/cdn-action.png)](./media/cdn-standard-rules-engine/cdn-action.png#lightbox)
   
   > [!NOTE]
   > Vous pouvez sélectionner plusieurs actions dans la liste déroulante **Ajouter une action**. Pour obtenir une liste détaillée des actions, consultez [Actions du moteur de règles Standard](cdn-standard-rules-engine-actions.md).

6. Sélectionnez **Enregistrer** pour enregistrer la nouvelle règle. La règle peut désormais être utilisée.
   
   > [!IMPORTANT]
   > Les modifications de règle peuvent prendre jusqu’à 15 minutes avant d’être propagées à travers Azure CDN.
   >
   

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble d’Azure CDN](cdn-overview.md)
- [Informations de référence sur le moteur de règles Standard](cdn-standard-rules-engine-reference.md)
- [Conditions de correspondance du moteur de règles Standard](cdn-standard-rules-engine-match-conditions.md)
- [Actions du moteur de règles Standard](cdn-standard-rules-engine-actions.md)
