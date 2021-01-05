---
title: Tutoriel - Commander Azure Data Box | Microsoft Docs
description: Ce tutoriel présente Azure Data Box, une solution hybride qui vous permet d’importer des données locales dans Azure, et explique comment commander Azure Data Box.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: tutorial
ms.date: 11/19/2020
ms.author: alkohli
ms.openlocfilehash: db97a34a79ac45525177a79a59a17426003aeab0
ms.sourcegitcommit: ab94795f9b8443eef47abae5bc6848bb9d8d8d01
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/27/2020
ms.locfileid: "96302617"
---
# <a name="tutorial-order-azure-data-box"></a>Tutoriel : Commander Azure Data Box

Azure Data Box est une solution hybride qui vous permet d’importer vos données locales dans Azure de manière rapide, simple et fiable. Vous transférez vos données vers un périphérique de stockage de 80 To (capacité utilisable) fourni par Microsoft, puis renvoyez l’appareil. Ces données sont ensuite chargées dans Azure.

Ce tutoriel explique comment commander une Azure Data Box. Ce tutoriel vous fournira des informations sur :

> [!div class="checklist"]
>
> * Conditions préalables au déploiement de la Data Box
> * Commander une Data Box
> * Suivre la commande
> * Annuler la commande

## <a name="prerequisites"></a>Prérequis

# <a name="portal"></a>[Portail](#tab/portal)

Respectez les prérequis de configuration suivants pour l’appareil et le service Data Box avant de déployer l’appareil :

[!INCLUDE [Prerequisites](../../includes/data-box-deploy-ordered-prerequisites.md)]

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

[!INCLUDE [Prerequisites](../../includes/data-box-deploy-ordered-prerequisites.md)]

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

Vous pouvez vous connecter à Azure et exécuter des commandes Azure CLI de l’une des deux façons suivantes :

* Vous pouvez installer l’interface CLI et exécuter des commandes CLI localement.
* Vous pouvez exécuter des commandes CLI à partir du Portail Azure, dans Azure Cloud Shell.

Nous utilisons Azure CLI par le biais de Windows PowerShell pour le tutoriel, mais vous êtes libre de choisir l’une ou l’autre option.

### <a name="for-azure-cli"></a>Pour Azure CLI

Avant de commencer, assurez-vous que :

#### <a name="install-the-cli-locally"></a>Installer la CLI localement

