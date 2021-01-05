---
title: Déployer une application .NET dans un conteneur vers Azure Service Fabric
description: Découvrez comment mettre en conteneur une application .NET existante à l’aide de Visual Studio et comment déboguer des conteneurs Service Fabric en local. L’application en conteneur est envoyée par push dans un registre de conteneur Azure et déployée dans un cluster Service Fabric. Lorsqu’elle est déployée dans Azure, l’application utilise Azure SQL DB pour conserver des données.
ms.topic: tutorial
ms.date: 07/08/2019
ms.openlocfilehash: 8be9de495fa6bc5689a2dba5384f5df3112cbb38
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96485520"
---
# <a name="tutorial-deploy-a-net-application-in-a-windows-container-to-azure-service-fabric"></a>Tutoriel : Déployer une application .NET dans un conteneur Windows vers Azure Service Fabric

Ce didacticiel explique comment mettre une application ASP.NET existante en conteneur et l’empaqueter en tant qu’application Service Fabric.  Exécutez les conteneurs en local sur le cluster de développement Service Fabric puis déployez l’application vers Azure.  L’application conserve les données dans [Azure SQL Database](../azure-sql/database/sql-database-paas-overview.md).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Mettre en conteneur une application existante à l’aide de Visual Studio
> * Créer une base de données dans Azure SQL Database
> * Créer un Azure Container Registry
> * Déployer une application Service Fabric sur Azure

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

1. Si vous n’avez pas d’abonnement Azure, [créer un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
2. Installez [Docker CE pour Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows?tab=description) afin que vous puissiez exécuter des conteneurs sur Windows 10.
3. Installez le [runtime Service Fabric version 6.2 ou supérieure](service-fabric-get-started.md) et le [kit de développement logiciel (SDK) Service Fabric version 3.1](service-fabric-get-started.md) ou supérieure.
4. Installez [Visual Studio 2019 version 16.1](https://www.visualstudio.com/) ou ultérieure avec les charges de travail **Développement Azure** et **Développement web et ASP.NET**.
5. Installez [Azure PowerShell][link-azure-powershell-install].

## <a name="download-and-run-fabrikam-fiber-callcenter"></a>Télécharger et exécuter Fabrikam Fiber CallCenter

1. Téléchargez l’exemple d’application [Fabrikam Fiber CallCenter][link-fabrikam-github].  Cliquez sur le lien **Télécharger l’archive**.  À partir du répertoire *sourceCode* du fichier *fabrikam.zip*, extrayez le fichier *sourceCode.zip* puis le répertoire *VS2015* sur votre ordinateur.

2. Vérifiez que l’application Fabrikam Fiber CallCenter génère et s’exécute sans erreur.  Lancez Visual Studio en tant qu’**administrateur** et ouvrez le fichier [FabrikamFiber.CallCenter.sln][link-fabrikam-github].  Appuyez sur F5 pour déboguer et exécuter l’application.

   ![Capture d’écran de la page d’accueil de l’application Fabrikam Fiber CallCenter s’exécutant sur l’hôte local. La page montre un tableau de bord avec une liste d’appels de support.][fabrikam-web-page]

## <a name="containerize-the-application"></a>Conteneuriser l’application

1. Cliquez avec le bouton droit sur le projet **FabrikamFiber.Web** > **Ajouter** > **Prise en charge de l’orchestrateur de conteneur**.  Sélectionnez **Service Fabric** en tant qu’orchestrateur de conteneur et cliquez sur **OK**.

2. Si vous y êtes invité, cliquez sur **Oui** pour passer de Docker aux conteneurs Windows maintenant.

   Un nouveau projet d’application Service Fabric **FabrikamFiber.CallCenterApplication** est créé dans la solution.  Un fichier Docker est ajouté au projet **FabrikamFiber.Web** existant.  Un répertoire **PackageRoot** est aussi ajouté au projet **FabrikamFiber.Web**, qui contient le manifeste et les paramètres du nouveau service FabrikamFiber.Web.

   Le conteneur est désormais prêt à être généré et empaqueté dans une application Service Fabric. Une fois l’image de conteneur intégrée sur votre ordinateur, vous pouvez la placer dans n’importe quel registre de conteneur et la transmettre à n’importe quel hôte pour l’exécuter.

## <a name="create-an-azure-sql-db"></a>Créer une base de données Azure SQL DB

Lors de l’exécution de l’application Fabrikam Fiber CallCenter en production, les données doivent être conservées dans une base de données. Comme il n’existe aucun moyen de garantir la persistance des données dans un conteneur, vous ne pouvez pas stocker les données de production de SQL Server dans un conteneur.

Nous recommandons [Azure SQL Database](../azure-sql/database/powershell-script-content-guide.md). Pour configurer et exécuter une base de données SQL Server gérée dans Azure, exécutez le script suivant.  Modifiez les variables de script selon vos besoins. *clientIP* représente l’adresse IP de votre ordinateur de développement. Notez le nom du serveur généré par le script.

```powershell
$subscriptionID="<subscription ID>"

# Sign in to your Azure account and select your subscription.
Login-AzAccount -SubscriptionId $subscriptionID

# The data center and resource name for your resources.
$dbresourcegroupname = "fabrikam-fiber-db-group"
$location = "southcentralus"

# The server name: Use a random value or replace with your own value (do not capitalize).
$servername = "fab-fiber-$(Get-Random)"

# Set an admin login and password for your database.
# The login information for the server.
$adminlogin = "ServerAdmin"
$password = "Password@123"

# The IP address of your development computer that accesses the SQL DB.
$clientIP = "<client IP>"

# The database name.
$databasename = "call-center-db"

# Create a new resource group for your deployment and give it a name and a location.
New-AzResourceGroup -Name $dbresourcegroupname -Location $location

# Create the SQL server.
New-AzSqlServer -ResourceGroupName $dbresourcegroupname `
    -ServerName $servername `
    -Location $location `
    -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $adminlogin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))

