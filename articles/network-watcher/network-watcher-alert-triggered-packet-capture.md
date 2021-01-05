---
title: Utiliser une capture de paquet pour surveiller un réseau proactif avec des alertes - Azure Functions
titleSuffix: Azure Network Watcher
description: Cet article décrit la création d’une capture de paquets déclenchée par des alertes avec Azure Network Watcher
services: network-watcher
documentationcenter: na
author: damendo
ms.assetid: 75e6e7c4-b3ba-4173-8815-b00d7d824e11
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: damendo
ms.openlocfilehash: 3b6cb195f44bf6c868402481480d9b10802c4d59
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94965667"
---
# <a name="use-packet-capture-for-proactive-network-monitoring-with-alerts-and-azure-functions"></a>Utiliser une capture de paquets pour effectuer une surveillance proactive du réseau avec des alertes et Azure Functions

La fonctionnalité de capture de paquets Network Watcher crée des sessions de capture afin d’effectuer le suivi du trafic en direction et en provenance des machines virtuelles. Le fichier de capture peut contenir un filtre défini pour suivre uniquement le trafic que vous souhaitez analyser. Ces données sont ensuite stockées dans un objet blob de stockage ou localement sur l’ordinateur invité.

Cette fonctionnalité peut être démarrée à distance à partir d’autres scénarios d’automatisation comme Azure Functions. La capture de paquets vous permet d’exécuter des captures proactives en fonction des anomalies définies sur le réseau. Elle permet aussi de collecter des statistiques réseau, d’obtenir des informations sur les intrusions, de déboguer des communications client-serveur, etc.

Les ressources qui sont déployées dans Azure s’exécutent 24 heures sur 24, 7 jours sur 7. Ni votre équipe ni vous ne pouvez surveiller activement l’état de toutes les ressources 24 heures sur 24, 7 jours sur 7. Par exemple, que se passe-t-il si un problème se produit à 2 h ?

En utilisant Network Watcher, les alertes et les fonctions dans l’écosystème Azure, vous pouvez répondre de façon proactive aux problèmes de votre réseau avec les données et les outils.

![Le diagramme montre une extension Network Watcher sur une machine virtuelle qui est acheminée vers une erreur de type Segments TCP envoyés > 100, qui est transmise à Azure Functions, puis à Network Watcher et qui revient à l’extension Network Watcher.][scenario]


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Conditions préalables requises

* La version la plus récente [d’Azure PowerShell](/powershell/azure/install-Az-ps).
* Une instance existante de Network Watcher. Si vous n’en avez pas, [créez une instance de Network Watcher](network-watcher-create.md).
* Une machine virtuelle existante dans la même région que Network Watcher, avec [l’extension Windows](../virtual-machines/extensions/network-watcher-windows.md) ou [l’extension de machine virtuelle Linux](../virtual-machines/extensions/network-watcher-linux.md).

## <a name="scenario"></a>Scénario

Dans cet exemple, votre machine virtuelle envoie plus de segments TCP que d’habitude, et vous souhaitez être alerté. Les segments TCP sont utilisés ici à titre d’exemple, mais vous pouvez utiliser n’importe quelle condition d’alerte.

Lorsque vous recevez une alerte, il est intéressant d’obtenir des données au niveau des paquets afin de comprendre pourquoi la communication a augmenté. Vous pouvez ensuite prendre des mesures pour rétablir une communication régulière sur la machine virtuelle.

Ce scénario suppose que vous possédez une instance existante de Network Watcher, ainsi qu’un groupe de ressources avec une machine virtuelle valide.

La liste suivante est une vue d’ensemble du flux de travail qui se produit :

1. Une alerte est déclenchée sur votre machine virtuelle.
1. L’alerte appelle votre fonction Azure via un webhook.
1. Votre fonction Azure traite l’alerte et démarre une session de capture de paquets Network Watcher.
1. La capture de paquets s’exécute sur la machine virtuelle et collecte le trafic.
1. Le fichier de capture de paquets est chargé vers un compte de stockage afin de subir un examen et un diagnostic.

Pour automatiser ce processus, nous créons et connectons une alerte sur notre machine virtuelle pour qu’elle se déclenche lorsque l’incident se produit. Nous créons également une fonction pour appeler Network Watcher.

Ce scénario :

* crée une fonction Azure qui démarrera une capture de paquets ;
* crée une règle d’alerte sur une machine virtuelle et configure la règle d’alerte pour appeler la fonction Azure.

## <a name="create-an-azure-function"></a>Création d’une fonction Azure

