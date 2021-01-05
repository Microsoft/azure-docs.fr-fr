---
title: Activer le Bureau à distance pour un rôle à l’aide de Visual Studio (Azure Cloud Services)
description: Configuration de l’application de service cloud Azure pour autoriser les connexions Bureau à distance
services: cloud-services
author: ghogen
manager: jillfra
ms.assetid: f5727ebe-9f57-4d7d-aff1-58761e8de8c1
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.topic: conceptual
ms.workload: azure-vs
ms.date: 03/06/2018
ms.author: ghogen
ms.openlocfilehash: f4622e44c795182ee68c617f335c9e1651d3adcc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80294388"
---
# <a name="enable-remote-desktop-connection-for-a-role-in-azure-cloud-services-using-visual-studio"></a>Activer une connexion Bureau à distance pour un rôle dans Azure Cloud Services avec Visual Studio

> [!div class="op_single_selector"]
> * [Azure portal](cloud-services-role-enable-remote-desktop-new-portal.md)
> * [PowerShell](cloud-services-role-enable-remote-desktop-powershell.md)
> * [Visual Studio](cloud-services-role-enable-remote-desktop-visual-studio.md)

Le Bureau à distance vous permet d'accéder au bureau d'un rôle en cours d'exécution dans Azure. Vous pouvez utiliser une connexion Bureau à distance pour dépanner et diagnostiquer les problèmes rencontrés par votre application lorsqu'elle est en cours d'exécution.

L’assistant publication que Visual Studio fournit aux services cloud inclut une option pour activer le Bureau à distance pendant le processus de publication à l’aide des informations d’identification que vous fournissez. L’utilisation de cette option est appropriée lorsque vous utilisez Visual Studio 2017 version 15.4 et les versions antérieures.

Avec Visual Studio 2017 version 15.5 et les versions ultérieures, toutefois, il est recommandé d’éviter d’activer le Bureau à distance via l’Assistant publication, sauf si vous êtes l’unique développeur. Pour toute situation dans laquelle le projet peut être ouvert par d’autres développeurs, activez plutôt Bureau à distance via le portail Azure, via PowerShell ou depuis un pipeline de mise en production dans un flux de travail de déploiement continu. Cette recommandation est due à une modification de la façon dont Visual Studio communique avec le Bureau à distance sur le service cloud de la machine virtuelle, comme expliqué dans cet article.

## <a name="configure-remote-desktop-through-visual-studio-2017-version-154-and-earlier"></a>Configurer le Bureau à distance via Visual Studio 2017 version 15.4 et versions antérieures

Lorsque vous utilisez Visual Studio 2017 version 15.4 et les versions antérieures, vous pouvez utiliser l’option **Activer le Bureau à distance pour tous les rôles** dans l’Assistant publication. Vous pouvez toujours utiliser l’Assistant avec Visual Studio 2017 15.5 et versions ultérieures, mais n’utilisez pas l’option Bureau à distance.

1. Dans Visual Studio, démarrez l’Assistant publication en cliquant avec le bouton droit sur votre projet de service cloud dans l’Explorateur de solutions et en choisissant **Publier**.

2. Connectez-vous à votre abonnement Azure si nécessaire, puis sélectionnez **Suivant**.

3. Sur la page **Paramètres**, sélectionnez **Activer le Bureau à distance pour tous les rôles**, puis sélectionnez le lien **Paramètres...**  pour ouvrir la boîte de dialogue **Configuration Bureau à distance**.

4. Au bas de la boîte de dialogue, sélectionnez **Plus d’options**. Cette commande affiche une liste déroulante dans laquelle vous pouvez créer ou sélectionner un certificat afin de chiffrer les informations d’identification lors de la connexion Bureau à distance.

   > [!Note]
   > Les certificats dont vous avez besoin pour une connexion Bureau à distance sont différents de ceux que vous utilisez pour d'autres opérations Azure. Le certificat de l'accès à distance doit avoir une clé privée.

