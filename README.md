# KIZUNA — Anime & Manga Tracker

Eine kostenlose, mobile-freundliche Web-App zum Tracking deiner Anime und Manga. Kein eigener Server und kein laufender PC nötig.

## Funktionen

- Anime und Manga **getrennt** speichern (z. B. Blue Lock Anime fertig, Manga Kapitel 338)
- Status: Am Schauen, Am Lesen, Abgeschlossen, Geplant, Pausiert, Abgebrochen
- Episode/Kapitel, optional Gesamtzahl, Notizen, optional eigenes Cover
- Schnell-Button `+1` für aktuelle Serien, Suche und Filter
- Offlinefähige PWA, zum iPhone-Homescreen hinzufügen
- JSON Export und Import als Backup
- Optional kostenloser geräteübergreifender Supabase-Sync (eigenes Konto)

**Wichtig:** Die importierten Angaben sind so übertragen, wie sie aus der Liste hervorgehen. Titel, die als gesehen abgehakt waren, sind meist als `Abgeschlossen` markiert (auch wenn eventuell nur eine Staffel fertig ist). `Baki` ist `Abgebrochen`. `One Piece` (Episode 1170), `Bleach` (Episode 142), `Gachiakuta` (Episode 7), `Blue Lock` Manga (Kapitel 338) und `Vagabond` Manga (Kapitel 87) sind aktiv. Unbekannte Episodenzahlen bleiben leer und werden nicht erfunden. Die App aktualisiert veröffentlichte Gesamtzahlen nicht automatisch, sondern lässt sie manuell bearbeiten.

## Gratis veröffentlichen (GitHub Pages)

1. Auf github.com kostenlos registrieren bzw. anmelden.
2. Ein neues **öffentliches** Repository erstellen, z. B. `kizuna`.
3. Alle Dateien **aus dem ZIP** ins Hauptverzeichnis des Repositories hochladen (`index.html`, `sw.js`, `manifest.webmanifest`, Icons, etc.).
4. Im Repository `Settings` → `Pages` → `Build and deployment` → `Deploy from a branch` → `main` → `/(root)` → `Save` auswählen.
5. Nach kurzer Wartezeit ist die Website unter `https://DEINNAME.github.io/kizuna/` verfügbar.
6. Website auf deinem iPhone in **Safari** öffnen → `Teilen` → `Zum Home-Bildschirm`. Danach sieht sie aus wie eine App.

GitHub Pages hostet statische Websites kostenlos für öffentliche Repositories. Alle Benutzerdaten bleiben bis zur Cloud-Einrichtung ausschliesslich im Browser auf deinem Gerät, **nicht** bei GitHub. Ohne Sync kann das Löschen von Safari-Websitedaten die Collection löschen. Exportiere regelmässig ein Backup.

## Optional: Kostenloser Cloud Sync mit Supabase

Ein eigenes Supabase-Free-Projekt ist für eine private Sammlung normalerweise ausreichend. Kostenlose Kontingente, Inaktivitätsregeln und Limits können sich ändern. Du musst weder einen Server mieten noch einen PC laufen lassen.

1. Kostenlos bei https://supabase.com anmelden und ein Projekt erstellen.
2. Im SQL Editor des Projekts den folgenden SQL-Block ausführen:

```sql
create table if not exists public.collections (
  user_id uuid primary key references auth.users(id) on delete cascade,
  entries jsonb not null default '[]'::jsonb,
  updated_at timestamptz not null default now()
);

alter table public.collections enable row level security;

create policy "Users see their collection" on public.collections
  for select to authenticated using ((select auth.uid()) = user_id);

create policy "Users insert their collection" on public.collections
  for insert to authenticated with check ((select auth.uid()) = user_id);

create policy "Users update their collection" on public.collections
  for update to authenticated using ((select auth.uid()) = user_id)
  with check ((select auth.uid()) = user_id);
```

Wenn die Policies schon existieren, den SQL-Block nicht nochmals ausführen (oder die alten Policies vorher löschen).

3. In Supabase unter **Authentication → URL Configuration** als `Site URL` deine GitHub-Pages-URL eintragen und dieselbe vollständige URL (mit `/kizuna/`) zu den **Redirect URLs** hinzufügen.
4. In **Authentication → Providers → Email** E-Mail-Anmeldung (OTP/Magic Link) aktiv lassen. Prüfen, dass die Bestätigungsmail auch bei dir ankommt.
5. In **Project Settings / Connect / API Keys** (je nach aktueller Supabase-Oberfläche) die **Project URL** und den öffentlichen **Publishable Key** oder Legacy **anon Key** kopieren. **Nie `service_role`, `secret` oder einen Datenbank-Passwort-Key in die Website eingeben!**
6. In KIZUNA → `Settings` → `Cloud-Sync einrichten`: URL, Publishable Key und deine E-Mail eingeben → Login-Link senden.
7. Den Link aus der Mail öffnen. Bei erfolgreicher Anmeldung werden deine Einträge mit dem Projekt synchronisiert.
8. Für ein zweites Gerät dieselbe veröffentlichte Website öffnen, dieselbe Cloud-Konfiguration und E-Mail verwenden und anmelden.

**Konflikte:** Bei der erstmaligen Verbindung mit einer bereits existierenden Cloud-Collection fragt die App, ob Cloud-Daten geladen werden sollen oder die lokalen Daten überschreiben sollen. Im normalen Betrieb verwendet diese einfache Version „letzte Speicherung gewinnt“. Nicht zwei Geräte gleichzeitig offline bearbeiten und später konkurrierend synchronisieren.

**Datenschutz:** Die eigene GitHub-Pages-Seite und der öffentliche Key sind öffentlich einsehbar. Die Datenbank ist durch aktivierte Row-Level-Security und nutzerbezogene Policies geschützt. Notizen und Titel werden an dein eigenes Supabase-Projekt geschickt, wenn du Cloud verwendest. Das Projekt sollte privat bleiben; die Datenbank ist **nicht** zur Freigabe zwischen unterschiedlichen Accounts gedacht.

## Technik

Vanilla JavaScript, HTML, CSS, Local Storage, Service Worker, optional `supabase-js` (ESM vom CDN). Keine npm-Abhängigkeiten und kein Build erforderlich. Die App funktioniert auch ohne Supabase offline nach dem ersten Laden.

## Geplante Verbesserungen

- Automatischer Metadaten-Import (AniList/MAL)
- Cover-Suche statt manuelle URLs
- Bewertungen, Favoriten, Release-Kalender
- Robustere Konfliktbehandlung beim Sync
