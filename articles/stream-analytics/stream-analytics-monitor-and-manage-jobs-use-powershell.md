---
title: Surveiller et gérer les travaux Azure Stream Analytics avec PowerShell
description: Cet article décrit comment utiliser Azure PowerShell et les applets de commande pour surveiller et gérer des travaux Azure Stream Analytics.
author: jseb225
ms.author: jeanb
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 03/28/2017
ms.openlocfilehash: 4d41e8231f5bd186408ea511c0f096a6a5f26c83
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93123640"
---
# <a name="monitor-and-manage-stream-analytics-jobs-with-azure-powershell-cmdlets"></a>Surveillance et gestion des travaux Stream Analytics à l’aide des applets de commande Azure PowerShell
Découvrez comment surveiller et gérer les ressources Stream Analytics à l’aide d’applets de commande Azure PowerShell et de scripts PowerShell qui exécutent les tâches Stream Analytics de base.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites-for-running-azure-powershell-cmdlets-for-stream-analytics"></a>Conditions requises pour l'exécution des applets de commande Azure PowerShell pour Stream Analytics
* Créez un groupe de ressources Azure dans votre abonnement. Voici un exemple de script Azure PowerShell : Pour obtenir des informations sur Azure PowerShell, consultez [Installation et configuration d’Azure PowerShell](/powershell/azure/).  

Azure PowerShell 0.9.8 :  

```powershell
# Log in to your Azure account
Add-AzureAccount
# Select the Azure subscription you want to use to create the resource group if you have more han one subscription on your account.
Select-AzureSubscription -SubscriptionName <subscription name>
# If Stream Analytics has not been registered to the subscription, remove remark symbol below (#)to run the Register-AzureProvider cmdlet to register the provider namespace.
#Register-AzureProvider -Force -ProviderNamespace 'Microsoft.StreamAnalytics'
# Create an Azure resource group
New-AzureResourceGroup -Name <YOUR RESOURCE GROUP NAME> -Location <LOCATION>
```

Azure PowerShell 1.0 :

```powershell
# Log in to your Azure account
Connect-AzAccount
# Select the Azure subscription you want to use to create the resource group.
Get-AzSubscription -SubscriptionName "your sub" | Select-AzSubscription
# If Stream Analytics has not been registered to the subscription, remove remark symbol below (#)to run the Register-AzureProvider cmdlet to register the provider namespace.
#Register-AzResourceProvider -Force -ProviderNamespace 'Microsoft.StreamAnalytics'
# Create an Azure resource group
New-AzResourceGroup -Name <YOUR RESOURCE GROUP NAME> -Location <LOCATION>
```


> [!NOTE]
> Par défaut, la surveillance n’est pas activée pour les travaux Stream Analytics créés par programme.  Vous pouvez activer manuellement la supervision dans le portail Azure. Pour cela, accédez à la page Superviser du travail et cliquez sur le bouton Activer. Vous pouvez également procéder par programmation en suivant les étapes décrites dans [Azure Stream Analytics - Superviser les travaux Stream Analytics par programmation](stream-analytics-monitor-jobs.md).
> 
> 

## <a name="azure-powershell-cmdlets-for-stream-analytics"></a>Applets de commande Azure PowerShell pour Stream Analytics
Vous pouvez utiliser les applets de commande Azure PowerShell suivantes pour surveiller et gérer des travaux Azure Stream Analytics. Notez qu'il existe différentes versions d'Azure PowerShell. 
**Dans les exemples répertoriés, la première commande s'applique à Azure PowerShell 0.9.8, la deuxième commande s'applique à Azure PowerShell 1.0.** Les commandes Azure PowerShell 1.0 contiennent toujours « Az ».

### <a name="get-azurestreamanalyticsjob--get-azstreamanalyticsjob"></a>Get-AzureStreamAnalyticsJob | Get-AzStreamAnalyticsJob
Répertorie tous les travaux Stream Analytics définis dans l’abonnement Azure ou le groupe de ressources spécifié, ou obtient des informations sur un travail spécifique au sein d’un groupe de ressources.

