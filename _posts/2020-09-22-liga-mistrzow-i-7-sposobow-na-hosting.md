---
title: "Liga Mistrzów i 7 sposobów na hosting!"
date: 2020-09-22-18T15:34:30+02:00
header:
  og_image: /assets/images/liga-mistrzow-i-7-sposobow-na-hosting.jpg
categories:
  - champions-league
---

{% include champions-league-article-list.md %}

Dzisiaj o hostingu naszej aplikacji. Ogarnęliśmy już bazę użytkowników (całość), feature są jeszcze in-progress, bo “to się jeszcze dogada 🤦‍♂️” (patrz specyfikacja https://poznajazure.pl/champions-league/ ), to warto wspomnieć, jak możemy “postawić” naszą aplikację, tak by chociaż makiety UI dało się serwować. Sposobów jest naprawdę dużo i pewnie każdy może być “idealny” dla Ciebie.



## Poznajmy naszych kandydatów
Zacznijmy od na 100% nie pełnej listy:
- Azure Storage + Azure CDN + ... - idealne na statyczne strony, acz zrobienie i użycie certyfikatu dla własnej domeny tak banalne nie jest. Szczególnie, gdy chcemy przyspieszyć całość, używając Azure CDN. Ostatnio mierzył się z tym Tomasz Onyszko i efekt jego pracy dostępny jest na Twitter i w Thread Reader App: https://threadreaderapp.com/thread/1277618337753845760.html. Przy okazji jak przejrzysz wątki poboczne, to zobaczysz inne propozycje, które znacząco ułatwiają taką konfigurację.
- Azure Functions - wiadomo serverless, najnowsza miłość wszystkich. No dobra już nie taka “nowa”, ale nadal “brak” serwerów korci. Czy się da? No pewnie! Czy to najlepsza opcja? Tutaj klasyczna odpowiedź konsultanta, czyli “to zależy”. Moim zadaniem nie jest najlepszy sposób na hosting całej aplikacji. Ale już np: w połączeniu ze statyczna stroną da się zrobić całkiem fajne combo. Zarządzanie nim nie będzie już jednak takie proste. A CI/CD będzie interesującym wyzwaniem 😜
- Azure SignalR + static web page + X - jak nie jesteś ze świata dotnet to małe wyjaśnienie. SignalR aktywne umożliwia “wypychanie” wiadomości do zarejestrowanych klientów na stronach w JavaScript. Czyli server-push. W wersji Azure nie musisz prawie o niczym myśleć, tylko zapłacić 😊 No dobra to jest małe oszustwo, bo coś musi się łączyć do tego Azure SignalR, żeby “dawać” mu dane do wypchnięcia. Może być to aplikacja w dotnet/NodeJS/Java/itd, czy też Azure Function. Czyli pomysł sam w sobie może i dobry, ale wymaga jeszcze większego kombinowania niż Azure Function, a problem sprowadza się do hostingu 😁
- Virtual Machine - Windows z IIS lub Linux z Nginx. Można? Można. Dla wielu osób najbardziej naturalny wybór. Powiem więcej. Bardzo dużo deweloperów od tego rozwiązania zaczyna, bo są do niego przyzwyczajeni. Nie po to mamy jednak chmurę, żeby używać czystych VM prawda? Szczególnie że taką maszynę trzeba zabezpieczyć samemu, a to już nie jest taka prosta sprawa. Szczególnie za pierwszym razem 🤪
- Azure Container Instances - nareszcie opcja z kontenerami. W teorii najprostszy sposób na wystawienie kontenera. który coś liczy, a potem jego “archiwizacja”. Czy będzie to pasowało dla strony chodzącej 24/7? Dobre pytanie prawda 🤔
- Azure Kubernetes Service (AKS) - a co jak już hostować to po bandzie i statyczne na razie pliki dajemy na K8s. Wygląda to trochę jak armata na muchę, ale kto co lubi. Jest jednak hype, buzzwords i inne takie 🤪
- Azure Web App w 3 odmianach, czyli wersja Windows, Linux oraz Docker (nawet z Docker Compose) - najbardziej klasyczne rozwiązanie do hostingu pojedynczej aplikacji. Aktualnie wspiera całkiem spory zestaw technologii wspieranych “natywnie” (m.in: dotnet, Java, Python, Ruby), a do tego mamy jeszcze Docker, więc możemy użyć wszystkiego 😊

## Czas na rosyjską ruletkę
Teraz czas na usunięcie części opcji, czyli proces decyzyjny. Kolejność losowa:

- Static web page to raczej nie będzie, więc Azure Storage odpada. Acz cennikowo wyszłoby super 😁
- Azure SignalR, jak zauważyłeś na pewno, sam w sobie nic nie daje. Oczywiście bardzo dobrze potrafi rozszerzyć istniejącą funkcjonalność o komunikacje od serwera do klienta, ale i tak potrzebujemy magicznej różdżki, która te dane wyprodukuje 🎩
- Virtual machine - zupełnie nie. Tak jak pisałem to naturalny wybór, ale długofalowo ciężki w utrzymaniu. Jeżeli nie masz bardzo konkretnego powodu do użycia VMki (np: GPU) to ciężko w chmurze ten wybór uzasadnić. Oczywiście zarówno Ty jak i ja znamy firmy, w których to będzie jedyna opcja nie wymagająca załatwiania papierów w milionie departamentów☹️
- Serverless, czyli Azure Function, korci prawda? Wszyscy o tym mówią, każdy chciałby spróbować, itd. Pamiętasz jednak wymagania? To ma być dla Ligii Mistrzów. Jak nie masz doświadczenia w serverless, to moim zdaniem nie jest to najlepszy, projekt, żeby zacząć, a dział PR może tego nie przeżyć 😏. Do tego dochodzi ustalenie kosztów dla zamawiającego. Z jednej strony super, bo w modelu hostowania serverless, jesteśmy wstanie podać dokładną kwotę per jedna sesja. Jednak ciężko to wyliczyć, nie mając dużego doświadczenia.
- Co do Azure Container Instances to jest takie powiedzenie: “jeśli coś jest głupie ale działa, to wcale nie jest głupie”. Pewnie je słyszałeś. Głupie jest to powiedzenie i tyle 🤪 A na poważnie: zerknij w cennik (pamiętaj o ilości środowisk), poszukaj jak zrobić własną domenę i sam podejmij decyzję, czy jest to sposób w który chcesz wdrażać aplikację. Dodatkowa rada: pamiętaj o wdrożeniach typu “zero-downtime”
- AKS to armata. Jak będą mikroserwisy na pewno warto wrócić do tematu, ale to chyba nie ten moment. Chyba, że naprawdę znamy k8s i do tego chcemy za pomocą np: KEDA załatwić sprawne autoskalowanie, czyli prawdopodobne w aplikacji nagłe skoki użycia (np: losowanie biletów, albo słynny tweet męża autorki “health plan by ann”). Do tego, żeby sprawnie operować Kubernetes, to trzeba go znać i to naprawdę nieźle. Czas nas bardzo goni, więc musisz podjąć decyzję czy "stać" Cię na inwestycję w tym momencie.
- Azure Web Apps - niby pasuje idealne, ale tanio nie jest. Za 3.5GB RAM w wersji standard, która dodaje nam deployment slot, umożliwiające wdrażanie aplikacji bez przerw dla klientów trzeba zapłacić miesięcznie prawie 550 zł. Dużo prawda?

No to co nam zostało? No właśnie, wszystkie opcje wykluczyłem, nic nie zostało i tu właśnie wkraczasz Ty, czyli "the Azure ninja", ubrany cały na biało! Pamiętasz swoją plakietkę na drzwiach? No właśnie. Czas działać i pisać! Zachęcam Cię do sprawdzenia chociaż jeden z powyższych opcji. Ja już napisałem kilka plusów i minusów, ale ciekawi jesteśmy Twojej opinii. 
Zapraszamy Cię do nas na listę mailingową (poniżej zapisy) i na Gitter, gdzie omawiamy dalsze części, czyli na https://poznajazure.pl/

{% include mail-banner.html %}
