---
title: Forum aux questions
description: Questions fréquentes (FAQ) sur le service Azure Spatial Anchors
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 11/20/2020
ms.topic: overview
ms.service: azure-spatial-anchors
ms.openlocfilehash: 01eb0ce83efa54366e027d35d9c3c3bbf86eff7b
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95487365"
---
# <a name="frequently-asked-questions-about-azure-spatial-anchors"></a>Questions fréquentes (FAQ) sur Azure Spatial Anchors

Azure Spatial Anchors est un service cloud managé et une plateforme de développement qui autorise des expériences de réalité mixte multi-utilisateurs sensibles à l’espace sur les appareils HoloLens, iOS et Android.

Pour plus d’informations, consultez [Vue d’ensemble d’Azure Spatial Anchors](overview.md).

## <a name="azure-spatial-anchors-product-faqs"></a>Questions fréquentes (FAQ) sur Azure Spatial Anchors

**Q : Quels appareils le service Azure Spatial Anchors prend en charge ?**

**R :** Azure Spatial Anchors permet aux développeurs de générer des applications sur HoloLens, sur des appareils iOS prenant en charge ARKit et sur des appareils Android prenant en charge ARCore ; dans le cas d’iOS et d’Android, cela englobe les téléphones et les tablettes.

**Q : Dois-je être connecté au cloud pour pouvoir utiliser Azure Spatial Anchors ?**

