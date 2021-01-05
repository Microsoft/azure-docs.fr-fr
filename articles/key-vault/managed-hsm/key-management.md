---
title: Gérer les clés dans un HSM managé - Azure Key Vault | Microsoft Docs
description: Utilisez cet article pour gérer les clés dans un HSM managé
services: key-vault
author: amitbapat
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: tutorial
ms.date: 09/15/2020
ms.author: ambapat
ms.openlocfilehash: 3f054638e09061c652946c9c2db1a32db73c23d9
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92521031"
---
# <a name="manage-a-managed-hsm-using-the-azure-cli"></a>Gérer un HSM managé à l’aide de l’interface Azure CLI

> [!NOTE]
> Key Vault prend en charge deux types de ressources : les coffres et les HSM managés. Cet article concerne le **HSM managé** . Si vous souhaitez apprendre à gérer un coffre, consultez [Gérer Key Vault à l’aide de l’interface de ligne de commande Azure](../general/manage-with-cli2.md).

Pour obtenir une vue d’ensemble du HSM managé, consultez [Qu’est-ce que HSM managé ?](overview.md)

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour effectuer les étapes de cet article, vous devez disposer des éléments suivants :

* Un abonnement à Microsoft Azure. Si vous n’en avez pas, vous pouvez vous inscrire pour bénéficier d’un [essai gratuit](https://azure.microsoft.com/pricing/free-trial).
* Azure CLI 2.12.0 ou version ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI]( /cli/azure/install-azure-cli).
* Un HSM managé dans votre abonnement. Consultez [Démarrage rapide : Provisionner et activer un HSM managé à l’aide d’Azure CLI](quick-create-cli.md) pour provisionner et activer un HSM managé.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="sign-in-to-azure"></a>Connexion à Azure

Pour vous connecter à Azure à l’aide de l’interface CLI, vous pouvez taper la commande suivante :

```azurecli
az login
```

Pour plus d’informations sur les options de connexion via l’interface CLI, consultez [Se connecter avec Azure CLI](/cli/azure/authenticate-azure-cli?view=azure-cli-latest&preserve-view=true)

> [!NOTE]
> Toutes les commandes ci-dessous présentent deux méthodes d’utilisation. L’une avec les paramètres **--hsm-name** et **--name** (pour le nom de la clé) et une autre avec le paramètre **--id** où vous pouvez spécifier l’intégralité de l’URL, notamment le nom de la clé le cas échéant. Cette dernière méthode est utile quand l’appelant (un utilisateur ou une application) ne dispose pas d’un accès en lecture au plan de contrôle et a uniquement un accès limité au plan de données.

## <a name="create-an-hsm-key"></a>Créer une clé HSM

Utilisez la commande `az keyvault key create` pour créer une clé.

### <a name="create-an-rsa-key"></a>Créer une clé RSA

L’exemple ci-dessous montre comment créer une clé **RSA** 3072 bits qui sera utilisée uniquement pour des opérations **wrapKey, unwrapKey** (--ops). 


```azurecli-interactive
az keyvault key create --hsm-name ContosoMHSM --name myrsakey --ops wrapKey unwrapKey --kty RSA-HSM --size 3072

## OR
# Note the key name (myrsakey) in the URI

az keyvault key create --id https://ContosoMHSM.managedhsm.azure.net/keys/myrsakey --ops wrapKey unwrapKey --kty RSA-HSM --size 3072
```

Notez que l’opération `get` retourne uniquement la clé publique et les attributs de clé. Elle ne retourne pas la clé privée (en cas de clé asymétrique) ni le matériel clé (en cas de clé symétrique).

### <a name="create-an-ec-key"></a>Créer une clé EC

L’exemple ci-dessous montre comment créer une clé **EC** avec la courbe P-256 qui sera utilisée uniquement pour les opérations **signer et vérifier** (--ops) et comporte deux balises, d’ **utilisation** et **appname** . Les balises vous aident à ajouter des métadonnées supplémentaires à la clé pour le suivi et la gestion.