La première étape consiste à créer une fonction Azure pour traiter l’alerte et créer une capture de paquets.

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez **Créer une ressource** > **Compute** > **Function App**.

    ![Création d’une application de fonction][1-1]

2. Dans le panneau **Function App**, entrez les valeurs suivantes puis sélectionnez **OK** pour créer l’application :

    |**Paramètre** | **Valeur** | **Détails** |
    |---|---|---|
    |**Nom de l’application**|PacketCaptureExample|Le nom de l’application de fonction.|
    |**Abonnement**|[Votre abonnement] L’abonnement dans lequel créer l’application de fonction.||
    |**Groupe de ressources**|PacketCaptureRG|Le groupe de ressources qui contiendra l’application de fonction.|
    |**Plan d’hébergement**|Plan de consommation| Le type de plan qu’utilise votre application de fonction. Deux options sont disponibles : Consommation ou Plan Azure App Service. |
    |**Lieu**|USA Centre| La région dans laquelle créer l’application de fonction.|
    |**Compte de stockage**|{généré automatiquement}| Le compte de stockage dont Azure Functions a besoin pour le stockage général.|

3. Dans le panneau **Function App PacketCaptureExample**, sélectionnez **Fonctions** > **Fonction personnalisée** > **+** .

4. Sélectionnez **HttpTrigger-Powershell**, puis entrez les informations restantes. Enfin, pour créer la fonction, sélectionnez **Créer**.

    |**Paramètre** | **Valeur** | **Détails** |
    |---|---|---|
    |**Scénario**|Expérimental|Type de scénario|
    |**Nommer votre fonction**|AlertPacketCapturePowerShell|Nom de la fonction|
    |**Niveau d’autorisation**|Fonction|Niveau d’autorisation de la fonction|

![Exemples de fonctions][functions1]

> [!NOTE]
> Le modèle PowerShell est expérimental et n’a pas de prise en charge complète.

Les personnalisations sont requises pour cet exemple et sont décrites dans les étapes suivantes.

### <a name="add-modules"></a>Ajouter des modules

Pour utiliser les cmdlets PowerShell de Network Watcher, chargez le module PowerShell le plus récent dans l’application de fonction.

1. Sur votre ordinateur local avec les derniers modules Azure PowerShell installés, exécutez la commande PowerShell suivante :

    ```powershell
    (Get-Module Az.Network).Path
    ```

    Cet exemple vous donne le chemin local de vos modules Azure PowerShell. Ces dossiers sont utilisés dans une étape ultérieure. Les modules utilisés dans ce scénario sont :

   * Az.Network

   * Az.Accounts

   * Az.Resources

     ![Dossiers PowerShell][functions5]

1. Sélectionnez **Paramètres Function App** > **Accéder à l’Éditeur App Service**.

    ![Paramètres Function App][functions2]

1. Cliquez avec le bouton droit sur le dossier **AlertPacketCapturePowershell** et créez un dossier nommé **azuremodules**. 

4. Créez un sous-dossier pour chacun des modules dont vous avez besoin.

    ![Dossier et sous-dossiers][functions3]

    * Az.Network

    * Az.Accounts

    * Az.Resources

1. Cliquez avec le bouton droit sur le sous-dossier **Az.Network**, puis sélectionnez **Charger des fichiers**. 

6. Accédez à vos modules Azure. Dans le dossier **Az.Network** local, sélectionnez tous les fichiers. Sélectionnez ensuite **OK**. 

7. Répétez ces étapes pour **Az.Accounts** et **Az.Resources**.

    ![Charger des fichiers][functions6]

1. Lorsque vous avez terminé, chaque dossier doit contenir les fichiers du module PowerShell de votre ordinateur local.

    ![Fichiers PowerShell][functions7]

### <a name="authentication"></a>Authentication

Pour utiliser les applets de commande PowerShell, vous devez vous authentifier. Vous configurez l’authentification dans l’application de fonction. Pour ce faire, vous devez configurer les variables d’environnement et charger un fichier de clé chiffré dans l’application de fonction.

> [!NOTE]
> Ce scénario ne fournit qu’un exemple d’implémentation de l’authentification avec Azure Functions. Il existe d’autres façons de procéder.

#### <a name="encrypted-credentials"></a>Informations d’identification chiffrées

Le script PowerShell suivant crée un fichier de clé appelé **PassEncryptKey.key**. Il fournit également une version chiffrée du mot de passe qui est donné. Ce mot de passe est le même que celui défini pour l’application Azure Active Directory qui est utilisée pour l’authentification.

