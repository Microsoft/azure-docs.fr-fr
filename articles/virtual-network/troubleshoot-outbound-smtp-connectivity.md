---
title: Résoudre les problèmes de connectivité SMTP sortante dans Azure | Microsoft Docs
description: Découvrez la méthode recommandée pour l’envoi des e-mails et la manière de résoudre les problèmes de connectivité SMTP sortante dans Azure.
services: virtual-network
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/20/2018
ms.author: genli
ms.openlocfilehash: d2e5996da5a1fe3f5b154d57ee509f25e54e30ac
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96862358"
---
# <a name="troubleshoot-outbound-smtp-connectivity-issues-in-azure"></a>Résoudre les problèmes de connectivité SMTP sortante dans Azure

À partir du 15 novembre 2017, les e-mails sortants envoyés directement à des domaines externes (comme outlook.com et gmail.com) à partir d’une machine virtuelle sont accessibles uniquement à certains types d’abonnements dans Microsoft Azure. Les connexions SMTP sortantes qui utilisent le port TCP 25 ont été bloquées. (Le port 25 est principalement utilisé pour la remise d’e-mails non authentifiés.)

Ce changement de comportement s’applique uniquement aux nouveaux abonnements et aux nouveaux déploiements depuis le 15 novembre 2017.

## <a name="recommended-method-of-sending-email"></a>Méthode recommandée de l’envoi d’e-mails

