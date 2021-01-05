---
title: Héberger un site web statique dans Stockage Azure
description: Découvrez comment servir du contenu statique (fichiers HTML, CSS, JavaScript et images) directement à partir d’un conteneur dans un compte GPv2 Stockage Azure.
author: normesta
ms.service: storage
ms.subservice: blobs
ms.topic: conceptual
ms.author: normesta
ms.date: 03/04/2020
ms.custom: devx-track-js, devx-track-azurecli
ms.openlocfilehash: 8ca670049b49500e6b6310bca25cb78ded31a294
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95537847"
---
# <a name="host-a-static-website-in-azure-storage"></a>Héberger un site web statique dans Stockage Azure

Vous pouvez servir du contenu statique (fichiers HTML, CSS, JavaScript et images) directement à partir d’un conteneur dans un compte GPv2 Stockage Azure. Pour en savoir plus, voir [Hébergement de site web statique dans Stockage Azure](storage-blob-static-website.md).

Cet article explique comment activer l’hébergement de site web statique avec le Portail Azure, Azure CLI ou PowerShell.

> [!NOTE]
> Veillez à créer un compte de stockage Standard universel v2. Les sites web statiques ne sont pas disponibles dans les autres types de comptes de stockage.

## <a name="enable-static-website-hosting"></a>Activer l’hébergement des sites web statiques

L’hébergement de site web statique est une fonctionnalité que vous devez activer sur le compte de stockage.

### <a name="portal"></a>[Portail](#tab/azure-portal)

1. Pour commencer, connectez-vous au [portail Azure](https://portal.azure.com/).

2. Recherchez votre compte de stockage et affichez la vue d’ensemble du compte.

3. Sélectionnez **Site web statique** pour afficher la page de configuration des sites web statiques.

4. Sélectionnez **Activé** pour activer l’hébergement de site web statique pour le compte de stockage.

5. Dans le champ **Nom du document d’index**, spécifiez une page d’index par défaut (par exemple, *index.html*). 

   La page d’index par défaut s’affiche quand un utilisateur accède à la racine de votre site web statique.  

6. Dans le champ **Chemin du document d’erreur**, spécifiez une page d’erreur par défaut (par exemple, *404.html*). 

   La page d’erreur par défaut s’affiche quand un utilisateur tente d’accéder à une page qui n’existe pas dans votre site web statique.

7. Cliquez sur **Enregistrer**. Le portail Azure affiche maintenant le point de terminaison de votre site web statique. 

    ![Activer l’hébergement de site web statique pour un compte de stockage](media/storage-blob-static-website-host/enable-static-website-hosting.png)

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

<a id="cli"></a>

Vous pouvez activer l’hébergement de site web statique à l’aide de l’[interface de ligne de commande Azure](/cli/azure/?view=azure-cli-latest) (Azure CLI).

1. Commencez par ouvrir [Azure Cloud Shell](../../cloud-shell/overview.md?view=azure-cli-latest) ou, si vous avez [installé](/cli/azure/install-azure-cli?view=azure-cli-latest) Azure CLI localement, ouvrez une application console de commandes telle que Windows PowerShell.

2. Si votre identité est associée à plusieurs abonnements, définissez comme abonnement actif l’abonnement du compte de stockage qui doit héberger votre site web statique.

   ```azurecli-interactive
   az account set --subscription <subscription-id>
   ```

   Remplacez la valeur d’espace réservé `<subscription-id>` par l’ID de votre abonnement.

3. Activez l’hébergement de site web statique.

   ```azurecli-interactive
   az storage blob service-properties update --account-name <storage-account-name> --static-website --404-document <error-document-name> --index-document <index-document-name>
   ```

   * Remplacez la valeur d’espace réservé `<storage-account-name>` par le nom de votre compte de stockage.

   * Remplacez l’espace réservé `<error-document-name>` par le nom du document d’erreur qui s’affiche quand un navigateur demande sur votre site une page inexistante.

   * Remplacez la valeur d’espace réservé `<index-document-name>` par le nom du document d’index. Ce document est généralement « index.html ».

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

<a id="powershell"></a>

Vous pouvez activer l’hébergement de site web statique à l’aide du module Azure PowerShell.

1. Ouvrez une fenêtre Commande Windows PowerShell.

2. Vérifiez que vous disposez du module Azure PowerShell Az version 0.7 ou ultérieure.

   ```powershell
   Get-InstalledModule -Name Az -AllVersions | select Name,Version
   ```

   Si vous devez installer ou mettre à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-Az-ps).

3. Connectez-vous à votre abonnement Azure avec la commande `Connect-AzAccount` et suivez les instructions à l’écran.

   ```powershell
   Connect-AzAccount
   ```

4. Si votre identité est associée à plusieurs abonnements, définissez comme abonnement actif l’abonnement du compte de stockage qui doit héberger votre site web statique.

   ```powershell
   $context = Get-AzSubscription -SubscriptionId <subscription-id>
   Set-AzContext $context
   ```

   Remplacez la valeur d’espace réservé `<subscription-id>` par l’ID de votre abonnement.

