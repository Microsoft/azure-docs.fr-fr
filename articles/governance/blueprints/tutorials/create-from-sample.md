---
title: 'Tutoriel : Exemple de blueprint dans un nouvel environnement'
description: Dans ce tutoriel, vous utilisez un exemple de blueprint pour créer une définition de blueprint qui définit jusqu’à deux groupes de ressources et configure une attribution de rôle pour chacun.
ms.date: 08/27/2020
ms.topic: tutorial
ms.openlocfilehash: 891240dcafe0cbfbfab4ff1fd415cba4abf682a5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89048730"
---
# <a name="tutorial-create-an-environment-from-a-blueprint-sample"></a>Tutoriel : Créer un nouvel environnement à partir d’un exemple de blueprint

Les exemples de blueprint illustrent ce qui est réalisable à l’aide d’Azure Blueprints. Chaque exemple a été conçu avec une intention ou un but spécifique, mais ne permet pas de créer un environnement complet à lui seul. Chacun d’eux est conçu comme base d’exploration d’Azure Blueprints avec diverses combinaisons d’artefacts, de conceptions et de paramètres.

Le tutoriel suivant utilise l’exemple de blueprint **Groupes de ressources avec RBAC** pour présenter différents aspects du service Azure Blueprints. Cet article contient les étapes suivantes :

> [!div class="checklist"]
> - Créer une définition de blueprint à partir de l’exemple
> - Marquer la copie de l’exemple en tant que **Publié**
> - Affecter votre copie du blueprint à un abonnement existant
> - Examiner les ressources déployées pour l’affectation
> - Annuler l’affectation du blueprint afin de retirer les verrous

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, il est nécessaire de disposer d’un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="create-blueprint-definition-from-sample"></a>Créer une définition de blueprint à partir de l’exemple

Tout d’abord, implémentez l’exemple de blueprint. L’importation crée un nouveau blueprint dans votre environnement à partir de l’exemple.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Dans la page **Démarrage**à gauche, sélectionnez le bouton **Créer** sous _Créer un blueprint_.

1. Recherchez l’exemple de blueprint **Groupes de ressources avec RBAC** sous _Autres exemples_, puis sélectionnez-le.

1. Entrez les _Fonctions de base_ de l’exemple de blueprint :

   - **Nom du blueprint** : Entrez le nom de votre copie de l’exemple de blueprint. Pour ce didacticiel, nous utiliserons le nom _two-rgs-with-role-assignments_.
   - **Emplacement de la définition** : Utilisez le bouton points de suspension et sélectionnez le groupe d’abonnement ou l’abonnement dans lequel enregistrer votre copie de l’exemple.

1. Sélectionnez l’onglet _Artefacts_ dans le haut de la page ou **Suivant : Artefacts** dans le bas de la page.

1. Passez en revue la liste des artefacts qui composent l’exemple de blueprint. Cet exemple définit deux groupes de ressources, dont les noms d’affichage sont _ProdRG_ et _PreProdRG_. Le nom final et l’emplacement de chaque groupe de ressources sont définis lors de l’assignation du blueprint. Le groupe de ressources _ProdRG_ se voit attribuer le rôle _Contributeur_ et le groupe de ressources _PreProdRG_ se voit attribuer les rôles  _Propriétaire_ et _Lecteurs_. Les rôles affectés dans la définition sont statiques, mais l’utilisateur, l’application ou le groupe auquel le rôle est affecté est défini pendant l’affectation du blueprint.

1. Sélectionnez **Enregistrer comme brouillon** lorsque vous avez terminé de passer en revue l’exemple de blueprint.

Cette étape crée une copie de l’exemple de définition de blueprint dans le groupe d’administration ou l’abonnement sélectionné. La définition de blueprint enregistrée est gérée comme n’importe quel blueprint créé à partir de zéro. Vous pouvez enregistrer l’exemple dans votre groupe d’administration ou votre abonnement autant de fois que nécessaire. Toutefois, chaque copie doit porter un nom unique.

