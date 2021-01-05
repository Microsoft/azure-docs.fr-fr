---
title: Créer, afficher et gérer des alertes de journal d’activité dans Azure Monitor
description: Créer des alertes de journal d’activité à l’aide du portail Azure, d’un modèle Azure Resource Manager et d’Azure PowerShell.
ms.topic: conceptual
ms.subservice: alerts
ms.date: 06/25/2019
ms.openlocfilehash: 7914f3ee57d014de2f34fca301b587c6297bebe3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90983094"
---
# <a name="create-view-and-manage-activity-log-alerts-by-using-azure-monitor"></a>Créer, afficher et gérer des alertes de journal d’activité à l’aide d’Azure Monitor  

## <a name="overview"></a>Vue d’ensemble

Les alertes de journal d’activité s’activent lorsqu’un nouvel événement du journal d’activité correspond aux conditions spécifiées dans l’alerte.

Elles concernent les ressources Azure. Pour les créer, il est possible d’utiliser un modèle Azure Resource Manager. Elles peuvent également être créées, mises à jour ou supprimées dans le portail Azure. En général, les alertes de journal d’activité permettent de recevoir une notification en cas de modification particulière des ressources de l’abonnement Azure, souvent à l’échelle d’un groupe de ressources ou d’une ressource en particulier. Par exemple, vous pouvez être averti lorsqu’une machine virtuelle dans l’exemple de groupe de ressources **myProductionResourceGroup** est supprimée. Vous pouvez également vouloir être averti si de nouveaux rôles sont attribués à un utilisateur dans votre abonnement.

> [!IMPORTANT]
> Il n’est pas possible de créer des alertes de notification Service Health avec l’interface de création d’alertes de journal d'activité. Pour en savoir plus sur la création et l’utilisation des notifications Service Health, consultez [Recevoir des alertes de journal d’activité sur les notifications Service Health](../../service-health/alerts-activity-log-service-notifications-portal.md).

Lorsque vous créez des règles d’alerte, vérifiez les points suivants :

- L’abonnement présent dans l’étendue n’est pas différent de celui dans lequel l’alerte est créée.
- Les critères doivent être : niveau, état, appelant, groupe de ressources, ID de ressource, type de ressource ou catégorie d’événement sur lesquels l’alerte est configurée.
- Il n'existe pas de condition « anyOf » ou de conditions imbriquées dans le fichier JSON de configuration d'alerte. En principe, une seule condition « allOf » est autorisée sans autre condition « allOf » ou « anyOf ».
- Lorsque la catégorie est « administration », vous devez spécifier au moins l’un des critères précédents dans votre alerte. L’alerte créée ne s’activera peut-être pas à chaque fois qu’un événement sera créé dans les journaux d’activité.
- Les alertes ne peuvent pas être créées pour des événements dans la catégorie Alerte du journal d’activité.

## <a name="azure-portal"></a>Portail Azure

Vous pouvez utiliser le portail Azure pour créer et modifier des règles d'alerte du journal d'activité. L’expérience est intégrée au journal d’activité Azure pour garantir la transparence de la création d’alertes pour des événements d’intérêt spécifiques.

### <a name="create-with-the-azure-portal"></a>Créer avec le Portail Azure

Procédez comme suit.

1. Dans le portail Azure, sélectionnez **Surveiller** > **Alertes**.
2. Sélectionnez **Nouvelle règle d'alerte** dans le coin supérieur gauche de la fenêtre **Alertes**.

     ![Nouvelle règle d’alerte](media/alerts-activity-log/AlertsPreviewOption.png)

     La fenêtre **Créer une règle** s’affiche.

      ![Nouvelles options de règle d’alerte](media/alerts-activity-log/create-new-alert-rule-options.png)

