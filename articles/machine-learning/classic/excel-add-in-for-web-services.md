---
title: 'ML Studio (classique) : Complément Excel pour les services web – Azure'
description: Comment utiliser les services web Azure Machine Learning directement dans Excel sans écrire de code.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18
ms.date: 02/01/2018
ms.openlocfilehash: fb87afcc6edc02d241dca763aff9ecdd594e4711
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93310078"
---
# <a name="excel-add-in-for-azure-machine-learning-studio-classic-web-services"></a>Complément Excel de services web Azure Machine Learning Studio (classique)

**S’APPLIQUE À :**  ![S’applique à ](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![Ne s’applique pas à ](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)


Excel permet d'appeler facilement des services web directement sans qu'il soit nécessaire d'écrire du code.

## <a name="steps-to-use-an-existing-web-service-in-the-workbook"></a>Procédure d’utilisation d’un service web existant dans le classeur

1. Ouvrez l’ [exemple de fichier Excel](https://aka.ms/amlexcel-sample-2), qui contient le complément Excel et les données concernant les passagers sur le Titanic. 
 
    > [!NOTE]
    > - Vous pouvez voir la liste des services web associés au fichier. En bas, la case de prédiction automatique s’affiche. Si vous activez la prédiction automatique, les prédictions de **tous** vos services seront mises à jour chaque fois que les entrées sont modifiées. Si la case n’est pas cochée, vous devrez cliquer sur le bouton « Prédire tout » pour actualiser. Pour activer la prédiction automatique au niveau du service, passez à l’étape 6.
    > - Le complément Excel d’Azure Machine Learning appellera le magasin de compléments Office pour charger. Si votre organisation interdit l’accès au magasin de compléments Office, une erreur s’affiche lors du chargement du complément. Dans ce cas, déployez le complément Excel d’Azure Machine Learning à partir du Centre d’administration Microsoft 365. Appelez ensuite le complément et ajoutez le service web manuellement en collant l’URL et la clé API.

 

2. Choisissez le service web en cliquant dessus : « Titanic Survivor Predictor (exemple de complément Excel) [Score] » dans cet exemple.
   
    ![Sélectionner un service web](./media/excel-add-in-for-web-services/image1.png)
3. Vous accédez alors à la section **Prédire**.  Ce classeur contient déjà des exemples de données mais, pour un classeur vide, vous pouvez également sélectionner une cellule dans Excel et cliquer sur **Utiliser les exemples de données**.
4. Sélectionnez les données avec les en-têtes et cliquez sur l'icône de plage de données d'entrée.  Assurez-vous que la case « Mes données ont des en-têtes » est activée.
5. Sous **Sortie** , entrez le numéro de la cellule dans laquelle vous souhaitez insérer le résultat, en l’occurrence, « H1 ».
6. Cliquez sur **Prédire**. Si vous cochez la case de prédiction automatique, toutes les modifications apportées aux zones sélectionnées (celles spécifiées en tant qu’entrée) déclenchent une requête et une mise à jour des cellules de sortie, sans que vous n’ayez à appuyer sur le bouton de prédiction.
   
    ![Section Prédire](./media/excel-add-in-for-web-services/image1.png)

Déployez un service web ou utilisez un service web existant. Pour plus d’informations sur le déploiement d’un service web, consultez [Tutoriel 3 : Déployer un modèle de risque de crédit](tutorial-part3-credit-risk-deploy.md).

Obtenez la clé API de votre service web. L’emplacement à partir duquel exécutez cette action diffère selon que vous avez publié un service web Machine Learning classique ou un nouveau service web Machine Learning.

**Utiliser un service web classique** 

1. Dans Machine Learning Studio (classique), cliquez sur la section **WEB SERVICES** située sur le volet de gauche, puis sélectionnez le service web.
   
    ![Studio - Sélectionner un service web](./media/excel-add-in-for-web-services/image4.png)
2. Copiez la clé API du service web.
   
    ![Studio clé API](./media/excel-add-in-for-web-services/image5.png)
3. Sous l’onglet **TABLEAU DE BORD** pour le service web, cliquez sur le lien **REQUÊTE/RÉPONSE**.
4. Recherchez la section **URI de requête** .  Copiez et enregistrez l’URL.

> [!NOTE]
> Il est désormais possible de se connecter au portail [Azure Machine Learning Web Services](https://services.azureml.net) pour obtenir la clé API d’un service web Machine Learning classique.
> 
> 

**Utiliser un nouveau service web**

1. Dans le portail [Services web Azure Machine Learning](https://services.azureml.net), cliquez sur **Services web** , puis sélectionnez votre service web. 
2. Cliquez sur **Consommer**.
3. Recherchez la section **des informations de base sur la consommation** . Copiez et enregistrez la **Clé primaire** et l’URL de **Demande-réponse**.

## <a name="steps-to-add-a-new-web-service"></a>Procédure d’ajout d’un nouveau service web

1. Déployez un service web ou utilisez un service web existant. Pour plus d’informations sur le déploiement d’un service web, consultez [Tutoriel 3 : Déployer un modèle de risque de crédit](tutorial-part3-credit-risk-deploy.md).
2. Cliquez sur **Consommer**.
3. Recherchez la section **des informations de base sur la consommation** . Copiez et enregistrez la **Clé primaire** et l’URL de **Demande-réponse**.
4. Dans Excel, accédez à la section **Services web** (si vous vous trouvez dans la section **Prédire** , cliquez sur la flèche de retour pour accéder à la liste des services web).
   
    ![Accéder à la sélection du service web](./media/excel-add-in-for-web-services/image3.png)
5. Cliquez sur **Ajouter un service web**.
6. Collez l’URL dans la zone de texte du complément Excel intitulée **URL**.
7. Collez l’API/Clé primaire dans la zone de texte intitulée **Clé API**.
8. Cliquez sur **Add**.
   
    ![URL et clé API pour un service web classique.](./media/excel-add-in-for-web-services/image6.png)
9. Pour utiliser le service web, suivez les instructions de la section « Procédure d’utilisation d’un service web existant » ci-dessus.

## <a name="sharing-your-workbook"></a>Partage de votre classeur
Si vous enregistrez votre classeur, l'API/la clé primaire pour les services web que vous avez ajoutés seront également enregistrés. Cela signifie que vous devez uniquement partager le classeur avec des personnes de confiance.

Posez les questions que vous voulez dans la section de commentaire suivante ou dans notre [forum](/answers/topics/azure-machine-learning.html).