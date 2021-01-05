---
title: Résoudre les problèmes de mise à l’échelle automatique avec Virtual Machine Scale Sets
description: Dépanner la mise à l’échelle automatique avec des jeux de mise à l’échelle de machine virtuelle Découvrez les problèmes couramment rencontrés et apprenez à les résoudre.
author: avirishuv
ms.author: avverma
ms.topic: troubleshooting
ms.service: virtual-machine-scale-sets
ms.subservice: autoscale
ms.date: 06/25/2020
ms.reviwer: jushiman
ms.custom: avverma
ms.openlocfilehash: 11302c301bee466f678d544d0c4838c39cec9c8e
ms.sourcegitcommit: d2222681e14700bdd65baef97de223fa91c22c55
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2020
ms.locfileid: "91818538"
---
# <a name="troubleshooting-autoscale-with-virtual-machine-scale-sets"></a>Dépannage de la mise à l’échelle automatique avec des jeux de mise à l’échelle de machine virtuelle
**Problème** : vous avez créé une infrastructure de mise à l’échelle automatique dans Azure Resource Manager à l’aide de groupes de machines virtuelles identiques (par exemple en déployant un modèle comme https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-bottle-autoscale ), vos règles de mise à l’échelle sont définies et fonctionnent très bien, sauf que, quelle que soit la charge placée sur les machines virtuelles, elle n’est pas mise à l’échelle automatiquement.

## <a name="troubleshooting-steps"></a>Étapes de dépannage
Parmi les éléments à prendre en considération :