3. Sous **Définir une condition d’alerte,** indiquez les informations suivantes, puis sélectionnez **Fait** :

   - **Cible de l’alerte :** Pour afficher et sélectionner la cible de la nouvelle alerte, utilisez **Filtrer par abonnement** / **Filtrer par type de ressource**. Sélectionnez la ressource ou le groupe de ressources dans la liste affichée.

     > [!NOTE]
     > 
     > Vous pouvez sélectionner uniquement ressource suivi [Azure Resource Manager](../../azure-resource-manager/management/overview.md), groupe de ressources ou abonnement complet pour un signal de journal d'activité. 

     **Vue Exemple de cible de l’alerte**

     ![Sélectionner la cible](media/alerts-activity-log/select-target.png)

   - Sous **Critères cibles**, sélectionnez **Ajouter des critères**. Tous les signaux disponibles pour la cible sont affichés, y compris ceux des différentes catégories du **journal d'activité**. Le nom de la catégorie est ajouté au nom du **service d’analyse**.

   - Sélectionnez le signal dans la liste des différentes opérations possibles pour le type **Journal d’activité**.

     Vous pouvez sélectionner la chronologie de l’historique du journal et la logique d’alerte correspondante pour ce signal cible :

     **Écran Ajouter des critères**

     ![Ajouter des critères](media/alerts-activity-log/add-criteria.png)
     
     > [!NOTE]
     > 
     >  Pour obtenir des règles de haute qualité et efficaces, nous demandons d’ajouter au moins une condition supplémentaire aux règles avec le signal « All administrative ». 
     > Dans le cadre de la définition de l’alerte, vous devez remplir l’une des listes déroulantes : « Niveau de l’événement », « État » ou « Initié par » et par, la règle sera plus spécifique.

     - **Échelle de l’historique** : les événements disponibles pour l’opération sélectionnée peuvent s’être déroulés au cours des 6, 12 ou 24 dernières heures ou la dernière semaine.

     - **Logique d'alerte** :

       - **Niveau de l'événement** : Niveau de gravité de l’événement : _Commentaires_, _Information_, _Avertissement_, _Erreur_ ou _Critique_.
       - **État** : État de l’événement : _Démarré_, _Échec_ ou _Réussi_.
       - **Événement lancé par** : également appelé l’appelant. L’adresse e-mail ou l’identificateur Azure Active Directory de l’utilisateur qui a effectué l’opération.

       Une logique d’alerte a été appliquée à cet exemple de graphique de signal :

       ![Critères sélectionnés](media/alerts-activity-log/criteria-selected.png)

4. Sous **Définir les détails de l’alerte**, indiquez les informations suivantes :

    - **Nom de la règle d’alerte** : nom de la nouvelle règle d’alerte.
    - **Description** : Description de la nouvelle règle d’alerte.
    - **Enregistrer l'alerte dans le groupe de ressources** : sélectionnez le groupe de ressources dans lequel vous souhaitez enregistrer cette nouvelle règle.

5. Dans le menu déroulant sous **Groupe d’actions**, spécifiez le groupe d’actions que vous souhaitez affecter à cette nouvelle règle d’alerte. Vous pouvez également [créer un groupe d’actions](./action-groups.md) et l’affecter à la nouvelle règle. Pour créer un groupe, sélectionnez **+ Nouveau groupe**.

6. Pour activer les règles après les avoir créées, cliquez sur **Oui** dans l’option **Activer la règle lors de la création**.
7. Sélectionnez **Créer une règle d’alerte**.

    La nouvelle règle d’alerte de journal d’activité est créée et un message de confirmation s’affiche en haut à droite de la fenêtre.

    Vous pouvez activer, désactiver, modifier ou supprimer une règle. En savoir plus sur la gestion des règles de journal d’activité.


