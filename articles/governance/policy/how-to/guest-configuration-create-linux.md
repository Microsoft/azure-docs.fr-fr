---
title: Créer des stratégies Guest Configuration pour Linux
description: Découvrez comment créer une stratégie Guest Configuration pour des machines virtuelles Linux.
ms.date: 08/17/2020
ms.topic: how-to
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 1f6308250717d35dc725b097575bf3921646c6a0
ms.sourcegitcommit: ab94795f9b8443eef47abae5bc6848bb9d8d8d01
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/27/2020
ms.locfileid: "96302710"
---
# <a name="how-to-create-guest-configuration-policies-for-linux"></a>Créer des stratégies Guest Configuration pour Linux

Avant de créer des stratégies personnalisées, lisez les informations de présentation de [Configuration d’invité Azure Policy](../concepts/guest-configuration.md).
 
Pour en savoir plus sur la création de stratégies Guest Configuration pour Windows, consultez la page [Créer des stratégies Guest Configuration pour Windows](./guest-configuration-create.md)

Lors de l’audit de Linux, Guest Configuration utilise [Chef InSpec](https://www.inspec.io/). Le profil InSpec définit la condition dans laquelle la machine doit se trouver. Si l’évaluation de la configuration échoue, l’**auditIfNotExists** d’effet de stratégie est déclenché et la machine est considérée comme **non conforme**.

La [configuration d’invité Azure Policy](../concepts/guest-configuration.md) peut être utilisée uniquement pour auditer les paramètres à l’intérieur des machines. La correction des paramètres à l’intérieur des machines n’est pas encore disponible.

Utilisez les actions suivantes pour créer votre propre configuration pour la validation de l’état d’une machine Azure ou non-Azure.

> [!IMPORTANT]
> Les définitions de stratégie personnalisées avec la configuration invité dans les environnements Azure Government et Azure Chine sont des fonctionnalités d’évaluation.
>
> L’extension Guest Configuration (Configuration d’invité) est requise pour effectuer des audits sur des machines virtuelles Azure. Pour déployer l’extension à grande échelle sur toutes les machines Linux, attribuez la définition de stratégie suivante : `Deploy prerequisites to enable Guest Configuration Policy on Linux VMs`
> 
> N’utilisez pas de secrets ni d’informations confidentielles dans les packages de contenu personnalisés.

## <a name="install-the-powershell-module"></a>Installer le module PowerShell

Le module Guest Configuration automatise le processus de création de contenu personnalisé, notamment :

- La création d’un artefact de contenu Guest Configuration (.zip)
- Le test automatisé de l’artefact
- La création d’une définition de stratégie
- La publication de la stratégie

Ce module peut être installé sur un ordinateur exécutant Windows, macOS ou Linux avec PowerShell 6.2 ou une version ultérieure exécutée localement, ou avec [Azure Cloud Shell](https://shell.azure.com), ou avec l’[image Azure PowerShell Core Docker](https://hub.docker.com/r/azuresdk/azure-powershell-core).

> [!NOTE]
> La compilation des configurations n’est pas prise en charge sur Linux.

### <a name="base-requirements"></a>Configuration de base requise

Systèmes d’exploitation sur lesquels le module peut être installé :

- Linux
- macOS
- Windows

> [!NOTE]
> La cmdlet `Test-GuestConfigurationPackage` requiert OpenSSL version 1.0 en raison d’une dépendance d’OMI. Cela provoque une erreur sur tout environnement avec OpenSSL 1.1 ou version ultérieure.
>
> L’exécution de la cmdlet `Test-GuestConfigurationPackage` est prise en charge uniquement sur Windows pour le module Guest Configuration version 2.1.0.

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

## <a name="guest-configuration-artifacts-and-policy-for-linux"></a>Artefacts Guest Configuration et stratégie pour Linux

Même dans les environnements Linux, Guest Configuration utilise Desired State Configuration en tant qu’abstraction de langage. L’implémentation est basée sur le code natif (C++) de sorte qu’elle ne nécessite pas le chargement de PowerShell. Toutefois, elle nécessite une configuration MOF décrivant les détails de l’environnement.
DSC agit comme un wrapper pour InSpec afin de normaliser son exécution, ainsi que les modalités de fourniture des paramètres et de renvoi des résultats au service. Une connaissance de base de DSC est suffisante l’utilisation de contenu InSpec personnalisé.

#### <a name="configuration-requirements"></a>Exigences de configuration

Le nom de la configuration personnalisée doit être cohérent partout. Le nom du fichier .zip du package de contenu, celui de la configuration dans le fichier MOF et celui de l’affectation d’invité dans le modèle Azure Resource Manager (modèle ARM) doivent être identiques.

Les cmdlets PowerShell aident à créer le package.
Aucun dossier de niveau racine ou dossier de version n’est requis.
Le format du package doit être un fichier .zip. Et il ne peut pas dépasser une taille totale de 100 Mo lors de la décompression.

### <a name="custom-guest-configuration-configuration-on-linux"></a>Configuration Guest Configuration personnalisée sur Linux

Guest Configuration sur Linux utilise la ressource `ChefInSpecResource` pour fournir au moteur le nom du [profil InSpec](https://www.inspec.io/docs/reference/profiles/). **Name** est la seule propriété de ressource obligatoire. Créez un fichier YaML et un fichier de script Ruby, comme indiqué ci-dessous.

Tout d’abord, créez le fichier YaML utilisé par InSpec. Le fichier fournit des informations de base sur l’environnement. Un exemple est fourni ci-dessous :

```YaML
name: linux-path
title: Linux path
maintainer: Test
summary: Test profile
license: MIT
version: 1.0.0
supports:
    - os-family: unix
```

Enregistrez ce fichier nommé `inspec.yml` dans un dossier nommé `linux-path` dans le répertoire de votre projet.

Ensuite, créez le fichier Ruby avec l’abstraction de langage InSpec utilisée pour auditer l’ordinateur.

```Ruby
describe file('/tmp') do
    it { should exist }
end
```

Enregistrez ce fichier nommé `linux-path.rb` dans un nouveau dossier nommé `controls` dans le répertoire `linux-path`.

Enfin, créez une configuration, importez le module de ressources **PSDesiredStateConfiguration** et compilez la configuration.

```powershell
# import PSDesiredStateConfiguration module
import-module PSDesiredStateConfiguration

# Define the configuration and import GuestConfiguration
Configuration AuditFilePathExists
{
    Import-DscResource -ModuleName 'GuestConfiguration'

    Node AuditFilePathExists
    {
        ChefInSpecResource 'Audit Linux path exists'
        {
            Name = 'linux-path'
        }
    }
}

# Compile the configuration to create the MOF files
AuditFilePathExists -out ./Config
```

Enregistrez ce fichier sous le nom `config.ps1` dans le dossier du projet. Exécutez-le dans PowerShell en exécutant `./config.ps1` dans le terminal. Un fichier mof est créé.

La commande `Node AuditFilePathExists` n’est pas techniquement obligatoire, mais elle produit un fichier `AuditFilePathExists.mof` plutôt que `localhost.mof`par défaut. Le fait d’avoir le nom de fichier. mof à la suite de la configuration permet d’organiser facilement de nombreux fichiers à grande échelle.

Vous devez maintenant avoir une structure de projet comme indiqué ci-dessous :

```file
/ AuditFilePathExists
    / Config
        AuditFilePathExists.mof
    / linux-path
        inspec.yml
        / controls
            linux-path.rb 
```

Les fichiers de prise en charge doivent être regroupés en un package. Le package obtenu est utilisé par Guest Configuration pour créer les définitions d’Azure Policy.

La cmdlet `New-GuestConfigurationPackage` crée le package. Paramètres de la cmdlet `New-GuestConfigurationPackage` lors de la création du contenu Linux :

- **Name** : Nom du package Guest Configuration.
- **Configuration** : Chemin d’accès complet au document de configuration compilé.
- **Chemin d’accès** : Chemin d’accès au dossier de sortie. Ce paramètre est facultatif. S’il n’est pas spécifié, le package est créé dans le répertoire actif.
- **ChefInspecProfilePath** : Chemin d’accès complet au profil InSpec. Ce paramètre est pris en charge uniquement lors de la création de contenu pour auditer Linux.

Exécutez la commande suivante pour créer un package à l’aide de la configuration fournie à l’étape précédente :

```azurepowershell-interactive
New-GuestConfigurationPackage `
  -Name 'AuditFilePathExists' `
  -Configuration './Config/AuditFilePathExists.mof' `
  -ChefInSpecProfilePath './'
```

Après avoir créé le package de configuration et avant de le publier sur Azure, vous pouvez tester le package à partir de votre station de travail ou de votre environnement d’intégration continue et de déploiement continu (CI/CD). La cmdlet GuestConfiguration `Test-GuestConfigurationPackage` comprend le même agent dans votre environnement de développement que celui utilisé dans les machines Azure. Via cette solution, vous pouvez effectuer un test d’intégration en local avant la publication dans les environnements cloud facturés.

Étant donné que l’agent évalue l’environnement local, vous devez, dans la plupart des cas, vous devez exécuter la cmdlet Test- sur la même plateforme de système d’exploitation que celle que vous envisagez d’auditer.

Paramètres de la cmdlet `Test-GuestConfigurationPackage` :

- **Name** : Nom de la stratégie Guest Configuration.
- **Paramètre** : Paramètres de stratégie fournis au format Hashtable.
- **Chemin d’accès** : Chemin d’accès complet du package Guest Configuration.

Exécutez la commande suivante pour tester le package créé par l’étape précédente :

```azurepowershell-interactive
Test-GuestConfigurationPackage `
  -Path ./AuditFilePathExists/AuditFilePathExists.zip
```

La cmdlet prend aussi en charge l’entrée depuis le pipeline PowerShell. Dirige la sortie de la cmdlet `New-GuestConfigurationPackage` vers la cmdlet `Test-GuestConfigurationPackage`.

```azurepowershell-interactive
New-GuestConfigurationPackage -Name AuditFilePathExists -Configuration ./Config/AuditFilePathExists.mof -ChefInspecProfilePath './' | Test-GuestConfigurationPackage
```

L’étape suivante consiste à publier le fichier dans Stockage Blob Azure.  La commande `Publish-GuestConfigurationPackage` requiert le module `Az.Storage`.

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
    -ContentUri 'https://storageaccountname.blob.core.windows.net/packages/AuditFilePathExists.zip?st=2019-07-01T00%3A00%3A00Z&se=2024-07-01T00%3A00%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=JdUf4nOCo8fvuflOoX%2FnGo4sXqVfP5BYXHzTl3%2BovJo%3D' `
    -DisplayName 'Audit Linux file path.' `
    -Description 'Audit that a file path exists on a Linux machine.' `
    -Path './policies' `
    -Platform 'Linux' `
    -Version 1.0.0 `
    -Verbose
```

Les fichiers suivants sont créés par `New-GuestConfigurationPolicy` :

- **auditIfNotExists.json**

La sortie de la cmdlet retourne un objet contenant le nom complet de l’initiative et le chemin d’accès aux fichiers de stratégie.

Enfin, publiez les définitions de stratégie à l’aide de la cmdlet `Publish-GuestConfigurationPolicy`. La cmdlet ne dispose que du paramètre **Path** qui pointe vers l’emplacement des trois fichiers JSON créés par `New-GuestConfigurationPolicy`.

Pour exécuter la commande Publish, vous devez avoir accès à la création de stratégies dans Azure. Les exigences spécifiques en matière d’autorisations sont documentées dans la page [vue d’ensemble d’Azure Policy](../overview.md). Le meilleur rôle intégré est le rôle **Contributeur de la stratégie de ressource**.

```azurepowershell-interactive
Publish-GuestConfigurationPolicy `
  -Path './policies'
```

 La cmdlet `Publish-GuestConfigurationPolicy` accepte le chemin à partir du pipeline PowerShell. Cette fonctionnalité signifie que vous pouvez créer les fichiers de stratégie et les publier dans un ensemble unique de commandes redirigées.

 ```azurepowershell-interactive
 New-GuestConfigurationPolicy `
  -ContentUri 'https://storageaccountname.blob.core.windows.net/packages/AuditFilePathExists.zip?st=2019-07-01T00%3A00%3A00Z&se=2024-07-01T00%3A00%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=JdUf4nOCo8fvuflOoX%2FnGo4sXqVfP5BYXHzTl3%2BovJo%3D' `
  -DisplayName 'Audit Linux file path.' `
  -Description 'Audit that a file path exists on a Linux machine.' `
  -Path './policies' `
 | Publish-GuestConfigurationPolicy
 ```

Avec la stratégie créée dans Azure, la dernière étape consiste à attribuer la définition. Apprenez à attribuer la définition à l’aide du [portail](../assign-policy-portal.md), d’[Azure CLI](../assign-policy-azurecli.md) et d’[Azure PowerShell](../assign-policy-powershell.md).

### <a name="using-parameters-in-custom-guest-configuration-policies"></a>Utilisation de paramètres dans des stratégies personnalisées Guest Configuration

Guest Configuration prend en charge la substitution des propriétés d’une configuration lors d’une exécution. Cette fonctionnalité signifie que les valeurs du fichier MOF dans le package n’ont pas à être considérées comme statiques. Les valeurs de substitution sont fournies via Azure Policy et n’impactent pas la création ni la compilation des configurations.

Avec InSpec, les paramètres sont généralement gérés en tant qu’entrée au moment de l’exécution ou en tant que code à l’aide d’attributs. La configuration d’invité masque ce processus afin que l’entrée puisse être fournie lors de l’attribution de la stratégie. Un fichier d’attributs est automatiquement créé sur l’ordinateur. Vous n’avez pas besoin de créer et d’ajouter un fichier à votre projet. Il existe deux étapes pour ajouter des paramètres à votre projet d’audit Linux.

Définissez l’entrée dans le fichier Ruby où vous scriptez ce qui doit être audité sur l’ordinateur. Un exemple est fourni ci-dessous.

```Ruby
attr_path = attribute('path', description: 'The file path to validate.')

describe file(attr_path) do
    it { should exist }
end
```

Les cmdlets `New-GuestConfigurationPolicy` et `Test-GuestConfigurationPolicyPackage` incluent un paramètre nommé **Parameter**. Ce paramètre prend une table de hachage qui comprend toutes les informations concernant chaque paramètre et crée automatiquement toutes les sections nécessaires des fichiers utilisés pour créer chaque définition Azure Policy.

L’exemple suivant crée une définition de stratégie pour auditer un chemin d’accès de fichier, que l’utilisateur fournit au moment de l’attribution de la stratégie.

```azurepowershell-interactive
$PolicyParameterInfo = @(
    @{
        Name = 'FilePath'                             # Policy parameter name (mandatory)
        DisplayName = 'File path.'                    # Policy parameter display name (mandatory)
        Description = "File path to be audited."      # Policy parameter description (optional)
        ResourceType = "ChefInSpecResource"           # Configuration resource type (mandatory)
        ResourceId = 'Audit Linux path exists'        # Configuration resource property name (mandatory)
        ResourcePropertyName = "AttributesYmlContent" # Configuration resource property name (mandatory)
        DefaultValue = '/tmp'                         # Policy parameter default value (optional)
    }
)

# The hashtable also supports a property named 'AllowedValues' with an array of strings to limit input to a list

New-GuestConfigurationPolicy
    -ContentUri 'https://storageaccountname.blob.core.windows.net/packages/AuditFilePathExists.zip?st=2019-07-01T00%3A00%3A00Z&se=2024-07-01T00%3A00%3A00Z&sp=rl&sv=2018-03-28&sr=b&sig=JdUf4nOCo8fvuflOoX%2FnGo4sXqVfP5BYXHzTl3%2BovJo%3D' `
    -DisplayName 'Audit Linux file path.' `
    -Description 'Audit that a file path exists on a Linux machine.' `
    -Path './policies' `
    -Parameter $PolicyParameterInfo `
    -Version 1.0.0
```

Pour les stratégies Linux, ajoutez la propriété **AttributesYmlContent** à votre configuration et remplacez les valeurs le cas échéant. L’agent Guest Configuration crée automatiquement le fichier YAML utilisé par InSpec pour stocker les attributs. Reportez-vous à l’exemple ci-dessous.

```powershell
Configuration AuditFilePathExists
{
    Import-DscResource -ModuleName 'GuestConfiguration'

    Node AuditFilePathExists
    {
        ChefInSpecResource 'Audit Linux path exists'
        {
            Name = 'linux-path'
            AttributesYmlContent = "path: /tmp"
        }
    }
}
```

## <a name="policy-lifecycle"></a>Cycle de vie de la stratégie

Pour publier une mise à jour de la définition de stratégie, trois champs sont importants.

> [!NOTE]
> La propriété `version` de l’affectation de configuration invité n’a d’influence que sur les packages qui sont hébergés par Microsoft. La meilleure pratique pour le contenu personnalisé du contrôle de version consiste à inclure la version dans le nom de fichier.

- **Version** : Lorsque vous exécutez l’applet de commande `New-GuestConfigurationPolicy`, vous devez spécifier un numéro de version supérieur à celui actuellement publié.
- **contentUri** : Lorsque vous exécutez la cmdlet `New-GuestConfigurationPolicy`, vous devez spécifier un URI vers l’emplacement du package. L’inclusion d’une version de package dans le nom de fichier garantit que la valeur de cette propriété change dans chaque version.
- **contentHash** : Cette propriété est automatiquement mise à jour par l’applet de commande `New-GuestConfigurationPolicy`. Il s’agit d’une valeur de hachage du package créé par `New-GuestConfigurationPackage`. Cette propriété doit être correcte pour le fichier `.zip` que vous publiez. Si seule la propriété **contentUri** est mise à jour, l’extension n’accepte pas le package de contenu.

Le moyen le plus simple de publier un package mis à jour consiste à répéter le processus décrit dans cet article et à fournir un numéro de version mis à jour. Ce processus garantit que toutes les propriétés ont été correctement mises à jour.

### <a name="filtering-guest-configuration-policies-using-tags"></a>Filtrage des stratégies Guest Configuration à l’aide de balises

Les stratégies créées par les cmdlet dans le module Guest Configuration peuvent éventuellement inclure un filtre pour les balises. Le paramètre **-Tag** de `New-GuestConfigurationPolicy` prend en charge un groupe de tables de hachage contenant des balises individuelles. Les balises sont ajoutées à la section `If` de la définition de stratégie et ne peuvent pas être modifiées par une attribution de stratégie.

Vous trouverez ci-dessous un exemple d’extrait de définition de stratégie qui permettra de filtrer les balises.

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
      // Original Guest Configuration content will follow
    }
  ]
}
```

## <a name="optional-signing-guest-configuration-packages"></a>Facultatif : Signature des packages Guest Configuration

Les stratégies personnalisées Guest Configuration utilisent le hachage SHA256 pour vérifier que le package de stratégie n’a pas changé.
Si vous le souhaitez, les clients peuvent également utiliser un certificat pour signer des packages et forcer l’extension Guest Configuration à autoriser uniquement le contenu signé.

Pour activer ce scénario, vous devez effectuer deux étapes. Exécutez l’applet de commande pour signer le package de contenu et ajoutez une balise aux machines qui doivent nécessiter du code pour être signées.

Pour utiliser la fonctionnalité de validation de signature, exécutez la cmdlet `Protect-GuestConfigurationPackage` pour signer le package avant sa publication. Cette cmdlet nécessite un certificat de « signature du code ».

Paramètres de la cmdlet `Protect-GuestConfigurationPackage` :

- **Chemin d’accès** : Chemin d’accès complet du package Guest Configuration.
- **PublicGpgKeyPath** : Chemin d'accès à la clé GPG publique. Ce paramètre est uniquement pris en charge lors de la signature de contenu pour Linux.

Vous trouverez une bonne référence de création de clés GPG à utiliser avec les machines Linux dans cet article sur GitHub, [Génération d’une clé GPG](https://help.github.com/en/articles/generating-a-new-gpg-key).

L’agent GuestConfiguration s’attend à trouver la clé publique du certificat dans le chemin d’accès `/usr/local/share/ca-certificates/extra` sur des machines Linux. Pour que le nœud vérifie le contenu signé, installez la clé publique du certificat sur la machine avant d’appliquer la stratégie personnalisée. Ce processus peut être effectué à l’aide de n’importe quelle technique à l’intérieur de la machine virtuelle, ou à l’aide d’Azure Policy. Vous trouverez un exemple modèle en suivant [ce lien](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-push-certificate-windows).
La stratégie d’accès Key Vault doit autoriser le fournisseur de ressources de calcul à accéder aux certificats lors des déploiements. Pour les étapes détaillées, consultez [Configurer Key Vault pour des machines virtuelles dans Azure Resource Manager](../../../virtual-machines/windows/key-vault-setup.md#use-templates-to-set-up-key-vault).

Une fois votre contenu publié, ajoutez une balise nommée `GuestConfigPolicyCertificateValidation` et avec une valeur `enabled` à toutes les machines virtuelles où la signature du code doit être requise. Pour plus d'informations sur la façon dont les balises peuvent être délivrées à grande échelle à l'aide d'Azure Policy, consultez les [Exemples de balises](../samples/built-in-policies.md#tags). Une fois cette balise en place, la définition de stratégie générée via la cmdlet `New-GuestConfigurationPolicy` met en œuvre l’exigence via l’extension Guest Configuration.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur l’audit des machines virtuelles avec [Guest Configuration](../concepts/guest-configuration.md).
- Découvrez comment [créer des stratégies par programmation](./programmatically-create.md).
- Découvrez comment [obtenir des données de conformité](./get-compliance-data.md).
