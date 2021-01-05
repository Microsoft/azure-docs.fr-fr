---
title: Gérer Azure Data Lake Analytics à l’aide de Python
description: Cet article explique comment utiliser Python pour gérer des comptes, des sources de données, des utilisateurs et des travaux Data Lake Analytics.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/08/2018
ms.custom: devx-track-python
ms.openlocfilehash: fda09047d138e404eeca87ed3eba9bb72ff62d56
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92220225"
---
# <a name="manage-azure-data-lake-analytics-using-python"></a>Gérer Azure Data Lake Analytics à l’aide de Python
[!INCLUDE [manage-selector](../../includes/data-lake-analytics-selector-manage.md)]

Cet article explique comment gérer des comptes, des sources de données, des utilisateurs et des travaux Azure Data Lake Analytics avec Python.

## <a name="supported-python-versions"></a>Versions de Python prises en charge

* Utilisez la version 64 bits de Python.
* Vous pouvez utiliser la distribution Python standard, disponible dans les **[téléchargements Python.org](https://www.python.org/downloads/)** . 
* De nombreux développeurs trouvent pratique d’utiliser la **[distribution Anaconda Python](https://www.anaconda.com/download/)** .  
* Cet article a été écrit à l’aide de Python version 3.6 à partir de la distribution Python standard

## <a name="install-azure-python-sdk"></a>Installer le Kit de développement logiciel (SDK) Python

Installez les modules suivants :

* Le module **azure-mgmt-resource** inclut d’autres modules Azure pour Active Directory, etc.
* Le module **azure-datalake-store** inclut les opérations du système de fichiers d’Azure Data Lake Store. 
* Le module **azure-mgmt-datalake-store** inclut les opérations de gestion du compte Azure Data Lake Store.
* Le module **azure-mgmt-datalake-analytics** comporte les opérations d’Azure Data Lake Analytics. 

Tout d’abord, vérifiez que vous disposez des `pip` les plus récents en exécutant la commande suivante :

```console
python -m pip install --upgrade pip
```

Ce document a été écrit à l’aide de `pip version 9.0.1`.

Utilisez les commandes `pip` suivantes pour installer les modules à partir de la ligne de commande :

```console
pip install azure-mgmt-resource
pip install azure-datalake-store
pip install azure-mgmt-datalake-store
pip install azure-mgmt-datalake-analytics
```

## <a name="create-a-new-python-script"></a>Créer un script Python

Collez le code suivant dans le script :

```python
# Use this only for Azure AD service-to-service authentication
#from azure.common.credentials import ServicePrincipalCredentials

# Use this only for Azure AD end-user authentication
#from azure.common.credentials import UserPassCredentials

# Required for Azure Resource Manager
from azure.mgmt.resource.resources import ResourceManagementClient
from azure.mgmt.resource.resources.models import ResourceGroup

# Required for Azure Data Lake Store account management
from azure.mgmt.datalake.store import DataLakeStoreAccountManagementClient
from azure.mgmt.datalake.store.models import DataLakeStoreAccount

# Required for Azure Data Lake Store filesystem management
from azure.datalake.store import core, lib, multithread

# Required for Azure Data Lake Analytics account management
from azure.mgmt.datalake.analytics.account import DataLakeAnalyticsAccountManagementClient
from azure.mgmt.datalake.analytics.account.models import DataLakeAnalyticsAccount, DataLakeStoreAccountInformation

# Required for Azure Data Lake Analytics job management
from azure.mgmt.datalake.analytics.job import DataLakeAnalyticsJobManagementClient
from azure.mgmt.datalake.analytics.job.models import JobInformation, JobState, USqlJobProperties

# Required for Azure Data Lake Analytics catalog management
from azure.mgmt.datalake.analytics.catalog import DataLakeAnalyticsCatalogManagementClient

# Use these as needed for your application
import logging
import getpass
import pprint
import uuid
import time
```

Exécutez ce script pour vérifier que les modules peuvent être importés.

## <a name="authentication"></a>Authentication

### <a name="interactive-user-authentication-with-a-pop-up"></a>Authentification utilisateur interactif avec une fenêtre contextuelle

Cette méthode n’est pas prise en charge.

### <a name="interactive-user-authentication-with-a-device-code"></a>Authentification utilisateur interactif avec un code d’appareil

```python
user = input(
    'Enter the user to authenticate with that has permission to subscription: ')
password = getpass.getpass()
credentials = UserPassCredentials(user, password)
```

### <a name="noninteractive-authentication-with-spi-and-a-secret"></a>Authentification non interactive avec SPI et un secret

```python
credentials = ServicePrincipalCredentials(
    client_id='FILL-IN-HERE', secret='FILL-IN-HERE', tenant='FILL-IN-HERE')
```

### <a name="noninteractive-authentication-with-api-and-a-certificate"></a>Authentification non interactive avec API et un certificat

Cette méthode n’est pas prise en charge.

## <a name="common-script-variables"></a>Variables de script courantes

Ces variables sont utilisées dans les exemples.

```python
subid = '<Azure Subscription ID>'
rg = '<Azure Resource Group Name>'
location = '<Location>'  # i.e. 'eastus2'
adls = '<Azure Data Lake Store Account Name>'
adla = '<Azure Data Lake Analytics Account Name>'
```

## <a name="create-the-clients"></a>Créer les clients

```python
resourceClient = ResourceManagementClient(credentials, subid)
adlaAcctClient = DataLakeAnalyticsAccountManagementClient(credentials, subid)
adlaJobClient = DataLakeAnalyticsJobManagementClient(
    credentials, 'azuredatalakeanalytics.net')
```

## <a name="create-an-azure-resource-group"></a>Créer un groupe de ressources Azure

```python
armGroupResult = resourceClient.resource_groups.create_or_update(
    rg, ResourceGroup(location=location))
```

## <a name="create-data-lake-analytics-account"></a>Créer un compte Analytique Data Lake

Commencez par créer un compte Store.

```python
adlsAcctResult = adlsAcctClient.account.create(
    rg,
    adls,
    DataLakeStoreAccount(
        location=location)
    )
).wait()
```
Puis créez un compte ADLA qui utilise ce Store.

```python
adlaAcctResult = adlaAcctClient.account.create(
    rg,
    adla,
    DataLakeAnalyticsAccount(
        location=location,
        default_data_lake_store_account=adls,
        data_lake_store_accounts=[DataLakeStoreAccountInformation(name=adls)]
    )
).wait()
```

## <a name="submit-a-job"></a>Soumettre un travail

```python
script = """
@a  = 
    SELECT * FROM 
        (VALUES
            ("Contoso", 1500.0),
            ("Woodgrove", 2700.0)
        ) AS 
              D( customer, amount );
OUTPUT @a
    TO "/data.csv"
    USING Outputters.Csv();
"""

jobId = str(uuid.uuid4())
jobResult = adlaJobClient.job.create(
    adla,
    jobId,
    JobInformation(
        name='Sample Job',
        type='USql',
        properties=USqlJobProperties(script=script)
    )
)
```

## <a name="wait-for-a-job-to-end"></a>Attendre la fin d’un travail

```python
jobResult = adlaJobClient.job.get(adla, jobId)
while(jobResult.state != JobState.ended):
    print('Job is not yet done, waiting for 3 seconds. Current state: ' +
          jobResult.state.value)
    time.sleep(3)
    jobResult = adlaJobClient.job.get(adla, jobId)

print('Job finished with result: ' + jobResult.result.value)
```

## <a name="list-pipelines-and-recurrences"></a>Lister les pipelines et les récurrences
Si des métadonnées de pipeline ou de récurrence sont jointes à vos travaux, vous pouvez lister les pipelines et les récurrences.

```python
pipelines = adlaJobClient.pipeline.list(adla)
for p in pipelines:
    print('Pipeline: ' + p.name + ' ' + p.pipelineId)

recurrences = adlaJobClient.recurrence.list(adla)
for r in recurrences:
    print('Recurrence: ' + r.name + ' ' + r.recurrenceId)
```

## <a name="manage-compute-policies"></a>Gérer les stratégies de calcul

L’objet DataLakeAnalyticsAccountManagementClient fournit des méthodes de gestion des stratégies de calcul pour un compte Data Lake Analytics.

### <a name="list-compute-policies"></a>Lister les stratégies de calcul

Le code suivant récupère une liste de stratégies de calcul pour un compte Data Lake Analytics.

```python
policies = adlaAccountClient.computePolicies.listByAccount(rg, adla)
for p in policies:
    print('Name: ' + p.name + 'Type: ' + p.objectType + 'Max AUs / job: ' +
          p.maxDegreeOfParallelismPerJob + 'Min priority / job: ' + p.minPriorityPerJob)
```

### <a name="create-a-new-compute-policy"></a>Créer une nouvelle stratégie de calcul

Le code suivant crée une nouvelle stratégie de calcul pour un compte Data Lake Analytics, en définissant les unités Analytics maximales disponibles pour l’utilisateur spécifié sur 50 et la priorité minimale du travail sur 250.

```python
userAadObjectId = "3b097601-4912-4d41-b9d2-78672fc2acde"
newPolicyParams = ComputePolicyCreateOrUpdateParameters(
    userAadObjectId, "User", 50, 250)
adlaAccountClient.computePolicies.createOrUpdate(
    rg, adla, "GaryMcDaniel", newPolicyParams)
```

## <a name="next-steps"></a>Étapes suivantes

- Pour afficher le même didacticiel en utilisant d’autres outils, cliquez sur les sélecteurs d’onglet en haut de la page.
- Pour connaître U-SQL, voir [Prise en main du langage U-SQL d’Analytique Data Lake Azure](data-lake-analytics-u-sql-get-started.md).
- Pour les tâches de gestion, consultez [Gestion d’Azure Data Lake Analytics à l’aide du portail Azure](data-lake-analytics-manage-use-portal.md).

