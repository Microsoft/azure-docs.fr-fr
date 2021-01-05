---
title: Découvrez les options de support d’Azure Service Fabric
description: Versions de clusters Azure Service Fabric prises en charge et liens pour soumettre des tickets de support
author: pkcsf
ms.topic: troubleshooting
ms.date: 8/24/2018
ms.author: pkc
ms.openlocfilehash: 977cd79de629670cef526f072340f8897fa6446e
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92316496"
---
# <a name="azure-service-fabric-support-options"></a>Options de support d’Azure Service Fabric

Nous avons créé un certain nombre d’options de demande de support pour répondre aux besoins de gestion de vos clusters et de vos charges de travail d’application Service Fabric. Selon l’urgence du support nécessaire et la gravité du problème, vous pouvez choisir l’option qui vous convient.

## <a name="report-production-issues-or-request-paid-support-for-azure"></a>Signaler des problèmes de production ou demander un support payant pour Azure

Pour signaler des problèmes relatifs à votre cluster Service Fabric s’exécutant sur Azure, ouvrez un ticket de support [sur le portail Azure](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview) ou le [portail de support Microsoft](https://support.microsoft.com/oas/default.aspx?prid=16146).

Pour en savoir plus :
 
- [Support Microsoft pour Azure](https://azure.microsoft.com/support/plans/?b=16.44).
- [Support Premier Microsoft](https://support.microsoft.com/en-us/premier).

> [!Note]
> Les clusters s’exécutant sur un niveau de fiabilité bronze ou un cluster à nœud unique vous permettent uniquement d’exécuter des charges de travail de test. Si vous rencontrez des problèmes avec un cluster en cours d’exécution sur la fiabilité bronze ou un cluster à nœud unique, l’équipe du support technique de Microsoft vous aidera à atténuer le problème, mais elle n’effectuera pas d’analyse de la cause racine. Pour plus d’informations, reportez-vous à [Caractéristiques de fiabilité du cluster](./service-fabric-cluster-capacity.md#reliability-characteristics-of-the-cluster).
>
> Pour de plus amples renseignements sur les conditions exigées lors de la mise en service d’un cluster en production, reportez-vous à la [check-list de préparation à la production](./service-fabric-production-readiness-checklist.md).

<a id="getlivesitesupportonprem"></a>

## <a name="report-production-issues-or-request-paid-support-for-standalone-service-fabric-clusters"></a>Signaler des problèmes de production ou demander un support payant pour des clusters Service Fabric autonomes

Pour signaler des problèmes relatifs à votre cluster Service Fabric s’exécutant localement ou sur d’autres clouds, vous pouvez ouvrir un ticket de support professionnel sur le [portail de support Microsoft](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview).

Pour en savoir plus :

- [Support professionnel Microsoft local](https://support.microsoft.com/en-us/gp/offerprophone?wa=wsignin1.0).
- [Support Premier Microsoft](https://support.microsoft.com/en-us/premier).

## <a name="report-azure-service-fabric-issues"></a>Signaler des problèmes avec Azure Service Fabric

Nous avons défini un référentiel GitHub pour signaler les problèmes de Service Fabric.  De plus, nous surveillons activement les forums suivants.

### <a name="github-repo"></a>Référentiel GitHub 

Signalez les problèmes concernant Azure Service Fabric sur la [page Service Fabric de GitHub](https://github.com/microsoft/service-fabric/issues). Ce référentiel est destiné au signalement et au suivi des problèmes ainsi qu’aux demandes de petites fonctionnalités en relation avec Azure Service Fabric. **N’utilisez pas ce moyen pour signaler des problèmes de site en ligne** .

### <a name="stackoverflow-and-msdn-forums"></a>Forums StackOverflow et MSDN

L’[étiquette Service Fabric sur StackOverflow][stackoverflow] et le [forum Service Fabric sur MSDN][msdn-forum] sont recommandés pour poser des questions générales sur le fonctionnement de la plateforme et sur la manière dont vous pouvez l’utiliser pour accomplir certaines tâches.

### <a name="azure-feedback-forum"></a>Forum de commentaires Azure

Le [forum de commentaires Azure pour Service Fabric][uservoice-forum] est le meilleur endroit pour soumettre des idées de fonctionnalités de produit importantes. Nous examinons les demandes les plus populaires et les prenons en compte dans notre planification à moyen et à long terme. Nous vous encourageons à chercher l’appui de la communauté pour vos suggestions.

## <a name="service-fabric-preview-versions---unsupported-for-production-use"></a>Préversions de Service Fabric – non prises en charge en production

Nous créons parfois des versions préliminaires spéciales contenant des modifications significatives de fonctionnalités pour lesquelles nous aimerions avoir un retour d’information rapide. Vous devez utiliser ces versions préliminaires uniquement dans des environnements de test isolés qui ne traitent pas de charges de travail de production. Votre cluster de production doit toujours exécuter une version de Service Fabric prise en charge et stable. Nous n’offrons aucune option de support technique payant pour ces versions préliminaires.

Les versions préliminaires commencent toujours par un numéro de version majeure et mineure : 255. Par exemple, la version 255.255.5703.949 de Service Fabric est disponible en préversion et ne doit être utilisée que dans les clusters de test. Ces versions préliminaires sont également annoncées sur le [blog de l’équipe Service Fabric](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric) et s’accompagnent d’informations sur les fonctionnalités incluses. Utilisez l’une des options répertoriées dans la rubrique [Signaler des problèmes Azure Service Fabric](#report-azure-service-fabric-issues) pour poser des questions ou laisser un commentaire.

## <a name="next-steps"></a>Étapes suivantes

[Versions de Service Fabric prises en charge](service-fabric-versions.md)

<!--references-->
[Microsoft Q&A question page]: /answers/topics/azure-service-fabric.html
[stackoverflow]: https://stackoverflow.com/questions/tagged/azure-service-fabric
[uservoice-forum]: https://feedback.azure.com/forums/293901-service-fabric
[acom-docs]: ./index.yml
[sample-repos]: /samples/browse/?products=azure
[msdn-forum]: https://social.msdn.microsoft.com/forums/azure/en-US/home?category=windowsazureplatform