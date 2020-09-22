---
title: "Liga Mistrzów i konta użytkowników!"
date: 2020-09-21-18T15:34:30+02:00
header:
  teaser: https://media-exp1.licdn.com/dms/image/C4E12AQHoBQi3r9OoMA/article-cover_image-shrink_720_1280/0?e=1606348800&v=beta&t=g7pajmt7olrYWVXHQjUKNfzr80z92mVgBPQ4FeTWnDI
categories:
  - champions-league
---

Wyobraź sobie następującą sytuację. Pracujesz sobie w pewnej firmie i na drzwiach masz tabliczkę (do wyboru): "senior architekt", "główny projektant" albo po prostu "Ten-Którego-Imienia-Nie-Wolno-Wymawiać" 😉. Przybiega do Ciebie szef wszystkich szefów, czyli jak wiadomo Krzysztof Jarzyna ze Szczecina i zaczyna monolog z następującym tekstem: "Nasi handlowcy odnieśli wczoraj ZAJE...ISTY sukces. Robimy apkę dla do losowania biletów na finały Ligi Mistrzów!". Ty patrzysz nerwowo w kalendarz, szybko wrzucasz w Google potwierdzenie terminu i już wiesz - mamy tylko miesiąc na dostarczenie aplikacji.

## Wymagania
Co udało się zebrać naszym "specjalistom" od sprzedaży:

- Docelowa ilość użytkowników to 5k-50k no chyba, że mąż Anny Lewandowskiej z "healthy plan by ann" zrobi retweet to może być więcej
- Ważnym wydarzeniem będzie losowanie na żywo i sprawdzanie wyników przez uczestników
- Mogą być różne sposoby zdobycia biletów: quizy, losowania, inne - to się jeszcze dogada 🤦‍♂️
- Aplikacja musi działać już za miesiąc, ma obsłużyć tegoroczne finały i może przyszłe
- Bardzo ważny jest podany z góry koszt utrzymania aplikacji.

W tym miejscu pozwolę sobie na krótką listę elementów do uwzględnienia:

- Hosting - pamiętaj o tym, że cena robi różnicę, musi się skalować i prawie na raz mogą wejść wszyscy użytkownicy (moment losowania). Do tego dochodzi HA!
- Bazę danych - musi obsługiwać skoki w użyciu (patrz punkt wyżej)
- Dane użytkowników, to BARDZO wrażliwa część. Część z nich może chcieć mieć MFA!
- SPA czy nie-SPA to też może mieć znaczenie :)

Nim przejdziesz dalej
Powyższe zadanie jest częścią inicjatywy Poznaj Azure, a poniżej masz pierwszą część rozwiązania dotyczącej bazy pod konta użytkowników. Aktualnie do naszych czytelników poszły już 4 maile, które dostaniesz pod zapisaniu się.

Kolejne części są publikowane raz na dwa tygodnie, tak byś Ty miał szansę zastanowić się nad rozwiązaniem, a także byśmy wspólnie mieli czas na dyskusje na Gitter.

## Konta użytkowników
### Bez chmury
Jakbym cofnął się wstecz kilka lat to powiedziałbym, że to najprostsze zadanie we wszechświecie. Bierzemy swoją ulubioną technologię (u mnie wtedy na 100% byłby to .NET Framework), dokładamy bazę danych i robimy zgodnie z tutorialem. Jak patrzę na to dziś, wybór nie jest taki zły, bo przynajmniej uniknąłbym tych dwóch problemów:

