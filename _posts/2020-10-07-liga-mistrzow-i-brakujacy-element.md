---
title: "Liga Mistrzów i brakujący element"
date: 2020-10-07-18T15:34:30+02:00
header:
  og_image: /assets/images/liga-mistrzow-i-brakujacy-element.jpg
categories:
  - champions-league
---

{% include champions-league-article-list.md %}

Nadszedł ten piękny dzień, że już prawie wszystkie ważne decyzje za nami. Mamy wybraną bazę danych, hosting i wiemy jak będziemy obsługiwać konta użytkowników.

No dobrze, ale czy czegoś brakuje? Jedną z częstych odpowiedzi, przy tego typu aplikacji, jest Content Delivery Network (CDN), który umożliwia nam przyspieszenie i odciążenie naszych serwerów. Dodatkowo warto zwrócić uwagę na obsługę HTTP2. Od kilku ładnych lat jest to już standard, a jednak bardzo często twórcy stron internetowych o nim zapominają.

Ale dlaczego się nad tym pochylamy? Chodzi o szybkość działania z perspektywy użytkownika. Kilkanaście lat temu (bodajże w 2007) Steve Sounders stworzył listę 14 zasad, którą powinna spełniać strona internetowa, żeby była szybka:

- Rule 1 - Make Fewer HTTP Requests
- Rule 2 - Use a Content Delivery Network
- Rule 3 - Add an Expires Header
- Rule 4 - Gzip Components
- Rule 5 - Put Stylesheets at the Top
- Rule 6 - Put Scripts at the Bottom
- Rule 7 - Avoid CSS Expressions
- Rule 8 - Make JavaScript and CSS External
- Rule 9 - Reduce DNS Lookups
- Rule 10 - Minify JavaScript
- Rule 11 - Avoid Redirects
- Rule 12 - Remove Duplicate Scripts
- Rule 13 - Configure ETags
- Rule 14 - Make AJAX Cacheable

Każdy kto optymalizował swoją stronę internetową wcześniej czy później na jakąś formę tej listy trafił, albo używał narzędzia, które na niej bazuje.

Jeżeli masz ochotę to możesz posłuchać 15 minut mojej prezentacji z 2017 roku jak ma się HTTP/2 do powyższej listy. Link prowadzi do odpowiedniego fragmentu: [https://youtu.be/S6y81TMr_lA?t=678](https://youtu.be/S6y81TMr_lA?t=678). Jeżeli ciekawi Cię historia protokołu HTTP to zapraszam od początku [https://youtu.be/S6y81TMr_lA](https://youtu.be/S6y81TMr_lA), bo całość trwa 29 minut.

Jak nie masz ochoty oglądać 😅 to podpowiem, że niektóre elementy lekko się przedawniły, ale większość została 😁

To jednak nie wszystko. Musimy jeszcze w dzisiejszych czasach zapanować nad certyfikatem SSL, zoptymalizować tak zwany "First Contentful Paint" (jeżeli pojęcie nie jest Ci znane to polecam [https://web.dev/first-contentful-paint/](https://web.dev/first-contentful-paint/)).

Dużo roboty prawda? Prawda! Szacując z grubego palca to dodatkowe 3 miesiące pracy 😜A jak bym powiedział, że istnieje narzędzie które może to zrobić za nas? I jeszcze dodałbym, że to co potrzebujemy da się ogarnąć za $0. Serio za 🆓, nie zgubiłem nigdzie po drodze 1 z przodu.

Narzędzie o którym mowa to Cloudflare. Tutaj pewnie nastąpi rozczarowanie, bo to nie jest usługa Azure. No ale jak to się mówi nie można mieć wszystkiego 😂

Cloudflare ogarnie nam::

- HTTP/2
- GZip
- DNS
- Certyfikat (domyślnie Let’s Encrypt, ale można wgrać własny)
- Cache na różne sposóby
- CDN bo proxy jest na całym świecie

A jak to się ma do powyższej listy Steva Sounders? Naprawdę nieźle:

- Rule 1 - Make Fewer HTTP Requests - ✅- Wprowadza i wymusza HTTP2, więc ilość request ma mniejsze - znaczenie
- Rule 2 - Use a Content Delivery Network - ✅- Tym właśnie jest Cloudflare
- Rule 3 - Add an Expires Header - ✅- Jak najbardziej tak, dzięki temu zapewniony jest cache
- Rule 4 - Gzip Components - ✅- Oczywiście, że tak :)
- Rule 5 - Put Stylesheets at the Top - ⚠️- To zależy już od nas jako twórców strony, acz HTTP2 + nowoczesne przeglądarki potrafią nas częściowo uratować 😜
- Rule 6 - Put Scripts at the Bottom - ⚠️- Jak wyżej
- Rule 7 - Avoid CSS Expressions - ❌- Niestety nie
- Rule 8 - Make JavaScript and CSS External - ❌- Też nie
- Rule 9 - Reduce DNS Lookups - ✅- Jak najbardziej tak, no chyba że się sami ręcznie postaramy i dodamy - 3 dodatkowe redirect 😜
- Rule 10 - Minify JavaScript - ✅- Cloudflare jak najbardziej obsługuje ten punkt
- Rule 11 - Avoid Redirects - ✅/⚠️- To zależy, bo Cloudflare zapamiętuje część za nas, ale można się postarać ...
- Rule 12 - Remove Duplicate Scripts - ✅/⚠️- Nie wszystko
- Rule 13 - Configure ETags - ✅- Tak bo zależy od tego poprawny cache.
- Rule 14 - Make AJAX Cacheable - ❌- Niestety nie, bo linki zależą od nas, acz jak mówiłem w prezentacji w dzisiejszych czasach nie ma to aż takiego znaczenia.

Czyli między 7 a 9 na 14 punktów załatwia nam zewnętrzna usługa. Ładny wynik, prawda? A do tego mamy jeszcze dodatkowe bonusy jak na przykład certyfikat SSL.

Rezultat jest taki, że strona poznajAzure.pl, która jak wiecie jest w 100% statyczna i stoi na Github Page ładuje się 16% szybciej. W płatnym planie dostajemy jeszcze więcej fajnych opcji, ale już te w darmowym robią mega wrażenie.

To by było na tyle na dziś. A co z pracą domową? Jeżeli masz ochotę to proponuję zastanowić się jak wystawić Azure Static Web page z własną domeną i certyfikatem SSL nie używając do tego Cloudflare. Podpowiedzi można szukać w tym wątku: [https://threadreaderapp.com/thread/1277618337753845760.html](https://threadreaderapp.com/thread/1277618337753845760.html)

Jeżeli niechcesz czekać to następne zapraszam do zapisania się poniżej😁

{% include mail-banner.html %}
