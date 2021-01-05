---
title: Mettre à niveau une entité composite - LUIS
description: Mettez à niveau une entité de machine-learning composite vers une entité de machine-learning avec le processus de mise à niveau dans le portail LUIS.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 05/04/2020
ms.openlocfilehash: 46e9ece70d9f980065c719ee1205eb46591b45c0
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95025240"
---
# <a name="upgrade-composite-entity-to-machine-learning-entity"></a>Mettre à niveau une entité composite vers une entité de machine-learning

Mettez à niveau une entité composite vers une entité de machine-learning pour générer une entité qui reçoit des prédictions plus complètes avec une meilleure décomposition pour déboguer l’entité.

## <a name="current-version-model-restrictions"></a>Restrictions du modèle de la version actuelle

Le processus de mise à niveau crée des entités de machine-learning, sur la base des entités composites existantes trouvées dans votre application, dans une nouvelle version de votre application. Cela inclut les rôles et les enfants des entités composites. Le processus change également les étiquettes dans les exemples d’énoncés de manière à utiliser la nouvelle entité.

## <a name="upgrade-process"></a>Mise à niveau

Le processus de mise à niveau :
* Crée une nouvelle entité de machine-learning pour chaque entité composite.
* Les entités enfants :
    * Si l’entité enfant n’est utilisée que dans l’entité composite, elle ne sera ajoutée qu’à l’entité de machine-learning.
    * Si l’entité enfant est utilisée dans l’entité composite _et_ en tant qu’entité distincte (étiquetée dans les exemples d’énoncés), elle sera ajoutée à la version en tant qu’entité et en tant que sous-entité de la nouvelle entité de machine-learning.
    * Si l’entité enfant utilise un rôle, chaque rôle est converti en sous-entité du même nom.
    * Si l’entité enfant est une entité non machine-learning (expression régulière, entité de liste ou entité prédéfinie), une nouvelle sous-entité est créée sous le même nom et celle-ci possède une fonctionnalité qui utilise l’entité non machine-learning avec la fonctionnalité requise ajoutée.
* Les noms sont conservés mais doivent être uniques au même niveau de sous-entité/frère. Reportez-vous à la section [Limites d’attribution de noms uniques](./luis-limits.md#name-uniqueness).
* Les étiquettes des exemples d’énoncés sont remplacées par la nouvelle entité de machine-learning avec des sous-entités.

Consultez le tableau suivant pour comprendre les modifications apportées à votre modèle :

|Ancien objet|Nouvel objet|Notes|
|--|--|--|
|Entité composite|entité de machine-learning avec structure|Les deux objets sont des objets parents.|
|L’entité enfant de l’entité composite est **entité simple**|sous-entité|Les deux objets sont des objets enfants.|
|L’entité enfant de l’entité composite est **Entité prédéfinie** telle que Nombre|sous-entité avec le nom de l’Entité prédéfinie telle que Nombre, et la sous-entité possède la _fonctionnalité_ de l’Entité Nombre prédéfinie avec l’option de contrainte définie sur _true_.|La sous-entité contient une fonctionnalité avec une contrainte au niveau de la sous-entité.|
|L’entité enfant de l’entité composite est **Entité prédéfinie** telle que Nombre, et l’entité prédéfinie a un **rôle**|sous-entité avec nom de rôle, et la sous-entité possède la fonctionnalité de l’Entité Nombre prédéfinie avec l’option de contrainte définie sur true.|La sous-entité contient une fonctionnalité avec une contrainte au niveau de la sous-entité.|
|Role|sous-entité|Le nom du rôle devient celui de la sous-entité. La sous-entité est un descendant direct de l’entité de machine-learning.|

## <a name="begin-upgrade-process"></a>Démarrer le processus de mise à niveau

Avant de procéder à la mise à niveau, veillez à :

* Changer de version si vous n’utilisez pas celle qui doit être mise à niveau


1. Démarrez le processus de mise à niveau à partir de la notification ou attendez la prochaine invite planifiée.

    > [!div class="mx-imgBorder"]
    > ![Démarrer la mise à niveau à partir des notifications](./media/update-composite-entity/notification-begin-update.png)

1. Dans la fenêtre contextuelle, sélectionnez **Mettre à niveau maintenant**.

1. Lisez la section relative aux **conséquences de la mise à niveau**, puis sélectionnez **Continuer**.

1. Sélectionnez dans la liste les entités composites à mettre à niveau, puis sélectionnez **Continuer**.

1. Vous pouvez déplacer les modifications pour lesquelles aucun apprentissage n’a été effectué de la version actuelle vers la version mise à niveau en activant les cases à cocher.

1. Sélectionnez **Continuer** pour démarrer le processus de mise à niveau.

1. La barre de progression indique l’état du processus de mise à niveau.

1. Une fois le processus terminé, vous êtes sur une nouvelle version avec la nouvelle entité de machine-learning. Sélectionnez **Essayer vos nouvelles entités** pour voir la nouvelle entité.

    Si la mise à niveau ou l’apprentissage a échoué pour la nouvelle entité, une notification fournit des informations complémentaires.

1. Dans la page Liste des entités, les nouvelles entités sont signalées par la mention **NEW** en regard du nom du type.

## <a name="next-steps"></a>Étapes suivantes

* [Créateurs et collaborateurs](luis-how-to-collaborate.md)