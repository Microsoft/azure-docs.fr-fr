---
title: Exigences en bande passante de RDP (Remote Desktop Protocol) pour Windows Virtual Desktop - Azure
titleSuffix: Azure
description: Comprendre les exigences en bande passante de RDP pour Windows Virtual Desktop.
author: gundarev
ms.topic: conceptual
ms.date: 11/16/2020
ms.author: denisgun
ms.openlocfilehash: c9ef72241ac62efd8555de59bb52949321364035
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96325177"
---
# <a name="remote-desktop-protocol-rdp-bandwidth-requirements"></a>Exigences en bande passante de RDP (Remote Desktop Protocol)

RDP (Remote Desktop Protocol) est une technologie sophistiquée qui utilise diverses techniques pour perfectionner la remise des graphiques à distance du serveur sur l’appareil client. En fonction du cas d’utilisation, de la disponibilité des ressources de calcul et de la bande passante réseau, RDP ajuste dynamiquement divers paramètres pour offrir la meilleure expérience utilisateur.

Le protocole RDP (Remote Desktop Protocol) multiplexe plusieurs canaux virtuels dynamiques (DVC) en un seul canal de données envoyé sur différents transports réseau. Il existe des DVC distincts pour les graphiques à distance, l’entrée, la redirection des appareils, l’impression, etc. Les partenaires WVD peuvent également implémenter leurs extensions qui utilisent des interfaces DVC.
La quantité de données envoyées via le protocole RDP dépend de l’activité de l’utilisateur. Par exemple, un utilisateur peut travailler sur du contenu textuel de base pendant la majeure partie de la session et consommer une bande passante minimale, mais générer ensuite l’impression d’un document de 200 pages sur l’imprimante locale. Ce travail d’impression utilise une quantité importante de bande passante réseau.

Lors de l’utilisation d’une session à distance, la bande passante disponible de votre réseau a un impact considérable sur la qualité de votre expérience. Différentes applications et résolutions d’affichage nécessitent des configurations réseau différentes. Dès lors, il est essentiel de vous assurer que votre réseau est configuré en fonction de vos besoins.

## <a name="estimating-bandwidth-utilization"></a>Estimation de l’utilisation de la bande passante

RDP utilise différents algorithmes de compression pour différents types de données. Le tableau ci-dessous décrit l’estimation des transferts de données :

