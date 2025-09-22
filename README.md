# Modern_reactive_web_app

<!-- TOC -->
  * [WebFlux](#F⃣-webflux)
  * [2️⃣ Project Reactor](#2-project-reactor)
  * [3️⃣ Svelte](#3-svelte)
  * [Jak to zapadá do sebe při striktním oddělení front-end a back-end](#jak-to-zapadá-do-sebe-při-striktním-oddělení-front-end-a-back-end)
  * [Příklad architektury](#příklad-architektury)
  * [1️⃣ Dnešní běžné kombinace pro oddělený FE/BE](#1-dnešní-běžné-kombinace-pro-oddělený-febe)
  * [2️⃣ Co se stane s vaší současnou technologií](#2-co-se-stane-s-vaší-současnou-technologií)
  * [3️⃣ Zásadní rozdíl v přístupu](#3-zásadní-rozdíl-v-přístupu)
  * [4️⃣ WebSocket vs. REST](#4-websocket-vs-rest)
  * [5️⃣ Typická moderní kombinace pro nový projekt](#5-typická-moderní-kombinace-pro-nový-projekt)
  * [**Fáze 1 – Příprava backendu na API-first**](#fáze-1--příprava-backendu-na-api-first)
  * [**Fáze 2 – Proof of Concept pro nový frontend**](#fáze-2--proof-of-concept-pro-nový-frontend)
  * [**Fáze 3 – Paralelní přepis UI**](#fáze-3--paralelní-přepis-ui)
  * [**Fáze 4 – Plný přechod a rozšíření**](#fáze-4--plný-přechod-a-rozšíření)
  * [**Vizualizace postupu**](#vizualizace-postupu)
  * [**FÁZE 1 – API-first příprava backendu**](#fáze-1--api-first-příprava-backendu)
  * [**FÁZE 2 – Proof of Concept pro Svelte**](#fáze-2--proof-of-concept-pro-svelte)
  * [**FÁZE 3 – Paralelní přepis UI**](#fáze-3--paralelní-přepis-ui-1)
  * [**FÁZE 4 – Plný přechod a rozšíření**](#fáze-4--plný-přechod-a-rozšíření-1)
  * [**Výsledná architektura**](#výsledná-architektura)
  * [**Fáze 1 – Dnešní stav (JSP monolit)**](#fáze-1--dnešní-stav-jsp-monolit)
  * [**Fáze 2 – Přidání API a první SPA modul**](#fáze-2--přidání-api-a-první-spa-modul)
  * [**Fáze 3 – Paralelní běh starého a nového UI**](#fáze-3--paralelní-běh-starého-a-nového-ui)
  * [**Fáze 4 – Plný přechod (oddělený FE/BE)**](#fáze-4--plný-přechod-oddělený-febe)
  * [1️⃣ Klasické MVC (servlet) vs. WebFlux](#1-klasické-mvc-servlet-vs-webflux)
  * [2️⃣ Jak to funguje v reaktivní aplikaci](#2-jak-to-funguje-v-reaktivní-aplikaci)
  * [3️⃣ Kam session ukládat v produkci](#3-kam-session-ukládat-v-produkci)
  * [4️⃣ Dopad na SPA aplikaci](#4-dopad-na-spa-aplikaci)
<!-- TOC -->


Jen výtah informací z ChatGPT ohledně toho jak nejlépe vytvořit moderní webovou aplikaci, kde je striktně rozdělený back-end a front-end.

Pojďme si to rozplést, protože **WebFlux**, **Reactor** a **Svelte** se pohybují v úplně jiných vrstvách moderní webové aplikace, a i když spolu mohou spolupracovat, jejich role jsou odlišné.

---

## 1️⃣ WebFlux

* **Co to je:** Modul Spring Frameworku pro **reaktivní vývoj webových aplikací a API**.
* **Účel:** Slouží k implementaci **serverové logiky** (back-end) na bázi **reaktivního programování**.
* **Charakteristika:**

  * Nepoužívá tradiční **Servlet API**, ale **Netty** (nebo jiný ne-blocking server).
  * Podporuje jak **REST API**, tak i **server-sent events (SSE)** nebo **WebSockety**.
  * Umí zpracovávat velké množství souběžných požadavků bez blokování vláken.
* **Kdy použít:**

  * Pokud očekáváš vysokou zátěž (IO-bound úlohy, streamování dat, chaty, push notifikace).
  * Pokud chceš psát v **reaktivním stylu** místo klasického imperativního kódu.

---

## 2️⃣ Project Reactor

* **Co to je:** **Základní reaktivní knihovna** (od Pivotal, nyní VMware) pro Java.
* **Účel:** Implementace **Reactive Streams** specifikace v Javě.
* **Charakteristika:**

  * Poskytuje hlavní typy:

    * `Mono<T>` → reaktivní publisher pro **0 nebo 1 prvek**.
    * `Flux<T>` → reaktivní publisher pro **0 až N prvků**.
  * Nabízí operátory pro mapování, filtrování, slučování, zpracování chyb atd.
* **Vztah k WebFluxu:**

  * **WebFlux je postaven na Reactoru** – to znamená, že když píšeš kontroler ve WebFluxu, vracíš `Mono` nebo `Flux`.

---

## 3️⃣ Svelte

* **Co to je:** **Frontendový JavaScript/TypeScript framework** (něco jako React, Vue nebo Angular).
* **Účel:** Starat se o **zobrazení (UI)** a interakci uživatele v prohlížeči.
* **Charakteristika:**

  * Svelte je tzv. **compiler framework** – už při buildu přeloží komponenty do čistého JS, takže výsledná aplikace běží rychleji a bez velkého runtime frameworku.
  * Umožňuje reaktivní vazby přímo v syntaxi (`$:` reaktivní proměnné).
  * Hodí se na **Single Page Application (SPA)** i **Server-Side Rendering (SSR)**.
* **Vztah k WebFlux/Reactor:**

  * **Žádný přímý** – Svelte je **frontend**, zatímco WebFlux/Reactor jsou **backend**.
  * Ale můžou spolu komunikovat přes **REST API** nebo **GraphQL API** (nebo WebSocket).

---

## Jak to zapadá do sebe při striktním oddělení front-end a back-end

Když stavíš rozsáhlou aplikaci s odděleným front-endem a back-endem:

**📦 Back-end (Java, Spring WebFlux + Reactor)**

* Implementuješ API (`/api/...`) pro CRUD operace, autentizaci, streamování dat atd.
* Používáš reaktivní přístup → méně blokování, vyšší propustnost.
* API může být REST, GraphQL nebo event-driven (např. Kafka).

**🎨 Front-end (Svelte)**

* Načítá data z API pomocí `fetch()` nebo knihovny typu Axios.
* Zobrazuje data uživateli a reaguje na jeho akce.
* Může běžet úplně odděleně a komunikovat jen přes HTTP(S).

---

## Příklad architektury

```
[ Prohlížeč ]
      |
      |  HTTP(S) JSON / GraphQL / WebSocket
      v
[ Frontend: Svelte ]
      |
      |  API volání
      v
[ Backend: Spring WebFlux (Reactor) ]
      |
      |  Přístup k DB / Messaging / další služby
      v
[ Databáze / Kafka / Redis ]
```

---

📌 **Shrnutí:**

* **Reactor** = knihovna pro reaktivní proudy v Javě.
* **WebFlux** = Spring modul pro tvorbu reaktivních webových API, postavený na Reactoru.
* **Svelte** = frontendový framework pro tvorbu UI, běží v prohlížeči.
* **Svelte** není obdoba WebFlux/Reactor – je to **jiná vrstva** aplikace.
* Spolupracují tak, že **Svelte** volá API napsané ve **WebFluxu**.

---

Jestli chceš, můžu ti připravit **konkrétní startovní projekt**, kde bude **back-end ve WebFluxu** a **front-end ve Svelte**, aby spolu komunikovaly přes REST API a WebSockety.
To ti ukáže, jak celý ekosystém funguje pohromadě.

> OK a jaká je v dnešním kontextu nejpoužívanější kombinace? Jsem součástí týmu vývojářů, kteří chtějí postavit novou verzi webové aplikace zcela od základů a to tak, že rozdělíme přes API striktně back-end a front-end část. Nyní používáme na > back-end JAVA + Spring (ne SpringBoot) a pro front-end JSP stránky, vlastní CSS a javascript kód a nějaké front-end knihovny/technologie/frameworky jako např. bootstrap, jquery, datatables, webpack, charts.js, tinymce, bootbox, grunt, eaflet, ... a vlastní JSTL tagy. Toto všechno by v nové verzi s web-socket odpadlo?

Rozumím – momentálně máte **full-stack monolit v Javě** s JSP, JSTL, jQuery a hromadou knihoven přímo napojených na backend přes servletový rendering.
To, co popisuješ, se dnes označuje jako **server-side rendered UI** s **Java EE stylem**.

Pokud chcete **striktní oddělení front-end a back-end** a moderní architekturu, znamená to **úplnou změnu filozofie** vývoje.
Pojďme si to projít bod po bodu.

---

## 1️⃣ Dnešní běžné kombinace pro oddělený FE/BE

V enterprise i startup světě se teď nejčastěji setkáš s tímto vzorcem:

**Backend (API)**

* **Java + Spring Boot** (většina týmů dnes přechází na Boot i z čistého Springu kvůli jednodušší konfiguraci).
* API přes **REST** nebo **GraphQL**.
* Reaktivní varianta: **Spring WebFlux + Reactor** (pokud je potřeba vysoká propustnost, streaming, event-driven komunikace).
* Klasická varianta: **Spring MVC** (synchronní model, jednodušší pro CRUD API).
* WebSocket/SSE jen pro **real-time** části (chaty, notifikace, live dashboardy).

**Frontend (SPA)**

* **React** (nejrozšířenější v enterprise projektech, velká komunita, hodně knihoven).
* **Angular** (populární hlavně v korporacích, stabilní, silná typová kontrola přes TypeScript).
* **Vue** (oblíbené pro menší až střední projekty, rychlý vývoj, snadnější nástup).
* **Svelte** (novější, menší runtime, výborný výkon, roste popularita, ale menší ekosystém než React).
* Stylování: **Tailwind CSS** nebo moderní CSS moduly místo ručního psaní obrovských CSS souborů.

---

## 2️⃣ Co se stane s vaší současnou technologií

Pokud přejdete na **oddělený front-end**:

* **JSP, JSTL tagy, jQuery** – kompletně zmizí.
  UI už nebude generovat server, ale JavaScript aplikace běžící v prohlížeči.
* **Bootstrap, Charts.js, DataTables, TinyMCE** – zůstanou **jen pokud je frontend chce použít**.
  Třeba v Reactu můžeš použít Charts.js nebo jiný charting library. Bootstrap můžeš nahradit Tailwindem.
* **Webpack, Grunt** – dnes se spíš používají modernější buildery:

  * **Vite** (extrémně rychlý, používá se v Svelte, Vue, React projektech).
  * **esbuild** nebo **Rollup** pro bundlování.
* **WebSocket** – ne nahradí všechno, ale použije se **tam, kde je potřeba real-time** komunikace.
* **Vlastní CSS a JS** – stále můžeš mít, ale logika a struktura UI se přesune do komponent frameworku (Svelte, React, …).

---

## 3️⃣ Zásadní rozdíl v přístupu

Dnes:

```
Prohlížeč → HTTP request → Java Servlet → JSP (HTML + JSTL + data) → Odpověď
```

Zítra:

```
Prohlížeč → načte SPA (React/Svelte/Angular) → API volání (REST/GraphQL/WebSocket) → JSON data
```

Tzn. server přestane generovat HTML pro každou stránku – bude posílat **jen data**.

---

## 4️⃣ WebSocket vs. REST

* **REST/GraphQL** – stále hlavní způsob komunikace mezi FE a BE (CRUD operace, business logika).
* **WebSocket/SSE** – jen pro real-time části (např. když chcete posílat data okamžitě bez polling).

Nepředstavuj si to tak, že **WebSocket nahradí REST** – v 90 % případů ho budeš používat jen doplňkově.

---

## 5️⃣ Typická moderní kombinace pro nový projekt

Pokud bych dnes začínal od nuly v týmu, který je zvyklý na Javu, doporučil bych:

* **Backend:**

  * Java 21+
  * Spring Boot 3.x
  * Spring WebFlux (pokud chcete jít do reaktivního modelu hned), jinak klasický Spring MVC.
  * Spring Security (JWT tokeny pro FE autentizaci).
  * API: REST (možná GraphQL pro složité datové dotazy).
  * WebSocket jen tam, kde má smysl.

* **Frontend:**

  * Svelte + TypeScript + Vite (rychlý, jednoduchý, moderní) **nebo** React + TypeScript.
  * Tailwind CSS na stylování.
  * Knihovny podle potřeby (Charts.js, DataTables alternativy, Rich text editor, Map komponenty).

---

💡 **Důležité:** přechod na SPA architekturu znamená, že spousta věcí, co dnes řeší JSP (např. vykreslení tabulky), se přesune do JavaScriptové aplikace – a backend bude jen poskytovat JSON data.

---

Pokud chceš, můžu ti připravit **tabulku staré vs. nové technologie** a ukázat, **co přesně nahradit čím** při migraci, aby to sedělo na váš případ.
To by ti pomohlo vysvětlit i týmu, co všechno se změní a proč.

> Super dej mě tabulku

Tady máš přehlednou tabulku **"Staré → Nové"** pro přechod z JSP monolitu na moderní oddělený FE/BE stack:

| **Současná technologie**               | **Nová alternativa**                                                              | **Poznámka / Proč změna**                                                                          |
| -------------------------------------- | --------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| **JSP + JSTL**                         | **Svelte** *(nebo React/Angular/Vue)*                                             | HTML se už negeneruje na serveru – UI běží jako SPA v prohlížeči, data získává z API.              |
| **Java Servlet MVC (Spring bez Boot)** | **Spring Boot 3.x + WebFlux nebo Spring MVC**                                     | Boot výrazně zjednoduší konfiguraci, WebFlux přinese reaktivní IO pro vyšší škálovatelnost.        |
| **jQuery**                             | **Čistý JS/TypeScript** nebo frameworkové DOM API                                 | Moderní JS už řeší většinu toho, co dřív jQuery – plus UI frameworky mají vlastní reaktivní model. |
| **Bootstrap**                          | **Tailwind CSS** *(nebo moderní CSS framework)*                                   | Tailwind umožňuje rychlejší a konzistentnější stylování, bez nutnosti přepisovat CSS.              |
| **DataTables**                         | **Tabulková knihovna pro daný framework** (např. TanStack Table pro React/Svelte) | Přizpůsobené pro SPA – podporuje stránkování, filtrování, server-side loading.                     |
| **Charts.js**                          | **Charts.js nebo ApexCharts** (verze pro daný framework)                          | Můžeš zachovat Charts.js, jen se integruje jinak.                                                  |
| **TinyMCE**                            | **TinyMCE / Quill / TipTap** (jako komponenta pro framework)                      | Zachováš nebo vybereš nový editor, ale integruješ ho do SPA.                                       |
| **Bootbox**                            | **Modální komponenty frameworku**                                                 | V SPA se modal okna řeší jako komponenty, ne jako globální JS pluginy.                             |
| **Leaflet**                            | **Leaflet + wrapper pro framework**                                               | Můžeš zachovat Leaflet, ale použiješ ho přes komponentový wrapper.                                 |
| **Vlastní CSS**                        | **CSS moduly nebo Tailwind**                                                      | Komponentové UI má oddělené styly, lepší údržba a méně kolizí názvů tříd.                          |
| **Grunt/Webpack**                      | **Vite** *(nebo Webpack 5, ale Vite je rychlejší)*                                | Vite má rychlejší build a HMR (hot module reload) – ideální pro vývoj SPA.                         |
| **Server-side session**                | **JWT / OAuth 2.0 / OpenID Connect**                                              | Autentizace se přesune na tokeny – funguje i pro API-only backend.                                 |
| **Formuláře generované na serveru**    | **Formuláře v SPA + volání API**                                                  | Logika validace i odeslání běží v prohlížeči, server dostává čistá JSON data.                      |
| **REST jen pro AJAX v JSP**            | **REST/GraphQL jako hlavní komunikační vrstva**                                   | API se stává primárním rozhraním mezi FE a BE.                                                     |
| **Bez WebSocketů**                     | **WebSocket / SSE pro real-time**                                                 | Jen pro části aplikace, kde má smysl (notifikace, live data).                                      |

---

📌 **Co z toho plyne pro váš tým:**

1. Backend zůstane v Javě a Springu, ale **odlehčí se od generování UI** – bude to čisté API.
2. Frontendový tým bude plně samostatný, vyvíjí SPA, která jen volá API.
3. Deployment může být oddělený – API běží na jednom serveru, SPA se nasazuje třeba přes CDN.
4. WebSocket **nenahradí REST**, ale doplní ho tam, kde je třeba okamžitá aktualizace.

---

Jestli chceš, můžu ti na základě téhle tabulky připravit **migrační roadmapu**, kde budou konkrétní kroky **v jakém pořadí co nahradit**, aby se dalo přecházet postupně a ne „big bangem“.
To by ti pomohlo i při plánování práce mezi frontend a backend částí.

> Super sem s tím!

Tak jo – tady máš **migrační roadmapu** na přechod z vašeho současného JSP monolitu na moderní oddělený FE/BE stack.
Rozdělil jsem ji do **4 hlavních fází**, aby to bylo postupné a tým to zvládl bez šoku.

---

## **Fáze 1 – Příprava backendu na API-first**

🎯 Cíl: Backend umí poskytovat API nezávisle na JSP, ale staré UI zatím funguje.

1. **Analýza současných JSP** – zjistit, jaká data se kde načítají, které části aplikace volají backend.
2. **Vyčlenit business logiku** z JSP a kontrolerů do service vrstvy (pokud to tak už není).
3. **Přidat REST endpointy** (Spring MVC nebo WebFlux) pro vybrané části aplikace – API bude vracet JSON místo HTML.
4. **Zavést DTO** (Data Transfer Objects) pro API, aby nebyly v API přímo JPA entity.
5. **Připravit autentizaci přes JWT** (paralelně se stávající session-based přihlášením).
6. **Zavést CORS konfiguraci** pro budoucí oddělený frontend.

---

## **Fáze 2 – Proof of Concept pro nový frontend**

🎯 Cíl: Malá část UI běží jako moderní SPA vedle starého systému.

1. Vybrat **frontendový framework** (např. Svelte + TypeScript + Vite).
2. Vytvořit **malou samostatnou SPA aplikaci** (např. jen dashboard nebo formulář).
3. SPA volá nově vytvořené REST API.
4. Backend nasadit API na `/api/...`, frontend jako statické soubory z jiného adresáře/domény.
5. Nasadit POC do testovacího prostředí a získat zpětnou vazbu od týmu i uživatelů.

---

## **Fáze 3 – Paralelní přepis UI**

🎯 Cíl: Většina aplikace se přesune do SPA, backend je už čisté API.

1. Po modulech nahrazovat JSP částmi SPA:

   * Začít od **méně kritických modulů** (rychlá zkušenost týmu).
   * Postupně přepisovat i složitější stránky (formuláře, tabulky, grafy).
2. Každý nový modul frontendu volá výhradně REST/GraphQL API.
3. Přenést všechny komponenty (grafy, mapy, tabulky) do SPA s využitím moderních knihoven.
4. Upravit autentizaci – staré session-based přihlášení vypnout, všude používat JWT.
5. Zavést **role-based access control** v API, protože už nebude UI kontrola na serveru.

---

## **Fáze 4 – Plný přechod a rozšíření**

🎯 Cíl: Aplikace běží plně odděleně, API je připravené i pro jiné klienty.

1. **Odstranit JSP a starý servlet rendering**.
2. Backend funguje čistě jako API server (REST/GraphQL + případně WebSocket/SSE).
3. SPA nasazovat samostatně (např. na CDN, Netlify, Vercel), API běží odděleně (Java server).
4. Přidat WebSocket/SSE pro real-time funkcionalitu (notifikace, live data).
5. Nasadit CI/CD pipeline pro samostatné nasazování FE a BE.
6. Připravit API dokumentaci (OpenAPI/Swagger) – pro budoucí integrace.

---

## **Vizualizace postupu**

```
Fáze 1: Starý JSP + API vrstvy (příprava)
Fáze 2: API + první SPA modul
Fáze 3: API + většina SPA modulů
Fáze 4: Čisté API + kompletní SPA
```

---

💡 **Tip pro váš tým:**
Během **Fáze 1 a 2** budete muset udržovat dva způsoby komunikace (HTML render + API).
Jakmile přejdete do **Fáze 3**, doporučuji už backend vyvíjet **pouze jako API** a veškeré UI změny dělat už jen ve SPA.
Tak minimalizujete riziko duplicitní práce.

---

Můžu ti k tomu teď připravit **detailní technologický stack pro každou fázi** – tzn. jaké konkrétní knihovny, konfigurace a postupy použít, aby to sedělo pro **Java + Spring Boot WebFlux + Svelte** kombinaci.
To by vám ušetřilo hodně času při rozhodování.

> Ano prosím

Perfektní – tady máš **technologický stack + doporučené postupy pro každou fázi** přechodu na kombinaci **Java + Spring Boot WebFlux + Svelte**.
Budu brát v potaz, že přecházíte z čistého Springu (bez Bootu) s JSP na oddělený FE/BE.

---

## **FÁZE 1 – API-first příprava backendu**

🎯 Cíl: Zprovoznit REST API a oddělit logiku od JSP.

**Backend stack**

* **Java 21+** (výhody: výkon, virtual threads, nové jazykové featury)
* **Spring Boot 3.x** (přechod z čistého Springu – méně XML, více konvencí)
* **Spring WebFlux** (reaktivní, non-blocking)
* **Spring Data R2DBC** *(pokud chcete reaktivní přístup k DB)* nebo JPA *(pokud zatím zůstanete u synchronního přístupu)*
* **Spring Security** (JWT autentizace)
* **MapStruct** (mapování Entity <-> DTO)
* **Lombok** *(jen pokud nechcete psát boilerplate)*

**Postupy**

1. Zavést `/api/...` namespace pro všechny nové REST endpointy.
2. Používat DTO objekty, nikdy neposílat entity přímo.
3. Přidat `springdoc-openapi` pro Swagger UI dokumentaci API.
4. Připravit JWT autentizaci (login endpoint vrací token).
5. Nastavit CORS pro doménu budoucího FE.

---

## **FÁZE 2 – Proof of Concept pro Svelte**

🎯 Cíl: Ověřit komunikaci mezi Svelte a WebFlux API.

**Frontend stack**

* **SvelteKit** (Svelte s routerem a SSR podporou, build přes Vite)
* **TypeScript** (typová kontrola už od začátku)
* **Tailwind CSS** (rychlé stylování)
* **Axios** (nebo Fetch API pro HTTP volání)
* **ESLint + Prettier** (formátování a linting kódu)
* **Vitest** (testy pro komponenty)

**Postupy**

1. Vytvořit samostatný repo pro frontend (oddělené od backendu).
2. Vytvořit `.env` pro API endpoint (např. `VITE_API_URL=http://localhost:8080/api`).
3. Vytvořit jednoduchou stránku (např. dashboard), která načte data z API a zobrazí je v tabulce.
4. Otestovat JWT login – uložit token v `localStorage` nebo `sessionStorage`.
5. Zajistit, aby FE běžel na jiné portu než BE (např. FE: 5173, BE: 8080).

---

## **FÁZE 3 – Paralelní přepis UI**

🎯 Cíl: Nahradit JSP moduly SPA komponentami.

**Backend stack – stejné jako Fáze 1**, ale:

* Zavést **role-based access control** v API (`@PreAuthorize`).
* Přidat WebSocket endpointy (Spring WebFlux + RSocket nebo STOMP) pro real-time data.
* Zavést **API versioning** (např. `/api/v1/...`).

**Frontend stack – stejné jako Fáze 2**, ale přidat:

* **TanStack Table** (pro datové tabulky – stránkování, filtrování)
* **ApexCharts** nebo **Charts.js** (grafy)
* **TinyMCE / TipTap / Quill** (rich text editor)
* **Svelte-Leaflet** (pokud používáte mapy)
* **Socket.IO klient** nebo **native WebSocket API** pro real-time data
* **Zustand / Svelte store** pro globální stav aplikace

**Postupy**

1. Každý modul FE má vlastní Svelte route.
2. Komponenty organizovat do `/lib/components` a `/lib/services` (API volání).
3. Backend validuje data přes Bean Validation (`@Valid`, `@NotNull`).
4. Testovat API samostatně přes Postman/Insomnia ještě před integrací do FE.
5. Pravidelně nasazovat na testovací prostředí, aby oba týmy viděly postup.

---

## **FÁZE 4 – Plný přechod a rozšíření**

🎯 Cíl: Oddělený deployment, CI/CD a rozšíření o real-time funkce.

**Backend stack – rozšíření**

* **Spring Cloud Gateway** (API gateway, pokud bude více mikroslužeb)
* **Spring Cloud Config** (centrální konfigurace)
* **Micrometer + Prometheus + Grafana** (monitoring API)
* **Keycloak** nebo **Auth0** (pokud chcete enterprise SSO/OIDC login)

**Frontend stack – rozšíření**

* **Sentry** (chybový monitoring)
* **Playwright / Cypress** (end-to-end testy)
* **PWA podpora** (offline režim, push notifikace)
* **Lazy loading** modulů pro rychlejší start aplikace

**Postupy**

1. Nasazovat FE na CDN (Netlify, Vercel, Cloudflare Pages).
2. API běží samostatně (např. v Kubernetes nebo jako Docker kontejner).
3. CI/CD pipeline: při pushi na main → build → deploy na test/produkci.
4. Dokumentace API veřejně dostupná (Swagger/OpenAPI).
5. Přidat real-time notifikace přes WebSockety (SSE pro jednodušší integraci).

---

## **Výsledná architektura**

```
[ Prohlížeč – SPA (SvelteKit) ]
        |
        | HTTPS REST / GraphQL
        v
[ API Gateway / Spring Boot WebFlux ]
        |
        +--> Business logika (Services)
        +--> DB (R2DBC / JPA)
        +--> WebSocket/SSE
```

---

Jestli chceš, můžu ti teď nakreslit **diagram architektury migrace**, který vizuálně ukáže,
jak se aplikace postupně transformuje z JSP monolitu na čisté API + SPA.
To by bylo ideální i pro interní prezentaci týmu a vedení.

> Ano prosím

Skvěle – tady máš **diagram architektury migrace** v textové podobě (ASCII) i s vysvětlením, jak se aplikace krok po kroku změní.
Můžeme ho později převést i do vizuální grafiky pro prezentaci.

---

## **Fáze 1 – Dnešní stav (JSP monolit)**

```
[ Prohlížeč ]
     |
     |  HTTP request (HTML)
     v
[ Java + Spring MVC (bez Bootu) ]
     |
     +--> JSP + JSTL (UI generování)
     +--> Business logika
     +--> DB přístup
```

* HTML se generuje přímo na serveru.
* jQuery/Bootstrap jsou součástí JSP šablon.
* API vrstvy prakticky neexistují (kromě pár AJAX endpointů).

---

## **Fáze 2 – Přidání API a první SPA modul**

```
[ Prohlížeč ]
     |                           [ Prohlížeč ]
     |  HTML (staré JSP)              |  SPA modul (Svelte)
     v                                 |  → Volá API
[ Java + Spring Boot + WebFlux ] <----+
     |
     +--> /api/... (JSON)
     +--> JSP pro staré části
     +--> Business logika
     +--> DB
```

* Backend umí obojí: staré JSP i nové JSON API.
* Malý modul SPA běží samostatně a komunikuje s API.
* JWT autentizace zavedena pro nový SPA modul.

---

## **Fáze 3 – Paralelní běh starého a nového UI**

```
[ Prohlížeč – SPA (SvelteKit) ]
       |
       | HTTPS (JSON)
       v
[ Java + Spring Boot + WebFlux ]
       |
       +--> /api/v1/... (JSON)
       +--> Business logika
       +--> DB
[ Prohlížeč – Staré JSP moduly ]
       |
       v
(Dočasně z backendu)
```

* Většina modulů už běží ve SPA.
* Staré JSP mizí postupně.
* API je hlavní způsob komunikace.

---

## **Fáze 4 – Plný přechod (oddělený FE/BE)**

```
[ Prohlížeč – SPA (SvelteKit) ]
       |
       | HTTPS REST / WebSocket
       v
[ API Gateway / Spring Boot WebFlux ]
       |
       +--> Microservices / Services
       +--> DB (R2DBC/JPA)
       +--> WebSocket/SSE
```

* Žádné JSP, žádný server-side rendering v Javě.
* Frontend běží samostatně (CDN / Netlify / Vercel).
* Backend běží čistě jako API server, snadno škálovatelný.
* Možnost přidat další klienty (mobilní aplikace, integrace).

---

🎯 **Výhody prezentace pro tým / vedení**

* Ukazuje **jasný postup** bez „big bang“ přechodu.
* Zdůrazňuje **oddělení odpovědností** – FE tým má vlastní kód, BE tým API.
* Umožňuje **paralelní vývoj** a rychlé nasazování nových částí.

---

Pokud chceš, můžu ti z toho hned udělat **grafickou verzi v moderním stylu** (barevné bloky, šipky, legendy),
abyste ji mohli dát do prezentace pro kolegy a manažery.
Díky tomu by i ne-technické vedení hned vidělo, *co* a *proč* se mění.

> Ano prosím


<img width="1024" height="1536" alt="image" src="https://github.com/user-attachments/assets/36417e80-7c89-4beb-b42b-8894d21be4bc" />

> Jak pracuje reaktivní webová aplikace se session uživatele?

V reaktivní webové aplikaci (např. Spring WebFlux) se **uživatelská session** řeší trochu jinak než v klasickém servletovém světě (Spring MVC), protože WebFlux neběží na `HttpSession` a servletovém API, ale na **neblokujícím reaktivním serveru** (Netty, Undertow, apod.).

Tady je detailní rozbor:

---

## 1️⃣ Klasické MVC (servlet) vs. WebFlux

| Vlastnost   | Spring MVC (`HttpSession`)                               | Spring WebFlux (`WebSession`)                           |
| ----------- | -------------------------------------------------------- | ------------------------------------------------------- |
| API         | `javax.servlet.http.HttpSession`                         | `org.springframework.web.server.WebSession`             |
| Úložiště    | typicky v paměti serveru, případně v distribuované cache | defaultně v paměti, ale snadno rozšířitelné o Redis, DB |
| Životnost   | řízená kontejnerem (Tomcat/Jetty)                        | řízená WebFlux runtime                                  |
| Přístup     | synchronní metody (`getAttribute`, `setAttribute`)       | reaktivní API (`Mono<WebSession>`)                      |
| Serializace | blokující                                                | neblokující / asynchronní                               |

---

## 2️⃣ Jak to funguje v reaktivní aplikaci

* **Identifikace uživatele** probíhá stále přes **cookie** (např. `SESSION`), podobně jako v servlet světě.
* Po přijetí požadavku WebFlux zavolá `WebSessionManager`, který:

  1. Z cookie načte session ID.
  2. Dohledá session v úložišti (paměť, Redis, ...).
  3. Vrátí `Mono<WebSession>`.
* V průběhu požadavku můžeš session **asynchronně číst a měnit**:

  ```java
  @GetMapping("/profile")
  public Mono<String> profile(ServerWebExchange exchange) {
      return exchange.getSession()
          .map(webSession -> {
              String username = (String) webSession.getAttribute("username");
              return "Hello " + username;
          });
  }
  ```
* Na konci requestu WebFlux session **uloží zpět** (pokud byla změněna).

---

## 3️⃣ Kam session ukládat v produkci

Protože reaktivní aplikace má často **více instancí** (škálování, mikroservisy), je lepší:

* **Nepoužívat pouze in-memory** (každá instance má svůj store).
* Použít **Spring Session** s:

  * Redis (nejčastější)
  * R2DBC (reaktivní DB)
  * vlastní implementaci

Příklad s Redis:

```java
dependencies {
    implementation "org.springframework.session:spring-session-data-redis"
}
```

```java
@Configuration
@EnableRedisWebSession
public class SessionConfig {
}
```

---

## 4️⃣ Dopad na SPA aplikaci

Pokud máš **striktní oddělení front-end / back-end** (např. SvelteKit ↔ Spring WebFlux):

* Session můžeš stále používat (cookie-based).
* Ale často se místo toho používá **JWT** (JSON Web Token) v `Authorization` hlavičce.
* JWT nevyžaduje server-side session store → škálování je jednodušší.

---

💡 **Shrnutí:**
Reaktivní aplikace stále může mít session, ale používá **`WebSession`** místo `HttpSession`. Funguje to asynchronně a úložiště je volitelné, v produkci se typicky používá Redis. U SPA projektů se však často místo session preferuje **token-based autentizace** (JWT).
