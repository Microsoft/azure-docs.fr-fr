---
title: Résolution des problèmes liés à Azure Data Factory
description: Découvrez comment résoudre les problèmes liés à l'utilisation de Azure Data Factory.
services: data-factory
documentationcenter: ''
ms.assetid: 38fd14c1-5bb7-4eef-a9f5-b289ff9a6942
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
author: dcstwh
ms.author: weetok
ms.reviewer: maghan
manager: anandsub
robots: noindex
ms.openlocfilehash: bd18a26a1c199e1ecc32cfc371d2931b1dee0c3f
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96494968"
---
# <a name="troubleshoot-data-factory-issues"></a>Résolution des problèmes liés à Data Factory
> [!NOTE]
> Cet article s’applique à la version 1 d’Azure Data Factory. 

Cet article propose des conseils pour la résolution des problèmes d'utilisation d'Azure Data Factory. Cet article ne répertorie pas tous les problèmes possibles lors de l'utilisation du service, mais il aborde certains problèmes ainsi que des conseils de résolution généraux.   

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="troubleshooting-tips"></a>Conseils de dépannage
### <a name="error-the-subscription-is-not-registered-to-use-namespace-microsoftdatafactory"></a>Erreur : L’abonnement n’est pas inscrit pour utiliser l’espace de noms « Microsoft.DataFactory »
Si vous recevez cette erreur, cela signifie que le fournisseur de ressources Azure Data Factory n'a pas été enregistré sur votre ordinateur. Effectuez les actions suivantes :

1. Lancez Azure PowerShell.
2. Connectez-vous à votre compte Azure à l’aide de la commande suivante.

    ```powershell
    Connect-AzAccount
    ```
3. Exécutez la commande suivante pour enregistrer le fournisseur Azure Data Factory.

    ```powershell        
    Register-AzResourceProvider -ProviderNamespace Microsoft.DataFactory
    ```

### <a name="problem-unauthorized-error-when-running-a-data-factory-cmdlet"></a>Problème : Erreur non autorisée lors de l’exécution d’une applet de commande Data Factory
Vous n’utilisez probablement pas le compte ou l’abonnement Azure correct pour Azure PowerShell. Utilisez les applets de commande suivantes pour sélectionner le compte et l’abonnement Azure corrects à utiliser avec Azure PowerShell.

1. Connect-AzAccount : utilisez l’ID d’utilisateur et le mot de passe corrects
2. Get-AzSubscription : affichez tous les abonnements du compte.
3. Select-AzSubscription &lt;nom de l’abonnement&gt; - Sélectionnez l’abonnement correct. Utilisez le même que celui que vous utilisez pour créer une fabrique de données sur le portail Azure.

### <a name="problem-fail-to-launch-data-management-gateway-express-setup-from-azure-portal"></a>Problème : Échec du lancement de l’installation rapide de la passerelle de gestion des données à partir du portail Azure
L’installation rapide de la passerelle de gestion des données nécessite Internet Explorer ou un navigateur web compatible avec Microsoft ClickOnce. Si le programme d'installation rapide ne démarre pas, effectuez l'une des opérations suivantes :