- Przechowywanie haseł w base64 (https://security.stackexchange.com/questions/194646/is-it-okay-to-save-passwords-as-base64-strings-with-no-other-hashing-or-encrypti#:~:text=Base64%20is%20not%20an%20encryption,storing%20it%20without%20any%20encoding)
- SQL Injection, ponieważ .NET Framework mnie przed tym zabezpiecza

Ale czy ochroniłbym wystarczająco dobrze swoich użytkowników? Szczególnie przed wyciekiem bazy danych. To już niestety zależy już od bardzo wielu czynników 😁

Sytuacja jest analogiczna w Java, Python, PHP czy NodeJS.

### Opcje do wyboru (nie tylko Azure), czyli niekompletna lista
Oczywiście na pewno kojarzysz robienie bazy użytkowników bardziej “ręcznie” i mówiąc szczerze ja też. Trochę wody jednak w rzekach upłynęło, ja stałem się mądrzejszy, więc dzisiaj podszedłbym do tego zadania trochę inaczej. Po pierwsze zobaczmy co jest dostępne na rynku do zarządzania użytkownikami (lista na 100% niekompletna):

- Active Directory, OpenLDAP, itd. - narzędzia idealnie sprawdzające się wewnątrz firmy “skalibrowane” pod zarządzanie użytkownikami i urządzeniami wpiętymi w sieć. Jestem przekonany, że byłeś, jesteś i będziesz członkiem jakiegoś drzewa podczas zatrudnienia w firmach. Do tego istnieją oczywiście rozszerzenia jak ADFS czy DEX, które pomagają nam zapanować nad bardziej skomplikowaną strukturą. I trzeba sobie powiedzieć szczerze, wewnątrz firmy trudno znaleźć lepsze narzędzia. A kiedy potrzeba coś szytego na miarę rozszerzamy istniejące usługi o dodatkowe komponenty zarządzania uprawnieniami, grupami, czy dynamicznymi dostępami. Podsumowując “młot” do bardzo konkretnego zastosowania i nie pasujący raczej do naszych potrzeb.
- Rozwiązania gotowe do zainstalowania i modyfikacji jak na przykład Keycloak czy Identity Server 4. Osobiście korzystałem z Identity Server i powiem szczerze, że jest to kompletne rozwiązanie, w którym ciężko coś “zepsuć”, ale własne doświadczenie mówi mi: “da się”. Wsparcie zarówno OpenID Connect jaki OAuth 2.0 zapewnia nam w zasadzie dowolność technologii, którą użyjemy w innych częściach aplikacji, a open source pozwala nam mieć nadzieję, na bardzo szybkie wykrywanie ewentualnych błędów.
- Rozwiązania SaaS jak na przykład Auth0, Userbase czy Azure B2C. Dzięki nim nie musimy przejmować się dodatkami jak na przykład integracja z Have I Been Pwned (https://haveibeenpwned.com/) czy FIDO2 (https://fidoalliance.org/fido2/) . Oczywiście czasem są one niezgodne z wizją “biznesową”, bo nie da się czegoś zrobić. Bardzo często to “nie da się” jest najlepszą możliwą opcją ochrony dla naszych użytkowników.

### Kryteria eliminacji
No dobrze to co powinniśmy wybrać? Jest kilka aspektów, które wziąłbym pod uwagę:

- Cena danego rozwiązania i jego ewentualne skalowanie (czytaj: jak cena zależy od ilości użytkowników)
- Łatwość implementacji i dostosowania do naszych potrzeb (czytaj: czy da się wgrać swojego CSS 🤪)
- Łatwość migracji, a w zasadzie ucieczki z danego rozwiązania
- Łatwość administracji danym rozwiązaniem.

### Kto odpadnie pierwszy?
W sumie to już to napisałem przy okazji prezentacji rozwiązania. W tym przypadku rezygnuje z Active Directory czy OpenLDAP. Bądźmy szczerzy to jest użycie armaty na muchę, albo próba wciśnięcia kwadratowego klocka do okrągłego otworu. Wiadomo da się, szczególnie jak ma się pilnik. Po pierwsze goni nas czas, a po drugie to nie do końca do tego służy.

### Moje czyli ...
Drugim punktem do odstrzału jest własne rozwiązanie. Tak wiem, że ciężko się rezygnuje z własnego pomysłu, ale będzie po prostu bezpieczniej. Serio? Myślę, że tak, poprawna implementacja resetu hasła, FIDO2, logowania za pomocą magic link, czy zwykłego MFA nie jest prosta, nie mówiąc już o logowaniu przez GitHub do łatwych nie należy. Oczywiście są gotowe rozwiązania, miliony NPMów, PyPIów, paczek Maven, NuGet czy PHP. Tylko zarządzanie tym nie jest proste. Oczywiście tutaj musi paść istotne stwierdzenie! Baza haseł i logowanie nie jest kluczowym elementem aplikacji do losowania biletów na Ligę Mistrzów. W przypadku banku powyższy mógłby okazać się zupełnie nietrafiony, bo obowiązują jakieś tam regulacje prawne.

### Ostatnie dwa i algorytm decyzyjny
Zostały dwa. Przyznam szczerze, że ze względu na historię projektową w moim własnym życiu pojawiał się w tym miejscu Identity Server 4. Kilku znajomych, do których napisałem w tym temacie, napisało Keycloak, bo bliżej im do świata Java. Rozwiązania dość podobne w swej idei, dające możliwość rozwoju, zastąpienia czy zmiany pewnych elementów (przykład dodatkowego problemu: Dlaczego w bankowej aplikacji wystarczy PIN 4-cyfrowy i jest to wystarczająco bezpieczne, a hasło musi już mieć co najmniej 8 znaków zawierających małe, duże litery i jeszcze cyfry lub znaki specjalne?). Podział dla mnie jest prosty i sprowadza się do odpowiedzi na poniższy schemat:

- Jeżeli musimy móc przenieść się z hostingu A, na hosting B -> to Identity Server, Keycloak lub coś podobnego
- Jeżeli wiemy, że będziemy robić jakiś custom -> to jak wyżej
- Jeżeli doszliśmy tutaj i cena w zależności od przewidywanej liczby użytkowników jest akceptowalna oraz mamy zaufanie do wybranej opcji to bierzemy SaaS.

### Zaufanie to podstawa
W przypadku opisywanej aplikacji SaaS jest wydaje się spoko. Szczególnie, że dochodzi nam obsługa botnet, ataków DDoS czy nagłego wzrostu użycia. Sprawdźmy jednak ostatni punkt. Jeżeli chodzi o zaufanie to z wymienionych przeze mnie opcji ufam dwóm: Auth0 oraz Azure B2C. Dlaczego? Bo w obu przypadkach są rzeczywiste rozwiązania na dużą skalę wraz z opisami. Dla Auth0 znajdziecie je pod adresem https://auth0.com/customers/. Dla Azure B2C trzeba użyć wyszukiwarki na stronie i mamy też pokaźną kolekcję: https://customers.microsoft.com/en-us/search?sq=%22Azure%20Active%20Directory%20B2C%22&ff=&p=2&so=story_publish_date%20desc. W mojej punktacji wygrywa tutaj Azure B2C z jednego powodu. Między innymi zaufał im Real Madryt (https://customers.microsoft.com/en-us/story/real-madrid), więc w piłce nożnej mają już doświadczenie 😂

### Funcjonalności i cena
Druga kwestia do porównania to oczywiście funkcjonalności. Czy da się osadzić własny CSS? Czy mają wsparcie dla logowania z GitHub? Czy da się użyć FIDO2? Czy…? Na to pytanie nie będę wstanie odpowiedzieć, ale na 99% oba spełnią wymagania aplikacji do losowania biletów. Dlaczego? Bo to ludziom zależy, żeby wygrać bilet, więc pewnie będą gotowi się chwilę pomęczyć. Tutaj zadanie dla Ciebie do zastanowienia: jakie funkcjonalności uważasz za niezbędne. Ten proces myślowy i spojrzenie na Azure B2C, Auth0 czy inne umożliwi Ci podjęcie decyzji.

No to na sam koniec pozostała cena. Zakładaliśmy od 5 tysięcy użytkowników do jednego miliona. Czyli mówiąc wprost, totalnie nie wiadomo ile i wyjdzie to w praniu. Prosty przelicznik (stan na dzisiaj) mówi mi, że Auth0 to $1138 na miesiąc za 50 tysięcy użytkowników, a powyżej mam się skontaktować z departamentem sprzedaży. W Azure B2C jest w tej chwili rozliczanie za aktywnego użytkownika w danym miesiącu, czyli Monthly Active Users (MAU) i pierwsze 50 tysięcy jest gratis. Oczywiście zero jest lepsze niż $1138, ale spójrzmy jak jest dalej. Przykładowe wyliczenie ze strony dla 100k jest następujące:

(50,000 MAUs x €0 (Free tier)) + (50,000 MAUs x €0.0047) = €231.908. Do tego płacimy jeszcze za MFA za pomocą SMS lub telefon: €0.026.

### Łyżka dziegciu
Które wybrać? Ciężko powiedzieć, acz na pierwszy rzut oka Azure B2C wydaje się tańsze😁, szczególnie że za nieaktywnych użytkowników w danym miesiącu nic nie płacimy. Kluczowe będą jednak jak zawsze drobiazgi. Dla aplikacji Liga Mistrzów, Azure B2C wydaje się dobrym wyborem. Jest tylko jedno ALE. Brak wsparcia dla własnych domen. Tak serio w Azure B2C nadal nie ma wsparcia dla własnych domeni pracują nad tym od 2017 roku (więcej: https://feedback.azure.com/forums/169401-azure-active-directory/suggestions/15334317-customer-owned-domains).

Oczywiście kiedy naprawdę robilibyśmy aplikację dla FIFA, która miałaby szansę stać się hitem, to jestem przekonany, że Microsoft poszedłby nam na ustępstwo i tak jak dla kilku dużych firm (np: Subway czy wspomniany Real Madryt) zacząłby wspierać naszą domenę. W innym wypadku gdy własna domena do logowania to “must have” kierujemy swój wzrok w na Auth0 i pewnie piszemy do “sales dept.” 😁

## Podsumowanie
Na sam koniec sprawa nie jest prosta i najlepszą odpowiedzią jest standardowa odpowiedź konsultanta, czyli “to zależy”. Czynników, które należy wziąć pod uwagę jest cała masa. Nikt nie mówił, że to proste. Z chęcią przeczytam Twoje zdanie poniżej, albo na Gitter. Zapraszam Cię też do zapisania się do naszej inicjatywy poniżej, gdzie będę miał szansę dosłać Ci pozostałe części naszej układanki.

{% include mail-banner.html %}