# Create the firewall rule to allow your development computer to access the server.
New-AzSqlServerFirewallRule -ResourceGroupName $dbresourcegroupname `
    -ServerName $servername `
    -FirewallRuleName "AllowClient" -StartIpAddress $clientIP -EndIpAddress $clientIP

# Create the database in the server.
New-AzSqlDatabase  -ResourceGroupName $dbresourcegroupname `
    -ServerName $servername `
    -DatabaseName $databasename `
    -RequestedServiceObjectiveName "S0"

Write-Host "Server name is $servername"
```

> [!TIP]
> Si vous vous trouvez derrière un pare-feu d’entreprise, l’adresse IP de votre ordinateur de développement peut ne pas être exposée à Internet. Pour vérifier que la base de données dispose de l’adresse IP correcte pour la règle de pare-feu, accédez au [Portail Azure](https://portal.azure.com) et recherchez votre base de données dans la section Bases de données SQL. Cliquez sur son nom, puis dans la section Vue d’ensemble, cliquez sur « Set server firewall » (Configurer le pare-feu du serveur). « Client IP address » (Adresse IP client) représente l’adresse IP de votre ordinateur de développement. Assurez-vous qu’elle corresponde à l’adresse IP dans la règle « AllowClient ».

## <a name="update-the-web-config"></a>Mettre à jour la configuration web

De retour dans le projet **FabrikamFiber.Web**, mettez à jour la chaîne de connexion dans le fichier **web.config** pour pointer vers le serveur SQL Server dans le conteneur.  Mettez à jour la partie *Server* de la chaîne de connexion du serveur avec le nom du serveur créé par le script précédent. Ce nom est du genre « fab-fiber-751718376.database.windows.net ». Dans le code XML suivant, vous devez mettre à jour uniquement l’attribut `connectionString` ; les attributs `providerName` et `name` n’ont pas besoin d’être changés.

```xml
<add name="FabrikamFiber-Express" connectionString="Server=<server name>,1433;Initial Catalog=call-center-db;Persist Security Info=False;User ID=ServerAdmin;Password=Password@123;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;" providerName="System.Data.SqlClient" />
<add name="FabrikamFiber-DataWarehouse" connectionString="Server=<server name>,1433;Initial Catalog=call-center-db;Persist Security Info=False;User ID=ServerAdmin;Password=Password@123;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;" providerName="System.Data.SqlClient" />
  
```

>[!NOTE]
>Vous pouvez utiliser l’instance SQL Server de votre choix pour le débogage local, tant qu’elle est accessible à partir de votre hôte. **localdb** ne prend cependant pas en charge la communication `container -> host`. Si vous souhaitez utiliser une autre base de données SQL lors de la création d’une version build de votre application web, ajoutez une autre chaîne de connexion à votre fichier *web.release.config*.

## <a name="run-the-containerized-application-locally"></a>Exécutez l’application en conteneur localement

Appuyez sur **F5** pour exécuter et déboguer l’application dans un conteneur sur le cluster de développement Service Fabric local. Cliquez sur **Oui** si une boîte de message s’affiche vous demandant d’accorder au groupe « ServiceFabricAllowedUsers » la permission de lire et d’exécuter sur votre répertoire de projet Visual Studio.

## <a name="create-a-container-registry"></a>Créer un registre de conteneur

Maintenant que l’application est exécutée localement, commencez à préparer le déploiement vers Azure.  Les images de conteneur doivent être stockées dans un registre de conteneurs.  Créez un [registre de conteneurs Azure](../container-registry/container-registry-intro.md) à l’aide du script suivant. Le nom du registre de conteneurs est visible par d’autres abonnements Azure, il doit donc être unique.
Avant de déployer l’application vers Azure, vous envoyez par push l’image de conteneur dans le registre.  Lorsque l’application déploie le cluster dans Azure, l’image de conteneur est extraite de ce registre.

```powershell
# Variables
$acrresourcegroupname = "fabrikam-acr-group"
$location = "southcentralus"
$registryname="fabrikamregistry$(Get-Random)"

New-AzResourceGroup -Name $acrresourcegroupname -Location $location

$registry = New-AzContainerRegistry -ResourceGroupName $acrresourcegroupname -Name $registryname -EnableAdminUser -Sku Basic
```

## <a name="create-a-service-fabric-cluster-on-azure"></a>Créer un cluster Service Fabric dans Azure

Les applications Service Fabric peuvent être exécutées sur un cluster, ou sur un ensemble de machines physiques ou virtuelles connectées au réseau.  Avant de pouvoir déployer l’application sur Azure, créez un cluster Service Fabric dans Azure.

Vous pouvez :

* Créez un cluster de test à partir de Visual Studio. Cette option vous permet de créer un cluster sécurisé directement depuis Visual Studio avec les configurations que vous voulez.
* [Créer un cluster sécurisé à partir d’un modèle](service-fabric-tutorial-create-vnet-and-windows-cluster.md)

Ce didacticiel crée un cluster à partir de Visual Studio, idéal pour ces scénarios test. Si vous créez un cluster d’une autre façon ou si vous utilisez un cluster existant, vous pouvez copier et coller votre point de terminaison de connexion ou le sélectionner depuis votre abonnement.

Avant de démarrer, ouvrez FabrikamFiber.Web->PackageRoot->ServiceManifest.xml dans l’Explorateur de solutions. Notez le port du front-end web listé dans **Point de terminaison**.

Lors de la création du cluster :

1. Cliquez avec le bouton droit sur le projet de l’application **FabrikamFiber.CallCenterApplication** dans l’Explorateur de solutions et choisissez **Publier**.
2. Connectez-vous avec votre compte Azure afin d’avoir accès à votre ou vos abonnements.
3. Sous le menu déroulant pour le **Point de terminaison de connexion**, sélectionnez l’option **Créer un cluster...** .
4. Dans la boîte de dialogue **Créer un cluster**, modifiez les paramètres suivants :

    a. Spécifiez le nom de votre cluster dans le champ **Nom du cluster**, ainsi que l’abonnement et l’emplacement à utiliser. Notez le nom de votre groupe de ressources de cluster.

    b. Facultatif : Vous pouvez modifier le nombre de nœuds. Par défaut, vous disposez de trois nœuds, qui est le minimum requis pour tester des scénarios Service Fabric.

    c. Sélectionnez l’onglet **Certificat**. Dans cet onglet, tapez un mot de passe à utiliser pour sécuriser le certificat de votre cluster. Ce certificat aide à sécuriser votre cluster. Vous pouvez aussi modifier le chemin vers l’emplacement où vous voulez enregistrer le certificat. Visual Studio peut aussi importer le certificat pour vous, car il s’agit d’une étape nécessaire à la publication de l’application dans le cluster.

    d. Sélectionnez l’onglet **Détail de la machine virtuelle**. Spécifiez le mot de passe que vous voulez utiliser pour les machines virtuelles qui forment le cluster. Le nom d’utilisateur et le mot de passe peuvent être utilisés pour se connecter aux machines virtuelles à distance. Vous devez aussi sélectionner une taille de machine virtuelle et pouvez changer l’image de la machine virtuelle si nécessaire.

    > [!IMPORTANT]
    > Choisissez une référence SKU prenant en charge les conteneurs en cours d’exécution. Le système d’exploitation Windows Server sur vos nœuds de cluster doit être compatible avec le système d’exploitation Windows Server de votre conteneur. Pour plus d’informations, consultez [Système d’exploitation Windows Server du conteneur et compatibilité avec le système d’exploitation hôte](service-fabric-get-started-containers.md#windows-server-container-os-and-host-os-compatibility). Par défaut, ce tutoriel crée une image Docker basée sur Windows Server 2016 LTSC. Les conteneurs basés sur cette image seront exécutés sur les clusters créés avec Windows Server 2016 Datacenter avec Containers. Toutefois, si vous créez un cluster ou utilisez un cluster existant basé sur une autre version de Windows Server, vous devez changer l’image du système d’exploitation sur laquelle le conteneur est basé. Ouvrez le **dockerfile** dans le projet **FabrikamFiber.Web**, commentez toute instruction `FROM` existante basée sur une version précédente de Windows Server, puis ajoutez une instruction `FROM` basée sur la balise de la version souhaitée à partir de la [page DockerHub Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore). Pour plus d’informations sur les versions Windows Server Core, les calendriers de prise en charge et la gestion de versions, consultez [Informations de publication de Windows Server Core](/windows-server/get-started/windows-server-release-info). 

    e. Dans l’onglet **Avancé**, indiquez le port de l’application à ouvrir dans l’équilibreur de charge lorsque le cluster est déployé. Il s’agit du port que vous avez noté avant de commencer la création du cluster. Vous pouvez aussi ajouter une clé Application Insights existante à utiliser pour y diriger les fichiers de journal des applications.

    f. Une fois la modification des paramètres terminée, cliquez sur le bouton **Créer**.

5. La création prend plusieurs minutes, la fenêtre de sortie vous informe lorsque le cluster est complètement créé.

## <a name="allow-your-application-running-in-azure-to-access-sql-database"></a>Autoriser votre application s’exécutant dans Azure à accéder à SQL Database

Précédemment, vous avez créé une règle de pare-feu SQL pour autoriser l’accès à votre application exécutée localement.  Ensuite, vous devez autoriser l’application qui s’exécute dans Azure à accéder à la base de données SQL.  Créez un [point de terminaison de service de réseau virtuel](../azure-sql/database/vnet-service-endpoint-rule-overview.md) pour le cluster Service Fabric puis créez une règle pour autoriser ce point de terminaison à accéder à la base de données SQL. Veillez à spécifier la variable de groupe de ressources de cluster que vous avez notée pendant la création du cluster.

```powershell
# Create a virtual network service endpoint
$clusterresourcegroup = "<cluster resource group>"
$resource = Get-AzResource -ResourceGroupName $clusterresourcegroup -ResourceType Microsoft.Network/virtualNetworks | Select-Object -first 1
$vnetName = $resource.Name

Write-Host 'Virtual network name: ' $vnetName

# Get the virtual network by name.
$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName $clusterresourcegroup `
  -Name              $vnetName

Write-Host "Get the subnet in the virtual network:"

# Get the subnet, assume the first subnet contains the Service Fabric cluster.
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet | Select-Object -first 1

$subnetName = $subnet.Name
$subnetID = $subnet.Id
$addressPrefix = $subnet.AddressPrefix

Write-Host "Subnet name: " $subnetName " Address prefix: " $addressPrefix " ID: " $subnetID

# Assign a Virtual Service endpoint 'Microsoft.Sql' to the subnet.
$vnet = Set-AzVirtualNetworkSubnetConfig `
  -Name            $subnetName `
  -AddressPrefix   $addressPrefix `
  -VirtualNetwork  $vnet `
  -ServiceEndpoint Microsoft.Sql | Set-AzVirtualNetwork

$vnet.Subnets[0].ServiceEndpoints;  # Display the first endpoint.

# Add a SQL DB firewall rule for the virtual network service endpoint
$subnet = Get-AzVirtualNetworkSubnetConfig `
  -Name           $subnetName `
  -VirtualNetwork $vnet;

$VNetRuleName="ServiceFabricClusterVNetRule"
$vnetRuleObject1 = New-AzSqlServerVirtualNetworkRule `
  -ResourceGroupName      $dbresourcegroupname `
  -ServerName             $servername `
  -VirtualNetworkRuleName $VNetRuleName `
  -VirtualNetworkSubnetId $subnetID;
```

## <a name="deploy-the-application-to-azure"></a>Déploiement de l’application dans Azure

À présent que l’application est prête, vous pouvez la déployer sur le cluster dans Azure directement à partir de Visual Studio.  Cliquez avec le bouton droit sur le projet de l’application **FabrikamFiber.CallCenterApplication** dans l’Explorateur de solutions et choisissez **Publier**.  Dans **Point de terminaison de connexion**, sélectionnez le point de terminaison du cluster précédemment créé.  Dans **Azure Container Registry**, sélectionnez le registre de conteneur précédemment créé.  Cliquez sur **Publier** pour déployer l’application vers le cluster dans Azure.

![Publication de l’application][publish-app]

Vous pouvez suivre la procession du déploiement dans la fenêtre de sortie. Lorsque l’application est déployée, ouvrez un navigateur et saisissez l’adresse du cluster et le port de l’application. Par exemple : `https://fabrikamfibercallcenter.southcentralus.cloudapp.azure.com:8659/`.

![Capture d’écran de la page d’accueil de l’application Fabrikam Fiber CallCenter s’exécutant sur azure.com. La page montre un tableau de bord avec une liste d’appels de support.][fabrikam-web-page-deployed]

## <a name="set-up-continuous-integration-and-deployment-cicd-with-a-service-fabric-cluster"></a>Configurer l’intégration et le déploiement continus (CI/CD) avec un cluster Service Fabric

Pour savoir comment utiliser Azure DevOps en vue de configurer le déploiement d’applications CI/CD dans un cluster Service Fabric, consultez [Tutoriel : Déployer une application avec intégration et déploiement continus sur un cluster Service Fabric](service-fabric-tutorial-deploy-app-with-cicd-vsts.md). Le processus décrit dans ce didacticiel est identique pour ce projet (FabrikamFiber) ; vous devez simplement omettre le téléchargement de l’exemple Voting et utiliser FabrikamFiber comme nom de référentiel au lieu de Voting.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous avez terminé, veillez à supprimer toutes les ressources créées.  La façon la plus simple est de supprimer les groupes de ressources qui contiennent le cluster Service Fabric, Azure SQL DB et Azure Container Registry.

```powershell
$dbresourcegroupname = "fabrikam-fiber-db-group"
$acrresourcegroupname = "fabrikam-acr-group"
$clusterresourcegroupname="fabrikamcallcentergroup"

# Remove the Azure SQL DB
Remove-AzResourceGroup -Name $dbresourcegroupname

# Remove the container registry
Remove-AzResourceGroup -Name $acrresourcegroupname

# Remove the Service Fabric cluster
Remove-AzResourceGroup -Name $clusterresourcegroupname
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
>
> * Mettre en conteneur une application existante à l’aide de Visual Studio
> * Créer une base de données dans Azure SQL Database
> * Créer un Azure Container Registry
> * Déployer une application Service Fabric sur Azure

Dans la partie suivante du tutoriel, découvrez comment [déployer une application de conteneur avec CI/CD sur un cluster Service Fabric](service-fabric-tutorial-deploy-container-app-with-cicd-vsts.md).

[link-fabrikam-github]: https://aka.ms/fabrikamcontainer
[link-azure-powershell-install]: /powershell/azure/install-Az-ps
[link-servicefabric-create-secure-clusters]: service-fabric-cluster-creation-via-arm.md
[link-visualstudio-cd-extension]: https://aka.ms/cd4vs
[link-servicefabric-containers]: service-fabric-get-started-containers.md
[link-azure-portal]: https://portal.azure.com
[link-sf-clustertemplate]: https://aka.ms/securepreviewonelineclustertemplate
[link-azure-pricing-calculator]: https://azure.microsoft.com/pricing/calculator/
[link-azure-subscription]: https://azure.microsoft.com/free/
[link-vsts-account]: https://www.visualstudio.com/team-services/pricing/
[link-azure-sql]: /azure/sql-database/

[fabrikam-web-page]: media/service-fabric-host-app-in-a-container/fabrikam-web-page.png
[fabrikam-web-page-deployed]: media/service-fabric-host-app-in-a-container/fabrikam-web-page-deployed.png
[publish-app]: media/service-fabric-host-app-in-a-container/publish-app.png