```azurecli-interactive
az keyvault key create --hsm-name ContosoMHSM --name myec256key --ops sign verify  --tags ‘usage=signing] appname=myapp’ --kty EC-HSM --curve P-256

## OR
# Note the key name (myec256key) in the URI

az keyvault key create --id https://ContosoMHSM.managedhsm.azure.net/keys/myec256key --ops sign verify  --tags ‘usage=signing] appname=myapp’ --kty EC-HSM --curve P-256
```

### <a name="create-a-256-bit-symmetric-key"></a>Créer une clé symétrique 256 bits

L’exemple ci-dessous montre comment créer une clé **symétrique** 256 bits qui sera utilisée uniquement pour les opérations **chiffrer et déchiffrer** (--ops).

```azurecli-interactive
az keyvault key create --hsm-name ContosoMHSM --name myaeskey --ops encrypt decrypt  --tags --kty oct-HSM --size 256

## OR
# Note the key name (myaeskey) in the URI

az keyvault key create --id https://ContosoMHSM.managedhsm.azure.net/keys/myaeskey --ops encrypt decrypt  --tags ‘usage=signing] appname=myapp’ --kty oct-HSM --size 256
```

## <a name="view-key-attributes-and-tags"></a>Afficher les balises et les attributs de clé

Utilisez la commande `az keyvault key show` pour afficher les attributs, les versions et les balises d’une clé.

```azurecli-interactive
az keyvault key show --hsm-name ContosoHSM --name myrsakey

## OR
# Note the key name (myaeskey) in the URI

az keyvault key show --id https://ContosoMHSM.managedhsm.azure.net/keys/myrsakey

```

## <a name="list-keys"></a>Afficher la liste des clés

Utilisez la commande `az keyvault key list` pour lister toutes les clés à l’intérieur d’un HSM managé.

```azurecli-interactive
az keyvault key list --hsm-name ContosoHSM

## OR
# use full URI

az keyvault key list --id https://ContosoMHSM.managedhsm.azure.net/

```

## <a name="delete-a-key"></a>Supprimer une clé

Utilisez la commande `az keyvault key delete` pour supprimer une clé d’un HSM managé. Notez que la suppression réversible est toujours activée. Par conséquent, une clé supprimée reste dans l’état supprimé et peut être récupérée jusqu’à ce que le nombre de jours de conservation soit écoulé quand la clé est vidée (supprimée définitivement) sans récupération possible.

```azurecli-interactive
az keyvault key delete --hsm-name ContosoHSM --name myrsakey

## OR
# Note the key name (myaeskey) in the URI

az keyvault key delete --id https://ContosoMHSM.managedhsm.azure.net/keys/myrsakey
```

## <a name="list-deleted-keys"></a>Lister les clés supprimées

Utilisez la commande `az keyvault key list-deleted` pour lister toutes les clés à l’état supprimé dans votre HSM managé.

```azurecli-interactive
az keyvault key list-deleted --hsm-name ContosoHSM

## OR
# use full URI

az keyvault key list-deleted --id https://ContosoMHSM.managedhsm.azure.net/
```

## <a name="recover-undelete-a-deleted-key"></a>Récupérer une clé supprimée (en annuler la suppression)

Utilisez la commande `az keyvault key list-deleted` pour lister toutes les clés à l’état supprimé dans votre HSM managé. Si vous devez récupérer une clé supprimée (en annuler la suppression) à l’aide du paramètre --id, vous devez noter la valeur `recoveryId` de la clé supprimée obtenue à partir de la commande `az keyvault key list-deleted`.

```azurecli-interactive
az keyvault key recover --hsm-name ContosoHSM --name myrsakey

## OR
# Note the key name (myaeskey) in the URI

az keyvault key recover --id https://ContosoMHSM.managedhsm.azure.net/deletedKeys/myrsakey
```

