# [NOM DU PROJET]

> [Description courte du projet en une ligne]

## Stack technique

- **Framework web** : [Vite + React | TanStack Start]
- **Framework backend** : [Hono (Cloudflare Workers)]
- **Mobile** : [React Native + Expo (si applicable)]
- **Langage** : TypeScript (strict mode)
- **Runtime** : Node.js
- **Package manager** : pnpm
- **Base de données** : PostgreSQL via Supabase
- **Authentification** : Supabase Auth
- **Styling** : Tailwind CSS
- **Validation** : Zod

## Commandes

- `pnpm install` : installer les dépendances
- `pnpm dev` : lancer le serveur de développement
- `pnpm build` : build de production
- `pnpm lint` : lancer ESLint
- `pnpm format` : formater avec Prettier
- `pnpm typecheck` : vérification des types TypeScript
- `pnpm test` : lancer les tests unitaires (Vitest)
- `pnpm test:e2e` : lancer les tests end-to-end (Playwright)

## Architecture frontend / mobile

Ce projet suit une architecture **feature-based** avec une séparation en trois couches : UI, Business, Data.

```
src/
├── features/
│   └── [module]/
│       ├── ui/               # Composants React purs (rendu)
│       ├── business/         # Hooks de logique métier (orchestration, états dérivés)
│       ├── data/             # Accès données (API, queries, stores)
│       │   ├── api/          # Appels Supabase bruts
│       │   ├── queries/      # Hooks TanStack Query (useQuery, useMutation)
│       │   ├── stores/       # Stores Zustand (globaux ou locaux via Context)
│       │   └── index.ts      # Hooks façade (useUser, usePayment, etc.)
│       ├── types/            # Schémas Zod modèle + types dérivés
│       ├── utils/            # Fonctions utilitaires pures
│       ├── constants/        # Valeurs constantes
│       └── index.ts          # Barrel export public
├── shared/
│   ├── ui/                   # Composants UI réutilisables (design system)
│   ├── hooks/                # Hooks partagés
│   ├── lib/                  # Clients (supabase, etc.)
│   ├── types/                # Types globaux
│   └── schemas/              # Schémas Zod partagés
├── routes/                   # Définition des routes (TanStack Router / Expo Router)
└── app.tsx                   # Point d'entrée
```

Flux de dépendances (unidirectionnel) : `ui/ → business/ → data/ → types/`

## Architecture backend

```
src/
├── index.ts                  # Point d'entrée : middlewares globaux + router
├── router.ts                 # Monte les routeurs de chaque feature
├── shared/
│   ├── middlewares/           # Middlewares globaux (auth, cors, logging)
│   ├── lib/                  # Clients partagés (supabase, etc.)
│   ├── types/                # Types/schémas partagés
│   └── utils/                # Réponse standardisée, helpers
├── features/
│   └── [module]/
│       ├── routes/           # Handlers de route ([id].get.ts, [id].patch.ts)
│       ├── services/         # Logique métier et orchestration
│       ├── repositories/     # Appels Supabase encapsulés
│       ├── middlewares/       # Middlewares spécifiques à la feature
│       ├── types/            # Schémas Zod (modèle + opérationnels)
│       ├── utils/            # Utilitaires de la feature
│       ├── router.ts         # Routeur Hono de la feature
│       └── index.ts          # Barrel export (exporte le routeur)
```

Flux d'une requête : `Route → (valide input Zod) → Service → Repository → Supabase`

## Tests

Les tests servent de **documentation produit**. L'arborescence des fichiers de test doit permettre de comprendre ce que fait l'application d'un coup d'œil.

```
tests/
├── utils/                    # Fixtures, mocks, helpers partagés
├── [domaine]/                # auth/, profile/, payment/...
│   └── [action]/             # login/, register/... (si plusieurs variantes)
│       └── [use-case].spec.ts
```

## Contexte métier

[Décrire ici le domaine métier du projet, les entités principales, et les flux utilisateur clés.]

## Fichiers à ne jamais modifier

- `pnpm-lock.yaml` (géré automatiquement)
- `.env` et `.env.*` (contiennent des secrets)
- `supabase/migrations/` (géré par la CLI Supabase)