Nous vous recommandons d’utiliser les services de relais SMTP authentifiés (qui se connectent généralement via le port TCP 587 ou 443, mais prennent en charge d’autres ports aussi) pour envoyer un e-mail à partir de machines virtuelles Azure ou d’Azure App Services. Ces services sont utilisés pour maintenir la réputation de l’IP ou du domaine afin de minimiser le risque que des fournisseurs de messagerie tiers ne rejettent le message. Ces services de relais SMTP incluent, mais sans s’y limiter, [SendGrid](https://sendgrid.com/partners/azure/). Il est également possible que vous ayez un service de relais SMTP sécurisé en cours d’exécution en local que vous pouvez utiliser.

L’utilisation de ces services de remise d’e-mails n’est pas limitée dans Azure, quel que soit le type d’abonnement.

## <a name="enterprise-agreement"></a>Contrat Entreprise

Pour les utilisateurs Contrat Entreprise Azure, il n’y a aucune modification de la capacité technique à envoyer un e-mail sans utiliser un relais authentifié. Les utilisateurs Contrat Entreprise nouveaux et existants peuvent essayer la remise d’e-mails sortants à partir de machines virtuelles Azure directement à des fournisseurs de messagerie externes sans aucune restriction de la plateforme Azure. Bien qu’il n’est pas garanti que les fournisseurs de messagerie accepteront les e-mails entrants de n’importe quel utilisateur donné, les tentatives de remise ne seront pas bloquées par la plateforme Azure pour les machines virtuelles au sein d’abonnements Contrat Entreprise. Vous devrez travailler directement avec les fournisseurs de messagerie pour résoudre les problèmes de remise de message ou de filtrage du courrier indésirable qui impliquent des fournisseurs spécifiques.

## <a name="pay-as-you-go"></a>Pay-As-You-Go

Si vous vous êtes inscrit avant le 15 novembre 2017 à l’abonnement Paiement à l’utilisation, il n’y aura aucune modification de la capacité technique à essayer la remise d’e-mails sortants. Vous pourrez toujours essayer la remise d’e-mails sortants à partir de machines virtuelles Azure directement depuis ces abonnements à des fournisseurs de messagerie externes sans aucune restriction de la plateforme Azure. Là encore, il n’est pas garanti que les fournisseurs de messagerie accepteront les e-mails entrants de n’importe quel utilisateur donné, et les utilisateurs devront travailler directement avec les fournisseurs de messagerie pour résoudre les problèmes de remise de message ou de filtrage du courrier indésirable qui impliquent des fournisseurs spécifiques.

Pour les abonnements Paiement à l’utilisation créés après le 15 novembre 2017, des restrictions techniques bloquent les e-mails envoyés directement à partir de machines virtuelles se trouvant dans ces abonnements. Si vous souhaitez pouvoir envoyer des e-mails à partir de machines virtuelles Azure directement à des fournisseurs de messagerie externes (sans utiliser de relais SMTP authentifié), et que vous avez un compte valide et conforme avec un historique des paiements, vous pouvez effectuer une demande de suppression de la restriction dans la section **Connectivité** du panneau **Diagnostiquer et résoudre** pour une ressource Réseau virtuel Microsoft Azure dans le portail Azure. S’il remplit les conditions requises, votre abonnement sera activé ou vous recevrez des instructions sur les étapes suivantes. 

Lorsqu’un abonnement Paiement à l’utilisation est exempté et que les machines virtuelles ont été arrêtées et démarrées dans le portail Azure, toutes les machines virtuelles de cet abonnement sont exemptées. L’exemption s’applique uniquement à l’abonnement demandé et au trafic de machine virtuelle acheminé directement vers Internet.

> [!NOTE]
> Microsoft se réserve le droit de révoquer cette exemption s’il s’avère qu’une violation des conditions d’utilisation du service s’est produite.

## <a name="msdn-azure-pass-azure-in-open-education-azure-for-students-vistual-studio-and-free-trial"></a>MSDN, Pass Azure, Azure dans Open, Azure for Education, Microsoft Azure for Students, Visual Studio et Essai gratuit

Si vous avez créé un abonnement MSDN, Pass Azure, Azure dans Open, Azure for Education, Microsoft Azure for Students, Essai gratuit ou tout abonnement Visual Studio après le 15 novembre 2017, des restrictions techniques bloqueront les e-mails envoyés à partir de machines virtuelles dans ces abonnements directement aux fournisseurs de messagerie. Les restrictions sont là pour éviter les abus. Aucune demande pour supprimer cette restriction ne sera accordée.

Si vous utilisez ces types d’abonnements, vous êtes invité à utiliser des services de relais SMTP, comme indiqué plus haut dans cet article, ou à changer votre type d’abonnement.

## <a name="cloud-service-provider-csp"></a>Fournisseur de services cloud (CSP)

Si vous utilisez des ressources Azure par le biais d’un CSP, vous pouvez effectuer une demande de suppression de la restriction dans la section **Connectivité** du panneau **Diagnostiquer et résoudre** pour une ressource Réseau virtuel Microsoft Azure dans le portail Azure. S’il remplit les conditions requises, votre abonnement sera activé ou vous recevrez des instructions sur les étapes suivantes.

## <a name="microsoft-partner-network-mpn-bizspark-plus-or-azure-sponsorship"></a>Microsoft Partner Network (MPN), BizSpark Plus ou Azure Sponsorship

Pour les abonnements Microsoft Partner Network (MPN), BizSpark Plus ou Azure Sponsorship créés après le 15 novembre 2017, des restrictions techniques bloquent les e-mails envoyés directement à partir de machines virtuelles dans le cadre de ces abonnements. Si vous souhaitez pouvoir envoyer des e-mails à partir de machines virtuelles Azure directement à des fournisseurs de messagerie externes (sans utiliser de relais SMTP authentifié), vous pouvez en faire la demande en ouvrant une demande de support en utilisant le type de problème suivant : **Technique** > **Réseau virtuel** > **Connectivité** > **Impossible d’envoyer un e-mail (SMTP/Port 25)** . N’oubliez pas d’ajouter plus d’informations sur la raison pour laquelle votre déploiement doit envoyer des e-mails directement aux fournisseurs de messagerie au lieu d’utiliser un relais authentifié. Les demandes seront revues et approuvées à la discrétion de Microsoft. Les demandes peuvent être accordées uniquement après l’exécution de contrôles anti-fraude supplémentaires. 

Lorsqu’un abonnement est exempté et que les machines virtuelles ont été arrêtées et démarrées dans le portail Azure, toutes les machines virtuelles de cet abonnement sont exemptées. L’exemption s’applique uniquement à l’abonnement demandé et au trafic de machine virtuelle acheminé directement vers Internet.

## <a name="restrictions-and-limitations"></a>Limitations et restrictions

- Le routage du trafic du port 25 par le biais de services Azure PaaS comme [Pare-feu Azure](https://azure.microsoft.com/services/azure-firewall/) n’est pas pris en charge.

## <a name="need-help-contact-support"></a>Vous avez besoin d’aide ? Contacter le support technique

Si vous avez besoin d’aide, [contactez le support technique](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour obtenir une prise en charge rapide de votre problème en utilisant le type de problème suivant : **Technique** > **Réseau virtuel** > **Connectivité** > **Impossible d’envoyer un e-mail (SMTP/Port 25)** .
