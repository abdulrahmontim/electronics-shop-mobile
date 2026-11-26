# AGENTS.md — Bench Supply mobile

## Scope

This repository contains the Expo/React Native Android client for Bench Supply. The web shop lives in a separate repository, `electronics-shop`. Keep the repositories independent: no Next.js imports, no web CSS, no shared source tree, and no second backend.

The mobile app is a native client for the same shop: same Supabase project, same users, same products, same cart, and same orders. Do not create mobile-only accounts, a mobile-only catalogue, fake production data, or a separate order system.

## Stack

- Expo SDK 57, React Native, Expo Router, TypeScript strict, npm.
- `@supabase/supabase-js` with public `EXPO_PUBLIC_*` values only.
- `expo-web-browser` plus AsyncStorage for Supabase PKCE sign-in.
- Native components only: `View`, `Text`, `Image`, `Pressable`, `TextInput`, `ScrollView`, `FlatList`, `ActivityIndicator`, and similar. No DOM elements, CSS, WebView as the main app, or browser-only APIs.
- Keep money as integer naira and format it in one shared helper.

## Layout

Keep the existing structure:

- `src/app/`: root layout, `(tabs)` for Shop/Cart/Orders/Account, `product/[slug]`, `order/[id]`, `checkout.tsx`, and `login.tsx`.
- `src/components/`: native buttons, fields, chips, messages, product art, product cards, quantity stepper, and layout.
- `src/constants/theme.ts` for colours, spacing, and type scale.
- `src/lib/`: Supabase, auth, session, cart, products, orders, checkout, config, format, and resistor helpers.
- `assets/`, `app.json`, `eas.json`, `.env.example`, `README.md`, `package.json`, and `tsconfig.json`.

Route names are part of the contract. Keep them stable unless the owner asks for a change.

## Shared backend contract

- The canonical SQL lives in the web repository. Do not duplicate or rewrite it here. The owner must apply `supabase/schema.sql`, `supabase/seed.sql`, and `supabase/cart.sql` before mobile data can load.
- Read products from the same public Supabase `products` table. Do not hardcode products or prices.
- Read orders through Supabase RLS. A customer may see only their own orders.
- Place orders only by POSTing to the deployed web app's `/api/checkout`. Never insert into `orders` or `order_items` from the phone, and never call Mailgun from this app.
- Keep the checkout payload compatible with the web route: delivery fields plus an array of `product_id` and `quantity`.
- Only `EXPO_PUBLIC_SUPABASE_URL`, `EXPO_PUBLIC_SUPABASE_ANON_KEY`, and `EXPO_PUBLIC_API_URL` may be public. Never put service-role keys, Mailgun keys, database passwords, or other server secrets in the bundle.
- `EXPO_PUBLIC_API_URL` must point to a reachable deployed web app, not `localhost`.

## Cart contract

- The database is authoritative for signed-in carts.
- Signed-out carts may live in AsyncStorage, but they must be merged into `cart_items` on sign-in and then cleared locally.
- Quantities stay between 1 and 20. Duplicate products merge into one line.
- Refresh from the backend after add, quantity change, remove, sign-in, cart open, and app foregrounding.
- Never let stale local state overwrite the database. Show clear messages when a product is unavailable or a sync fails.
- Preserve both directions of the shared-cart behaviour: web to mobile and mobile to web.

## Authentication

- Use Supabase Auth with PKCE, the same project as the web app, and no mobile-only user system.
- Store the PKCE verifier and session in AsyncStorage. Never store passwords.
- Build the redirect URI with Expo Linking and ensure it is allowlisted in Supabase Auth Redirect URLs.
- A development build or APK is required for the app's custom scheme. Expo Go is suitable only for limited checks because it cannot register that scheme.
- Google Cloud Console normally needs no mobile-specific change because the OAuth handshake starts at Supabase.
- Never log access tokens, refresh tokens, OAuth codes, or secrets.

## UX

- Use the web shop as the visual and functional reference, then adapt it for touch. Do not shrink the desktop layout or invent a new identity.
- Preserve the Bench Supply palette, resistor motif, typography, plain sentence case, and action labels.
- Design for varied phone sizes with flexible layouts, safe areas, scrolling, text wrapping, comfortable touch targets, and keyboard handling. Do not tune the app for one device or hardcode screen widths.
- Provide loading, empty, and error states for products, product detail, auth, cart, checkout, and orders. Errors should say what went wrong and what to do next.
- Keep navigation obvious and reachable. Use accessibility labels where icons carry meaning.

## Working rules

- Inspect the existing code before editing. Keep unrelated work. Delete nothing unrelated.
- Commit after each major step. Never push, deploy, force-reset Git, or delete data.
- Use placeholders for credentials. Never commit secrets or a real `.env` file.
- Do not suppress type errors with `any` just to pass the build.
- Verify with `npx tsc --noEmit`, Expo start/run:android where possible, and the shared-cart checks in both directions. Confirm a second account cannot see the first account's cart or orders.
- Ask the owner only when genuinely blocked, such as by a credential you cannot create here.
- Report changes, verification results, dependencies added, manual setup required, and anything not tested against real services.