5. Obtenez le contexte du compte de stockage qui définit le compte de stockage à utiliser.

   ```powershell
   $storageAccount = Get-AzStorageAccount -ResourceGroupName "<resource-group-name>" -AccountName "<storage-account-name>"
   $ctx = $storageAccount.Context
   ```

   * Remplacez la valeur d’espace réservé `<resource-group-name>` par le nom de votre groupe de ressources.

   * Remplacez la valeur d’espace réservé `<storage-account-name>` par le nom de votre compte de stockage.

6. Activez l’hébergement de site web statique.

   ```powershell
   Enable-AzStorageStaticWebsite -Context $ctx -IndexDocument <index-document-name> -ErrorDocument404Path <error-document-name>
   ```

   * Remplacez l’espace réservé `<error-document-name>` par le nom du document d’erreur qui s’affiche quand un navigateur demande sur votre site une page inexistante.

   * Remplacez la valeur d’espace réservé `<index-document-name>` par le nom du document d’index. Ce document est généralement « index.html ».

---

## <a name="upload-files"></a>Charger des fichiers 

### <a name="portal"></a>[Portail](#tab/azure-portal)

Ces instructions vous indiquent comment charger des fichiers à l’aide de la version de l’Explorateur Stockage qui figure dans le portail Azure. Toutefois, vous pouvez également utiliser la version de l’[Explorateur Stockage](https://azure.microsoft.com/features/storage-explorer/) qui est exécutée en dehors du portail Azure. Vous pouvez utiliser [AzCopy](../common/storage-use-azcopy-v10.md), PowerShell, CLI ou n’importe quelle application personnalisée capable de charger des fichiers dans le conteneur **$web** de votre compte. Pour obtenir la procédure pas à pas qui permet de charger des fichiers à l’aide de Visual Studio Code, consultez [Tutoriel : Héberger un site web statique sur le Stockage Blob](./storage-blob-static-website-host.md).

1. Sélectionnez **Explorateur Stockage (préversion)** .

2. Développez le nœud **Conteneurs d’objets blob**, puis sélectionnez le conteneur **$web**.

3. Choisissez le bouton **Charger** pour charger des fichiers.

   ![Charger des fichiers](media/storage-blob-static-website/storage-blob-static-website-upload.png)

4. Si vous souhaitez que le navigateur affiche le contenu du fichier, vérifiez que le type de contenu de ce fichier est défini sur `text/html`. 

   ![Vérifier le type du contenu](media/storage-blob-static-website/storage-blob-static-website-content-type.png)

   >[!NOTE]
   > L’Explorateur Stockage définit automatiquement cette propriété sur `text/html` pour les extensions courantes comme `.html`. Toutefois, dans certains cas, vous devrez la définir vous-même. Si vous ne définissez pas cette propriété sur `text/html`, le navigateur invitera les utilisateurs à télécharger le fichier au lieu d’afficher son contenu. Pour définir cette propriété, cliquez avec le bouton droit sur le fichier, puis cliquez sur **Propriétés**.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Chargez des objets dans le conteneur *$web* à partir d’un répertoire source.

Cet exemple part du principe que vous exécutez des commandes à partir d’une session Azure Cloud Shell.

```azurecli-interactive
az storage blob upload-batch -s <source-path> -d '$web' --account-name <storage-account-name>
```

> [!NOTE] 
> Si le navigateur invite les utilisateurs à télécharger le fichier au lieu de restituer le contenu, vous pouvez ajouter `--content-type 'text/html; charset=utf-8'` à la commande. 

* Remplacez la valeur d’espace réservé `<storage-account-name>` par le nom de votre compte de stockage.

* Remplacez la valeur d’espace réservé `<source-path>` par le chemin d’accès de l’emplacement où se trouvent les fichiers que vous voulez charger.

> [!NOTE]
> Si vous utilisez l’emplacement d’installation d’Azure CLI, vous pouvez utiliser le chemin d’accès de tout emplacement sur votre ordinateur local. Par exemple : `C:\myFolder`.
>
> Si vous utilisez Azure Cloud Shell, vous devez faire référence à un partage de fichiers visible pour Cloud Shell. Cet emplacement peut être le partage de fichiers du partage cloud ou un partage de fichiers existant que vous montez à partir du Cloud Shell. Pour savoir comment procéder, voir [Conserver des fichiers dans Azure Cloud Shell](../../cloud-shell/persisting-shell-storage.md).

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Chargez des objets dans le conteneur *$web* à partir d’un répertoire source.

```powershell
# upload a file
set-AzStorageblobcontent -File "<path-to-file>" `
-Container `$web `
-Blob "<blob-name>" `
-Context $ctx
```

> [!NOTE] 
> Si le navigateur invite les utilisateurs à télécharger le fichier au lieu de restituer le contenu, vous pouvez ajouter `-Properties @{ ContentType = "text/html; charset=utf-8";}` à la commande.

* Remplacez la valeur d’espace réservé `<path-to-file>` par le chemin d’accès complet du fichier que vous souhaitez charger (par exemple, `C:\temp\index.html`).

* Remplacez la valeur d’espace réservé `<blob-name>` par le nom que vous souhaitez attribuer à l’objet blob obtenu (par exemple, `index.html`).

---

<a id="portal-find-url"></a>

## <a name="find-the-website-url"></a>Trouver l’URL du site web

Vous pouvez afficher les pages de votre site dans un navigateur en utilisant l’URL publique du site web.

### <a name="portal"></a>[Portail](#tab/azure-portal)

Dans le volet qui s’affiche en regard de la page de Vue d’ensemble du compte de votre compte de stockage, sélectionnez **Site web statique**. L’URL de votre site s’affiche dans le champ **Point de terminaison principal**.

![Métrique des métriques de sites web statiques dans Stockage Azure](./media/storage-blob-static-website/storage-blob-static-website-url.png)

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Recherchez l’URL publique de votre site web statique à l’aide de la commande suivante :

```azurecli-interactive
az storage account show -n <storage-account-name> -g <resource-group-name> --query "primaryEndpoints.web" --output tsv
```

* Remplacez la valeur d’espace réservé `<storage-account-name>` par le nom de votre compte de stockage.

* Remplacez la valeur d’espace réservé `<resource-group-name>` par le nom de votre groupe de ressources.

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Recherchez l’URL publique de votre site web statique à l’aide de la commande suivante :

```powershell
 $storageAccount = Get-AzStorageAccount -ResourceGroupName "<resource-group-name>" -Name "<storage-account-name>"
Write-Output $storageAccount.PrimaryEndpoints.Web
```

* Remplacez la valeur d’espace réservé `<resource-group-name>` par le nom de votre groupe de ressources.

* Remplacez la valeur d’espace réservé `<storage-account-name>` par le nom de votre compte de stockage.

---

<a id="metrics"></a>

## <a name="enable-metrics-on-static-website-pages"></a>Activer les métriques sur des pages de site web statiques

Une fois les métriques activées, les statistiques de trafic relatives aux fichiers du conteneur **$web** sont signalées dans le tableau de bord des métriques.

1. Cliquez sur **Métriques** sous la section **Moniteur** du menu compte de stockage.

   > [!div class="mx-imgBorder"]
   > ![Lien de métriques](./media/storage-blob-static-website/metrics-link.png)

   > [!NOTE]
   > Les données de métriques sont générées par raccordement aux différentes API de métriques. Le portail affiche uniquement les membres d’API utilisés dans un délai d’exécution donné afin de se focaliser exclusivement sur les membres qui renvoient des données. Pour garantir votre capacité à sélectionner le membre d’API nécessaire, la première étape consiste à étendre le délai d’exécution.

2. Cliquez sur le bouton de délai d’exécution, choisissez un délai d'exécution, puis cliquez sur **Appliquer**.

   ![Intervalle de temps des métriques de sites web statiques dans Stockage Azure](./media/storage-blob-static-website/storage-blob-static-website-metrics-time-range.png)

3. Sélectionnez **Blob** dans la liste déroulante *Espace de noms*.

   ![Espace de noms des métriques de sites web statiques dans Stockage Azure](./media/storage-blob-static-website/storage-blob-static-website-metrics-namespace.png)

4. Ensuite, sélectionnez la métrique **Sortie**.

   ![Capture d'écran illustrant la métrique Sortie des sites web statiques Stockage Azure.](./media/storage-blob-static-website/storage-blob-static-website-metrics-metric.png)

5. Dans le sélecteur **Agrégation**, sélectionnez *Somme*.

   ![Agrégation des métriques de sites web statiques dans Stockage Azure](./media/storage-blob-static-website/storage-blob-static-website-metrics-aggregation.png)

6. Cliquez sur le bouton **Ajouter un filtre**, puis choisissez **Nom API** dans le sélecteur *Propriété*.

   ![Nom d’API des métriques de sites web statiques dans Stockage Azure](./media/storage-blob-static-website/storage-blob-static-website-metrics-api-name.png)

7. Cochez la case en regard de **GetWebContent** dans le sélecteur *Valeurs* pour remplir le rapport des métriques.

   ![Valeur GetWebContent des métriques de sites web statiques dans Stockage Azure](./media/storage-blob-static-website/storage-blob-static-website-metrics-getwebcontent.png)

   >[!NOTE]
   > La case à cocher **GetWebContent** s’affiche uniquement si ce membre d’API a été utilisé dans un délai d'exécution donné. Le portail affiche uniquement les membres d’API utilisés dans un délai d’exécution donné afin de se focaliser exclusivement sur les membres qui renvoient des données. Si vous ne trouvez pas un membre d’API spécifique dans cette liste, développez le délai d’exécution.

## <a name="next-steps"></a>Étapes suivantes

* Apprenez à configurer un domaine personnalisé avec votre site web statique. Consultez [Mapper un domaine personnalisé à un point de terminaison de Stockage Blob Azure](storage-custom-domain-name.md).