* Installez [Azure CLI](/cli/azure/install-azure-cli) version 2.0.67 ou ultérieure. Vous pouvez également effectuer l’[installation à l’aide de MSI](https://aka.ms/installazurecliwindows).

**Se connecter à Azure**

Ouvrez une fenêtre de commande Windows PowerShell, puis connectez-vous à Azure avec la commande [az login](/cli/azure/reference-index#az-login) :

```azurecli
PS C:\Windows> az login
```

Voici la sortie d’une connexion réussie :

```output
You have logged in. Now let us find all the subscriptions to which you have access.
[
   {
      "cloudName": "AzureCloud",
      "homeTenantId": "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
      "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "isDefault": true,
      "managedByTenants": [],
      "name": "My Subscription",
      "state": "Enabled",
      "tenantId": "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
      "user": {
          "name": "gusp@contoso.com",
          "type": "user"
      }
   }
]
```

**Installer l’extension CLI Azure Data Box**

Avant de pouvoir utiliser les commandes CLI Azure Data Box, vous devez installer l’extension. Les extensions Azure CLI vous donnent accès à des commandes expérimentales et en préversion qui ne sont pas encore offertes par l’interface CLI principale. Pour plus d’informations sur les extensions, consultez [Utiliser des extensions avec Azure CLI](/cli/azure/azure-cli-extensions-overview).

Pour installer l’extension pour Azure Data Box, exécutez la commande suivante : `az extension add --name databox` :

```azurecli

    PS C:\Windows> az extension add --name databox
```

Si l’extension est correctement installée, la sortie suivante s’affiche :

```output
    The installed extension 'databox' is experimental and not covered by customer support. Please use with discretion.
    PS C:\Windows>

    # az databox help

    PS C:\Windows> az databox -h

    Group
        az databox

    Subgroups:
        job [Experimental] : Commands to manage databox job.

    For more specific examples, use: az find "az databox"

        Please let us know how we are doing: https://aka.ms/clihats
```

#### <a name="use-azure-cloud-shell"></a>Utiliser Azure Cloud Shell

Vous pouvez utiliser [Azure Cloud Shell](https://shell.azure.com/), un environnement d’interpréteur de commandes interactif hébergé Azure, par le biais de votre navigateur pour exécuter des commandes CLI. Azure Cloud Shell prend en charge Bash ou Windows PowerShell avec les services Azure. L’interface Azure CLI est préinstallée et configurée pour être utilisée avec votre compte. Sélectionnez le bouton Cloud Shell du menu situé dans la section supérieure droite du portail Azure :

![Sélection du menu Cloud Shell](../storage/common/media/storage-quickstart-create-account/cloud-shell-menu.png)

Ce bouton lance un interpréteur de commandes interactif que vous pouvez utiliser pour exécuter les étapes décrites dans cet article de guide pratique.

# <a name="powershell"></a>[PowerShell](#tab/azure-ps)

[!INCLUDE [Prerequisites](../../includes/data-box-deploy-ordered-prerequisites.md)]

### <a name="for-azure-powershell"></a>Pour Azure PowerShell

Avant de commencer, vous devez :

* Installer Windows PowerShell version 6.2.4 ou supérieure
* Installer le module Azure PowerShell (AZ)
* Installer le module Azure Data Box (Az.DataBox)
* Connectez-vous à Azure.

#### <a name="install-azure-powershell-and-modules-locally"></a>Installer Azure PowerShell et les modules localement

**Installer ou mettre à niveau Windows PowerShell**

Windows PowerShell version 6.2.4 ou supérieure doit être installé. Pour connaître la version de PowerShell installée, exécutez : `$PSVersionTable`.

La sortie suivante s’affiche :

```azurepowershell
    PS C:\users\gusp> $PSVersionTable
    
    Name                           Value
    ----                           -----
    PSVersion                      6.2.3
    PSEdition                      Core
    GitCommitId                    6.2.3
    OS                             Microsoft Windows 10.0.18363
    Platform                       Win32NT
    PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0…}
    PSRemotingProtocolVersion      2.3
    SerializationVersion           1.1.0.1
    WSManStackVersion              3.0
```

Si votre version de Windows PowerShell est inférieure à la version 6.2.4, vous devez la mettre à niveau. Pour installer la dernière version de Windows PowerShell, consultez [Installation d’Azure PowerShell](/powershell/scripting/install/installing-powershell?view=powershell-7&preserve-view=true).

**Installer Azure PowerShell et les modules Data Box**

Vous devez installer les modules Azure PowerShell afin d’utiliser Azure PowerShell pour commander un service Azure Data Box. Pour installer les modules Azure PowerShell :

1. Installez le [module Azure PowerShell Az](/powershell/azure/new-azureps-module-az).
2. Installez ensuite Az.DataBox à l’aide de la commande `Install-Module -Name Az.DataBox`.

```azurepowershell
PS C:\PowerShell\Modules> Install-Module -Name Az.DataBox
PS C:\PowerShell\Modules> Get-InstalledModule -Name "Az.DataBox"

Version              Name                                Repository           Description
-------              ----                                ----------           -----------
0.1.1                Az.DataBox                          PSGallery            Microsoft Azure PowerShell - DataBox ser…
```

#### <a name="sign-in-to-azure"></a>Connexion à Azure

Ouvrez une fenêtre de commande Windows PowerShell, puis connectez-vous à Azure avec la commande [Connect-AzAccount](/powershell/module/az.accounts/Connect-AzAccount) :

```azurepowershell
PS C:\Windows> Connect-AzAccount
```

Voici la sortie d’une connexion réussie :

```output
WARNING: To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code FSBFZMBKC to authenticate.

Account              SubscriptionName                          TenantId                             Environment
-------              ----------------                          --------                             -----------
gusp@contoso.com     MySubscription                            aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa AzureCloud

PS C:\Windows\System32>
```

Pour plus d’informations sur la connexion à Azure à l’aide de Windows PowerShell, consultez [Se connecter avec Azure PowerShell](/powershell/azure/authenticate-azureps).

---

## <a name="order-data-box"></a>Commander une Data Box

# <a name="portal"></a>[Portail](#tab/portal)

Effectuez les étapes suivantes dans le portail Azure pour commander un appareil.

1. Utilisez vos informations d’identification Microsoft Azure pour vous connecter à cette URL : [https://portal.azure.com](https://portal.azure.com).
2. Sélectionnez **+ Créer une ressource**, puis recherchez *Azure Data Box*. Sélectionnez **Azure Data Box**.

   ![Capture d’écran de la section Nouveau avec Azure Data Box dans le champ de recherche](media/data-box-deploy-ordered/select-data-box-import-02.png)

3. Sélectionnez **Create** (Créer).

   ![Capture d’écran de la section Azure Data Box avec l’option Créer en évidence](media/data-box-deploy-ordered/select-data-box-import-03.png)

4. Vérifiez si le service Data Box est disponible dans votre région. Entrez ou sélectionnez les informations suivantes, puis sélectionnez **Appliquer**.

    |Paramètre  |Valeur  |
    |---------|---------|
    |Type de transfert     | Sélectionnez **Importer vers Azure**.        |
    |Abonnement     | Sélectionnez un abonnement EA, CSP ou Azure pour le service Data Box. <br> L’abonnement est lié à votre compte de facturation.       |
    |Groupe de ressources | Sélectionnez un groupe de ressources existant. Un groupe de ressources est un conteneur logique pour les ressources qui peuvent être gérées ou déployées ensemble. |
    |Pays/région source    |    Sélectionnez le pays ou la région où vos données se trouvent actuellement.         |
    |Région Azure de destination     |     Sélectionnez la région Azure où vous souhaitez transférer des données. <br> Pour plus d’informations, consultez la [disponibilité des régions](data-box-overview.md#region-availability).            |

    [![Démarrage d’une commande d’importation Azure Data Box](media/data-box-deploy-ordered/select-data-box-import-04-b.png#lightbox)

5. Sélectionnez **Data Box**. La capacité maximale utilisable pour une commande unique est de 80 To. Vous pouvez créer plusieurs commandes pour des tailles de données supérieures.

    ![Tailles de données disponibles : Data Box Disk 40 téraoctets ; Data Box 100 téraoctets ; Data Box Heavy, 1000 téraoctets ; Envoyer vos propres disques 1 téraoctet.](media/data-box-deploy-ordered/select-data-box-import-05.png)

6. Sous **Ordre**, accédez à l’onglet **Fonctions de base**. Entrez ou sélectionnez les informations suivantes, puis sélectionnez **Suivant : Destination des données>** .

    |Paramètre  |Valeur  |
    |---------|---------|
    |Abonnement      | L’abonnement est automatiquement renseigné en fonction de la sélection antérieure.|
    |Resource group    | Groupe de ressources que vous avez sélectionné précédemment. |
    |Nom de la commande d’importation | Indiquez un nom convivial pour suivre la commande. <br> Le nom peut comporter entre 3 et 24 caractères qui peuvent être des lettres, des chiffres et des traits d’union. <br> Il doit commencer et se terminer par une lettre ou un chiffre.    |

    ![Assistant Commande d’importation Data Box, écran De base, avec informations correctes renseignées](media/data-box-deploy-ordered/select-data-box-import-06.png)<!--Generic subscription. Cut note. Box command.-->

7. Dans l’écran **Destination des données**, sélectionnez la **Destination des données** : soit les comptes de stockage, soit les disques managés.

    Si vous utilisez un ou plusieurs **comptes de stockage** comme destination de stockage, vous voyez s’afficher l’écran suivant :

    ![Assistant Commande d’importation Data Box, écran Destination des données, avec comptes de stockage sélectionnés](media/data-box-deploy-ordered/select-data-box-import-07.png)

    Selon la région Azure spécifiée, sélectionnez un ou plusieurs comptes de stockage dans la liste filtrée d’un compte de stockage existant. La Data Box peut être liée à 10 comptes de stockage maximum. Vous pouvez également créer un **compte de stockage blob**, **Usage général v1** ou **Usage général v2**.

    Les comptes de stockage avec des réseaux virtuels sont pris en charge. Pour autoriser le service Data Box à travailler avec des comptes de stockage sécurisés, activez les services approuvés dans les paramètres de pare-feu réseau du compte de stockage. Pour plus d’informations, découvrez comment [Ajouter Azure Data Box en tant que service approuvé](../storage/common/storage-network-security.md#exceptions).

    Si vous utilisez Data Box pour créer un ou plusieurs **disques managés** à partir de disques durs virtuels (VHD) locaux, vous devez également fournir les informations suivantes :

    |Paramètre  |Valeur  |
    |---------|---------|
    |Groupes de ressources     | Créez un groupe de ressources si vous envisagez de créer des disques managés à partir de disques durs virtuels locaux. Vous pouvez utiliser un groupe de ressources existant uniquement si celui-ci a été créé avant la création d’une commande Data Box de disques managés par le service Data Box. <br> Spécifiez plusieurs groupes de ressources séparés par des points-virgules. Un maximum de 10 groupes de ressources sont pris en charge.|

    ![Assistant Commande d’importation Data Box, écran Destination des données, avec Disques managés sélectionné](media/data-box-deploy-ordered/select-data-box-import-07-b.png)

    Le compte de stockage spécifié pour les disques managés est utilisé comme compte de stockage intermédiaire. Le service Data Box charge les disques durs virtuels en tant qu’objets blob de pages sur le compte de stockage intermédiaire avant de convertir celui-ci en disques managés et de le déplacer vers les groupes de ressources. Pour plus d’informations, voir [Vérifier le chargement des données dans Azure](data-box-deploy-picked-up.md#verify-data-upload-to-azure).

   > [!NOTE]
   > Si un objet blob de pages n’est pas correctement converti en disque managé, il reste dans le compte de stockage et des frais de stockage vous sont facturés.

8. Sélectionnez **Suivant : Sécurité** pour continuer.

    L’écran **Sécurité** vous permet d’utiliser votre propre clé de chiffrement et votre propre appareil, de partager des mots de passe et de choisir d’utiliser le chiffrement double.

    Tous les paramètres dans l’écran **Sécurité** sont facultatifs. Si vous ne changez aucun paramètre, les paramètres par défaut s’appliquent.

    ![Écran Sécurité de l’Assistant Commande d’importation Data Box](media/data-box-deploy-ordered/select-data-box-import-security-01.png)

9. Si vous souhaitez utiliser votre propre clé gérée par le client afin de protéger la clé d’accès de déverrouillage pour votre nouvelle ressource, développez **Type de chiffrement**.

    La configuration d’une clé gérée par le client pour Azure Data Box est facultative. Par défaut, Data Box utilise une clé managée par Microsoft pour protéger la clé d’accès de déverrouillage.

    Une clé gérée par le client n’affecte pas la manière dont les données sont chiffrées sur l’appareil. La clé est utilisée uniquement pour chiffrer la clé d’accès de déverrouillage de l’appareil.

    Si vous ne souhaitez pas utiliser de clé gérée par le client, passez à l’étape 15.

   ![Écran Sécurité présentant les paramètres Type de chiffrement](./media/data-box-deploy-ordered/customer-managed-key-01.png)

10. Sélectionnez **Clé gérée par le client** comme type de clé. Choisissez ensuite **Sélectionner un coffre de clés et une clé**.
   
    ![Écran Sécurité, paramètres d’une clé gérée par le client](./media/data-box-deploy-ordered/customer-managed-key-02.png)

11. Dans le panneau **Sélectionner une clé dans Azure Key Vault**, l’abonnement est automatiquement renseigné.

    - Pour **Coffre de clés**, vous pouvez sélectionner un coffre de clés existant dans la liste déroulante.

      ![Écran Sélectionner une clé dans Azure Key Vault](./media/data-box-deploy-ordered/customer-managed-key-03.png)

    - Vous pouvez également sélectionner **Créer** pour créer un coffre de clés. Dans l’écran **Créer un coffre de clés**, entrez le groupe de ressources et un nom de coffre de clés. Assurez-vous que les options **Suppression réversible** et **Protection de purge** sont activées. Acceptez toutes les autres valeurs par défaut, puis sélectionnez **Vérifier + créer**.

      ![Paramètres Créer un coffre de clés](./media/data-box-deploy-ordered/customer-managed-key-04.png)

      Passez en revue les informations de votre coffre de clés, puis sélectionnez **Créer**. Patientez quelques minutes avant la fin de la création du coffre de clés.

      ![Écran d’examen du nouveau coffre Azure Key Vault](./media/data-box-deploy-ordered/customer-managed-key-05.png)

12. Dans **Sélectionner une clé dans Azure Key Vault**, vous pouvez sélectionner une clé existante dans le coffre de clés.

    ![Sélectionner une clé existante dans Azure Key Vault](./media/data-box-deploy-ordered/customer-managed-key-06.png)

    Si vous voulez créer un coffre de clés, sélectionnez **Créer**. Vous devez utiliser une clé RSA. La taille peut être supérieure ou égale à 2048. Entrez un nom pour votre nouvelle clé, acceptez les autres valeurs par défaut, puis sélectionnez **Créer**.

      ![Option permettant de créer une clé](./media/data-box-deploy-ordered/customer-managed-key-07.png)

      Vous serez averti quand la clé aura été créée dans votre coffre de clés.

13. Sélectionnez la **Version** de la clé à utiliser, puis choisissez **Sélectionner**.

      ![Nouvelle clé créée dans le coffre de clés](./media/data-box-deploy-ordered/customer-managed-key-08.png)

    Si vous voulez créer une version de clé, sélectionnez **Créer**.

    ![Ouvrir une boîte de dialogue permettant de créer une version de clé](./media/data-box-deploy-ordered/customer-managed-key-08-a.png)

    Choisissez les paramètres de la nouvelle version de clé, puis sélectionnez **Créer**.

    ![Créer une nouvelle version de la clé](./media/data-box-deploy-ordered/customer-managed-key-08-b.png)

    Les paramètres **Type de chiffrement** de l’écran **Sécurité** montrent votre coffre de clés et votre clé.

    ![Clé et coffre de clés pour une clé gérée par le client](./media/data-box-deploy-ordered/customer-managed-key-09.png)

14. Sélectionnez une identité d’utilisateur que vous utiliserez pour gérer l’accès à cette ressource. Choisissez **Sélectionner une identité d’utilisateur**. Dans le volet de droite, sélectionnez l’abonnement et l’identité managée à utiliser. Choisissez ensuite **Sélectionner**.

    Une identité managée affectée par l’utilisateur est une ressource Azure autonome qui peut être utilisée pour gérer plusieurs ressources. Pour plus d’informations, consultez [Types d’identités managées](/azure/active-directory/managed-identities-azure-resources/overview).  

    Si vous avez besoin de créer une identité managée, suivez les instructions indiquées dans [Créer, répertorier, supprimer ou affecter un rôle à une identité managée affectée par l’utilisateur à l’aide du portail Azure](/azure/active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal).
    
    ![Sélectionner une identité d’utilisateur](./media/data-box-deploy-ordered/customer-managed-key-10.png)

    L’identité d’utilisateur est indiquée dans les paramètres **Type de chiffrement**.

    ![Identité d’utilisateur sélectionnée dans les paramètres Type de chiffrement](./media/data-box-deploy-ordered/customer-managed-key-11.png)

15. Si vous ne souhaitez pas utiliser les mots de passe générés par le système qu’Azure Data Box utilise par défaut, développez **Apporter son propre mot de passe** dans l’écran **Sécurité**.

    Les mots de passe générés par le système sont sécurisés et sont recommandés, à moins que votre organisation en décide autrement.

    ![Développement des options Bring your own password pour une commande d’importation de Data Box](media/data-box-deploy-ordered/select-data-box-import-security-02.png) 

   - Afin d’utiliser votre propre mot de passe pour votre nouvel appareil, dans **Définir la préférence pour le mot de passe de l’appareil**, sélectionnez **Utiliser votre propre mot de passe**, puis tapez un mot de passe conforme aux exigences de sécurité.
   
     ![Options d’utilisation de votre propre mot de passe d’appareil sur l’écran Sécurité pour une commande d’importation Data Box](media/data-box-deploy-ordered/select-data-box-import-security-03.png)

 - Afin d’utiliser vos propres mots de passe pour les partages :

   - Dans **Définir la préférence pour les mots de passe de partage**, sélectionnez **Utiliser vos propres mots de passe**, puis **Sélectionnez des mots de passe pour les partages**.
     
        ![Options d’utilisation de vos mots de passe de partage sur l’écran Sécurité pour une commande d’importation Data Box](media/data-box-deploy-ordered/select-data-box-import-security-04.png)

    - Tapez un mot de passe pour chaque compte de stockage dans la commande. Le mot de passe sera utilisé sur tous les partages pour le compte de stockage.
     
        Afin d’utiliser le même mot de passe pour tous les comptes de stockage, sélectionnez **Copy to all** (Copier pour tous). Lorsque vous avez terminé, sélectionnez **Enregistrer**.
     
        ![Écran d’entrée de mots de passe de partage pour une commande d’importation de Data Box](media/data-box-deploy-ordered/select-data-box-import-security-05.png)

       Dans l’écran **Sécurité**, vous pouvez utiliser **Afficher ou modifier les mots de passe** pour changer les mots de passe.

16. Dans **Sécurité**, si vous voulez activer le double chiffrement basé sur logiciel, développez **Double-encryption (for highly secure environments)** Double chiffrement (pour les environnements hautement sécurisés) et sélectionnez **Enable double encryption for the order** (Activer le double chiffrement pour la commande).

    ![Écran Sécurité de l’importation de Data Box ; activation du chiffrement basé sur logiciel pour une commande de Data Box](media/data-box-deploy-ordered/select-data-box-import-security-07.png)

    Le chiffrement basé sur le logiciel est effectué en plus du chiffrement AES 256 bits des données sur Data Box.

    > [!NOTE]
    > L’activation de cette option peut augmenter le temps nécessaire au traitement des commandes et à la copie des données. Une fois votre commande créée, vous ne pouvez pas changer cette option.

    Sélectionnez **Suivant : Détails du contact** pour continuer.

17. Sous **Détails du contact**, sélectionnez **+ Ajouter une adresse de livraison**.

    ![Dans l’écran Détails du contact, ajoutez les adresses d’expédition à votre commande d’importation Azure Data Box.](media/data-box-deploy-ordered/select-data-box-import-08-a.png)

18. Dans la zone **Adresse d’expédition**, indiquez vos nom et prénom, le nom et l’adresse postale de la société, et un numéro de téléphone valide. Sélectionnez **Valider l’adresse**. Le service valide l’adresse d’expédition de disponibilité du service. Si le service est disponible pour l’adresse de livraison indiquée, vous recevez une notification à cet effet.

    ![Capture d’écran de la boîte de dialogue Ajouter une adresse de livraison avec les options Expédier avec et l’option Ajouter une adresse d’expédition en évidence.](media/data-box-deploy-ordered/select-data-box-import-10.png)

    Si vous avez sélectionné l’expédition autogérée, vous recevez une notification par e-mail une fois la commande passée. Pour plus d’informations sur l’expédition gérée par vous-même, consultez [Utiliser l’expédition autogérée](data-box-portal-customer-managed-shipping.md).

19. Sélectionnez **Ajouter une adresse de livraison** dès lors que les détails de l’expédition ont été correctement validés. Vous revenez à l’onglet **Détails du contact**.

20. Quand vous êtes de nouveau dans l’onglet **Détails du contact**, ajoutez une ou plusieurs adresses e-mail. Le service envoie des notifications par courrier électronique concernant les mises à jour de l’état de la commande aux adresses de messagerie spécifiées.

    Nous vous recommandons d’utiliser un e-mail de groupe afin de continuer à recevoir des notifications si un administrateur du groupe quitte l’entreprise.

    ![Section e-mail des détails du contact dans l’Assistant Commander](media/data-box-deploy-ordered/select-data-box-import-08-c.png)

21. Sous **Passer en revue + commander**, passez en revue les informations liées à la commande, au contact, à la notification et aux conditions de confidentialité. Cochez la case correspondant à l’acceptation des conditions de confidentialité.

22. Sélectionnez **Commander**. La création d’une commande peut prendre quelques minutes.

    ![Écran Vérifier et commander de l’Assistant Commander](media/data-box-deploy-ordered/select-data-box-import-11.png)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Effectuez les étapes suivantes l’aide d’Azure CLI pour commander un appareil :

1. Notez vos paramètres pour votre commande Data Box. Ces paramètres incluent vos informations personnelles/professionnelles, le nom de l’abonnement, les informations sur l’appareil et les informations d’expédition. Vous aurez besoin d’utiliser ces paramètres comme paramètres lors de l’exécution de la commande CLI pour créer la commande Data Box. Le tableau suivant indique la configuration des paramètres utilisés pour `az databox job create` :

   | Configuration (paramètre) | Description |  Exemple de valeur |
   |---|---|---|
   |resource-group| Créez-en un nouveau ou utilisez un groupe existant. Un groupe de ressources est un conteneur logique pour les ressources qui peuvent être gérées ou déployées ensemble. | « myresourcegroup »|
   |name| Nom de la commande que vous créez. | « mydataboxorder »|
   |contact-name| Nom associé à l’adresse de livraison. | « Gus Poland »|
   |phone| Numéro de téléphone de la personne ou de l’entreprise qui recevra la commande.| « 14255551234 »
   |location| Région Azure la plus proche de vous qui expédiera votre appareil.| « USA Ouest »|
   |sku| Appareil Data Box spécifique que vous commandez. Les valeurs autorisées sont : « DataBox », « DataBoxDisk » et « DataBoxHeavy »| « DataBox » |
   |email-list| Adresses e-mail associées à la commande.| "gusp@contoso.com" |
   |street-address1| Adresse postale de l’endroit où la commande sera expédiée. | « 15700 NE 39th St » |
   |street-address2| Informations sur l’adresse secondaire, comme le numéro de l’appartement ou le numéro du bâtiment. | « Bld 123 » |
   |city| Ville dans laquelle l’appareil sera expédié. | « Redmond » |
   |state-or-province| État dans lequel l’appareil sera expédié.| « WA » |
   |country| Pays dans lequel l’appareil sera expédié. | « États-Unis » |
   |postal-code| Code postal associé à l’adresse d’expédition.| « 98052 »|
   |company-name| Nom de l’entreprise pour laquelle vous travaillez.| « Contoso, LTD » |
   |compte de stockage| Compte de Stockage Azure à partir duquel vous voulez importer des données.| « mystorageaccount »|
   |debug| Inclure les informations de débogage à la journalisation détaillée  | --debug |
   |help| Afficher les informations d’aide pour cette commande. | --help -h |
   |only-show-errors| Afficher uniquement les erreurs, en supprimant les avertissements. | --only-show-errors |
   |output -o| Définit le format de sortie.  Valeurs autorisées : json, jsonc, none, table, tsv, yaml, yamlc. La valeur par défaut est json. | --output "json" |
   |query| Chaîne de requêtes JMESPath. Pour plus d’informations, consultez [JMESPath](http://jmespath.org/). | --query <string>|
   |verbose| Inclure la journalisation détaillée. | --verbose |

2. Dans l’invite de commandes de votre choix ou dans votre terminal, exécutez [az data box job create](/cli/azure/ext/databox/databox/job?view=azure-cli-latest#ext-databox-az-databox-job-create&preserve-view=true) pour créer votre commande Azure Data Box.

   ```azurecli
   az databox job create --resource-group <resource-group> --name <order-name> --location <azure-location> --sku <databox-device-type> --contact-name <contact-name> --phone <phone-number> --email-list <email-list> --street-address1 <street-address-1> --street-address2 <street-address-2> --city "contact-city" --state-or-province <state-province> --country <country> --postal-code <postal-code> --company-name <company-name> --storage-account "storage-account"
   ```

   Voici un exemple d’utilisation de la commande :

   ```azurecli
   az databox job create --resource-group "myresourcegroup" \
                         --name "mydataboxtest3" \
                         --location "westus" \
                         --sku "DataBox" \
                         --contact-name "Gus Poland" \
                         --phone "14255551234" \
                         --email-list "gusp@contoso.com" \
                         --street-address1 "15700 NE 39th St" \
                         --street-address2 "Bld 25" \
                         --city "Redmond" \
                         --state-or-province "WA" \
                         --country "US" \
                         --postal-code "98052" \
                         --company-name "Contoso" \
                         --storage-account mystorageaccount
   ```

   Voici la sortie de l’exécution de la commande :

   ```output
   Command group 'databox job' is experimental and not covered by customer support. Please use with discretion.
   {
     "cancellationReason": null,
     "deliveryInfo": {
        "scheduledDateTime": "0001-01-01T00:00:00+00:00"
   },
   "deliveryType": "NonScheduled",
   "details": null,
   "error": null,
   "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.DataBox/jobs/mydataboxtest3",
   "identity": {
     "type": "None"
   },
   "isCancellable": true,
   "isCancellableWithoutFee": true,
   "isDeletable": false,
   "isShippingAddressEditable": true,
   "location": "westus",
   "name": "mydataboxtest3",
   "resourceGroup": "myresourcegroup",
   "sku": {
     "displayName": null,
     "family": null,
     "name": "DataBox"
   },
   "startTime": "2020-06-10T23:28:27.354241+00:00",
   "status": "DeviceOrdered",
   "tags": {},
   "type": "Microsoft.DataBox/jobs"

   }
   PS C:\Windows>

   ```

3. Toutes les commandes Azure CLI utilisent JSON comme format de sortie par défaut, sauf si vous le changez. Vous pouvez changer le format de sortie à l’aide du paramètre global `--output <output-format>`. Le fait de remplacer le format par « table » améliore la lisibilité de la sortie.

   Voici la même commande que celle que nous venons d’exécuter avec un petit ajustement pour changer la mise en forme :

    ```azurecli
    az databox job create --resource-group "myresourcegroup" --name "mydataboxtest4" --location "westus" --sku "DataBox" --contact-name "Gus Poland" --phone "14255551234" --email-list "gusp@contoso.com" --street-address1 "15700 NE 39th St" --street-address2 "Bld 25" --city "Redmond" --state-or-province "WA" --country "US" --postal-code "98052" --company-name "Contoso" --storage-account mystorageaccount --output "table"
   ```

   Voici la sortie de l’exécution de la commande :

   ```output

    Command group 'databox job' is experimental and not covered by customer support. Please use with discretion.
    DeliveryType    IsCancellable    IsCancellableWithoutFee    IsDeletable    IsShippingAddressEditable    Location    Name            ResourceGroup    StartTime                         Status
    --------------  ---------------  -------------------------  -------------  ---------------------------  ----------  --------------  ---------------  --------------------------------  -------------
    NonScheduled    True             True                       False          True                         westus      mydataboxtest4  myresourcegroup  2020-06-18T03:48:00.905893+00:00  DeviceOrdered

    ```

# <a name="powershell"></a>[PowerShell](#tab/azure-ps)

Effectuez les étapes suivantes l’aide d’Azure PowerShell pour commander un appareil :

1. Avant de créer l’ordre d’importation, vous devez obtenir votre compte de stockage et enregistrer l’objet de compte de stockage dans une variable.

   ```azurepowershell
    $storAcct = Get-AzStorageAccount -Name "mystorageaccount" -ResourceGroup "myresourcegroup"
   ```

2. Notez vos paramètres pour votre commande Data Box. Ces paramètres incluent vos informations personnelles/professionnelles, le nom de l’abonnement, les informations sur l’appareil et les informations d’expédition. Vous devrez les utiliser comme paramètres quand vous exécuterez la commande PowerShell pour créer la commande Data Box. Le tableau suivant indique la configuration des paramètres utilisés pour [New-AzDataBoxJob](/powershell/module/az.databox/New-AzDataBoxJob).

    | Configuration (paramètre) | Description |  Exemple de valeur |
    |---|---|---|
    |ResourceGroupName [Obligatoire]| Utilisez un groupe de ressources existant. Un groupe de ressources est un conteneur logique pour les ressources qui peuvent être gérées ou déployées ensemble. | « myresourcegroup »|
    |Name [Obligatoire]| Nom de la commande que vous créez. | « mydataboxorder »|
    |ContactName [Obligatoire]| Nom associé à l’adresse de livraison. | « Gus Poland »|
    |PhoneNumber [Obligatoire]| Numéro de téléphone de la personne ou de l’entreprise qui recevra la commande.| « 14255551234 »
    |Location [Obligatoire]| Région Azure la plus proche de vous qui expédiera votre appareil.| "WestUS"|
    |DataBoxType [Obligatoire]| Appareil Data Box spécifique que vous commandez. Les valeurs autorisées sont : « DataBox », « DataBoxDisk » et « DataBoxHeavy »| « DataBox » |
    |EmailId [Obligatoire]| Adresses e-mail associées à la commande.| "gusp@contoso.com" |
    |StreetAddress1 [Obligatoire]| Adresse postale de l’endroit où la commande sera expédiée. | « 15700 NE 39th St » |
    |StreetAddress2| Informations sur l’adresse secondaire, comme le numéro de l’appartement ou le numéro du bâtiment. | « Bld 123 » |
    |StreetAddress3| Troisième information d’adresse. | |
    |City [Obligatoire]| Ville dans laquelle l’appareil sera expédié. | « Redmond » |
    |StateOrProvinceCode [Obligatoire]| État dans lequel l’appareil sera expédié.| « WA » |
    |CountryCode [Obligatoire]| Pays dans lequel l’appareil sera expédié. | « États-Unis » |
    |PostalCode [Obligatoire]| Code postal associé à l’adresse d’expédition.| « 98052 »|
    |CompanyName| Nom de l’entreprise pour laquelle vous travaillez.| « Contoso, LTD » |
    |StorageAccountResourceId [Obligatoire]| ID du compte de stockage Azure à partir duquel vous voulez importer des données.| <AzStorageAccount>.id |

3. Dans l’invite de commandes de votre choix ou dans votre terminal, utilisez la commande [New-AzDataBoxJob](/powershell/module/az.databox/New-AzDataBoxJob) pour créer votre commande Azure Data Box.

   ```azurepowershell
    PS> $storAcct = Get-AzureStorageAccount -StorageAccountName "mystorageaccount"
    PS> New-AzDataBoxJob -Location "WestUS" \
                         -StreetAddress1 "15700 NE 39th St" \
                         -PostalCode "98052" \
                         -City "Redmond" \
                         -StateOrProvinceCode "WA" \
                         -CountryCode "US" \
                         -EmailId "gusp@contoso.com" \
                         -PhoneNumber 4255551234 \
                         -ContactName "Gus Poland" \
                         -StorageAccount $storAcct.id \
                         -DataBoxType DataBox \
                         -ResourceGroupName "myresourcegroup" \
                         -Name "myDataBoxOrderPSTest"
   ```

   Voici la sortie de l’exécution de la commande :

   ```output
    jobResource.Name     jobResource.Sku.Name jobResource.Status jobResource.StartTime jobResource.Location ResourceGroup
    ----------------     -------------------- ------------------ --------------------- -------------------- -------------
    myDataBoxOrderPSTest DataBox              DeviceOrdered      07-06-2020 05:25:30   westus               myresourcegroup
   ```

---

## <a name="track-the-order"></a>Suivre la commande

# <a name="portal"></a>[Portail](#tab/portal)

Une fois la commande passée, vous pouvez suivre son état à partir du portail Azure. Accédez à votre commande Data Box, puis à **Vue d’ensemble** pour voir l’état. Le portail affiche la commande avec l’état **Commandé**.

Si l’appareil n’est pas disponible, vous recevez une notification. Si l’appareil est disponible, Microsoft identifie l’appareil à expédier et le prépare. Pendant la préparation de l’appareil, les actions suivantes se produisent :

* Des partages SMB sont créés pour chaque compte de stockage associé à l’appareil.
* Pour chaque partage, des informations d’identification d’accès (nom d’utilisateur et mot de passe) sont générées.
* Un mot de passe qui permet de déverrouiller l’appareil est également généré.
* La Data Box est verrouillée pour empêcher tout accès non autorisé à l’appareil à tout moment.

Une fois l’appareil préparé, le portail affiche la commande avec l’état **Traité**.

![Commande Data Box qui a été traitée](media/data-box-overview/data-box-order-status-processed.png)

Ensuite, Microsoft prépare et achemine l’appareil via un transporteur régional. Vous recevez un numéro de suivi une fois l’appareil expédié. Le portail affiche la commande dont l’état est **Distribué**.

![Commande Data Box qui a été expédié](media/data-box-overview/data-box-order-status-dispatched.png)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

### <a name="track-a-single-order"></a>Suivre une commande unique

Pour obtenir des informations de suivi sur une commande Azure Data Box unique existante, exécutez [az databox job show](/cli/azure/ext/databox/databox/job?view=azure-cli-latest#ext-databox-az-databox-job-show&preserve-view=true). La commande affiche des informations sur la commande, notamment (liste non exhaustive) : nom, groupe de ressources, informations de suivi, ID d’abonnement, informations de contact, type d’expédition et référence SKU de l’appareil.

   ```azurecli
   az databox job show --resource-group <resource-group> --name <order-name>
   ```

   Le tableau suivant indique les informations des paramètres pour `az databox job show` :

   | Paramètre | Description |  Exemple de valeur |
   |---|---|---|
   |resource-group [Obligatoire]| Nom du groupe de ressources associé à la commande. Un groupe de ressources est un conteneur logique pour les ressources qui peuvent être gérées ou déployées ensemble. | « myresourcegroup »|
   |name [Obligatoire]| Nom de la commande à afficher. | « mydataboxorder »|
   |debug| Inclure les informations de débogage à la journalisation détaillée | --debug |
   |help| Afficher les informations d’aide pour cette commande. | --help -h |
   |only-show-errors| Afficher uniquement les erreurs, en supprimant les avertissements. | --only-show-errors |
   |output -o| Définit le format de sortie.  Valeurs autorisées : json, jsonc, none, table, tsv, yaml, yamlc. La valeur par défaut est json. | --output "json" |
   |query| Chaîne de requêtes JMESPath. Pour plus d’informations, consultez [JMESPath](http://jmespath.org/). | --query <string>|
   |verbose| Inclure la journalisation détaillée. | --verbose |

   Voici un exemple de la commande avec le format de sortie défini sur « table » :

   ```azurecli
    PS C:\WINDOWS\system32> az databox job show --resource-group "myresourcegroup" \
                                                --name "mydataboxtest4" \
                                                --output "table"
   ```

   Voici la sortie de l’exécution de la commande :

   ```output
    Command group 'databox job' is experimental and not covered by customer support. Please use with discretion.
    DeliveryType    IsCancellable    IsCancellableWithoutFee    IsDeletable    IsShippingAddressEditable    Location    Name            ResourceGroup    StartTime                         Status
    --------------  ---------------  -------------------------  -------------  ---------------------------  ----------  --------------  ---------------  --------------------------------  -------------
    NonScheduled    True             True                       False          True                         westus      mydataboxtest4  myresourcegroup  2020-06-18T03:48:00.905893+00:00  DeviceOrdered
   ```

> [!NOTE]
> L’ordre de liste peut être pris en charge au niveau de l’abonnement et fait du groupe de ressources un paramètre facultatif (au lieu d’un paramètre obligatoire).

### <a name="list-all-orders"></a>Lister toutes les commandes

Si vous avez commandé plusieurs appareils, vous pouvez exécuter [az databox job list](/cli/azure/ext/databox/databox/job?view=azure-cli-latest#ext-databox-az-databox-job-list&preserve-view=true) pour voir toutes vos commandes Azure Data Box. La commande liste toutes les commandes qui appartiennent à un groupe de ressources spécifique. Également affichés dans la sortie : nom de la commande, état de l’expédition, région Azure, type de livraison, état de la commande. Les commandes annulées sont également incluses dans la liste.
La commande affiche également les horodatages de chaque commande.

```azurecli
az databox job list --resource-group <resource-group>
```

Le tableau suivant indique les informations des paramètres pour `az databox job list` :

   | Paramètre | Description |  Exemple de valeur |
   |---|---|---|
   |resource-group [Obligatoire]| Nom du groupe de ressources qui contient les commandes. Un groupe de ressources est un conteneur logique pour les ressources qui peuvent être gérées ou déployées ensemble. | « myresourcegroup »|
   |debug| Inclure les informations de débogage à la journalisation détaillée | --debug |
   |help| Afficher les informations d’aide pour cette commande. | --help -h |
   |only-show-errors| Afficher uniquement les erreurs, en supprimant les avertissements. | --only-show-errors |
   |output -o| Définit le format de sortie.  Valeurs autorisées : json, jsonc, none, table, tsv, yaml, yamlc. La valeur par défaut est json. | --output "json" |
   |query| Chaîne de requêtes JMESPath. Pour plus d’informations, consultez [JMESPath](http://jmespath.org/). | --query <string>|
   |verbose| Inclure la journalisation détaillée. | --verbose |

   Voici un exemple de la commande avec le format de sortie défini sur « table » :

   ```azurecli
    PS C:\WINDOWS\system32> az databox job list --resource-group "GDPTest" --output "table"
   ```

   Voici la sortie de l’exécution de la commande :

   ```output
   Command group 'databox job' is experimental and not covered by customer support. Please use with discretion.
   CancellationReason                                               DeliveryType    IsCancellable    IsCancellableWithoutFee    IsDeletable    IsShippingAddressEditable    Location    Name                 ResourceGroup    StartTime                         Status
   ---------------------- ----------------------------------------  --------------  ---------------  -------------------------  -------------  ---------------------------  ----------  -------------------  ---------------  --------------------------------  -------------
   OtherReason This was a test order for documentation purposes.    NonScheduled    False            False                      True           False                        westus      gdpImportTest        MyResGrp         2020-05-26T23:20:57.464075+00:00  Cancelled
   NoLongerNeeded This order was created for documentation purposes.NonScheduled    False            False                      True           False                        westus      mydataboxExportTest  MyResGrp         2020-05-27T00:04:16.640397+00:00  Cancelled
   IncorrectOrder                                                   NonScheduled    False            False                      True           False                        westus      mydataboxtest2       MyResGrp         2020-06-10T16:54:23.509181+00:00  Cancelled
                                                                    NonScheduled    True             True                       False          True                         westus      mydataboxtest3       MyResGrp         2020-06-11T22:05:49.436622+00:00  DeviceOrdered
                                                                    NonScheduled    True             True                       False          True                         westus      mydataboxtest4       MyResGrp         2020-06-18T03:48:00.905893+00:00  DeviceOrdered
   PS C:\WINDOWS\system32>
   ```

# <a name="powershell"></a>[PowerShell](#tab/azure-ps)

### <a name="track-a-single-order"></a>Suivre une commande unique

Pour obtenir des informations de suivi sur une commande Azure Data Box unique existante, exécutez [Get-AzDataBoxJob](/powershell/module/az.databox/Get-AzDataBoxJob). La commande affiche des informations sur la commande, notamment (liste non exhaustive) : nom, groupe de ressources, informations de suivi, ID d’abonnement, informations de contact, type d’expédition et référence SKU de l’appareil.

> [!NOTE]
> `Get-AzDataBoxJob` permet d’afficher les commandes uniques et multiples. Pour les commandes uniques, cependant, vous devez spécifier le nom de la commande.

   ```azurepowershell
    Get-AzDataBoxJob -ResourceGroupName <String> -Name <String>
   ```

   Le tableau suivant indique les informations des paramètres pour `Get-AzDataBoxJob` :

   | Paramètre | Description |  Exemple de valeur |
   |---|---|---|
   |ResourceGroup [Obligatoire]| Nom du groupe de ressources associé à la commande. Un groupe de ressources est un conteneur logique pour les ressources qui peuvent être gérées ou déployées ensemble. | « myresourcegroup »|
   |Name [Obligatoire]| Nom de la commande sur laquelle obtenir des informations. | « mydataboxorder »|
   |ResourceId| ID de la ressource associée à la commande. |  |

   Voici un exemple de la commande avec la sortie :

   ```azurepowershell
    PS C:\WINDOWS\system32> Get-AzDataBoxJob -ResourceGroupName "myResourceGroup" -Name "myDataBoxOrderPSTest"
   ```

   Voici la sortie de l’exécution de la commande :

   ```output
   jobResource.Name     jobResource.Sku.Name jobResource.Status jobResource.StartTime jobResource.Location ResourceGroup
   ----------------     -------------------- ------------------ --------------------- -------------------- -------------
   myDataBoxOrderPSTest DataBox              DeviceOrdered      7/7/2020 12:37:16 AM  WestUS               myResourceGroup
   ```

### <a name="list-all-orders"></a>Lister toutes les commandes

Si vous avez commandé plusieurs appareils, vous pouvez exécuter [Get-AzDataBoxJob](/powershell/module/az.databox/Get-AzDataBoxJob) pour voir toutes vos commandes Azure Data Box. La commande liste toutes les commandes qui appartiennent à un groupe de ressources spécifique. Également affichés dans la sortie : nom de la commande, état de l’expédition, région Azure, type de livraison, état de la commande. Les commandes annulées sont également incluses dans la liste.
La commande affiche également les horodatages de chaque commande.

```azurepowershell
Get-AzDataBoxJob -ResourceGroupName <String>
```

Voici un exemple de la commande :

```azurepowershell
PS C:\WINDOWS\system32> Get-AzDataBoxJob -ResourceGroupName "myResourceGroup"
```

Voici la sortie de l’exécution de la commande :

```output
jobResource.Name     jobResource.Sku.Name jobResource.Status jobResource.StartTime jobResource.Location ResourceGroup
----------------     -------------------- ------------------ --------------------- -------------------- -------------
guspImportTest       DataBox              Cancelled          5/26/2020 11:20:57 PM WestUS               myResourceGroup
mydataboxExportTest  DataBox              Cancelled          5/27/2020 12:04:16 AM WestUS               myResourceGroup
mydataboximport1     DataBox              Cancelled          6/26/2020 11:00:34 PM WestUS               myResourceGroup
myDataBoxOrderPSTest DataBox              Cancelled          7/07/2020 12:37:16 AM WestUS               myResourceGroup
mydataboxtest2       DataBox              Cancelled          6/10/2020 4:54:23  PM WestUS               myResourceGroup
mydataboxtest4       DataBox              DeviceOrdered      6/18/2020 3:48:00  AM WestUS               myResourceGroup
PS C:\WINDOWS\system32>
```

---

## <a name="cancel-the-order"></a>Annuler la commande

# <a name="portal"></a>[Portail](#tab/portal)

Pour annuler cette commande, dans le portail Azure, accédez à **Vue d’ensemble**, puis sélectionnez **Annuler** dans la barre de commandes.

Après avoir passé commande, vous pouvez l’annuler à tout moment tant qu’elle n’a pas été traitée.

Pour supprimer une commande annulée, accédez à **Vue d’ensemble**, puis sélectionnez **Supprimer** dans la barre de commandes.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

### <a name="cancel-an-order"></a>Annuler une commande

Pour annuler une commande Azure Data Box, exécutez [az databox job cancel](/cli/azure/ext/databox/databox/job?view=azure-cli-latest#ext-databox-az-databox-job-cancel&preserve-view=true). Vous devez spécifier la raison de l’annulation de la commande.

   ```azurecli
   az databox job cancel --resource-group <resource-group> --name <order-name> --reason <cancel-description>
   ```

   Le tableau suivant indique les informations des paramètres pour `az databox job cancel` :

   | Paramètre | Description |  Exemple de valeur |
   |---|---|---|
   |resource-group [Obligatoire]| Nom du groupe de ressources associé à la commande à supprimer. Un groupe de ressources est un conteneur logique pour les ressources qui peuvent être gérées ou déployées ensemble. | « myresourcegroup »|
   |name [Obligatoire]| Nom de la commande à supprimer. | « mydataboxorder »|
   |reason [Obligatoire]| Motif de l’annulation de la commande. | « J’ai entré des informations erronées et j’ai dû annuler la commande. » |
   |Oui| Ne pas demander de confirmation. | --yes (-y)| --yes -y |
   |debug| Inclure les informations de débogage à la journalisation détaillée | --debug |
   |help| Afficher les informations d’aide pour cette commande. | --help -h |
   |only-show-errors| Afficher uniquement les erreurs, en supprimant les avertissements. | --only-show-errors |
   |output -o| Définit le format de sortie.  Valeurs autorisées : json, jsonc, none, table, tsv, yaml, yamlc. La valeur par défaut est json. | --output "json" |
   |query| Chaîne de requêtes JMESPath. Pour plus d’informations, consultez [JMESPath](http://jmespath.org/). | --query <string>|
   |verbose| Inclure la journalisation détaillée. | --verbose |

   Voici un exemple de la commande avec la sortie :

   ```azurecli
   PS C:\Windows> az databox job cancel --resource-group "myresourcegroup" --name "mydataboxtest3" --reason "Our budget was slashed due to **redacted** and we can no longer afford this device."
   ```

   Voici la sortie de l’exécution de la commande :

   ```output
   Command group 'databox job' is experimental and not covered by customer support. Please use with discretion.
   Are you sure you want to perform this operation? (y/n): y
   PS C:\Windows>
   ```

### <a name="delete-an-order"></a>Supprimer une commande

Si vous avez annulé une commande Azure Data Box, vous pouvez exécuter [az databox job delete](/cli/azure/ext/databox/databox/job?view=azure-cli-latest#ext-databox-az-databox-job-delete&preserve-view=true) pour la supprimer.

   ```azurecli
   az databox job delete --name [-n] <order-name> --resource-group <resource-group> [--yes] [--verbose]
   ```

   Le tableau suivant indique les informations des paramètres pour `az databox job delete` :

   | Paramètre | Description |  Exemple de valeur |
   |---|---|---|
   |resource-group [Obligatoire]| Nom du groupe de ressources associé à la commande à supprimer. Un groupe de ressources est un conteneur logique pour les ressources qui peuvent être gérées ou déployées ensemble. | « myresourcegroup »|
   |name [Obligatoire]| Nom de la commande à supprimer. | « mydataboxorder »|
   |subscription| Nom ou ID (GUID) de votre abonnement Azure. | « xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx » |
   |Oui| Ne pas demander de confirmation. | --yes (-y)| --yes -y |
   |debug| Inclure les informations de débogage à la journalisation détaillée | --debug |
   |help| Afficher les informations d’aide pour cette commande. | --help -h |
   |only-show-errors| Afficher uniquement les erreurs, en supprimant les avertissements. | --only-show-errors |
   |output -o| Définit le format de sortie.  Valeurs autorisées : json, jsonc, none, table, tsv, yaml, yamlc. La valeur par défaut est json. | --output "json" |
   |query| Chaîne de requêtes JMESPath. Pour plus d’informations, consultez [JMESPath](http://jmespath.org/). | --query <string>|
   |verbose| Inclure la journalisation détaillée. | --verbose |

Voici un exemple de la commande avec la sortie :

   ```azurecli
   PS C:\Windows> az databox job delete --resource-group "myresourcegroup" --name "mydataboxtest3" --yes --verbose
   ```

   Voici la sortie de l’exécution de la commande :

   ```output
   Command group 'databox job' is experimental and not covered by customer support. Please use with discretion.
   command ran in 1.142 seconds.
   PS C:\Windows>
   ```

# <a name="powershell"></a>[PowerShell](#tab/azure-ps)

### <a name="cancel-an-order"></a>Annuler une commande

Pour annuler une commande Azure Data Box, exécutez [Stop-AzDataBoxJob](/powershell/module/az.databox/stop-azdataboxjob). Vous devez spécifier la raison de l’annulation de la commande.

```azurepowershell
Stop-AzDataBoxJob -ResourceGroup <String> -Name <String> -Reason <String>
```

Le tableau suivant indique les informations des paramètres pour `Stop-AzDataBoxJob` :

| Paramètre | Description |  Exemple de valeur |
|---|---|---|
|ResourceGroup [Obligatoire]| Nom du groupe de ressources associé à la commande à annuler. Un groupe de ressources est un conteneur logique pour les ressources qui peuvent être gérées ou déployées ensemble. | « myresourcegroup »|
|Name [Obligatoire]| Nom de la commande à supprimer. | « mydataboxorder »|
|Reason [Obligatoire]| Motif de l’annulation de la commande. | « J’ai entré des informations erronées et j’ai dû annuler la commande. » |
|Force | Force l’exécution de la cmdlet sans confirmation de l’utilisateur. | -Force |

Voici un exemple de la commande avec la sortie :

```azurepowershell
PS C:\PowerShell\Modules> Stop-AzDataBoxJob -ResourceGroupName myResourceGroup \
                                            -Name "myDataBoxOrderPSTest" \
                                            -Reason "I entered erroneous information and had to cancel."
```

Voici la sortie de l’exécution de la commande :

```output
Confirm
"Cancelling Databox Job "myDataBoxOrderPSTest
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): y
PS C:\WINDOWS\system32>
```

### <a name="delete-an-order"></a>Supprimer une commande

Si vous avez annulé une commande Azure Data Box, vous pouvez exécuter [Remove-AzDataBoxJob](/powershell/module/az.databox/remove-azdataboxjob) pour la supprimer.

```azurepowershell
Remove-AzDataBoxJob -Name <String> -ResourceGroup <String>
```

Le tableau suivant indique les informations des paramètres pour `Remove-AzDataBoxJob` :

| Paramètre | Description |  Exemple de valeur |
|---|---|---|
|ResourceGroup [Obligatoire]| Nom du groupe de ressources associé à la commande à supprimer. Un groupe de ressources est un conteneur logique pour les ressources qui peuvent être gérées ou déployées ensemble. | « myresourcegroup »|
|Name [Obligatoire]| Nom de la commande à supprimer. | « mydataboxorder »|
|Force | Force l’exécution de la cmdlet sans confirmation de l’utilisateur. | -Force |

Voici un exemple de la commande avec la sortie :

```azurepowershell
PS C:\Windows> Remove-AzDataBoxJob -ResourceGroup "myresourcegroup" \
                                   -Name "mydataboxtest3"
```

Voici la sortie de l’exécution de la commande :

```output
Confirm
"Removing Databox Job "mydataboxtest3
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): y
PS C:\Windows>
```

---

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez découvert des articles Azure Data Box comme les suivants :

> [!div class="checklist"]
>
> * Conditions préalables au déploiement de la Data Box
> * Commander une Data Box
> * Suivre la commande
> * Annuler la commande

Passez au tutoriel suivant pour apprendre à configurer votre Data Box.

> [!div class="nextstepaction"]
> [Configurer votre Azure Data Box](./data-box-deploy-set-up.md)