Pour bien comprendre les conditions dans lesquelles des règles d’alerte peuvent être créées sur le journal d’activité, il est possible d’explorer ou de filtrer les événements par le biais du [Journal d’activité dans le portail Azure](./activity-log.md#view-the-activity-log). Dans l’écran **Azure Monitor - Journal d’activité**, vous pouvez filtrer ou rechercher l’événement requis, puis créer une alerte à l’aide du bouton **Ajouter une alerte de journal d’activité**. Suivez ensuite les étapes 4 à 7 comme indiqué précédemment.
    
 ![Ajouter une alerte du journal d’activité](media/alerts-activity-log/add-activity-log.png)
    

### <a name="view-and-manage-in-the-azure-portal"></a>Afficher et gérer dans le Portail Azure

1. Dans le portail Azure, sélectionnez **Surveiller** > **Alertes**. Dans l’angle supérieur gauche de la fenêtre, sélectionnez **Gérer les règles d'alerte**.

    ![Capture d’écran montrant le journal d’activité avec la zone de recherche mise en surbrillance.](media/alerts-activity-log/manage-alert-rules.png)

    La liste des règles disponibles s’affiche.

2. Recherchez la règle de journal d’activité à modifier.

    ![Rechercher des règles d’alerte de journal d'activité](media/alerts-activity-log/searth-activity-log-rule-to-edit.png)

    Vous pouvez utiliser les filtres disponibles (_Abonnement_, _Groupe de ressources_, _Ressource_, _Type de signal_ ou _État_) pour trouver la règle d’activité que vous souhaitez modifier.

   > [!NOTE]
   > 
   > Les seuls champs que vous pouvez éditer sont : **Description** , **Critères cibles** et **Groupes d’actions**.

3. Sélectionnez la règle, puis double-cliquez pour modifier ses options. Apportez les modifications nécessaires, puis sélectionnez **Enregistrer**.

   ![Gérer les règles d’alerte](media/alerts-activity-log/activity-log-rule-edit-page.png)

4. Vous pouvez activer, désactiver ou supprimer une règle. Sélectionnez l’option souhaitée en haut de la fenêtre, après avoir sélectionné la règle comme décrit à l’étape 2.


## <a name="azure-resource-manager-template"></a>Modèle Azure Resource Manager
Pour créer une règle d’alerte de journal d’activité à l’aide d’un modèle Azure Resource Manager, créez une ressource de type `microsoft.insights/activityLogAlerts`. Puis, renseignez toutes les propriétés associées. Voici un modèle qui crée une règle d’alerte de journal d’activité :

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "activityLogAlertName": {
      "type": "string",
      "metadata": {
        "description": "Unique name (within the Resource Group) for the Activity log alert."
      }
    },
    "activityLogAlertEnabled": {
      "type": "bool",
      "defaultValue": true,
      "metadata": {
        "description": "Indicates whether or not the alert is enabled."
      }
    },
    "actionGroupResourceId": {
      "type": "string",
      "metadata": {
        "description": "Resource Id for the Action group."
      }
    }
  },
  "resources": [   
    {
      "type": "Microsoft.Insights/activityLogAlerts",
      "apiVersion": "2017-04-01",
      "name": "[parameters('activityLogAlertName')]",      
      "location": "Global",
      "properties": {
        "enabled": "[parameters('activityLogAlertEnabled')]",
        "scopes": [
            "[subscription().id]"
        ],        
        "condition": {
          "allOf": [
            {
              "field": "category",
              "equals": "Administrative"
            },
            {
              "field": "operationName",
              "equals": "Microsoft.Resources/deployments/write"
            },
            {
              "field": "resourceType",
              "equals": "Microsoft.Resources/deployments"
            }
          ]
        },
        "actions": {
          "actionGroups":
          [
            {
              "actionGroupId": "[parameters('actionGroupResourceId')]"
            }
          ]
        }
      }
    }
  ]
}
```
L’exemple JSON précédent peut, par exemple, être enregistré en tant que sampleActivityLogAlert.json pour les besoins de cette procédure pas à pas, et peut être déployé à l’aide d’[Azure Resource Manager dans le portail Azure](../../azure-resource-manager/templates/deploy-portal.md).

  > [!NOTE]
  > 
  > Notez que le plus haut niveau d'alerte du journal d'activité pouvant être défini est l'abonnement.
  > Cela signifie qu’il n’y a pas d’option permettant de définir une alerte sur deux abonnements. par conséquent, la définition doit être une alerte par abonnement.

Les champs suivants sont les options que vous pouvez utiliser dans le modèle Azure Resource Manager pour les champs de conditions : Notez que « Resource Health », « Advisor » et « Service Health » ont des champs de propriétés supplémentaires pour leurs champs spéciaux. 
1. resourceId :  ID de la ressource concernée dans l’événement du journal d’activité sur lequel l’alerte doit être générée.
2. category : Catégorie de l’événement du journal d’activité. Par exemple : Administrative, ServiceHealth, ResourceHealth, Autoscale, Security, Recommendation, Policy.
3. caller : Adresse e-mail ou identificateur Azure Active Directory de l’utilisateur qui a effectué l’opération de l’événement du journal d’activité.
4. level : Niveau de l’activité dans l’événement du journal d’activité sur lequel l’alerte doit être générée. Par exemple : Critical, Error, Warning, Informational, Verbose.
5. operationName : Nom de l’opération dans l’événement du journal d’activité. Par exemple : Microsoft.Resources/deployments/write
6. resourceGroup : Nom du groupe de ressources pour la ressource affectée dans l’événement du journal d’activité.
7. resourceProvider : [Explication des fournisseurs et types de ressources Azure](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fazure%2Fazure-resource-manager%2Fmanagement%2Fresource-providers-and-types&data=02%7C01%7CNoga.Lavi%40microsoft.com%7C90b7c2308c0647c0347908d7c9a2918d%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637199572373543634&sdata=4RjpTkO5jsdOgPdt%2F%2FDOlYjIFE2%2B%2BuoHq5%2F7lHpCwQw%3D&reserved=0). Pour obtenir la liste qui mappe les fournisseurs de ressources aux services Azure, consultez [Fournisseurs de ressources pour les services Azure](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fazure%2Fazure-resource-manager%2Fmanagement%2Fazure-services-resource-providers&data=02%7C01%7CNoga.Lavi%40microsoft.com%7C90b7c2308c0647c0347908d7c9a2918d%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637199572373553639&sdata=0ZgJPK7BYuJsRifBKFytqphMOxMrkfkEwDqgVH1g8lw%3D&reserved=0).
8. status : Chaîne décrivant l’état de l’opération dans l’événement d’activité. Par exemple : Started, In Progress, Succeeded, Failed, Active, Resolved
9. subStatus : En général, le code d’état HTTP de l’appel REST correspondant. Peut également inclure d’autres chaînes décrivant un sous-état.   Par exemple : OK (code d’état HTTP : 200), Créé (code d’état HTTP : 201), Accepté (code d’état HTTP : 202, Aucun contenu (code d’état HTTP : 204, Requête incorrecte (code d’état HTTP : 400, Introuvable (code d’état HTTP : 404), Conflit (code d’état HTTP : 409), Erreur interne du serveur (code d’état HTTP : 500), Service indisponible (code d’état HTTP : 503), Dépassement de délai de la passerelle (code d'état HTTP : 504).
10. resourceType: Type de la ressource affectée par l’événement. Par exemple : Microsoft.Resources/deployments

Par exemple :

```json
"condition": {
          "allOf": [
            {
              "field": "category",
              "equals": "Administrative"
            },
            {
              "field": "resourceType",
              "equals": "Microsoft.Resources/deployments"
            }
          ]
        }

