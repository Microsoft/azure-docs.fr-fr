---
title: Ajouter un point de terminaison HTTPS à l’aide de Kestrel
description: Dans le cadre de ce didacticiel, vous allez apprendre à ajouter un point de terminaison HTTPS à un service web frontal ASP.NET Core à l’aide de Kestrel et à déployer l’application sur un cluster.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc, devx-track-csharp
ms.openlocfilehash: c675f8ece8369bcfc0055343221ac82aea59dec1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91326233"
---
# <a name="tutorial-add-an-https-endpoint-to-an-aspnet-core-web-api-front-end-service-using-kestrel"></a>Tutoriel : Ajouter un point de terminaison HTTPS à un service frontal API Web ASP.NET Core à l’aide de Kestrel

Ce tutoriel est le troisième de la série.  Vous allez apprendre à activer HTTPS dans un service ASP.NET Core s’exécutant sur Service Fabric. À l’issue de cette procédure, vous disposerez d’une application de vote avec un service web frontal ASP.NET Core HTTPS écoutant le port 443. Si vous ne souhaitez pas créer l’application de vote manuellement en suivant les instructions de l’article [Créer une application .NET Service Fabric](service-fabric-tutorial-deploy-app-to-party-cluster.md), vous pouvez [télécharger le code source](https://github.com/Azure-Samples/service-fabric-dotnet-quickstart/) pour obtenir l’application terminée.

Dans ce troisième volet, vous apprenez à :

> [!div class="checklist"]
> * Définir un point de terminaison HTTPS dans le service
> * Configurer Kestrel pour l’utilisation de HTTPS
> * Installer le certificat TLS/SSL sur les nœuds du cluster distant
> * Accorder l’accès SERVICE RÉSEAU à la clé privée du certificat
> * Ouvrir le port 443 dans l’équilibreur de charge Azure
> * Déployer l’application sur un cluster distant

Cette série de tutoriels vous montre comment effectuer les opérations suivantes :
> [!div class="checklist"]
> * [Créer une application .NET Service Fabric](service-fabric-tutorial-deploy-app-to-party-cluster.md)
> * [Déployer l’application sur un cluster distant](service-fabric-tutorial-deploy-app-to-party-cluster.md)
> * Ajouter un point de terminaison HTTPS à un service frontal ASP.NET Core
> * [Configurer CI/CD à l’aide d’Azure Pipelines](service-fabric-tutorial-deploy-app-with-cicd-vsts.md)
> * [Configurer la surveillance et les diagnostics pour l’application](service-fabric-tutorial-monitoring-aspnet.md)


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel :

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* [Installez Visual Studio 2019](https://www.visualstudio.com/) version 16.5 ou ultérieure avec les charges de travail **Développement Azure** et **Développement web et ASP.NET**.
* [Installez le Kit de développement logiciel (SDK) Service Fabric](service-fabric-get-started.md)

## <a name="obtain-a-certificate-or-create-a-self-signed-development-certificate"></a>Obtenir un certificat ou créer un certificat de développement auto-signé

Pour les applications de production, utilisez un certificat délivré par une [autorité de certification (AC)](https://wikipedia.org/wiki/Certificate_authority). Dans le cadre d’opérations de développement et de test, vous pouvez créer et utiliser un certificat auto-signé. Le Kit de développement logiciel (SDK) Service Fabric fournit le script *CertSetup.ps1*, qui crée un certificat auto-signé et l’importe dans le magasin de certificats `Cert:\LocalMachine\My`. Ouvrez une invite de commandes en tant qu’administrateur, puis exécutez la commande ci-après pour créer un certificat avec le sujet « CN=mytestcert » :

```powershell
PS C:\program files\microsoft sdks\service fabric\clustersetup\secure> .\CertSetup.ps1 -Install -CertSubjectName CN=mytestcert
```

Si vous disposez déjà d’un fichier PFX de certificat, exécutez la commande ci-après pour importer le certificat dans le magasin de certificats `Cert:\LocalMachine\My` :

```powershell

PS C:\mycertificates> Import-PfxCertificate -FilePath .\mysslcertificate.pfx -CertStoreLocation Cert:\LocalMachine\My -Password (ConvertTo-SecureString "!Passw0rd321" -AsPlainText -Force)


   PSParentPath: Microsoft.PowerShell.Security\Certificate::LocalMachine\My

Thumbprint                                Subject
----------                                -------
3B138D84C077C292579BA35E4410634E164075CD  CN=zwin7fh14scd.westus.cloudapp.azure.com
```

## <a name="define-an-https-endpoint-in-the-service-manifest"></a>Définir un point de terminaison HTTPS dans le manifeste de service

Lancez Visual Studio en tant **qu’administrateur** et ouvrez la solution de vote. Dans l’Explorateur de solutions, ouvrez *VotingWeb/PackageRoot/ServiceManifest.xml*. Le manifeste de service définit les points de terminaison de service.  Recherchez la section **Endpoints**, modifiez le point de terminaison "ServiceEndpoint" existant  en le remplaçant par "EndpointHttps", puis définissez le protocole sur *https*, le type sur *Input* et le port sur *443*.  Enregistrez vos modifications.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="VotingWebPkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <StatelessServiceType ServiceTypeName="VotingWebType" />
  </ServiceTypes>

  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <ExeHost>
        <Program>VotingWeb.exe</Program>
        <WorkingFolder>CodePackage</WorkingFolder>
      </ExeHost>
    </EntryPoint>
  </CodePackage>

  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <Endpoint Protocol="https" Name="EndpointHttps" Type="Input" Port="443" />
    </Endpoints>
  </Resources>
</ServiceManifest>
```

## <a name="configure-kestrel-to-use-https"></a>Configurer Kestrel pour l’utilisation de HTTPS

Dans l’Explorateur de solutions, ouvrez le fichier *VotingWeb/VotingWeb.cs*.  Configurez Kestrel pour l’utilisation de HTTPS et recherchez le certificat dans le magasin `Cert:\LocalMachine\My`. Ajoutez les instructions using suivantes :

```csharp
using System.Net;
using Microsoft.Extensions.Configuration;
using System.Security.Cryptography.X509Certificates;
```

Mettez à jour l’élément `ServiceInstanceListener` pour qu’il utilise le nouveau point de terminaison *EndpointHttps* et qu’il écoute le port 443. Lors de la configuration de l’hôte web pour utiliser le serveur Kestrel, vous devez configurer Kestrel pour qu’il écoute les adresses IPv6 sur toutes les interfaces réseau : `opt.Listen(IPAddress.IPv6Any, port, listenOptions => {...}`.

```csharp
new ServiceInstanceListener(
serviceContext =>
    new KestrelCommunicationListener(
        serviceContext,
        "EndpointHttps",
        (url, listener) =>
        {
            ServiceEventSource.Current.ServiceMessage(serviceContext, $"Starting Kestrel on {url}");

            return new WebHostBuilder()
                .UseKestrel(opt =>
                {
                    int port = serviceContext.CodePackageActivationContext.GetEndpoint("EndpointHttps").Port;
                    opt.Listen(IPAddress.IPv6Any, port, listenOptions =>
                    {
                        listenOptions.UseHttps(FindMatchingCertificateBySubject());
                        listenOptions.NoDelay = true;
                    });
                })
                .ConfigureAppConfiguration((builderContext, config) =>
                {
                    config.AddJsonFile("appsettings.json", optional: false, reloadOnChange: true);
                })

                .ConfigureServices(
                    services => services
                        .AddSingleton<HttpClient>(new HttpClient())
                        .AddSingleton<FabricClient>(new FabricClient())
                        .AddSingleton<StatelessServiceContext>(serviceContext))
                .UseContentRoot(Directory.GetCurrentDirectory())
                .UseStartup<Startup>()
                .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.None)
                .UseUrls(url)
                .Build();
        }))
```

Ajoutez également la méthode ci-après pour permettre à Kestrel de trouver le certificat dans le magasin `Cert:\LocalMachine\My` au moyen du sujet.  

Remplacez la chaîne « &lt;your_CN_value&gt; » par « mytestcert » si vous avez créé un certificat auto-signé avec la commande PowerShell précédente, ou utilisez le CN de votre certificat.
Sachez que dans le cas d’un déploiement local vers `localhost`, il est préférable d’utiliser « CN=localhost » pour éviter les exceptions d’authentification.

```csharp
private X509Certificate2 FindMatchingCertificateBySubject(string subjectCommonName)
{
    using (var store = new X509Store(StoreName.My, StoreLocation.LocalMachine))
    {
        store.Open(OpenFlags.OpenExistingOnly | OpenFlags.ReadOnly);
        var certCollection = store.Certificates;
        var matchingCerts = new X509Certificate2Collection();
    
    foreach (var enumeratedCert in certCollection)
    {
      if (StringComparer.OrdinalIgnoreCase.Equals(subjectCommonName, enumeratedCert.GetNameInfo(X509NameType.SimpleName, forIssuer: false))
        && DateTime.Now < enumeratedCert.NotAfter
        && DateTime.Now >= enumeratedCert.NotBefore)
        {
          matchingCerts.Add(enumeratedCert);
        }
    }

        if (matchingCerts.Count == 0)
    {
        throw new Exception($"Could not find a match for a certificate with subject 'CN={subjectCommonName}'.");
    }
        
        return matchingCerts[0];
    }
}


```

## <a name="grant-network-service-access-to-the-certificates-private-key"></a>Accorder l’accès SERVICE RÉSEAU à la clé privée du certificat

Au cours d’une étape précédente, vous avez importé le certificat dans le magasin `Cert:\LocalMachine\My` sur l’ordinateur de développement.  Maintenant, octroyez explicitement au compte exécutant le service (SERVICE RÉSEAU, par défaut) l’accès à la clé privée du certificat. Vous pouvez effectuer cette étape manuellement (à l’aide de l’outil certlm.msc), mais il est préférable d’exécuter automatiquement un script PowerShell en [configurant un script de démarrage](service-fabric-run-script-at-service-startup.md) dans l’élément **SetupEntryPoint** du manifeste de service.

>[!NOTE]
> Service Fabric prend en charge la déclaration de certificats de point de terminaison par empreinte ou par nom commun de sujet. Dans ce cas, le runtime configure la liaison et définit dans la liste de contrôle d’accès une entrée qui lie la clé privée du certificat à l’identité sous laquelle le service est en cours d’exécution. En outre, le runtime supervise le certificat pour savoir s’il doit faire l’objet de modifications ou d’un renouvellement, et redéfinit en conséquence la clé privée correspondante dans la liste de contrôle d’accès.

### <a name="configure-the-service-setup-entry-point"></a>Configurer le point d’entrée d’installation du service

Dans l’Explorateur de solutions, ouvrez *VotingWeb/PackageRoot/ServiceManifest.xml*.  Dans la section **CodePackage**, ajoutez le nœud **SetupEntryPoint**, puis un nœud **ExeHost**.  Dans **ExeHost**, définissez **Program** sur « Setup.bat » et **WorkingFolder** sur « CodePackage ».  Au démarrage du service VotingWeb, le script Setup.bat s’exécute dans le dossier CodePackage avant que VotingWeb.exe ne démarre.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="VotingWebPkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <StatelessServiceType ServiceTypeName="VotingWebType" />
  </ServiceTypes>

  <CodePackage Name="Code" Version="1.0.0">
    <SetupEntryPoint>
      <ExeHost>
        <Program>Setup.bat</Program>
        <WorkingFolder>CodePackage</WorkingFolder>
      </ExeHost>
    </SetupEntryPoint>

    <EntryPoint>
      <ExeHost>
        <Program>VotingWeb.exe</Program>
        <WorkingFolder>CodePackage</WorkingFolder>
      </ExeHost>
    </EntryPoint>
  </CodePackage>

  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <Endpoint Protocol="https" Name="EndpointHttps" Type="Input" Port="443" />
    </Endpoints>
  </Resources>
</ServiceManifest>
```

### <a name="add-the-batch-and-powershell-setup-scripts"></a>Ajouter le fichier de commandes et les scripts d’installation PowerShell

Pour exécuter PowerShell à partir du point **SetupEntryPoint**, vous pouvez exécuter PowerShell.exe dans un fichier de commandes qui pointe vers un fichier PowerShell. Commencez par ajouter le fichier de commandes au projet de service.  Dans l’Explorateur de solutions, cliquez avec le bouton droit sur **VotingWeb** et sélectionnez **Ajouter**->**Nouvel élément**, puis ajoutez un nouveau fichier nommé « Setup.bat ».  Modifiez le fichier *Setup.bat* et ajoutez la commande suivante :

```cmd
powershell.exe -ExecutionPolicy Bypass -Command ".\SetCertAccess.ps1"
```

Modifiez les propriétés du fichier *Setup.bat* en définissant **Copier dans le répertoire de sortie** sur « Copier si plus récent ».

![Définir les propriétés du fichier][image1]

Dans l’Explorateur de solutions, cliquez avec le bouton droit sur **VotingWeb** et sélectionnez **Ajouter**->**Nouvel élément**, puis ajoutez un nouveau fichier nommé « SetCertAccess.ps1 ».  Modifiez le fichier *SetCertAccess.ps1*, puis ajoutez le script suivant :

```powershell
$subject="mytestcert"
$userGroup="NETWORK SERVICE"

Write-Host "Checking permissions to certificate $subject.." -ForegroundColor DarkCyan

$cert = (gci Cert:\LocalMachine\My\ | where { $_.Subject.Contains($subject) })[-1]

if ($cert -eq $null)
{
    $message="Certificate with subject:"+$subject+" does not exist at Cert:\LocalMachine\My\"
    Write-Host $message -ForegroundColor Red
    exit 1;
}elseif($cert.HasPrivateKey -eq $false){
    $message="Certificate with subject:"+$subject+" does not have a private key"
    Write-Host $message -ForegroundColor Red
    exit 1;
}else
{
    $keyName=$cert.PrivateKey.CspKeyContainerInfo.UniqueKeyContainerName

    $keyPath = "C:\ProgramData\Microsoft\Crypto\RSA\MachineKeys\"
    $fullPath=$keyPath+$keyName
    $acl=(Get-Item $fullPath).GetAccessControl('Access')


    $hasPermissionsAlready = ($acl.Access | where {$_.IdentityReference.Value.Contains($userGroup.ToUpperInvariant()) -and $_.FileSystemRights -eq [System.Security.AccessControl.FileSystemRights]::FullControl}).Count -eq 1

    if ($hasPermissionsAlready){
        Write-Host "Account $userGroup already has permissions to certificate '$subject'." -ForegroundColor Green
        return $false;
    } else {
        Write-Host "Need add permissions to '$subject' certificate..." -ForegroundColor DarkYellow

        $permission=$userGroup,"Full","Allow"
        $accessRule=new-object System.Security.AccessControl.FileSystemAccessRule $permission
        $acl.AddAccessRule($accessRule)
        Set-Acl $fullPath $acl

        Write-Output "Permissions were added"

        return $true;
    }
}

```

Modifiez les propriétés du fichier *SetCertAccess.ps1* en définissant **Copier dans le répertoire de sortie** sur « Copier si plus récent ».

### <a name="run-the-setup-script-as-a-local-administrator"></a>Exécuter le script d’installation an tant qu’administrateur local

Par défaut, l’exécutable du point d’entrée d’installation du service est exécuté avec les mêmes informations d’identification que Service Fabric (généralement, le compte NetworkService). Le fichier *SetCertAccess.ps1* requiert des privilèges d’administrateur. Dans le manifeste de l’application, vous pouvez changer les autorisations de sécurité de manière à exécuter le script de démarrage sous un compte d’administrateur local.

Dans l’Explorateur de solutions, ouvrez *Voting/ApplicationPackageRoot/ApplicationManifest.xml*. Commencez par créer une section **Principals** et par y ajouter un nouvel utilisateur (par exemple, "SetupAdminUser"). Ajoutez le compte d’utilisateur SetupAdminUser au groupe des administrateurs système.
Ensuite, dans la section VotingWebPkg **ServiceManifestImport**, configurez un élément **RunAsPolicy** pour appliquer le principal SetupAdminUser au point d’entrée d’installation. Cette stratégie indique à Service Fabric que le fichier Setup.bat s’exécute en tant que SetupAdminUser (avec des privilèges d’administrateur).

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="VotingType" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="VotingData_MinReplicaSetSize" DefaultValue="3" />
    <Parameter Name="VotingData_PartitionCount" DefaultValue="1" />
    <Parameter Name="VotingData_TargetReplicaSetSize" DefaultValue="3" />
    <Parameter Name="VotingWeb_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="VotingDataPkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
  </ServiceManifestImport>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="VotingWebPkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <RunAsPolicy CodePackageRef="Code" UserRef="SetupAdminUser" EntryPointType="Setup" />
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <Service Name="VotingData">
      <StatefulService ServiceTypeName="VotingDataType" TargetReplicaSetSize="[VotingData_TargetReplicaSetSize]" MinReplicaSetSize="[VotingData_MinReplicaSetSize]">
        <UniformInt64Partition PartitionCount="[VotingData_PartitionCount]" LowKey="0" HighKey="25" />
      </StatefulService>
    </Service>
    <Service Name="VotingWeb" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="VotingWebType" InstanceCount="[VotingWeb_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
  <Principals>
    <Users>
      <User Name="SetupAdminUser">
        <MemberOf>
          <SystemGroup Name="Administrators" />
        </MemberOf>
      </User>
    </Users>
  </Principals>
</ApplicationManifest>
```

## <a name="run-the-application-locally"></a>Exécuter l’application localement

Dans l’Explorateur de solutions, sélectionnez l’application **Voting** et définissez la propriété **URL de l’application** sur « https:\//localhost:443 ».

Enregistrez tous les fichiers, puis appuyez sur la touche F5 pour exécuter l’application localement.  Une fois l’application déployée, un navigateur web s’ouvre en accédant à l’adresse https:\//localhost:443. Si vous utilisez un certificat auto-signé, vous obtenez un message d’avertissement signalant que votre PC n’a pas confiance en la sécurité de ce site web.  Poursuivez sur la page web.

![Capture d’écran de l’exemple d’application Voting Service Fabric s’exécutant dans une fenêtre de navigateur avec l’URL https://localhost/.][image2]

## <a name="install-certificate-on-cluster-nodes"></a>Installer le certificat sur les nœuds de cluster

Avant de déployer l’application dans Azure, installez le certificat dans le magasin `Cert:\LocalMachine\My` de tous les nœuds du cluster distant.  Vous pouvez déplacer les services d’un nœud à l’autre du cluster.  Lorsque le service web frontal démarrera sur un nœud de cluster, le script de démarrage recherchera le certificat et configurera les autorisations d’accès.

Commencez par exporter le certificat dans un fichier PFX. Ouvrez l’application certlm.msc, puis accédez à **Personnel**>**Certificats**.  Cliquez avec le bouton droit sur le certificat *mytestcert*, puis sélectionnez **Toutes les tâches**>**Exporter**.

![Exportation du certificat][image4]

Dans l’Assistant Exportation, choisissez **Oui, exporter la clé privée** et choisissez le format PFX (Personal Information Exchange, Échange d’informations personnelles).  Exportez le fichier vers *C:\Users\sfuser\votingappcert.pfx*.

Ensuite, installez le certificat sur le cluster distant à l’aide des [scripts PowerShell fournis](./scripts/service-fabric-powershell-add-application-certificate.md).

> [!Warning]
> Un certificat auto-signé suffit pour les applications de développement et de test. Pour les applications de production, utilisez un certificat délivré par une [autorité de certification (AC)](https://wikipedia.org/wiki/Certificate_authority) plutôt qu’un certificat auto-signé.

## <a name="open-port-443-in-the-azure-load-balancer-and-virtual-network"></a>Ouvrir le port 443 dans l’équilibreur de charge et le réseau virtuel Azure

Ouvrez le port 443 dans l’équilibreur de charge s’il n’est pas encore ouvert.

```powershell
$probename = "AppPortProbe6"
$rulename="AppPortLBRule6"
$RGname="voting_RG"
$port=443

# Get the load balancer resource
$resource = Get-AzResource | Where {$_.ResourceGroupName –eq $RGname -and $_.ResourceType -eq "Microsoft.Network/loadBalancers"}
$slb = Get-AzLoadBalancer -Name $resource.Name -ResourceGroupName $RGname

# Add a new probe configuration to the load balancer
$slb | Add-AzLoadBalancerProbeConfig -Name $probename -Protocol Tcp -Port $port -IntervalInSeconds 15 -ProbeCount 2

# Add rule configuration to the load balancer
$probe = Get-AzLoadBalancerProbeConfig -Name $probename -LoadBalancer $slb
$slb | Add-AzLoadBalancerRuleConfig -Name $rulename -BackendAddressPool $slb.BackendAddressPools[0] -FrontendIpConfiguration $slb.FrontendIpConfigurations[0] -Probe $probe -Protocol Tcp -FrontendPort $port -BackendPort $port

# Set the goal state for the load balancer
$slb | Set-AzLoadBalancer
```

Faites de même pour le réseau virtuel associé.

```powershell
$rulename="allowAppPort$port"
$nsgname="voting-vnet-security"
$RGname="voting_RG"
$port=443

# Get the NSG resource
$nsg = Get-AzNetworkSecurityGroup -Name $nsgname -ResourceGroupName $RGname

# Add the inbound security rule.
$nsg | Add-AzNetworkSecurityRuleConfig -Name $rulename -Description "Allow app port" -Access Allow `
    -Protocol * -Direction Inbound -Priority 3891 -SourceAddressPrefix "*" -SourcePortRange * `
    -DestinationAddressPrefix * -DestinationPortRange $port

# Update the NSG.
$nsg | Set-AzNetworkSecurityGroup
```

## <a name="deploy-the-application-to-azure"></a>Déploiement de l’application dans Azure

Enregistrez tous les fichiers, basculez du mode Débogage vers le mode Mise en production, puis appuyez sur la touche F6 pour régénérer l’application.  Dans l’Explorateur de solutions, cliquez avec le bouton droit sur **Voting**, puis sélectionnez **Publier**. Sélectionnez le point de terminaison de connexion du cluster créé au cours du didacticiel [Déployer une application sur un cluster](service-fabric-tutorial-deploy-app-to-party-cluster.md), ou sélectionnez un autre cluster.  Cliquez sur **Publier** pour publier l’application sur le cluster distant.

Lors du déploiement de l’application, ouvrez un navigateur web et accédez à `https://mycluster.region.cloudapp.azure.com:443` (mettez à jour l’URL avec le point de terminaison de connexion de votre cluster). Si vous utilisez un certificat auto-signé, vous obtenez un message d’avertissement signalant que votre PC n’a pas confiance en la sécurité de ce site web.  Poursuivez sur la page web.

![Capture d’écran de l’exemple d’application Voting Service Fabric s’exécutant dans une fenêtre de navigateur avec l’URL https://mycluster.region.cloudapp.azure.com:443.][image3]

## <a name="next-steps"></a>Étapes suivantes

Dans cette partie du tutoriel, vous avez appris à :

> [!div class="checklist"]
> * Définir un point de terminaison HTTPS dans le service
> * Configurer Kestrel pour l’utilisation de HTTPS
> * Installer le certificat TLS/SSL sur les nœuds du cluster distant
> * Accorder l’accès SERVICE RÉSEAU à la clé privée du certificat
> * Ouvrir le port 443 dans l’équilibreur de charge Azure
> * Déployer l’application sur un cluster distant

Passez au tutoriel suivant :
> [!div class="nextstepaction"]
> [Configurer CI/CD à l’aide d’Azure Pipelines](service-fabric-tutorial-deploy-app-with-cicd-vsts.md)

[image1]: ./media/service-fabric-tutorial-dotnet-app-enable-https-endpoint/SetupBatProperties.png
[image2]: ./media/service-fabric-tutorial-dotnet-app-enable-https-endpoint/VotingAppLocal.png
[image3]: ./media/service-fabric-tutorial-dotnet-app-enable-https-endpoint/VotingAppAzure.png
[image4]: ./media/service-fabric-tutorial-dotnet-app-enable-https-endpoint/ExportCert.png
