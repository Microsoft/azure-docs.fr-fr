---
title: Remplacer le comportement HTTP avec Azure CDN – Moteur de règles Verizon Premium
description: Le moteur de règles vous permet de personnaliser la manière dont Azure CDN de de Verizon Premium gère les requêtes HTTP, telles que le blocage de la remise de certains types de contenus, la définition d’une stratégie de mise en cache et la modification des en-têtes HTTP.
services: cdn
author: asudbring
ms.service: azure-cdn
ms.topic: how-to
ms.date: 05/31/2019
ms.author: allensu
ms.openlocfilehash: a49912bc2275e478d657f06587c4ddc830210d3a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87040206"
---
# <a name="override-http-behavior-using-the-azure-cdn-from-verizon-premium-rules-engine"></a>Modifier le comportement HTTP à l’aide du moteur de règles Azure CDN de Verizon Premium

[!INCLUDE [cdn-premium-feature](../../includes/cdn-premium-feature.md)]

## <a name="overview"></a>Vue d’ensemble

Le moteur de règles Azure CDN vous permet de personnaliser la gestion des requêtes HTTP. Par exemple, le blocage de la livraison de certains types de contenu, définir une stratégie de mise en cache ou modifier un en-tête HTTP. Ce didacticiel présente la création d’une règle qui modifie le comportement de mise en cache des ressources CDN. Pour plus d’informations sur la syntaxe de moteur de règles, consultez [Référence du moteur de règles Azure CDN](cdn-verizon-premium-rules-engine-reference.md).

## <a name="access"></a>Accès

Pour accéder au moteur de règles, vous devez d’abord sélectionner **Gérer** à partir du haut de la page **profil CDN** pour accéder à la page de gestion Azure CDN. Selon si votre point de terminaison est optimisé ou non pour l’accélération de site dynamique (DSA), vous accédez ensuite au moteur de règles avec l’ensemble de règles approprié pour votre type de point de terminaison :

- Points de terminaison optimisés pour la livraison web générale ou autres optimisations non-DSA :
    
    Sélectionnez l’onglet **HTTP Large**, puis sélectionnez **Moteur de règles**.

    ![Moteur de règles pour le protocole HTTP](./media/cdn-rules-engine/cdn-http-rules-engine.png)

- Points de terminaison optimisés pour DSA :
    
    Sélectionnez l’onglet **ADN**, puis sélectionnez **Moteur de règles**.
    
    ADN est un terme utilisé par Verizon pour spécifier le contenu DSA. Toutes les règles créées ici sont ignorées par les points de terminaison dans votre profil qui ne sont pas optimisés pour DSA.

    ![Moteur de règles pour DSA](./media/cdn-rules-engine/cdn-dsa-rules-engine.png)

## <a name="tutorial"></a>Didacticiel

1. Dans la page **Profil CDN**, sélectionnez **Gérer**.
   
    ![Bouton Gérer du profil CDN](./media/cdn-rules-engine/cdn-manage-btn.png)
   
    Le portail de gestion CDN s'ouvre.

2. Sélectionnez l’onglet **HTTP Large**, puis sélectionnez **Moteur de règles**.
   
    Les options d’une nouvelle règle s’affichent.
   
    ![Options de nouvelle règle CDN](./media/cdn-rules-engine/cdn-new-rule.png)
   
   > [!IMPORTANT]
   > L'ordre dans lequel plusieurs règles sont répertoriées affecte la façon dont elles sont gérées. Une règle ultérieure peut remplacer les actions spécifiées par une règle antérieure. Par exemple, si vous avez une règle qui autorise l’accès à une ressource selon une propriété de requête et une autre règle qui refuse l’accès à toutes les requêtes, la deuxième règle l’emporte sur la première. Les règles remplacent les règles antérieures uniquement si elles interagissent avec les mêmes propriétés.
   >

3. Entrez un nom dans la zone de texte **Nom / Description** .

4. Identifiez le type de requêtes auxquelles la règle s’applique. Utilisez la condition de correspondance par défaut, **Toujours**.
   
   ![Condition de correspondance de règle CDN](./media/cdn-rules-engine/cdn-request-type.png)
   
   > [!NOTE]
   > Plusieurs conditions de correspondance sont disponibles dans la liste déroulante. Pour plus d’informations sur la condition de correspondance actuellement sélectionnée, sélectionnez l’icône bleue d’information à sa gauche.
   >
   >  Pour obtenir la liste détaillée des expressions conditionnelles, consultez [Expressions conditionnelles du moteur de règles](cdn-verizon-premium-rules-engine-reference-match-conditions.md).
   >  
   > Pour obtenir la liste détaillée des conditions de correspondance, consultez [Conditions de correspondance du moteur de règles](cdn-verizon-premium-rules-engine-reference-match-conditions.md).
   >
   >

5. Cliquez sur le bouton **+** à côté de **Fonctionnalités** pour ajouter une fonctionnalité.  Dans la liste déroulante située à gauche, sélectionnez **Forcer l'âge maximal interne**.  Dans la zone de texte qui s'affiche, entrez **300**. Conservez les valeurs par défaut restantes.
   
   ![Fonctionnalité de règle CDN](./media/cdn-rules-engine/cdn-new-feature.png)
   
   > [!NOTE]
   > Plusieurs fonctionnalités sont disponibles dans la liste déroulante. Pour plus d’informations sur la fonctionnalité actuellement sélectionnée, sélectionnez l’icône bleue d’information à sa gauche.
   >
   > Pour **Forcer l’âge maximal interne**, le composant `Cache-Control` et les en-têtes `Expires` sont substitués pour contrôler quand le nœud de périmètre CDN actualise le composant à partir de l’origine. Dans cet exemple, le nœud de périmètre CDN met en cache le composant de 300 secondes, ou 5 minutes, avant qu’il actualise le composant à partir de son origine.
   >
   > Pour obtenir une liste détaillée des fonctionnalités, consultez [Fonctionnalités du moteur de règles](cdn-verizon-premium-rules-engine-reference-features.md).
   >
   >

6. Sélectionnez **Ajouter** pour enregistrer la nouvelle règle.  La nouvelle règle est en attente d'approbation. Une fois celle-ci approuvée, l’état passe de **XML en attente** à **XML actif**.
   
   > [!IMPORTANT]
   > Les modifications de règles peuvent prendre jusqu’à 10 minutes avant d’être propagées à travers Azure CDN.
   >
   >

## <a name="see-also"></a>Voir aussi

- [Vue d’ensemble d’Azure CDN](cdn-overview.md)
- [Informations de référence du moteur de règles](cdn-verizon-premium-rules-engine-reference.md)
- [Conditions de correspondance du moteur de règles](cdn-verizon-premium-rules-engine-reference-match-conditions.md)
- [Expressions conditionnelles du moteur de règles](cdn-verizon-premium-rules-engine-reference-conditional-expressions.md)
- [Fonctionnalités du moteur de règles](cdn-verizon-premium-rules-engine-reference-features.md)
- [Azure Fridays : les nouvelles fonctionnalités Premium puissantes du CDN Azure](https://azure.microsoft.com/documentation/videos/azure-cdns-powerful-new-premium-features/) (vidéo)