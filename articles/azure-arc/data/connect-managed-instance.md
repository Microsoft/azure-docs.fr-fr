---
title: Se connecter à SQL Managed Instance avec Azure Arc
description: Se connecter à SQL Managed Instance avec Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: vin-yu
ms.author: vinsonyu
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: abd27e15ccf5b421e69e78b2b726d192ffdecacb
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92372359"
---
# <a name="connect-to-azure-arc-enabled-sql-managed-instance"></a>Se connecter à SQL Managed Instance avec Azure Arc

Cet article explique comment établir une connexion à votre instance Managed Instance SQL pour Azure Arc. 

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="view-azure-arc-enabled-sql-managed-instances"></a>Afficher les instances SQL Managed Instance avec Azure Arc

Pour afficher l’instance Azure SQL Managed Instance compatible avec Azure Arc et les points de terminaison externes, utilisez la commande suivante :

```console
azdata arc sql mi list
```

La sortie doit se présenter comme suit :

```console
Name    Replicas    ExternalEndpoint    State
------  ----------  ----------------  -------
sqldemo 1/1         10.240.0.4:32023  Ready
```

Si vous utilisez AKS ou kubeadm ou OpenShift, etc., vous pouvez copier l’adresse IP externe et le numéro de port à partir d’ici et vous y connecter à l’aide de votre outil favori pour vous connecter à une instance SQL Server/SQL Azure, par exemple Azure Data Studio ou SQL Server Management Studio.  Toutefois, si vous utilisez la machine virtuelle de démarrage rapide, consultez la section ci-dessous pour obtenir des informations spécifiques sur la façon de vous connecter à cette machine virtuelle en dehors d’Azure. 

> [!NOTE]
> Vos stratégies d’entreprise peuvent bloquer l’accès à l’adresse IP et au port, surtout si elles sont créées dans le cloud public.

## <a name="connect"></a>Se connecter 

Connectez-vous avec Azure Data Studio, SQL Server Management Studio ou SQLCMD.

Ouvrez Azure Data Studio et connectez-vous à votre instance avec l’adresse IP et le numéro de port du point de terminaison externe ci-avant. Si vous utilisez une machine virtuelle Azure, vous aurez besoin de l’ _adresse IP_ publique, qui est identifiable à l’aide de la [Remarque spéciale sur les déploiements de machines virtuelles Azure](#special-note-about-azure-virtual-machine-deployments).

Exemple :

- Serveur :  52.229.9.30,30913
- Nom d’utilisateur : sa
- Mot de passe : votre mot de passe SQL spécifié au moment de l’approvisionnement

> [!NOTE]
> Vous pouvez utiliser Azure Data Studio pour [Visualiser les tableaux de bord des instances SQL managées](azure-data-studio-dashboards.md#view-the-sql-managed-instance-dashboards).

Pour vous connecter à l’aide de SQLCMD, de Linux ou de Windows, vous pouvez utiliser une commande comme celle-ci. Entrez le mot de passe SQL quand vous y êtes invité :

```bash
sqlcmd -S 52.229.9.30,30913 -U sa
```

## <a name="special-note-about-azure-virtual-machine-deployments"></a>Remarque spéciale sur les déploiements de machines virtuelles Azure

Si vous utilisez une machine virtuelle Azure, l’adresse IP du point de terminaison n’affiche pas l’adresse IP publique. Utilisez la commande suivante pour localiser l’adresse IP externe :

```azurecli
az network public-ip list -g azurearcvm-rg --query "[].{PublicIP:ipAddress}" -o table
```

Vous pouvez ensuite combiner l’adresse IP publique avec le port pour établir votre connexion.

Vous devrez peut-être également exposer le port de l’instance sql à l’aide de la passerelle de sécurité réseau (NSG). Pour autoriser le trafic via la passerelle de sécurité réseau (NSG), vous devez ajouter une règle que vous pouvez effectuer à l’aide de la commande suivante.

Pour définir une règle, vous devez connaître le nom de votre groupe de sécurité réseau, que vous pouvez découvrir à l’aide de la commande ci-dessous :

```azurecli
az network nsg list -g azurearcvm-rg --query "[].{NSGName:name}" -o table
```

Une fois que vous avez le nom du groupe de sécurité réseau, vous pouvez ajouter une règle de pare-feu à l’aide de la commande suivante. Les exemples de valeurs ci-dessous créent une règle NSG pour le port 30913 et autorisent la connexion à partir de **n’importe quelle adresse IP source** .  Il ne s’agit pas d'une meilleure pratique de sécurité.  Vous pouvez mieux verrouiller les choses en spécifiant une valeur-source-adresse-préfixes spécifique à votre adresse IP du client ou une plage d’adresses IP qui couvre les adresses IP de votre équipe ou de votre organisation.

Remplacez la valeur du paramètre `--destination-port-ranges` ci-dessous par le numéro de port que vous avez obtenu à partir de la commande `azdata sql instance list`F ci-dessus.

```azurecli
az network nsg rule create -n db_port --destination-port-ranges 30913 --source-address-prefixes '*' --nsg-name azurearcvmNSG --priority 500 -g azurearcvm-rg --access Allow --description 'Allow port through for db access' --destination-address-prefixes '*' --direction Inbound --protocol Tcp --source-port-ranges '*'
```

## <a name="next-steps"></a>Étapes suivantes

- [Visualiser les tableaux de bord des instances SQL managées](azure-data-studio-dashboards.md#view-the-sql-managed-instance-dashboards)
- [Afficher SQL Managed Instance dans le Portail Azure](view-arc-data-services-inventory-in-azure-portal.md)
