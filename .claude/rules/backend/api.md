---
paths:
  - "**/api/**"
  - "**/routes/**"
  - "**/services/**"
  - "**/repositories/**"
  - "**/router.ts"
---

# Règles API REST (Hono / Cloudflare Workers)

## Architecture

Le backend utilise **Hono** déployé sur **Cloudflare Workers**. Chaque feature expose un routeur monté dans le routeur principal.

### Point d'entrée

```tsx
// src/index.ts — middlewares globaux + routeur principal
import { Hono } from "hono";
import { router } from "./router";

const app = new Hono();

// Middlewares globaux
app.use("*", cors());
app.use("*", logger());

app.route("/", router);

export default app;
```

```tsx
// src/router.ts — monte les routeurs de chaque feature
import { Hono } from "hono";
import { profileRouter } from "./features/profile";
import { paymentRouter } from "./features/payment";

const router = new Hono();

router.route("/profile", profileRouter);
router.route("/payment", paymentRouter);

export { router };
```

### Couches d'une feature backend

- **Route** (`routes/`) : handler mince. Valide l'input avec Zod, appelle le service, retourne la réponse formatée. Aucune logique métier.
- **Service** (`services/`) : orchestration et logique métier. Combine plusieurs repositories, applique les règles métier, coordonne les effets de bord.
- **Repository** (`repositories/`) : accès données pur. Une fonction = une requête Supabase. Réutilisable par plusieurs services.
- **Middleware** (`middlewares/`) : middleware Hono spécifiques à la feature (ownership, rate limiting, etc.).

### Convention de nommage des fichiers de route

Les fichiers de route rendent transparent le path et la méthode HTTP :

```
features/profile/routes/
├── index.get.ts          # GET /profile
├── index.post.ts         # POST /profile
├── [id].get.ts           # GET /profile/:id
├── [id].patch.ts         # PATCH /profile/:id
└── [id].delete.ts        # DELETE /profile/:id
```

### Conventions REST

- `GET /resources` : liste
- `GET /resources/:id` : détail
- `POST /resources` : création
- `PATCH /resources/:id` : mise à jour partielle (préférer `PATCH` à `PUT`)
- `DELETE /resources/:id` : suppression

## Format de réponse standardisé

Toutes les réponses API suivent le même format :

```tsx
// shared/utils/response.ts
type ApiResponse<T> = {
	data: T | null;
	error: { code: string; message: string } | null;
	meta?: { page?: number; total?: number };
};

export const formatResponse = <T>({ data, error, meta }: Partial<ApiResponse<T>>): ApiResponse<T> => ({
	data: data ?? null,
	error: error ?? null,
	...(meta && { meta }),
});
```

Utilisation dans une route :

```tsx
// Succès
return c.json(formatResponse({ data: profile }));

// Erreur
return c.json(formatResponse({ error: { code: "NOT_FOUND", message: "Profil introuvable" } }), 404);
```

## Schémas Zod et typage

- Les **schémas modèle** (entités métier) et les **schémas opérationnels** (body, query params) vivent dans `types/` de la feature.
- Le schéma sert à la fois de validation dans la route et de typage dans le service :

```tsx
// types/update-profile.schema.ts
export const updateProfileBody = z.object({
	displayName: z.string().min(1).optional(),
	bio: z.string().max(500).optional(),
});
export type UpdateProfileBody = z.infer<typeof updateProfileBody>;

// routes/[id].patch.ts — utilise le schéma pour valider
const body = updateProfileBody.parse(await c.req.json());
const profile = await updateProfile(id, body);

// services/update-profile.service.ts — utilise le type pour typer
export const updateProfile = async (id: string, body: UpdateProfileBody) => { ... };
```

## Gestion des erreurs

- Les erreurs de validation Zod retournent un 400 avec le détail.
- Les erreurs métier (ressource introuvable, conflit) retournent le code HTTP approprié (404, 409, etc.).
- Les erreurs inattendues sont loggées et retournent un 500 avec un message générique. Ne jamais exposer de stack trace ou de message technique au client.
- Centraliser le error handling dans un middleware Hono global (`app.onError()`).

## Sécurité

- Ne jamais exposer de données sensibles dans les réponses API (mots de passe, tokens, secrets).
- Valider systématiquement les inputs avec Zod avant tout traitement.
- Les endpoints authentifiés vérifient la session Supabase via un middleware avant d'atteindre la route.