5. Sélectionnez un certificat dans la liste ou choisissez **&lt;Créer... &gt;** . Si vous créez un nouveau certificat, entrez un nom convivial pour le nouveau certificat lorsque vous y êtes invité, puis sélectionnez **OK**. Le nouveau certificat s’affiche dans la liste déroulante.

6. Créez un nom d’utilisateur et un mot de passe. Vous ne pouvez pas utiliser un compte existant. Ne spécifiez pas « Administrateur » comme nom d’utilisateur pour le nouveau compte.

7. Choisissez une date à laquelle le compte expirera et après laquelle les connexions Bureau à distance seront bloquées.

8. Une fois que vous avez fourni toutes les informations requises, sélectionnez le bouton **OK**. Visual Studio ajoute les paramètres du Bureau à distance aux fichiers `.cscfg` et `.csdef` de votre projet, y compris le mot de passe chiffré à l’aide du certificat choisi.

9. Effectuez les étapes restantes à l’aide du bouton **Suivant**, puis sélectionnez **Publier** lorsque vous êtes prêt à publier votre service cloud. Si vous n’êtes pas prêt à publier, sélectionnez **Annuler** et répondez **Oui** lorsque vous êtes invité à enregistrer les modifications. Vous pouvez publier votre service cloud ultérieurement avec ces paramètres.

## <a name="configure-remote-desktop-when-using-visual-studio-2017-version-155-and-later"></a>Configurer le Bureau à distance lorsque vous utilisez Visual Studio 2017 version 15.5 et versions ultérieures

Avec Visual Studio 2017 15.5 et versions ultérieures, vous pouvez toujours utiliser l’Assistant publication avec un projet de service cloud. Vous pouvez également utiliser l’option**Activer le Bureau à distance pour tous les rôles** si vous êtes l’unique développeur.

Si vous travaillez dans le cadre d’une équipe, vous devez activer à la place le Bureau à distance sur le service cloud Azure à l’aide du [portail Azure](cloud-services-role-enable-remote-desktop-new-portal.md) ou de [PowerShell](cloud-services-role-enable-remote-desktop-powershell.md).

Cette recommandation est due à une modification de la façon dont Visual Studio 2017 15.5 et versions ultérieures communiquent avec la machine virtuelle du service cloud. Lorsque vous activez le Bureau à distance via l’Assistant de publication, les versions antérieures de Visual Studio communiquent avec la machine virtuelle via ce qu’on appelle le « plug-in RDP ». Au lieu de cela, Visual Studio 2017 15.5 et versions ultérieures communiquent en utilisant l’extension « RDP » qui est plus sécurisée et plus souple. Ce changement permet également de s’aligner avec les méthodes du portail Azure et de PowerShell, qui utilisent aussi l’extension RDP pour activer le Bureau à distance.

Lorsque Visual Studio communique avec l’extension RDP, il transmet un mot de passe en texte brut via le protocole TLS. Toutefois, les fichiers de configuration du projet stockent uniquement un mot de passe chiffré, qui peut être déchiffré en texte brut uniquement avec le certificat local qui a servi à chiffrer au départ.

Si vous déployez le projet de service cloud à partir du même ordinateur de développement à chaque fois, alors ce certificat local est disponible. Dans ce cas, vous pouvez toujours utiliser l’option **Activer le Bureau à distance pour tous les rôles** ans l’Assistant publication.

Toutefois, si vous ou d’autres développeurs souhaitez déployer le projet de service cloud à partir de différents ordinateurs, ces autres ordinateurs n’ont pas le certificat nécessaire pour décrypter le mot de passe. Par conséquent, le message d’erreur suivant s’affiche :

```output
Applying remote desktop protocol (RDP) extension.
Certificate with thumbprint [thumbprint] doesn't exist.
```

Vous pouvez modifier le mot de passe chaque fois que vous déployez le service cloud, mais cette action n’est pas pratique pour toute personne ayant besoin d’utiliser le Bureau à distance.

Si vous partagez le projet avec une équipe, il est alors préférable de désactiver l’option dans l’Assistant publication et d’activer à la place le Bureau à distance directement via le [portail Azure](cloud-services-role-enable-remote-desktop-new-portal.md) ou à l’aide de [PowerShell](cloud-services-role-enable-remote-desktop-powershell.md).