**Exemple 1**

Azure PowerShell 0.9.8 :  

```powershell
Get-AzureStreamAnalyticsJob
```

Azure PowerShell 1.0 :  

```powershell
Get-AzStreamAnalyticsJob
```

Cette commande PowerShell retourne des informations sur tous les travaux Stream Analytics dans l’abonnement Azure.

**Exemple 2**

Azure PowerShell 0.9.8 :  

```powershell
Get-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US 
```

Azure PowerShell 1.0 :  

```powershell
Get-AzStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US 
```

Cette commande PowerShell retourne des informations sur tous les travaux Stream Analytics dans le groupe de ressources StreamAnalytics-Default-Central-US.

**Exemple 3**

Azure PowerShell 0.9.8 :  

```powershell
Get-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob
```

Azure PowerShell 1.0 :  

```powershell
Get-AzStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob
```

Cette commande PowerShell retourne des informations sur le travail Stream Analytics StreamingJob dans le groupe de ressources StreamAnalytics-Default-Central-US.

### <a name="get-azurestreamanalyticsinput--get-azstreamanalyticsinput"></a>Get-AzureStreamAnalyticsInput | Get-AzStreamAnalyticsInput
Répertorie toutes les entrées qui sont définies dans un travail Stream Analytics spécifié ou obtient des informations sur une entrée spécifique.

**Exemple 1**

Azure PowerShell 0.9.8 :  

```powershell
Get-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob
```

Azure PowerShell 1.0 :  

```powershell
Get-AzStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob
```

Cette commande PowerShell retourne des informations sur toutes les entrées définies dans le travail StreamingJob.

**Exemple 2**

Azure PowerShell 0.9.8 :  

```powershell
Get-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name EntryStream
```

Azure PowerShell 1.0 :  

```powershell
Get-AzStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name EntryStream
```

Cette commande PowerShell retourne des informations sur l’entrée nommée EntryStream définie dans le travail StreamingJob.

### <a name="get-azurestreamanalyticsoutput--get-azstreamanalyticsoutput"></a>Get-AzureStreamAnalyticsOutput | Get-AzStreamAnalyticsOutput
Répertorie toutes les sorties qui sont définies dans un travail Stream Analytics spécifié ou obtient des informations sur une sortie spécifique.

**Exemple 1**

Azure PowerShell 0.9.8 :  

```powershell
Get-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob
```

Azure PowerShell 1.0 :  

```powershell
Get-AzStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob
```

Cette commande PowerShell retourne des informations sur les sorties définies dans le travail StreamingJob.

**Exemple 2**

Azure PowerShell 0.9.8 :  

```powershell
Get-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name Output
```

Azure PowerShell 1.0 :  

```powershell
Get-AzStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name Output
```

Cette commande PowerShell retourne des informations sur la sortie nommée Output définie dans le travail StreamingJob.

### <a name="get-azurestreamanalyticsquota--get-azstreamanalyticsquota"></a>Get-AzureStreamAnalyticsQuota | Get-AzStreamAnalyticsQuota
Obtient des informations sur le quota des unités de diffusion en continu d'une région spécifiée.

**Exemple 1**

Azure PowerShell 0.9.8 :  

```powershell
Get-AzureStreamAnalyticsQuota -Location "Central US" 
```

Azure PowerShell 1.0 :  

```powershell
Get-AzStreamAnalyticsQuota -Location "Central US" 
```

Cette commande PowerShell retourne des informations sur le quota et l’utilisation des unités de diffusion en continu dans la région USA Centre.

### <a name="get-azurestreamanalyticstransformation--get-azstreamanalyticstransformation"></a>Get-AzureStreamAnalyticsTransformation | Get-AzStreamAnalyticsTransformation
Obtient des informations sur une transformation spécifique définie dans un travail Stream Analytics.

