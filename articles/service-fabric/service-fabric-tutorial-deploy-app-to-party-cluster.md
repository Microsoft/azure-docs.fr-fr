---
title: Déployer une application Service Fabric sur un cluster dans Azure
description: Découvrez comment déployer une application existante sur un cluster Azure Service Fabric récemment créé à partir de Visual Studio.
author: athinanthny
ms.topic: tutorial
ms.date: 07/22/2019
ms.author: mikhegn
ms.custom: mvc
ms.openlocfilehash: e35b655dc8b735214de891884fe40fb951dd16cd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91441281"
---
# <a name="tutorial-deploy-a-service-fabric-application-to-a-cluster-in-azure"></a>Tutoriel : Déployer une application Service Fabric sur un cluster dans Azure

Ce tutoriel est le deuxième d’une série. Il vous montre comment déployer une application Azure Service Fabric sur un nouveau cluster dans Azure.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> * Créer un cluster.
> * Déployer une application sur un cluster distant à l’aide de Visual Studio.

Dans cette série de didacticiels, nous allons aborder les points suivants :
> [!div class="checklist"]
> * [Créer une application .NET Service Fabric](service-fabric-tutorial-create-dotnet-app.md).
> * Déployer l’application sur un cluster distant.
> * [Ajouter un point de terminaison HTTPS à un service frontal ASP.NET Core](service-fabric-tutorial-dotnet-app-enable-https-endpoint.md).
> * [Configurer CI/CD à l’aide d’Azure Pipelines](service-fabric-tutorial-deploy-app-with-cicd-vsts.md).
> * [Configurer la surveillance et les diagnostics pour l’application](service-fabric-tutorial-monitoring-aspnet.md).

## <a name="prerequisites"></a>Conditions préalables requises

