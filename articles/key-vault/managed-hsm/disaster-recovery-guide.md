---
title: Que faire si une interruption de service Azure affecte le HSM managé - Azure Key Vault | Microsoft Docs
description: Découvrez comment procéder si une interruption de service Azure affecte le HSM managé.
services: key-vault
author: amitbapat
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 09/15/2020
ms.author: ambapat
ms.openlocfilehash: 69a0272061d8518119114e8fe7b023c889639844
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96171556"
---
# <a name="managed-hsm-disaster-recovery"></a>Récupération d’urgence du HSM managé

Vous souhaiterez peut-être créer un réplica exact de votre HSM si l’original est perdu ou indisponible pour l’une des raisons suivantes :

- Il a été supprimé, puis vidé.
- Une défaillance irrémédiable a eu lieu dans la région, entraînant la destruction de toutes les partitions membres.

Vous pouvez recréer l’instance HSM dans la même région ou dans une autre région si vous disposez des éléments suivants :
- Le [domaine de sécurité](security-domain.md) du HSM source.
- Les clés privées (au moins le numéro de quorum) qui chiffrent le domaine de sécurité.
- La [sauvegarde](backup-restore.md) complète la plus récente du HSM réalisée à partir du HSM source.

Voici les étapes de la récupération d’urgence :

1. Créer une instance HSM.
1. Activez la « Récupération du domaine de sécurité ». Une nouvelle paire de clés RSA (clé d’échange de domaine de sécurité) sera générée pour le transfert du domaine de sécurité puis envoyée en réponse, laquelle sera téléchargée sous forme de clé publique SecurityDomainExchangeKey.
1. Créez puis chargez le « Fichier de transfert de domaine de sécurité ». Vous aurez besoin des clés privées qui chiffrent le domaine de sécurité. Les clés privées sont utilisées localement et ne sont pas transférées lors de ce processus.
1. Effectuez une sauvegarde du nouveau HSM. Une sauvegarde est nécessaire avant toute restauration, même si le HSM est vide. Les sauvegardes permettent d’effectuer des restaurations simples.
1. Restaurer la sauvegarde la plus récente du HSM à partir du HSM source

Ces étapes vont vous permettre de répliquer manuellement le contenu du HSM dans une autre région. Le nom du HSM (et l’URI du point de terminaison de service) seront différents. Vous devrez donc peut-être modifier la configuration de votre application pour utiliser ces clés à partir d’un autre emplacement.

## <a name="create-a-new-managed-hsm"></a>Créer un HSM managé

Utilisez la commande `az keyvault create` pour créer un HSM managé. Ce script a trois paramètres obligatoires : le nom d’un groupe de ressources, le nom d’un HSM et l’emplacement géographique.

Vous devez fournir les entrées suivantes pour créer une ressource de HSM managé :

- Le nom du HSM.
- Le groupe de ressources dans lequel il sera placé, à l’intérieur de votre abonnement.
- L’emplacement Azure.
- La liste des administrateurs initiaux.

L’exemple ci-dessous crée un HSM nommé **ContosoMHSM** dans le groupe de ressources **ContosoResourceGroup** qui réside à l’emplacement **USA Est 2**, avec l’**utilisateur actuellement connecté** comme seul administrateur.

```azurecli-interactive
oid=$(az ad signed-in-user show --query objectId -o tsv)
az keyvault create --hsm-name "ContosoMHSM" --resource-group "ContosoResourceGroup" --location "East US 2" --administrators $oid
```

> [!NOTE]
> La création peut prendre quelques minutes. Une fois celle-ci terminée, vous êtes prêt à activer votre HSM.

La sortie de cette commande affiche les propriétés du HSM managé que vous avez créé. Les deux propriétés les plus importantes sont :

* **nom** : Dans l’exemple, son nom est ContosoMHSM. Vous allez utiliser ce nom pour d’autres commandes Key Vault.
* **hsmUri** : Dans l’exemple, l’URI est « https://contosohsm.managedhsm.azure.net  ». les applications qui utilisent votre HSM via son API REST doivent utiliser cet URI.

Votre compte Azure est pour l’instant le seul autorisé à effectuer des opérations sur ce HSM managé. À l’heure actuelle, personne d’autre n’y est autorisé.

## <a name="activate-the-security-domain-recovery-mode"></a>Activer le mode Récupération du domaine de sécurité

Dans le processus de création normal, à ce stade, nous initialisons puis téléchargeons un nouveau domaine de sécurité. Toutefois, étant donné que nous exécutons une procédure de récupération d’urgence, nous demandons au HSM de passer en mode de récupération de domaine de sécurité et de télécharger une clé d’échange de domaine de sécurité. La clé d’échange de domaine de sécurité est une clé publique RSA qui sera utilisée pour chiffrer le domaine de sécurité avant son chargement dans le HSM. La clé privée correspondante est protégée dans le HSM, afin de sécuriser le contenu de votre domaine de sécurité pendant le transfert.