```powershell
#Variables
$keypath = "C:\temp\PassEncryptKey.key"
$AESKey = New-Object Byte[] 32
$Password = "<insert a password here>"

#Keys
[Security.Cryptography.RNGCryptoServiceProvider]::Create().GetBytes($AESKey) 
Set-Content $keypath $AESKey

#Get encrypted password
$secPw = ConvertTo-SecureString -AsPlainText $Password -Force
$AESKey = Get-content $KeyPath
$Encryptedpassword = $secPw | ConvertFrom-SecureString -Key $AESKey
$Encryptedpassword
```

Dans l’éditeur App Service de l’application de fonction, créez un dossier nommé **clés** sous **AlertPacketCapturePowerShell**. Chargez ensuite le fichier **PassEncryptKey.key** que vous avez créé dans l’exemple PowerShell précédent.

![Clé de fonctions][functions8]

### <a name="retrieve-values-for-environment-variables"></a>Récupérer des valeurs pour les variables d’environnement

La dernière exigence consiste à configurer les variables d’environnement nécessaires pour accéder aux valeurs pour l’authentification. Voici une liste des variables d’environnement qui sont créées :

* AzureClientID

* AzureTenant

* AzureCredPassword


#### <a name="azureclientid"></a>AzureClientID

L’ID client est l’ID d’application d’une application dans Azure Active Directory.

1. Si vous ne disposez pas d’une application à utiliser, exécutez l’exemple suivant pour créer une application.

    ```powershell
    $app = New-AzADApplication -DisplayName "ExampleAutomationAccount_MF" -HomePage "https://exampleapp.com" -IdentifierUris "https://exampleapp1.com/ExampleFunctionsAccount" -Password "<same password as defined earlier>"
    New-AzADServicePrincipal -ApplicationId $app.ApplicationId
    Start-Sleep 15
    New-AzRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $app.ApplicationId
    ```

   > [!NOTE]
   > Le mot de passe que vous utilisez lors de la création de l’application doit être le même que celui créé précédemment lors de l’enregistrement du fichier de clé.

1. Dans le portail Azure, sélectionnez **Abonnements**. Sélectionnez l’abonnement à utiliser, puis sélectionnez **Contrôle d’accès (IAM)** .

    ![IAM de fonctions][functions9]

1. Choisissez le compte à utiliser, puis sélectionnez **Propriétés**. Copiez l’ID de l’application.

    ![ID de l’application de fonctions][functions10]

#### <a name="azuretenant"></a>AzureTenant

Obtenez l’ID de locataire en exécutant l’exemple PowerShell suivant :

```powershell
(Get-AzSubscription -SubscriptionName "<subscriptionName>").TenantId
```

#### <a name="azurecredpassword"></a>AzureCredPassword

La valeur de la variable d’environnement AzureCredPassword est celle obtenue en exécutant l’exemple PowerShell suivant. Cet exemple est identique à celui indiqué dans la section **Informations d’identification chiffrées** précédente. La valeur requise est le résultat de la variable `$Encryptedpassword`.  Ceci est le mot de passe du principal de service que vous avez chiffré à l’aide du script PowerShell.

```powershell
#Variables
$keypath = "C:\temp\PassEncryptKey.key"
$AESKey = New-Object Byte[] 32
$Password = "<insert a password here>"

#Keys
[Security.Cryptography.RNGCryptoServiceProvider]::Create().GetBytes($AESKey) 
Set-Content $keypath $AESKey

#Get encrypted password
$secPw = ConvertTo-SecureString -AsPlainText $Password -Force
$AESKey = Get-content $KeyPath
$Encryptedpassword = $secPw | ConvertFrom-SecureString -Key $AESKey
$Encryptedpassword
```

### <a name="store-the-environment-variables"></a>Stocker les variables d’environnement

1. Accédez à l’application de fonction. Sélectionnez ensuite **Paramètres Function App** > **Configurer les paramètres de l’application**.

    ![Configuration des paramètres d’application][functions11]

1. Ajoutez les variables d’environnement et leurs valeurs aux paramètres de l’application, puis sélectionnez **Enregistrer**.

    ![Paramètres de l’application][functions12]

### <a name="add-powershell-to-the-function"></a>Ajout de PowerShell à la fonction

À présent, il est temps d’appeler Network Watcher à partir de la fonction Azure. Selon les besoins, l’implémentation de cette fonction peut varier. Toutefois, le flux général du code est comme suit :

1. Traiter les paramètres d’entrée.
2. Interroger les captures de paquets existantes pour vérifier les limites et résoudre les conflits de noms.
3. Créer une capture de paquets avec les paramètres appropriés.
4. Sonder régulièrement la capture de paquets jusqu’à la fin.
5. Informer l’utilisateur de la fin de la session de capture de paquets.

