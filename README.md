# Dokumentacja Procesu Wdrożeniowego: Aplikacja PulseGuard (v2.0)

## 1. Charakterystyka Produktu i Stack Techniczny
**PulseGuard** to system wysokiej dostępności (High Availability). Aplikacja musi działać niezawodnie w warunkach krytycznych, co determinuje wybór technologii:

* **Engine:** Flutter 3.x z natywnymi mostami (**Method Channels**) dla systemowych usług lokalizacji i Bluetooth.
* **Sensor Fusion:** Implementacja filtrów Kalmana w celu odfiltrowania szumów z akcelerometru, co pozwala uniknąć tzw. *false-positives*.
* **Komunikacja:** Protokół **MQTT** z mechanizmem *Quality of Service (QoS) 2*, gwarantującym dostarczenie pakietu z lokalizacją nawet przy niestabilnym łączu.

---

## 2. Zaawansowany Pipeline CI/CD (GitHub Actions)
Automatyzacja w PulseGuard kładzie szczególny nacisk na stabilność i eliminację błędów w logice powiadamiania ratunkowego.

CI – Continuous Integration (Ciągła Integracja)
To praktyka polegająca na tym, że programiści regularnie (często kilka razy dziennie) scalają swoje zmiany w kodzie źródłowym do głównego repozytorium

 CD – Continuous Delivery / Deployment (Ciągłe Dostarczanie / Wdrażanie)
To zestaw praktyk realizowanych po zakończeniu procesu CI. Automatyzuje wdrażanie zmian w kodzie do środowisk testowych lub produkcyjnych.


### Szczegółowy Workflow:
1.  **Static Analysis & Security Linting:**
    * Użycie `dart_code_metrics` do monitorowania złożoności cyklomatycznej (cel: < 10).
    * `osv-scanner` – skanowanie bibliotek pod kątem znanych podatności w bazie Google Open Source Vulnerabilities.
2.  **Smart Caching:**
    * Wykorzystanie `actions/cache` dla folderów `.pub-cache` oraz narzędzi budowania Gradle/CocoaPods (redukcja czasu builda o ok. 60%).
3.  **Automated Smoke Tests:**
    * Uruchomienie aplikacji na emulatorze i sprawdzenie, czy główny wątek (UI Thread) nie blokuje się przy starcie usług tła.

---

## 3. Ekosystem Fastlane – Automatyzacja „Ostatniej Mili”
Narzędzie **Fastlane** zarządza nie tylko sklepem, ale też dystrybucją certyfikatów dla funkcji **Critical Alerts**.

| Funkcja | Opis działania |
| :--- | :--- |
| **Match** | Synchronizacja certyfikatów i profili provisioningu pomiędzy deweloperami. |
| **Rugged Screenshots** | Automatyczne generowanie grafik marketingowych dla różnych rozdzielczości (widok nocny, wysoki kontrast). |
| **Beta Distribution** | Wysyłka buildu do grupy "Beta-Field-Testers" (ratownicy GOPR) po pomyślnym buildzie na `develop`. |

---

## 4. Strategiczne Wersjonowanie (SemVer)
Stosujemy ścisłą hierarchię, gdzie stabilność jest absolutnym priorytetem:

* **versionCode (np. 1042):** Unikalny identyfikator techniczny, powiązany z numerem buildu w CI.
* **versionName (np. 3.4.1):**
    * **Major (3):** Przełomowe zmiany (np. nowy protokół komunikacji satelitarnej).
    * **Minor (4):** Nowe funkcjonalności (np. wsparcie dla nowych urządzeń Garmin).
    * **Patch (1):** Optymalizacje wydajności i poprawki błędów (Hotfixy).

---

## 5. Trójstopniowa Strategia Testów
Zapewnienie działania aplikacji w sytuacji zagrożenia życia wymaga testów wykraczających poza standardowe UI.

### 5.1. Testy Jednostkowe (Unit Tests)
Weryfikacja algorytmów obliczających wektor przyspieszenia wypadkowego:
$$\vec{a}_{total} = \sqrt{a_x^2 + a_y^2 + a_z^2} - g$$

### 5.2. Testy Integracyjne (Integration)
Używamy frameworka **Patrol**, który pozwala na interakcję z systemowymi oknami dialogowymi (np. zgoda na lokalizację "Zawsze").

### 5.3. Hardware-in-the-loop (HIL)
Fizyczne urządzenia testowe są montowane na platformach wibracyjnych, które symulują wstrząsy podczas jazdy rowerem po ekstremalnym terenie.

---

## 6. Monitoring i Observability
* **Sentry Error Tracking:** Raportowanie błędów z uwzględnieniem poziomu naładowania baterii i siły sygnału GPS w momencie awarii.
* **Battery Lab:** Ciągłe monitorowanie zużycia energii (cel: < 4% na godzinę przy włączonym aktywnym śledzeniu).
* **Feature Flags:** Możliwość zdalnego wyłączenia nowej funkcji map bez konieczności nowej publikacji w sklepie.

---

## 7. Bezpieczeństwo i Niezawodność
* **Fail-Safe Mechanism:** Jeśli główny proces aplikacji ulegnie awarii, systemowy "Watchdog" automatycznie restartuje usługę monitorowania.
* **Data Sovereignty:** Dane o lokalizacji są szyfrowane algorytmem **AES-256** i przechowywane tylko przez 24h.
* **TLS Pinning:** Aplikacja akceptuje połączenia tylko z serwerami PulseGuard o konkretnym odcisku palca certyfikatu.

---

## 8. Proces Publikacji w Google Play

### Krok 1: Safety Declaration
Złożenie oświadczenia o wykorzystaniu wrażliwych uprawnień (lokalizacja w tle) – kluczowe dla funkcji wykrywania upadków.

### Krok 2: Asset Preparation
Dostarczenie filmów demonstracyjnych pokazujących reakcję aplikacji na symulowany wypadek dla recenzentów Google.

### Krok 3: Review & Rollout
* **Staged Rollout:** Udostępnienie wersji początkowo dla 10% użytkowników.
* **Monitoring Vitals:** Jeśli współczynnik ANR przekroczy **0.47%**, proces wdrażania zostaje automatycznie wstrzymany.

---

### Autorzy:
**Hubert Miłuch**,  
**Jan Rampalski** 
