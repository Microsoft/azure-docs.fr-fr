---
title: Configuration à grande échelle des paramètres de diagnostic de coffre
description: Configurez les paramètres de diagnostic Log Analytics pour tous les coffres d’une étendue donnée avec Azure Policy.
ms.topic: conceptual
ms.date: 02/14/2020
ms.openlocfilehash: 55461937381f7551c42714c835d4755ab65f175b
ms.sourcegitcommit: 2989396c328c70832dcadc8f435270522c113229
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92171522"
---
# <a name="configure-vault-diagnostics-settings-at-scale"></a>Configuration à grande échelle des paramètres de diagnostic de coffre

La solution de reporting fournie par la Sauvegarde Azure s’appuie sur Log Analytics (LA). Pour que les données d’un coffre donné soient envoyées à Log Analytics, il faut créer un [paramètre de diagnostic](./backup-azure-diagnostic-events.md) pour ce coffre.

Il est souvent fastidieux d’ajouter manuellement un paramètre de diagnostic par coffre. Par ailleurs, il faut activer les paramètres de diagnostic de chaque nouveau coffre créé pour pouvoir afficher ses rapports.

Pour simplifier la création de paramètres de diagnostic à grande échelle (avec Log Analytics comme destination), la Sauvegarde Azure fournit une stratégie [Azure Policy](../governance/policy/index.yml) intégrée. Cette stratégie ajoute un paramètre de diagnostic Log Analytics à tous les coffres d’un abonnement ou d’un groupe de ressources donné. Les sections suivantes donnent des instructions relatives à l’utilisation de cette stratégie.

## <a name="supported-scenarios"></a>Scénarios pris en charge

* La stratégie peut être appliquée simultanément à tous les coffres Recovery Services d’un abonnement donné (ou à un groupe de ressources dans l’abonnement). L’utilisateur qui attribue la stratégie doit disposer d’un accès **Propriétaire** à l’abonnement auquel la stratégie est attribuée.

* L’espace de travail Log Analytics indiqué par l’utilisateur (auquel seront envoyées les données de diagnostic) peut se trouver dans un abonnement différent des coffres auxquels la stratégie est attribuée. L’utilisateur doit disposer d’un accès **Lecteur**, **Contributeur** ou **Propriétaire** à l’abonnement dans lequel se trouve l’espace de travail Log Analytics spécifié.

* L’étendue du groupe d’administration n’est pas prise en charge pour le moment.

* La stratégie intégrée n’est actuellement pas disponible dans les clouds nationaux.

[!INCLUDE [backup-center.md](../../includes/backup-center.md)]

## <a name="assigning-the-built-in-policy-to-a-scope"></a>Attribution de la stratégie intégrée à une étendue

Pour attribuer la stratégie aux coffres dans l’étendue requise, suivez les étapes ci-dessous :

1. Connectez-vous au Portail Azure et accédez au tableau de bord **Stratégie**.
2. Sélectionnez **Définitions** dans le menu de gauche pour obtenir la liste de toutes les stratégies intégrées dans les ressources Azure.
3. Filtrez la liste sur **Catégorie=Sauvegarde**. Recherchez la stratégie nommée **[Preview]: Déployer les paramètres de diagnostic du coffre Recovery Services sur l’espace de travail Log Analytics pour les catégories propres à la ressource**.

    ![Volet Définition de stratégie](./media/backup-azure-policy-configure-diagnostics/policy-definition-blade.png)

4. Sélectionnez le nom de la stratégie. Vous serez redirigé vers sa définition détaillée.

    ![Définition détaillée de la stratégie](./media/backup-azure-policy-configure-diagnostics/detailed-policy-definition.png)

5. Sélectionnez le bouton **Attribuer** en haut du volet. Cela vous redirige vers le volet **Attribuer une stratégie**.

6. Sous **Fonctions de base**, sélectionnez le bouton de sélection à côté du champ **Étendue**. Cela ouvre un volet contextuel à droite dans lequel vous pouvez sélectionner l’abonnement de la stratégie à appliquer. Si vous le souhaitez, vous pouvez également sélectionner un groupe de ressources, de façon à ce que la stratégie ne s’applique qu’aux coffres de ce groupe.

    ![Concepts de base de l’attribution de stratégie](./media/backup-azure-policy-configure-diagnostics/policy-assignment-basics.png)

