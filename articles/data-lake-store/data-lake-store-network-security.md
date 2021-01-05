---
title: Sécurité du réseau dans Azure Data Lake Storage Gen1 | Microsoft Docs
description: Comprendre l’intégration au réseau virtuel dans Azure Data Lake Storage Gen1
services: data-lake-store
documentationcenter: ''
author: twooley
manager: mtillman
editor: cgronlun
ms.service: data-lake-store
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 10/09/2018
ms.author: elsung
ms.openlocfilehash: 716e3766fdd7c1999efa12456346862a9902d7a0
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92108709"
---
# <a name="virtual-network-integration-for-azure-data-lake-storage-gen1"></a>Intégration de réseau virtuel dans Azure Data Lake Storage Gen1

Cet article présente l’intégration au réseau virtuel pour Azure Data Lake Storage Gen1. Avec l’intégration de réseau virtuel, vous pouvez configurer vos comptes pour qu’ils acceptent uniquement le trafic provenant de sous-réseaux et de réseaux virtuels spécifiques. 

Cette fonctionnalité permet de protéger votre compte Data Lake Storage contre les menaces externes.

L’intégration au réseau virtuel dans Data Lake Storage Gen1 fait appel à la sécurité des points de terminaison de service de réseau virtuel entre votre réseau virtuel et Azure Active Directory (Azure AD) afin de générer des revendications de sécurité supplémentaires dans le jeton d’accès. Ces revendications permettent ensuite d’authentifier votre réseau virtuel à votre compte Data Lake Storage Gen1 et d’y accéder.

