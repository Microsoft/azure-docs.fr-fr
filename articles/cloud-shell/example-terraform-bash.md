---
title: Effectuer un déploiement avec Terraform à partir d’Azure Cloud Shell | Microsoft Docs
description: Effectuer un déploiement avec Terraform à partir d’Azure Cloud Shell
services: Azure
documentationcenter: ''
author: tomarchermsft
manager: routlaw
tags: azure-cloud-shell
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 11/15/2017
ms.author: tarcher
ms.custom: devx-track-terraform, devx-track-azurecli
ms.openlocfilehash: dbbd1511a6063223a2b76786b3c7c4394e2fa407
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92745440"
---
# <a name="deploy-with-terraform-from-bash-in-azure-cloud-shell"></a>Effectuer un déploiement avec Terraform à partir de Bash dans Azure Cloud Shell
Cet article vous guide dans la création d’un groupe de ressources avec le [fournisseur Terraform AzureRM](https://www.terraform.io/docs/providers/azurerm/index.html).

[Hashicorp Terraform](https://www.terraform.io/) est un outil open source qui codifie les API dans des fichiers de configuration déclaratifs qui peuvent être partagés entre les membres d’une équipe, qui peuvent les modifier, les réviser et gérer leurs versions. Le fournisseur Microsoft AzureRM est utilisé pour interagir avec les ressources prises en charge par Azure Resource Manager via les API AzureRM.

## <a name="automatic-authentication"></a>Authentification automatique
Terraform est installé par défaut dans Bash dans Cloud Shell. Cloud Shell authentifie automatiquement votre abonnement Azure CLI par défaut pour déployer des ressources via les modules Terraform Azure.

Terraform utilise l’abonnement Azure CLI par défaut qui est défini. Pour mettre à jour les abonnements par défaut, exécutez :

```azurecli-interactive
az account set --subscription mySubscriptionName
```

## <a name="walkthrough"></a>Procédure pas à pas
### <a name="launch-bash-in-cloud-shell"></a>Lancer Bash dans Cloud Shell
1. Lancer Cloud Shell à partir de votre emplacement favori
2. Vérifier que votre abonnement favori est défini

```azurecli-interactive
az account show
```

### <a name="create-a-terraform-template"></a>Créer un modèle Terraform
Créez un modèle Terraform nommé main.tf avec votre éditeur de texte favori.

```
vim main.tf
```

Copiez-collez le code suivant dans Cloud Shell.

```
resource "azurerm_resource_group" "myterraformgroup" {
    name = "myRgName"
    location = "West US"
}
```

Enregistrez votre fichier et quittez l’éditeur de texte.

### <a name="terraform-init"></a>Terraform init
Commencez par exécuter `terraform init`.

```
justin@Azure:~$ terraform init

Initializing provider plugins...

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, it is recommended to add version = "..." constraints to the
corresponding provider blocks in configuration, with the constraint strings
suggested below.

* provider.azurerm: version = "~> 0.2"

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

La [commande terraform init](https://www.terraform.io/docs/commands/init.html) est utilisée pour initialiser un répertoire de travail contenant des fichiers de configuration Terraform. La commande `terraform init` est la première commande qui doit être exécutée après l’écriture d’une nouvelle configuration Terraform ou le clonage d’une configuration existante à partir de la gestion de versions. Vous pouvez exécuter cette commande plusieurs fois.

### <a name="terraform-plan"></a>plan Terraform
Affichez un aperçu des ressources qui seront créées par le modèle Terraform avec `terraform plan`.

```
justin@Azure:~$ terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.


------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  + azurerm_resource_group.demo
      id:       <computed>
      location: "westus"
      name:     "myRGName"
      tags.%:   <computed>


Plan: 1 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```

La [commande terraform plan](https://www.terraform.io/docs/commands/plan.html) est utilisée pour créer un plan d’exécution. Terraform effectue une actualisation, sauf si celle-ci est explicitement désactivée, et détermine les actions nécessaires pour atteindre l’état souhaité spécifié dans les fichiers de configuration. Vous pouvez enregistrer le plan en spécifiant -out, puis le fournir à terraform apply pour garantir que seules les actions préplanifiées sont exécutées.

### <a name="terraform-apply"></a>Terraform apply
Provisionnez les ressources Azure avec `terraform apply`.

```
justin@Azure:~$ terraform apply
azurerm_resource_group.demo: Creating...
  location: "" => "westus"
  name:     "" => "myRGName"
  tags.%:   "" => "<computed>"
azurerm_resource_group.demo: Creation complete after 0s (ID: /subscriptions/mySubIDmysub/resourceGroups/myRGName)

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

La [commande terraform apply](https://www.terraform.io/docs/commands/apply.html) est utilisée pour appliquer les modifications nécessaires permettant d’atteindre l’état souhaité de la configuration.

### <a name="verify-deployment-with-azure-cli"></a>Vérifier le déploiement avec Azure CLI
Exécutez `az group show -n myRgName` pour vérifier que la ressource a été correctement provisionnée.

```azurecli-interactive
az group show -n myRgName
```

### <a name="clean-up-with-terraform-destroy"></a>Nettoyer avec terraform destroy
Nettoyez le groupe de ressources créé avec la [commande Terraform destroy](https://www.terraform.io/docs/commands/destroy.html) pour nettoyer l’infrastructure créée par Terraform.

```
justin@Azure:~$ terraform destroy
azurerm_resource_group.demo: Refreshing state... (ID: /subscriptions/mySubID/resourceGroups/myRGName)

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  - azurerm_resource_group.demo


Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

azurerm_resource_group.demo: Destroying... (ID: /subscriptions/mySubID/resourceGroups/myRGName)
azurerm_resource_group.demo: Still destroying... (ID: /subscriptions/mySubID/resourceGroups/myRGName, 10s elapsed)
azurerm_resource_group.demo: Still destroying... (ID: /subscriptions/mySubID/resourceGroups/myRGName, 20s elapsed)
azurerm_resource_group.demo: Still destroying... (ID: /subscriptions/mySubID/resourceGroups/myRGName, 30s elapsed)
azurerm_resource_group.demo: Still destroying... (ID: /subscriptions/mySubID/resourceGroups/myRGName, 40s elapsed)
azurerm_resource_group.demo: Destruction complete after 45s

Destroy complete! Resources: 1 destroyed.
```

Vous avez créé une ressource Azure via Terraform. Consultez les étapes suivantes pour continuer à découvrir Cloud Shell.

## <a name="next-steps"></a>Étapes suivantes
[En savoir plus sur le fournisseur Terraform Azure](https://www.terraform.io/docs/providers/azurerm/#)<br>
[Démarrage rapide de Bash dans Cloud Shell](quickstart.md)
