---
title: Gérer des bibliothèques pour Apache Spark
description: Découvrez comment ajouter et gérer des bibliothèques utilisées par Apache Spark dans Azure Synapse Analytics.
services: synapse-analytics
author: euangMS
ms.service: synapse-analytics
ms.topic: conceptual
ms.date: 10/16/2020
ms.author: midesa
ms.reviewer: jrasnick
ms.subservice: spark
ms.openlocfilehash: 62610e1b86671021e66891ae232bacbd4b3e40ed
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96458816"
---
# <a name="manage-libraries-for-apache-spark-in-azure-synapse-analytics"></a>Gérer des bibliothèques pour Apache Spark dans Azure Synapse Analytics

Les bibliothèques fournissent du code réutilisable que vous pouvez inclure dans vos programmes ou projets. Pour que vos applications disposent d’un code tiers ou généré en local, vous pouvez installer une bibliothèque sur l’un de vos pools Apache Spark serverless. Une fois qu’une bibliothèque est installée pour un pool Spark, elle est disponible pour toutes les sessions utilisant le même pool. 

## <a name="before-you-begin"></a>Avant de commencer
- Pour installer et mettre à jour des bibliothèques, vous devez disposer des autorisations **Contributeur aux données Blob du stockage** ou **Propriétaire des données Blob du stockage** sur le compte de stockage Gen2 principal qui est lié à l’espace de travail Azure Synapse Analytics.
  
## <a name="default-installation"></a>Installation par défaut
Apache Spark dans Azure Synapse Analytics propose une installation complète d’Anacondas, ainsi que de bibliothèques supplémentaires. Pour la liste complète des bibliothèques, consultez la [prise en charge des versions d’Apache Spark](apache-spark-version-support.md). 

Lors du démarrage d’une instance Spark, ces bibliothèques sont automatiquement incluses. Vous pouvez ajouter des packages Python et des packages personnalisés supplémentaires au niveau du pool Spark.


## <a name="manage-python-packages"></a>Gérer les packages Python
Une fois que vous avez identifié les bibliothèques que vous souhaitez utiliser pour votre application Spark, vous pouvez les installer dans un pool Spark. 

 Un fichier *requirements.txt* (sortie de la commande `pip freeze`) permet de mettre à niveau l’environnement virtuel. Les packages répertoriés dans ce fichier pour l’installation ou la mise à niveau sont téléchargés à partir de PyPi au moment du démarrage du pool. Ce fichier de configuration requise est utilisé chaque fois qu’une instance Spark est créée à partir de ce pool Spark.

> [!IMPORTANT]
> - Si le package que vous installez est volumineux ou si son installation prend beaucoup de temps, cela affecte le temps de démarrage de l’instance Spark.
> - Les packages qui requièrent une prise en charge par le compilateur au moment de l’installation, tels que GCC, ne sont pas pris en charge.
> - Il n’est pas possible de ramener les packages à une version antérieure, mais uniquement de les ajouter ou de les mettre à niveau.
> - Pour installer des bibliothèques, vous devez disposer des autorisations Contributeur aux données Blob du stockage ou Propriétaire des données Blob du stockage sur le compte de stockage Gen2 principal qui est lié à l’espace de travail Synapse.

### <a name="requirements-format"></a>Format de la configuration requise

