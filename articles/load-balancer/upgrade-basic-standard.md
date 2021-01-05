---
title: Passer de Public De base vers Public Standard - Azure Load Balancer
description: Cet article explique comment mettre à niveau Azure Public Load Balancer en passant de la référence SKU De base à la référence SKU Standard
services: load-balancer
author: irenehua
ms.service: load-balancer
ms.topic: how-to
ms.date: 01/23/2020
ms.author: irenehua
ms.openlocfilehash: d54994a7c64718835e70381f92abed83ef693018
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96938509"
---
# <a name="upgrade-azure-public-load-balancer"></a>Mettre à niveau l’équilibreur de charge public Azure
[Azure Standard Load Balancer](load-balancer-overview.md) offre un ensemble complet de fonctionnalités et une haute disponibilité avec la redondance de zone. Pour en savoir plus sur la référence SKU de Load Balancer, consultez le [tableau comparatif](./skus.md#skus).

Une mise à niveau se compose de deux phases :

1. Modifier la méthode d’allocation d’adresses IP de dynamique en statique.
2. Exécuter le script PowerShell pour effectuer la mise à niveau et la migration du trafic.

## <a name="upgrade-overview"></a>Présentation de la mise à niveau

Un script Azure PowerShell est disponible qui effectue les opérations suivantes :

* Il crée un SKU Standard de Load Balancer avec l’emplacement que vous spécifiez dans le même groupe de ressources que l’instance Load Balancer De base.
* Il met à niveau une IP publique d’un SKU De base à un SKU Standard sur place.
* Il copie de façon fluide les configurations de l’équilibreur de charge De base sur l’équilibreur de charge Standard nouvellement créé.
* Il crée une règle de trafic sortant par défaut qui permet une connectivité sortante.

### <a name="caveatslimitations"></a>Mises en garde/Limitations

* Le script prend en charge la mise à niveau de Public Load Balancer uniquement. Pour plus d’informations sur la mise à niveau du Basic Load Balancer interne et pour obtenir des instructions, consultez [cette page](./upgrade-basicinternal-standard.md).
* La méthode d’allocation de l’IP publique doit être remplacée par « statique » avant d’exécuter le script. 
* Si votre équilibreur de charge ne dispose pas de configuration d’adresse IP front-end ni de pool de back-ends, vous risquez de rencontrer une erreur lors de l’exécution du script. Assurez-vous qu’ils ne sont pas vides.

### <a name="change-allocation-method-of-the-public-ip-address-to-static"></a>Modifier la méthode d’allocation de l’IP publique en statique

* **Voici les étapes recommandées :

    1. Pour toutes les tâches de ce guide de démarrage rapide, connectez-vous au [portail Azure](https://portal.azure.com).
 
    1. Sélectionnez **Toutes les ressources** dans le menu de gauche, puis sélectionnez l’**IP publique De base associée à Basic Load Balancer** dans la liste de ressources.
   
    1. Sous **Paramètres**, sélectionnez **Configurations**.
   
    1. Sous **Affectation**, sélectionnez **Statique**.
    1. Sélectionnez **Enregistrer**.
    >[!NOTE]
    >Pour les machines virtuelles avec des adresses IP publiques, vous devez d’abord créer des adresses IP standard là où la même adresse IP n’est pas garantie. Dissociez les machines virtuelles des adresses IP de base et associez-les aux adresses IP standard nouvellement créées. Vous êtes alors en mesure de suivre les instructions permettant d’ajouter des machines virtuelles au pool de back-ends de Standard Load Balancer. 

* **Création de machines virtuelles à ajouter aux pools de back-ends de l’instance Standard Load Balancer publique nouvellement créée**.
    * Des instructions supplémentaires sur la création d’une machine virtuelle et son association à Standard Load Balancer sont disponibles [ici](./quickstart-load-balancer-standard-public-portal.md#create-virtual-machines).


## <a name="download-the-script"></a>Télécharger le script

Téléchargez le script de migration à partir de [PowerShell Gallery](https://www.powershellgallery.com/packages/AzurePublicLBUpgrade/4.0).
## <a name="use-the-script"></a>Utiliser le script

Vous disposez de deux options selon vos préférences et votre configuration de l’environnement PowerShell local :

* Si vous n’avez pas installé les modules Azure Az ou si vous êtes prêt à désinstaller les modules Azure Az, la meilleure option consiste à utiliser l’option `Install-Script` pour exécuter le script.
* Si vous devez conserver les modules Azure Az, la meilleure solution qui s’offre à vous consiste à télécharger le script et à l’exécuter directement.

Pour déterminer si vous avez installé les modules Azure Az, exécutez `Get-InstalledModule -Name az`. Si vous ne voyez aucun module Az installé, vous pouvez utiliser la méthode `Install-Script`.

### <a name="install-using-the-install-script-method"></a>Installer à l’aide de la méthode Install-Script

Pour pouvoir utiliser cette option, les modules Azure Az ne doivent pas être installés sur votre ordinateur. S’ils sont installés, la commande suivante affiche une erreur. Vous pouvez désinstaller les modules Azure Az ou utiliser l’autre option pour télécharger le script manuellement et l’exécuter.
  
Exécutez le script avec la commande suivante :

`Install-Script -Name AzurePublicLBUpgrade`

Cette commande installe également les modules Az requis.  

### <a name="install-using-the-script-directly"></a>Installer en utilisant directement le script

Si certains modules Azure Az sont installés et ne peuvent pas être désinstallés (ou si vous ne souhaitez pas les désinstaller), vous pouvez télécharger manuellement le script en utilisant l’onglet **Téléchargement manuel** dans le lien de téléchargement du script. Le script est téléchargé sous forme de fichier nupkg brut. Pour installer le script à partir de ce fichier nupkg, consultez [Téléchargement manuel de package](/powershell/scripting/gallery/how-to/working-with-packages/manual-download).

Pour exécuter le script :

1. Utilisez `Connect-AzAccount` pour vous connecter à Azure.

1. Utilisez `Import-Module Az` pour importer les modules Az.

1. Examinez les paramètres requis :

   * **oldRgName : [Chaîne] : Obligatoire** – Il s’agit du groupe de ressources pour votre instance Basic Load Balancer existante que vous souhaitez mettre à niveau. Pour trouver cette valeur de chaîne, accédez au Portail Azure, sélectionnez votre source Basic Load Balancer, puis cliquez sur la **Vue d’ensemble** de l’équilibreur de charge. Le groupe de ressources figure dans cette page.
   * **oldLBName : [Chaîne] : Obligatoire** – Il s’agit du nom de l’instance Basic Load Balancer existante que vous souhaitez mettre à niveau. 
   * **newLBName : [Chaîne] : Obligatoire** – Il s’agit du nom de l’instance Standard Load Balancer à créer.
1. Exécutez le script en utilisant les paramètres appropriés. Cette opération peut prendre entre cinq et sept minutes.

    **Exemple**

   ```azurepowershell
   AzurePublicLBUpgrade.ps1 -oldRgName "test_publicUpgrade_rg" -oldLBName "LBForPublic" -newLbName "LBForUpgrade"
   ```

### <a name="create-an-outbound-rule-for-outbound-connection"></a>Créer une règle de trafic sortant pour une connexion sortante

Suivez les [instructions](./quickstart-load-balancer-standard-public-powershell.md#create-outbound-rule-configuration) pour créer une règle de trafic sortant afin de pouvoir
* définir la NAT de trafic sortant à partir de zéro ;
* mettre à l’échelle et adapter le comportement de la NAT de trafic sortant.

## <a name="common-questions"></a>Questions courantes

### <a name="are-there-any-limitations-with-the-azure-powershell-script-to-migrate-the-configuration-from-v1-to-v2"></a>Existe-t-il des restrictions avec le script Azure PowerShell pour migrer la configuration de la version v1 à v2 ?

Oui. Consultez [Mises en garde/Limitations](#caveatslimitations).

### <a name="how-long-does-the-upgrade-take"></a>Combien de temps dure la mise à niveau ?

Il faut généralement environ 5 à 10 minutes pour que le script se termine, ce qui peut prendre plus de temps en fonction de la complexité de la configuration de votre équilibreur de charge. Par conséquent, gardez le temps d’arrêt à l’esprit et planifiez le basculement si nécessaire.

### <a name="does-the-azure-powershell-script-also-switch-over-the-traffic-from-my-basic-load-balancer-to-the-newly-created-standard-load-balancer"></a>Le script Azure PowerShell bascule-t-il également le trafic de mon instance Basic Load Balancer nouvellement créée ?

Oui. Le script Azure PowerShell met non seulement à niveau l’IP publique et copie la configuration de Basic Load Balancer vers Standard Load Balancer, mais migre également la machine virtuelle derrière l’équilibreur de charge public standard nouvellement créée. 

### <a name="i-ran-into-some-issues-with-using-this-script-how-can-i-get-help"></a>J’ai rencontré des problèmes en utilisant ce script. Comment obtenir de l’aide ?
  
Vous pouvez envoyer un e-mail à slbupgradesupport@microsoft.com, ouvrir un dossier de support auprès du support Azure ou les deux.

## <a name="next-steps"></a>Étapes suivantes

[Présentation de la référence Standard d’Azure Load Balancer (préversion)](load-balancer-overview.md)
