---
title: Sérialisation de cache de jeton personnalisée (MSAL4j)
titleSuffix: Microsoft identity platform
description: Découvrir comment sérialiser le cache de jetons pour MSAL pour Java
services: active-directory
author: sangonzal
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 11/07/2019
ms.author: sagonzal
ms.reviewer: nacanuma
ms.custom: aaddev, devx-track-java
ms.openlocfilehash: e85cfb8c03111b889a9f95509f337d415c517163
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87312365"
---
# <a name="custom-token-cache-serialization-in-msal-for-java"></a>Sérialisation personnalisée du cache de jetons dans MSAL.NET pour Java

Pour faire persister le cache de jeton entre les instances de votre application, vous devez personnaliser la sérialisation. Les classes et interfaces Java concernées par la sérialisation du cache de jeton sont les suivantes :

- [ITokenCache](https://static.javadoc.io/com.microsoft.azure/msal4j/0.5.0-preview/com/microsoft/aad/msal4j/ITokenCache.html) :  interface représentant le cache du jeton de sécurité.
- [ITokenCacheAccessAspect](https://static.javadoc.io/com.microsoft.azure/msal4j/0.5.0-preview/com/microsoft/aad/msal4j/ITokenCacheAccessAspect.html) : interface représentant l’opération d’exécution du code avant et après l’accès. Vous devez remplacer (@Override) *beforeCacheAccess* et *afterCacheAccess* par la logique responsable de la sérialisation et de la désérialisation du cache.
- [ITokenCacheContext](https://static.javadoc.io/com.microsoft.azure/msal4j/0.5.0-preview/com/microsoft/aad/msal4j/ITokenCacheAccessContext.html) : interface représentant le contexte dans lequel l’accès au cache de jeton est effectué. 

Voici une implémentation naïve de la sérialisation personnalisée de la sérialisation/désérialisation du cache de jetons. Ne copiez pas et ne collez pas ceci dans un environnement de production.

```Java
static class TokenPersistence implements ITokenCacheAccessAspect {
String data;

TokenPersistence(String data) {
        this.data = data;
}

@Override
public void beforeCacheAccess(ITokenCacheAccessContext iTokenCacheAccessContext) {
        iTokenCacheAccessContext.tokenCache().deserialize(data);
}

@Override
public void afterCacheAccess(ITokenCacheAccessContext iTokenCacheAccessContext) {
        data = iTokenCacheAccessContext.tokenCache().serialize();
}
```

```Java
// Loads cache from file
String dataToInitCache = readResource(this.getClass(), "/cache_data/serialized_cache.json");

ITokenCacheAccessAspect persistenceAspect = new TokenPersistence(dataToInitCache);

// By setting *TokenPersistence* on the PublicClientApplication, MSAL will call *beforeCacheAccess()* before accessing the cache and *afterCacheAccess()* after accessing the cache. 
PublicClientApplication app = 
PublicClientApplication.builder("my_client_id").setTokenCacheAccessAspect(persistenceAspect).build();
```

## <a name="learn-more"></a>En savoir plus

En savoir plus sur [Obtenir et supprimer des comptes depuis le cache de jeton à l’aide de MSAL pour Java](msal-java-get-remove-accounts-token-cache.md).