* De combien de processeurs virtuels chaque machine virtuelle dispose-t-elle et chargez-vous chaque processeur virtuel ?
  L’exemple de modèle Azure QuickStart précédent a un script do_work.php, qui charge un seul processeur virtuel. Si vous utilisez une machine virtuelle plus volumineuse qu’une machine virtuelle à un seul processeur virtuel telle qu’une machine Standard_A1 ou D1, vous devez exécuter cette charge plusieurs fois. Vérifiez le nombre de processeurs virtuels de vos machines virtuelles en consultant [Tailles des machines virtuelles Windows dans Azure](../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
* Combien de machines virtuelles sont dans le groupe de machines virtuelles identique et travaillez-vous sur chacune d’elles ?
  
    La mise à l’échelle augmente uniquement lorsque l’utilisation moyenne du processeur sur **toutes** les machines virtuelles d’un groupe identique dépasse la valeur seuil, sur la période définie dans les règles de mise à l’échelle automatique.
* Vous avez manqué un événement de mise à l’échelle ?
  
    Consultez les journaux d’audit dans le portail Azure pour les événements de mise à l’échelle. Peut-être qu’une mise à l’échelle vers le haut ou vers le bas a été ignorée. Vous pouvez filtrer sur « Mise à l’échelle ».
  
    ![Journaux d’audit][audit]
* Les seuils d’augmentation et de diminution de la taille des instances sont-ils suffisamment différents ?
  
    Supposons que vous définissez une règle pour effectuer un scale-out lorsque l’utilisation moyenne du processeur est supérieure à 50 % pendant cinq minutes et une règle pour effectuer un scale-in lorsque l’utilisation moyenne du processeur est inférieure à 50 %. Ce paramètre entraîne un problème d’oscillation lorsque l’utilisation du processeur est proche de ce seuil, avec des actions de mise à l’échelle entraînant constamment l’augmentation et la diminution de la taille du groupe. De ce fait, le service de mise à l’échelle automatique tente d’empêcher l’oscillation, ce qui peut se manifester par une absence de mise à l’échelle. Par conséquent, assurez-vous que les seuils d’augmentation et de diminution de la taille des instances sont suffisamment différents pour laisser une marge lors de la mise à l’échelle.
* Avez-vous écrit votre propre modèle JSON ?
  
    Il est facile de commettre des erreurs. Commencez avec un modèle comme celui ci-dessus, dont l’efficacité a été prouvée, et apportez de petites modifications incrémentielles. 
* Pouvez vous effectuer manuellement un scale-in ou un scale-out ?
  
    Essayez de redéployer la ressource du groupe de machines virtuelles identiques avec un paramètre de capacité différent pour modifier le nombre de machines virtuelles manuellement. Un exemple de modèle se trouve ici : https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-scale-existing – Vous devrez peut-être modifier le modèle pour vous assurer qu’il a la même taille de machine que votre groupe identique. Si vous pouvez modifier le nombre de machines virtuelles manuellement, vous savez que le problème est lié à la mise à l’échelle automatique.
* Consultez vos ressources Microsoft.Compute/virtualMachineScaleSet et Microsoft.Insights dans [l’explorateur de ressources Azure](https://resources.azure.com/)
  
    Azure Resource Explorer est un outil de dépannage indispensable qui vous indique l’état de vos ressources Azure Resource Manager. Cliquez sur votre abonnement et examinez le groupe de ressources sur lequel vous effectuez un dépannage. Sous le fournisseur de ressources de calcul, consultez le groupe de machines virtuelles identiques que vous avez créé et consultez la vue d’instance, qui vous indique l’état d’un déploiement. Vérifiez également la vue d’instance des machines virtuelles dans le groupe de machines virtuelles identiques. Ensuite, accédez au fournisseur de ressources Microsoft.Insights et vérifiez que les règles de mise à l’échelle automatique sont correctes.
* L’extension de diagnostic fonctionne-t-elle et génère-t-elle des données de performance ?
  
    **Mise à jour :** la mise à l’échelle automatique Azure a été améliorée pour utiliser un pipeline de métriques basées sur l’hôte qui ne nécessite plus l’installation d’une extension de diagnostic. Les paragraphes suivants ne s’appliquent plus si vous créez une application de mise à l’échelle automatique en utilisant le nouveau pipeline. Un exemple de modèles Azure convertis pour utiliser le pipeline hôte est disponible ici : https://github.com/Azure/azure-quickstart-templates/tree/master/201-vmss-bottle-autoscale. 
  
    Le recours aux métriques basées sur l’hôte pour la mise à l’échelle automatique est préférable pour les raisons suivantes :
  
  * Moins de composants mobiles (en l’absence d’une extension de diagnostics) doivent être installés.
  * Les modèles sont plus simples. Ajoutez simplement des règles de mise à l’échelle automatique Insights à un modèle de groupe identique existant.
  * Reporting plus fiable et lancement plus rapide de nouvelles machines virtuelles.
    
    Les seules raisons qui justifient de continuer à utiliser une extension de diagnostic sont la nécessité d’une mise à l’échelle/d’un reporting de diagnostic de mémoire. Les métriques basées sur l’hôte ne génèrent aucun rapport sur la mémoire.
    
    En sachant cela, poursuivez uniquement la lecture de cet article si vous utilisez des extensions de diagnostic pour votre mise à l’échelle automatique.
    
    La mise à l’échelle dans Azure Resource Manager peut fonctionner (mais n’est plus obligatoire) au moyen d’une extension de machine virtuelle appelée « Extension de diagnostics ». Elle envoie des données de performance à un compte de stockage que vous définissez dans le modèle. Ces données sont ensuite regroupées par le service Azure Monitor.
    
    Si le service Insights ne peut pas lire les données à partir des machines virtuelles, il doit vous envoyer un e-mail. Par exemple, vous obtenez un e-mail quand les machines virtuelles sont en panne. Veillez à consulter vos e-mails, à l’adresse e-mail que vous avez spécifiée lorsque vous avez créé votre compte Azure.
    
    Vous pouvez également examiner vous-même les données. Examinez le compte de stockage Azure à l’aide d’un explorateur de cloud. Par exemple, à l’aide de [Visual Studio Cloud Explorer](https://visualstudiogallery.msdn.microsoft.com/aaef6e67-4d99-40bc-aacf-662237db85a2), connectez-vous, puis choisissez l’abonnement Azure que vous utilisez. Ensuite, examinez le nom du compte de stockage des diagnostics référencé dans la définition de l’extension de diagnostic dans votre modèle de déploiement.
    
    ![Cloud Explorer][explorer]
    
    Apparaît un ensemble de tables où sont stockées les données de chaque machine virtuelle. Prenons Linux et la mesure de processeur comme exemple. Examinez les lignes les plus récentes. Visual Studio Cloud Explorer prend en charge un langage de requête pour vous permettre d’exécuter une requête. Par exemple, vous pouvez exécuter une requête pour « Timestamp gt datetime'2016-02-02T21:20:00Z' » pour vous assurer d’obtenir les événements les plus récents. Le fuseau horaire correspond à l’heure UTC. Les données qui figurent ici correspondent-elles aux règles de mise à l’échelle que vous avez configurées ? Dans l’exemple suivant, l’utilisation du processeur de la machine 20 a commencé à augmenter jusqu’à 100 % au cours des cinq dernières minutes.
    
    ![Tables de stockage][tables]
    
    Si les données ne sont pas visibles, cela implique que le problème provient de l’extension de diagnostic en cours d’exécution sur les machines virtuelles. Si les données sont présentes, cela implique un problème lié à vos règles de mise à l’échelle ou au service Insights. Vérifiez le [statut Azure](https://azure.microsoft.com/status/).
    
    Une fois que vous avez effectué ces étapes, si vous rencontrez toujours des problèmes de mise à l’échelle automatique, vous pouvez essayer les ressources suivantes : 
    * Lisez les forums à partir de la [page de questions Microsoft Q&A ](/answers/topics/azure-virtual-machines.html) ou de [Stack Overflow](https://stackoverflow.com/questions/tagged/azure). 
    * Enregistrez un appel au support. Soyez prêt à partager le modèle et une vue de vos données de performance.

[audit]: ./media/virtual-machine-scale-sets-troubleshoot/image3.png
[explorer]: ./media/virtual-machine-scale-sets-troubleshoot/image1.png
[tables]: ./media/virtual-machine-scale-sets-troubleshoot/image4.png
