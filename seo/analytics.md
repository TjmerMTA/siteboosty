# Analytics: GA4 + Microsoft Clarity + Search Console

Both tools are free. Part 1 is for the designer/developer (code), part 2 is for the owner (get the ids), part 3 is for the owner (Search Console). Owner steps are in Russian.

Dev notes are in English.

## 1. Code (designer pastes into index.html)

Same pattern as `GADS_ID`: an id lives in `window.SB`, and the tag loads **only if the id is filled** (empty or `XXX…` placeholder = nothing loads, no network request).

### 1a. Config line (replace the existing `window.SB` line)

```html
<script>window.SB = { TG: "webdev342", PHONE: "+380686573262", GADS_ID: "AW-18425867863/koaNCPjw4uwcENfckdJE", GA4_ID: "", CLARITY_ID: "" };</script>
```

Owner later fills `GA4_ID: "G-XXXXXXXXXX"` and `CLARITY_ID: "xxxxxxxxxx"`.

### 1b. Replace the existing gtag block (the `<script>` with `dataLayer` / `gtag` / `lead()`)

One `gtag.js` file is loaded for both Ads and GA4 (loading it twice would double-count). `lead()` keeps the Ads conversion call unchanged and adds a GA4 `generate_lead` event.

```html
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  (function(){
    function filled(v){ return !!v && !/X{3,}/.test(v); }
    var ads=(window.SB.GADS_ID||'').split('/')[0], ga4=window.SB.GA4_ID||'';
    var ids=[]; if(filled(ads)) ids.push(ads); if(filled(ga4)) ids.push(ga4);
    if(!ids.length) return;
    var s=document.createElement('script'); s.async=true; s.src='https://www.googletagmanager.com/gtag/js?id='+ids[0]; document.head.appendChild(s);
    gtag('js', new Date());
    ids.forEach(function(id){ gtag('config', id); });
  })();
  function lead(kind){ try {
    var g=new URLSearchParams(location.search).get('g')||'direct';
    gtag('event','conversion',{ send_to: window.SB.GADS_ID, event_label: kind+'/'+g });   // Google Ads (unchanged)
    gtag('event','generate_lead',{ lead_type: kind, g_variant: g });                       // GA4 (goes to every configured id except Ads-only send_to)
  } catch(e){} }
</script>
```

The `gtag('js'…)` / `gtag('config', id)` calls follow Google's official install snippet (developers.google.com/tag-platform/gtagjs/install).

### 1c. Microsoft Clarity (add right after the block above)

```html
<script>
  (function(){ var id=window.SB.CLARITY_ID; if(!id||/X{3,}/.test(id)) return;
    (function(c,l,a,r,i,t,y){c[a]=c[a]||function(){(c[a].q=c[a].q||[]).push(arguments)};
      t=l.createElement(r);t.async=1;t.src="https://www.clarity.ms/tag/"+i;
      y=l.getElementsByTagName(r)[0];y.parentNode.insertBefore(t,y);
    })(window,document,"clarity","script",id); })();
</script>
```

Note: this is the standard Clarity loader written from the widely-published form. I could not open Microsoft's own copy of it. After the owner has the id, compare it with Clarity → Settings → Setup → Get tracking code. The only thing that must match is the `https://www.clarity.ms/tag/<id>` URL.

### 1d. Check it works (after ids are filled and published)

- GA4: open the site, then GA4 → Reports → Realtime. You should see yourself. Click "Написати в Telegram": event `generate_lead` appears.
- Clarity: DevTools → Network → a POST to `https://www.clarity.ms/collect` (this is Microsoft's own check). Or Clarity dashboard → Recordings → live.
- In GA4 mark `generate_lead` as a key event (the old name was "conversion"). The menu wording changes from time to time: look for Admin → Events → "Mark as key event".

## 2. Для владельца: как получить id (по 5 строк)

**GA4 (Google Analytics)**
1. Зайдите на analytics.google.com под тем же Google-аккаунтом, что и для Ads.
2. Admin (шестерёнка внизу слева) → Create → Property. Название: SiteBoosty, часовой пояс Kyiv, валюта UAH.
3. Платформа Web → адрес https://siteboosty.com, название потока SiteBoosty.
4. Откроется поток. Справа вверху «Measurement ID», вида `G-XXXXXXXXXX`. (Потом: Admin → Data streams → ваш поток.)
5. Скопируйте его и отправьте сюда. Вставим в `GA4_ID`.

**Microsoft Clarity** (записи экрана и карты кликов)
1. Откройте clarity.microsoft.com и войдите через Google или Microsoft.
2. New project. Название SiteBoosty, адрес https://siteboosty.com, категория любая подходящая.
3. Settings → Setup → «Get tracking code» (ручная установка).
4. Id проекта — короткий код (около 10 символов) в конце кода, после `"script",`. Он же виден в адресе проекта.
5. Скопируйте только этот код и отправьте сюда. Вставим в `CLARITY_ID`.

(Названия кнопок у Google и Microsoft меняются, но порядок такой: создать → указать сайт → скопировать id.)

## 3. Для владельца: Search Console через DNS TXT

Внимание: по проверке `dig`/`whois` DNS домена **находится у GoDaddy** (серверы имён `ns65/ns66.domaincontrol.com`), а не у Hostinger. TXT-запись добавляйте там, где живут серверы имён, то есть в GoDaddy. Если запись добавить в Hostinger, она не заработает.

1. search.google.com/search-console → «Add property» → тип **Domain** → введите `siteboosty.com` → Continue.
2. Google покажет TXT-запись вида `google-site-verification=…`. Нажмите Copy.
3. GoDaddy → My Products → у домена siteboosty.com кнопка DNS → Add New Record.
   - Type: **TXT**
   - Name: **@**
   - Value: вставьте скопированную строку целиком
   - TTL: оставьте по умолчанию → Save.
   (Тут названия кнопок я не проверял вживую, они могут отличаться.)
4. Вернитесь в Search Console, нажмите **Verify**. Если пишет «не найдено», подождите 15–60 минут (иногда до суток) и нажмите снова.
5. Запись **не удаляйте**: без неё подтверждение слетает. Она не мешает сайту. (Сейчас у домена нет других TXT-записей.)
6. Sitemap: слева «Sitemaps» → в поле «Add a new sitemap» напишите `sitemap.xml` → Submit. Статус должен стать «Success». Сначала проверьте, что https://siteboosty.com/sitemap.xml открывается в браузере. Это работает только после публикации сайта директором.
7. Чтобы страница быстрее попала в Google: вверху в поле URL inspection вставьте `https://siteboosty.com/` → «Request indexing». Для Google Ads это не обязательно.

Чего не делать: не удалять и не менять A-записи и CNAME (там GitHub Pages: 185.199.108–111.153 и www → tjmermta.github.io), иначе сайт пропадёт.
