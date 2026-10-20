# Manuel de parcours Next.js : clone simplifié d'Airbnb

> Un fil rouge unique, 7 étapes, 4 à 6 semaines à raison de quelques heures par semaine.
> Public visé : développeur ayant déjà des bases en React, HTML/CSS et JavaScript/TypeScript.

---

## Sommaire

1. [Comment utiliser ce manuel](#comment-utiliser-ce-manuel)
2. [Prérequis](#prérequis)
3. [Étape 0 : Mise en place](#étape-0--mise-en-place)
4. [Étape 1 : Interface statique](#étape-1--interface-statique)
5. [Étape 2 : Routing et rendu](#étape-2--routing-et-rendu)
6. [Étape 3 : Base de données](#étape-3--base-de-données)
7. [Étape 4 : Mutations et formulaires](#étape-4--mutations-et-formulaires)
8. [Étape 5 : Authentification](#étape-5--authentification)
9. [Étape 6 : Qualité et mise en production](#étape-6--qualité-et-mise-en-production)
10. [Bonus](#bonus)
11. [Pièges fréquents](#pièges-fréquents)
12. [Ressources](#ressources)
13. [Journal de progression](#journal-de-progression)

---

## Comment utiliser ce manuel

Chaque étape suit la même structure :

| Rubrique | Contenu |
|---|---|
| **Objectif** | Ce que vous saurez faire à la fin |
| **Notions** | Les concepts à comprendre |
| **À faire** | Les tâches concrètes |
| **Livrable** | Ce qui doit fonctionner avant de passer à la suite |
| **Checklist** | Cases à cocher pour valider |
| **Pour aller plus loin** | Défis optionnels |

### Règles de travail

1. **Pas de copier-coller aveugle.** Bloquez 20 à 30 minutes avant de chercher la solution.
2. **Doc officielle d'abord.** Next.js évolue vite : beaucoup de tutoriels utilisent l'ancien *Pages Router* (`getServerSideProps`, `getStaticProps`), qui n'est pas ce que nous voyons ici.
3. **Un commit par étape au minimum**, avec un message clair (`feat: page détail logement`).
4. **Déployez tôt et souvent**, pour détecter les différences entre local et production.
5. **Écrivez une note de fin d'étape** (voir le [journal de progression](#journal-de-progression)).

### Note sur les versions

Les API de Next.js changent d'une version majeure à l'autre (par exemple, `params` et `searchParams` sont désormais asynchrones dans les versions récentes, et le *middleware* a connu des évolutions). **Vérifiez toujours la documentation de la version que vous installez** sur [nextjs.org/docs](https://nextjs.org/docs).

---

## Prérequis

- Node.js (version LTS récente) et un gestionnaire de paquets (npm, pnpm ou yarn)
- Git et un compte GitHub
- Un compte Vercel (gratuit)
- Un éditeur de code (VS Code recommandé)
- Connaissances attendues : composants React, `useState`/`useEffect`, props, bases de TypeScript, Flexbox/Grid

---

## Étape 0 : Mise en place

**Durée estimée :** 1 soirée

### Objectif
Avoir un projet versionné et déjà déployé.

### À faire

```bash
npx create-next-app@latest airbnb-clone
cd airbnb-clone
npm run dev
```

Choix recommandés à l'installation : TypeScript, ESLint, Tailwind CSS, dossier `src/`, App Router.

```bash
git init
git add .
git commit -m "chore: initial commit"
# créer le dépôt sur GitHub puis :
git remote add origin <url>
git push -u origin main
```

- Importer le dépôt sur [vercel.com](https://vercel.com) pour un déploiement automatique à chaque `push`.
- Installer shadcn/ui :

```bash
npx shadcn@latest init
npx shadcn@latest add button card input
```

### Structure cible

```
src/
├─ app/
│  ├─ layout.tsx
│  └─ page.tsx
├─ components/
├─ lib/
└─ data/
```

### Livrable
Une page « Hello » en ligne sur une URL Vercel.

### Checklist
- [ ] Le projet démarre en local
- [ ] Le dépôt GitHub est créé
- [ ] Le site est déployé sur Vercel
- [ ] shadcn/ui est initialisé

---

## Étape 1 : Interface statique

**Durée estimée :** 3 à 4 jours

### Objectif
Reproduire fidèlement la page d'accueil avec des données en dur.

### Notions
- Découpage en composants réutilisables
- Tailwind CSS : responsive (`sm:`, `md:`, `lg:`), `grid`, `flex`, états (`hover:`, `focus:`)
- `next/image` (dimensions, `fill`, `sizes`, `priority`) et `next/font`
- Design *mobile-first*

### À faire
1. Créer `src/data/listings.ts` avec une dizaine de logements :

```ts
export type Listing = {
  id: string;
  title: string;
  city: string;
  pricePerNight: number;
  rating: number;
  image: string;
};

export const listings: Listing[] = [
  {
    id: "1",
    title: "Studio lumineux près du port",
    city: "Marseille",
    pricePerNight: 85,
    rating: 4.8,
    image: "/images/listing-1.jpg",
  },
  // ...
];
```

2. Construire les composants : `Header`, `SearchBar`, `ListingCard`, `ListingGrid`, `Footer`.
3. Mettre en place une grille responsive (1 colonne sur mobile, jusqu'à 4 sur grand écran).
4. Ajouter une police via `next/font`.

### Livrable
Page d'accueil responsive, visuellement proche du modèle.

### Checklist
- [ ] Au moins 5 composants distincts
- [ ] Affichage correct de 320 px à 1440 px
- [ ] Images optimisées avec `next/image`
- [ ] Aucun avertissement dans la console

### Pour aller plus loin
- Ajouter un mode sombre.
- Ajouter une barre de catégories défilante horizontalement.

---

## Étape 2 : Routing et rendu

**Durée estimée :** 4 à 5 jours

### Objectif
Maîtriser l'App Router et la distinction Server / Client Components.

### Notions
- Routage par dossiers : `app/logements/[id]/page.tsx`
- `layout.tsx` imbriqués
- Fichiers spéciaux : `loading.tsx`, `error.tsx`, `not-found.tsx`
- `generateMetadata` pour le SEO
- `generateStaticParams` pour pré-générer des pages
- **Server Components (par défaut) vs Client Components (`"use client"`)**

### La règle à retenir
Posez-vous la question pour chaque composant : *« A-t-il besoin d'état, d'effets ou d'événements navigateur ? »*
- **Non** → laissez-le en Server Component.
- **Oui** → `"use client"`, et poussez cette directive **le plus bas possible** dans l'arbre des composants.

### À faire
1. Créer la page de détail :

```tsx
// src/app/logements/[id]/page.tsx
import { notFound } from "next/navigation";
import { listings } from "@/data/listings";

type Props = { params: Promise<{ id: string }> };

export async function generateMetadata({ params }: Props) {
  const { id } = await params;
  const listing = listings.find((l) => l.id === id);
  return { title: listing ? listing.title : "Logement introuvable" };
}

export default async function ListingPage({ params }: Props) {
  const { id } = await params;
  const listing = listings.find((l) => l.id === id);
  if (!listing) notFound();

  return (
    <main>
      <h1>{listing!.title}</h1>
      {/* ... */}
    </main>
  );
}
```

2. Rendre chaque carte cliquable avec `next/link`.
3. Ajouter `loading.tsx` (squelette de chargement), `error.tsx` (avec bouton « Réessayer ») et `not-found.tsx`.
4. Créer un composant client minimal, par exemple un carrousel d'images ou un bouton « partager ».

### Livrable
Navigation complète accueil → détail, avec états de chargement et d'erreur.

### Checklist
- [ ] Page dynamique `[id]` fonctionnelle
- [ ] Métadonnées différentes par logement
- [ ] `loading`, `error` et `not-found` testés
- [ ] Vous savez expliquer pourquoi chaque `"use client"` est nécessaire

### Pour aller plus loin
- Routes parallèles ou interceptées (afficher le détail dans une modale).

---

## Étape 3 : Base de données

**Durée estimée :** 5 à 6 jours

### Objectif
Remplacer les données en dur par une vraie base, et filtrer via l'URL.

### Notions
- ORM : **Prisma** ou **Drizzle**
- PostgreSQL hébergé : **Supabase** ou **Neon**
- Requêtes dans les Server Components
- Filtres via `searchParams` (l'URL comme source de vérité)
- Cache et revalidation : `revalidatePath`, `revalidateTag`

### Schéma de départ (Prisma)

```prisma
model User {
  id        String    @id @default(cuid())
  email     String    @unique
  name      String?
  listings  Listing[]
  bookings  Booking[]
  favorites Favorite[]
}

model Listing {
  id            String   @id @default(cuid())
  title         String
  description   String
  city          String
  pricePerNight Int
  imageUrl      String
  ownerId       String
  owner         User     @relation(fields: [ownerId], references: [id])
  bookings      Booking[]
  favorites     Favorite[]
  createdAt     DateTime @default(now())
}

model Booking {
  id        String   @id @default(cuid())
  startDate DateTime
  endDate   DateTime
  userId    String
  listingId String
  user      User     @relation(fields: [userId], references: [id])
  listing   Listing  @relation(fields: [listingId], references: [id])
}

model Favorite {
  userId    String
  listingId String
  user      User    @relation(fields: [userId], references: [id])
  listing   Listing @relation(fields: [listingId], references: [id])

  @@id([userId, listingId])
}
```

### À faire
1. Créer la base, configurer `DATABASE_URL` dans `.env.local`.
2. Initialiser Prisma, lancer la migration, écrire un script de *seed*.
3. Créer un client Prisma partagé dans `src/lib/db.ts` (singleton, pour éviter de multiplier les connexions en développement).
4. Lire les logements dans la page d'accueil et la page de détail.
5. Implémenter la recherche : `/?city=Lille&maxPrice=120`.
6. Mettre à jour l'URL depuis un composant client avec `useRouter` et `useSearchParams`.

### Livrable
Recherche et filtres fonctionnels sur des données persistées.

### Checklist
- [ ] Migration et seed reproductibles
- [ ] Aucun secret commité (`.env*` dans `.gitignore`)
- [ ] Filtres conservés dans l'URL (partageable, bouton retour fonctionnel)
- [ ] Vous comprenez quand une page est rendue statiquement ou dynamiquement

### Pour aller plus loin
- Pagination ou défilement infini.
- Index de base de données sur les colonnes filtrées.

---

## Étape 4 : Mutations et formulaires

**Durée estimée :** 4 à 5 jours

### Objectif
Créer et modifier des données avec les Server Actions.

### Notions
- **Server Actions** (`"use server"`)
- Validation avec **Zod**, côté serveur obligatoirement
- Formulaires : `react-hook-form` ou formulaires natifs + `useActionState`
- Interface optimiste : `useOptimistic`
- `useTransition` pour les états d'attente
- Upload d'images : Uploadthing ou Cloudinary

### Exemple de Server Action

```ts
// src/app/actions/create-listing.ts
"use server";

import { z } from "zod";
import { revalidatePath } from "next/cache";
import { db } from "@/lib/db";

const schema = z.object({
  title: z.string().min(5).max(100),
  city: z.string().min(2),
  pricePerNight: z.coerce.number().int().positive(),
});

export async function createListing(prevState: unknown, formData: FormData) {
  const parsed = schema.safeParse(Object.fromEntries(formData));
  if (!parsed.success) {
    return { errors: parsed.error.flatten().fieldErrors };
  }

  // ownerId viendra de la session à l'étape 5
  await db.listing.create({ data: { ...parsed.data /*, ownerId, ... */ } });
  revalidatePath("/");
  return { success: true };
}
```

### À faire
1. Formulaire « Publier une annonce » avec validation et messages d'erreur.
2. Upload d'une image et enregistrement de son URL.
3. Bouton **Favori** avec mise à jour optimiste.
4. Formulaire de réservation (dates) avec vérification des chevauchements.

### Livrable
Un utilisateur peut publier une annonce, la mettre en favori et réserver.

### Checklist
- [ ] Validation Zod côté serveur
- [ ] Erreurs affichées à l'utilisateur
- [ ] Favori instantané même sur connexion lente
- [ ] Réservations qui se chevauchent refusées

### Rappel de sécurité
Une Server Action est un **point d'entrée public** : traitez-la comme une route d'API. Validez toujours les entrées et vérifiez les droits de l'appelant à l'intérieur.

---

## Étape 5 : Authentification

**Durée estimée :** 3 à 4 jours

### Objectif
Identifier les utilisateurs et protéger les données.

### Notions
- Sessions, cookies, JWT
- OAuth (Google, GitHub)
- Protection des routes (middleware / proxy selon la version) **et** vérification côté serveur
- Autorisation : authentifié ≠ autorisé

### Choix de la solution
Comparez rapidement avant de décider :

| Solution | Points forts | Points d'attention |
|---|---|---|
| **Auth.js** | Très répandu, gratuit, nombreux providers | Configuration plus manuelle |
| **Better Auth** | Moderne, orienté TypeScript | Écosystème plus récent |
| **Clerk** | Très rapide à intégrer, UI prête | Service tiers, tarification selon l'usage |

> Vérifiez l'état et la documentation actuels de chaque option : ce secteur évolue vite.

### À faire
1. Installer la solution choisie et configurer GitHub ou Google OAuth.
2. Lier les utilisateurs à la table `User`.
3. Protéger `/mes-annonces` et `/mes-reservations`.
4. Dans chaque Server Action : récupérer la session, **refuser si absente**, puis vérifier que l'utilisateur est propriétaire de la ressource.
5. Afficher l'état de connexion dans le `Header`.

### Livrable
Espaces « Mes annonces » et « Mes réservations » réservés à l'utilisateur connecté.

### Checklist
- [ ] Connexion et déconnexion fonctionnelles
- [ ] Impossible de modifier l'annonce d'autrui, même en appelant l'action directement
- [ ] Redirection vers la connexion pour les pages protégées
- [ ] Aucun secret OAuth dans le dépôt

---

## Étape 6 : Qualité et mise en production

**Durée estimée :** 4 à 5 jours

### Objectif
Rendre le projet fiable et présentable.

### À faire

**Tests**
```bash
npm i -D vitest @testing-library/react
npm i -D @playwright/test
npx playwright install
```
- Vitest : fonctions utilitaires, logique de calcul de prix, schémas Zod.
- Playwright : parcours complet (recherche → détail → connexion → réservation).

**Audit**
- Lighthouse : viser de bons scores en performance, accessibilité, bonnes pratiques, SEO.
- Accessibilité : navigation clavier, labels de formulaires, contrastes, textes alternatifs.

**Production**
- Variables d'environnement configurées sur Vercel
- `sitemap.xml` et `robots.txt` (`app/sitemap.ts`, `app/robots.ts`)
- Gestion d'erreurs et journalisation
- README soigné : description, captures d'écran, stack, installation, choix techniques

### Livrable
Un projet présentable en portfolio, avec tests et documentation.

### Checklist
- [ ] Au moins un test E2E sur le parcours principal
- [ ] Audit Lighthouse relu et corrigé
- [ ] README complet
- [ ] Aucune clé ni secret dans l'historique Git

---

## Bonus

- **Carte interactive** : Leaflet (gratuit, OpenStreetMap) ou Mapbox
- **Paiement** : Stripe en mode test
- **Internationalisation** : `next-intl` (français / anglais)
- **Animations** : Framer Motion
- **Intégration continue** : GitHub Actions (lint, tests, build)
- **Observabilité** : Sentry pour le suivi d'erreurs

---

## Pièges fréquents

| Piège | Remède |
|---|---|
| Mettre `"use client"` partout | Le garder sur les « feuilles » de l'arbre |
| Suivre un tutoriel basé sur le Pages Router | Vérifier que le tutoriel utilise `app/` |
| Faire confiance au client (prix, droits, identifiants) | Tout recalculer et vérifier côté serveur |
| Ouvrir trop de connexions à la base en dev | Singleton Prisma, base avec pooling |
| Cache inattendu (données « figées ») | Lire la doc sur le cache et la revalidation de votre version |
| Commit du fichier `.env` | `.gitignore` dès le départ ; si c'est arrivé, **révoquer les clés** |
| Images non optimisées | `next/image` avec `sizes` correct |
| Composant qui plante en `window is not defined` | Code navigateur uniquement dans un composant client / `useEffect` |

---

## Ressources

- **Documentation Next.js** : <https://nextjs.org/docs>
- **Next.js Learn** (cours officiel, dashboard) : <https://nextjs.org/learn>
- **React** : <https://react.dev>
- **Tailwind CSS** : <https://tailwindcss.com/docs>
- **Prisma** : <https://www.prisma.io/docs>
- **Zod** : <https://zod.dev>
- **Playwright** : <https://playwright.dev>
- **Frontend Mentor** (défis d'interface sans souci de droits) : <https://www.frontendmentor.io>

---

## Journal de progression

Copiez ce modèle à la fin de chaque étape.

```md
### Étape N : <titre>
- **Dates :** du JJ/MM au JJ/MM
- **Temps passé :**
- **Ce que j'ai appris :**
- **Ce qui m'a bloqué (et comment je l'ai résolu) :**
- **Ce que je referais autrement :**
- **Lien du commit / du déploiement :**
```

### Suivi global

- [ ] Étape 0 : Mise en place
- [ ] Étape 1 : Interface statique
- [ ] Étape 2 : Routing et rendu
- [ ] Étape 3 : Base de données
- [ ] Étape 4 : Mutations et formulaires
- [ ] Étape 5 : Authentification
- [ ] Étape 6 : Qualité et mise en production
- [ ] Bonus choisis : ________________

---

*Bon courage : le but n'est pas de copier Airbnb, mais de comprendre chaque brique que vous ajoutez.*