Dès que la notification **Définition de blueprint enregistrée** apparaît dans le portail, passez à l’étape suivante.

## <a name="publish-the-sample-copy"></a>Publier la copie de l’exemple

Votre copie de l’exemple de blueprint est à présent créée dans votre environnement. Elle est créée en mode **Brouillon** et doit être **publiée** avant de pouvoir être attribuée et déployée. La copie de l’exemple de blueprint peut être personnalisée en fonction de votre environnement et de vos besoins. Pour ce didacticiel, nous n’apporterons aucune modification.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Sélectionnez la page **Définitions de blueprint** à gauche. Utilisez les filtres pour trouver la définition de blueprint _two-rgs-with-role-assignments_, puis sélectionnez celle-ci.

1. Sélectionnez **Publier le blueprint** dans le haut de la page. Dans le nouveau volet à droite, définissez la **Version** de votre copie d’exemple de blueprint sur _1.0_. Cette propriété est utile si vous prévoyez d’effectuer une modification ultérieurement. Complétez **Change notes** (Remarques sur les modifications) avec un libellé, tel que « Première version publiée à partir de l’exemple de blueprint des groupes de ressources avec RBAC. » Sélectionnez ensuite **Publier** en bas de la page.

Cette étape permet d’affecter le plan à un abonnement. Après publication, des modifications sont toujours possibles. Les modifications supplémentaires nécessitent la publication avec une nouvelle valeur de **Version** afin de suivre les différences entre les versions différentes d’une même définition de blueprint.

Dès que la notification **Définition de blueprint publiée** apparaît dans le portail, passez à l’étape suivante.

## <a name="assign-the-sample-copy"></a>Affecter la copie de l’exemple

Une fois que la copie de l’exemple de blueprint a été **publiée**, elle peut être affectée à un abonnement dans le groupe d’administration où elle a été enregistrée. C’est à cette étape que les paramètres sont fournis pour que chaque déploiement de la copie de l’exemple de blueprint soit unique.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Sélectionnez la page **Définitions de blueprint** à gauche. Utilisez les filtres pour trouver la définition de blueprint _two-rgs-with-role-assignments_, puis sélectionnez celle-ci.

1. Sélectionnez **Affecter le blueprint** dans le haut de la page.

