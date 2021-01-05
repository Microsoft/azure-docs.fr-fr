---
title: Créer des stratégies Guest Configuration pour Windows
description: Découvrez comment créer une stratégie Guest Configuration pour des machines virtuelles Windows.
ms.date: 08/17/2020
ms.topic: how-to
ms.openlocfilehash: 124f747a1e7c7925efc2519ee826d62034e69cc5
ms.sourcegitcommit: ab94795f9b8443eef47abae5bc6848bb9d8d8d01
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/27/2020
ms.locfileid: "96302684"
---
# <a name="how-to-create-guest-configuration-policies-for-windows"></a>Créer des stratégies Guest Configuration pour Windows

Avant de créer des définitions de stratégie personnalisées, il est judicieux de lire les informations conceptuelles sur la page [Azure Policy Guest Configuration](../concepts/guest-configuration.md).
 
Pour en savoir plus sur la création de stratégies Guest Configuration pour Linux, consultez la page [Créer des stratégies Guest Configuration pour Linux](./guest-configuration-create-linux.md)

Lors de l’audit Windows, Guest Configuration utilise un module de ressources [Desired State Configuration](/powershell/scripting/dsc/overview/overview) (DSC) pour créer le fichier de configuration. La configuration DSC définit la condition dans laquelle la machine doit se trouver. Si l’évaluation de la configuration échoue, l’**auditIfNotExists** d’effet de stratégie est déclenché et la machine est considérée comme **non conforme**.

La [configuration d’invité Azure Policy](../concepts/guest-configuration.md) peut être utilisée uniquement pour auditer les paramètres à l’intérieur des machines. La correction des paramètres à l’intérieur des machines n’est pas encore disponible.

Utilisez les actions suivantes pour créer votre propre configuration pour la validation de l’état d’une machine Azure ou non-Azure.

> [!IMPORTANT]
> Les définitions de stratégie personnalisées avec la configuration invité dans les environnements Azure Government et Azure Chine sont des fonctionnalités d’évaluation.
>
> L’extension Guest Configuration (Configuration d’invité) est requise pour effectuer des audits sur des machines virtuelles Azure.
> Pour déployer l’extension à grande échelle sur tous les ordinateurs Windows, attribuez les définitions de stratégie suivantes : `Deploy prerequisites to enable Guest Configuration Policy on Windows VMs`
> 
> N’utilisez pas de secrets ni d’informations confidentielles dans des packages de contenu personnalisés.

## <a name="install-the-powershell-module"></a>Installer le module PowerShell

Le module Guest Configuration automatise le processus de création de contenu personnalisé, notamment :

- La création d’un artefact de contenu Guest Configuration (.zip)
- Le test automatisé de l’artefact
- La création d’une définition de stratégie
- La publication de la stratégie

