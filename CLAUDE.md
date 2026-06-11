# CLAUDE.md — Ecopulz Command Center (Webapp)

> Lees dit VOLLEDIG voordat je iets doet. Dit is je complete context.

---

## WAT IS DIT

Een single-file HTML/JS webapp (`index.html`, ~5014 regels) gehost op Cloudflare Pages.
URL: https://ecopulz-ai.pages.dev
Backend: Supabase project `knaohykrcxyanhidbhkt`
Dit is het Command Center voor Ecopulz — een AI-gestuurde SDR- en contentautomation-platform.

---

## HUIDIGE STAAT

**Alles wat werkt (F1-F14 volledig af):**
- SDR pipeline met 13 agents (scoring, enrichment, outreach, content, ads, recruitment)
- AFAS bidirectionele sync (elke 10 min via pg_cron)
- LinkedIn API live (OAuth, campaign push, metrics pull, content publishing + scheduling)
- Analytics dashboard
- Marketing Machine (content series, campagnes)
- Lead intake functie met Perplexity enrichment + AFAS push
- 10-staps AFAS-compliant sales funnel

**PERPLEXITY_KEY is nu live in Supabase secrets.**

---

## SPRINT 1 — WAT JIJ MOET DOEN

### Prioriteit 1: UI naar "gelikt" niveau brengen

De webapp werkt maar ziet er nog niet uit als een premium product. Maak het visueel top-tier:

1. **Design systeem consistent maken**
   - Alle cards, modals, tabellen dezelfde styling
   - Consistent gebruik van spacing, border-radius, schaduwen
   - Hover states en transitions overal smooth
   - Loading states waar nodig

2. **Dashboard verbeteren**
   - KPI cards met micro-animaties / sparklines
   - Pipeline funnel visualisatie
   - Recente activiteit feed
   - Snelle acties prominent

3. **Leads pagina**
   - AFAS badge op leads die gesyncte zijn (check `afas_bcco` of `afas_dbid` kolom)
   - Conditionele UI: outreach sectie verbergen na fase 002 (voorbij eerste contact)
   - Notities veld dat synct met AFAS (PUT KnSalesRelationOrg → Rm veld)
   - Pricing calculator fix (bestaande bug)
   - Fase-visualisatie als progress bar

4. **Content pagina**
   - Post preview cards met platform-specifieke styling
   - Content kalender view
   - Performance metrics per post

5. **Sector Radar**
   - LinkedIn URLs voor sector_reactions: gebruik `post_url` kolom direct, NOOIT reconstructie uit activity ID (19-digit IDs verliezen precisie door JS float)

6. **Algemene polish**
   - Dark mode is het thema — hou het consistent
   - Poppins font, teal (#00C9A4) als primary
   - Responsive basics (sidebar collapse werkt al)
   - Lege states voor alle secties (geen lege tabellen)
   - Error handling met gebruiksvriendelijke meldingen
   - Smooth page transitions

### Prioriteit 2: Perplexity live (Tavily weg)

Nu PERPLEXITY_KEY er is:
1. **Test lead-intake** — maak een test-lead via de UI, check of enrichment + AFAS push werkt
2. **Sector monitoring upgraden** — vervang alle Tavily calls door Perplexity in de relevante Edge Functions
3. **Tavily volledig uitfaseren** — verwijder TAVILY_API_KEY references

---

## TECHNISCHE DETAILS

### Supabase
- Project: `knaohykrcxyanhidbhkt`
- URL: `https://knaohykrcxyanhidbhkt.supabase.co`
- Gebruik `anon key` in de frontend (staat al in index.html)
- Edge Functions via: `https://knaohykrcxyanhidbhkt.supabase.co/functions/v1/{functienaam}`

### Edge Functions (deployed)
| Functie | Doel |
|---|---|
| afas-push | CC → AFAS (4-staps: KnOrg + adres + KnSalesRelationOrg + CmForecast) |
| afas-sync | Bidirectionele sync elke 10 min |
| afas-import | Eenmalige import (al gedraaid) |
| afas-test | Debug tool |
| lead-intake | Perplexity enrichment + afas-push orchestratie |

### Database Webhook
- `leads` INSERT → triggert `lead-intake` Edge Function

### Belangrijke DB tabellen
- `leads` — alle leads met AFAS sync velden (afas_bcco, afas_dbid, afas_forecast_prid)
- `tasks` — taken voor Finn/Olaf
- `content_posts` — content machine
- `sector_signals` — sector radar
- `touches` — contactmomenten
- `agents` — agent prompts en config (ALTIJD fetch via `.eq('nummer', X).single()`)
- `content_master` — master content library
- `content_ideas` — gegenereerde ideeën
- `agent_runs` — audit trail
- `fase_mapping` — AFAS fase mapping (kolom heet `naam`, niet `afas_naam`)
- `organisaties` — bedrijven met verrijkte data
- `api_tokens` — OAuth tokens (LinkedIn etc.)

### AFAS regels
- Token = `btoa(full XML string)` als Bearer
- Environment: 33694
- Push flow: POST KnOrg → PUT KnOrg (adres) → POST KnSalesRelationOrg → POST/PUT CmForecast
- Forecast update: PUT CmForecast met PrId + SqNo:1
- Notities: PUT KnSalesRelationOrg met Rm veld
- EmId's: Finn='1000577', Olaf='1000003'

### LinkedIn regels
- NOOIT URLs reconstrueren uit activity IDs — gebruik altijd `post_url` kolom
- 19-digit IDs verliezen precisie door JavaScript float limitatie

### CSS variabelen (bestaand thema)
```css
--teal: #00C9A4;
--salmon: #FF8072;
--blue: #A8E0E0;
--yellow: #F5C542;
--red: #E53535;
--bg: #16181a;
--card: rgba(255,255,255,.045);
--t1: #e8ece8; /* primary text */
--t2: #8ca48c; /* secondary text */
--t3: #55665a; /* tertiary text */
--f: 'Poppins', sans-serif;
--r: 8px;
```

---

## WERKWIJZE

- **Altijd complete code blokken** — geen partial snippets of "voeg dit toe" instructies
- **Bij grote wijzigingen**: maak eerst een backup kopie, werk dan sectie voor sectie
- **Test na elke wijziging** in de browser
- **Commit na elke werkende feature** met duidelijke message
- **Agent prompts staan in de `agents` tabel** — nooit hardcoden in Edge Functions
- **sed vermijden** voor XML/speciale tekens — gebruik bestandsbewerking tools

---

## WAT JE NIET MOET DOEN

- Geen nieuwe tabellen aanmaken (schema is compleet)
- Geen Edge Functions deployen zonder te testen
- Geen API keys hardcoden
- Geen Supabase secrets met `SUPABASE_` prefix (reserved)
- Geen LinkedIn URLs reconstrueren uit IDs
- Niet de AFAS push flow veranderen (die werkt)

---

*Sprint 1 focus: maak het MOOI en test Perplexity. Geen nieuwe features, geen nieuwe APIs.*
