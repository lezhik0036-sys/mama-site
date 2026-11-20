# MAMA — публикация сайта mama-app.ru (GitHub Pages) и DNS в Selectel

Цель: чтобы `https://mama-app.ru` **открывался** и на главной были видны **название
и ИНН ООО ТД «ГАРАНТ»** — два требования, из-за которых T2 отклонил имя отправителя.
Сайт статический, в **ОТДЕЛЬНОМ** репозитории `mama-site` — не зависит от backend
(`api.mama-app.ru`) и от сервера mama-api-01.

## Текущее состояние зоны mama-app.ru (по вашим данным)
- NS → a/b/c/d.ns.selectel.ru, SOA есть → зона делегирована и обслуживается Selectel. ✅
- `api.mama-app.ru A → 135.106.244.63` — backend, **НЕ ТРОГАТЬ**.
- apex `mama-app.ru` — нет A/AAAA; `www` — нет CNAME. → их и добавляем.

## 0. Реквизиты заполнены (подтверждённые данные ООО ТД «ГАРАНТ»)
В `index.html` и `privacy.html` указаны ИНН 5032231151, КПП 773301001, ОГРН 1105032008960,
юр. адрес, e-mail и телефон из карточки предприятия. Workflow `pages.yml` всё равно
проверяет и **не опубликует** сайт, если вернётся хоть одна метка `[[ЗАПОЛНИТЬ...]]`.
Перед финальной публикацией независимо сверьте ИНН/ОГРН в ЕГРЮЛ (egrul.nalog.ru).

## 1. Репозиторий и GitHub Pages
1. Создайте **public** репозиторий `lezhik0036-sys/mama-site`, загрузите в него файлы из `mama-site.zip`
   (включая `CNAME` и `.github/workflows/pages.yml`).
2. Settings → Pages → Build and deployment → Source: **GitHub Actions**.
3. Settings → Pages → Custom domain: `mama-app.ru` (значение `CNAME` в репозитории уже это задаёт).
4. После выпуска сертификата включите **Enforce HTTPS** (GitHub сам выпустит Let's Encrypt).

## 2. Точные DNS-записи в панели Selectel (зона mama-app.ru)
Добавьте НОВЫЕ записи, **не меняя** существующую `api` A. Поле «имя/host»: `@` — корень зоны.

| Имя (host) | Тип | Значение | TTL |
|---|---|---|---|
| `@` | A | `185.199.108.153` | 3600 |
| `@` | A | `185.199.109.153` | 3600 |
| `@` | A | `185.199.110.153` | 3600 |
| `@` | A | `185.199.111.153` | 3600 |
| `@` | AAAA | `2606:50c0:8000::153` | 3600 |
| `@` | AAAA | `2606:50c0:8001::153` | 3600 |
| `@` | AAAA | `2606:50c0:8002::153` | 3600 |
| `@` | AAAA | `2606:50c0:8003::153` | 3600 |
| `www` | CNAME | `lezhik0036-sys.github.io.` | 3600 |

Значения A/AAAA — официальные адреса GitHub Pages (сверьте на странице документации
GitHub Pages «Managing a custom domain» перед вводом; AAAA можно не добавлять, тогда работает по IPv4).
**Не создавайте** CNAME на корень `@` — для apex используются именно A/AAAA.
`api.mama-app.ru` остаётся как есть.

### (Опционально) Защита домена от перехвата
В GitHub: Settings → Pages → «Verify domain» покажет TXT-запись вида
`_github-pages-challenge-lezhik0036-sys TXT <код>` — добавьте её в Selectel (host
`_github-pages-challenge-lezhik0036-sys`, тип TXT, значение из GitHub). Код выдаёт GitHub,
я его не придумываю.

## 3. Проверка после настройки (выполните у себя — у Claude нет внешнего DNS/сети)
```bash
dig NS mama-app.ru +short                 # a/b/c/d.ns.selectel.ru
dig A mama-app.ru +short                  # четыре 185.199.10x.153
dig AAAA mama-app.ru +short               # четыре 2606:50c0:800x::153 (если добавляли)
dig CNAME www.mama-app.ru +short          # lezhik0036-sys.github.io.
dig A api.mama-app.ru +short              # 135.106.244.63 — без изменений
curl -I https://mama-app.ru/              # HTTP/2 200 после выпуска сертификата
echo | openssl s_client -servername mama-app.ru -connect mama-app.ru:443 2>/dev/null | openssl x509 -noout -issuer -dates
```
Готово, когда `curl -I https://mama-app.ru/` → `200`, сертификат валиден, а на главной
видны «ООО ТД «ГАРАНТ»» и ИНН. Онлайн-альтернатива: dnschecker.org, SSL Labs.

## 4. Повторная заявка в SMS.RU/T2
Имя отправителя `Mama-app.ru` совпадает с доменом. Приложите ссылку на открытый
`https://mama-app.ru` с видимыми названием и ИНН и документы ООО ТД «ГАРАНТ».
