---
title: Créer et publier une application .Net Core sur un cluster Linux distant
description: Créer et publier des applications .NET Core ciblant un cluster Linux distant à partir de Visual Studio
author: peterpogorski
ms.topic: troubleshooting
ms.date: 5/20/2019
ms.author: pepogors
ms.openlocfilehash: c30eedb6782e4172d677f16e27441f28c78cdd89
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75614347"
---
# <a name="use-visual-studio-to-create-and-publish-net-core-applications-targeting-a-remote-linux-service-fabric-cluster"></a>Utiliser Visual Studio pour créer et publier des applications .NET Core ciblant un cluster Service Fabric Linux distant
Avec les outils Visual Studio, vous pouvez développer et publier des applications .NET Core Service Fabric ciblant un cluster Service Fabric Linux. La version du SDK doit être 3.4 ou ultérieure pour déployer une application .NET Core ciblant des clusters Service Fabric Linux à partir de Visual Studio.

> [!Note]
> Visual Studio ne prend pas en charge le débogage des applications Service Fabric qui ciblent Linux.
>

## <a name="create-a-service-fabric-application-targeting-net-core"></a>Créer une application Service Fabric ciblant .NET Core
1. Lancez Visual Studio en tant qu’**administrateur**.
2. Créez un projet en sélectionnant **Fichier->Nouveau->Projet**.
3. Dans la boîte de dialogue **Nouveau projet**, sélectionnez **Cloud->Application Service Fabric**.
![create-application]
4. Nommez l’application, puis cliquez sur **OK**.
5. Dans la page **Nouveau service Service Fabric**, sélectionnez le type de service que vous souhaitez créer sous la **section .NET Core**.
![create-service]

## <a name="deploy-to-a-remote-linux-cluster"></a>Déployer sur un cluster Linux distant
1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur l’application, puis sélectionnez **Générer**.
![build-application]
2. Une fois le processus de génération pour l’application terminé, cliquez avec le bouton droit sur le service et choisissez de modifier le **fichier csproj**.
![edit-csproj]
3. Remplacez la valeur de la propriété UpdateServiceFabricManifestEnabled True par **False** si le service est un **type de projet d’acteur**. Si votre application n’a pas de service d’acteur, passez à l’étape 4.
```xml
    <UpdateServiceFabricManifestEnabled>False</UpdateServiceFabricManifestEnabled>
```
> [!Note]
> Si vous affectez à UpdateServiceFabricManifestEnabled la valeur False, vous désactivez les mises à jour de ServiceManifest.xml pendant une génération. Aucune modification apportée au service (ajout, suppression ou renommage) n’est répercutée dans le fichier ServiceManifest.xml. Si des modifications sont apportées, vous devez soit mettre à jour le fichier ServiceManifest manuellement ou affecter temporairement à UpdateServiceFabricManifestEnabled la valeur True et générer le service qui mettra à jour le fichier ServiceManifest.xml, puis rétablir la valeur False.
>

4. Mettez à jour RuntimeIdentifier à partir de win7-x64 vers la plateforme cible dans le projet de service.
```xml
    <RuntimeIdentifier>ubuntu.16.04-x64</RuntimeIdentifier>
```
5. Dans le fichier ServiceManifest, mettez à jour le programme entrypoint pour supprimer .exe. 
```xml
    <EntryPoint> 
    <ExeHost> 
        <Program>Actor1</Program> 
    </ExeHost> 
    </EntryPoint>
```
6. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur l’application, puis sélectionnez **Publier**. La boîte de dialogue **Publier** s’affiche.
7. Dans **Point de terminaison de connexion**, sélectionnez le point de terminaison pour le cluster Service Fabric Linux distant que vous souhaitez cibler.
![publish-application]

<!--Image references-->
[create-application]:./media/service-fabric-how-to-vs-remote-linux-cluster/create-application-remote-linux.png
[create-service]:./media/service-fabric-how-to-vs-remote-linux-cluster/create-service-remote-linux.png
[build-application]:./media/service-fabric-how-to-vs-remote-linux-cluster/build-application-remote-linux.png
[edit-csproj]:./media/service-fabric-how-to-vs-remote-linux-cluster/edit-csproj-remote-linux.png
[publish-application]:./media/service-fabric-how-to-vs-remote-linux-cluster/publish-remote-linux.png

## <a name="next-steps"></a>Étapes suivantes
* Découvrir comment [Bien démarrer avec Service Fabric en utilisant .NET Core](https://azure.microsoft.com/resources/samples/service-fabric-dotnet-core-getting-started/)
