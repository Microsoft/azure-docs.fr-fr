---
title: Publier une annonce dans un lab avec Azure DevTest Labs | Microsoft Docs
description: Découvrez comment publier une annonce personnalisée dans un lab existant pour informer les utilisateurs des dernières modifications ou des derniers ajouts apportés au lab dans Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 72c6ed7ab691ece094673ff5a158ffa769a9f7a1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88270765"
---
# <a name="post-an-announcement-to-a-lab-in-azure-devtest-labs"></a>Publier une annonce dans un lab avec Azure DevTest Labs

En tant qu’administrateur de lab, vous pouvez publier une annonce personnalisée dans un lab existant pour informer les utilisateurs des dernières modifications ou ajouts apportés au lab. Par exemple, pour les renseigner sur :

- Les nouvelles tailles de machine virtuelle disponibles
- Les images actuellement inutilisables
- Les mises à jour des stratégies de lab

Une fois publiée, l’annonce s’affiche dans la page Vue d’ensemble du lab, et l’utilisateur peut la sélectionner pour obtenir plus de détails.

La fonctionnalité d’annonce est destinée à être utilisée pour les notifications temporaires.  Vous pouvez facilement désactiver une annonce lorsqu’elle n’est plus nécessaire.

## <a name="steps-to-post-an-announcement-in-an-existing-lab"></a>Étapes de la publication d’une annonce dans un lab existant

1. Connectez-vous au [portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).
1. Si nécessaire, sélectionnez **Autres services**, puis **DevTest Labs** dans la liste. (Votre lab peut déjà être affiché dans le Tableau de bord sous **Toutes les ressources**).
1. Dans la liste des labs, sélectionnez le lab dans lequel vous souhaitez publier une annonce.
1. Dans la zone **Vue d’ensemble** du laboratoire, sélectionnez **Configuration et stratégies**.

    ![Bouton Configuration et stratégies](./media/devtest-lab-announcements/devtestlab-config-and-policies.png)

1. Sur la gauche, sous **PARAMÈTRES**, sélectionnez **Annonce du lab**.

    ![Bouton Annonce du lab](./media/devtest-lab-announcements/devtestlab-announcements.png)

1. Pour créer un message à l’intention des utilisateurs de ce lab, définissez **Activé** sur **Oui**.

1. Vous pouvez entrer une **Date d’expiration** pour spécifier la date et l’heure après lesquelles l’annonce ne sera plus présentée aux utilisateurs. À défaut, l’annonce restera jusqu'à ce que vous la désactiviez.

   > [!NOTE]
   > Après expiration, l’annonce ne sera plus présentée aux utilisateurs, mais elle subsistera toujours dans le volet **Annonce du labo**. Vous pourrez lui apporter des modifications et la réactiver.
   >
   >

1. Entrez un **Titre de l’annonce** et le **Texte de l’annonce**.

   Le titre peut comporter jusqu’à 100 caractères, et il est visible par l’utilisateur dans la page Vue d’ensemble du lab. Si l’utilisateur sélectionne le titre, le texte de l’annonce s’affiche.

   Le texte de l’annonce accepte Markdown. Au fur et à mesure que vous entrez le texte de l’annonce, vous pouvez voir le message s’afficher dans la zone Aperçu en bas de l’écran.

    ![Écran d’annonce du lab pour créer le message.](./media/devtest-lab-announcements/devtestlab-post-announcement.png)


1. Sélectionnez **Enregistrer** lorsque l’annonce est prête pour la publication.

Quand vous ne souhaitez plus afficher cette annonce pour les utilisateurs du lab, revenez à la page **Annonce du lab** et définissez **Activé** sur **Non**. Si vous avez spécifié une date d’expiration, l’annonce sera automatiquement désactivée à la date et à l’heure correspondantes.

## <a name="steps-for-users-to-view-an-announcement"></a>Étapes de l’affichage d’une annonce pour la consultation par les utilisateurs