Ce module peut être installé sur un ordinateur exécutant Windows, macOS ou Linux avec PowerShell 6.2 ou une version ultérieure exécutée localement, ou avec [Azure Cloud Shell](https://shell.azure.com), ou avec l’[image Azure PowerShell Core Docker](https://hub.docker.com/r/azuresdk/azure-powershell-core).

> [!NOTE]
> La compilation des configurations n’est pas encore prise en charge sur Linux.

### <a name="base-requirements"></a>Configuration de base requise

Systèmes d’exploitation sur lesquels le module peut être installé :

- Linux
- macOS
- Windows

Le module de ressource Guest Configuration nécessite les logiciels suivants :

- PowerShell 6.2 ou ultérieur. S’il n’est pas installé, suivez [ces instructions](/powershell/scripting/install/installing-powershell).
- Azure PowerShell 1.5.0 ou ultérieur. S’il n’est pas installé, suivez [ces instructions](/powershell/azure/install-az-ps).
  - Seuls les modules Az « Az.Accounts » et « Az.Resources » sont requis.

### <a name="install-the-module"></a>Installer le module

Pour installer le module **GuestConfiguration** dans PowerShell :

1. À partir d’une invite de commandes PowerShell, exécutez la commande suivante :

   ```azurepowershell-interactive
   # Install the Guest Configuration DSC resource module from PowerShell Gallery
   Install-Module -Name GuestConfiguration
   ```

1. Vérifiez que le module a été importé :

   ```azurepowershell-interactive
   # Get a list of commands for the imported GuestConfiguration module
   Get-Command -Module 'GuestConfiguration'
   ```

## <a name="guest-configuration-artifacts-and-policy-for-windows"></a>Artefacts Guest Configuration et stratégie pour Windows

Guest Configuration utilise Desired State Configuration de PowerShell comme abstraction de langage pour l’écriture des éléments à auditer dans Windows. L’agent charge une instance autonome de PowerShell 6.2 afin qu’il n’y ait pas de conflit avec l’utilisation de PowerShell DSC dans Windows PowerShell 5.1 et qu’il ne soit pas nécessaire de préinstaller PowerShell 6.2 ou une version ultérieure.

Pour obtenir une vue d’ensemble des concepts et de la terminologie DSC, consultez [Vue d’ensemble PowerShell DSC](/powershell/scripting/dsc/overview/overview).

### <a name="how-guest-configuration-modules-differ-from-windows-powershell-dsc-modules"></a>Différences entre les modules Guest Configuration et les modules DSC Windows PowerShell

Lorsque Guest Configuration audite un ordinateur, la séquence d’événements est différente de celle de Windows PowerShell DSC.

1. L’agent exécute d’abord `Test-TargetResource` pour déterminer si la configuration est dans l’état approprié.
1. La valeur booléenne retournée par la fonction détermine si l’état Azure Resource Manager de l’affectation d’invité doit être conforme ou non conforme.
1. Le fournisseur exécute `Get-TargetResource` pour retourner l’état actuel de chaque paramètre, afin que des détails soient disponibles à la fois sur la raison pour laquelle un ordinateur n’est pas conforme et pour confirmer que l’état actuel est conforme.

Les paramètres d’Azure Policy qui transmettent des valeurs à des affectations Guest Configuration doivent être de type _chaîne_. Il n’est pas possible de passer des tableaux en paramètre, même si la ressource DSC les prend en charge.

### <a name="get-targetresource-requirements"></a>Exigences Get-TargetResource

La fonction `Get-TargetResource` a des exigences pour Guest Configuration qu’elle n’a pas pour la configuration d’état souhaité Windows.

- La table de hachage qui est retournée doit inclure une propriété nommée **Reasons**.
- La propriété Reasons doit être un tableau.
- Chaque élément du tableau doit être une table de hachage avec les clés **Code** et **Phrase**.

La propriété Reasons est utilisée par le service pour normaliser la manière dont les informations sont présentées quand un ordinateur n’est pas conforme. Vous pouvez considérer chaque élément de Reasons comme une « raison » pour laquelle la ressource n’est pas conforme. La propriété est un tableau, car une ressource peut ne pas être conforme pour plusieurs raisons.

Les propriétés **Code** et **Phrase** sont attendues par le service. Lorsque vous créez une ressource personnalisée, définissez le texte (en général stdout) que vous souhaitez afficher pour expliquer pourquoi la ressource n’est pas conforme à la valeur de **Phrase**. **Code** a des exigences de mise en forme spécifiques, visant à afficher clairement les informations sur la ressource utilisée pour effectuer l’audit. Cette solution rend Guest Configuration extensible. Vous pouvez exécuter n’importe quelle commande tant que la sortie peut être retournée sous forme de valeur de chaîne pour la propriété **Phrase**.

- **Code** (chaîne) : nom de la ressource, répété, puis un nom abrégé sans espaces comme identificateur de la raison. Ces trois valeurs doivent être séparées par des points-virgules, sans espaces.
  - Par exemple, `registry:registry:keynotpresent`.
- **Phrase** (chaîne) : texte lisible par l’utilisateur expliquant pourquoi le paramètre n’est pas conforme.
  - Par exemple, `The registry key $key is not present on the machine.`.

```powershell
$reasons = @()
$reasons += @{
  Code = 'Name:Name:ReasonIdentifer'
  Phrase = 'Explain why the setting is not compliant'
}
return @{
    reasons = $reasons
}
```

La propriété Reasons doit être ajoutée au schéma MOF de la ressource en tant que classe incorporée.

```mof
[ClassVersion("1.0.0.0")] 
class Reason
{
    [Read] String Phrase;
    [Read] String Code;
};

[ClassVersion("1.0.0.0"), FriendlyName("ResourceName")]
class ResourceName : OMI_BaseResource
{
    [Key, Description("Example description")] String Example;
    [Read, EmbeddedInstance("Reason")] String Reasons[];
};
```

### <a name="configuration-requirements"></a>Exigences de configuration

Le nom de la configuration personnalisée doit être cohérent partout. Le nom du fichier .zip du package de contenu, celui de la configuration dans le fichier MOF et celui de l’affectation d’invité dans le modèle Azure Resource Manager (modèle ARM) doivent être identiques.

### <a name="scaffolding-a-guest-configuration-project"></a>Structuration d’un projet Guest Configuration

Les développeurs qui souhaitent accélérer la mise en œuvre et travailler à partir de l’exemple de code peuvent installer un projet de la communauté appelé **Guest Configuration Project**. Le projet installe un modèle pour le module PowerShell [Plaster](https://github.com/powershell/plaster). Cet outil peut être utilisé pour structurer un projet, (y compris une configuration de travail et un exemple de ressource), ainsi qu’un ensemble de tests [Pester](https://github.com/pester/pester) pour valider le projet. Le modèle comprend également des exécuteurs de tâches pour Visual Studio Code permettant d’automatiser la création et la validation du package Guest Configuration. Pour plus d’informations, consultez le projet GitHub [Guest Configuration Project](https://github.com/microsoft/guestconfigurationproject).

Pour plus d’informations sur l’utilisation des configurations, consultez [Écrire, compiler et appliquer une configuration](/powershell/scripting/dsc/configurations/write-compile-apply-configuration).

### <a name="expected-contents-of-a-guest-configuration-artifact"></a>Contenu attendu d’un artefact Guest Configuration

Le package obtenu est utilisé par Guest Configuration pour créer les définitions d’Azure Policy. Le package comprend les éléments suivants :

- La configuration DSC compilée au format MOF
- Dossier Modules
  - Module GuestConfiguration
  - Module DscNativeResources
  - Modules de ressources DSC (Windows) requis par MOF

Les cmdlets PowerShell aident à créer le package.
Aucun dossier de niveau racine ou dossier de version n’est requis.
Le format du package doit être un fichier .zip et ne peut pas dépasser une taille totale de 100 Mo lors de la décompression.

### <a name="storing-guest-configuration-artifacts"></a>Stockage des artefacts Guest Configuration

Le package .zip doit être stocké dans un emplacement accessible par les machines virtuelles gérées.
Exemples : référentiels GitHub, référentiel Azure ou stockage Azure. Si vous préférez ne pas rendre le package public, vous pouvez inclure un [jeton SAS](../../../storage/common/storage-sas-overview.md) dans l’URL. Vous pouvez également implémenter le [point de terminaison de service](../../../storage/common/storage-network-security.md#grant-access-from-a-virtual-network) pour les machines dans un réseau privé, bien que cette configuration s’applique uniquement à l’accès au package et ne communique pas avec le service.

## <a name="step-by-step-creating-a-custom-guest-configuration-audit-policy-for-windows"></a>Étape par étape, création d’une stratégie d’audit Guest Configuration personnalisée pour Windows

Créez une configuration DSC pour auditer les paramètres. L’exemple de script PowerShell suivant crée une configuration appelée **AuditBitLocker**, importe le module de ressources **PsDscResources** et utilise la ressource `Service` pour faire l’audit d’un service en cours d’exécution. Le script de configuration peut être exécuté à partir d’un ordinateur Windows ou macOS.

```powershell
# Add PSDscResources module to environment
Install-Module 'PSDscResources'

# Define the DSC configuration and import GuestConfiguration
Configuration AuditBitLocker
{
    Import-DscResource -ModuleName 'PSDscResources'

    Node AuditBitlocker {
      Service 'Ensure BitLocker service is present and running'
      {
          Name = 'BDESVC'
          Ensure = 'Present'
          State = 'Running'
      }
    }
}

# Compile the configuration to create the MOF files
AuditBitLocker ./Config
```

Enregistrez ce fichier sous le nom `config.ps1` dans le dossier du projet. Exécutez-le dans PowerShell en exécutant `./config.ps1` dans le terminal. Un nouveau fichier mof est créé.

La commande `Node AuditBitlocker` n’est pas techniquement obligatoire, mais elle produit un fichier `AuditBitlocker.mof` plutôt que `localhost.mof`par défaut. Le fait d’avoir le nom de fichier. mof à la suite de la configuration permet d’organiser facilement de nombreux fichiers à grande échelle.

Une fois la compilation du fichier MOF terminée, les fichiers de prise en charge doivent être regroupés en un package. Le package obtenu est utilisé par Guest Configuration pour créer les définitions d’Azure Policy.

La cmdlet `New-GuestConfigurationPackage` crée le package. Les modules nécessaires à la configuration doivent être disponibles dans `$Env:PSModulePath`. Paramètres de la cmdlet `New-GuestConfigurationPackage` lors de la création du contenu Windows :

- **Name** : Nom du package Guest Configuration.
- **Configuration** : Chemin d’accès complet au document de configuration DSC compilé.
- **Chemin d’accès** : Chemin d’accès au dossier de sortie. Ce paramètre est facultatif. S’il n’est pas spécifié, le package est créé dans le répertoire actif.

Exécutez la commande suivante pour créer un package à l’aide de la configuration fournie à l’étape précédente :

```azurepowershell-interactive
New-GuestConfigurationPackage `
  -Name 'AuditBitlocker' `
  -Configuration './Config/AuditBitlocker.mof'
```

Après avoir créé le package de configuration et avant de le publier sur Azure, vous pouvez tester le package à partir de votre station de travail ou de votre environnement d’intégration continue et de déploiement continu (CI/CD). La cmdlet GuestConfiguration `Test-GuestConfigurationPackage` comprend le même agent dans votre environnement de développement que celui utilisé dans les machines Azure. Via cette solution, vous pouvez effectuer un test d’intégration en local avant la publication dans les environnements cloud facturés.

Étant donné que l’agent évalue l’environnement local, vous devez, dans la plupart des cas, vous devez exécuter la cmdlet Test- sur la même plateforme de système d’exploitation que celle que vous envisagez d’auditer. Le test utilise uniquement les modules qui sont inclus dans le package de contenu.

Paramètres de la cmdlet `Test-GuestConfigurationPackage` :

- **Name** : Nom de la stratégie Guest Configuration.
- **Paramètre** : Paramètres de stratégie fournis au format Hashtable.
- **Chemin d’accès** : Chemin d’accès complet du package Guest Configuration.

Exécutez la commande suivante pour tester le package créé par l’étape précédente :

```azurepowershell-interactive
Test-GuestConfigurationPackage `
  -Path ./AuditBitlocker.zip
```

La cmdlet prend aussi en charge l’entrée depuis le pipeline PowerShell. Dirige la sortie de la cmdlet `New-GuestConfigurationPackage` vers la cmdlet `Test-GuestConfigurationPackage`.

```azurepowershell-interactive
New-GuestConfigurationPackage -Name AuditBitlocker -Configuration ./Config/AuditBitlocker.mof | Test-GuestConfigurationPackage
```

L’étape suivante consiste à publier le fichier dans Stockage Blob Azure. La commande `Publish-GuestConfigurationPackage` requiert le module `Az.Storage`.

```azurepowershell-interactive
Publish-GuestConfigurationPackage -Path ./AuditBitlocker.zip -ResourceGroupName myResourceGroupName -StorageAccountName myStorageAccountName
```

Une fois qu’un package de stratégie personnalisée Guest Configuration a été créé et chargé, créez la définition de la stratégie Guest Configuration. La cmdlet `New-GuestConfigurationPolicy` prend un package de stratégie personnalisé et crée une définition de stratégie.

Paramètres de la cmdlet `New-GuestConfigurationPolicy` :

- **ContentUri** : URI http(s) publique du package de contenu Guest Configuration.
- **DisplayName** : Nom d'affichage de la stratégie.
- **Description** : Description de la stratégie.
- **Paramètre** : Paramètres de stratégie fournis au format Hashtable.
- **Version** : Version de stratégie.
- **Chemin d’accès** : Chemin de destination où les définitions de stratégie sont créées.
- **Plateforme** : Plateforme cible (Windows/Linux) pour la stratégie et le package de contenu Guest Configuration.
- **Tag** ajoute un ou plusieurs filtres de balise à la définition de stratégie
- **Category** définit le champ de métadonnées catégorie dans la définition de stratégie

L’exemple suivant crée les définitions de stratégie dans le chemin d’accès spécifié à partir d’un package de stratégie personnalisé :

```azurepowershell-interactive
New-GuestConfigurationPolicy `
    -ContentUri 'https://storageaccountname.blob.core.windows.net/packages/AuditBitLocker.zip?st=2019-07-01T00%3A00%3A00Z&se=2024-07-01T00%3A00%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=JdUf4nOCo8fvuflOoX%2FnGo4sXqVfP5BYXHzTl3%2BovJo%3D' `
    -DisplayName 'Audit BitLocker Service.' `
    -Description 'Audit if BitLocker is not enabled on Windows machine.' `
    -Path './policies' `
    -Platform 'Windows' `
    -Version 1.0.0 `
    -Verbose
```

Les fichiers suivants sont créés par `New-GuestConfigurationPolicy` :

- **auditIfNotExists.json**

La sortie de la cmdlet retourne un objet contenant le nom complet de l’initiative et le chemin d’accès aux fichiers de stratégie.

Enfin, publiez les définitions de stratégie à l’aide de la cmdlet `Publish-GuestConfigurationPolicy`. La cmdlet ne dispose que du paramètre **Path** qui pointe vers l’emplacement des trois fichiers JSON créés par `New-GuestConfigurationPolicy`.

Pour exécuter la commande Publish, vous devez avoir accès à la création de stratégies dans Azure. Les exigences spécifiques en matière d’autorisations sont documentées dans la page [vue d’ensemble d’Azure Policy](../overview.md). Le meilleur rôle intégré est le rôle **Contributeur de la stratégie de ressource**.

```azurepowershell-interactive
Publish-GuestConfigurationPolicy -Path '.\policyDefinitions'
```

La cmdlet `Publish-GuestConfigurationPolicy` accepte le chemin à partir du pipeline PowerShell. Cette fonctionnalité signifie que vous pouvez créer les fichiers de stratégie et les publier dans un ensemble unique de commandes redirigées.

```azurepowershell-interactive
New-GuestConfigurationPolicy `
 -ContentUri 'https://storageaccountname.blob.core.windows.net/packages/AuditBitLocker.zip?st=2019-07-01T00%3A00%3A00Z&se=2024-07-01T00%3A00%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=JdUf4nOCo8fvuflOoX%2FnGo4sXqVfP5BYXHzTl3%2BovJo%3D' `
  -DisplayName 'Audit BitLocker service.' `
  -Description 'Audit if the BitLocker service is not enabled on Windows machine.' `
  -Path './policies' `
 | Publish-GuestConfigurationPolicy
```

Avec la stratégie créée dans Azure, la dernière étape consiste à attribuer la définition. Apprenez à attribuer la définition à l'aide du [portail](../assign-policy-portal.md), d'[Azure CLI](../assign-policy-azurecli.md) et d'[Azure PowerShell](../assign-policy-powershell.md).

### <a name="filtering-guest-configuration-policies-using-tags"></a>Filtrage des stratégies Guest Configuration à l’aide de balises

Les définitions de stratégie créées par les cmdlet dans le module Guest Configuration peuvent éventuellement inclure un filtre pour les balises. Le paramètre **Tag** de `New-GuestConfigurationPolicy` prend en charge un groupe de tables de hachage contenant des balises individuelles. Les balises sont ajoutées à la section `If` de la définition de stratégie et ne peuvent pas être modifiées par une attribution de stratégie.

Vous trouverez ci-dessous un exemple d’extrait de définition de stratégie permettant de filtrer les balises.

```json
"if": {
  "allOf" : [
    {
      "allOf": [
        {
          "field": "tags.Owner",
          "equals": "BusinessUnit"
        },
        {
          "field": "tags.Role",
          "equals": "Web"
        }
      ]
    },
    {
      // Original Guest Configuration content
    }
  ]
}
```

### <a name="using-parameters-in-custom-guest-configuration-policy-definitions"></a>Utilisation de paramètres dans des définitions de stratégie personnalisées Guest Configuration

Guest Configuration prend en charge la substitution des propriétés d’une configuration lors d’une exécution. Cette fonctionnalité signifie que les valeurs du fichier MOF dans le package n’ont pas à être considérées comme statiques. Les valeurs de substitution sont fournies via Azure Policy et n’impactent pas la création ni la compilation des configurations.

Les cmdlets `New-GuestConfigurationPolicy` et `Test-GuestConfigurationPolicyPackage` incluent un paramètre nommé **Paramètre**. Ce paramètre utilise une définition hashtable qui comprend toutes les informations concernant chaque paramètre et crée les sections nécessaires de chaque fichier de la définition Azure Policy.

L’exemple suivant crée une définition de stratégie pour auditer un service, que l’utilisateur sélectionne dans une liste au moment de l’attribution de la stratégie.

```azurepowershell-interactive
# This DSC Resource text:
Service 'UserSelectedNameExample'
      {
          Name = 'ParameterValue'
          Ensure = 'Present'
          State = 'Running'
      }

# Would require the following hashtable:
$PolicyParameterInfo = @(
    @{
        Name = 'ServiceName'                                            # Policy parameter name (mandatory)
        DisplayName = 'windows service name.'                           # Policy parameter display name (mandatory)
        Description = "Name of the windows service to be audited."      # Policy parameter description (optional)
        ResourceType = "Service"                                        # DSC configuration resource type (mandatory)
        ResourceId = 'UserSelectedNameExample'                                   # DSC configuration resource id (mandatory)
        ResourcePropertyName = "Name"                                   # DSC configuration resource property name (mandatory)
        DefaultValue = 'winrm'                                          # Policy parameter default value (optional)
        AllowedValues = @('BDESVC','TermService','wuauserv','winrm')    # Policy parameter allowed values (optional)
    }
)

New-GuestConfigurationPolicy
    -ContentUri 'https://storageaccountname.blob.core.windows.net/packages/AuditBitLocker.zip?st=2019-07-01T00%3A00%3A00Z&se=2024-07-01T00%3A00%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=JdUf4nOCo8fvuflOoX%2FnGo4sXqVfP5BYXHzTl3%2BovJo%3D' `
    -DisplayName 'Audit Windows Service.' `
    -Description 'Audit if a Windows Service is not enabled on Windows machine.' `
    -Path '.\policyDefinitions' `
    -Parameter $PolicyParameterInfo `
    -Version 1.0.0
```

## <a name="extending-guest-configuration-with-third-party-tools"></a>Extension de Guest Configuration avec des outils tiers

Les packages d’artefacts pour Guest Configuration peuvent être étendus pour inclure des outils tiers.
L’extension Guest Configuration requiert le développement de deux composants.

- Une ressource Desired State Configuration (DSC) qui gère toutes les activités liées à la gestion de l’outil tiers
  - Installer
  - Appeler
  - Convertir la sortie
- Contenu dans le format correct pour l’outil à consommer en mode natif

La ressource DSC requiert un développement personnalisé d’il n’existe pas encore de solution de communauté.
Les solutions de la communauté peuvent être affichées en recherchant la balise [GuestConfiguration](https://www.powershellgallery.com/packages?q=Tags%3A%22GuestConfiguration%22) dans PowerShell Gallery.

> [!Note]
> L’extensibilité de Guest Configuration est un scénario de type BYOL (apportez votre propre licence). Veillez à respecter les conditions générales de tout outil tiers avant de l’utiliser.

Une fois la ressource DSC installée dans l’environnement de développement, utilisez le paramètre **FilesToInclude** pour `New-GuestConfigurationPackage` afin d’inclure le contenu de la plateforme tierce dans l’artefact de contenu.

### <a name="step-by-step-creating-a-content-artifact-that-uses-third-party-tools"></a>Étape par étape, création d’un artefact de contenu qui utilise des outils tiers

Seule l’applet de commande `New-GuestConfigurationPackage` requiert une modification des instructions pas à pas pour les artefacts de contenu DSC. Pour cet exemple, utilisez le module `gcInSpec` pour étendre Guest Configuration afin d’auditer des ordinateurs Windows à l’aide de la plateforme InSpec plutôt que du module intégré utilisé sur Linux. Le module de la communauté est conservé en tant que [projet open source dans GitHub](https://github.com/microsoft/gcinspec).

Installez les modules requis dans votre environnement de développement :

```azurepowershell-interactive
# Update PowerShellGet if needed to allow installing PreRelease versions of modules
Install-Module PowerShellGet -Force

# Install GuestConfiguration module prerelease version
Install-Module GuestConfiguration -allowprerelease

# Install commmunity supported gcInSpec module
Install-Module gcInSpec
```

Tout d’abord, créez le fichier YaML utilisé par InSpec. Le fichier fournit des informations de base sur l’environnement. Un exemple est fourni ci-dessous :

```YaML
name: wmi_service
title: Verify WMI service is running
maintainer: Microsoft Corporation
summary: Validates that the Windows Service 'winmgmt' is running
copyright: Microsoft Corporation
license: MIT
version: 1.0.0
supports:
  - os-family: windows
```

Enregistrez ce fichier nommé `wmi_service.yml` dans un dossier nommé `wmi_service` à l’intérieur du répertoire de votre projet.

Ensuite, créez le fichier Ruby avec l’abstraction de langage InSpec utilisée pour auditer l’ordinateur.

```Ruby
control 'wmi_service' do
  impact 1.0
  title 'Verify windows service: winmgmt'
  desc 'Validates that the service, is installed, enabled, and running'

  describe service('winmgmt') do
    it { should be_installed }
    it { should be_enabled }
    it { should be_running }
  end
end

```

Enregistrez ce fichier `wmi_service.rb` dans un nouveau dossier nommé `controls` à l’intérieur du répertoire `wmi_service`.

Enfin, créez une configuration, importez le module de ressource **GuestConfiguration** et utilisez la ressource `gcInSpec` pour définir le nom du profil InSpec.

```powershell
# Define the configuration and import GuestConfiguration
Configuration wmi_service
{
    Import-DSCResource -Module @{ModuleName = 'gcInSpec'; ModuleVersion = '2.1.0'}
    node 'wmi_service'
    {
        gcInSpec wmi_service
        {
            InSpecProfileName       = 'wmi_service'
            InSpecVersion           = '3.9.3'
            WindowsServerVersion    = '2016'
        }
    }
}

# Compile the configuration to create the MOF files
wmi_service -out ./Config
```

Vous devez maintenant avoir une structure de projet comme indiqué ci-dessous :

```file
/ wmi_service
    / Config
        wmi_service.mof
    / wmi_service
        wmi_service.yml
        / controls
            wmi_service.rb 
```

Les fichiers de prise en charge doivent être regroupés en un package. Le package obtenu est utilisé par Guest Configuration pour créer les définitions d’Azure Policy.

La cmdlet `New-GuestConfigurationPackage` crée le package. Pour le contenu tiers, utilisez le paramètre **FilesToInclude** afin d’ajouter le contenu InSpec au package. Il n’est pas nécessaire de spécifier le paramètre **ChefProfilePath** comme pour les packages Linux.

- **Name** : Nom du package Guest Configuration.
- **Configuration** : Chemin d’accès complet au document de configuration compilé.
- **Chemin d’accès** : Chemin d’accès au dossier de sortie. Ce paramètre est facultatif. S’il n’est pas spécifié, le package est créé dans le répertoire actif.
- **FilesoInclude** : Chemin d’accès complet au profil InSpec.

Exécutez la commande suivante pour créer un package à l’aide de la configuration fournie à l’étape précédente :

```azurepowershell-interactive
New-GuestConfigurationPackage `
  -Name 'wmi_service' `
  -Configuration './Config/wmi_service.mof' `
  -FilesToInclude './wmi_service'  `
  -Path './package' 
```

## <a name="policy-lifecycle"></a>Cycle de vie de la stratégie

Si vous souhaitez publier une mise à jour de la stratégie, votre attention est requise dans trois domaines.

> [!NOTE]
> La propriété `version` de l’affectation de configuration invité n’a d’influence que sur les packages qui sont hébergés par Microsoft. La meilleure pratique pour le contenu personnalisé du contrôle de version consiste à inclure la version dans le nom de fichier.

- **Version** : Lorsque vous exécutez l’applet de commande `New-GuestConfigurationPolicy`, vous devez spécifier un numéro de version supérieur à celui actuellement publié.
- **contentUri** : Lorsque vous exécutez la cmdlet `New-GuestConfigurationPolicy`, vous devez spécifier un URI vers l’emplacement du package. L’inclusion d’une version de package dans le nom de fichier garantit que la valeur de cette propriété change dans chaque version.
- **contentHash** : Cette propriété est automatiquement mise à jour par l’applet de commande `New-GuestConfigurationPolicy`. Il s’agit d’une valeur de hachage du package créé par `New-GuestConfigurationPackage`. Cette propriété doit être correcte pour le fichier `.zip` que vous publiez. Si seule la propriété **contentUri** est mise à jour, l’extension n’accepte pas le package de contenu.

Le moyen le plus simple de publier un package mis à jour consiste à répéter le processus décrit dans cet article et à fournir un numéro de version mis à jour. Ce processus garantit que toutes les propriétés ont été correctement mises à jour.

## <a name="optional-signing-guest-configuration-packages"></a>Facultatif : Signature des packages Guest Configuration

Les stratégies personnalisées Guest Configuration utilisent le hachage SHA256 pour vérifier que le package de stratégie n’a pas changé.
Si vous le souhaitez, les clients peuvent également utiliser un certificat pour signer des packages et forcer l’extension Guest Configuration à autoriser uniquement le contenu signé.

Pour activer ce scénario, vous devez effectuer deux étapes. Exécutez l’applet de commande pour signer le package de contenu et ajoutez une balise aux machines qui doivent nécessiter du code pour être signées.

Pour utiliser la fonctionnalité de validation de signature, exécutez la cmdlet `Protect-GuestConfigurationPackage` pour signer le package avant sa publication. Cette cmdlet nécessite un certificat de « signature du code ».

```azurepowershell-interactive
$Cert = Get-ChildItem -Path cert:\LocalMachine\My | Where-Object {($_.Subject-eq "CN=mycert") }
Protect-GuestConfigurationPackage -Path .\package\AuditWindowsService\AuditWindowsService.zip -Certificate $Cert -Verbose
```

Paramètres de la cmdlet `Protect-GuestConfigurationPackage` :

- **Chemin d’accès** : Chemin d’accès complet du package Guest Configuration.
- **Certificat** : Certificat de signature de code pour signer le package. Ce paramètre est uniquement pris en charge lors de la signature de contenu pour Windows.

L’agent GuestConfiguration s’attend à trouver la clé publique du certificat dans « Autorités de certification racines de confiance » sur des machines Windows et dans le chemin `/usr/local/share/ca-certificates/extra` sur des machines Linux. Pour que le nœud vérifie le contenu signé, installez la clé publique du certificat sur la machine avant d’appliquer la stratégie personnalisée. Ce processus peut être effectué à l’aide de n’importe quelle technique à l’intérieur de la machine virtuelle ou à l’aide d’Azure Policy. Vous trouverez un exemple modèle en suivant [ce lien](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-push-certificate-windows).
La stratégie d’accès Key Vault doit autoriser le fournisseur de ressources de calcul à accéder aux certificats lors des déploiements. Pour les étapes détaillées, consultez [Configurer Key Vault pour des machines virtuelles dans Azure Resource Manager](../../../virtual-machines/windows/key-vault-setup.md#use-templates-to-set-up-key-vault).

Voici un exemple d’exportation de la clé publique à partir d’un certificat de signature, à importer vers la machine.

```azurepowershell-interactive
$Cert = Get-ChildItem -Path cert:\LocalMachine\My | Where-Object {($_.Subject-eq "CN=mycert3") } | Select-Object -First 1
$Cert | Export-Certificate -FilePath "$env:temp\DscPublicKey.cer" -Force
```

Une fois votre contenu publié, ajoutez une balise nommée `GuestConfigPolicyCertificateValidation` et avec une valeur `enabled` à toutes les machines virtuelles où la signature du code doit être requise. Pour plus d'informations sur la façon dont les balises peuvent être délivrées à grande échelle à l'aide d'Azure Policy, consultez les [Exemples de balises](../samples/built-in-policies.md#tags). Une fois cette balise en place, la définition de stratégie générée via la cmdlet `New-GuestConfigurationPolicy` met en œuvre l’exigence via l’extension Guest Configuration.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur l’audit des machines virtuelles avec [Guest Configuration](../concepts/guest-configuration.md).
- Découvrez comment [créer des stratégies par programmation](./programmatically-create.md).
- Découvrez comment [obtenir des données de conformité](./get-compliance-data.md).
