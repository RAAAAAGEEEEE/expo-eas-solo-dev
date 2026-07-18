# RevenueCat API v2 — cheatsheet pratique

Appris en configurant une monétisation freemium (mensuel/annuel/à vie) entièrement par API, sans passer par le dashboard RevenueCat. Base URL : `https://api.revenuecat.com/v2`. Auth : `Authorization: Bearer <clé secrète sk_...>` sur chaque requête.

La clé secrète est **scopée à un seul projet** (celui depuis lequel elle a été générée) — `GET /v2/projects` avec cette clé ne renvoie que ce projet, pas les autres projets du même compte. C'est une bonne nouvelle côté sécurité : pas de risque d'affecter un autre projet par erreur avec la même clé.

## Toujours commencer par un état des lieux

Avant de créer quoi que ce soit, lister l'existant — un projet peut déjà contenir une config placeholder issue d'un onboarding automatique RevenueCat ou d'un essai avec un autre outil (vu en pratique : bundle ID `com.vibecode.monapp.xxxxx` au lieu du vrai bundle ID de l'app) :

```bash
PROJECT_ID="proj..."
SK="Bearer sk_..."

curl -s -H "Authorization: $SK" "https://api.revenuecat.com/v2/projects/$PROJECT_ID/apps"
curl -s -H "Authorization: $SK" "https://api.revenuecat.com/v2/projects/$PROJECT_ID/entitlements"
curl -s -H "Authorization: $SK" "https://api.revenuecat.com/v2/projects/$PROJECT_ID/offerings"
curl -s -H "Authorization: $SK" "https://api.revenuecat.com/v2/projects/$PROJECT_ID/products"
```

Si le `bundle_id` d'une app ne correspond pas au vrai `app.json`, ou si les `store_identifier` des produits ont un préfixe suspect, **le signaler à l'utilisateur avant de corriger** (peut-être une config qu'il a faite lui-même et dont il faut vérifier l'origine).

## Créer un entitlement

```bash
curl -s -X POST "https://api.revenuecat.com/v2/projects/$PROJECT_ID/entitlements" \
  -H "Authorization: $SK" -H "Content-Type: application/json" \
  -d '{"lookup_key": "premium", "display_name": "Premium"}'
```
Erreur `resource_already_exists` si le `lookup_key` existe déjà — normal si un onboarding automatique l'a déjà créé, pas la peine de retenter.

## Corriger le bundle_id d'une app (POST, pas PATCH)

```bash
curl -s -X POST "https://api.revenuecat.com/v2/projects/$PROJECT_ID/apps/$APP_ID" \
  -H "Authorization: $SK" -H "Content-Type: application/json" \
  -d '{"app_store": {"bundle_id": "com.vraie.bundle.id"}}'
```
Contre-intuitif : c'est un `POST` sur la ressource existante qui sert de "update", pas un `PATCH`/`PUT`.

## Créer un produit — piège sur les produits réels vs Test Store

```bash
# Produit REEL App Store (le duration vient d'Apple, pas de RevenueCat)
curl -s -X POST "https://api.revenuecat.com/v2/projects/$PROJECT_ID/products" \
  -H "Authorization: $SK" -H "Content-Type: application/json" \
  -d '{"app_id":"'$APP_ID'","store_identifier":"com.monapp.monthly","type":"subscription","display_name":"Premium Monthly"}'
```
⚠️ Ne JAMAIS inclure `"subscription": {"duration": "P1M"}` pour un produit `app_store`/`play_store` réel — l'API répond `parameter_error: "Subscription parameters are only supported for simulated store products."`. La durée n'est acceptée que pour les produits **Test Store** (`type: "test_store"` sur l'app), où là il faut la préciser explicitement.

Pour un achat unique (à vie) : `"type": "non_consumable"` et pas de bloc `subscription`.

`DELETE /v2/projects/{id}/products/{product_id}` supprime un produit — pas de `PATCH` pour modifier un `store_identifier` existant, il faut supprimer et recréer.

## Rattacher un produit à un package — chemin et forme du body à ne pas rater

Les packages sont accessibles directement sous le projet, **pas** nichés sous `/offerings/{id}/packages/{id}/` (piège fréquent en devinant depuis la structure logique) :

```bash
# Lister les produits d'un package
curl -s -H "Authorization: $SK" "https://api.revenuecat.com/v2/projects/$PROJECT_ID/packages/$PACKAGE_ID/products"

# Détacher un produit (chemin confirmé fonctionnel malgré la doc ambiguë) :
# DELETE le produit directement suffit en general (voir plus haut) ; sinon :
curl -s -X POST "https://api.revenuecat.com/v2/projects/$PROJECT_ID/packages/$PACKAGE_ID/actions/detach_products" \
  -H "Authorization: $SK" -H "Content-Type: application/json" \
  -d '{"products": [{"product_id": "prod..."}]}'

# Attacher un produit — le body attend "products" (tableau d'objets), PAS "product_ids" (tableau de strings) :
curl -s -X POST "https://api.revenuecat.com/v2/projects/$PROJECT_ID/packages/$PACKAGE_ID/actions/attach_products" \
  -H "Authorization: $SK" -H "Content-Type: application/json" \
  -d '{"products": [{"product_id": "prod...", "eligibility_criteria": "all"}]}'
```
`eligibility_criteria: "all"` est la valeur par défaut sensée pour un produit sans restriction (pas de segmentation par cohorte).

## Configurer la clé "Achat intégré" (StoreKit 2) par API — évite à l'utilisateur de coller un secret dans le dashboard

Une fois que l'utilisateur a téléchargé le `.p8` sur son disque (⚠️ voir SKILL.md — un seul téléchargement possible) et donné le chemin du fichier + le Key ID :

```bash
# Lire le contenu du .p8 directement depuis le disque de l'utilisateur, ne jamais le faire coller dans le chat
P8_CONTENT=$(cat "/chemin/vers/AuthKey_XXXXXXXXXX.p8")

curl -s -X POST "https://api.revenuecat.com/v2/projects/$PROJECT_ID/apps/$APP_ID" \
  -H "Authorization: $SK" -H "Content-Type: application/json" \
  -d "$(python3 -c "import json,sys; print(json.dumps({'app_store': {'subscription_private_key': sys.argv[1], 'subscription_key_id': sys.argv[2], 'subscription_key_issuer': sys.argv[3]}}))" "$P8_CONTENT" "$KEY_ID" "$ISSUER_ID")"
```
(Passer par un petit script pour construire le JSON évite les problèmes d'échappement des sauts de ligne du contenu `.p8` dans une chaîne shell directe.)

L'`Issuer ID` est visible en haut de la page "Achat intégré" ou "API App Store Connect" dans App Store Connect (Utilisateurs et accès → Intégrations) — il est le même pour toutes les clés d'un compte développeur, pas la peine d'en chercher un différent par clé.

## Vérifier après coup

```bash
curl -s -H "Authorization: $SK" "https://api.revenuecat.com/v2/projects/$PROJECT_ID/apps/$APP_ID"
# Champs à surveiller : app_store_connect_api_key_configured / subscription_key_configured
# → false tant que la clé Achat intégré n'est pas correctement enregistrée
```