1. À partir du [portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040), sélectionnez un lab.

1. Si une annonce est publiée pour ce lab, un avis s’affiche en haut de la page Vue d’ensemble du lab. Cet avis est le titre de l’annonce qui a été spécifié lors de la création de l’annonce.

    ![Annonce du lab dans la page Vue d’ensemble](./media/devtest-lab-announcements/devtestlab-user-announcement.png)

1. L’utilisateur peut sélectionner le message pour afficher l’annonce entière.

    ![Informations supplémentaires pour l’annonce du lab](./media/devtest-lab-announcements/devtestlab-user-announcement-text.png)

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="azure-resource-manager-template"></a>Modèle Azure Resource Manager
Vous pouvez spécifier une annonce dans le cadre d’un modèle Azure Resource Manager comme indiqué dans l’exemple suivant :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "name": {
            "type": "string",
            "defaultValue": "devtestlabfromarm"
        },
        "regionId": {
            "type": "string",
            "defaultValue": "eastus"
        }
    },
    "resources": [
        {
            "apiVersion": "2017-04-26-preview",
            "name": "[parameters('name')]",
            "type": "Microsoft.DevTestLab/labs",
            "location": "[parameters('regionId')]",
            "tags": {},
            "properties": {
                "labStorageType": "Premium",
                "announcement":
                {
                    "title": "Important! Three images are currently inaccessible. Click for more information.",
                    "markdown":"# Images are inaccessible\n\nThe following 3 images are currently not available for use: \n\n- image1\n- image2\n- image3\n\nI am working to fix the problem ASAP.",
                    "enabled": "Enabled",
                    "expirationDate":"2018-12-31T06:00:00+00:00",
                    "expired": "false"
                },
                "support": {
                    "markdown": "",
                    "enabled": "Enabled"
                }
            },
            "resources": [
                {
                    "apiVersion": "2017-04-26-preview",
                    "name": "LabVmsShutdown",
                    "location": "[parameters('regionId')]",
                    "type": "schedules",
                    "dependsOn": [
                        "[resourceId('Microsoft.DevTestLab/labs', parameters('name'))]"
                    ],
                    "properties": {
                        "status": "Enabled",
                        "timeZoneId": "Eastern Standard Time",
                        "dailyRecurrence": {
                            "time": "1900"
                        },
                        "taskType": "LabVmsShutdownTask",
                        "notificationSettings": {
                            "status": "Disabled",
                            "timeInMinutes": 30
                        }
                    }
                },
                {
                    "apiVersion": "2017-04-26-preview",
                    "name": "[concat('Dtl', parameters('name'))]",
                    "type": "virtualNetworks",
                    "location": "[parameters('regionId')]",
                    "dependsOn": [
                        "[resourceId('Microsoft.DevTestLab/labs', parameters('name'))]"
                    ]
                }
            ]
        }
    ]
}
```

Vous pouvez déployer un modèle Azure Resource Manager à l’aide d’une des méthodes suivantes :

- [Azure portal](../azure-resource-manager/templates/deploy-portal.md)
- [Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md)
- [Azure CLI](../azure-resource-manager/templates/deploy-cli.md)
- [REST API](../azure-resource-manager/templates/deploy-rest.md)

## <a name="next-steps"></a>Étapes suivantes
* Si vous modifiez ou définissez une stratégie de lab, vous avez peut-être envie de publier une annonce pour en informer les utilisateurs. La page [Définir des stratégies et des planifications](devtest-lab-set-lab-policy.md) vous renseigne sur l’application de conventions et de restrictions sur votre abonnement à l’aide de stratégies personnalisées.
* Explorez la [Galerie de modèles de démarrage rapide d’Azure Resource Manager DevTest Labs](https://github.com/Azure/azure-devtestlab/tree/master/samples/DevTestLabs/QuickStartTemplates).
