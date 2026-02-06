---
paths:
  - "**/app/**"
  - "**/features/**"
  - "**/shared/**"
---

# Règles React Native + Expo

> Ces règles s'appliquent uniquement aux projets mobiles utilisant React Native avec Expo. Supprimer ce fichier pour les projets web-only.

## Framework

- Utiliser **Expo** (managed workflow) par défaut. Ejection (`expo prebuild`) uniquement si un module natif non supporté l'exige.
- Utiliser **Expo Router** pour la navigation (file-based routing).

## Architecture

L'architecture feature-based est la même que pour le web : `ui/`, `business/`, `data/`, `types/`, `utils/`, `constants/`, `index.ts`. Les mêmes règles de séparation des responsabilités et de flux de dépendances s'appliquent.

## Composants

- Utiliser les composants de `react-native` (`View`, `Text`, `Pressable`, etc.), jamais de `div`, `span`, ou de HTML.
- Ne pas utiliser `TouchableOpacity` ou `TouchableHighlight`. Préférer `Pressable` qui est le standard actuel.
- Les styles utilisent `StyleSheet.create()` pour la performance, définis en bas du fichier de composant.
- Ne pas utiliser Tailwind sur mobile (NativeWind) sauf si le projet a été configuré pour. Préférer `StyleSheet` pour la lisibilité et la performance.

## Navigation

- Les routes sont définies via le file system d'Expo Router.
- Les paramètres de navigation sont typés.
- Les layouts partagés (`_layout.tsx`) gèrent l'authentification et les wrappers de navigation.

## Performance

- Les **listes longues** utilisent `FlashList` (de Shopify) plutôt que `FlatList` pour de meilleures performances.
- Les images distantes utilisent `expo-image` (avec cache intégré). Les images statiques sont chargées via `require()`.
- Les animations utilisent `react-native-reanimated`. Ne pas utiliser `Animated` de React Native de base.

## Spécificités plateforme

- Si un comportement diffère entre iOS et Android, utiliser `Platform.select()` ou des fichiers avec extensions `.ios.tsx` / `.android.tsx`.
- Toujours tester sur les deux plateformes.

## Assets et configuration

- Les assets (images, fonts) sont dans le dossier `assets/`.
- La configuration Expo est dans `app.json` ou `app.config.ts`. Ne jamais modifier `app.json` directement si `app.config.ts` existe.

## Stores et persistence

- Les mêmes patterns de state management que le web s'appliquent (voir les règles de state management).
- Le stockage persistant local utilise `expo-secure-store` pour les données sensibles (tokens) et `@react-native-async-storage/async-storage` pour le reste.