```
Pour plus d’informations sur les champs du journal d’activité, consultez [ceci](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fazure%2Fazure-monitor%2Fplatform%2Factivity-log-schema&data=02%7C01%7CNoga.Lavi%40microsoft.com%7C90b7c2308c0647c0347908d7c9a2918d%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C637199572373563632&sdata=6QXLswwZgUHFXCuF%2FgOSowLzA8iOALVgvL3GMVhkYJY%3D&reserved=0).



> [!NOTE]
> L'activation d'une nouvelle règle d'alerte de journal d'activité peut prendre jusqu'à 5 minutes.

## <a name="rest-api"></a>API REST 
L’[API Azure Monitor Activity Log Alerts](/rest/api/monitor/activitylogalerts) est une API REST. Elle est entièrement compatible avec l'API REST Azure Resource Manager. Elle peut être utilisée via PowerShell à l’aide d’applets de commande Resource Manager en plus de l’interface de ligne de commande (CLI) Azure.

## <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

### <a name="deploy-the-resource-manager-template-with-powershell"></a>Déployer le modèle Azure Resource Manager avec PowerShell
Pour utiliser PowerShell afin de déployer l’exemple Resource Manager indiqué dans la section [Modèle Azure Resource Manager](#azure-resource-manager-template) précédente, utilisez la commande suivante :

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "myRG" -TemplateFile sampleActivityLogAlert.json -TemplateParameterFile sampleActivityLogAlert.parameters.json
```

