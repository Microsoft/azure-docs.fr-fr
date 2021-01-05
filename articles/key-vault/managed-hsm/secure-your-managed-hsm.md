---
title: Sécuriser l’accès à un HSM managé - HSM managé Azure Key Vault
description: Découvrez comment sécuriser l’accès au HSM managé avec RBAC Azure et le RBAC local HSM managé
services: key-vault
author: amitbapat
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: tutorial
ms.date: 09/15/2020
ms.author: ambapat
ms.openlocfilehash: 99918d039052c9913400b85ac3caa4a1a5481155
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94445317"
---
# <a name="secure-access-to-your-managed-hsms"></a>Sécuriser l’accès à vos HSM managés

Le HSM managé Azure Key Vault est un service cloud qui protège les clés de chiffrement. Comme il s’agit de données sensibles et critiques, vous devez sécuriser l’accès à vos HSM managés en permettant seulement aux applications et utilisateurs autorisés d’y accéder. Cet article fournit une vue d’ensemble du modèle de contrôle d’accès HSM managé. Il décrit l’authentification et l’autorisation, puis explique comment sécuriser l’accès à vos HSM managés.

Ce tutoriel fait appel à un exemple simple qui montre comment réaliser la séparation des tâches et du contrôle d’accès avec RBAC Azure et le RBAC local HSM managé. Pour en savoir plus sur le modèle de contrôle d’accès HSM managé, consultez [Contrôle d’accès HSM managé](access-control.md).

## <a name="prerequisites"></a>Prérequis

Pour effectuer les étapes de cet article, vous devez disposer des éléments suivants :

* Un abonnement à Microsoft Azure. Si vous n’en avez pas, vous pouvez vous inscrire pour bénéficier d’un [essai gratuit](https://azure.microsoft.com/pricing/free-trial).
* Azure CLI 2.12.0 ou version ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI]( /cli/azure/install-azure-cli).
* HSM managé dans votre abonnement. Consultez [Démarrage rapide : Provisionner et activer un HSM managé à l’aide d’Azure CLI](quick-create-cli.md) pour provisionner et activer un HSM managé.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="sign-in-to-azure"></a>Connexion à Azure

Pour vous connecter à Azure à l’aide de l’interface CLI, vous pouvez taper la commande suivante :

```azurecli
az login
```

Pour plus d’informations sur les options de connexion via l’interface CLI, consultez [Se connecter avec Azure CLI](/cli/azure/authenticate-azure-cli?view=azure-cli-latest&preserve-view=true)

## <a name="example"></a>Exemple

Dans cet exemple, nous développons une application qui utilise une clé RSA 2 048 bits pour les opérations de signature. Notre application s’exécute dans une machine virtuelle Azure avec une [identité managée](../../active-directory/managed-identities-azure-resources/overview.md). La clé RSA utilisée pour la signature est stockée dans notre HSM managé.

Nous avons identifié les rôles suivants qui gèrent, déploient et auditent notre application :

- **Équipe de sécurité** : personnel informatique du bureau du chef de la sécurité ou contributeurs équivalents. L’équipe de sécurité est chargée de protéger les clés. Clés RSA ou EC pour la signature ainsi que les clés RSA ou AES pour le chiffrement des données.
- **Développeurs et opérateurs** : équipe qui développe l’application et la déploie dans Azure. Les membres de cette équipe ne font pas partie de l’équipe de sécurité. Ils ne doivent pas avoir accès aux données sensibles comme les clés RSA. Seule l’application qu’ils déploient doit avoir accès à ces données sensibles.
- **Auditeurs** : ce rôle s’applique aux contributeurs qui ne sont pas membres de l’équipe de développement ou du personnel informatique général. Ils passent en revue l’utilisation et la maintenance des certificats, clés et secrets pour assurer la conformité aux normes de sécurité.

Il existe un autre rôle qui dépasse le cadre de notre application : l’administrateur d’abonnement (ou groupe de ressources). L’administrateur d’abonnement configure les autorisations d’accès initiales pour l’équipe de sécurité. Il accorde l’accès à l’équipe de sécurité à l’aide d’un groupe de ressources comprenant les ressources requises par l’application.

Nous devons autoriser les opérations suivantes pour nos rôles :

**Équipe de sécurité**
- Créer le HSM managé.
- Télécharger le domaine de sécurité du HSM managé (pour la reprise d’activité)
- Activez la journalisation.
- Générer ou importer des clés
- Créer les sauvegardes du HSM managé pour la reprise d’activité.
- Définir le RBAC local HSM managé pour autoriser des utilisateurs et des applications à effectuer des opérations spécifiques.
- Regénérer périodiquement les clés.

**Développeurs et opérateurs**
- Obtenir une référence (URI de clé) auprès de l’équipe de sécurité de la clé RSA utilisée pour la signature.
- Développer et déployer l’application qui accède par programmation à la clé.

**Auditeurs**
- Passer en revue les dates d’expiration des clés pour vérifier qu’elles sont à jour
- Superviser les affectations de rôles pour garantir que les clés ne sont accessibles qu’aux utilisateurs/applications autorisés
- Examiner les journaux pour HSM managé afin de confirmer l’utilisation correcte des clés en conformité aux normes en matière de sécurité des données.

Le tableau suivant récapitule les affectations de rôles aux équipes et aux ressources pour accéder au HSM managé.

| Role | Rôle de plan de gestion | Rôle de plan de données |
| --- | --- | --- |
| Équipe de sécurité | Contributeur du HSM managé | Administrateur du HSM managé |
| Développeurs et opérateurs | None | None |
| Auditeurs | None | Auditeur du chiffrement du HSM managé |
| Identité managée de la machine virtuelle utilisée par l’application| None | Utilisateur du chiffrement du HSM managé |
| Identité managée du compte de stockage utilisé par l’application| None| Chiffrement du service du HSM managé |

