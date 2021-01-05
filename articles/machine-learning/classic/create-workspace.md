---
title: 'ML Studio (classique) : Création d’un espace de travail – Azure'
description: Pour utiliser Azure Machine Learning Studio (classique), vous devez disposer d’un espace de travail Machine Learning Studio (classique). Cet espace de travail contient les outils dont vous avez besoin pour créer, gérer et publier des expériences.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18
ms.date: 12/07/2017
ms.openlocfilehash: 0e2a67141970d1f5091457066db9b18a00ffecf8
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95486158"
---
# <a name="create-and-share-an-machine-learning-studio-classic-workspace"></a>Création et partage d’un espace de travail Azure Machine Learning Studio (classique)

**S’APPLIQUE À :**  ![Cette coche signifie que cet article s’applique à Machine Learning Studio (classique).](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![Cette croix (X) signifie que cet article s’applique à Azure Machine Learning.](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)

Pour utiliser Azure Machine Learning Studio (classique), vous devez disposer d’un espace de travail Machine Learning Studio (classique). Cet espace de travail contient les outils dont vous avez besoin pour créer, gérer et publier des expériences.

## <a name="create-a-studio-classic-workspace"></a>Créer un espace de travail Studio (classique)

Pour ouvrir un espace de travail de Machine Learning Studio (classique), vous devez être connecté au compte Microsoft utilisé pour créer l’espace de travail ou recevoir une invitation du propriétaire à rejoindre l’espace de travail. À partir du portail Azure, vous pouvez gérer l’espace de travail, notamment configurer l’accès.

1. Connectez-vous au [portail Azure](https://portal.azure.com/)

    > [!NOTE]
    > Pour vous connecter et créer un espace de travail Studio (classique), vous devez être administrateur d’abonnements Azure. 
    >
    > 

2. Cliquez sur **+Nouveau**.

3. Dans la zone de recherche, saisissez **espace de travail Machine Learning Studio (classique)** et sélectionnez l’élément correspondant. Ensuite, cliquez sur **Créer** en bas de la page.

4. Entrez les informations sur votre espace de travail :

   - Le *Nom de l’espace de travail* peut contenir jusqu’à 260 caractères. Il ne doit pas se terminer par un espace. Le nom ne peut pas contenir ces caractères : `< > * % & : \ ? + /`
   - Le *plan de service web* que vous choisissez (ou créez), ainsi que le *niveau tarifaire* associé que vous sélectionnez, est utilisé si vous déployez des services web à partir de cet espace de travail.

     ![Créer un espace de travail Studio (classique)](./media/create-workspace/create-new-workspace.png)

5. Cliquez sur **Créer**.

   Machine Learning est actuellement disponible dans un nombre limité de régions. Si votre abonnement n’inclut pas l’une de ces régions, le message d’erreur suivant risque de s’afficher : « Vous n’avez aucun abonnement dans les régions autorisées. »  Pour demander qu’une région soit ajoutée à votre abonnement, créez une nouvelle demande d’assistance depuis le portail Azure, choisissez **Facturation** comme type de problème, puis suivez les invites pour envoyer votre demande.


> [!NOTE]
> Machine Learning Studio (classique) s’appuie sur un compte de stockage Azure fourni par vous pour enregistrer les données intermédiaires au moment de l’exécution du flux de travail. Une fois l'espace de travail créé, si le compte de stockage est supprimé ou si les clés d'accès sont modifiées, l'espace de travail cesse de fonctionner et toutes ses expériences échouent.
Si vous supprimez accidentellement le compte de stockage, recréez-le avec le même nom dans la même région que celui que vous avez supprimé et resynchronisez la clé d'accès. Si vous avez modifié les clés d’accès du compte de stockage, resynchronisez-les dans l’espace de travail à l’aide du portail Azure.

Une fois l’espace de travail déployé, vous pouvez l’ouvrir dans Machine Learning Studio (classique).

1. Accédez à Machine Learning Studio (classique) à l’adresse [https://studio.azureml.net/](https://studio.azureml.net/).

2. Sélectionnez votre espace de travail dans le coin supérieur droit.

    ![Sélectionner un espace de travail](./media/create-workspace/open-workspace.png)

3. Cliquez sur **my experiments (mes expérimentations)** .

    ![Ouvrir des expérimentations](./media/create-workspace/my-experiments.png)

Pour plus d’informations sur la gestion de votre espace de travail Studio (classique), consultez [Gérer un espace de travail Azure Machine Learning Studio (classique)](manage-workspace.md).
Si vous rencontrez un problème lors de la création de votre espace de travail, consultez [Guide de résolution des problèmes : Créer un espace de travail Machine Learning Studio (classique) et s’y connecter](index.yml).


## <a name="share-an-azure-machine-learning-studio-classic-workspace"></a>Partager un espace de travail Azure Machine Learning Studio (classique)
Une fois l’espace de travail Machine Learning Studio (classique) créé, vous pouvez inviter des utilisateurs à partager l’accès à celui-ci et à l’ensemble de ses expériences, jeux de données, etc. Vous pouvez ajouter des utilisateurs dans l’un des deux rôles :

* **Utilisateur** : un utilisateur de l’espace de travail peut créer, ouvrir, modifier et supprimer des expérimentations, jeux de données, etc. dans l’espace de travail.
* **Propriétaire** : un propriétaire peut inviter et supprimer des utilisateurs dans l’espace de travail, ainsi que les actions qu’ils sont autorisés à effectuer.

> [!NOTE]
> Le compte d’administrateur qui crée l’espace de travail y est automatiquement ajouté en tant que propriétaire de l’espace de travail. Toutefois, les autres administrateurs ou utilisateurs de cet abonnement ne bénéficient pas automatiquement d’un accès à l’espace de travail : vous devez les inviter explicitement.
> 
> 

### <a name="to-share-a-studio-classic-workspace"></a>Pour partager un espace de travail Studio (classique)

1. Connectez-vous à Machine Learning Studio (classique) à l’adresse [https://studio.azureml.net/Home](https://studio.azureml.net/Home)

2. Dans le panneau gauche, cliquez sur **SETTINGS (PARAMÈTRES)** .

3. Cliquez sur l’onglet **USERS (UTILISATEURS)** .

4. En bas de la page, cliquez sur **INVITE MORE USERS (INVITER D’AUTRES UTILISATEURS)** .

    ![Paramètres Studio](./media/create-workspace/settings.png)

5. Entrez une ou plusieurs adresses e-mail. Les utilisateurs ont besoin d’un compte Microsoft valide ou d’un compte d’entreprise (issu d’Azure Active Directory).

6. Indiquez si vous voulez ajouter des utilisateurs en tant que propriétaire ou utilisateur.

7. Cliquez sur la coche **OK** .

Chaque utilisateur ajouté reçoit un e-mail contenant des instructions de connexion à l’espace de travail partagé.

> [!NOTE]
> Pour que les utilisateurs puissent déployer ou gérer des services web dans cet espace de travail, ils doivent être collaborateurs ou administrateurs dans l’abonnement Azure. 

## <a name="troubleshoot-storage-accounts"></a>Résoudre les problèmes des comptes de stockage


Le service Machine Learning a besoin d'un compte de stockage pour stocker les données. Vous pouvez utiliser un compte de stockage existant ou en créer un lors de la création de l’espace de travail Machine Learning Studio (classique) (si vous disposez d’un quota pour la création d’un compte de stockage).

Une fois l’espace de travail Machine Learning Studio (classique) créé, vous pouvez vous connecter à Machine Learning Studio (classique) au moyen du compte Microsoft utilisé pour créer l’espace de travail. Si le message d’erreur « Espace de travail introuvable » s’affiche (comme dans la capture d’écran suivante), effectuez les étapes suivantes pour supprimer les cookies de votre navigateur.

![Workspace not found](media/troubleshooting-creating-ml-workspace/screen3.png)

**Pour supprimer les cookies du navigateur**

1. Si vous utilisez Internet Explorer, cliquez sur le bouton **Outils** dans le coin supérieur droit et sélectionnez **Options Internet**.  

   ![Options Internet](media/troubleshooting-creating-ml-workspace/screen4.png)

2. Sous l'onglet **Général**, cliquez sur **Supprimer…**

   ![Onglet Général](media/troubleshooting-creating-ml-workspace/screen5.png)

3. Dans la boîte de dialogue **Supprimer l’historique de navigation**, vérifiez que **Cookies et données de sites web** est sélectionné, puis cliquez sur **Supprimer**.

   ![Delete cookies](media/troubleshooting-creating-ml-workspace/screen6.png)

Une fois les cookies supprimés, redémarrez le navigateur, puis accédez à la page [Microsoft Azure Machine Learning Studio (classique)](https://studio.azureml.net). Lorsque vous êtes invité à fournir un nom d’utilisateur et un mot de passe, entrez le même compte Microsoft que celui utilisé pour créer l’espace de travail.


## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la gestion d’un espace de travail, voir [Gérer un espace de travail Azure Machine Learning Studio (classique)](manage-workspace.md).