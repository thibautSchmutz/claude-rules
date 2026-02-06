---
paths:
  - "**/*.ts"
  - "**/*.tsx"
---

# Règles TypeScript

## Interdictions strictes

- **`any` est interdit.** Utiliser `unknown` si le type est réellement inconnu, puis le narrower avec un type guard ou un schéma Zod.
- **`@ts-ignore` est interdit.** Utiliser `@ts-expect-error` avec un commentaire explicatif uniquement si absolument nécessaire, et ouvrir un ticket pour corriger la cause.
- **`as` (type assertion) est à éviter.** Préférer les type guards, les schémas Zod (`z.parse()`), ou les generics. `as const` est acceptable.
- **`enum` est interdit.** Utiliser des `const` objets avec `as const` et en dériver le type union.

## Typage

- Activer le mode **strict** dans `tsconfig.json`.
- Toujours typer explicitement les **retours de fonctions** exportées.
- Utiliser `type` plutôt que `interface`, sauf pour les objets qui seront étendus par `extends`.
- Les props de composants React sont typées avec `type` et suffixées par `Props` : `type UserCardProps = { ... }`.
- Préférer les **union types** aux booléens quand un état a plus de deux possibilités : `type Status = "idle" | "loading" | "error" | "success"` plutôt que `isLoading: boolean`.

## Fonctions

- Utiliser des **arrow functions** par défaut.
- Les fonctions utilitaires pures doivent être typées avec des generics quand c'est pertinent.
- Éviter les fonctions avec plus de 3 paramètres. Au-delà, utiliser un objet de paramètres typé.

## Imports

- Utiliser des **path aliases** (`@/` ou `~/`) plutôt que des chemins relatifs profonds (`../../../`).
- Organiser les imports dans cet ordre : (1) modules Node/externes, (2) alias internes, (3) imports relatifs.
- Ne jamais importer directement depuis les fichiers internes d'un autre module feature. Passer par le barrel export (`index.ts`).

## Validation avec Zod

- Chaque entité venant de l'extérieur (API, formulaire, URL params) doit être validée avec un schéma Zod.
- Dériver les types TypeScript depuis les schémas Zod avec `z.infer<typeof schema>` pour éviter la duplication.

### Colocalisation des schémas

Les schémas Zod suivent le **principe de colocation** :

- **Schémas modèle** (entités métier : `userSchema`, `paymentSchema`) → dans `types/` du module. Ils sont partagés entre plusieurs couches et potentiellement exportés vers d'autres modules.
- **Schémas techniques** (validation d'une réponse API brute, transformation intermédiaire) → colocalisés dans le fichier qui les utilise (dans `data/` côté frontend, dans le fichier de route côté backend). Ils ne sont pas exportés.
- **Schémas opérationnels backend** (body d'un PATCH, query params) → dans `types/` du module backend car ils servent à la fois de validation dans la route et de typage dans le service.
- **Règle simple** : si un schéma est importé par plus d'un fichier, il va dans `types/`. Sinon, il reste local.