**Exemple 1**

Azure PowerShell 0.9.8 :  

```powershell
Get-AzureStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name StreamingJob
```

Azure PowerShell 1.0 :  

```powershell
Get-AzStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name StreamingJob
```

Cette commande PowerShell retourne des informations sur la transformation nommée StreamingJob dans le travail StreamingJob.

### <a name="new-azurestreamanalyticsinput--new-azstreamanalyticsinput"></a>New-AzureStreamAnalyticsInput | New-AzStreamAnalyticsInput
Crée une entrée dans un travail Stream Analytics ou met à jour une entrée spécifiée existante.

Vous pouvez spécifier le nom de l'entrée dans le fichier .json ou sur la ligne de commande. Si vous spécifiez les deux, le nom indiqué sur la ligne de commande doit être identique à celui contenu dans le fichier.

Si vous spécifiez une entrée qui existe déjà et que vous ne spécifiez pas le paramètre -Force, l’applet de commande vous demande s’il faut remplacer l’entrée existante.

Si vous spécifiez le paramètre -Force et un nom d’entrée existant, l’entrée est remplacée sans confirmation.

Pour plus d’informations sur la structure et le contenu du fichier JSON, reportez-vous à la section [Créer une entrée (Azure Stream Analytics)][msdn-rest-api-create-stream-analytics-input] de la [bibliothèque de référence des API REST de gestion de Stream Analytics][stream.analytics.rest.api.reference].

**Exemple 1**

Azure PowerShell 0.9.8 :  

```powershell
New-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -File "C:\Input.json" 
```

Azure PowerShell 1.0 :  

```powershell
New-AzStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -File "C:\Input.json" 
```

Cette commande PowerShell crée une entrée à partir du fichier Input.json. Si une entrée existante avec le nom spécifié dans le fichier de définition d'entrée est déjà définie, l'applet de commande vous demande s'il faut la remplacer.

**Exemple 2**

Azure PowerShell 0.9.8 :  

```powershell
New-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -File "C:\Input.json" -Name EntryStream
```

Azure PowerShell 1.0 :  

```powershell
New-AzStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -File "C:\Input.json" -Name EntryStream
```

Cette commande PowerShell crée une entrée dans le travail nommé EntryStream. Si une entrée existante portant ce nom est déjà définie, l'applet de commande vous demande s'il faut la remplacer.

**Exemple 3**

Azure PowerShell 0.9.8 :  

```powershell
New-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -File "C:\Input.json" -Name EntryStream -Force
```

Azure PowerShell 1.0 :  

```powershell
New-AzStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -File "C:\Input.json" -Name EntryStream -Force
```

Cette commande PowerShell remplace la définition de la source d’entrée existante nommée EntryStream par la définition qui se trouve dans le fichier.

### <a name="new-azurestreamanalyticsjob--new-azstreamanalyticsjob"></a>New-AzureStreamAnalyticsJob | New-AzStreamAnalyticsJob
Crée un travail Stream Analytics dans Microsoft Azure ou met à jour la définition d'un travail existant spécifié.

Vous pouvez spécifier le nom du travail dans le fichier .json ou sur la ligne de commande. Si vous spécifiez les deux, le nom indiqué sur la ligne de commande doit être identique à celui contenu dans le fichier.

Si vous spécifiez un nom de travail qui existe déjà et que vous ne spécifiez pas le paramètre -Force, l’applet de commande vous demande s’il faut remplacer le travail existant.

Si vous spécifiez le paramètre -Force et un nom de travail existant, la définition de travail est remplacée sans confirmation.

Pour plus d’informations sur la structure et le contenu du fichier JSON, reportez-vous à la section [Créer un travail Stream Analytics][msdn-rest-api-create-stream-analytics-job] de la [bibliothèque de référence des API REST de gestion de Stream Analytics][stream.analytics.rest.api.reference].

