---
title: Réécrire les en-têtes de requête et de réponse HTTP sur le portail – Azure Application Gateway
description: Découvrez comment utiliser le portail Azure pour configurer une instance d’Azure Application Gateway pour réécrire les en-têtes HTTP dans les requêtes et réponses passant par la passerelle
services: application-gateway
author: abshamsft
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/13/2019
ms.author: absha
ms.custom: mvc
ms.openlocfilehash: 79314db13531f1fcf518c7931d4a1aa9158a172b
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397193"
---
# <a name="rewrite-http-request-and-response-headers-with-azure-application-gateway---azure-portal"></a>Réécrire les en-têtes de requête et de réponse HTTP avec Azure Application Gateway - Azure portal

Cet article explique comment utiliser le portail Microsoft Azure pour configurer une instance de la [référence SKU Application Gateway v2](./application-gateway-autoscaling-zone-redundant.md) afin de réécrire les en-têtes HTTP des requêtes et des réponses.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="before-you-begin"></a>Avant de commencer

Vous devez disposer d’une instance de la référence SKU Application Gateway v2 pour suivre la procédure décrite dans cet article. La réécriture des en-têtes n’est pas prise en charge dans la référence SKU v1. Si vous ne disposez pas de la référence SKU v2, créez une instance de [référence SKU Application Gateway v2](./tutorial-autoscale-ps.md) avant de commencer.

## <a name="create-required-objects"></a>Créer les objets requis

Pour configurer la réécriture d’en-tête HTTP, vous devez effectuer ces étapes.

1. Créez les objets nécessaires à la réécriture d’en-tête HTTP :

   - **Action de réécriture**  : permet de spécifier la requête et les champs d’en-tête de requête que vous souhaitez réécrire, ainsi que la nouvelle valeur des en-têtes. Vous pouvez associer une ou plusieurs conditions de réécriture à une action de réécriture.

   - **Condition de réécriture**  : configuration facultative. Les conditions de réécriture évaluent le contenu des requêtes et réponses HTTP(S). L’action de réécriture se produit si la requête ou la réponse HTTP(S) correspondent à la condition de réécriture.

     Si vous associez plusieurs conditions à une action, cette dernière ne se produit que lorsque toutes les conditions sont remplies. En d’autres termes, il s’agit d’une opération AND logique.

   - **Règle de réécriture**  : contient plusieurs combinaisons d’actions/conditions de réécriture.

   - **Séquence de règle**  : permet de déterminer l’ordre dans lequel les règles de réécriture s’exécutent. Cette configuration est utile quand vous disposez de plusieurs règles de réécriture dans un jeu de réécritures. Une règle de réécriture qui présente une valeur de séquence de règle inférieure s’exécute en premier. Si vous attribuez la même valeur de séquence de règle à deux règles de réécriture, l’ordre d’exécution n’est pas déterministe.

   - **Jeu de réécritures**  : contient plusieurs règles de réécriture qui seront associées à une règle de routage de requête.

2. Attachez le jeu de réécriture à une règle d’acheminement. La configuration de réécriture est attachée à l’écouteur source via la règle de routage. Quand vous utilisez une règle de routage de base, la configuration de réécriture d’en-tête est associée à un écouteur source et correspond à une réécriture d’en-tête globale. Quand vous utilisez une règle de routage basée sur le chemin, la configuration de réécriture d’en-tête est définie sur le mappage du chemin d’URL. Dans ce cas, elle s’applique uniquement à la zone de chemin spécifique d’un site.

Vous pouvez créer plusieurs jeux de réécritures d’en-tête HTTP et appliquer chacun d’eux à différents écouteurs. En revanche, vous ne pouvez appliquer qu’un seul jeu de réécritures à un écouteur spécifique.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com/) avec votre compte Azure.

## <a name="configure-header-rewrite"></a>Configurer la réécriture d’en-tête

Dans cet exemple, nous allons modifier une URL de redirection en réécrivant l’en-tête de localisation dans la réponse HTTP envoyée par une application back-end.

1. Sélectionnez **Toutes les ressources** , puis sélectionnez votre passerelle d’application.

