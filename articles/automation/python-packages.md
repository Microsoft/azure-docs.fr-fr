---
title: Gérer des packages Python 2 dans Azure Automation
description: Cet article explique comment gérer des packages Python 2 dans Azure Automation.
services: automation
ms.subservice: process-automation
ms.date: 02/25/2019
ms.topic: conceptual
ms.custom: devx-track-python
ms.openlocfilehash: 1ab0b2def1a22470c1d0b6339e1525cd683b4a0b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90987571"
---
# <a name="manage-python-2-packages-in-azure-automation"></a>Gérer des packages Python 2 dans Azure Automation

Azure Automation vous permet d’exécuter des runbooks Python 2 sur Azure et sur les Runbooks Worker hybrides Linux. Pour aider à simplifier les runbooks, vous pouvez utiliser des packages Python afin d’importer les modules dont vous avez besoin. Cet article décrit comment gérer et utiliser les packages Python dans Azure Automation.

## <a name="import-packages"></a>Importer des packages

Dans votre compte Automation, sélectionnez **Packages Python 2** sous **Ressources partagées**. Cliquez sur **+ Add a Python 2 package** (+ Ajouter un package Python 2).

:::image type="content" source="media/python-packages/add-python-package.png" alt-text="Capture d’écran de la page des packages Python 2 qui montre les packages Python 2 dans le menu de gauche et Ajouter un package Python 2 mis en évidence.":::

Sur la page Ajouter un package Python 2, sélectionnez un package local à charger. Le package peut être un fichier **.whl** ou **.tar.gz**. Lorsque le package est sélectionné, cliquez sur **OK** pour le charger.

:::image type="content" source="media/python-packages/upload-package.png" alt-text="Capture d’écran de la page des packages Python 2 qui montre les packages Python 2 dans le menu de gauche et Ajouter un package Python 2 mis en évidence.":::

Une fois qu’un package a été importé, il est listé dans la page Packages Python 2 de votre compte Automation. Pour supprimer un package, intervenez sur sa page, sélectionnez-le, puis cliquez sur **Supprimer**.

:::image type="content" source="media/python-packages/package-list.png" alt-text="Capture d’écran de la page des packages Python 2 qui montre les packages Python 2 dans le menu de gauche et Ajouter un package Python 2 mis en évidence.":::

## <a name="import-packages-with-dependencies"></a>Importer des packages avec des dépendances

Azure Automation ne résout pas les dépendances de packages Python pendant le processus d’importation. Il existe deux méthodes pour importer un package avec toutes ses dépendances. Seule l’une des étapes suivantes doit être utilisée pour importer les packages dans votre compte Automation.

### <a name="manually-download"></a>Téléchargement manuel

Sur un ordinateur Windows 64 bits où sont installés [Python 2.7](https://www.python.org/downloads/release/latest/python2) et [pip](https://pip.pypa.io/en/stable/), exécutez la commande suivante pour télécharger un package et toutes ses dépendances :

```cmd
C:\Python27\Scripts\pip2.7.exe download -d <output dir> <package name>
```

Une fois les packages téléchargés, vous pouvez les importer dans votre compte Automation.

### <a name="runbook"></a>Runbook

 Pour obtenir un runbook, [importez les packages Python 2 de pypi dans le compte Azure Automation](https://gallery.technet.microsoft.com/scriptcenter/Import-Python-2-packages-57f7d509) dans votre compte Automation à partir de la galerie. Vérifiez que les paramètres d’exécution sont définis sur **Azure**, puis démarrez le runbook avec les paramètres. Pour que le compte Automation fonctionne, le runbook a besoin d’un compte d’identification. Veillez à démarrer chaque paramètre avec le commutateur, comme indiqué dans la liste et la capture d’écran suivantes :

* -s \<subscriptionId\>
* -g \<resourceGroup\>
* -a \<automationAccount\>
* -m \<modulePackage\>

:::image type="content" source="media/python-packages/import-python-runbook.png" alt-text="Capture d’écran de la page des packages Python 2 qui montre les packages Python 2 dans le menu de gauche et Ajouter un package Python 2 mis en évidence.":::

Le runbook vous permet de spécifier le package à télécharger. Par exemple, l’utilisation du paramètre `Azure` permet de télécharger tous les modules Azure et toutes les dépendances (environ 105).

Une fois le runbook terminé, vous pouvez consulter les **packages Python 2** sous **Ressources partagées** dans votre compte Automation pour vérifier que le package a bien été importé.

## <a name="use-a-package-in-a-runbook"></a>Utiliser un package dans un runbook

Vous pouvez utiliser un package importé dans un runbook. L’exemple suivant utilise le [package de l’utilitaire Azure Automation](https://github.com/azureautomation/azure_automation_utility). Ce package facilite l’utilisation de Python avec Azure Automation. Pour utiliser le package, suivez les instructions du référentiel GitHub et ajoutez-le au runbook. Par exemple, vous pouvez utiliser `from azure_automation_utility import get_automation_runas_credential` pour importer la fonction de récupération du compte d’identification.

```python
import azure.mgmt.resource
import automationassets
from azure_automation_utility import get_automation_runas_credential

# Authenticate to Azure using the Azure Automation RunAs service principal
runas_connection = automationassets.get_automation_connection("AzureRunAsConnection")
azure_credential = get_automation_runas_credential()

# Intialize the resource management client with the RunAs credential and subscription
resource_client = azure.mgmt.resource.ResourceManagementClient(
    azure_credential,
    str(runas_connection["SubscriptionId"]))

# Get list of resource groups and print them out
groups = resource_client.resource_groups.list()
for group in groups:
    print group.name
```

## <a name="develop-and-test-runbooks-offline"></a>Développer et tester les runbooks en mode hors connexion

Pour développer et tester vos runbooks Python 2 en mode hors connexion, utilisez par exemple le module [Azure Automation python emulated assets](https://github.com/azureautomation/python_emulated_assets) (Ressources émulées Python dans Azure Automation) sur GitHub. Ce module vous permet de référencer vos ressources partagées telles que les informations d’identification, variables, connexions et certificats.

## <a name="next-steps"></a>Étapes suivantes

Pour préparer un runbook Python, consultez [Créer un runbook Python](learn/automation-tutorial-runbook-textual-python2.md).