```azurecli-interactive
az keyvault security-domain init-recovery --hsm-name ContosoMHSM2 --sd-exchange-key ContosoMHSM2-SDE.cer
```

## <a name="upload-security-domain-to-destination-hsm"></a>Charger le domaine de sécurité dans le HSM de destination

Pour cette étape, vous aurez besoin des éléments suivants :
- La clé d’échange de domaine de sécurité que vous avez téléchargée à l’étape précédente.
- Le domaine de sécurité du HSM source.
- Au moins le numéro de quorum des clés privées qui sont utilisées pour chiffrer le domaine de sécurité.

La commande `az keyvault security-domain upload` effectue les opérations suivantes :

- Déchiffre le domaine de sécurité du HSM source à l’aide des clés privées que vous fournissez 
- Crée un objet blob de chargement de domaine de sécurité chiffré à partir de la clé d’échange de domaine de sécurité que nous avons téléchargée à l’étape précédente
- Charge l’objet blob de chargement de domaine de sécurité dans le HSM pour effectuer la récupération du domaine de sécurité

Dans l’exemple ci-dessous, nous utilisons le domaine de **ContosoMHSM** ainsi que les deux clés privées correspondantes, puis nous les chargeons dans **ContosoMHSM2**, qui attend de recevoir un domaine de sécurité. 

```azurecli-interactive
az keyvault security-domain upload --hsm-name ContosoMHSM2 --sd-exchange-key ContosoMHSM-SDE.cer --sd-file ContosoMHSM-SD.json --sd-wrapping-keys cert_0.key cert_1.key
```

Désormais, le HSM source (ContosoMHSM) et le HSM de destination (ContosoMHSM2) ont le même domaine de sécurité. Nous pouvons maintenant restaurer une sauvegarde complète à partir du HSM source dans le HSM de destination.

## <a name="create-a-backup-as-a-restore-point-of-your-new-hsm"></a>Créer une sauvegarde (comme point de restauration) de votre nouveau HSM

Il est toujours judicieux de procéder à une sauvegarde complète avant d’exécuter une restauration HSM complète, afin de disposer d’un point de restauration en cas de problème avec la restauration.

Pour créer une sauvegarde HSM, vous aurez besoin des éléments suivants :
- Un compte de stockage dans lequel stocker la sauvegarde
- Un conteneur Stockage Blob situé dans ce compte de stockage, dans lequel le processus de sauvegarde créera un nouveau dossier où stocker les sauvegardes chiffrées

Nous appliquons la commande `az keyvault backup` à la sauvegarde du HSM dans le conteneur de stockage **mhsmbackupcontainer**, qui se trouve dans le compte de stockage **ContosoBackup** pour l’exemple ci-dessous. Nous créons un jeton SAS qui expire au bout de 30 minutes et nous le fournissons au HSM managé pour l’écriture de la sauvegarde.

```azurecli-interactive
end=$(date -u -d "30 minutes" '+%Y-%m-%dT%H:%MZ')
skey=$(az storage account keys list --query '[0].value' -o tsv --account-name ContosoBackup)
az storage container create --account-name  mhsmdemobackup --name mhsmbackupcontainer  --account-key $skey
sas=$(az storage container generate-sas -n mhsmbackupcontainer --account-name ContosoBackup --permissions crdw --expiry $end --account-key $skey -o tsv)
az keyvault backup start --hsm-name ContosoMHSM2 --storage-account-name ContosoBackup --blob-container-name mhsmdemobackupcontainer --storage-container-SAS-token $sas

```

## <a name="restore-backup-from-source-hsm"></a>Restaurer la sauvegarde à partir du HSM source

Pour cette étape, vous aurez besoin des éléments suivants :

- Le compte de stockage et le conteneur d’objets blob où sont stockées les sauvegardes du HSM source
- Le nom du dossier à partir duquel vous souhaitez restaurer la sauvegarde Si vous créez des sauvegardes régulièrement, il y aura de nombreux dossiers à l’intérieur de ce conteneur.


```azurecli-interactive
end=$(date -u -d "30 minutes" '+%Y-%m-%dT%H:%MZ')
skey=$(az storage account keys list --query '[0].value' -o tsv --account-name ContosoBackup)
sas=$(az storage container generate-sas -n mhsmdemobackupcontainer --account-name ContosoBackup --permissions rl --expiry $end --account-key $skey -o tsv)
az keyvault restore start --hsm-name ContosoMHSM2 --storage-account-name ContosoBackup --blob-container-name mhsmdemobackupcontainer --storage-container-SAS-token $sas --backup-folder mhsm-ContosoMHSM-2020083120161860
```

Vous avez maintenant terminé le processus complet de récupération d’urgence. Le contenu du HSM source au moment de la sauvegarde est copié dans le HSM de destination, ainsi que l’ensemble des clés, versions, attributs, étiquettes et attributions de rôles.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur le domaine de sécurité, consultez [À propos du domaine de sécurité du HSM managé](security-domain.md)
- Suivez les [bonnes pratiques du HSM managé](best-practices.md).
