---
paths:
  - "**/supabase/**"
  - "**/lib/supabase*"
  - "**/repositories/**"
  - "**/data/api/**"
---

# Règles Supabase

## Client Supabase

- Utiliser le **SDK Supabase** pour tous les accès à la base de données. Ne pas écrire de SQL brut côté application.
- Le client Supabase est initialisé une seule fois dans `shared/lib/supabase.ts` et réutilisé partout.
- Distinguer le client **côté serveur** (avec service role key) et **côté client** (avec anon key). Ne jamais exposer la service role key côté client.

## Requêtes

- Toujours sélectionner explicitement les colonnes nécessaires avec `.select("col1, col2")`. Ne pas utiliser `.select("*")` sauf pour du prototypage rapide.
- Toujours vérifier la propriété `error` retournée par les appels Supabase. Ne jamais supposer que `data` est non-null sans vérification.
- Côté backend, les appels Supabase sont encapsulés dans des fonctions de **repository** (`repositories/`), réutilisables par plusieurs services.
- Côté frontend, les appels Supabase sont encapsulés dans `data/api/`, consommés par les hooks TanStack Query dans `data/queries/`.

```tsx
// ✅ Correct (repository backend ou data/api frontend)
export const getUserById = async (userId: string) => {
	const { data, error } = await supabase
		.from("users")
		.select("id, name, email")
		.eq("id", userId)
		.single();

	if (error) {
		throw new Error(`Erreur lors de la récupération de l'utilisateur : ${error.message}`);
	}

	return data;
};
```

## Authentification

- Utiliser **Supabase Auth** pour toute la gestion d'authentification.
- Côté frontend : la session utilisateur est gérée via le listener `onAuthStateChange`. Les routes protégées vérifient la session au niveau du routeur (TanStack Router `beforeLoad` / Expo Router layout), pas dans chaque composant.
- Côté backend : un middleware Hono vérifie le token JWT Supabase et injecte l'utilisateur dans le contexte de la requête.

## Types

- Générer les types TypeScript depuis le schéma Supabase avec `npx supabase gen types typescript`.
- Stocker les types générés dans un fichier dédié (`shared/types/database.ts`).
- Regénérer les types après chaque migration.

## Migrations

- Les migrations sont gérées exclusivement via la **CLI Supabase** (`npx supabase migration new`, `npx supabase db push`).
- Ne jamais modifier un fichier de migration existant. Créer une nouvelle migration corrective si nécessaire.
- Les RLS (Row Level Security) policies doivent être définies pour chaque table. Aucune table ne doit rester sans RLS en production.

## Edge Functions

- Si des edge functions Supabase sont utilisées, elles suivent les mêmes conventions TypeScript que le reste du projet.
- Chaque edge function est dans son propre dossier sous `supabase/functions/`.