| Type de données | Sens | Comment estimer |
|---|---|---|
| Graphiques à distance | Hôte de session vers client | [Consultez les instructions détaillées](#estimating-bandwidth-used-by-remote-graphics) |
|  Pulsations | Les deux directions | ~ 20 octets toutes les 5 secondes  |
| Entrée | Client vers hôte de session | La quantité de données est basée sur l’activité de l’utilisateur, inférieure à 100 octets pour la plupart des opérations  |
| transferts de fichiers | Les deux directions | Les transferts de fichiers utilisent la compression en bloc. Utilisez la compression .zip pour l’approximation |
| Impression | Hôte de session vers client | Le transfert du travail d’impression dépend du pilote et de l’utilisation de la compression en bloc, utilisez la compression .zip pour l’approximation |

Les besoins en bande passante dans d’autres scénarios peuvent varier en fonction de votre utilisation, par exemple :

* Audioconférence et vidéoconférence
* Communication en temps réel
* Streaming de vidéos 4K

## <a name="estimating-bandwidth-used-by-remote-graphics"></a>Estimation de la bande passante utilisée par les graphiques à distance

Il est difficile de prédire l’utilisation de la bande passante par le Bureau à distance. Les activités de l’utilisateur génèrent la majeure partie du trafic du Bureau à distance. Chaque utilisateur est unique et les différences dans leurs modes de travail peuvent changer de manière significative l’utilisation du réseau.

La meilleure façon de comprendre les besoins en bande passante consiste à surveiller les connexions utilisateur réelles. La surveillance peut être effectuée par les compteurs de performances intégrés ou par l’équipement réseau.

Toutefois, dans de nombreux cas, vous pouvez estimer l’utilisation du réseau en comprenant le fonctionnement du protocole RDP (Remote Desktop Protocol) et en analysant les modes de travail de vos utilisateurs.

Le protocole à distance livre les graphiques générés par le serveur à distance pour les afficher sur un moniteur local. Plus spécifiquement, il fournit l’image bitmap de bureau entièrement composée sur le serveur.
Alors que l’envoi d’une image bitmap de bureau semble être une tâche simple à première vue, elle nécessite une quantité importante de ressources. Par exemple, une image de bureau de 1080p dans sa forme non compressée a une taille d’environ 8 Mo. L’affichage de cette image sur l’écran connecté localement avec un faible taux d’actualisation de l’écran de 30 Hz nécessite une bande passante d’environ 237 Mo/s.

Pour réduire la quantité de données transférées sur le réseau, RDP utilise la combinaison de plusieurs techniques, notamment, mais sans s’y limiter :

* Optimisations de la fréquence d’images
* Classification du contenu de l’écran
* Codecs spécifiques au contenu
* Codage d’image progressif
* Mise en cache côté client

Pour mieux comprendre les graphiques à distance, prenez en compte les éléments suivants :

* Plus les graphiques sont riches, plus ils nécessitent de bande passante
  * Le texte, les éléments d’interface utilisateur de fenêtre et les zones de couleur unie consomment moins de bande passante que tout autre élément.
  * Les images naturelles sont les contributeurs les plus significatifs à l’utilisation de la bande passante. Mais la mise en cache côté client contribue à sa réduction.
* Seules les parties modifiées de l’écran sont transmises. Si aucune mise à jour n’est visible à l’écran, aucune mise à jour n’est envoyée.
* La lecture vidéo et le contenu à débit élevé sont essentiellement un diaporama d’images. RDP utilise dynamiquement les codecs vidéo appropriés pour les livrer avec une fréquence d’images proche de la valeur d’origine. Toutefois, ce sont toujours des graphiques, et il s’agit toujours du contributeur le plus significatif à l’utilisation de la bande passante.
* La durée d’inactivité du Bureau à distance signifie qu’il n’y a pas ou peu de mises à jour de l’écran. Par conséquent, l’utilisation du réseau est minime pendant les périodes d’inactivité.
* Lorsque la fenêtre client Bureau à distance est réduite, aucune mise à jour graphique n’est envoyée à partir de l’hôte de session.

Gardez à l’esprit que la contrainte imposée à votre réseau dépend à la fois de la fréquence d’images de sortie de la charge de travail de votre application et de votre résolution d’affichage. Si la fréquence d’images ou la résolution d’affichage augmente, le besoin en bande passante augmente également. Par exemple, une charge de travail légère avec un affichage haute résolution nécessite plus de bande passante qu’une charge de travail légère avec une résolution normale ou basse. Les besoins en bande passante varient en fonction de la résolution d’affichage.

Le tableau ci-dessous décrit l’estimation des données utilisées par les différents scénarios graphiques. Ces chiffres s’appliquent à une configuration à un seul moniteur avec la résolution 1920x1080, le mode graphique par défaut et le mode graphique H.264/AVC 444.

| Scénario | Mode par défaut | Mode H.264/AVC 444 | Thumbnail | Description du scénario |
|:---|---:|---:|---|---|
| Idle | 0,3 Kbits/s | 0,3 Kbits/s |:::image type="content" source="media/idle.png" alt-text="Capture d’écran de la connexion inactive":::| L’utilisateur a suspendu son travail et il n’y a aucune mise à jour active de l’écran |
| Microsoft Word | 100-150 Kbits/s | 200-300 Kbits/s |:::image type="content" source="media/word.gif" alt-text="Animation de Microsoft Word":::| L’utilisateur travaille activement dans Microsoft Word (il tape du texte, colle des graphiques et bascule entre des documents) |
| Microsoft Excel | 150-200 Kbits/s | 400-500 Kbits/s |:::image type="content" source="media/excel.gif" alt-text="Animation de Microsoft Excel":::| L’utilisateur travaille activement dans Microsoft Excel, plusieurs cellules avec des formules et des graphiques sont mises à jour simultanément |
| Microsoft PowerPoint | 4-4,5 Mbits/s | 1,6-1,8 Mbits/s |:::image type="content" source="media/powerpoint.gif" alt-text="Animation de Microsoft PowerPoint":::| L’utilisateur travaille activement dans Microsoft PowerPoint (il tape du texte, colle du contenu). L’utilisateur modifie également des graphiques enrichis et utilise des effets de transition entre des diapositives |
| Navigation sur le web | 6-6,5 Mbits/s | 0,9-1 Mbit/s |:::image type="content" source="media/web.gif" alt-text="Animation de navigation sur le web":::| L’utilisateur travaille activement dans un site web riche en graphiques qui contient plusieurs images statiques et animées. L’utilisateur fait défiler les pages horizontalement et verticalement |
| Galerie d’images | 3,3-3,6 Mbits/s | 0,7-0,8 Mbit/s |:::image type="content" source="media/image-gallery.gif" alt-text="Animation de la galerie d’images":::| L’utilisateur travaille activement dans l’application Galerie d’images. navigation, zoom, redimensionnement et rotation d’images |
| Lecture de vidéo | 8,5-9,5 Mbits/s | 2,5-2,8 Mbits/s |:::image type="content" source="media/video.gif" alt-text="Animation de lecture vidéo":::| L’utilisateur regarde une vidéo de 30 ips qui consomme la moitié de l’écran |
| Lecture vidéo en plein écran | 7,5-8,5 Mbits/s | 2,5-3,1 Mbits/s |:::image type="content" source="media/fullscreen-video.gif" alt-text="Animation de la lecture vidéo en plein écran":::| L’utilisateur regarde une vidéo de 30 ips en plein écran |

## <a name="dynamic-bandwidth-allocation"></a>Allocation dynamique de bande passante

RDP est un protocole moderne conçu pour s’adapter de manière dynamique aux conditions de réseau variables.
Au lieu d’utiliser les limites matérielles de l’utilisation de la bande passante, RDP utilise la détection de réseau continue qui surveille activement la bande passante réseau disponible et le temps d’aller-retour des paquets. En fonction des résultats, RDP sélectionne dynamiquement les options de codage graphique et alloue de la bande passante pour la redirection des appareils et autres canaux virtuels.  
Cette technologie permet à RDP d’utiliser le canal réseau complet lorsqu’il est disponible et de se retirer rapidement lorsqu’un autre élément requiert du réseau.
RDP détecte cela et ajuste la qualité de l’image, la fréquence d’images ou les algorithmes de compression si d’autres applications demandent du réseau.

## <a name="limit-network-bandwidth-use-with-throttle-rate"></a>Limiter l’utilisation de la bande passante réseau avec le taux d’accélération

Dans la plupart des scénarios, il n’est pas nécessaire de limiter l’utilisation de la bande passante, car la limitation peut affecter l’expérience utilisateur. Toutefois, sur les réseaux restreints, vous souhaiterez peut-être limiter l’utilisation du réseau. Un autre exemple est un réseau loué qui est facturé selon la quantité de trafic utilisée.

Dans ces cas, vous pouvez limiter le trafic réseau sortant RDP en spécifiant un taux d’accélération dans la stratégie QoS.

  >[!NOTE]
  > [Vérifiez que RDP Shortpath est activé](./shortpath.md) : le taux d’accélération et la limitation ne sont pas pris en charge pour le transport de connexion inverse.

### <a name="implement-throttle-rate-limiting-on-session-host-using-group-policy"></a>Implémenter la limitation du taux d’accélération sur l’hôte de session à l’aide de Stratégie de groupe

Vous pouvez utiliser QoS (Qualité de service) basée sur la stratégie dans Stratégie de groupe pour définir le taux d’accélération prédéfini.

Pour créer une stratégie QoS pour les hôtes de session joints à un domaine, connectez-vous d’abord à un ordinateur sur lequel la Gestion des stratégies de groupe a été installée. Ouvrez Gestion des stratégie de groupe (sélectionnez Démarrer, pointez sur Outils d’administration, puis sélectionnez Gestion des stratégie de groupe), puis procédez comme suit :

1. Dans Gestion des stratégies de groupe, localisez le conteneur dans lequel la nouvelle stratégie doit être créée. Par exemple, si tous vos ordinateurs hôtes de session sont situés dans une unité d’organisation nommée **Hôtes de session**, la nouvelle stratégie doit être créée dans l’unité d’organisation Hôtes de session.

2. Cliquez avec le bouton droit sur le conteneur approprié, puis sélectionnez **Créer un objet GPO dans ce domaine, et le lier ici**.

3. Dans la boîte de dialogue **Nouvel objet Stratégie de groupe**, tapez le nom du nouvel objet Stratégie de groupe dans la zone **Nom** et sélectionnez **OK**.

4. Cliquez avec le bouton droit sur la nouvelle stratégie, puis sélectionnez **Modifier**.

5. Dans l’Éditeur Gestion des stratégies de groupe, développez **Configuration de l’ordinateur**, développez **Paramètres Windows**, cliquez avec le bouton droit sur **QoS basée sur la stratégie**, puis sélectionnez **Créer une stratégie**.

6. Dans la boîte de dialogue **QoS basée sur la stratégie**, dans la page d’ouverture, tapez un nom pour la nouvelle stratégie dans la zone **Nom**. Sélectionnez **Spécifier le taux d’accélération du trafic sortant** et définissez la valeur requise, puis sélectionnez **Suivant**.

7. Sur la page suivante, sélectionnez **Uniquement les applications avec ce nom d’exécutable** et entrez le nom **svchost.exe**, puis sélectionnez **Suivant**. Ce paramètre indique à la stratégie de prioriser uniquement le trafic correspondant à partir du service Bureau à distance.

8. Sur la troisième page, assurez-vous que les options **Toute adresse IP source** et **Toute adresse IP de destination** sont sélectionnées. Sélectionnez **Suivant**. Ces deux paramètres garantissent que les paquets seront gérés quels que soient l’ordinateur (adresse IP) qui a envoyé les paquets et l’ordinateur (adresse IP) qui recevra les paquets.

9. Sur la quatrième page, sélectionnez **UDP** dans la liste déroulante **Sélectionner le protocole auquel cette stratégie QoS s’applique**.

10. Sous le titre **Spécifier le numéro de port source**, sélectionnez **À partir de ce port ou de cette plage source**. Dans la zone de texte correspondante, tapez **3390**. Sélectionnez **Terminer**.

Les nouvelles stratégies que vous avez créées ne prennent pas effet tant que la Stratégie de groupe n’a pas été actualisée sur vos ordinateurs hôtes de session. Bien que la Stratégie de groupe s’actualise régulièrement, vous pouvez forcer une actualisation immédiate en procédant comme suit :

1. Sur chaque hôte de session pour lequel vous souhaitez actualiser la Stratégie de groupe, ouvrez une invite de commandes en tant qu’administrateur (*Exécuter en tant qu’administrateur*).

2. À l’invite de commandes, tapez

   ```console
   gpupdate /force
   ```

### <a name="implement-throttle-rate-limiting-on-session-host-using-powershell"></a>Implémenter la limitation du taux d’accélération sur l’hôte de session à l’aide de PowerShell

Vous pouvez définir le taux d’accélération pour RDP Shortpath à l’aide de la cmdlet PowerShell ci-dessous :

```powershell
New-NetQosPolicy -Name "RDP Shortpath" -AppPathNameMatchCondition "svchost.exe" -IPProtocolMatchCondition UDP -IPSrcPortStartMatchCondition 3390 -IPSrcPortEndMatchCondition 3390 -DSCPAction 46 -NetworkProfile All
```

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur les besoins en bande passante pour Windows Virtual Desktop, consultez [Comprendre les exigences en bande passante de RDP (Remote Desktop Protocol) pour Windows Virtual Desktop](rdp-bandwidth.md).
* Pour en savoir plus sur la connectivité réseau de Windows Virtual Desktop, consultez [Comprendre la connectivité réseau de Windows Virtual Desktop](network-connectivity.md).
* Pour vous familiariser avec QoS (Qualité de service) pour Windows Virtual Desktop, consultez [Implémenter QoS (Qualité de service) pour Windows Virtual Desktop](rdp-quality-of-service-qos.md).