Les trois rôles d’équipe doivent avoir accès à d’autres ressources avec des autorisations HSM managé. Pour déployer des machines virtuelles (ou la fonctionnalité Web Apps d’Azure App Service), les développeurs et opérateurs ont besoin d’un accès `Contributor` à ces types de ressources. Les auditeurs ont besoin d’un accès en lecture au compte de stockage où les journaux pour HSM managé sont stockés.

Pour affecter des rôles de plan de gestion (RBAC Azure), vous pouvez utiliser le portail Azure ou l’une des autres interfaces de gestion comme Azure CLI ou Azure PowerShell. Pour affecter des rôles de plan de données HSM managé, vous devez utiliser Azure CLI.

Les extraits Azure CLI présentés dans cette section sont générés avec les hypothèses suivantes :

- L’administrateur Azure Active Directory a créé des groupes de sécurité pour représenter les trois rôles : Contoso Security Team, Contoso App DevOps et Contoso App Auditors. L’administrateur a ajouté des utilisateurs à leurs groupes respectifs.
- Toutes les ressources se trouvent dans le groupe de ressources **ContosoAppRG**.
- Les journaux pour HSM managé sont stockés dans le compte de stockage **contosologstorage**.
- Le HSM managé **ContosoMHSM** et le compte de stockage **contosologstorage** figurent au même emplacement Azure.

L’administrateur d’abonnement affecte le rôle `Managed HSM Contributor` à l’équipe de sécurité. Ce rôle permet à l’équipe de sécurité de gérer les HSM managés existants et d’en créer de nouveaux. S’il existe des HSM managés, l’équipe doit se voir attribuer le rôle « Administrateur du HSM managé » pour pouvoir les gérer.

```azurecli-interactive
# This role assignment allows Contoso Security Team to create new Managed HSMs
az role assignment create --assignee-object-id $(az ad group show -g 'Contoso Security Team' --query 'objectId' -o tsv) --assignee-principal-type Group --role "Managed HSM Contributor"

# This role assignment allows Contoso Security Team to become administrator of existing managed HSM
az keyvault role assignment create  --hsm-name ContosoMHSM --assignee $(az ad group show -g 'Contoso Security Team' --query 'objectId' -o tsv) --scope / --role "Managed HSM Administrator"
```

L’équipe de sécurité configure la journalisation et affecte des rôles aux auditeurs et à l’application de machine virtuelle.

```azurecli-interactive
# Enable logging
hsmresource=$(az keyvault show --hsm-name ContosoMHSM --query id -o tsv)
storageresource=$(az storage account show --name contosologstorage --query id -o tsv)
az monitor diagnostic-settings create --name MHSM-Diagnostics --resource $hsmresource --logs    '[{"category": "AuditEvent","enabled": true}]' --storage-account $storageresource

# Assign the "Crypto Auditor" role to Contoso App Auditors group. It only allows them to read.
az keyvault role assignment create  --hsm-name ContosoMHSM --assignee $(az ad group show -g 'Contoso App Auditors' --query 'objectId' -o tsv) --scope / --role "Managed HSM Crypto Auditor"

# Grant the "Crypto User" role to the VM's managed identity. It allows to create and use keys. 
# However it cannot permanently delete (purge) keys
az keyvault role assignment create  --hsm-name ContosoMHSM --assignee $(az vm identity show --name "vmname" --resource-group "ContosoAppRG" --query objectId -o tsv) --scope / --role "Managed HSM Crypto Auditor"

# Assign "Managed HSM Crypto Service Encryption" role to the Storage account ID
storage_account_principal=$(az storage account show --id $storageresource --query identity.principalId -o tsv)
# (if no identity exists), then assign a new one
[ "$storage_account_principal" ] || storage_account_principal=$(az storage account update --assign-identity --id $storageresource)

az keyvault role assignment create --hsm-name ContosoMHSM --role "Managed HSM Crypto Service Encryption" --assignee $storage_account_principal
```

Ce tutoriel affiche uniquement les actions relatives au contrôle d’accès pour l’essentiel. Les autres actions et opérations liées au déploiement de l’application dans votre machine virtuelle, l’activation du chiffrement avec une clé gérée par le client pour un compte de stockage et la création d’un HSM managé ne sont pas indiquées ici pour que l’exemple reste axé sur le contrôle d’accès et la gestion des rôles.

Notre exemple décrit un scénario simple. Les scénarios réels peuvent être plus complexes. Vous pouvez ajuster les autorisations à votre coffre de clés selon vos besoins. Nous partons du principe que l’équipe de sécurité fournit les références aux clés et aux secrets (URI et empreintes) dont se sert l’équipe DevOps dans ses applications. Les développeurs et les opérateurs n’ont besoin d’aucun accès au plan de données. Nous avons vu comment sécuriser votre coffre de clés. Une considération similaire doit être accordée pour sécuriser [vos machines virtuelles](https://azure.microsoft.com/services/virtual-machines/security/), vos [comptes de stockage](../../storage/blobs/security-recommendations.md) et d’autres ressources Azure.

## <a name="resources"></a>Ressources

- [Documentation Azure RBAC](../../role-based-access-control/overview.md)
- [RBAC Azure : Rôles intégrés](../../role-based-access-control/built-in-roles.md)
- [Gérer RBAC Azure avec Azure CLI](../../role-based-access-control/role-assignments-cli.md)

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir un tutoriel de mise en route destiné aux administrateurs, consultez [Qu’est-ce que HSM managé ?](overview.md).

Pour plus d’informations sur la journalisation de l’utilisation pour la journalisation pour HSM managé, consultez [Journalisation pour HSM managé](logging.md).
