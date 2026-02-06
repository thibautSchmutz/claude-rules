---
name: code-review
description: Revue de code approfondie basée sur les conventions du projet. Utiliser lors de la review de code, avant de soumettre une PR, ou pour analyser la qualité du code modifié.
allowed-tools: Read, Grep, Glob
---

# Revue de code

## Instructions

Effectuer une revue de code en vérifiant les points suivants, dans l'ordre :

### 1. Conformité TypeScript
- Pas de `any`, `@ts-ignore`, ou `as` injustifié
- Types explicites sur les retours de fonctions exportées
- Schémas Zod pour toute donnée externe, types dérivés avec `z.infer`

### 2. Architecture et séparation des responsabilités

**Frontend / Mobile :**
- Respect de la structure feature-based (`ui/`, `business/`, `data/`, `types/`)
- Les composants dans `ui/` ne consomment jamais `data/` directement (passent par `business/`)
- Les hooks façade dans `data/index.ts` sont la seule interface d'accès aux données
- Le flux de dépendances est unidirectionnel : `ui/ → business/ → data/ → types/`

**Backend :**
- Respect de la structure `routes/ → services/ → repositories/`
- Les routes ne contiennent pas de logique métier
- Les repositories ne contiennent pas de logique métier
- Les appels Supabase sont encapsulés dans les repositories

### 3. Conventions générales
- Named exports uniquement (sauf exception framework)
- Fichiers en kebab-case, < 300 lignes
- Early returns utilisés
- Arrow functions
- Pas d'import croisé entre features (passer par barrel exports)

### 4. State management (frontend / mobile)
- Bonne solution choisie selon la hiérarchie (local → route → query → store local → store global)
- Pas de state serveur dans Zustand (doit être dans TanStack Query)
- Query keys cohérentes et centralisées

### 5. Gestion des erreurs
- Pas de catch vide
- Erreurs Supabase vérifiées
- Validation Zod en entrée
- Backend : réponses au format standardisé `{ data, error, meta }`

### 6. Sécurité
- Pas de secrets exposés
- Pas de `.select("*")` en production
- RLS vérifié si nouvelles tables

### 7. Principes de développement
- KISS : pas de complexité accidentelle
- YAGNI : pas de code "au cas où"
- SRP : chaque fonction/fichier a une responsabilité unique
- DRY : pas de duplication au-delà de 2 occurrences sans extraction

## Format de sortie

Présenter les résultats sous forme de liste avec 3 catégories :
- 🔴 **Bloquant** : doit être corrigé avant merge
- 🟡 **Suggestion** : amélioration recommandée
- 🟢 **Bien fait** : points positifs à souligner
