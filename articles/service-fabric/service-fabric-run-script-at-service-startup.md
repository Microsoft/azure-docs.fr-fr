---
title: Exécuter un script au démarrage d’un service Azure Service Fabric
description: Découvrez comment configurer une stratégie pour un point d’entrée d’installation du service Service Fabric, et comment exécuter un script au démarrage du service.
author: athinanthny
ms.topic: conceptual
ms.date: 03/21/2018
ms.author: atsenthi
ms.openlocfilehash: a25f16f08ab8ae9564363f179d19d4b30c5315fa
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96012525"
---
# <a name="run-a-service-startup-script-as-a-local-user-or-system-account"></a>Exécuter un script de démarrage du service en tant qu’utilisateur ou compte système local
Avant de démarrer l’exécutable du service Service Fabric, il peut être nécessaire d’effectuer certaines tâches de configuration.  Par exemple, configurer les variables d’environnement. Dans le manifeste du service, vous pouvez spécifier un script qui doit être exécuté avant le démarrage de l’exécutable du service. En configurant une stratégie « RunAs » pour le point d’entrée d’installation du service, vous pouvez changer le compte sous lequel l’exécutable d’installation doit être exécuté.  Un point d’entrée d’installation distinct vous permet d’exécuter une configuration à privilèges élevés pour une courte période. Ainsi, l’exécutable de l’hôte de service n’a pas besoin d’être exécuté avec des privilèges élevés pendant de longues périodes.

Le point d’entrée d’installation (**SetupEntryPoint** dans le [manifeste de service](service-fabric-application-and-service-manifests.md)) est un point d’entrée privilégié qui, par défaut, s’exécute avec les mêmes informations d’identification que Service Fabric (en général, le compte *NetworkService*) avant tout autre point d’entrée. Le fichier exécutable spécifié par **EntryPoint** est généralement l’hôte de service à exécution longue. L’exécutable **EntryPoint** est exécuté lorsque l’exécution de **SetupEntryPoint** se termine correctement. Le processus résultant fait l’objet d’une surveillance et est redémarré (à partir de **SetupEntryPoint**) en cas d’interruption ou de défaillance. 

## <a name="configure-the-service-setup-entry-point"></a>Configurer le point d’entrée d’installation du service
Voici un exemple simple de manifeste de service pour un service sans état qui spécifie le script d’installation *MySetup.bat* dans le service **SetupEntryPoint**.  **Arguments** est utilisé pour passer des arguments au script lorsque celui-ci s’exécute.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="MyStatelessServicePkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <Description>An example service manifest.</Description>
  <ServiceTypes>
    <StatelessServiceType ServiceTypeName="MyStatelessServiceType" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <SetupEntryPoint>
      <ExeHost>
        <Program>MySetup.bat</Program>
        <Arguments>MyValue</Arguments>
        <WorkingFolder>Work</WorkingFolder>        
      </ExeHost>
    </SetupEntryPoint>
    <EntryPoint>
      <ExeHost>
        <Program>MyStatelessService.exe</Program>
      </ExeHost>
    </EntryPoint>
  </CodePackage>

  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <Endpoint Name="ServiceEndpoint" />
    </Endpoints>
  </Resources>