Avant de commencer ce tutoriel :

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Installez Visual Studio 2019](https://www.visualstudio.com/) et les charges de travail **Développement Azure** et **Développement web et ASP.NET**.
* [Installez le Kit de développement logiciel (SDK) Service Fabric](service-fabric-get-started.md).

> [!NOTE]
> Un compte gratuit peut ne pas répondre aux besoins nécessaires pour créer une machine virtuelle, ce qui empêche le suivi du tutoriel. Par ailleurs, un compte autre que professionnel ou scolaire peut rencontrer des problèmes d’autorisation pour créer le certificat sur le coffre de clés associé au cluster. Si vous rencontrez une erreur liée à la création du certificat, utilisez plutôt le portail pour créer le cluster. 

## <a name="download-the-voting-sample-application"></a>Télécharger l’exemple d’application de vote

Si vous n’avez pas créé l’exemple d’application de vote lors de la [première partie de cette série de didacticiels](service-fabric-tutorial-create-dotnet-app.md), vous pouvez le télécharger. Dans une fenêtre de commande, exécutez le code suivant pour cloner le référentiel de l’exemple d’application sur votre ordinateur local.

```git
git clone https://github.com/Azure-Samples/service-fabric-dotnet-quickstart 
```

Ouvrez l’application dans Visual Studio en tant qu’administrateur, puis générez l’application.

## <a name="create-a-cluster"></a>Créer un cluster

Maintenant que l’application est prête, créez un cluster Service Fabric et déployez l’application sur celui-ci. Un [cluster Service Fabric](./service-fabric-deploy-anywhere.md) est un groupe de machines virtuelles ou physiques connectées au réseau, sur lequel vos microservices sont déployés et gérés.

Dans ce tutoriel, vous créez un cluster de test de trois nœuds dans l’IDE Visual Studio, puis vous publiez l’application sur ce cluster. Pour plus d’informations sur la création d’un cluster de production, consultez le [tutoriel sur la création et la gestion d’un cluster](service-fabric-tutorial-create-vnet-and-windows-cluster.md). Vous pouvez également déployer l’application sur un cluster existant que vous avez précédemment créé dans le [portail Azure](https://portal.azure.com), à l’aide de scripts [PowerShell](./scripts/service-fabric-powershell-create-secure-cluster-cert.md) ou [Azure CLI](./scripts/cli-create-cluster.md), ou à partir d’un [modèle azure Resource Manager](service-fabric-tutorial-create-vnet-and-windows-cluster.md).

> [!NOTE]
> L’application Voting et bon nombre d’autres applications utilisent le proxy inverse Service Fabric pour communiquer entre les services. Le proxy inverse est activé par défaut sur les clusters créés à partir de Visual Studio. Si vous procédez au déploiement sur un cluster existant, vous devez [activer le proxy inverse dans le cluster](service-fabric-reverseproxy-setup.md) pour que l’application Voting fonctionne.


### <a name="find-the-votingweb-service-endpoint"></a>Rechercher le point de terminaison de service VotingWeb

Le service web front-end de l’application Voting écoute sur un port spécifique (8080 si vous avez suivi les étapes décrites dans la [première partie de cette série de tutoriels](service-fabric-tutorial-create-dotnet-app.md)). Lorsque l’application est déployée sur un cluster dans Azure, le cluster et l’application s’exécutent derrière un équilibreur de charge Azure. Le port de l’application doit être ouvert dans l’équilibreur de charge Azure à l’aide d’une règle. La règle envoie le trafic entrant au service web par le biais de l’équilibreur de charge. Le port se trouve dans le fichier **VotingWeb/PackageRoot/ServiceManifest.xml** de l’élément **point de terminaison**. 

```xml
<Endpoint Protocol="http" Name="ServiceEndpoint" Type="Input" Port="8080" />
```

Prenez note du point de terminaison de service, car vous en aurez besoin dans une étape ultérieure.  Si vous procédez au déploiement sur un cluster existant, ouvrez ce port en créant une sonde et une règle d’équilibrage de charge dans l’équilibreur de charge Azure à l’aide d’un [script PowerShell](./scripts/service-fabric-powershell-open-port-in-load-balancer.md) ou par le biais de l’équilibreur de charge pour ce cluster dans le [portail Azure](https://portal.azure.com).

### <a name="create-a-test-cluster-in-azure"></a>Créer un cluster de test dans Azure
Dans l’Explorateur de solutions, cliquez avec le bouton droit sur **Voting**, puis sélectionnez **Publier**.

Dans **Point de terminaison de connexion**, sélectionnez **Créer un cluster**.  Si vous procédez au déploiement sur un cluster existant, sélectionnez le point de terminaison de cluster dans la liste.  La boîte de dialogue Créer un cluster Service Fabric s’ouvre.

Sous l’onglet **Cluster**, entrez le **Nom du cluster** (par exemple, « mytestcluster »), sélectionnez votre abonnement, choisissez une région pour le cluster (par exemple, USA Centre Sud), entrez le nombre de nœuds de cluster (nous recommandons trois nœuds pour un cluster de test), puis entrez un groupe de ressources (par exemple, « mytestclustergroup »). Cliquez sur **Suivant**.

![La capture d’écran montre l’onglet Cluster de la boîte de dialogue Créer un cluster Service Fabric.](./media/service-fabric-tutorial-deploy-app-to-party-cluster/create-cluster.png)

Sous l’onglet **Certificat**, entrez le mot de passe et le chemin de sortie du certificat de cluster. Un certificat auto-signé est créé en tant que fichier PFX et enregistré dans le chemin de sortie spécifié.  Le certificat est utilisé pour la sécurité nœud à nœud et client à nœud.  N’utilisez pas un certificat auto-signé pour les clusters de production.  Ce certificat est utilisé par Visual Studio pour l’authentification auprès du cluster et le déploiement d’applications. Sélectionnez **Importer un certificat** pour installer le fichier PFX dans le magasin de certificats CurrentUser\My de votre ordinateur.  Cliquez sur **Suivant**.

![La capture d’écran montre l’onglet Certificat de la boîte de dialogue Créer un cluster Service Fabric.](./media/service-fabric-tutorial-deploy-app-to-party-cluster/certificate.png)

Sous l’onglet **Détail de la machine virtuelle**, entrez le **Nom d’utilisateur** et le **Mot de passe** du compte administrateur du cluster.  Sélectionnez l’**Image de machine virtuelle** pour les nœuds de cluster et la **Taille de machine virtuelle** pour chaque nœud du cluster.  Cliquez sur l’onglet **Avancé**.

![La capture d’écran montre l’onglet Détail de la machine virtuelle de la boîte de dialogue Créer un cluster Service Fabric.](./media/service-fabric-tutorial-deploy-app-to-party-cluster/vm-detail.png)

Dans **Ports**, entrez le point de terminaison de service VotingWeb de l’étape précédente (par exemple, 8080).  Une fois le cluster créé, ces ports d’application sont ouverts dans l’équilibreur de charge Azure pour transférer le trafic vers le cluster.  Cliquez sur **Créer** pour créer le cluster, ce qui prend plusieurs minutes.

![La capture d’écran montre l’onglet Avancé de la boîte de dialogue Créer un cluster Service Fabric.](./media/service-fabric-tutorial-deploy-app-to-party-cluster/advanced.png)

## <a name="publish-the-application-to-the-cluster"></a>Publier l’application sur le cluster

Quand le nouveau cluster est prêt, vous pouvez déployer l’application Voting directement à partir de Visual Studio.

Dans l’Explorateur de solutions, cliquez avec le bouton droit sur **Voting**, puis sélectionnez **Publier**. La boîte de dialogue **Publier** s’affiche.

Dans **Point de terminaison de connexion**, sélectionnez le point de terminaison du cluster créé à l’étape précédente.  Par exemple, « mytestcluster.southcentral.cloudapp.azure.com:19000 ». Si vous sélectionnez **Paramètres de connexion avancés**, les informations de certificat doivent être renseignées automatiquement.  
![Publier une application Service Fabric](./media/service-fabric-tutorial-deploy-app-to-party-cluster/publish-app.png)

Sélectionnez **Publier**.

Une fois l’application déployée, ouvrez un navigateur et entrez l’adresse du cluster suivie de  **:8080**. Vous pouvez également entrer un autre port, si configuré. par exemple `http://mytestcluster.southcentral.cloudapp.azure.com:8080`. Vous voyez l’application en cours d’exécution dans le cluster dans Azure. Dans la page web de vote, essayez d’ajouter ou de supprimer des options de vote et de voter pour une ou plusieurs de ces options.

![Exemple de vote Service Fabric](./media/service-fabric-tutorial-deploy-app-to-party-cluster/application-screenshot-new-azure.png)


## <a name="next-steps"></a>Étapes suivantes
Dans cette partie du tutoriel, vous avez appris à :

> [!div class="checklist"]
> * Créer un cluster.
> * Déployer une application sur un cluster distant à l’aide de Visual Studio.

Passez au tutoriel suivant :
> [!div class="nextstepaction"]
> [Activer HTTPS](service-fabric-tutorial-dotnet-app-enable-https-endpoint.md)