L’exemple suivant correspond à du code PowerShell qui peut être utilisé dans la fonction. Il y a des valeurs qui doivent être remplacées pour **subscriptionId**, **resourceGroupName** et **storageAccountName**.

```powershell
            #Import Azure PowerShell modules required to make calls to Network Watcher
            Import-Module "D:\home\site\wwwroot\AlertPacketCapturePowerShell\azuremodules\Az.Accounts\Az.Accounts.psd1" -Global
            Import-Module "D:\home\site\wwwroot\AlertPacketCapturePowerShell\azuremodules\Az.Network\Az.Network.psd1" -Global
            Import-Module "D:\home\site\wwwroot\AlertPacketCapturePowerShell\azuremodules\Az.Resources\Az.Resources.psd1" -Global

            #Process alert request body
            $requestBody = Get-Content $req -Raw | ConvertFrom-Json

            #Storage account ID to save captures in
            $storageaccountid = "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}"

            #Packet capture vars
            $packetcapturename = "PSAzureFunction"
            $packetCaptureLimit = 10
            $packetCaptureDuration = 10

            #Credentials
            $tenant = $env:AzureTenant
            $pw = $env:AzureCredPassword
            $clientid = $env:AzureClientId
            $keypath = "D:\home\site\wwwroot\AlertPacketCapturePowerShell\keys\PassEncryptKey.key"

            #Authentication
            $secpassword = $pw | ConvertTo-SecureString -Key (Get-Content $keypath)
            $credential = New-Object System.Management.Automation.PSCredential ($clientid, $secpassword)
            Connect-AzAccount -ServicePrincipal -Tenant $tenant -Credential $credential #-WarningAction SilentlyContinue | out-null


            #Get the VM that fired the alert
            if($requestBody.context.resourceType -eq "Microsoft.Compute/virtualMachines")
            {
                Write-Output ("Subscription ID: {0}" -f $requestBody.context.subscriptionId)
                Write-Output ("Resource Group:  {0}" -f $requestBody.context.resourceGroupName)
                Write-Output ("Resource Name:  {0}" -f $requestBody.context.resourceName)
                Write-Output ("Resource Type:  {0}" -f $requestBody.context.resourceType)

                #Get the Network Watcher in the VM's region
                $networkWatcher = Get-AzResource | Where {$_.ResourceType -eq "Microsoft.Network/networkWatchers" -and $_.Location -eq $requestBody.context.resourceRegion}

                #Get existing packetCaptures
                $packetCaptures = Get-AzNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher

                #Remove existing packet capture created by the function (if it exists)
                $packetCaptures | %{if($_.Name -eq $packetCaptureName)
                { 
                    Remove-AzNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher -PacketCaptureName $packetCaptureName
                }}

                #Initiate packet capture on the VM that fired the alert
                if ((Get-AzNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher).Count -lt $packetCaptureLimit){
                    echo "Initiating Packet Capture"
                    New-AzNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher -TargetVirtualMachineId $requestBody.context.resourceId -PacketCaptureName $packetCaptureName -StorageAccountId $storageaccountid -TimeLimitInSeconds $packetCaptureDuration
                    Out-File -Encoding Ascii -FilePath $res -inputObject "Packet Capture created on ${requestBody.context.resourceID}"
                }
            } 
 ``` 
#### <a name="retrieve-the-function-url"></a>Récupérer l’URL de fonction 
1. Une fois que vous avez créé votre fonction, configurez votre alerte pour appeler l’URL associée à la fonction. Pour obtenir cette valeur, copiez l’URL de la fonction à partir de votre Function App.

    ![Recherche de l’URL de la fonction][functions13]

2. Copiez l’URL de la fonction pour votre application de fonction.

    ![Copie de l’URL de la fonction][2]

Si vous avez besoin de propriétés personnalisées dans la charge utile de la demande POST du webhook, consultez l’article [Configurer un webhook sur une alerte de métrique Azure](../azure-monitor/platform/alerts-webhooks.md).

## <a name="configure-an-alert-on-a-vm"></a>Configurer une alerte sur une machine virtuelle

Des alertes peuvent être configurées pour informer qu’une mesure spécifique dépasse un seuil qui lui a été affecté. Dans cet exemple, l’alerte est définie sur les segments TCP envoyés, mais elle peut être déclenchée pour plusieurs autres mesures. Dans cet exemple, une alerte est configurée pour appeler un webhook afin qu’il appelle la fonction.

### <a name="create-the-alert-rule"></a>Créer la règle d’alerte