**Exemple 1**

Azure PowerShell 0.9.8 :  

```powershell
New-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\JobDefinition.json" 
```

Azure PowerShell 1.0 :  

```powershell
New-AzStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\JobDefinition.json" 
```

Cette commande PowerShell crée un travail à partir de la définition qui se trouve dans JobDefinition.json. Si un travail existant avec le nom spécifié dans le fichier de définition de travail est déjà défini, l'applet de commande vous demande s'il faut le remplacer.

**Exemple 2**

Azure PowerShell 0.9.8 :  

```powershell
New-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\JobDefinition.json" -Name StreamingJob -Force
```

Azure PowerShell 1.0 :  

```powershell
New-AzStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\JobDefinition.json" -Name StreamingJob -Force
```

Cette commande PowerShell remplace la définition du travail StreamingJob.

### <a name="new-azurestreamanalyticsoutput--new-azstreamanalyticsoutput"></a>New-AzureStreamAnalyticsOutput | New-AzStreamAnalyticsOutput
Crée une sortie dans un travail Stream Analytics ou met à jour une sortie existante.  

Vous pouvez spécifier le nom de la sortie dans le fichier .json ou sur la ligne de commande. Si vous spécifiez les deux, le nom indiqué sur la ligne de commande doit être identique à celui contenu dans le fichier.

Si vous spécifiez une sortie qui existe déjà et que vous ne spécifiez pas le paramètre -Force, l’applet de commande vous demande s’il faut remplacer la sortie existante.

Si vous spécifiez le paramètre -Force et un nom de sortie existant, la sortie est remplacée sans confirmation.

Pour plus d’informations sur la structure et le contenu du fichier JSON, reportez-vous à la section [Créer une sortie (Azure Stream Analytics)][msdn-rest-api-create-stream-analytics-output] de la [bibliothèque de référence des API REST de gestion de Stream Analytics][stream.analytics.rest.api.reference].

**Exemple 1**

Azure PowerShell 0.9.8 :  

```powershell
New-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\Output.json" -JobName StreamingJob -Name output
```

Azure PowerShell 1.0 :  

```powershell
New-AzStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\Output.json" -JobName StreamingJob -Name output
```

Cette commande PowerShell crée une sortie nommée « output » dans le travail StreamingJob. Si une sortie existante portant ce nom est déjà définie, l'applet de commande vous demande s'il faut la remplacer.

**Exemple 2**

Azure PowerShell 0.9.8 :  

```powershell
New-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\Output.json" -JobName StreamingJob -Name output -Force
```

Azure PowerShell 1.0 :  

```powershell
New-AzStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\Output.json" -JobName StreamingJob -Name output -Force
```

Cette commande PowerShell remplace la définition de la sortie « output » dans le travail StreamingJob.

### <a name="new-azurestreamanalyticstransformation--new-azstreamanalyticstransformation"></a>New-AzureStreamAnalyticsTransformation | New-AzStreamAnalyticsTransformation
Crée une transformation dans un travail Stream Analytics ou met à jour la transformation existante.

Vous pouvez spécifier le nom de la transformation dans le fichier .json ou sur la ligne de commande. Si vous spécifiez les deux, le nom indiqué sur la ligne de commande doit être identique à celui contenu dans le fichier.

Si vous spécifiez une transformation qui existe déjà et que vous ne spécifiez pas le paramètre -Force, l’applet de commande vous demande s’il faut remplacer la transformation existante.

Si vous spécifiez le paramètre -Force et un nom de transformation existant, la transformation est remplacée sans confirmation.

Pour plus d’informations sur la structure et le contenu du fichier JSON, reportez-vous à la section [Créer une transformation (Azure Stream Analytics)][msdn-rest-api-create-stream-analytics-transformation] de la [bibliothèque de référence des API REST de gestion de Stream Analytics][stream.analytics.rest.api.reference].

**Exemple 1**