* Utilisez Internet Explorer ou un navigateur web compatible Microsoft ClickOnce.

    Si vous utilisez Chrome, accédez au [Chrome Web Store](https://chrome.google.com/webstore/), faites une recherche sur le mot-clé « ClickOnce », choisissez l’une des extensions ClickOnce, puis installez-la.

    Faites de même pour Firefox (installez un complément). Cliquez sur le bouton du menu dans la barre d’outils (trois lignes horizontales en haut à droite), cliquez sur Modules complémentaires, effectuez une recherche avec le mot-clé « ClickOnce », choisissez l’une des extensions de ClickOnce et installez le programme.
* Utilisez le lien **Configuration manuelle** qui s’affiche dans le même panneau sur le portail. Cette approche vous permet de télécharger le fichier d’installation et de l’exécuter manuellement. Une fois l'installation effectuée, vous verrez s’afficher la boîte de dialogue Configuration de la passerelle de gestion des données. Copiez la **clé** sur l’écran du portail et utilisez-la dans le gestionnaire de configuration pour enregistrer manuellement la passerelle auprès du service.  

### <a name="problem-fail-to-connect-to-sql-server"></a>Problème : Échec de la connexion à SQL Server
Lancez le **Gestionnaire de configuration de la passerelle de gestion des données** sur l’ordinateur passerelle et utilisez l’onglet **Résolution des problèmes** pour tester la connexion à SQL Server à partir de l’ordinateur passerelle. Consultez [Résolution des problèmes de passerelle](data-factory-data-management-gateway.md#troubleshooting-gateway-issues) pour obtenir des conseils sur la résolution des problèmes de connexion/passerelle.   

### <a name="problem-input-slices-are-in-waiting-state-forever"></a>Problème : L’état des tranches d’entrée est En attente depuis longtemps
Les tranches peuvent avoir l’état **En attente** pour diverses raisons. Une des raisons courantes est que la propriété **external** n’est pas définie sur **true**. Tout jeu de données généré en dehors de l’étendue d’Azure Data Factory doit être marqué avec la propriété **external** . Cette propriété indique que les données sont externes et qu’elles ne sont prises en charge par aucun pipeline dans la fabrique de données. Les tranches de données sont marquées comme prêtes ( **Ready** ) une fois que les données sont disponibles dans le magasin respectif.

Consultez l’exemple suivant pour l’utilisation de la propriété **external** . Vous pouvez éventuellement spécifier **externalData** _ quand vous affectez à la propriété external la valeur true.

Consultez l’article [Jeux de données](data-factory-create-datasets.md) pour plus d’informations sur cette propriété.

```json
{
  "name": "CustomerTable",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "MyLinkedService",
    "typeProperties": {
      "folderPath": "MyContainer/MySubFolder/",
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ",",
        "rowDelimiter": ";"
      }
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      }
    }
  }
}
```

Pour résoudre l’erreur, ajoutez la propriété *external* et la section **externalData** facultative à la définition JSON de la table d’entrée, puis recréez la table.

### <a name="problem-hybrid-copy-operation-fails"></a>Problème : Échec de l’opération de copie hybride
Consultez la page [Résolution des problèmes de passerelle](data-factory-data-management-gateway.md#troubleshooting-gateway-issues) pour savoir comment résoudre les problèmes de copie depuis/vers un magasin de données local avec la passerelle de gestion des données.

### <a name="problem-on-demand-hdinsight-provisioning-fails"></a>Problème : Échec du provisionnement HDInsight à la demande
Lorsque vous utilisez un service lié de type HDInsightOnDemand, vous devez spécifier un linkedServiceName qui pointe vers un stockage d’objets blob Azure. Le service Data Factory utilise ce stockage pour stocker les journaux d’activité et les fichiers d’accompagnement pour votre cluster HDInsight à la demande.  Parfois, l’approvisionnement d'un cluster HDInsight à la demande échoue avec l'erreur suivante :

```
Failed to create cluster. Exception: Unable to complete the cluster create operation. Operation failed with code '400'. Cluster left behind state: 'Error'. Message: 'StorageAccountNotColocated'.
```

Cette erreur indique généralement que l’emplacement du compte de stockage spécifié dans linkedServiceName ne se trouve pas dans l’emplacement de centre de données dans lequel l’approvisionnement de HDInsight est effectué. Exemple : si votre fabrique de données se trouve dans la région USA Ouest et que le stockage Azure est dans la région USA Est, l’approvisionnement à la demande échoue dans USA Ouest.

En outre, il existe une seconde propriété JSON additionalLinkedServiceNames avec laquelle les comptes de stockage supplémentaires peuvent être spécifiés dans HDInsight à la demande. Ces comptes de stockage supplémentaires liés doivent avoir le même emplacement que le cluster HDInsight, ou l’approvisionnement échoue avec la même erreur.

### <a name="problem-custom-net-activity-fails"></a>Problème : Échec de l’activité .NET personnalisée
Consultez la page [Déboguer un pipeline avec une activité personnalisée](data-factory-use-custom-activities.md#troubleshoot-failures) pour obtenir des instructions détaillées.

## <a name="use-azure-portal-to-troubleshoot"></a>Utilisation du portail Azure pour résoudre les problèmes
### <a name="using-portal-blades"></a>Utilisation des panneaux du portail
Consultez la page [Surveiller le pipeline](data-factory-monitor-manage-pipelines.md) pour obtenir la procédure.

### <a name="using-monitor-and-manage-app"></a>Utilisation de l’application Surveillance et gestion
Consultez [Surveiller et gérer les pipelines Data Factory à l’aide de l’application Surveillance et gestion](data-factory-monitor-manage-app.md) pour plus d’informations.

## <a name="use-azure-powershell-to-troubleshoot"></a>Utiliser Azure PowerShell pour résoudre les problèmes
### <a name="use-azure-powershell-to-troubleshoot-an-error"></a>Utiliser Azure PowerShell pour résoudre une erreur
Consultez la page [Surveiller les pipelines Data Factory à l’aide d’Azure PowerShell](data-factory-monitor-manage-pipelines.md) pour plus d’informations.

[adfgetstarted]: data-factory-copy-data-from-azure-blob-storage-to-sql-database.md
[use-custom-activities]: data-factory-use-custom-activities.md
[troubleshoot]: data-factory-troubleshoot.md
[developer-reference]: /previous-versions/azure/dn834987(v=azure.100)
[cmdlet-reference]: https://go.microsoft.com/fwlink/?LinkId=517456
[json-scripting-reference]: /previous-versions/azure/dn835050(v=azure.100)

[azure-portal]: https://portal.azure.com/

[image-data-factory-troubleshoot-with-error-link]: ./media/data-factory-troubleshoot/DataFactoryWithErrorLink.png

[image-data-factory-troubleshoot-datasets-with-errors-blade]: ./media/data-factory-troubleshoot/DatasetsWithErrorsBlade.png

[image-data-factory-troubleshoot-table-blade-with-problem-slices]: ./media/data-factory-troubleshoot/TableBladeWithProblemSlices.png

[image-data-factory-troubleshoot-activity-run-with-error]: ./media/data-factory-troubleshoot/ActivityRunDetailsWithError.png

[image-data-factory-troubleshoot-dataslice-blade-with-active-runs]: ./media/data-factory-troubleshoot/DataSliceBladeWithActivityRuns.png

[image-data-factory-troubleshoot-walkthrough2-with-errors-link]: ./media/data-factory-troubleshoot/Walkthrough2WithErrorsLink.png

[image-data-factory-troubleshoot-walkthrough2-datasets-with-errors]: ./media/data-factory-troubleshoot/Walkthrough2DataSetsWithErrors.png

[image-data-factory-troubleshoot-walkthrough2-table-with-problem-slices]: ./media/data-factory-troubleshoot/Walkthrough2TableProblemSlices.png

[image-data-factory-troubleshoot-walkthrough2-slice-activity-runs]: ./media/data-factory-troubleshoot/Walkthrough2DataSliceActivityRuns.png

[image-data-factory-troubleshoot-activity-run-details]: ./media/data-factory-troubleshoot/Walkthrough2ActivityRunDetails.png