Accédez à une machine virtuelle existante, puis ajoutez une règle d’alerte. Pour accéder à une documentation plus détaillée sur la configuration des alertes, consultez l’article [Créer des alertes dans Azure Monitor pour les services Azure - Portail Azure](../azure-monitor/platform/alerts-classic-portal.md). Entrez les valeurs suivantes dans le panneau **Règle d’alerte**, puis sélectionnez **OK**.

  |**Paramètre** | **Valeur** | **Détails** |
  |---|---|---|
  |**Nom**|TCP_Segments_Sent_Exceeded|Nom de la règle d’alerte.|
  |**Description**|Les segments TCP envoyés ont dépassé le seuil|Description de la règle d’alerte.|
  |**Mesure**|Segments TCP envoyés| Mesure à utiliser pour déclencher l’alerte. |
  |**Condition**|Supérieur à| Condition à utiliser lors de l’évaluation de la mesure.|
  |**Seuil**|100| La valeur de la métrique qui déclenche l’alerte. Cette valeur doit être définie sur une valeur valide pour votre environnement.|
  |**Période**|Dans les cinq dernières minutes| Détermine la période pendant laquelle rechercher le seuil de la mesure.|
  |**Webhook**|[URL du webhook de l’application de fonction]| L’URL du webhook de l’application de fonction créée dans les étapes précédentes.|

> [!NOTE]
> Par défaut, la métrique des segments TCP n’est pas activée. Pour en savoir plus sur l’activation de métriques supplémentaires, consultez [Prise en main d’Azure Monitor](../azure-monitor/overview.md).

## <a name="review-the-results"></a>Passer en revue les résultats.

Une fois les critères d’alerte remplis, une capture de paquets est créée. Accédez à Network Watcher, puis sélectionnez **Capture de paquets**. Sur cette page, vous pouvez sélectionner le lien de fichier de capture de paquets pour télécharger la capture de paquets.

![Afficher une capture de paquets][functions14]

Si le fichier de capture est stocké en local, vous pouvez le récupérer en vous connectant à la machine virtuelle.

Pour obtenir des instructions sur le téléchargement de fichiers à partir de comptes de stockage Azure, consultez [Prise en main du stockage d’objets blob Azure à l’aide de .NET](../storage/blobs/storage-quickstart-blobs-dotnet.md). Vous pouvez aussi utiliser [l’Explorateur de stockage](https://storageexplorer.com/).

Une fois la capture téléchargée, vous pouvez l’afficher à l’aide de n’importe quel outil en mesure de lire un fichier **.cap**. Les liens de deux de ces outils sont indiqués ci-après :

- [Microsoft Message Analyzer](/message-analyzer/microsoft-message-analyzer-operating-guide)
- [WireShark](https://www.wireshark.org/)

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment afficher vos captures de paquets en consultant l’article [Packet capture analysis with Wireshark (Analyse de la capture de paquets avec Wireshark)](network-watcher-deep-packet-inspection.md).


[1]: ./media/network-watcher-alert-triggered-packet-capture/figure1.png
[1-1]: ./media/network-watcher-alert-triggered-packet-capture/figure1-1.png
[2]: ./media/network-watcher-alert-triggered-packet-capture/figure2.png
[3]: ./media/network-watcher-alert-triggered-packet-capture/figure3.png
[functions1]:./media/network-watcher-alert-triggered-packet-capture/functions1.png
[functions2]:./media/network-watcher-alert-triggered-packet-capture/functions2.png
[functions3]:./media/network-watcher-alert-triggered-packet-capture/functions3.png
[functions4]:./media/network-watcher-alert-triggered-packet-capture/functions4.png
[functions5]:./media/network-watcher-alert-triggered-packet-capture/functions5.png
[functions6]:./media/network-watcher-alert-triggered-packet-capture/functions6.png
[functions7]:./media/network-watcher-alert-triggered-packet-capture/functions7.png
[functions8]:./media/network-watcher-alert-triggered-packet-capture/functions8.png
[functions9]:./media/network-watcher-alert-triggered-packet-capture/functions9.png
[functions10]:./media/network-watcher-alert-triggered-packet-capture/functions10.png
[functions11]:./media/network-watcher-alert-triggered-packet-capture/functions11.png
[functions12]:./media/network-watcher-alert-triggered-packet-capture/functions12.png
[functions13]:./media/network-watcher-alert-triggered-packet-capture/functions13.png
[functions14]:./media/network-watcher-alert-triggered-packet-capture/functions14.png
[scenario]:./media/network-watcher-alert-triggered-packet-capture/scenario.png