---
title: Activer automatiquement la sauvegarde lors de la création de machines virtuelles avec Azure Policy
description: Article décrivant comment utiliser Azure Policy pour activer automatiquement la sauvegarde de toutes les machines virtuelles créées dans une étendue donnée
ms.topic: conceptual
ms.date: 11/08/2019
ms.openlocfilehash: 78fe0ccdbf6f1cc3498d14530d7492a86e8bf730
ms.sourcegitcommit: 2989396c328c70832dcadc8f435270522c113229
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92174082"
---
# <a name="auto-enable-backup-on-vm-creation-using-azure-policy"></a>Activer automatiquement la sauvegarde lors de la création de machines virtuelles avec Azure Policy

L’une des principales responsabilités d’un administrateur de sauvegarde ou de conformité dans une organisation est de s’assurer que toutes les machines vitales pour l’entreprise sont sauvegardées avec la rétention appropriée.

Aujourd’hui, Sauvegarde Azure fournit une stratégie intégrée (à l’aide d’Azure Policy) qui peut être attribuée à **toutes les machines virtuelles Azure dans un emplacement spécifié au sein d’un abonnement ou d’un groupe de ressources**. Quand cette stratégie est affectée à une étendue donnée, toutes les machines virtuelles créées dans cette étendue sont automatiquement configurées pour la sauvegarde dans un **coffre existant au même emplacement et dans le même abonnement**. L’utilisateur peut spécifier le coffre et la stratégie de rétention auxquels les machines virtuelles sauvegardées doivent être associées.

## <a name="supported-scenarios"></a>Scénarios pris en charge

* La stratégie intégrée est actuellement prise en charge uniquement pour les machines virtuelles Azure. Les utilisateurs doivent veiller à ce que la stratégie de rétention spécifiée lors de l’attribution soit une stratégie de rétention de machine virtuelle. Reportez-vous à [ce document](./backup-azure-policy-supported-skus.md) pour voir toutes les références SKU de machines virtuelles prises en charge par cette stratégie.

* La stratégie peut être attribuée à un seul emplacement et à un seul abonnement à la fois. Pour activer la sauvegarde des machines virtuelles sur plusieurs emplacements et abonnements, plusieurs instances de l’attribution de stratégie doivent être créées, une pour chaque combinaison d’emplacement et d’abonnement.

* Le coffre spécifié et les machines virtuelles configurées pour la sauvegarde peuvent se trouver sous des groupes de ressources différents.

* L’étendue du groupe d’administration n’est pas prise en charge pour le moment.

* La stratégie intégrée n’est actuellement pas disponible dans les clouds nationaux.

[!INCLUDE [backup-center.md](../../includes/backup-center.md)]

## <a name="using-the-built-in-policy"></a>Utilisation de la stratégie intégrée

Pour attribuer la stratégie à l’étendue requise, suivez les étapes ci-dessous :

1. Connectez-vous au Portail Azure et accédez au tableau de bord **Stratégie**.
1. Sélectionnez **Définitions** dans le menu de gauche pour obtenir la liste de toutes les stratégies intégrées dans les ressources Azure.
1. Filtrez la liste sur **Catégorie=Sauvegarde**. La liste est filtrée jusqu’à l’obtention d’une stratégie unique nommée « Configurer la sauvegarde sur les machines virtuelles d’un emplacement vers un coffre central existant dans le même emplacement ».
![Tableau de bord Stratégie](./media/backup-azure-auto-enable-backup/policy-dashboard.png)
1. Sélectionnez le nom de la stratégie. Vous serez redirigé vers sa définition détaillée.
![Policy Definition pane](./media/backup-azure-auto-enable-backup/policy-definition-blade.png)
1. Sélectionnez le bouton **Attribuer** en haut du volet. Cela vous redirige vers le volet **Attribuer une stratégie**.
1. Sous **Fonctions de base**, sélectionnez le bouton de sélection à côté du champ **Étendue**. Cela ouvre un volet contextuel à droite dans lequel vous pouvez sélectionner l’abonnement de la stratégie à appliquer. Si vous le souhaitez, vous pouvez également sélectionner un groupe de ressources, de façon à ce que la stratégie soit appliquée uniquement pour les machines virtuelles d’un groupe de ressources particulier.
![Concepts de base de l’attribution d’une stratégie](./media/backup-azure-auto-enable-backup/policy-assignment-basics.png)
1. Dans l’onglet **Paramètres**, choisissez un emplacement dans la liste déroulante, puis sélectionnez le coffre et la stratégie de sauvegarde auxquels les machines virtuelles de l’étendue doivent être associées.
![Paramètres d’attribution de stratégie](./media/backup-azure-auto-enable-backup/policy-assignment-parameters.png)
1. Assurez-vous que la valeur **Effet** est configurée sur deployIfNotExists.
1. Accédez à **Vérifier+créer**, puis sélectionnez **Créer**.

> [!NOTE]
>
> Azure Policy peut également être utilisé sur des machines virtuelles existantes, à l’aide de la [correction](../governance/policy/how-to/remediate-resources.md).

> [!NOTE]
>
> Nous vous recommandons de ne pas attribuer cette stratégie à plus de 200 machines virtuelles à la fois. Si la stratégie est attribuée à plus de 200 machines virtuelles, ce nombre peut entraîner le déclenchement de la sauvegarde quelques heures plus tard que celle spécifiée par la planification.

## <a name="next-steps"></a>Étapes suivantes

[En savoir plus sur Azure Policy](../governance/policy/overview.md)