> [!NOTE]
> L’utilisation de ces fonctionnalités n’entraîne aucuns frais supplémentaires. Votre compte est facturé au tarif standard de Data Lake Storage Gen1. Pour plus d’informations, voir la [tarification](https://azure.microsoft.com/pricing/details/data-lake-store/?cdn=disable). Pour tous les autres services Azure que vous utilisez, voir la [tarification](https://azure.microsoft.com/pricing/#product-picker).

## <a name="scenarios-for-virtual-network-integration-for-data-lake-storage-gen1"></a>Scénarios d’intégration de réseau virtuel dans Data Lake Storage Gen1

Avec l’intégration de réseau virtuel Data Lake Storage Gen1, vous pouvez restreindre l’accès à votre compte Data Lake Storage Gen1 à des sous-réseaux et réseaux virtuels spécifiques. Une fois que votre compte est verrouillé sur le sous-réseau de réseau virtuel spécifié, les autres réseaux virtuels/machines virtuelles d’Azure ne sont pas autorisés à y accéder. Sur le plan fonctionnel, l’intégration au réseau virtuel dans Data Lake Storage Gen1 permet le même scénario que les [points de terminaison de service de réseau virtuel](../virtual-network/virtual-network-service-endpoints-overview.md). Quelques différences importantes sont détaillées dans les sections ci-dessous. 

![Diagramme des scénarios d’intégration de réseau virtuel dans Data Lake Storage Gen1](media/data-lake-store-network-security/scenario-diagram.png)

> [!NOTE]
> Les règles de pare-feu IP existantes peuvent être utilisées en plus des règles de réseau virtuel afin d’autoriser également l’accès depuis les réseaux locaux. 

## <a name="optimal-routing-with-data-lake-storage-gen1-virtual-network-integration"></a>Routage optimal grâce à l’intégration au réseau virtuel dans Data Lake Storage Gen1

L’[optimisation des itinéraires ](../virtual-network/virtual-network-service-endpoints-overview.md#key-benefits) depuis votre réseau virtuel constitue l’un des principaux avantages des points de terminaison de service de réseau virtuel. Vous pouvez effectuer la même optimisation des itinéraires pour les comptes Data Lake Storage Gen1. Utilisez les [itinéraires définis par l’utilisateur](../virtual-network/virtual-networks-udr-overview.md#user-defined) suivants entre votre réseau virtuel et votre compte Data Lake Storage Gen1.

**Adresse IP publique Data Lake Storage** : utilisez l’adresse IP publique de vos comptes Data Lake Storage Gen1 cibles. Pour identifier les adresses IP de vos comptes Data Lake Storage Gen1, [résolvez les noms DNS](./data-lake-store-connectivity-from-vnets.md#enabling-connectivity-to-azure-data-lake-storage-gen1-from-vms-with-restricted-connectivity) de vos comptes. Créez une entrée différente pour chaque adresse.

```azurecli
# Create a route table for your resource group.
az network route-table create --resource-group $RgName --name $RouteTableName

# Create route table rules for Data Lake Storage public IP addresses.
# There's one rule per Data Lake Storage public IP address. 
az network route-table route create --name toADLSregion1 --resource-group $RgName --route-table-name $RouteTableName --address-prefix <ADLS Public IP Address> --next-hop-type Internet

# Update the virtual network, and apply the newly created route table to it.
az network vnet subnet update --vnet-name $VnetName --name $SubnetName --resource-group $RgName --route-table $RouteTableName
```

## <a name="data-exfiltration-from-the-customer-virtual-network"></a>Exfiltration des données du réseau virtuel du client

En plus de sécuriser les comptes Data Lake Storage concernant l’accès à partir des réseaux virtuels, vous souhaitez peut-être aussi vérifier l’absence d’exfiltration vers un compte non autorisé.

Utilisez une solution de pare-feu dans votre réseau virtuel pour filtrer le trafic sortant en fonction de l’URL de compte de destination. Autorisez uniquement l’accès aux comptes Data Lake Storage Gen1 approuvés.

Voici quelques options disponibles :
- [Pare-feu Azure](../firewall/overview.md) : [déployez et configurez un pare-feu Azure](../firewall/tutorial-firewall-deploy-portal.md) pour votre réseau virtuel. Sécurisez le trafic Data Lake Storage sortant et verrouillez-le sur l’URL d’un compte connu et approuvé.
- [Appliance virtuelle réseau](https://azure.microsoft.com/solutions/network-appliances/) - pare-feu : votre administrateur peut limiter le recours à certains fournisseurs de pare-feu commerciaux uniquement. Utilisez une solution de pare-feu d’appliance virtuelle réseau disponible dans la Place de marché Azure aux mêmes fins.

> [!NOTE]
> L’utilisation de pare-feu dans le chemin d’accès de données introduit un tronçon supplémentaire dans ce chemin. Cela peut affecter les performances du réseau pour l’échange de données de bout en bout. La disponibilité du débit et la latence de la connexion peuvent être affectées. 

## <a name="limitations"></a>Limites

- Les clusters HDInsight créés avant la prise en charge de l’intégration de réseau virtuel dans Data Lake Storage Gen1 doivent être recréés pour prendre en charge cette nouvelle fonctionnalité.
 
- Lorsque vous créez un nouveau cluster HDInsight et que vous sélectionnez un compte Data Lake Storage Gen1 avec l’intégration de réseau virtuel activée, le processus échoue. Commencez par désactiver la règle de réseau virtuel. Sinon, dans le panneau **Pare-feu et réseaux virtuels** du compte Data Lake Storage, sélectionnez **Autoriser l’accès à tous les réseaux et services**. Créez ensuite le cluster HDInsight avant de réactiver la règle de réseau virtuel ou de désélectionner **Autoriser l’accès à tous les réseaux et services**. Pour en savoir plus, consultez la section [Exceptions](#exceptions).

- L’intégration au réseau virtuel dans Data Lake Storage Gen1 ne fonctionne pas avec les [identités managées pour les ressources Azure](../active-directory/managed-identities-azure-resources/overview.md).
  
- Les données de fichier et de dossier de votre compte Data Lake Storage Gen 1 avec réseau virtuel activé ne sont pas accessibles à partir du portail. Cette restriction inclut notamment l’accès à partir d’une machine virtuelle située sur le réseau virtuel et d’activités comme l’utilisation de l’Explorateur de données. Les activités de gestion du compte continuent de fonctionner. Les données de fichier et de dossier de votre compte Data Lake Storage avec réseau virtuel activé sont accessibles à partir de toutes les ressources hors portail. Ces ressources incluent l’accès au SDK, les scripts PowerShell et les autres services Azure lorsqu’ils ne proviennent pas du portail. 

## <a name="configuration"></a>Configuration

### <a name="step-1-configure-your-virtual-network-to-use-an-azure-ad-service-endpoint"></a>Étape 1 : Configurer votre réseau virtuel pour qu’il utilise un point de terminaison de service Azure AD

1.  Accédez au portail Azure et connectez-vous à votre compte.
 
2.  [Créez un réseau virtuel](../virtual-network/quick-create-portal.md) dans votre abonnement. Vous pouvez également accéder à un réseau virtuel existant. Le réseau virtuel doit se trouver dans la même région que le compte Data Lake Storage Gen1.
 
3.  Dans le panneau **Réseau virtuel**, sélectionnez **Points de terminaison fournisseurs**.
 
4.  Cliquez sur **Ajouter** pour ajouter un nouveau point de terminaison fournisseur.

    ![Ajouter un point de terminaison de service de réseau virtuel](media/data-lake-store-network-security/config-vnet-1.png)

5.  Choisissez **Microsoft.AzureActiveDirectory** comme service du point de terminaison.

     ![Sélectionner le point de terminaison fournisseur Microsoft.AzureActiveDirectory](media/data-lake-store-network-security/config-vnet-2.png)

6.  Choisissez les sous-réseaux pour lesquels vous souhaitez autoriser la connectivité. Sélectionnez **Ajouter**.

    ![Sélectionner le sous-réseau](media/data-lake-store-network-security/config-vnet-3.png)

7.  L’ajout du point de terminaison fournisseur peut prendre jusqu’à 15 minutes. Après l’avoir ajouté, il s’affiche dans la liste. Vérifiez qu’il s’affiche et que tous les détails correspondent à la configuration.
 
    ![Ajout réussi du point de terminaison fournisseur](media/data-lake-store-network-security/config-vnet-4.png)

### <a name="step-2-set-up-the-allowed-virtual-network-or-subnet-for-your-data-lake-storage-gen1-account"></a>Étape 2 : Configurer le réseau virtuel ou sous-réseau autorisé pour votre compte Data Lake Storage Gen1

1.  Après avoir configuré votre réseau virtuel, [créez un compte Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md#create-a-data-lake-storage-gen1-account) dans votre abonnement. Vous pouvez également accéder à un compte Data Lake Storage Gen1 existant. Le compte Data Lake Storage Gen1 doit se trouver dans la même région que le réseau virtuel.
 
2.  Sélectionnez **Pare-feu et réseaux virtuels**.

    > [!NOTE]
    > Si **Pare-feu et réseaux virtuels** ne s’affiche pas dans les paramètres, déconnectez-vous du portail. Fermez le navigateur et effacez-en le cache. Redémarrez la machine et essayez à nouveau.

       ![Ajouter une règle de réseau virtuel à votre compte Data Lake Storage](media/data-lake-store-network-security/config-adls-1.png)

3.  Sélectionnez **Réseaux sélectionnés**.
 
4.  Sélectionnez **Add existing virtual network** (Ajouter un réseau virtuel existant).

    ![Ajouter un réseau virtuel existant](media/data-lake-store-network-security/config-adls-2.png)

5.  Sélectionnez les réseaux virtuels et sous-réseaux auxquels autoriser la connectivité. Sélectionnez **Ajouter**.

    ![Choisir le réseau virtuel et les sous-réseaux](media/data-lake-store-network-security/config-adls-3.png)

6.  Assurez-vous que les réseaux virtuels et les sous-réseaux apparaissent correctement dans la liste. Sélectionnez **Enregistrer**.

    ![Enregistrer la nouvelle règle](media/data-lake-store-network-security/config-adls-4.png)

    > [!NOTE]
    > Cela peut prendre jusqu’à 5 minutes pour que les réglages prennent effet après l’enregistrement.

7.  [Facultatif] Dans la page **Pare-feu et réseaux virtuels**, au niveau de la section **Pare-feu**, vous pouvez autoriser la connectivité à partir d’adresses IP spécifiques. 

## <a name="exceptions"></a>Exceptions
Vous pouvez autoriser la connectivité à partir de machines virtuelles et services Azure en dehors des réseaux virtuels sélectionnés. Dans le panneau **Pare-feu et réseaux virtuels**, au niveau de la section **Exceptions**, faites votre choix parmi ces deux options :
 
- **Autoriser tous les services Azure à accéder à ce compte Data Lake Storage Gen1**. Cette option permet aux services Azure comme Azure Data Factory, Azure Event Hubs et toutes les machines virtuelles Azure de communiquer avec votre compte Data Lake Storage.

- **Autoriser Azure Data Lake Analytics à accéder à ce compte Data Lake Storage Gen1**. Cette option autorise la connectivité entre Data Lake Analytics et ce compte Data Lake Storage. 

  ![Exceptions de pare-feu et de réseau virtuel](media/data-lake-store-network-security/firewall-exceptions.png)

Nous vous recommandons de laisser ces exceptions désactivées. Activez-les uniquement si vous avez besoin d’une connectivité à d’autres services en dehors de votre réseau virtuel.