1. Indiquez les valeurs des paramètres pour l’affectation du blueprint :

   - Concepts de base

     - **Abonnements** : Sélectionnez un ou plusieurs des abonnements qui font partie du groupe d’administration où vous avez enregistré votre copie de l’exemple de blueprint. Si vous sélectionnez plusieurs abonnements, une affectation est créée pour chacun d’eux à l’aide des paramètres saisis.
     - **Nom de l’affectation** : Le nom est prérempli automatiquement en fonction du nom de la définition du blueprint.
     - **Emplacement** : Sélectionnez une région dans laquelle créer l’identité managée. Azure Blueprint utilise cette identité managée pour déployer tous les artefacts dans le blueprint affecté. Pour en savoir plus, consultez [Identités managées pour les ressources Azure](../../../active-directory/managed-identities-azure-resources/overview.md).
       Pour ce didacticiel, sélectionnez _USA Est 2_.
     - **Version de définition du blueprint** : Choisissez la version **Published**_1.0_ de votre copie de l’exemple de blueprint.

   - Verrouiller l'affectation

     Sélectionnez le mode de verrouillage _Lecture seule_. Pour plus d’informations, consultez [Verrouillage des ressources des blueprints](../concepts/resource-locking.md).

   - Identité managée

     Conservez l’option par défaut, _Affecté(e) par le système_. Pour plus d’informations, voir [Identités managées](../../../active-directory/managed-identities-azure-resources/overview.md).

   - Paramètres d'artefact

     Les paramètres définis dans cette section s’appliquent à l’artefact sous lequel elle est définie. Ces paramètres sont des [paramètres dynamiques](../concepts/parameters.md#dynamic-parameters) puisqu’ils sont définis lors de l’affectation du blueprint. Pour chaque artefact, définissez la valeur du paramètre en vous référant à la colonne **Valeur**. Pour `{Your ID}`, sélectionnez votre compte d’utilisateur Azure.

     |Nom de l’artefact|Type d’artefact|Nom du paramètre|Valeur|Description|
     |-|-|-|-|-|
     |Groupe de ressources ProdRG|Resource group|Nom|ProductionRG|Définit le nom du premier groupe de ressources.|
     |Groupe de ressources ProdRG|Resource group|Emplacement|USA Ouest 2|Définit l’emplacement du premier groupe de ressources.|
     |Contributeur|Attribution de rôle|Utilisateur ou groupe|{Votre ID}|Définit l’utilisateur ou le groupe auquel attribuer le rôle _Contributeur_ dans le premier groupe de ressources.|
     |Groupe de ressources PreProdRG|Resource group|Nom|PreProductionRG|Définit le nom du deuxième groupe de ressources.|
     |Groupe de ressources PreProdRG|Resource group|Emplacement|USA Ouest|Définit l’emplacement du deuxième groupe de ressources.|
     |Propriétaire|Attribution de rôle|Utilisateur ou groupe|{Votre ID}|Définit l’utilisateur ou le groupe auquel attribuer le rôle _Propriétaire_ dans le deuxième groupe de ressources.|
     |Lecteurs|Attribution de rôle|Utilisateur ou groupe|{Votre ID}|Définit l’utilisateur ou le groupe auquel attribuer le rôle _Lecteurs_ dans le deuxième groupe de ressources.|

1. Une fois tous les paramètres entrés, sélectionnez **Affecter** au bas de la page.

Cette étape déploie les ressources définies et configure l’option **Verrouiller l’affectation** sélectionnée. L’application des verrous de blueprint peut prendre jusqu’à 30 minutes.

Dès que la notification **Définition de blueprint affectée** apparaît dans le portail, passez à l’étape suivante.

## <a name="inspect-resources-deployed-by-the-assignment"></a>Examiner les ressources déployées par l’affectation

L’affectation du blueprint crée et suit les artefacts définis dans la définition du blueprint. L’état des ressources est visible dans la page de l’affectation du blueprint ou examinant celles-ci directement.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Sélectionnez la page **Blueprints affectés** à gauche. Utilisez les filtres pour trouver l’affectation du blueprint _Assignment-two-rgs-with-role-assignments_, puis sélectionnez celle-ci.

   Cette page répertorie l’affectation réussie, les ressources créées ainsi que l’état de verrouillage de blueprint qui leur est associé. Si l’affectation est mise à jour, le menu déroulant **Opération d’affectation** affiche des détails sur le déploiement de chaque version de définition. Chaque ressource répertoriée qui a été créée peut être sélectionnée, ce qui ouvre la page de propriétés associée.

1. Sélectionnez le groupe de ressources **ProductionRG**.

   Le nom du groupe de ressources est **ProductionRG** et non le nom de l’artefact, _ProdRG_. Ce nom correspond à la valeur définie lors de l’attribution de blueprint.

1. Sélectionnez la page **Contrôle d’accès (IAM)** à gauche, puis l’onglet **Attributions de rôles**.

   Ici, nous voyons que votre compte a reçu le rôle _Contributeur_ sur la portée de _Cette ressource_. L’affectation du blueprint _Assignment-two-rgs-with-role-assignments_ possède le rôle _Propriétaire_, qui a été utilisé pour créer le groupe de ressources. Ces autorisations permettent également de gérer les ressources avec des verrous configurés.

1. Dans la barre de navigation du portail Azure, sélectionnez **Assignment-two-rgs-with-role-assignments** pour retourner à une page précédente, puis sélectionnez le groupe de ressources **PreProductionRG**.

1. Sélectionnez la page **Contrôle d’accès (IAM)** à gauche, puis l’onglet **Attributions de rôles**.

   Ici, nous voyons que votre compte a reçu les rôles _Contributeur_ et _Lecteur_ sur la portée de _Cette ressource_. Comme le premier groupe de ressources, l’affectation du blueprint possède aussi le rôle _Propriétaire_.

1. Sélectionnez l’onglet **Affectations de refus**.

   L’affectation du blueprint a créé une instance [Refuser l’affectation](../../../role-based-access-control/deny-assignments.md) sur le groupe de ressources déployé pour appliquer le mode _Lecture seule_ de verrouillage du blueprint. Cette affectation de refus empêche un utilisateur disposant de droits appropriés dans l’onglet _Attributions de rôle_ d’effectuer des actions particulières. L’affectation de refus concerne _Tous les principaux_.

1. Sélectionnez l’affectation de refus, puis la page **Autorisations refusées** à gauche.

   L’affectation de refus empêche toutes les opérations avec la configuration **\*** et **Action**, mais autorise l’accès en lecture en excluant **\*/read** via **NotActions**.

1. Dans la barre de navigation du portail Azure, sélectionnez **PreProductionRG - Contrôle d’accès (IAM)** . Sélectionnez ensuite la page **Vue d’ensemble** à gauche, puis le bouton **Supprimer le groupe de ressources**. Entrez le nom _PreProductionRG_ pour confirmer la suppression et sélectionnez **Supprimer** dans le bas du volet.

   Le message de notification du portail **Échec de la suppression du groupe de ressources PreProductionRG**  apparaît alors. L’erreur indique que, même si votre compte est autorisé à supprimer le groupe de ressources, l’accès est refusé par l’affectation du blueprint. N’oubliez pas que nous avons sélectionné le mode _Lecture seule_ de verrouillage du blueprint pendant l’affectation du blueprint. Le verrou de blueprint empêche un compte disposant d’autorisations, y compris _Propriétaire_, de supprimer la ressource. Pour plus d’informations, consultez [Verrouillage des ressources des blueprints](../concepts/resource-locking.md).

Ces étapes montrent que nos ressources ont été créées conformément à la définition et que les verrous du blueprint ont empêché la suppression non souhaitée, même à partir d’un compte disposant d’autorisations.

## <a name="unassign-the-blueprint"></a>Annuler l’affectation du blueprint

La dernière étape consiste à supprimer l’affectation du blueprint et des ressources qu'il a déployées.
Le fait de supprimer l’affectation ne supprime pas les artefacts déployés.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Sélectionnez la page **Blueprints affectés** à gauche. Utilisez les filtres pour trouver l’affectation du blueprint _Assignment-two-rgs-with-role-assignments_, puis sélectionnez celle-ci.

1. Sélectionnez le bouton **Annuler l’affectation du blueprint** en haut de la page. Lisez l’avertissement dans la boîte de dialogue de confirmation, puis sélectionnez **OK**.

   La suppression du blueprint entraîne la suppression des verrous de celui-ci. Les ressources créées peuvent à nouveau être supprimées par un compte disposant d’autorisations.

1. Sélectionnez **Groupes de ressources** dans le menu Azure, puis sélectionnez **ProductionRG**.

1. Sélectionnez la page **Contrôle d’accès (IAM)** à gauche, puis l’onglet **Attributions de rôles**.

Du point de vue de la sécurité, les affectations de rôles restent déployées pour chaque groupe de ressources, mais l’affectation du blueprint n’a plus d’accès _Propriétaire_.

Dès que la notification **Affectation de blueprint supprimée** apparaît dans le portail, passez à l’étape suivante.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Une fois le didacticiel terminé, supprimez les ressources suivantes :

- Groupe de ressources _ProductionRG_
- Groupe de ressources _PreProductionRG_
- Définition de blueprint _two-rgs-with-role-assignments_

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à créer un blueprint à partir d’un exemple de définition. Pour plus d’informations sur Azure Blueprints, consultez l’article concernant le cycle de vie des blueprints.

> [!div class="nextstepaction"]
> [En savoir plus sur le cycle de vie des blueprints](../concepts/lifecycle.md)