</ServiceManifest>
```
## <a name="configure-the-policy-for-a-service-setup-entry-point"></a>Configurer la stratégie pour un point d’entrée de configuration de service
Par défaut, l’exécutable du point d’entrée d’installation du service est exécuté avec les mêmes informations d’identification que Service Fabric (généralement, le compte *NetworkService*).  Dans le manifeste de l’application, vous pouvez changer les autorisations de sécurité de manière à exécuter le script de démarrage sous un compte système local ou un compte d’administrateur.

### <a name="configure-the-policy-by-using-a-local-system-account"></a>Configurer la stratégie à l’aide d’un compte système local
L’exemple de manifeste d’application suivant montre comment configurer le point d’entrée d’installation du service pour qu’il soit exécuté sous un compte d’utilisateur administrateur (SetupAdminUser).

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="MyApplicationType" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="MyStatelessService_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="MyStatelessServicePkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <RunAsPolicy CodePackageRef="Code" UserRef="SetupAdminUser" EntryPointType="Setup" />
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <Service Name="MyStatelessService" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="MyStatelessServiceType" InstanceCount="[MyStatelessService_InstanceCount]">
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

Tout d’abord, créez une section **Principals** avec un nom d’utilisateur, par exemple SetupAdminUser. Le compte d’utilisateur SetupAdminUser est membre du groupe des administrateurs système.

Ensuite, sous la section **ServiceManifestImport**, configurez une stratégie pour appliquer ce principal à **SetupEntryPoint**. Cette stratégie indique à Service Fabric que quand le fichier **MySetup.bat** est exécuté, il doit être exécuté en tant que SetupAdminUser avec des privilèges d’administrateur. Étant donné que vous *n’avez pas* appliqué de stratégie au point d’entrée principal, le code dans **MyServiceHost.exe** s’exécute sous le compte **NetworkService** du système. Il s’agit du compte par défaut avec lequel tous les points d’entrée de service sont exécutés en RunAs.

### <a name="configure-the-policy-by-using-local-system-accounts"></a>Configurer la stratégie à l’aide de comptes système locaux
Il est souvent préférable d’exécuter le script de démarrage à l’aide d’un compte système local plutôt que d’un compte d’administrateur. En règle générale, il n’est pas possible d’exécuter la stratégie RunAs en tant que membre du groupe Administrateurs, car le contrôle d’accès utilisateur est activé par défaut sur les ordinateurs. Dans ce cas, nous vous recommandons d’exécuter SetupEntryPoint en tant que LocalSystem plutôt qu’utilisateur local ajouté au groupe Administrateurs. L’exemple suivant montre comment configurer SetupEntryPoint pour une exécution en tant que LocalSystem :

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="MyApplicationType" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="MyStatelessService_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="MyStatelessServicePkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
         <RunAsPolicy CodePackageRef="Code" UserRef="SetupLocalSystem" EntryPointType="Setup" />
      </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <Service Name="MyStatelessService" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="MyStatelessServiceType" InstanceCount="[MyStatelessService_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
  <Principals>
      <Users>
         <User Name="SetupLocalSystem" AccountType="LocalSystem" />
      </Users>
   </Principals>
</ApplicationManifest>
```

> [!NOTE]
> Dans le cas des clusters Linux, pour exécuter un service ou la configuration de point d’entrée en tant que **racine**, vous pouvez spécifier le **AccountType** en tant que **LocalSystem**.

## <a name="run-a-script-from-the-setup-entry-point"></a>Exécuter un script à partir du point d’entrée d’installation
Maintenant, ajoutez un script de démarrage au projet pour que celui-ci soit exécuté avec des privilèges d’administrateur. 

Dans Visual Studio, cliquez avec le bouton droit sur le projet de service et ajoutez un fichier appelé *MySetup.bat*.

Vérifiez ensuite que le fichier *MySetup.bat* se trouve dans le package de service. Par défaut, il ne l’est pas. Sélectionnez le fichier, puis cliquez avec le bouton droit pour afficher le menu contextuel et choisissez **Propriétés**. Dans la boîte de dialogue Propriétés, vérifiez que **Copier dans le répertoire de sortie** est défini sur **Copier si plus récent**. Voir la capture d’écran suivante.

![Visual Studio CopyToOutput pour fichier batch SetupEntryPoint][image1]

À présent, ouvrez le fichier *MySetup.bat*, puis ajoutez les commandes suivantes pour définir une variable d’environnement système et générer une sortie sous forme de fichier texte :

```
REM Set a system environment variable. This requires administrator privilege
setx -m TestVariable %*
echo System TestVariable set to > out.txt
echo %TestVariable% >> out.txt

REM To delete this system variable us
REM REG delete "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Environment" /v TestVariable /f
```

Ensuite, générez et déployez la solution vers un cluster de développement local. Une fois que le service a démarré, comme indiqué dans [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md), vous pouvez voir que l’exécution du fichier MySetup.bat a réussi de deux façons. Ouvrez une invite de commandes PowerShell et entrez :

```
PS C:\ [Environment]::GetEnvironmentVariable("TestVariable","Machine")
MyValue
```