### <a name="deploying-from-a-build-server-with-visual-studio-2017-version-155-and-later"></a>Déploiement à partir d’un serveur de builds avec Visual Studio 2017 15.5 et versions ultérieures

Vous pouvez déployer un projet de service cloud à partir d’un serveur de builds (par exemple, avec Azure DevOps Services) sur lequel Visual Studio 2017 15.5 ou une version ultérieure est installée sur l’agent de build. Avec cette configuration, le déploiement s’opère dans le même ordinateur que celui sur lequel le certificat de chiffrement est disponible.

Pour utiliser l’extension RDP à partir d’Azure DevOps Services, incluez les détails suivants dans votre pipeline de build :

1. Incluez `/p:ForceRDPExtensionOverPlugin=true` dans vos arguments MSBuild pour vous assurer que le déploiement fonctionne avec l’extension RDP plutôt que le plug-in RDP. Par exemple :

    ```
    msbuild AzureCloudService5.ccproj /t:Publish /p:TargetProfile=Cloud /p:DebugType=None
        /p:SkipInvalidConfigurations=true /p:ForceRDPExtensionOverPlugin=true
    ```

1. Après vos étapes de génération, ajouter l’étape de **déploiement du Service Cloud Azure** et définissez ses propriétés.

1. Après l’étape de déploiement, ajoutez une étape **Azure Powershell**, définissez sa propriété de **nom d’affichage** comme « Déploiement : activer RDP Extension Azure » (ou tout autre nom approprié) et sélectionnez l’abonnement Azure qui convient.

1. Définissez le **Type de Script** à « Inline » et collez le code ci-dessous dans le champ **Script Inline**. (Vous pouvez également créer un fichier `.ps1` dans votre projet avec ce script, définissez le **Type de script** sur « Chemin d’accès de fichier de script » et définissez le **Chemin d’accès du Script** pour qu’il pointe vers le fichier.)

    ```ps
    Param(
        [Parameter(Mandatory=$True)]
        [string]$username,

        [Parameter(Mandatory=$True)]
        [string]$password,

        [Parameter(Mandatory=$True)]
        [string]$serviceName,

        [Datetime]$expiry = ($(Get-Date).AddYears(1))
    )

    Write-Host "Service Name: $serviceName"
    Write-Host "User Name: $username"
    Write-Host "Expiry: $expiry"

    $securepassword = ConvertTo-SecureString -String $password -AsPlainText -Force
    $credential = New-Object System.Management.Automation.PSCredential $username,$securepassword

    # Try to remote existing RDP Extensions
    try
    {
        $existingRDPExtension = Get-AzureServiceRemoteDesktopExtension -ServiceName $servicename
        if ($existingRDPExtension -ne $null)
        {
            Remove-AzureServiceRemoteDesktopExtension -ServiceName $servicename -UninstallConfiguration
        }
    }
    catch
    {
    }

    Set-AzureServiceRemoteDesktopExtension -ServiceName $servicename -Credential $credential -Expiration $expiry -Verbose
    ```

## <a name="connect-to-an-azure-role-by-using-remote-desktop"></a>Se connecter à un rôle Azure à l'aide du Bureau à distance

Une fois que vous publié votre service cloud sur Azure et que vous avez activé le Bureau à distance, vous pouvez utiliser l’Explorateur de serveurs Visual Studio pour vous connecter à la machine virtuelle de service cloud :

1. Dans l’Explorateur de serveurs, développez le nœud **Azure** , puis développez le nœud d’un service cloud et d’un de ses rôles pour afficher une liste d’instances.

2. Cliquez avec le bouton droit sur un nœud d’instance et sélectionnez **Connexion à l’aide du Bureau à distance**.

3. Entrez le nom d'utilisateur et le mot de passe que vous avez créés précédemment. Vous êtes maintenant connecté à votre session à distance.

## <a name="additional-resources"></a>Ressources supplémentaires

[Configuration des services cloud](cloud-services-how-to-configure-portal.md)