Où sampleActivityLogAlert.parameters.json contient les valeurs communiquées pour les paramètres requis pour la création d’une règle d’alerte.

### <a name="use-activity-log-powershell-cmdlets"></a>Utiliser les applets de commande PowerShell pour le journal d'activité

Les alertes de journal d'activité disposent de cmdlets PowerShell dédiées :

- [Set-AzActivityLogAlert](/powershell/module/az.monitor/set-azactivitylogalert) : Crée ou met à jour une alerte de journal d’activité existante.
- [Get-AzActivityLogAlert](/powershell/module/az.monitor/get-azactivitylogalert) : Obtient une ou plusieurs ressources d’alerte de journal d'activité.
- [Enable-AzActivityLogAlert](/powershell/module/az.monitor/enable-azactivitylogalert) : Active une alerte de journal d’activité existante et définit ses balises.
- [Disable-AzActivityLogAlert](/powershell/module/az.monitor/disable-azactivitylogalert) : Désactive une alerte de journal d’activité existante et définit ses balises.
- [Remove-AzActivityLogAlert](/powershell/module/az.monitor/remove-azactivitylogalert) : Supprime une alerte de journal d’activité.

## <a name="azure-cli"></a>Azure CLI

Les commandes Azure CLI dédiées situées sous l'ensemble [az monitor activity-log alert](/cli/azure/monitor/activity-log/alert) permettent de gérer les règles d'alerte de journal d'activité.

Pour créer une règle d'alerte de journal d'activité, utilisez les commandes suivantes dans cet ordre :

1. [az monitor activity-log alert create](/cli/azure/monitor/activity-log/alert#az-monitor-activity-log-alert-create) : permet de créer une ressource de règle d'alerte de journal d'activité.
1. [az monitor activity-log alert scope](/cli/azure/monitor/activity-log/alert/scope) : permet d'ajouter une étendue pour la règle d'alerte de journal d'activité créée.
1. [az monitor activity-log alert action-group](/cli/azure/monitor/activity-log/alert/action-group) : permet d'ajouter un groupe d'actions à la règle d'alerte de journal d'activité.

Pour récupérer une ressource de règle d'alerte de journal d'activité, utilisez la commande Azure CLI [az monitor activity-log alert show](/cli/azure/monitor/activity-log/alert#az-monitor-activity-log-alert-show
). Pour afficher toutes les ressources de règle d'alerte de journal d'activité d'un groupe de ressources, utilisez la commande [az monitor activity-log alert list](/cli/azure/monitor/activity-log/alert#az-monitor-activity-log-alert-list).
Les ressources de règle d'alerte de journal d'activité peuvent être supprimées à l'aide de la commande Azure CLI [az monitor activity-log alert delete](/cli/azure/monitor/activity-log/alert#az-monitor-activity-log-alert-delete).

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur le [schéma de webhook des journaux d’activité](./activity-log-alerts-webhook.md).
- Lisez une [présentation des journaux d’activité](./activity-log-alerts.md).
- En savoir plus sur les [groupes d’actions](./action-groups.md).  
- En savoir plus sur les [notifications sur l’intégrité du service](../../service-health/service-notifications.md).