7. Sous **Paramètres**, entrez les informations suivantes :

    * **Nom du profil** : nom qui sera attribué aux paramètres de diagnostic créés par la stratégie.
    * **Espace de travail Log Analytics** : espace de travail Log Analytics auquel le paramètre de diagnostic doit être associé. Les données de diagnostic de tous les coffres de l’étendue de l’attribution de stratégie seront envoyées à l’espace de travail Log Analytics spécifié.

    * **Nom de la balise d’exclusion (facultatif) et Valeur de la balise d’exclusion (facultatif)**  : vous pouvez choisir d’exclure de l’attribution de stratégie les coffres contenant un nom et une valeur de balise spécifiques. Par exemple, si vous ne souhaitez **pas** ajouter un paramètre de diagnostic aux coffres dont la balise « isTest » a la valeur « yes », vous devez entrer « isTest » dans le champ **Nom de la balise d’exclusion** et « yes » dans le champ **Valeur de la balise d’exclusion**. Si l’un des deux champs (ou les deux) n’est pas renseigné, la stratégie sera appliquée à tous les coffres correspondants, indépendamment des balises qu’ils contiennent.

    ![Paramètres d’attribution de stratégie](./media/backup-azure-policy-configure-diagnostics/policy-assignment-parameters.png)

8. **Créer une tâche de correction** : une fois la stratégie attribuée à une étendue, les paramètres de diagnostic Log Analytics sont automatiquement configurés pour tous les nouveaux coffres créés dans cette étendue (dans les 30 minutes suivant leur création). Pour ajouter un paramètre de diagnostic à des coffres existants dans l’étendue, vous pouvez déclencher une tâche de correction à l’heure d’attribution de la stratégie. Pour déclencher une tâche de correction, cochez la case **Créer une tâche de correction**.

    ![Correction de l’attribution de stratégie](./media/backup-azure-policy-configure-diagnostics/policy-assignment-remediation.png)

9. Accédez à l’onglet **Vérifier + créer**, puis sélectionnez **Créer**.

## <a name="under-what-conditions-will-the-remediation-task-apply-to-a-vault"></a>Conditions d’application de la tâche de correction à un coffre

La tâche de correction s’applique aux coffres non conformes selon la définition de la stratégie. Un coffre est non conforme s’il remplit l’une des conditions suivantes :

* Le coffre ne comporte aucun paramètre de diagnostic.
* Le coffre comporte des paramètres de diagnostic, mais aucun des deux n’a **tous** les événements propres à la ressource activés avec Log Analytics comme destination ni **Propre à une ressource** sélectionné sur le bouton bascule.

Ainsi, même si l’utilisateur dispose d’un coffre pour lequel l’événement AzureBackupReport est activé en mode AzureDiagnostics (ce qui est pris en charge par Rapports de sauvegarde), la tâche de correction s’applique malgré tout à ce coffre, puisque le mode propre à la ressource est [à l’avenir](./backup-azure-diagnostic-events.md#legacy-event) recommandé pour créer des paramètres de diagnostic.

De plus, si seulement une partie des six événements propres à une ressource sont activés sur le coffre de l’utilisateur, la tâche de correction s’appliquera à ce coffre, puisque Rapports de sauvegarde ne fonctionnera comme prévu que si les six événements propres à la ressource sont activés.

> [!NOTE]
>
> Si le coffre comporte un paramètre de diagnostic avec un **sous-ensemble des catégories propres à une ressources** activées et qu’il est configuré de façon à envoyer des données à un espace de travail Log Analytics spécifique, par exemple « Espace de travail X », la tâche de correction échouera (pour ce coffre uniquement) si l’espace de travail Log Analytics de destination fourni dans l’attribution de stratégie est le **même** « Espace de travail X ».
>
>En effet, si les événements activés par deux paramètres de diagnostic différents sur la même ressource **se chevauchent** sous une forme ou une autre, les paramètres ne peuvent donc pas avoir le même espace de travail Log Analytics comme destination. Vous devrez résoudre ce problème manuellement, en accédant au coffre concerné et en configurant un paramètre de diagnostic avec un autre espace de travail Log Analytics comme destination.
>
> Il est à noter que la tâche de correction n’échoue **pas** si seul AzureBackupReport est activé pour le paramètre de diagnostic existant et que la destination est l’Espace de travail X, puisque, dans ce cas, il n’y a aucun chevauchement entre les événements activés par le paramètre existant et les événements activés par le paramètre créé par la tâche de correction.

## <a name="next-steps"></a>Étapes suivantes

* [En savoir plus sur l’utilisation des rapports de sauvegarde](./configure-reports.md)
* [En savoir plus sur Azure Policy](../governance/policy/index.yml)
* [Utilisation d’Azure Policy pour activer automatiquement la sauvegarde de toutes les machines virtuelles d’une étendue](./backup-azure-auto-enable-backup.md)
