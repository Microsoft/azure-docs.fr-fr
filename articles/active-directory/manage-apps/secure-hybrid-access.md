---
title: Accès hybride sécurisé à Azure AD | Microsoft Docs
description: Cet article décrit les solutions de partenaires pour intégrer vos applications locales, de cloud public ou de cloud privé héritées à Azure AD. Sécurisez vos applications héritées en connectant des contrôleurs ou des réseaux de livraison d’application à Azure AD.
services: active-directory
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: how-to
ms.workload: identity
ms.date: 9/10/2020
ms.author: gasinh
ms.collection: M365-identity-device-management
ms.openlocfilehash: 521439da044fb4fc3c2f578f28658215266138d9
ms.sourcegitcommit: e5f9126c1b04ffe55a2e0eb04b043e2c9e895e48
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96317093"
---
# <a name="secure-hybrid-access-secure-legacy-apps-with-azure-active-directory"></a>Sécuriser l’accès hybride : Sécuriser des applications héritées avec Azure Active Directory

Vous pouvez désormais protéger vos applications d’authentification héritées locales et dans le cloud en les connectant à Azure Active Directory (AD) avec :

- [Proxy d’application Azure AD](#secure-hybrid-access-sha-through-azure-ad-application-proxy)

- [Vos contrôleurs et réseaux de remise d’application existants](#sha-through-networking-and-delivery-controllers)

- [Les applications de réseau privé virtuel (VPN) et de périmètre défini par logiciel (SDP)](#sha-through-vpn-and-sdp-applications)

Vous pouvez mener à bien vos transitions et renforcer la sécurité de l’ensemble de vos applications grâce à des fonctionnalités de Azure AD telles que Azure AD [Conditional Access](../conditional-access/overview.md) et Azure AD [Identity Protection](../identity-protection/overview-identity-protection.md).

## <a name="secure-hybrid-access-sha-through-azure-ad-application-proxy"></a>Sécurisation de l’accès hybride (SHA) via le proxy d’application Azure AD
  
À l’aide du [Proxy d'application](./what-is-application-proxy.md), vous pouvez fournir un [accès à distance sécurisé](./application-proxy.md) à vos applications web locales. Vos utilisateurs n’ont pas besoin d’utiliser un VPN. Les utilisateurs bénéficient d’une connexion facile à leurs applications à partir de n’importe quel appareil après une [authentification unique](./add-application-portal-setup-sso.md). Le proxy d’application fournit l’accès à distance en tant que service et vous permet de [facilement publier vos applications locales](./application-proxy-add-on-premises-application.md) à des utilisateurs en dehors du réseau d’entreprise. Il vous permet de mettre à l’échelle votre gestion des accès cloud sans avoir à modifier vos applications locales. [Planifiez le déploiement d’un proxy d’application Azure AD](./application-proxy-deployment-plan.md) comme étape suivante.

## <a name="azure-ad-partner-integrations"></a>Intégrations de partenaires Azure AD

### <a name="sha-through-networking-and-delivery-controllers"></a>SHA via les contrôleurs de livraison et de mise en réseau

En plus du [Proxy d’application Azure AD](./what-is-application-proxy.md), pour vous permettre d’utiliser le [framework de Confiance Zéro](https://www.microsoft.com/security/blog/2020/04/02/announcing-microsoft-zero-trust-assessment-tool/), les partenaires Microsoft avec des fournisseurs tiers. Vous pouvez utiliser votre contrôleur de livraison et de mise en réseau et protéger facilement vos applications héritées, qui sont critiques pour vos processus d’entreprise, mais que vous ne pouviez pas protéger avant avec Azure AD. Il est probable que vous disposiez déjà de tout ce dont vous avez besoin pour commencer à protéger ces applications.

![L’image montre un accès hybride sécurisé avec les partenaires de mise en réseau et le proxy d’application](./media/secure-hybrid-access/secure-hybrid-access.png)

Les fournisseurs réseau suivants proposent des solutions prédéfinies et des conseils détaillés pour l’intégration avec Azure AD.

- [Akamai Enterprise Application Access (EAA)](../saas-apps/akamai-tutorial.md)

- [Citrix Application Delivery Controller (ADC)](../saas-apps/citrix-netscaler-tutorial.md)

- [F5 Big-IP APM](f5-aad-integration.md)

- [Kemp](../saas-apps/kemp-tutorial.md)

### <a name="sha-through-vpn-and-sdp-applications"></a>SHA par le biais d’applications VPN et SDP

À l’aide de solutions VPN et SDP, vous pouvez fournir un accès sécurisé à votre réseau d’entreprise à partir de n’importe quel appareil, à tout moment et à tout endroit, tout en protégeant les données de votre organisation. En ayant Azure AD comme fournisseur d’identité (IDP), vous pouvez utiliser des méthodes d’authentification et d’autorisation modernes comme [l’authentification unique](./what-is-single-sign-on.md) et [l’authentification multifacteur](../authentication/concept-mfa-howitworks.md) Azure AD pour sécuriser vos applications héritées locales.  

![L’image montre un accès hybride sécurisé avec les partenaires VPN et le proxy d’application ](./media/secure-hybrid-access/app-proxy-vpn.png)

Les fournisseurs de VPN et SDP suivants proposent des solutions prédéfinies et des conseils détaillés pour l’intégration avec Azure AD.

• [Cisco AnyConnect](../saas-apps/cisco-anyconnect.md)

• [Fortinet](../saas-apps/fortigate-ssl-vpn-tutorial.md)

• [F5 Big-IP APM](f5-aad-password-less-vpn.md)

• [Palo Alto Networks Global Protect](../saas-apps/paloaltoadmin-tutorial.md)

• [Zscaler Private Access (ZPA)](../saas-apps/zscalerprivateaccess-tutorial.md)
