---
title: Explorer les données et le modèle sur Windows
titleSuffix: Azure Data Science Virtual Machine
description: Effectuez des tâches de modélisation et d’exploration des données sur Data Science Virtual Machine Windows.
services: machine-learning
ms.service: machine-learning
ms.subservice: data-science-vm
ms.custom: devx-track-python
author: lobrien
ms.author: laobri
ms.topic: conceptual
ms.date: 05/08/2020
ms.openlocfilehash: 09a4dc8467b9b3c11c8daeb5ca8c62a0e26e79de
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96575004"
---
# <a name="data-science-with-a-windows-data-science-virtual-machine"></a>Science des données avec une Data Science Virtual Machine Windows

Windows Data Science Virtual Machine (DSVM) est un puissant environnement de développement pour la science des données dans lequel vous pouvez effectuer des tâches de modélisation et d’exploration des données. L’environnement est déjà généré et fourni avec plusieurs outils d’analytique de données courants qui facilitent la prise en main de votre analyse pour les déploiements locaux, cloud ou hybrides. 

L’environnement DSVM opère en étroite collaboration avec les services Azure. Il peut lire et traiter les données déjà stockées sur Azure, dans Azure Synapse (anciennement SQL DW), Azure Data Lake, Stockage Azure ou Azure Cosmos DB. Il peut aussi tirer parti d’autres outils d’analytique comme Azure Machine Learning.

Dans cet article vous allez découvrir comment utiliser votre DSVM afin d’effectuer des tâches de science des données et d’interagir avec d’autres services Azure. Voici quelques-unes des tâches que vous pouvez effectuer sur la DSVM :

- Utiliser un notebook Jupyter pour mener des expériences avec vos données dans un navigateur à l’aide de Python 2, Python 3 et Microsoft R. (Microsoft R est une version d’entreprise de R conçue pour les performances.)
- Explorer les données et développer des modèles localement sur la DSVM avec Microsoft Machine Learning Server et Python.
- Administrer vos ressources Azure à l’aide du portail Azure ou de PowerShell.
- Augmenter votre espace de stockage et partager des jeux de données/du code à grande échelle avec toute votre équipe en créant un partage Azure Files comme lecteur montable sur votre DSVM.
- Partager du code avec votre équipe à l’aide de GitHub. Accéder à votre dépôt à l’aide des clients Git préinstallés : Git Bash et Git GUI
- Accédez aux services de données et d’analytique Azure tels que Stockage Blob Azure, Azure Cosmos DB, Azure Synapse (anciennement SQL DW) et Azure SQL Database.
- Générer des rapports et un tableau de bord à l’aide de l’instance de Power BI Desktop préinstallée sur la DSVM, et les déployer dans le cloud.

- Installer des outils supplémentaires sur votre machine virtuelle.   