Azure PowerShell 0.9.8 :  

```powershell
New-AzureStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\Transformation.json" -JobName StreamingJob -Name StreamingJobTransform
```

Azure PowerShell 1.0 :  

```powershell
New-AzStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\Transformation.json" -JobName StreamingJob -Name StreamingJobTransform
```

Cette commande PowerShell crée une transformation nommée StreamingJobTransform dans le travail StreamingJob. Si une transformation existante est déjà définie avec ce nom, l'applet de commande vous demande s'il faut la remplacer.

**Exemple 2**

Azure PowerShell 0.9.8 :  

```powershell
New-AzureStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\Transformation.json" -JobName StreamingJob -Name StreamingJobTransform -Force
```

Azure PowerShell 1.0 :  

```powershell
New-AzStreamAnalyticsTransformation -ResourceGroupName StreamAnalytics-Default-Central-US -File "C:\Transformation.json" -JobName StreamingJob -Name StreamingJobTransform -Force
```

 Cette commande PowerShell remplace la définition de StreamingJobTransform dans le travail StreamingJob.

### <a name="remove-azurestreamanalyticsinput--remove-azstreamanalyticsinput"></a>Remove-AzureStreamAnalyticsInput | Remove-AzStreamAnalyticsInput
Supprime de manière asynchrone une entrée spécifique d'un travail Stream Analytics dans Microsoft Azure.  
Si vous spécifiez le paramètre -Force, l’entrée est supprimée sans confirmation.

**Exemple 1**

Azure PowerShell 0.9.8 :  

```powershell
Remove-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name EventStream
```

Azure PowerShell 1.0 :  

```powershell
Remove-AzStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name EventStream
```

Cette commande PowerShell supprime l’entrée EventStream dans le travail StreamingJob.  

### <a name="remove-azurestreamanalyticsjob--remove-azstreamanalyticsjob"></a>Remove-AzureStreamAnalyticsJob | Remove-AzStreamAnalyticsJob
Supprime de manière asynchrone un travail Stream Analytics spécifique dans Microsoft Azure.  
Si vous spécifiez le paramètre -Force, le travail est supprimé sans confirmation.

**Exemple 1**

Azure PowerShell 0.9.8 :  

```powershell
Remove-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob 
```

Azure PowerShell 1.0 :  

```powershell
Remove-AzStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob 
```

Cette commande PowerShell supprime le travail StreamingJob.  

### <a name="remove-azurestreamanalyticsoutput--remove-azstreamanalyticsoutput"></a>Remove-AzureStreamAnalyticsOutput | Remove-AzStreamAnalyticsOutput
Supprime de manière asynchrone une sortie spécifique d'un travail Stream Analytics dans Microsoft Azure.  
Si vous spécifiez le paramètre -Force, la sortie est supprimée sans confirmation.

**Exemple 1**

Azure PowerShell 0.9.8 :  

```powershell
Remove-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name Output
```

Azure PowerShell 1.0 :  

```powershell
Remove-AzStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name Output
```

Cette commande PowerShell supprime la sortie Output dans le travail StreamingJob.  

### <a name="start-azurestreamanalyticsjob--start-azstreamanalyticsjob"></a>Start-AzureStreamAnalyticsJob | Start-AzStreamAnalyticsJob
Déploie et démarre un travail Stream Analytics dans Microsoft Azure de façon asynchrone.

**Exemple 1**

Azure PowerShell 0.9.8 :  

```powershell
Start-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob -OutputStartMode CustomTime -OutputStartTime 2012-12-12T12:12:12Z
```

Azure PowerShell 1.0 :  

```powershell
Start-AzStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob -OutputStartMode CustomTime -OutputStartTime 2012-12-12T12:12:12Z
```

Cette commande PowerShell démarre le travail StreamingJob avec une heure de début de sortie personnalisée définie sur le 12 décembre 2012, 12:12:12 UTC.

