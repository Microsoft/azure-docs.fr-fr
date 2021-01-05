---
title: Causes courantes de recyclage des rôles de service cloud | Microsoft Docs
description: Le recyclage soudain d’un rôle de service cloud peut entraîner d’importants temps d’arrêt. Voici quelques problèmes courants entraînant le recyclage des rôles, qui peuvent vous aider à réduire les temps d’arrêt.
services: cloud-services
documentationcenter: ''
author: simonxjx
manager: dcscontentpm
editor: ''
tags: top-support-issue
ms.assetid: 533930d1-8035-4402-b16a-cf887b2c4f85
ms.service: cloud-services
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: tbd
ms.date: 06/15/2018
ms.author: v-six
ms.openlocfilehash: 0484eb919a9de11b64dcc3334c5a9a942d875ca6
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92075125"
---
# <a name="common-issues-that-cause-roles-to-recycle"></a>Problèmes courants provoquant le recyclage des rôles
Cet article examine certaines des causes courantes entraînant des problèmes de déploiement, et indique des conseils de dépannage pour vous aider à résoudre ces problèmes. Une instance de rôle qui ne démarre pas ou qui est recyclée entre les états Initialisation, Occupée et Arrêt indique un problème au niveau de l’application.

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="missing-runtime-dependencies"></a>Dépendances d'exécution manquantes
Si un rôle dans votre application repose sur un assembly qui ne fait pas partie du .NET Framework ou de la bibliothèque gérée Azure, vous devez inclure explicitement cet assembly dans le package d'application. N'oubliez pas que les autres infrastructures Microsoft ne sont pas disponibles par défaut dans Azure. Si votre rôle repose sur une telle infrastructure, vous devez ajouter ces assemblys au package d'application.

Avant de générer et de mettre en package votre application, vérifiez les éléments suivants :

* Si vous utilisez Visual Studio, assurez-vous que la propriété **Copy Local** est définie sur **True** pour chaque assembly référencé dans votre projet ne faisant pas partie du Kit de développement logiciel (SDK) Azure ni du .NET Framework.
* Assurez-vous que le fichier web.config ne fait pas référence à des assemblys inutilisés dans l’élément compilation.
* La propriété **Build Action** de chaque fichier .cshtml est définie sur **Content**. Cela garantit que les fichiers s’affichent correctement dans le package et permet à d’autres fichiers référencés d’apparaître dans le package.

## <a name="assembly-targets-wrong-platform"></a>L’assembly cible la mauvaise plateforme
Azure est un environnement 64 bits. Par conséquent, les assemblys .NET compilés pour une cible 32 bits ne fonctionneront pas sur Azure.

## <a name="role-throws-unhandled-exceptions-while-initializing-or-stopping"></a>Le rôle génère des exceptions non gérées lors de l'initialisation ou de l'arrêt
Les exceptions levées par les méthodes de la classe [RoleEntryPoint], notamment les méthodes [OnStart], [OnStop] et [Exécuter], sont des exceptions non prises en charge. Si une exception non gérée se produit dans une de ces méthodes, le rôle sera recyclé. Si le rôle est recyclé à plusieurs reprises, il peut lever une exception non prise en charge chaque fois qu’il tente de démarrer.

## <a name="role-returns-from-run-method"></a>Le rôle est renvoyé à partir de la méthode Run
La méthode [Exécuter] est destinée à être exécutée indéfiniment. Si votre code remplace la méthode [Exécuter] , il devrait être en veille indéfiniment. Si la méthode [Exécuter] est renvoyée, le rôle est recyclé.

## <a name="incorrect-diagnosticsconnectionstring-setting"></a>Chaîne DiagnosticsConnectionString incorrecte
Si l’application utilise les diagnostics Azure, votre fichier de configuration de service doit spécifier le paramètre de configuration `DiagnosticsConnectionString` . Ce paramètre doit spécifier une connexion HTTPS à votre compte de stockage dans Azure.

Pour vous assurer que votre paramètre `DiagnosticsConnectionString` est correct avant de déployer votre package d'application dans Azure, vérifiez les éléments suivants :  

* Le paramètre `DiagnosticsConnectionString` pointe vers un compte de stockage valide dans Azure.  
  Par défaut, ce paramètre pointe vers le compte de stockage émulé, et vous devez donc modifier explicitement ce paramètre avant de déployer votre package d'application. Si vous ne modifiez pas ce paramètre, une exception est générée lorsque l'instance de rôle tente de démarrer le moniteur de diagnostic. L'instance de rôle risque alors d’être recyclée indéfiniment.
* La chaîne de connexion est spécifiée au [format](../storage/common/storage-configure-connection-string.md) suivant. (Le protocole HTTPS doit être spécifié.) Remplacez *MyAccountName* par le nom de votre compte de stockage et *MyAccountKey* par votre clé d’accès :    

```console
DefaultEndpointsProtocol=https;AccountName=MyAccountName;AccountKey=MyAccountKey
```

  Si vous développez votre application à l’aide d’outils Azure pour Microsoft Visual Studio, vous pouvez utiliser les pages de propriétés pour définir cette valeur.

## <a name="exported-certificate-does-not-include-private-key"></a>Le certificat exporté ne contient aucune clé privée
Pour exécuter un rôle web sous TLS, vous devez vous assurer que votre certificat de gestion exporté inclut la clé privée. Si vous utilisez le *gestionnaire de certificats Windows* pour exporter le certificat, veillez à sélectionner **Oui** pour l’option **Exporter la clé privée**. Le certificat doit être exporté au format PFX, l’unique format actuellement pris en charge.

## <a name="next-steps"></a>Étapes suivantes
Affichez plus d’ [articles de résolution des problèmes](../index.yml?product=cloud-services&tag=top-support-issue) liés aux services cloud.

Affichez d’autres scénarios de recyclage des rôles dans la [série du blog de Kevin Williamson](/archive/blogs/kwill/windows-azure-paas-compute-diagnostics-data).

[RoleEntryPoint]: /previous-versions/azure/reference/ee758619(v=azure.100)
[OnStart]: /previous-versions/azure/reference/ee772851(v=azure.100)
[OnStop]: /previous-versions/azure/reference/ee772844(v=azure.100)
[Exécuter]: /previous-versions/azure/reference/ee772746(v=azure.100)