**R :** Pour l’heure, Azure Spatial Anchors nécessite une connexion réseau à Internet. Nous vous encourageons à nous communiquer vos retours d’expérience sur notre [site de commentaires](https://feedback.azure.com/forums/919252-azure-spatial-anchors).

**Q : Quelles sont les conditions de connectivité d’Azure Spatial Anchors ?**

**R :** Azure Spatial Anchors fonctionne avec des connexions haut débit Wi-Fi et mobiles.

**Q : Avec quelle précision Azure Spatial Anchors localise les ancres ?**

**R :** De nombreux facteurs peuvent nuire à la précision de la localisation des ancres, notamment les conditions d’éclairage, la présence de certains objets dans l’environnement et même la surface sur laquelle l’ancre est placée. Pour déterminer si la précision répond à vos besoins, testez les ancres dans des environnements similaires à ceux où vous prévoyez de les utiliser. Si vous constatez que la précision n’est pas optimale dans certains environnements, consultez [Journalisation et diagnostics dans Azure Spatial Anchors](./concepts/logging-diagnostics.md).

**Q : Combien de temps faut-il pour créer et localiser des ancres ?**

**R :** Le temps nécessaire à la création et à la localisation des ancres dépend de nombreux facteurs, notamment de la connexion réseau, du temps de traitement et de chargement de l’appareil et des spécificités de l’environnement. D’après les indications de nos clients qui créent des applications dans divers secteurs d’activité (industrie, distribution et jeux vidéo), le service offre une expérience utilisateur remarquable dans leurs scénarios.

## <a name="privacy-faq"></a>Questions fréquentes sur la confidentialité

**Q : Quand mon application place une ancre spatiale quelque part, est-ce que toutes les applications y ont accès ?**

**R :** Les ancres sont isolées par le compte Azure. Seules les applications qui ont accès à votre compte peuvent accéder aux ancres qui en dépendent.

**Q : Comment le service Azure Spatial Anchors stocke-t-il les données ?**

**R :** Toutes les données sont stockées chiffrées à l’aide d’une clé de chiffrement de données gérée par Microsoft et sont toutes stockées au niveau régional pour chacune des ressources.

**Q : Quelles informations d’environnement sont transmises et stockées dans le service quand Azure Spatial Anchors est utilisé ? Les images de l’environnement sont-elles transmises et stockées ?**

**R** : Quand des ancres sont créées ou localisées, les images de l’environnement sont traitées sur l’appareil dans un format dérivé. Ce format dérivé est transmis et stocké dans le service.

Pour illustrer cela, voici une image d’environnement et le nuage de points disséminés qui en est dérivé. Le nuage de points est la représentation géométrique de l’environnement qui est transmise et stockée dans le service. Pour chaque point présent dans le nuage de points disséminés, nous transmettons et stockons un hachage des caractéristiques visuelles de ce point. Le hachage est dérivé de données de pixels, mais ne les contient pas.

Azure Spatial Anchors respecte les [conditions du contrat de service Azure](https://go.microsoft.com/fwLink/?LinkID=522330&amp;amp;clcid=0x9) et la [déclaration de confidentialité Microsoft](https://go.microsoft.com/fwlink/?LinkId=521839&amp;clcid=0x409).

![Un environnement et son nuage de points disséminés qui en est dérivé](./media/sparse-point-cloud.png)
*Figure 1 : Un environnement et son nuage de points disséminés qui en est dérivé*

**Q : Est-il possible d’envoyer des informations de diagnostic à Microsoft ?**

**R** : Oui. Azure Spatial Anchors propose un mode diagnostics auquel les développeurs peuvent souscrire via l’API Azure Spatial Anchors. Ce mode s’avère utile si, par exemple, vous ne parvenez pas à créer et à localiser des ancres dans un environnement de manière prévisible. Nous pouvons dans ce cas vous demander un rapport de diagnostics contenant des informations utiles au débogage. Pour plus d’informations, consultez [Journalisation et diagnostics dans Azure Spatial Anchors](./concepts/logging-diagnostics.md).

## <a name="availability-and-pricing-faqs"></a>Questions fréquences sur la disponibilité et les tarifs

**Q : Fournissez-vous un contrat SLA ?**

**R :** Comme pour tous les services Azure, nous visons un taux de disponibilité supérieur à 99,9 %. 

**Q : Puis-je publier mes applications dans des app stores en utilisant Azure Spatial Anchors ? Puis-je utiliser Azure Spatial Anchors pour des scénarios de production critiques ?**

**R :** Oui, Azure Spatial Anchors est en disponibilité générale et est assorti d’un contrat de niveau de service Azure standard. Nous vous invitons à développer des applications pour vos déploiements de production et à [nous faire part de vos commentaires](https://feedback.azure.com/forums/919252-azure-spatial-anchors) à propos du produit.

**Q : Existe-t-il des limites ?**

**R** : Oui, il existe des limites.  Cependant, vous ne devriez pas les atteindre dans le cadre du développement et du test d’une application normale. Pour les déploiements de production, nous sommes prêts à répondre aux exigences élevées de nos clients. [Contactez-nous](mailto:azuremrs@microsoft.com) pour en discuter. 

**Q : Dans quelles régions Azure Spatial Anchors est disponible ?**

**R :** Azure Spatial Anchors est actuellement disponible dans les régions USA Ouest 2, USA Est, USA Est 2, USA Centre Sud, Europe Ouest, Europe Nord, Royaume-Uni Sud et Australie Est. Des régions supplémentaires seront également disponibles par la suite.

Cela signifie que les ressources de calcul et de stockage exploitées par ce service résident dans ces régions. Ceci dit, il n’existe aucune restriction quant à la situation géographique de vos clients. 

**Q : Le service Azure Spatial Anchors est-il payant ?**

**R :** Vous trouverez le détail des prix dans notre [page de tarification](https://azure.microsoft.com/pricing/details/spatial-anchors/).

## <a name="technical-faqs"></a>Questions fréquentes d’ordre technique

**Q : Comment le service Azure Spatial Anchors fonctionne ?**

**R :** Azure Spatial Anchors s’appuie sur des dispositifs de suivi de réalité mixte/augmentée. Ces dispositifs perçoivent l’environnement au moyen de caméras et suivent l’appareil avec « 6 degrés de liberté » (6DoF) à mesure qu’il se déplace dans l’espace.

Avec un dispositif de suivi 6DoF en guise de bloc de construction, Azure Spatial Anchors vous permet de désigner certains points d’intérêt dans votre environnement réel comme points d’« ancrage ». Par exemple, vous pouvez utiliser une ancre pour afficher le contenu à un emplacement spécifique dans le monde réel.

Quand vous créez une ancre, le SDK client capture les informations d’environnement autour de ce point et les transmet au service. Si un autre appareil recherche l’ancre dans le même espace, des informations similaires sont alors transmises au service. Ces données sont comparées aux données d’environnement stockées précédemment. La position de l’ancre par rapport à l’appareil est ensuite renvoyée pour être utilisée dans l’application.

**Q : Comment Azure Spatial Anchors s’intègre avec ARKit et ARCore sur iOS et Android ?**

**R :** Azure Spatial Anchors exploite les fonctionnalités de suivi natives d’ARKit et ARCore. Par ailleurs, nos kits SDK pour iOS et Android offrent des fonctionnalités comme la conservation des ancres dans un service cloud managé, ce qui permet à vos applications de relocaliser ces ancres en se connectant simplement au service.

**Q : Comment le service Azure Spatial Anchors s’intègre avec HoloLens ?**

**R :** Azure Spatial Anchors tire parti des fonctionnalités de suivi natives de HoloLens. Nous fournissons un SDK Azure Spatial Anchors pour créer des applications sur HoloLens. Le kit SDK s’intègre avec les fonctionnalités natives de HoloLens et offre des fonctionnalités supplémentaires. Ces fonctionnalités permettent notamment aux développeurs d’applications de conserver les ancres dans un service cloud managé, ce qui permet à vos applications de relocaliser ces ancres en se connectant au service.

**Q : Quelles plateformes et quels langages le service Azure Spatial Anchors prend en charge ?**

**R :** Les développeurs peuvent créer des applications avec Azure Spatial Anchors à l’aide d’outils et de frameworks adaptés à leur appareil :

- Unity sur HoloLens, iOS et Android
- Xamarin sur iOS et Android
- Swift ou Objective-C sur iOS
- Java ou Android NDK sur Android
- C++/WinRT sur HoloLens

Lancez-vous dans le [développement ici](index.yml).

**Q : Le service fonctionne-t-il avec Unreal ?**

**R :** La prise en charge de Unreal sera envisagée prochainement.

**Q : Quels ports et protocoles Azure Spatial Anchors utilise-t-il ?**

**R :** Azure Spatial Anchors communique via le port TCP 443 à l’aide d’un protocole chiffré. Pour l’authentification, il utilise [Azure Active Directory](../active-directory/index.yml), qui communique à l’aide de HTTPS sur le port 443.