Ensuite, notez le nom du nœud sur lequel le service a été déployé et démarré dans [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md). Par exemple : Nœud 2. Accédez au dossier de travail de l’instance d’application pour rechercher le fichier out.txt qui affiche la valeur de **TestVariable**. Par exemple, si ce service a été déployé sur le Nœud 2, vous pouvez accéder à ce chemin pour **MyApplicationType**:

```
C:\SfDevCluster\Data\_App\Node.2\MyApplicationType_App\work\out.txt
```

## <a name="run-powershell-commands-from-a-setup-entry-point"></a>Exécuter des commandes PowerShell à partir d’un point d’entrée d’installation
Pour exécuter PowerShell à partir du point **SetupEntryPoint**, vous pouvez exécuter **PowerShell.exe** dans un fichier de commandes qui pointe vers un fichier PowerShell. Tout d’abord, ajoutez un fichier PowerShell au projet de service, par exemple **MySetup.ps1**. N’oubliez pas de définir la propriété *Copier si plus récent* afin que le fichier soit également inclus dans le package de service. L’exemple suivant montre un exemple de fichier de commandes permettant de lancer un fichier PowerShell appelé MySetup.ps1, qui définit une variable d’environnement appelée **TestVariable**.

MySetup.bat pour lancer un fichier PowerShell :

```
powershell.exe -ExecutionPolicy Bypass -Command ".\MySetup.ps1"
```

Dans le fichier PowerShell, ajoutez la commande suivante pour définir une variable d’environnement système :

```
[Environment]::SetEnvironmentVariable("TestVariable", "MyValue", "Machine")
[Environment]::GetEnvironmentVariable("TestVariable","Machine") > out.txt
```

> [!NOTE]
> Par défaut, quand le fichier de commandes s’exécute, il détermine si le dossier d’application **work** contient des fichiers. Dans ce cas, quand MySetup.bat s’exécute, il doit rechercher le fichier MySetup.ps1 dans ce dossier, qui contient le **package de code** de l’application. À cette fin, définissez le dossier de travail :
> 
> 

```xml
<SetupEntryPoint>
    <ExeHost>
    <Program>MySetup.bat</Program>
    <WorkingFolder>CodePackage</WorkingFolder>
    </ExeHost>
</SetupEntryPoint>
```

## <a name="debug-a-startup-script-locally-using-console-redirection"></a>Déboguer un script de démarrage localement à l’aide de la redirection de la console
Parfois, à des fins de débogage, il est utile de visualiser la sortie de la console générée par l’exécution d’un script d’installation. Vous pouvez définir une stratégie de redirection de la console sur le point d’entrée d’installation dans le manifeste de service, qui écrit ensuite la sortie dans un fichier. La sortie du fichier est écrite dans le dossier d’application **log**, sur le nœud de cluster où l’application est déployée et exécutée. 

> [!WARNING]
> N’utilisez jamais la stratégie de redirection de console dans une application déployée dans un environnement de production, car cela peut affecter le basculement de l’application. N’utilisez cette stratégie *que* pour le développement local et à des fins de débogage.  
> 
> 

L’exemple de manifeste de service suivant montre comment configurer la redirection de console avec une valeur FileRetentionCount :

```xml
<SetupEntryPoint>
    <ExeHost>
    <Program>MySetup.bat</Program>
    <WorkingFolder>CodePackage</WorkingFolder>
    <ConsoleRedirection FileRetentionCount="10"/>
    </ExeHost>
</SetupEntryPoint>
```

Si vous modifiez le fichier MySetup.ps1 pour écrire une commande **Echo**, le contenu de celle-ci apparaît dans le fichier de sortie à des fins de débogage :

```
Echo "Test console redirection which writes to the application log folder on the node that the application is deployed to"
```

> [!WARNING]
> Une fois que vous avez débogué votre script, supprimez immédiatement cette stratégie de redirection de console.



<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## <a name="next-steps"></a>Étapes suivantes
* [Découvrir la sécurité des applications et des services](service-fabric-application-and-service-security.md)
* [Comprendre le modèle d'application](service-fabric-application-model.md)
* [Spécifier des ressources dans un manifeste de service](service-fabric-service-manifest-resources.md)
* [Déployer une application](service-fabric-deploy-remove-applications.md)

[image1]: ./media/service-fabric-application-runas-security/copy-to-output.png
