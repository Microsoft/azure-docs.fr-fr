---
title: 'Tutoriel : Autoriser l’accès avec Cloudyn dans Azure'
description: Dans ce tutoriel, vous découvrez comment autoriser l’accès aux données Cloudyn avec des comptes d’utilisateur qui définissent les niveaux d’accès aux entités.
author: bandersmsft
ms.author: banders
ms.date: 03/12/2020
ms.topic: tutorial
ms.service: cost-management-billing
ms.subservice: cloudyn
ms.custom: seodec18
ms.reviewer: benshy
ROBOTS: NOINDEX
ms.openlocfilehash: a3c10d95a57ea81f269af8f8f1cad49fc1c1b845
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92131256"
---
# <a name="tutorial-assign-access-to-cloudyn-data"></a>Tutoriel : Autoriser l'accès aux données Cloudyn

L’accès aux données Cloudyn est fourni par la gestion des utilisateurs ou des entités. Les comptes d’utilisateur Cloudyn déterminent l’accès aux *entités* et aux fonctions d’administration. Il existe deux types d’accès : administrateur et utilisateur. Sauf modifié par utilisateur, l’accès administrateur concède à l’utilisateur un accès non restreint à toutes les fonctions du portail Cloudyn, notamment la gestion des utilisateurs, la gestion des listes de destinataires et l’accès aux entités racines pour toutes les données d’entité. L’accès utilisateur permet aux utilisateurs finaux d’afficher et de créer des rapports à travers l’accès dont ils disposent aux données d’entité.

Les entités sont utilisées pour refléter la structure hiérarchique de votre organisation. Elles identifient les services, divisions et équipes de votre organisation dans Cloudyn. La hiérarchie des entités permet de suivre avec précision les dépenses réalisées par les entités.

Quand vous avez inscrit votre contrat ou compte Azure, un compte avec l’autorisation d’administrateur a été créé dans Cloudyn pour vous permettre de réaliser toutes les étapes de ce didacticiel. Ce tutoriel décrit l’accès aux données Cloudyn, notamment la gestion des utilisateurs et la gestion des entités. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Créer un utilisateur avec un accès administrateur
> * Créer un utilisateur avec un accès utilisateur
> * Supprimer un utilisateur
> * Supprimer ou exporter des données personnelles
> * Créer et gérer des entités


Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [cloudyn-note](../../../includes/cloudyn-note.md)]

## <a name="prerequisites"></a>Prérequis

- Vous devez disposer d’un compte Azure.
- Vous devez disposer d’une inscription à un essai gratuit ou d’un abonnement payant pour Cloudyn.

## <a name="create-a-user-with-admin-access"></a>Créer un utilisateur avec un accès administrateur

Même si vous disposez déjà d’un accès administrateur, vos collègues de travail au sein de votre organisation devaient peut-être également disposer d’un accès administrateur. Dans le portail Cloudyn, cliquez sur le symbole d’engrenage dans le coin supérieur droit et sélectionnez **User Management (Gestion des utilisateurs)** . Cliquez sur **Add New User (Ajouter un nouvel utilisateur)** pour ajouter un utilisateur.