## <a name="purge-permanently-delete-a-key"></a>Vider (supprimer définitivement) une clé

Utilisez la commande `az keyvault key purge` pour vider (supprimer définitivement) une clé.

> [!NOTE]
> Si la protection contre la suppression définitive est activée pour le HSM managé, l’opération de vidage ne sera pas autorisée. La clé est automatiquement vidée lorsque la période de conservation est écoulée.

```azurecli-interactive
az keyvault key purge --hsm-name ContosoHSM --name myrsakey

## OR
# Note the key name (myaeskey) in the URI

az keyvault key recover --id https://ContosoMHSM.managedhsm.azure.net/deletedKeys/myrsakey

```

## <a name="create-a-single-key-backup"></a>Créer une sauvegarde de clé unique

Utilisez `az keyvault key backup` pour créer une sauvegarde de clé. Le fichier de sauvegarde est un objet blob chiffré, lié par chiffrement au domaine de sécurité du HSM source. Il ne peut être restauré que dans des HSM partageant le même domaine de sécurité. Découvrez plus en détail le [domaine de sécurité](security-domain.md).

```azurecli-interactive
az keyvault key backup --hsm-name ContosoHSM --name myrsakey --file myrsakey.backup

## OR
# Note the key name (myaeskey) in the URI

az keyvault key backup --id https://ContosoMHSM.managedhsm.azure.net/deletedKeys/myrsakey  --file myrsakey.backup

```

## <a name="restore-a-single-key-from-backup"></a>Restaurer une clé unique à partir d’une sauvegarde

Utilisez `az keyvault key restore` pour restaurer une clé unique. Le HSM source dans lequel la sauvegarde a été créée doit partager le même domaine de sécurité que le HSM cible dans lequel la clé est restaurée.

> [!NOTE]
> La restauration échoue si une clé portant le même nom existe dans un état actif ou supprimé.

```azurecli-interactive
az keyvault key restore --hsm-name ContosoHSM --name myrsakey --file myrsakey.bakup

## OR
# Note the key name (myaeskey) in the URI

az keyvault key recover --id https://ContosoMHSM.managedhsm.azure.net/deletedKeys/myrsakey --file myrsakey.bakup

```

## <a name="import-a-key-from-a-file"></a>Importer une clé à partir d’un fichier

Utilisez la commande `az keyvault key import` pour importer une clé (uniquement RSA et EC) à partir d’un fichier. Le fichier de certificat doit avoir une clé privée et doit utiliser l’encodage PEM (comme défini dans les RFC [1421](https://tools.ietf.org/html/rfc1421), [1422](https://tools.ietf.org/html/rfc1422), [1423](https://tools.ietf.org/html/rfc1423), [1424](https://tools.ietf.org/html/rfc1424)).

```azurecli-interactive
az keyvault key import --hsm-name ContosoHSM --name myrsakey --pem-file mycert.key --password 'mypassword'

## OR
# Note the key name (myaeskey) in the URI

az keyvault key recover --id https://ContosoMHSM.managedhsm.azure.net/deletedKeys/myrsakey --pem-file mycert.key --password 'mypassword'
```

Pour importer une clé de votre HSM local vers le HSM managé, consultez [Importer des clés protégées par HSM dans un HSM managé (BYOK)](hsm-protected-keys-byok.md)

## <a name="next-steps"></a>Étapes suivantes

- Pour accéder à une documentation de référence complète sur l’interface Azure CLI pour les commandes Key Vault, consultez le document de [référence sur l’interface de ligne de commande Key Vault](/cli/azure/keyvault).
- Voir les informations de référence sur la programmation dans le [Guide du développeur Azure Key Vault](../general/developers-guide.md).
- Apprenez-en davantage sur la [gestion du rôle HSM managé](role-management.md)
- Apprenez-en davantage sur les [bonnes pratiques pour HSM managé](best-practices.md)