### <a name="stop-azurestreamanalyticsjob--stop-azstreamanalyticsjob"></a>Stop-AzureStreamAnalyticsJob | Stop-AzStreamAnalyticsJob
Arrête l'exécution d'un travail Stream Analytics dans Microsoft Azure de façon asynchrone et libère les ressources qui étaient utilisées. La définition du travail et les métadonnées restent disponibles dans votre abonnement par le biais du Portail Azure et des API de gestion ; ainsi, le travail peut être modifié et redémarré. Un travail à l'état Arrêté ne vous sera pas facturé.

**Exemple 1**

Azure PowerShell 0.9.8 :  

```powershell
Stop-AzureStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob 
```

Azure PowerShell 1.0 :  

```powershell
Stop-AzStreamAnalyticsJob -ResourceGroupName StreamAnalytics-Default-Central-US -Name StreamingJob 
```

Cette commande PowerShell arrête le travail StreamingJob.  

### <a name="test-azurestreamanalyticsinput--test-azstreamanalyticsinput"></a>Test-AzureStreamAnalyticsInput | Test-AzStreamAnalyticsInput
Teste la capacité de Stream Analytics à se connecter à une entrée spécifiée.

**Exemple 1**

Azure PowerShell 0.9.8 :  

```powershell
Test-AzureStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name EntryStream
```

Azure PowerShell 1.0 :  

```powershell
Test-AzStreamAnalyticsInput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name EntryStream
```

Cette commande PowerShell teste l’état de la connexion de l’entrée EntryStream dans StreamingJob.  

### <a name="test-azurestreamanalyticsoutput--test-azstreamanalyticsoutput"></a>Test-AzureStreamAnalyticsOutput | Test-AzStreamAnalyticsOutput
Teste la capacité de Stream Analytics à se connecter à une sortie spécifiée.

**Exemple 1**

Azure PowerShell 0.9.8 :  

```powershell
Test-AzureStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name Output
```

Azure PowerShell 1.0 :  

```powershell
Test-AzStreamAnalyticsOutput -ResourceGroupName StreamAnalytics-Default-Central-US -JobName StreamingJob -Name Output
```

Cette commande PowerShell teste l’état de la connexion de la sortie Output dans StreamingJob.  

## <a name="get-support"></a>Obtenir de l’aide
Pour obtenir de l’aide supplémentaire, consultez notre [page de questions Microsoft Q&A pour Azure Stream Analytics](/answers/topics/azure-stream-analytics.html). 

## <a name="next-steps"></a>Étapes suivantes
* [Présentation d’Azure Stream Analytics](stream-analytics-introduction.md)
* [Prise en main d'Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Mise à l’échelle des travaux Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Références sur le langage des requêtes d'Azure Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference)
* [Références sur l’API REST de gestion d’Azure Stream Analytics](/rest/api/streamanalytics/)

[msdn-switch-azuremode]: /previous-versions/azure/dn722470(v=azure.100)
[powershell-install]: /powershell/azure/
[msdn-rest-api-create-stream-analytics-job]: ./stream-analytics-quick-create-portal.md
[msdn-rest-api-create-stream-analytics-input]: ./stream-analytics-define-inputs.md
[msdn-rest-api-create-stream-analytics-output]: ./stream-analytics-define-outputs.md
[msdn-rest-api-create-stream-analytics-transformation]: /cli/azure/ext/stream-analytics/stream-analytics/transformation

[stream.analytics.introduction]: stream-analytics-introduction.md
[stream.analytics.get.started]: stream-analytics-real-time-fraud-detection.md
[stream.analytics.developer.guide]: ../stream-analytics-developer-guide.md
[stream.analytics.scale.jobs]: stream-analytics-scale-jobs.md
[stream.analytics.query.language.reference]: /stream-analytics-query/stream-analytics-query-language-reference
[stream.analytics.rest.api.reference]: /rest/api/streamanalytics/