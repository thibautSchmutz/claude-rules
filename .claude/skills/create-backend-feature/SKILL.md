---
name: create-backend-feature
description: Créer la structure d'un nouveau module feature côté backend (Hono / Cloudflare Workers). Utiliser quand on démarre un nouveau module, un nouveau domaine métier, ou un nouvel ensemble d'endpoints dans l'API. Ne PAS utiliser pour du code frontend ou mobile.
---

# Créer un module feature (backend)

## Contexte

Ce skill s'applique uniquement aux applications **backend (Hono / Cloudflare Workers)**. Pour le frontend ou le mobile, utiliser le skill `create-feature`.

## Instructions

Quand l'utilisateur demande de créer un nouveau module feature backend, générer la structure suivante sous `src/features/[nom-du-module]/` :

```
src/features/[nom-du-module]/
├── routes/               # Handlers de route (un fichier par endpoint)
│   ├── index.get.ts      # GET /[module]
│   ├── index.post.ts     # POST /[module]
│   ├── [id].get.ts       # GET /[module]/:id
│   ├── [id].patch.ts     # PATCH /[module]/:id
│   └── [id].delete.ts    # DELETE /[module]/:id
├── services/             # Logique métier et orchestration
├── repositories/         # Appels Supabase encapsulés et réutilisables
├── middlewares/           # Middlewares Hono spécifiques à la feature
├── types/                # Schémas Zod (modèle + opérationnels) + types dérivés
├── utils/                # Fonctions utilitaires de la feature
├── router.ts             # Routeur Hono qui assemble les routes de la feature
└── index.ts              # Barrel export (exporte le routeur)
```

## Règles de génération

1. **Nommage** : le nom du module est en kebab-case (`user-profile`, `payment`, `notification`).
2. **index.ts** : exporte le routeur Hono de la feature. C'est la seule chose que le routeur principal importe.
3. **routes/** : un fichier par endpoint. Le nom du fichier rend transparent le path et la méthode HTTP : `[id].patch.ts` = `PATCH /:id`.
4. **types/** : contient les schémas Zod **modèle** (entités métier) ET les schémas **opérationnels** (body de requête, query params). Chaque schéma exporte aussi son type dérivé avec `z.infer`.
5. **services/** : contient la logique métier. Un fichier par opération complexe. Peut appeler plusieurs repositories.
6. **repositories/** : contient les appels Supabase. Une fonction = une requête. Réutilisable par plusieurs services.
7. **Ne créer que les dossiers nécessaires.** Si la feature n'a pas de middlewares spécifiques, ne pas créer `middlewares/`.

## Flux d'une requête

```
Route → (valide input Zod) → Service → Repository → Supabase
```

- **Route** : handler mince. Valide l'input, appelle le service, retourne la réponse formatée. Aucune logique métier.
- **Service** : orchestration. Combine repositories, applique les règles métier, coordonne les effets de bord.
- **Repository** : accès données pur. Pas de logique métier.

Pour un CRUD simple sans logique métier, la route peut appeler le repository directement (sans service intermédiaire).

## Exemple : routeur de la feature

```tsx
// router.ts
import { Hono } from "hono";

const profileRouter = new Hono();

// Importer et monter les routes
// GET /profile
profileRouter.get("/", getProfilesHandler);
// GET /profile/:id
profileRouter.get("/:id", getProfileByIdHandler);
// PATCH /profile/:id
profileRouter.patch("/:id", ownershipMiddleware, updateProfileHandler);

export { profileRouter };
```

## Exemple : route

```tsx
// routes/[id].patch.ts
import { updateProfileBody } from "../types/update-profile.schema";
import { updateProfile } from "../services/update-profile.service";
import { formatResponse } from "@/shared/utils/response";

export const updateProfileHandler = async (c) => {
	const id = c.req.param("id");
	const body = updateProfileBody.parse(await c.req.json());
	const profile = await updateProfile(id, body);
	return c.json(formatResponse({ data: profile }));
};
```

## Exemple : schéma opérationnel

```tsx
// types/update-profile.schema.ts
import { z } from "zod";

export const updateProfileBody = z.object({
	displayName: z.string().min(1).optional(),
	bio: z.string().max(500).optional(),
});

export type UpdateProfileBody = z.infer<typeof updateProfileBody>;
```

## Exemple : service

```tsx
// services/update-profile.service.ts
import type { UpdateProfileBody } from "../types/update-profile.schema";
import { getProfileById, updateProfileInDb } from "../repositories/profile.repository";

export const updateProfile = async (id: string, body: UpdateProfileBody) => {
	const existing = await getProfileById(id);

	if (!existing) {
		throw new Error("Profil introuvable");
	}

	return await updateProfileInDb(id, body);
};
```

## Exemple : repository

```tsx
// repositories/profile.repository.ts
import { supabase } from "@/shared/lib/supabase";

export const getProfileById = async (id: string) => {
	const { data, error } = await supabase
		.from("profiles")
		.select("id, display_name, bio, avatar_url")
		.eq("id", id)
		.single();

	if (error) throw new Error(error.message);
	return data;
};

export const updateProfileInDb = async (id: string, updates: Record<string, unknown>) => {
	const { data, error } = await supabase
		.from("profiles")
		.update(updates)
		.eq("id", id)
		.select("id, display_name, bio, avatar_url")
		.single();

	if (error) throw new Error(error.message);
	return data;
};
```

## Exemple : barrel export

```tsx
// index.ts
export { profileRouter } from "./router";
```

## Après création, ne pas oublier

1. **Monter le routeur** dans `src/router.ts` :
   ```tsx
   import { profileRouter } from "./features/profile";
   router.route("/profile", profileRouter);
   ```

2. **Ajouter les tests** dans `tests/[module]/` en suivant la convention de nommage par use case.

## Checklist après création

- [ ] Le routeur est exporté via `index.ts`
- [ ] Le routeur est monté dans `src/router.ts`
- [ ] Au moins un schéma Zod modèle est défini pour l'entité principale
- [ ] Les schémas opérationnels sont dans `types/` et servent à la validation (route) ET au typage (service)
- [ ] Les appels Supabase sont encapsulés dans `repositories/`
- [ ] Les routes ne contiennent pas de logique métier
- [ ] Les réponses utilisent le format standardisé `{ data, error, meta }`
