---
title: Provisionner une instance Azure Spring Cloud avec Terraform
description: Provisionnez une instance Azure Spring Cloud avec Terraform.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 06/26/2020
ms.custom: devx-track-java
ms.openlocfilehash: 060ef2d08b849706b47b24748142c608292971b5
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96533789"
---
# <a name="provision-an-azure-spring-cloud-instance-with-terraform"></a>Provisionner une instance Azure Spring Cloud avec Terraform

**Cet article s’applique à :** ✔️ Java ✔️ C#

Cet exemple crée une instance Azure Spring Cloud à l’aide de Terraform. Les procédures vous guident tout au long de la création des ressources suivantes :

> [!div class="checklist"]
> * Groupe de ressources
> * Instance Azure Spring Cloud
> * Stockage Azure pour Log Analytics

> [!NOTE]
> Pour une prise en charge de Terraform, utilisez l’un des canaux de support suivants de la communauté HashiCorp concernant Terraform :
>
> * Questions, cas d’utilisation et modèles utiles : [Section Terraform du portail de la communauté HashiCorp](https://discuss.hashicorp.com/c/terraform-core)
> * Questions relatives aux fournisseurs : [Section concernant les fournisseurs Terraform du portail de la communauté HashiCorp](https://discuss.hashicorp.com/c/terraform-providers)

## <a name="prerequisites"></a>Prérequis

- **Abonnement Azure** : Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) avant de commencer.

## <a name="create-configuration-file"></a>Création d’un fichier de configuration

1. Connectez-vous au [portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).

1. Ouvrez [Azure Cloud Shell](../app-service/quickstart-java.md#use-azure-cloud-shell).

1. Démarrez l'éditeur Cloud Shell :

    ```bash
    code main.tf
    ```

1. La configuration dans cette étape modélise des ressources Azure, notamment un groupe de ressources Azure et une instance Azure Spring Cloud.

    ```hcl
    provider "azurerm" {
        features {}
    }

    resource "azurerm_resource_group" "example" {
      name     = "username"
      location = "eastus"
    }

    resource "azurerm_spring_cloud_service" "example" {
      name                = "usernamesp"
      resource_group_name = azurerm_resource_group.example.name
      location            = azurerm_resource_group.example.location

      config_server_git_setting {
        uri          = "https://github.com/Azure-Samples/piggymetrics-config"
        label        = "master"
        search_paths = ["."]
      }
    }
    ```

1. Enregistrez le fichier ( **&lt;Ctrl>S**) et quittez l’éditeur ( **&lt;Ctrl>Q**).

## <a name="apply-the-configuration"></a>Appliquer la configuration

Dans cette section, vous allez utiliser différentes commandes Terraform pour exécuter la configuration.

1. La commande [terraform init](https://www.terraform.io/docs/commands/init.html) initialise le répertoire de travail. Exécutez la commande suivante dans Cloud Shell :

    ```bash
    terraform init
    ```

1. La commande [terraform plan](https://www.terraform.io/docs/commands/plan.html) permet de valider la syntaxe de configuration. Le paramètre `-out` dirige les résultats vers un fichier. Le fichier de sortie pourra être utilisé ultérieurement pour appliquer la configuration. Exécutez la commande suivante dans Cloud Shell :

    ```bash
    terraform plan --out plan.out
    ```

1. La commande [terraform apply](https://www.terraform.io/docs/commands/apply.html) permet d’appliquer la configuration. La commande spécifie le fichier de sortie de l’étape précédente. Cette commande crée les ressources Azure. Exécutez la commande suivante dans Cloud Shell :

    ```bash
    terraform apply plan.out
    ```

1. Pour vérifier les résultats sur le portail Azure, accédez au nouveau groupe de ressources. La nouvelle instance d’**Azure Spring Cloud** s’affiche dans le nouveau groupe de ressources.

## <a name="update-configuration-to-config-logs-and-metrics"></a>Mettre à jour la configuration pour activer les journaux et les métriques

Cette section montre comment mettre à jour la configuration pour activer les journaux et les métriques pour Azure Spring Cloud avec un compte Stockage Azure.

1. Démarrez l'éditeur Cloud Shell :

    ```bash
    code main.tf
    ```

1. Ajoutez la configuration suivante à la fin du fichier :

    ```hcl
    resource "azurerm_storage_account" "example" {
      name                     = "usernamest"
      resource_group_name      = azurerm_resource_group.example.name
      location                 = azurerm_resource_group.example.location
      account_tier             = "Standard"
      account_replication_type = "GRS"
    }

    resource "azurerm_monitor_diagnostic_setting" "example" {
      name               = "example"
      target_resource_id = "${azurerm_spring_cloud_service.example.id}"
      storage_account_id = "${azurerm_storage_account.example.id}"

      log {
        category = "SystemLogs"
        enabled  = true

        retention_policy {
          enabled = false
        }
      }

      metric {
        category = "AllMetrics"

        retention_policy {
          enabled = false
        }
      }
    }
    ```

1. Enregistrez le fichier ( **&lt;Ctrl>S**) et quittez l’éditeur ( **&lt;Ctrl>Q**).

1. Comme dans la section précédente, exécutez la commande suivante pour apporter les modifications :

    ```bash
    terraform plan --out plan.out
    ```

1. Exécutez la commande `terraform apply` pour appliquer la configuration.

    ```bash
    terraform apply plan.out
    ```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez les ressources créées dans cet article.

Exécutez la commande [terraform destroy](https://www.terraform.io/docs/commands/destroy.html) pour supprimer les ressources Azure créées dans cet exercice :

```bash
terraform destroy -auto-approve
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Installer et configurer Terraform pour approvisionner des ressources Azure](/azure/developer/terraform/getting-started-cloud-shell).