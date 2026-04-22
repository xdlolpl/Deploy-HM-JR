# Dokumentacja Procesu Wdrożeniowego: Aplikacja EcoTrack 

## 1. Charakterystyka Produktu
**EcoTrack** to innowacyjna aplikacja mobilna stworzona w technologii **Flutter** (cross-platform), której celem jest edukacja i realne zmniejszenie śladu węglowego użytkowników.

### Kluczowe funkcjonalności:
* **Smart Scanner (OCR):** Wykorzystanie aparatu do skanowania paragonów i identyfikacji produktów o wysokim śladzie węglowym.
* **Eco-Dashboard:** Wizualizacja miesięcznej emisji $CO_2$ w formie interaktywnych wykresów.
* **Gamifikacja:** System wyzwań (np. "Tydzień bez mięsa"), za które użytkownik zdobywa wirtualne punkty i odznaki.
* **Integracja API:** Pobieranie danych o emisji z globalnych baz danych środowiskowych.

---

## 2. Architektura Pipeline CI/CD
Współczesny proces wydawniczy EcoTrack opiera się na eliminacji czynnika ludzkiego poprzez automatyzację powtarzalnych procesów.

### Definicje procesowe
* **CI (Continuous Integration):** Automatyczne budowanie i testowanie kodu przy każdym `commit` do gałęzi głównej (`main`). Pozwala na natychmiastowe wykrycie konfliktów.
* **CD (Continuous Deployment):** Automatyczna wysyłka przetestowanego kodu do Google Play Console bez ingerencji manualnej.

### Workflow w GitHub Actions
Pipeline podzielony jest na sekwencyjne zadania (**Jobs**) uruchamiane na runnerach Ubuntu/macOS:

1.  **Checkout & Cache:** Pobranie kodu i przywrócenie cache zależności (przyspieszenie buildów o 30-50%).
2.  **Linting & Static Analysis:** Weryfikacja standardów kodu (np. analiza `dart analyze`).
3.  **Test Suite:** Uruchomienie testów Unit & Widget. Błąd przerywa pipeline.
4.  **Build Release:** Kompilacja do formatu **Android App Bundle (.AAB)** – optymalizacja rozmiaru plików pod konkretne urządzenia.
5.  **Artifact Upload:** Archiwizacja pliku binarnego na serwerze GitHub.

---

## 3. Automatyzacja z Fastlane 
Narzędzie **Fastlane** stanowi pomost między kodem źródłowym a sklepem Google Play, automatyzując żmudne procesy administracyjne.

| Funkcja | Opis działania |
| :--- | :--- |
| **Zarządzanie certyfikatami** | Bezpieczne pobieranie kluczy podpisywania z zaszyfrowanego repozytorium. |
| **Screenshot Automation** | Automatyczne zrzuty ekranu na różnych modelach telefonów (Pixel 7, Samsung S24) w wielu językach. |
| **Metadata Management** | Wysyłka opisów, słów kluczowych i "Release Notes" jednym poleceniem: `fastlane deploy`. |

> **Wartość biznesowa:** Automatyzacja redukuje czas publikacji z 40 minut do ok. 5 minut, eliminując błędy ludzkie przy manualnym przesyłaniu paczek.

---

## 4. Strategiczne Wersjonowanie (SemVer)
Stosujemy standard **Semantic Versioning** wspierany przez Google Play:

* **versionCode (np. 102):** Liczba całkowita, unikalna dla każdego wdrożenia. System automatycznie inkrementuje tę wartość na podstawie numeru buildu w GitHub Actions.
* **versionName (np. 1.2.0):**
    * **Major (1):** Przełomowe zmiany (np. redesign UI).
    * **Minor (2):** Nowe funkcjonalności (np. moduł społecznościowy).
    * **Patch (0):** Poprawki błędów (Hotfixy).

---

## 5. Strategia Testów i Jakości
Stabilność EcoTrack gwarantuje trójwarstwowa piramida testów:

1.  **Testy Jednostkowe (Unit Tests):** Weryfikacja logiki, np. czy algorytm sumujący emisję $CO_2$ poprawnie obsługuje dane brzegowe.
2.  **Testy Integracyjne:** Sprawdzanie przepływu danych między modułami (np. Firebase Auth ↔ Profil użytkownika).
3.  **Testy UI (User Interface):** Roboty symulujące zachowanie użytkownika (klikanie, scrollowanie, sprawdzanie powiadomień Snack-bar).

---

## 6. Monitoring i Cykl Życia po Wdrożeniu
Wdrożenie to początek pętli zwrotnej (**Continuous Improvement**):

* **Firebase Crashlytics:** Raportowanie błędów krytycznych wraz ze "stack trace" i modelem urządzenia.
* **Performance Monitoring:** Analiza czasu ładowania wykresów (cel: < 2s).
* **Remote Config:** Dynamiczna zmiana parametrów UI (np. kolory kampanii) bez konieczności nowej publikacji w sklepie.

---

## 7. Bezpieczeństwo i Podpisywanie
* **Keystore Management:** Cyfrowy klucz dewelopera przechowywany jest w **GitHub Secrets**. Nigdy nie trafia do kodu źródłowego.
* **Szyfrowanie:** Dane lokalne w bazie SQLite chronione są kluczem **AES**, a komunikacja sieciowa odbywa się przez **TLS 1.3**.
* **Zgodność z RODO:** Funkcja całkowitego usunięcia danych użytkownika z poziomu ustawień.

---

## 8. Proces Publikacji w Google Play (Step-by-Step)

### Krok 1: Setup Konta
* Rejestracja w Google Play Console (opłata 25 USD).
* Weryfikacja tożsamości dewelopera.

### Krok 2: Konfiguracja Sklepu
* **SEO Optymalizacja:** Przygotowanie opisów bogatych w słowa kluczowe.
* **Grafiki:** Ikonka 512x512 (z kanałem alfa) oraz min. 4 screenshoty dla różnych form factorów.

### Krok 3: Weryfikacja Google
Proces recenzji (2-4 dni) obejmuje:
* Skanowanie pod kątem malware.
* Weryfikację uprawnień (np. dostęp do aparatu dla OCR).
* Testy jakości interfejsu (UX/UI Quality).

### Krok 4: Post-Release
Monitorowanie wskaźników **Vitals**. W przypadku wzrostu współczynnika ANR (*Application Not Responding*), następuje natychmiastowa ścieżka poprawkowa (Patch).

### Autorzy:
Hubert Miłuch
Jan Rampalski