> [!NOTE]
> Des frais d’utilisation supplémentaires s’appliquent pour la plupart des services de stockage et d’analytique des données mentionnés dans cet article. Pour plus d’informations, consultez la page des [tarifs Azure](https://azure.microsoft.com/pricing/).
> 
> 

## <a name="prerequisites"></a>Prérequis

* Vous avez besoin d’un abonnement Azure. Vous pouvez vous inscrire à un [essai gratuit](https://azure.microsoft.com/free/).
* Vous trouverez des instructions pour le provisionnement d’une Data Science Virtual Machine dans le portail Azure dans [Création d’une machine virtuelle](https://portal.azure.com/#create/microsoft-dsvm.dsvm-windowsserver-2016).


[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]


## <a name="use-jupyter-notebooks"></a>Utiliser des notebooks Jupyter
Jupyter Notebook fournit un IDE basé sur navigateur pour l’exploration et la modélisation des données. Vous pouvez utiliser Python 2, Python 3 ou R (open source et Microsoft R Server) dans un notebook Jupyter.

Pour démarrer Jupyter Notebook, sélectionnez l’icône **Jupyter Notebook** dans le menu **Démarrer** ou sur le Bureau. À l’invite de commande DSVM, vous pouvez également exécuter la commande ```jupyter notebook``` à partir du répertoire où se trouvent vos notebooks, ou bien à partir de l’emplacement où vous souhaitez en créer d’autres.  

Lorsque vous avez démarré Jupyter, accédez au répertoire `/notebooks` pour obtenir des exemples de blocs-notes préconfigurés dans le DSVM. Vous pouvez désormais :

* Sélectionner le notebook pour visualiser le code.
* Exécuter chaque cellule en sélectionnant Maj+Entrée.
* Exécuter l’intégralité du notebook en sélectionnant **Cellule** > **Exécuter**.
* Créer un notebook en sélectionnant l’icône Jupyter (coin supérieur gauche), le bouton **Nouveau** sur la droite, puis en choisissant la langue du notebook (également appelé noyau).   

> [!NOTE]
> Actuellement, les noyaux Python 2.7, Python 3.6, R, Julia et PySpark dans Jupyter sont pris en charge. Le noyau R prend en charge la programmation dans R et Microsoft R Open Source.   
> 
> 

Une fois dans le notebook, vous pouvez explorer vos données, générer le modèle et le tester avec les bibliothèques de votre choix.

## <a name="explore-data-and-develop-models-with-microsoft-machine-learning-server"></a>Explorer les données et développer des modèles avec Microsoft Machine Learning Server
Vous pouvez utiliser des langages tels que R et Python pour effectuer des analyses de données directement sur la DSVM.

Pour R, vous pouvez utiliser un IDE comme RStudio, accessible à partir du menu Démarrer ou sur le Bureau. Vous pouvez aussi utiliser Outils R pour Visual Studio. Microsoft a fourni des bibliothèques supplémentaires en plus d’Open source/CRAN-R pour permettre de mettre à l’échelle l’analytique et d’analyser des volumes de données plus importants que la taille de mémoire autorisée lors d’une analyse parallèle par blocs. 

Pour Python, vous pouvez utiliser un IDE comme Visual Studio Community Edition, dans lequel l'extension Outils Python pour Visual Studio (PTVS) est préinstallée. Par défaut, seul Python 3.6, l’environnement racine de Conda, est configuré sur PTVS. Pour activer Anaconda Python 2.7, effectuez les étapes suivantes :

1. Créez des environnements personnalisés pour chaque version en accédant à **Outils** > **Outils Python** > **Environnements Python**, puis en sélectionnant **+ Personnalisé** dans Visual Studio Community Edition.
1. Donnez une description et définissez **c:\anaconda\envs\python2** comme chemin de préfixe d’environnement pour Anaconda Python 2.7.
1. Sélectionnez **Détection automatique** > **Appliquer** pour enregistrer l’environnement.

Pour plus d’informations sur la façon de créer des environnements Python, consultez la [Documentation de PTVS](/visualstudio/python/).

Vous êtes maintenant prêt à créer un nouveau projet Python. Accédez à **Fichier** > **Nouveau** > **Projet** > **Python** et sélectionnez le type d’application Python que vous créez. Vous pouvez définir l’environnement Python pour le projet actuel sur la version souhaitée (Python 2.7 ou 3.6) en cliquant avec le bouton droit sur **Environnements Python** et en sélectionnant **Ajouter/supprimer des environnements Python**. Vous trouverez plus d’informations sur l’utilisation de PTVS dans la [documentation produit](/visualstudio/python/).



## <a name="manage-azure-resources"></a>Gérer des ressources Azure
La DSVM ne vous permet pas uniquement de créer votre solution d’analytique localement sur la machine virtuelle. Elle vous permet aussi d’accéder à des services sur la plateforme cloud Azure. Azure fournit plusieurs services de calcul, de stockage, d’analytique de données et autres, que vous pouvez administrer et auxquels vous pouvez accéder à partir de votre DSVM.

Pour gérer vos ressources cloud et votre abonnement Azure, deux options s’offrent à vous :
+ Utiliser votre navigateur web et accéder au [portail Azure](https://portal.azure.com)

+ Utiliser des scripts PowerShell. Exécuter Azure PowerShell à partir d’un raccourci sur le Bureau ou à partir du menu **Démarrer**. Pour plus d’informations, voir la [documentation Microsoft Azure PowerShell](../../azure-resource-manager/management/manage-resources-powershell.md) 

## <a name="extend-storage-by-using-shared-file-systems"></a>Étendre le stockage à l’aide de systèmes de fichiers partagés
Les scientifiques des données peuvent partager des jeux de données volumineux, du code ou d’autres ressources au sein de leur équipe. La DSVM dispose d’environ 45 Go d’espace disponible. Pour étendre votre espace de stockage, vous pouvez utiliser Azure Files et le monter sur une ou plusieurs instances DSVM ou y accéder par le biais de l’API REST. Vous pouvez également utiliser le [portail Azure](../../virtual-machines/windows/attach-managed-disk-portal.md) ou [Azure PowerShell](../../virtual-machines/windows/attach-disk-ps.md) pour ajouter des disques de données supplémentaires dédiés. 

> [!NOTE]
> L’espace maximal sur le partage Azure Files est de 5 To. La taille maximale de chaque fichier est de 1 To. 

Vous pouvez utiliser ce script dans Azure PowerShell pour créer un partage Azure Files :

```powershell
# Authenticate to Azure.
Connect-AzAccount
# Select your subscription
Get-AzSubscription –SubscriptionName "<your subscription name>" | Select-AzSubscription
# Create a new resource group.
New-AzResourceGroup -Name <dsvmdatarg>
# Create a new storage account. You can reuse existing storage account if you want.
New-AzStorageAccount -Name <mydatadisk> -ResourceGroupName <dsvmdatarg> -Location "<Azure Data Center Name For eg. South Central US>" -Type "Standard_LRS"
# Set your current working storage account
Set-AzCurrentStorageAccount –ResourceGroupName "<dsvmdatarg>" –StorageAccountName <mydatadisk>

# Create an Azure Files share
$s = New-AzStorageShare <<teamsharename>>
# Create a directory under the file share. You can give it any name
New-AzStorageDirectory -Share $s -Path <directory name>
# List the share to confirm that everything worked
Get-AzStorageFile -Share $s
```

Maintenant que vous avez créé un partage Azure Files, vous pouvez le monter sur n’importe quelle machine virtuelle dans Azure. Nous vous recommandons vivement de placer la machine virtuelle dans le même centre de données Azure que le compte de stockage, pour éviter toute latence et tout frais de transfert de données. Voici les commandes Azure PowerShell permettant de monter le lecteur sur la DSVM :

```powershell
# Get the storage key of the storage account that has the Azure Files share from the Azure portal. Store it securely on the VM to avoid being prompted in the next command.
cmdkey /add:<<mydatadisk>>.file.core.windows.net /user:<<mydatadisk>> /pass:<storage key>

# Mount the Azure Files share as drive Z on the VM. You can choose another drive letter if you want.
net use z:  \\<mydatadisk>.file.core.windows.net\<<teamsharename>>
```

Vous pouvez désormais accéder à ce lecteur comme à n’importe quel lecteur normal sur la machine virtuelle.

## <a name="share-code-in-github"></a>Partager du code dans GitHub
GitHub est un dépôt de code où vous trouverez des exemples de code et des sources pour différents outils utilisant des technologies partagées par la communauté des développeurs. Il utilise la technologie Git pour suivre et stocker les versions des fichiers de code. GitHub est également une plateforme qui vous permet de créer votre propre dépôt pour stocker le code et la documentation partagés de votre équipe, d’implémenter la gestion des versions et de contrôler les accès pour afficher le code et y contribuer. 

Visitez les [pages d’aide GitHub](https://help.github.com/) pour plus d’informations sur l’utilisation de Git. Vous pouvez utiliser GitHub pour collaborer avec votre équipe, utiliser le code développé par la communauté et apporter une contribution au code pour la communauté.

La DSVM est livrée avec des outils clients en ligne de commande et dans l’interface graphique utilisateur pour accéder au dépôt GitHub. L’outil en ligne de commande qui fonctionne avec Git et GitHub se nomme Git Bash. Visual Studio est installé sur la DSVM et comprend les extensions Git. Vous trouverez les icônes de ces outils dans le menu **Démarrer** et sur le Bureau.

Pour télécharger du code à partir d’un référentiel GitHub, utilisez la commande ```git clone```. Par exemple, pour télécharger le dépôt de science des données publié par Microsoft dans le répertoire actif, vous pouvez exécuter la commande suivante dans Git Bash :

```bash
git clone https://github.com/Azure/DataScienceVM.git
```

Dans Visual Studio, vous pouvez effectuer la même opération de clonage. La capture d’écran suivante montre comment accéder aux outils Git et GitHub dans Visual Studio :

![Capture d’écran de Visual Studio avec la connexion GitHub affichée](./media/vm-do-ten-things/VSGit.png)

Vous trouverez plus d’informations sur l’utilisation de Git pour travailler avec votre dépôt GitHub en consultant les ressources disponibles sur github.com. L’ [aide-mémoire](https://services.github.com/on-demand/downloads/github-git-cheat-sheet.pdf) constitue une référence utile.

## <a name="access-azure-data-and-analytics-services"></a>Accéder aux services de données et d’analytique Azure
### <a name="azure-blob-storage"></a>Stockage Blob Azure
Stockage Blob Azure est un service de stockage cloud fiable et économique pour tous les types de données. Cette section décrit comment déplacer des données vers le stockage d’objets blob et accéder à des données stockées dans un objet blob Azure.

#### <a name="prerequisites"></a>Prérequis

* Créez votre compte Stockage Blob Azure dans le [portail Azure](https://portal.azure.com).

   ![Capture d’écran du processus de création d’un compte de stockage dans le portail Azure](./media/vm-do-ten-things/create-azure-blob.png)

* Vérifiez que l’outil en ligne de commande AzCopy est préinstallé : ```C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy.exe```. Le répertoire qui contient le fichier azcopy.exe se trouve déjà sur votre variable d’environnement PATH. Ainsi, vous n’êtes pas obligé de taper tout le chemin de la commande lors de l’exécution de cet outil. Pour plus d’informations sur l’outil AzCopy, consultez la [documentation d’AzCopy](../../storage/common/storage-use-azcopy-v10.md).
* Lancez l’outil Azure Storage Explorer. Vous pouvez le télécharger à partir de la page web [Explorateur Stockage Azure](https://storageexplorer.com/). 

   ![Capture d’écran de l’Explorateur Stockage Azure accédant à un compte de stockage](./media/vm-do-ten-things/AzureStorageExplorer_v4.png)

#### <a name="move-data-from-a-vm-to-an-azure-blob-azcopy"></a>Déplacer des données d’une machine virtuelle vers un objet blob Azure : AzCopy

Pour déplacer des données entre vos fichiers locaux et le stockage d’objets blob, vous pouvez utiliser AzCopy sur la ligne de commande ou dans PowerShell :

```powershell
AzCopy /Source:C:\myfolder /Dest:https://<mystorageaccount>.blob.core.windows.net/<mycontainer> /DestKey:<storage account key> /Pattern:abc.txt
```

Remplacez **C:\myfolder** par le chemin de votre fichier, **mystorageaccount** par le nom de votre compte Stockage Blob, **mycontainer** par le nom du conteneur et **storage account key** par votre clé d’accès Stockage Blob. Vous trouverez les informations d’identification de votre compte de stockage dans le [portail Azure](https://portal.azure.com).

Exécutez la commande AzCopy dans PowerShell ou à partir d’une invite de commandes. Voici un exemple d’utilisation de la commande AzCopy :

```powershell
# Copy *.sql from a local machine to an Azure blob
"C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy" /Source:"c:\Aaqs\Data Science Scripts" /Dest:https://[ENTER STORAGE ACCOUNT].blob.core.windows.net/[ENTER CONTAINER] /DestKey:[ENTER STORAGE KEY] /S /Pattern:*.sql

# Copy back all files from an Azure blob container to a local machine

"C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy" /Dest:"c:\Aaqs\Data Science Scripts\temp" /Source:https://[ENTER STORAGE ACCOUNT].blob.core.windows.net/[ENTER CONTAINER] /SourceKey:[ENTER STORAGE KEY] /S
```

Une fois que vous avez exécuté la commande AzCopy pour copier vers un objet blob Azure, votre fichier apparaît dans l’Explorateur Stockage Azure.

![Capture d’écran du compte de stockage affichant le fichier CSV chargé](./media/vm-do-ten-things/AzCopy_run_finshed_Storage_Explorer_v3.png)

#### <a name="move-data-from-a-vm-to-an-azure-blob-azure-storage-explorer"></a>Déplacer des données d’une machine virtuelle vers un objet blob Azure : Explorateur de stockage Azure

Vous pouvez également charger des données du fichier local vers votre machine virtuelle à l’aide de l’Explorateur Stockage Azure :

* Pour charger des données vers un conteneur, sélectionnez le conteneur cible, puis le bouton **Charger**.![Capture d’écran du bouton Charger dans l’Explorateur Stockage Azure](./media/vm-do-ten-things/storage-accounts.png)
* Sélectionnez **...** à droite de la zone **Fichiers**, sélectionnez un ou plusieurs fichiers à charger à partir du système de fichiers, puis sélectionnez **Charger** pour lancer le chargement des fichiers.![Capture d’écran de la boîte de dialogue Charger des fichiers](./media/vm-do-ten-things/upload-files-to-blob.png)

#### <a name="read-data-from-an-azure-blob-python-odbc"></a>Lire des données à partir d’un objet blob : Python ODBC

Vous pouvez utiliser la bibliothèque BlobService pour lire les données directement à partir de l’objet blob dans un notebook Jupyter ou un programme Python.

Commencez par importer les packages requis :

```python
import pandas as pd
from pandas import Series, DataFrame
import numpy as np
import matplotlib.pyplot as plt
from time import time
import pyodbc
import os
from azure.storage.blob import BlobService
import tables
import time
import zipfile
import random
```

Ensuite, entrez les informations d’identification de votre compte Stockage Blob et lisez les données de l’objet Blob :

```python
CONTAINERNAME = 'xxx'
STORAGEACCOUNTNAME = 'xxxx'
STORAGEACCOUNTKEY = 'xxxxxxxxxxxxxxxx'
BLOBNAME = 'nyctaxidataset/nyctaxitrip/trip_data_1.csv'
localfilename = 'trip_data_1.csv'
LOCALDIRECTORY = os.getcwd()
LOCALFILE =  os.path.join(LOCALDIRECTORY, localfilename)

#download from blob
t1 = time.time()
blob_service = BlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)
blob_service.get_blob_to_path(CONTAINERNAME,BLOBNAME,LOCALFILE)
t2 = time.time()
print(("It takes %s seconds to download "+BLOBNAME) % (t2 - t1))

#unzip downloaded files if needed
#with zipfile.ZipFile(ZIPPEDLOCALFILE, "r") as z:
#    z.extractall(LOCALDIRECTORY)

df1 = pd.read_csv(LOCALFILE, header=0)
df1.columns = ['medallion','hack_license','vendor_id','rate_code','store_and_fwd_flag','pickup_datetime','dropoff_datetime','passenger_count','trip_time_in_secs','trip_distance','pickup_longitude','pickup_latitude','dropoff_longitude','dropoff_latitude']
print 'the size of the data is: %d rows and  %d columns' % df1.shape
```

Les données sont lues en tant que trame de données :

![Capture d’écran des 10 premières lignes de données](./media/vm-do-ten-things/IPNB_data_readin.png)


### <a name="azure-synapse-analytics-and-databases"></a>Azure Synapse Analytics et les bases de données
Azure Synapse Analytics est un entrepôt de données élastique en tant que service offrant une expérience SQL Server de classe Entreprise.

Vous pouvez configurer Azure Synapse Analytics en suivant les instructions dans [cet article](../../synapse-analytics/sql-data-warehouse/create-data-warehouse-portal.md). Une fois que vous avez configuré Azure Synapse Analytics, vous pouvez utiliser [cette procédure pas à pas](../team-data-science-process/sqldw-walkthrough.md) pour le chargement, l’exploration et la modélisation de données en utilisant les données d’Azure Synapse Analytics.

#### <a name="azure-cosmos-db"></a>Azure Cosmos DB
Azure Cosmos DB est une base de données NoSQL sur le cloud. Vous pouvez l’utiliser pour travailler avec des documents comme JSON, et pour stocker et interroger les documents.

Effectuez les étapes prérequises suivantes pour accéder à Azure Cosmos DB à partir de la DSVM :

1. Le Kit de développement logiciel (SDK) Azure Cosmos DB Python est déjà installé sur la DSVM. Pour le mettre à jour, exécutez ```pip install pydocumentdb --upgrade``` à partir d’une invite de commandes.
2. Créez un compte et une base de données Azure Cosmos DB à partir du [portail Azure](https://portal.azure.com).
3. Téléchargez l’outil de migration de données Azure Cosmos DB à partir du [Centre de téléchargement Microsoft](https://www.microsoft.com/download/details.aspx?id=53595) et extrayez-le dans le répertoire de votre choix.
4. Importez des données JSON (données volcano) stockées dans un [objet blob public](https://data.humdata.org/dataset/a60ac839-920d-435a-bf7d-25855602699d/resource/7234d067-2d74-449a-9c61-22ae6d98d928/download/volcano.json) dans Azure Cosmos DB avec les paramètres de commande suivants dans l’outil de migration. (Utilisez dtui.exe à partir du répertoire où vous avez installé l’outil de migration de données Azure Cosmos DB.) Entrez les paramètres d'emplacement source et cible suivant :
   
    `/s:JsonFile /s.Files:https://data.humdata.org/dataset/a60ac839-920d-435a-bf7d-25855602699d/resource/7234d067-2d74-449a-9c61-22ae6d98d928/download/volcano.json /t:DocumentDBBulk /t.ConnectionString:AccountEndpoint=https://[DocDBAccountName].documents.azure.com:443/;AccountKey=[[KEY];Database=volcano /t.Collection:volcano1`

Une fois les données importées, vous pouvez accéder à Jupyter et ouvrir le notebook intitulé *DocumentDBSample*. Il contient du code Python pour accéder à Azure Cosmos DB et effectuer des requêtes de base. Pour en savoir plus sur Azure Cosmos DB, consultez la [page de documentation](../../cosmos-db/index.yml) du service.

## <a name="use-power-bi-reports-and-dashboards"></a>Utiliser des rapports et des tableaux de bord Power BI 
Vous pouvez visualiser le fichier JSON Volcano de l’exemple Azure Cosmos DB précédent dans Power BI Desktop afin d’obtenir des insights visuels des données. Les étapes détaillées sont présentées dans [l’article Power BI](../../cosmos-db/powerbi-visualize.md). Les étapes principales sont les suivantes :

1. Ouvrez Power BI Desktop et sélectionnez **obtenir les données**. Spécifiez l’URL suivante : `https://cahandson.blob.core.windows.net/samples/volcano.json`.
2. Vous devez voir les enregistrements JSON importés sous forme de liste. Convertissez la liste en table afin que Power BI puisse l’utiliser.
4. Développez les colonnes en sélectionnant l’icône de développement (flèche).
5. Notez que cet emplacement est un champ **Enregistrement**. Développez l'enregistrement et sélectionnez uniquement les coordonnées. Une **coordonnée** est une colonne de liste.
6. Ajoutez une nouvelle colonne pour convertir la colonne de coordonnées de liste en colonne **LatLong** séparée par des virgules. Concaténez les deux éléments dans le champ de liste de coordonnées à l’aide de la formule ```Text.From([coordinates]{1})&","&Text.From([coordinates]{0})```.
7. Convertissez la colonne **Elevation** au format décimal et sélectionnez les boutons **Fermer** et **Appliquer**.

Au lieu des étapes précédentes, vous pouvez coller le code suivant. Il écrit les étapes utilisées dans l’Éditeur avancé dans Power BI afin d’écrire les transformations de données dans un langage de requête.

```pqfl
let
    Source = Json.Document(Web.Contents("https://cahandson.blob.core.windows.net/samples/volcano.json")),
    #"Converted to Table" = Table.FromList(Source, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    #"Expanded Column1" = Table.ExpandRecordColumn(#"Converted to Table", "Column1", {"Volcano Name", "Country", "Region", "Location", "Elevation", "Type", "Status", "Last Known Eruption", "id"}, {"Volcano Name", "Country", "Region", "Location", "Elevation", "Type", "Status", "Last Known Eruption", "id"}),
    #"Expanded Location" = Table.ExpandRecordColumn(#"Expanded Column1", "Location", {"coordinates"}, {"coordinates"}),
    #"Added Custom" = Table.AddColumn(#"Expanded Location", "LatLong", each Text.From([coordinates]{1})&","&Text.From([coordinates]{0})),
    #"Changed Type" = Table.TransformColumnTypes(#"Added Custom",{{"Elevation", type number}})
in
    #"Changed Type"
```

Vous disposez maintenant des données dans votre modèle de données Power BI. Votre instance de Power BI Desktop doit apparaître comme suit :

![Power BI Desktop](./media/vm-do-ten-things/PowerBIVolcanoData.png)

Vous pouvez commencer à générer des rapports et des visualisations à l’aide du modèle de données. Vous pouvez appliquer la procédure décrite dans [cet article Power BI](../../cosmos-db/powerbi-visualize.md#build-the-reports) pour générer un rapport.

## <a name="scale-the-dsvm-dynamically"></a>Mettre à l’échelle la DSVM de manière dynamique 
Vous pouvez effectuer un scale-up ou un scale-down de la DSVM pour répondre aux besoins de votre projet. Si vous n’avez pas besoin d’utiliser la machine virtuelle le soir ou le week-end, vous pouvez simplement l’arrêter à partir du [portail Azure](https://portal.azure.com).

> [!NOTE]
> Vous encourez des frais de calcul si vous utilisez simplement le bouton d’arrêt du système d’exploitation sur la machine virtuelle. Au lieu de cela, vous devez libérer votre DSVM à l’aide du Portail Azure ou Cloud Shell.
> 
> 

Vous devrez peut-être gérer une analyse à grande échelle, et il vous faudra peut-être davantage de capacité de processeur, de mémoire ou de disque. Si c’est le cas, vous trouverez un choix de tailles de machine virtuelle en termes de cœurs de processeur, d’instances basées sur GPU pour le deep learning, de capacité de mémoire et de types de disques (notamment les disques SSD) qui répondent à vos besoins budgétaires et de calcul. La liste complète des machines virtuelles, ainsi que leur tarif horaire de calcul, sont disponibles dans la page [Tarification des machines virtuelles Azure](https://azure.microsoft.com/pricing/details/virtual-machines/).


## <a name="add-more-tools"></a>Ajouter d’autres outils
Les outils préintégrés dans la DSVM peuvent répondre à de nombreux besoins d’analytique de données courants. Cela vous fait gagner du temps, car vous n’avez pas besoin d’installer et de configurer vos environnements un par un. C’est également synonyme d’économies, car vous payez uniquement pour les ressources que vous utilisez.

Vous pouvez utiliser d’autres services de données et d’analytique Azure présentés dans cet article pour améliorer votre environnement d’analytique. Dans certains cas, vous pouvez avoir besoin d’outils supplémentaires, notamment certains outils partenaires propriétaires. Vous disposez d’un accès administratif total sur la machine virtuelle pour installer les nouveaux outils dont vous avez besoin. Vous pouvez également installer des packages supplémentaires non préinstallés en Python et R. Pour Python, vous pouvez utiliser ```conda``` ou ```pip```. Pour R, vous pouvez utiliser ```install.packages()``` dans la console R, ou utiliser l’IDE et sélectionner **Packages** > **Installer les packages**.

## <a name="deep-learning"></a>Apprentissage approfondi

Outre les exemples basés sur les frameworks, vous pouvez obtenir un ensemble de procédures détaillées qui ont été validées sur la DSVM. Ces procédures vous permettent de vous lancer dans le développement d’applications de deep learning dans des domaines tels que la reconnaissance vocale et la compréhension d’image et de texte.   


- [Exécution de réseaux neuronaux sur différents frameworks](https://github.com/ilkarman/DeepLearningFrameworks) : cette procédure pas à pas montre comment migrer du code d’un framework vers un autre. Elle montre également comment comparer les performances des modèles et du runtime entre les frameworks. 

- [Guide pratique pour créer une solution de bout en bout afin de détecter des produits dans des images](https://github.com/Azure/cortana-intelligence-product-detection-from-images) : la détection d’image est une technique qui peut localiser des objets dans les images et les classer. Les retombées concrètes de cette technologie semblent très prometteuses dans de nombreux domaines professionnels. Par exemple, un détaillant peut utiliser cette technique pour déterminer quel produit un client a choisi dans un rayon. Il peut ensuite utiliser ces informations pour gérer le stock des produits. 

- [Deep learning pour l’audio](/archive/blogs/machinelearning/hearing-ai-getting-started-with-deep-learning-for-audio-on-azure) : ce tutoriel montre comment entraîner un modèle de deep learning pour la détection d’événements audio sur le [jeu de données de sons urbains](https://serv.cusp.nyu.edu/projects/urbansounddataset/urbansound8k.html). Il fournit également une vue d’ensemble de l’utilisation des données audio.

- [Classification de documents texte](https://github.com/anargyri/lstm_han) : cette procédure pas à pas montre comment créer et entraîner deux architectures de réseau neuronal : réseau d’attention hiérarchisée et réseau à mémoire à long ou court terme. Ces réseaux neuronaux utilisent l’API Keras de l’apprentissage profond pour classer les documents texte. 

## <a name="summary"></a>Résumé
Cet article décrit certaines des actions possibles sur la Microsoft Data Science Virtual Machine. Il existe bien d’autres actions que vous pouvez effectuer pour faire de la DSVM un environnement d’analytique efficace.