L’extrait de code suivant montre le format du fichier de configuration requise. Le nom du package PyPi est mentionné avec une version exacte. Ce fichier est au format décrit dans la documentation de référence [pip freeze](https://pip.pypa.io/en/stable/reference/pip_freeze/). Cet exemple épingle une version spécifique. 

```
absl-py==0.7.0
adal==1.2.1
alabaster==0.7.10
```

### <a name="install-python-packages"></a>Installer des packages Python
Lorsque vous développez votre application Spark, vous pouvez être amené à mettre à jour des bibliothèques existantes ou à en installer. Les bibliothèques peuvent être mises à jour pendant ou après la création du pool.

#### <a name="install-packages-during-pool-creation"></a>Installer des packages lors de la création du pool
Pour installer des bibliothèques dans un pool Spark pendant la création du pool :
   
1. Accédez à votre espace de travail Azure Synapse Analytics à partir du portail Azure.
   
2. Sélectionnez **Créer un pool Apache Spark**, puis sélectionnez l’onglet **Paramètres supplémentaires**. 
   
3. Chargez le fichier de configuration de l’environnement à l’aide du sélecteur de fichiers dans la section **Packages** de la page. 
   
    ![Ajouter des bibliothèques Python pendant la création du pool](./media/apache-spark-azure-portal-add-libraries/apache-spark-azure-portal-add-library-python.png "Ajouter des bibliothèques Python")
 

#### <a name="install-packages-from-the-synapse-workspace"></a>Installer des packages à partir de l’espace de travail Synapse
Pour mettre à jour ou ajouter des bibliothèques supplémentaires à un pool Spark à partir du portail Azure Synapse Analytics :

1.  Accédez à votre espace de travail Azure Synapse Analytics à partir du portail Azure.
   
2.  Lancez votre espace de travail Azure Synapse Analytics à partir du portail Azure.

3.  Sélectionnez **Gérer** à partir du volet de navigation principal, puis sélectionnez **Pools Apache Spark**.
   
4. Sélectionnez un pool Spark unique et chargez le fichier de configuration de l’environnement à l’aide du sélecteur de fichiers dans la section **Packages** de la page.

    ![Ajouter des bibliothèques Python dans Synapse](./media/apache-spark-azure-portal-add-libraries/apache-spark-azure-portal-update.png)
   
#### <a name="install-packages-from-the-azure-portal"></a>Installer des packages à partir du portail Azure
Pour installer une bibliothèque sur un pool Spark directement à partir du portail Azure :
   
 1. Accédez à votre espace de travail Azure Synapse Analytics à partir du portail Azure.
   
 2. Sous la section **Ressources Synapse**, sélectionnez l’onglet **Pools Apache Spark** et sélectionnez un pool Spark dans la liste.
   
 3. Sélectionnez **Packages** dans la section **Paramètres** du pool Spark. 

 4. Chargez le fichier de configuration de l’environnement à l’aide du sélecteur de fichiers.

    ![Capture d’écran qui met en surbrillance le bouton Charger le fichier config de l'environnement.](./media/apache-spark-azure-portal-add-libraries/apache-spark-add-library-azure.png "Ajouter des bibliothèques Python")

### <a name="verify-installed-libraries"></a>Vérifier les bibliothèques installées

Pour vérifier si les bonnes versions des bibliothèques appropriées sont installées, exécutez le code suivant :

```python
import pkg_resources
for d in pkg_resources.working_set:
     print(d)
```
### <a name="update-python-packages"></a>Mettre à jour les packages Python
Les packages peuvent être ajoutés ou modifiés à tout moment entre les sessions. Lors du chargement d’un nouveau fichier de configuration de package, les packages et versions existants sont remplacés.  

Pour mettre à jour ou désinstaller une bibliothèque :
1. Accédez à votre espace de travail Azure Synapse Analytics. 

2. À l’aide du portail Azure ou de l’espace de travail Azure Synapse, sélectionnez le **pool Apache Spark** que vous souhaitez mettre à jour.

3. Accédez à la section **Packages** et téléchargez un nouveau fichier de configuration d’environnement
   
4. Une fois que vous avez enregistré vos modifications, vous devez terminer les sessions actives et laisser le pool redémarrer. Si vous le souhaitez, vous pouvez forcer la fin des sessions actives en activant la case **Forcer les nouveaux paramètres**.

    ![Ajouter des bibliothèques Python](./media/apache-spark-azure-portal-add-libraries/update-libraries.png "Ajouter des bibliothèques Python")
   

> [!IMPORTANT]
> En sélectionnant l’option **Forcer les nouveaux paramètres**, vous terminez toutes les sessions actives pour le pool Spark sélectionné. Une fois les sessions terminées, vous devez attendre le redémarrage du pool. 
>
> Si ce paramètre est désactivé, vous devez attendre que la session Spark en cours se termine ou l’arrêter manuellement. Une fois la session terminée, vous devez laisser le pool redémarrer. 


## <a name="manage-a-python-wheel"></a>Gérer un fichier Wheel Python

### <a name="install-a-custom-wheel-file"></a>Installer un fichier Wheel personnalisé
Les packages Wheel personnalisés peuvent être installés sur le pool Apache Spark en chargeant tous les fichiers Wheel dans le compte Azure Data Lake Storage (Gen2) qui est lié à l’espace de travail Synapse. 

Les fichiers doivent être téléchargés vers le chemin d’accès suivant dans le conteneur par défaut du compte de stockage : 

```
abfss://<file_system>@<account_name>.dfs.core.windows.net/synapse/workspaces/<workspace_name>/sparkpools/<pool_name>/libraries/python/
```

>[!IMPORTANT]
>Des packages personnalisés peuvent être ajoutés ou modifiés entre les sessions. Toutefois, vous devrez attendre le redémarrage du pool et de la session pour voir le package mis à jour.

## <a name="next-steps"></a>Étapes suivantes
- Afficher les bibliothèques par défaut : [Prise en charge des versions d’Apache Spark](apache-spark-version-support.md)