Entrez les informations requises sur l’utilisateur. L’**ID de connexion** doit être une adresse de messagerie valide. Sélectionnez des autorisations de gestion des utilisateurs pour permettre à l’utilisateur de créer et modifier d’autres utilisateurs. L’option de gestion des listes de destinataires permet à l’utilisateur de modifier les listes de destinataires. Cloudyn envoie un lien avec les informations de connexion par e-mail à l’utilisateur quand vous sélectionnez **Notify user by email (Notifier l'utilisateur par e-mail)** . Lors de la première connexion, l’utilisateur définit un mot de passe.

Sous **User has admin access (L’utilisateur a un accès administrateur)** , l’entité racine de votre organisation est sélectionnée. Laissez-la sélectionnée et enregistrez les informations utilisateur. La sélection de l’entité racine permet à l’utilisateur de disposer de l’autorisation d’administrateur non seulement pour l’entité racine dans l’arborescence, mais également pour toutes les entités qui se trouvent sous cette entité.  
  ![Exemple illustrant l'accès administrateur dans la zone Ajouter un nouvel utilisateur](./media/tutorial-user-access/new-admin-access.png)

## <a name="create-a-user-with-user-access"></a>Créer un utilisateur avec un accès utilisateur
En principe, les utilisateurs ayant besoin d’accéder à des données Cloudyn, comme des tableaux de bord et des rapports, doivent disposer d’un accès utilisateur pour pouvoir les visualiser. Créez un nouvel utilisateur avec un accès utilisateur similaire à celui que vous avez créé avec l’accès administrateur, en tenant compte des différences suivantes :

- Décochez les cases **Allow User Management (Autoriser la gestion des utilisateurs)** et **Allow Recipient lists Management (Autoriser la gestion des listes de destinataires)** , et décochez tout dans la liste **User has admin access (L’utilisateur a un accès administrateur)** .
- Sélectionnez les entités auxquelles l’utilisateur a besoin d’accéder dans la liste **User has admin access (L’utilisateur a un accès administrateur)** .
- Vous pouvez également autoriser l’administrateur à accéder à des entités spécifiques, selon les besoins.

![Exemple illustrant l'accès utilisateur dans la zone Ajouter un nouvel utilisateur](./media/tutorial-user-access/new-user-access.png)

Pour visionner un tutoriel vidéo sur l’ajout d’utilisateurs, consultez [Ajout d’utilisateurs à Cloudyn](https://youtu.be/Nzn7GLahx30).

## <a name="delete-a-user"></a>Supprimer un utilisateur

Lorsque vous supprimez un utilisateur, toutes les entités auxquelles l’utilisateur a accès restent intactes. Les rapports *personnels* enregistrés sont supprimés lorsque l’utilisateur est supprimé. Les rapports *publics* enregistrés créés par l’utilisateur ne sont pas supprimés.

Vous ne pouvez pas vous supprimer en tant qu’utilisateur.

> [!WARNING]
> Quand vous supprimez un utilisateur, il ne peut pas être restauré.

1.    Dans le portail Cloudyn, cliquez sur le symbole d’engrenage dans le coin supérieur droit et sélectionnez **Gestion des utilisateurs**.
2.    Dans la liste des utilisateurs, sélectionnez l’utilisateur à supprimer, puis cliquez sur **Supprimer l’utilisateur** (symbole de poubelle).
3.    Dans la zone Supprimer l’utilisateur, cliquez sur **Oui**, puis sur **OK**.


## <a name="delete-or-export-personal-data"></a>Supprimer ou exporter des données personnelles

Si vous souhaitez supprimer ou exporter des données personnelles à partir de Cloudyn, vous devez créer un ticket de support. Lorsque le ticket de support est créé, il agit comme requête officielle (une requête d’objet de données). Microsoft agit ensuite rapidement pour supprimer le compte, ainsi que les données personnelles ou client. Pour en savoir plus sur la demande de suppression ou d’exportation de vos données, consultez [Data Subject Requests of Cloudyn Data](https://www.cloudyn.com/cloudyn-gdpr-requests) (Requêtes d’objet de données pour des données Cloudyn).

## <a name="create-and-manage-entities"></a>Créer et gérer des entités

Quand vous définissez la hiérarchie des entités de coût, une meilleure pratique consiste à identifier la structure de votre organisation. Des entités vous permettent de segmenter des dépenses par comptes ou abonnements individuels. Vous créez des entités de coût pour créer des groupes logiques afin de gérer et suivre les dépenses. Au fur et à mesure que vous créez l’arborescence, réfléchissez à la façon dont vous souhaitez ou devez afficher leurs coûts ventilés par unités commerciales, centres de coûts, environnements et services des ventes. L’arborescence des entités dans Cloudyn est flexible en raison de l’héritage des entités.

Les abonnements individuels à vos comptes cloud sont liés à des entités spécifiques. Vous pouvez associer une entité à un abonnement ou un compte de fournisseur de services cloud. Les entités sont par conséquent mutualisées. Vous pouvez autoriser des utilisateurs spécifiques à accéder uniquement à leur branche de l’entreprise à l’aide d’entités. Cela permet d’isoler les données, même à travers de grandes divisions d’une entreprise, comme des filiales. L’isolement des données facilite en outre la gouvernance.  

Quand vous avez inscrit votre contrat ou compte Azure sur Cloudyn, vos données de ressources Azure, notamment les données d’utilisation, de performances, de facturation et de balise de vos abonnements ont été copiées dans votre compte Cloudyn. Vous devez toutefois créer manuellement l’arborescence des entités. Si vous avez ignoré l’inscription à Azure Resource Manager, seules les données de facturation et quelques rapports de ressources sont disponibles dans le portail Cloudyn.

Dans le portail Cloudyn, cliquez sur le symbole d’engrenage dans le coin supérieur droit et sélectionnez **Cloud Accounts (Comptes cloud)** . Vous commencez par une seule entité (racine), puis vous créez l’arborescence des entités sous la racine. Voici un exemple de hiérarchie d’entités similaire à celle de nombreuses sociétés informatiques, une fois l’arborescence créée :

![Exemple d'arborescence d'entités sur la page Gestion des comptes](./media/tutorial-user-access/entity-tree.png)

En regard de **Entities (Entités)** , cliquez sur **Add Entity (Ajouter une entité)** . Entrez les informations relatives à la personne ou le service que vous souhaitez ajouter. Les champs **Full Name (Nom complet)** et **Email (E-mail)** ne doivent pas correspondre à des utilisateurs existants. Si vous souhaitez afficher la liste des niveaux d’accès, recherchez dans l’aide *Adding an entity (Ajout d’une entité)* .

![Exemple montrant le nom de l'entité et les niveaux d'accès dans la zone Ajouter une entité](./media/tutorial-user-access/add-entity.png)

Quand vous avez terminé, **enregistrez** l’entité.

### <a name="entity-access-levels"></a>Niveaux d’accès d’entité

Les niveaux d’accès d’entité associés à un accès utilisateur vous permettent de définir quel type d’actions sont disponibles dans le portail Cloudyn.

- **Entreprise** : offre la possibilité de créer et de gérer des entités de coût enfants.
- **Enterprise + Cost Allocation** (Entreprise + Répartition des coûts) : offre la possibilité de créer et gérer des entités de coûts enfants, y compris la répartition des coûts pour les comptes consolidés.
- **Enterprise, Cost based on parent cost allocation** (Entreprise, coûts basés sur la répartition des coûts parents) : offre la possibilité de créer et de gérer des entités de coûts enfants. Les coûts du compte sont basés sur le modèle de répartition des coûts parents.
- **Custom Dashboards Only** (Tableaux de bord personnalisés uniquement) : permet à l’utilisateur de voir uniquement des tableaux de bord personnalisés prédéfinis.
- **Dashboards Only** (Tableaux de bord uniquement) : permet à l’utilisateur de voir uniquement des tableaux de bord.

### <a name="create-a-cost-entity-hierarchy"></a>Créer une hiérarchie d’entité de coût

Pour créer une hiérarchie d’entité de coût, vous devez disposer d’un compte avec l’accès Entreprise ou Enterprise + Cost Allocation (Entreprise + Répartition des coûts).

Dans le portail Cloudyn, cliquez sur le symbole d’engrenage dans le coin supérieur droit et sélectionnez **Cloud Accounts** (Comptes cloud). L’arborescence **Entités** est affichée dans le volet gauche. Si nécessaire, développez l’arborescence de l’entité afin que vous puissiez afficher l’entité que vous souhaitez associer à un compte.  Vos comptes de fournisseur de services de cloud sont affichés sur les onglets dans le volet droit. Sélectionnez un onglet et cliquez sur un compte/abonnement à l’entité, faites-le glisser et déposez-le. La zone **Déplacer** vous informe que le compte a été déplacé. Cliquez sur **OK**.

Vous pouvez également associer plusieurs comptes à une entité. Sélectionnez les comptes, puis cliquez sur **Déplacer**. Dans la zone Move Accounts (Déplacer les comptes), sélectionnez l’entité dans laquelle vous souhaitez déplacer le compte, puis cliquez sur **Enregistrer**. La zone Move Accounts (Déplacer les comptes) vous invite à vérifier que vous souhaitez déplacer les comptes. Cliquez sur **Oui**, puis sur **OK**.

Pour visionner un tutoriel vidéo sur la création d’une hiérarchie d’entités de coût, consultez [Création d’une hiérarchie d’entités de coût dans Cloudyn](https://youtu.be/dAd9G7u0FmU).

Si vous êtes un utilisateur Contrat Entreprise Azure, regardez ce tutoriel vidéo sur l’association des comptes et des abonnements à des entités : [Connexion à Azure Resource Manager avec Cloudyn](https://youtu.be/oCIwvfBB6kk).

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Créer un utilisateur avec un accès administrateur
> * Créer un utilisateur avec un accès utilisateur
> * Supprimer un utilisateur
> * Supprimer ou exporter des données personnelles
> * Créer et gérer des entités


Si vous n’avez pas déjà activé l’accès de l’API Azure Resource Manager pour vos comptes, consultez l’article suivant.

> [!div class="nextstepaction"]
> [Activer des abonnements et comptes Azure](./activate-subs-accounts.md)