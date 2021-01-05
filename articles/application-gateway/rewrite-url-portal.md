---
title: Réécriture d’URL et de chaîne de requête avec Azure Application Gateway – Portail Azure
description: Découvrez comment utiliser le Portail Azure pour configurer une passerelle Azure Application Gateway de façon à réécrire l’URL et la chaîne de requête.
services: application-gateway
author: surajmb
ms.service: application-gateway
ms.topic: how-to
ms.date: 7/16/2020
ms.author: surmb
ms.openlocfilehash: ec58c6f97efdbcb91071bcea98bbbc614833246d
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92215771"
---
# <a name="rewrite-url-with-azure-application-gateway---azure-portal-preview"></a>Réécriture d’URL avec Azure Application Gateway – Portail Azure (préversion)

Cet article explique comment utiliser le Portail Azure pour configurer une instance [SKU Application Gateway v2](application-gateway-autoscaling-zone-redundant.md) de façon à réécrire l’URL.

>[!NOTE]
> La fonctionnalité de réécriture d’URL est en préversion et est disponible uniquement pour les références SKU Standard_v2 et WAF_v2 d’Application Gateway. Son utilisation n’est pas recommandée en environnement de production. Pour en savoir plus sur les préversions, consultez les [conditions d’utilisation](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="before-you-begin"></a>Avant de commencer

Vous devez disposer d’une instance de la référence SKU Application Gateway v2 pour suivre la procédure décrite dans cet article. La réécriture d’URL n’est pas prise en charge dans la référence SKU v1. Si vous ne disposez pas de la référence SKU v2, créez une instance de [référence SKU Application Gateway v2](tutorial-autoscale-ps.md) avant de commencer.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com/) avec votre compte Azure.

## <a name="configure-url-rewrite"></a>Configurer la réécriture d’URL

Dans l’exemple ci-dessous, chaque fois que l’URL de la demande contient */article* , son chemin et sa chaîne de requête sont réécrits.

`contoso.com/article/123/fabrikam` -> `contoso.com/article.aspx?id=123&title=fabrikam`

1. Sélectionnez **Toutes les ressources** , puis sélectionnez votre passerelle d’application.

2. Dans le volet gauche, sélectionnez **Réécrit** .

3. Sélectionnez **Jeu de réécriture**  :

    :::image type="content" source="./media/rewrite-url-portal/rewrite-url-portal-1.png" alt-text="Ajout d’un jeu de réécriture":::

4. Fournissez un nom pour le jeu de réécriture et associer-le à une règle d’acheminement :

    a. Entrez le nom du jeu de réécriture défini dans la zone **Nom** .
    
    b. Sélectionnez une ou plusieurs des règles présentées dans la liste **Règles d’acheminement associées** . La règle d’acheminement permet d’associer la configuration de réécriture à l’écouteur source. Seules les règles d’acheminement qui n’ont pas été associés à d’autres jeux de réécriture peuvent être sélectionnées. Les autres sont grisées.
    
    c. Sélectionnez **Suivant** .
    
    :::image type="content" source="./media/rewrite-url-portal/rewrite-url-portal-2.png" alt-text="Ajout d’un jeu de réécriture":::

5. Créer une règle de réécriture :

    a. Sélectionnez **Ajouter une règle de réécriture** .
    
    :::image type="content" source="./media/rewrite-url-portal/rewrite-url-portal-3.png" alt-text="Ajout d’un jeu de réécriture":::
    
    b. Entrez un nom pour la règle de réécriture dans la zone **Nom de règle de réécriture** . Entrez un numéro dans la zone **Séquence de règle** .

6. Dans cet exemple, nous allons réécrire le chemin d’URL et la chaîne de requête d’URL uniquement lorsque le chemin contient */article* . Ajoutez une condition permettant d’évaluer si c’est le cas.

    a. Sélectionnez **Ajouter une condition** , puis sélectionnez la zone contenant les instructions **Si** pour la développer.
    
    b. Dans cet exemple, nous souhaitons vérifier le modèle */article* dans le chemin d’URL. Sélectionnez **Variable serveur** dans la liste **Type de variable à vérifier** .
    
    c. Dans la liste **Variable serveur** , sélectionnez uri_path.
    
    d. Sous **Respect de la casse** , sélectionnez **Non** .
    
    e. Dans la liste **Opérateur** , sélectionnez **Égal (=)** .
    
    f. Entrez un modèle d’expression régulière. Dans cet exemple, nous allons utiliser le modèle `.*article/(.*)/(.*)`.
    
      () permet de capturer la sous-chaîne pour plus tard, quand il faudra composer l’expression permettant de réécrire le chemin d’URL. Vous pourrez trouver plus d’informations [ici](rewrite-http-headers-url.md#capturing).

    g. Sélectionnez **OK** .

    :::image type="content" source="./media/rewrite-url-portal/rewrite-url-portal-4.png" alt-text="Ajout d’un jeu de réécriture":::

 

7. Ajoutez une action pour réécrire l’URL et son chemin :

   a. Dans la liste **Réécrire le type** , sélectionnez **URL** .

   b. Dans la liste **type d’Action** , sélectionnez **Définir** .

   c. Sous **Composants** , sélectionnez **Chemin d’URL et chaîne de requête d’URL** .

   d. Dans **Valeur du chemin d’URL** , entrez la nouvelle valeur du chemin. Dans cet exemple, nous utiliserons **/article.aspx** . 

   e. Dans **Valeur de la chaîne de requête URL** , entrez la nouvelle valeur de la chaîne de requête d’URL. Dans cet exemple, nous utiliserons **id={var_uri_path_1}&title={var_uri_path_2}** .
    
    `{var_uri_path_1}` et `{var_uri_path_1}` permettent d’extraire les sous-chaînes capturées lors de l’évaluation de la condition dans cette expression `.*article/(.*)/(.*)`.
    
   f. Sélectionnez **OK** .

    :::image type="content" source="./media/rewrite-url-portal/rewrite-url-portal-5.png" alt-text="Ajout d’un jeu de réécriture":::

8. Cliquez sur **Créer** pour créer le jeu de réécriture.

9. Vérifiez que le nouveau jeu de réécriture apparaît dans la liste des jeux de réécriture :

    :::image type="content" source="./media/rewrite-url-portal/rewrite-url-portal-6.png" alt-text="Ajout d’un jeu de réécriture":::

## <a name="verify-url-rewrite-through-access-logs"></a>Vérification de la réécriture d’URL au moyen des journaux d’accès

Observez les champs ci-dessous dans les journaux d’accès pour voir si la réécriture d’URL s’est produite conformément à vos attentes.

* **originalRequestUriWithArgs** : ce champ contient l’URL de demande d’origine.
* **requestUri** : ce champ contient l’URL obtenue après l’opération de réécriture sur Application Gateway.

Pour plus d’informations sur l’ensemble des champs des journaux d’accès, cliquez [ici](application-gateway-diagnostics.md#for-application-gateway-and-waf-v2-sku).

##  <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la configuration des réécritures dans certains cas d’usage courants, consultez [Scénarios de réécriture courants](rewrite-http-headers.md).