2. Dans le volet gauche, sélectionnez **Réécrit**.

3. Sélectionnez **Jeu de réécriture**  :

   ![Ajouter un jeu de réécriture](media/rewrite-http-headers-portal/add-rewrite-set.png)

4. Fournissez un nom pour le jeu de réécriture et associer-le à une règle d’acheminement :

   - Entrez le nom du jeu de réécriture défini dans la zone **Nom**.
   - Sélectionnez une ou plusieurs des règles présentées dans la liste **Règles d’acheminement associées**. Vous pouvez sélectionner uniquement les règles qui n’ont pas été associés à d’autres jeux de réécriture. Les règles qui ont déjà été associées à d’autres jeux de réécriture ne sont pas disponibles.
   - Sélectionnez **Suivant**.
   
     ![Ajouter un nom et une association](media/rewrite-http-headers-portal/name-and-association.png)

5. Créer une règle de réécriture :

   - Sélectionnez **Ajouter une règle de réécriture**.

     ![Ajouter une règle de réécriture](media/rewrite-http-headers-portal/add-rewrite-rule.png)

   - Entrez un nom pour la règle de réécriture dans la zone **Nom de règle de réécriture**. Entrez un numéro dans la zone **Séquence de règle**.

     ![Ajouter un nom de règle de réécriture](media/rewrite-http-headers-portal/rule-name.png)

6. Dans cet exemple, nous allons réécrire l’en-tête d’emplacement uniquement lorsqu’il contient une référence à azurewebsites.net. Pour ce faire, ajoutez une condition destinée à évaluer si l’en-tête d’emplacement de la réponse contient ou non azurewebsites.net.

   - Sélectionnez **Ajouter une condition** , puis sélectionnez la zone contenant les instructions **Si** pour la développer.

     ![Ajouter une condition](media/rewrite-http-headers-portal/add-condition.png)

   - Dans la liste **Type de variable à vérifier** , sélectionnez **En-tête HTTP**.

   - Dans la liste **Type d’en-tête** , sélectionnez **Réponse**.

   - Comme dans cet exemple, nous évaluons l’en-tête d’emplacement, qui est un en-tête commun, sélectionnez **En-tête commun** sous **Nom d’en-tête**.

   - Dans la liste **En-tête commun** , sélectionnez **Emplacement**.

   - Sous **Respect de la casse** , sélectionnez **Non**.

   - Dans la liste **Opérateur** , sélectionnez **Égal (=)** .

   - Entrez un modèle d’expression régulière. Dans cet exemple, nous allons utiliser le modèle `(https?):\/\/.*azurewebsites\.net(.*)$`.

   - Sélectionnez **OK**.

     ![Configurer une condition Si](media/rewrite-http-headers-portal/condition.png)

7. Ajoutez une action pour réécrire l’en-tête d’emplacement :

   - Dans la liste **type d’Action** , sélectionnez **Définir**.

   - Dans la liste **Type d’en-tête** , sélectionnez **Réponse**.

   - Sous **Nom d’en-tête** , sélectionnez **En-tête commun**.

   - Dans la liste **En-tête commun** , sélectionnez **Emplacement**.

   - Entrez la valeur de l’en-tête. Dans cet exemple, nous allons utiliser `{http_resp_Location_1}://contoso.com{http_resp_Location_2}` en tant que valeur de l’en-tête. Cette valeur remplacera *azurewebsites.net* par *contoso.com* dans l’en-tête d’emplacement.

   - Sélectionnez **OK**.

     ![Ajouter une action](media/rewrite-http-headers-portal/action.png)

8. Sélectionnez **Créer** pour créer le jeu de réécriture :

   ![Sélectionner Créer](media/rewrite-http-headers-portal/create.png)

9. La vue Jeu de réécriture s’ouvre. Vérifiez que le jeu de réécriture que vous avez créé se trouve dans la liste des jeux de réécriture :

   ![Vue Jeu de réécriture](media/rewrite-http-headers-portal/rewrite-set-list.png)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la configuration de certains cas d’usage courants, consultez l’article présentant des [scénarios de réécriture d’en-têtes courants](./rewrite-http-headers.md).