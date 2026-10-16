# Pre-launch checklist: run before Google Ads goes live

Run on the published site (after the director merges and publishes). Tick each line yourself. Everything here can be checked in a browser, DevTools and `curl`.

## A. Files and indexing

- [ ] `curl -sI https://siteboosty.com/robots.txt` → `200`, body has `Allow: /` and the `Sitemap:` line.
- [ ] `curl -s https://siteboosty.com/sitemap.xml` → `200`, one `<loc>https://siteboosty.com/</loc>`, no `/psy/`.
- [ ] `curl -sI https://siteboosty.com/nonexistent-xyz` → status **404** and the body is the new light 404 page (links home + Telegram).
- [ ] `curl -s https://siteboosty.com/ | grep -c 'rel="canonical"'` → exactly **1**, and it is `https://siteboosty.com/`.
- [ ] Same canonical for `/?g=repair`, `/?g=speed`, `/?g=crm`, `/?g=seo` and `/?lang=ru` (the canonical must not contain `?g=`).
- [ ] Only one `<title>` and one `<meta name="description">` in the page; no `noindex` anywhere on `/`.
- [ ] Exactly one `<meta name="theme-color" content="#efede6">` (the page has one light theme; no dark-theme variant): `curl -s https://siteboosty.com/ | grep -c 'name="theme-color"'` → **1**.
- [ ] `/og.png` (1200×630), `/favicon.svg`, `/apple-touch-icon.png` all return `200` (not the 404 page).
- [ ] Open Graph preview: paste `https://siteboosty.com/` into Telegram "Saved messages" → card with image, title, text.
- [ ] JSON-LD parses: Google Rich Results Test or schema.org validator shows `ProfessionalService`, no errors.
- [ ] Redirects: `http://` and `www.` go to `https://siteboosty.com/` (checked on 2026-10-16: both 301).

## B. Language of what Google and users see

- [ ] Open `/?g=repair` in a browser whose language is **English** (or in a private window with `navigator.language=en`). The page must be in **Ukrainian** for an ad that is in Ukrainian. Today it is not (see the designer list in the report: auto-detect by browser language).
- [ ] Switch UA / RU / EN: title and description change, and the hero text for `?g=` changes with them.

## C. Ads landing: `?g=` headlines

Open each URL, hard reload, read the hero (h1 + text under it). They must not rotate and must match the ad group.

- [ ] `/?g=repair` → «Полагодимо сайт за 24 години», text ends with «Від 1 500 грн.»
- [ ] `/?g=speed` → «Прискоримо сайт за 1–3 дні», «Від 2 500 грн.»
- [ ] `/?g=crm` → «Заявки з сайту прямо в Telegram або CRM», «Від 3 000 грн…»
- [ ] `/?g=seo` → «Повернемо трафік з Google» (no price in the text)
- [ ] `/` (no param) → the fixed headline «Сайт зламався, гальмує або зник із Google?» with «Скажемо, в чому причина, за 30 хвилин — і скільки коштує це виправити.» under it. It does not rotate (the rotation was removed in the redesign). `/?g=none` or any unknown `g` shows the same.
- [ ] The prices in these headlines match the cards and packages lower on the page (repair 1 500, package prices 1 500 / 4 500 / 3 000). If they differ, tell the owner. Do not decide the number.
- [ ] Ad text and landing text agree (an ad must not promise something the hero does not).

## D. Conversion fires on Telegram and phone click

Open DevTools → Network, tick "Preserve log", filter `conversion` and `collect`.

- [ ] Click the header "Почати" button, the hero "Написати в Telegram", a package "Замовити", the sticky mobile button and "Хочу так само" in a case. Each must open `https://t.me/webdev342` (a real link, not `#`) and send a request to `googleadservices.com/pagead/conversion/18425867863/…` or `google.com/pagead/…` (label `koaNCPjw4uwcENfckdJE`).
- [ ] Click "Подзвонити" (hero and footer): opens `tel:+380686573262` and the same conversion request goes out.
- [ ] **The Telegram link leaves the page in the same tab.** Check the conversion request is really sent before the tab leaves (Preserve log on, status not "canceled"). If it is cancelled in some clicks, the designer must add `target="_blank" rel="noopener"` to the Telegram links or send the event with `transport_type: 'beacon'` / an `event_callback`. Do not go live with lost conversions.
- [ ] The group is passed: in Tag Assistant (or the request params) the conversion event shows `event_label` = `telegram/speed` for a `?g=speed` visit and `telegram/direct` for a direct visit.
- [ ] Google Tag Assistant (tagassistant.google.com) connected to the site shows the Ads tag `AW-18425867863` firing. In Google Ads → Goals → Conversions the "Lead-Telegram" action status turns to "Recording"/"Active" after the first test click. It can take a few hours, so test earlier than the launch day.
- [ ] If `GA4_ID` / `CLARITY_ID` are filled: Realtime in GA4 shows the visit and the `generate_lead` event; Clarity shows a POST to `clarity.ms/collect`. If they are empty, nothing loads (no request to googletagmanager except the Ads one).
- [ ] Test clicks from your own device pollute the numbers. Note the time and exclude them when you read the first report.

## E. Mobile speed (Ads Quality Score depends on it)

- [ ] PageSpeed Insights (pagespeed.web.dev) for `https://siteboosty.com/?g=speed`, **Mobile** tab. Write down the real Performance score, LCP, CLS, INP. Target: Performance ≥ 90, LCP < 2.5 s, CLS < 0.1.
- [ ] If the score is below 90, send the list of failed audits back to the designer. Do not put the page's "Lighthouse 90+" promise in ads until the page itself reaches it.
- [ ] On a real phone (or DevTools 375 px): no horizontal scroll, hero + both buttons visible without scrolling, the sticky button does not cover text or the buttons (it appears only after the hero buttons scroll away). There is no dark theme any more, so there is nothing to check there.
- [ ] Images below the fold are lazy, the hero loads without waiting for them (no layout jump).

## F. Honesty check of the page (rule: no invented numbers)

- [ ] Every price, term and number on the page is real. In particular, the "Було 34 / Стало 96, 7,8 с → 1,4 с" block in the hero is a mock-up with no source: it must be marked as an example or replaced with a real client result, or removed.
- [ ] Reviews and cases are only real ones (the 206 reviews and the archive link are real; the rest of the case numbers are from the owner).
- [ ] Opening hours / reply time in the hero hint («за 30 хвилин · 8:00–22:00 без вихідних») is what the owner can actually keep during ads.

## G. After launch (first 48 hours)

- [ ] Search Console: Domain property verified, `sitemap.xml` status "Success", URL inspection of `https://siteboosty.com/` → "URL is on Google" (or request indexing).
- [ ] Ads: clicks reach the site, conversions appear with the right group, no disapproved ads for "destination not working".
