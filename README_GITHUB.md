<!-- HEADER -->
<div align="center">

# 🛡️ DeepGuard — System Detekcji Materiałów AI

**Projekt Zespołowy · Politechnika Wrocławska · Wydział Informatyki i Telekomunikacji**
**Kierunek: Telekomunikacja · Nabór 2023 · Rok akademicki 2025/2026**

![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=flat-square&logo=python&logoColor=white)
![PyTorch](https://img.shields.io/badge/PyTorch-2.12-EE4C2C?style=flat-square&logo=pytorch&logoColor=white)
![EfficientNet](https://img.shields.io/badge/Model-EfficientNet--B0-6366f1?style=flat-square)
![AUC](https://img.shields.io/badge/AUC-0.87-22c55e?style=flat-square)
![Dataset](https://img.shields.io/badge/Dataset-24%20388%20filmów-f59e0b?style=flat-square)
![Team](https://img.shields.io/badge/Zespół-16%20osób-8b5cf6?style=flat-square)
![Status](https://img.shields.io/badge/Status-Wersja%20finalna-22c55e?style=flat-square)

---

**[⬇️ Pobierz aplikację (.exe) z Google Drive](https://drive.google.com/file/d/1WQHLxWzvoh3W2_ZzC6TEHfOcl-XGkgq0/view?usp=sharing)**  
**[📄 Pełna dokumentacja techniczna (PDF)](https://drive.google.com/file/d/1WQHLxWzvoh3W2_ZzC6TEHfOcl-XGkgq0/view?usp=sharing)**

</div>

---

## 📌 O projekcie

DeepGuard to system detekcji materiałów audiowizualnych typu deepfake, opracowany w ramach projektu zespołowego na Wydziale Informatyki i Telekomunikacji Politechniki Wrocławskiej. Projekt realizowany był przez 16-osobowy zespół w ciągu jednego semestru akademickiego 2025/2026 i obejmował pełny cykl inżynieryjny: od przeglądu literatury, przez zbieranie i przetwarzanie danych, projektowanie i trening modelu, aż po implementację aplikacji webowej i desktopowej oraz wdrożenie infrastruktury serwerowej.

System umożliwia użytkownikowi wgranie własnego pliku wideo, a następnie automatycznie analizuje go pod kątem obecności śladów generowania przez sztuczną inteligencję. W odpowiedzi zwracany jest wynik w postaci liczbowego współczynnika pewności oraz czytelnego werdyktu: **PRAWDOPODOBNIE PRAWDZIWE**, **PODEJRZANE** lub **WYGENEROWANE PRZEZ AI**.

> ⚠️ **WAŻNE OSTRZEŻENIE:** Wyniki generowane przez system mają charakter informacyjny i statystyczny. Model może się mylić, szczególnie w przypadku materiałów o niskiej rozdzielczości, krótkich sekwencji poniżej 2 sekund oraz nagrań z bardzo silną kompresją lub wysokim szumem ISO. Wynik systemu nie stanowi dowodu ani ostatecznego werdyktu. Zalecana jest zawsze dodatkowa weryfikacja przez człowieka, szczególnie przy materiałach o istotnym znaczeniu.

---

## ⬇️ Pobranie i uruchomienie aplikacji (.exe)

Gotowa aplikacja desktopowa dla systemu **Windows 10/11 (64-bit)** dostępna jest do pobrania jako plik ZIP z Google Drive. Nie wymaga instalacji Pythona ani żadnych bibliotek.

**Kroki:**

1. Pobierz plik ZIP z linku: **[Google Drive](https://drive.google.com/file/d/1WQHLxWzvoh3W2_ZzC6TEHfOcl-XGkgq0/view?usp=sharing)**
2. Rozpakuj cały folder w dowolne miejsce na dysku.
3. Wejdź do folderu i uruchom plik `AI-Video-Detector.exe`.
4. Przeciągnij plik wideo na okno aplikacji lub kliknij strefę, aby wybrać plik z dysku.
5. Poczekaj na zakończenie analizy — wynik pojawi się automatycznie.

> ⚠️ Należy uruchamiać **cały rozpakowany folder**, a nie sam plik `.exe`. Plik wykonywalny bez towarzyszących bibliotek nie uruchomi się poprawnie.

> ⚠️ Niektóre programy antywirusowe mogą oznaczyć plik jako podejrzany. Jest to typowy fałszywy alarm dla aplikacji spakowanych narzędziem PyInstaller. W razie potrzeby należy dodać wyjątek w ustawieniach antywirusa.

---

## 🌐 Aplikacja webowa

Oprócz wersji desktopowej projekt obejmuje również w pełni funkcjonalną aplikację webową, wdrożoną na serwerze zespołowym i dostępną przez przeglądarkę internetową. Interfejs webowy oferuje:

- przesyłanie pliku wideo metodą drag and drop lub przez wybór z dysku,
- pasek postępu z wizualizacją etapów analizy,
- ekran wyników z werdyktem, współczynnikiem pewności i panelem szczegółów technicznych,
- historię analiz i możliwość powrotu do poprzednich wyników,
- obsługę języków polskiego i angielskiego.

Frontend zbudowany jest w technologii **Next.js 16 / React 19 / TypeScript 5**, backend w **Flask (Python)** z bazą danych **SQLite** i automatyczną retencją danych (24 godziny). Serwer działa na **Fedora Linux** za pośrednictwem szyfrowanego tunelu **Cloudflare**, bez konieczności wystawiania publicznego adresu IP.

---

## 🧠 Jak działa model

### Koncepcja

Podstawowym założeniem projektu jest to, że współczesne generatory wideo, choć bardzo realistyczne wizualnie, pozostawiają nieusuwalne ślady w **domenie częstotliwościowej** obrazu, niewidoczne gołym okiem. Sieć GAN zostawia charakterystyczną siatkę w widmie FFT (tzw. efekt szachownicy), modele dyfuzyjne (Sora, Runway Gen-3) generują artefakty dekodera VAE co 8 pikseli, a modele autoregresyjne (CogVideoX) produkują stopniowy dryf błędów między klatkami. Żaden z tych śladów nie jest możliwy do usunięcia bez degradacji jakości materiału.

### Architektura dwustrumieniowa

Model łączy dwa równoległe strumienie przetwarzania:

**Strumień RGB (przestrzenny):** oparty na sieci **EfficientNet-B0** z wagami pretrenowanymi na ImageNet (transfer learning). Przetwarza każdą klatkę jako obraz 224x224 pikseli i wydobywa 1280-wymiarowy wektor cech semantycznych, teksturalnych i geometrycznych.

**Strumień FFT (częstotliwościowy):** dedykowana trójwarstwowa sieć splotowa CNN (1 kanał wejściowy, warstwy 1→16→32→64 filtrów). Każda klatka jest najpierw konwertowana do skali szarości, następnie poddawana dwuwymiarowej szybkiej transformacie Fouriera, centrowaniu widma i normalizacji logarytmicznej. Wynikowy spektrogram o wymiarze 224x224 jest wejściem dla sieci FFT, która wydobywa 64-wymiarowy wektor anomalii częstotliwościowych.

**Fuzja i klasyfikacja:** oba wektory są konkatenowane w jeden wektor 1344-wymiarowy, który trafia do klasyfikatora MLP (Linear 1344→256→1) z warstwą Dropout(0.3) zapobiegającą przeuczeniu.

**Agregacja temporalna:** model przetwarza 20 równomiernie próbkowanych klatek z całego materiału. Wynik końcowy to średnia arytmetyczna logitów ze wszystkich klatek, po czym stosowana jest funkcja sigmoidalna, która przekształca wartość do zakresu 0-1.

### Logika werdyktu

| Wynik modelu | Werdykt |
|---|---|
| ≥ 0.90 | 🔴 WYGENEROWANE PRZEZ AI |
| 0.40 – 0.90 | 🟠 PODEJRZANE |
| ≤ 0.40 | 🟢 PRAWDOPODOBNIE PRAWDZIWE |

Asymetryczne progi (zamiast klasycznego podziału w punkcie 0.5) zostały dobrane eksperymentalnie w celu minimalizacji fałszywych alarmów. Próg 0.90 dla klasy FAKE gwarantuje, że alarm generowany jest tylko przy bardzo wysokiej pewności obu strumieni. Strefa SUSPICIOUS służy do izolowania materiałów granicznych i kierowania ich do ręcznej weryfikacji.

---

## 📊 Wyniki ewaluacji

Model ewaluowano na wydzielonym zbiorze testowym 2000 materiałów wideo, nieużywanych podczas treningu ani walidacji.

| Metryka | Wynik |
|---|---|
| AUC (Area Under ROC Curve) | **0.87** |
| F1-Score dla materiałów GAN | 0.97 |
| F1-Score dla FaceSwap | 0.95 |
| F1-Score dla modeli dyfuzyjnych (Sora, Runway) | 0.91 |
| F1-Score dla modeli autoregresyjnych (CogVideoX) | 0.89 |
| Spadek skuteczności przy kompresji H.264 >2 Mbps | tylko 3% |

Wynik AUC = 0.87 oznacza, że z 87% prawdopodobieństwem model przypisze wyższy wynik ufności losowo wybranej próbce fałszywej niż prawdziwej.

> ⚠️ **Znane ograniczenia:** model może dawać błędne wyniki dla materiałów poniżej 2 sekund długości (zbyt mało klatek dla analizy temporalnej), filmów o rozdzielczości poniżej 480p (artefakty FFT stają się zbyt rozmyte), starszych nagrań archiwalnych z wysokim szumem ISO (może być błędnie rozpoznany jako sygnatura GAN) oraz materiałów z najnowszych generatorów nowej generacji nieobecnych w zbiorze treningowym.

---

## 📁 Zbiór danych treningowych

| Parametr | Wartość |
|---|---|
| Łączna liczba filmów | 24 388 |
| Łączny rozmiar | 67.2 GB |
| Filmy autentyczne | 12 194 |
| Filmy wygenerowane przez AI | 12 194 |
| Podział | 80% trening / 10% walidacja / 10% test |
| Liczba generatorów | 16 (GAN, dyfuzyjne, autoregresyjne) |

Zbiór obejmuje materiały z generatorów: Gen3, Sora, Runway, Kling, Luma, Pika, LTX2, Wan, FaceSwap, CogVideoX, Jimeng, Veo3 i innych, a także materiały autentyczne z publicznych benchmarków (FaceForensics++, DFD).

---

## 🛠️ Technologie

| Warstwa | Technologie |
|---|---|
| Model AI | PyTorch 2.12, TorchVision, EfficientNet-B0, NumPy, OpenCV |
| Aplikacja desktopowa | Python 3.11, CustomTkinter, tkinterdnd2, PyInstaller |
| Backend | Python, Flask, SQLite, SQLAlchemy, PM2 |
| Frontend | Next.js 16, React 19, TypeScript 5, Tailwind CSS 4 |
| Infrastruktura | Fedora Linux, Nginx, Cloudflare Tunnel, Tailscale, rclone, Google Drive |
| Trening | NVIDIA GeForce RTX 5070 Ti (16 GB VRAM), CUDA 12, AMP FP16 |

---

## 👥 Zespół projektowy

**Kierownictwo:**

| Rola | Imię i nazwisko |
|---|---|
| Kierownik projektu | Aleksander Serwik |
| Zastępca kierownika | Kacper Kasprzyszak |

**Podgrupy projektowe:**

| Podgrupa | Członkowie |
|---|---|
| Research | Szymon Mikołajewski, Illia Żukowski |
| Model i ML | Paweł Michułka, Mateusz Miśkiewicz, Bartosz Szydłowski |
| Dane | Illia Żukowski, Viktoriya Bogatkowa, Piotr Rakoczy |
| Infrastruktura sieciowa | Sebastian Ciosen, Szymon Mikołajewski |
| Backend | Nikodem Adamczyk, Kacper Milak |
| Frontend | Jakub Dołżycki |
| DevOps | Oleg Kawałko |
| Dokumentacja | Gabriela Chrapek, Julia Staniecka |

---

## 📂 Struktura repozytorium

```
DeepGuard/
├── ml/                          # Model i pipeline predykcji
│   ├── model.py                 # Architektura DeepfakeDetector
│   ├── detector.py              # Silnik inferencji
│   ├── best_model.pth           # Wytrenowane wagi modelu (17 MB)
│   └── train/                  # Skrypty treningowe
├── backend/                     # Serwer Flask REST API
│   ├── app.py
│   └── requirements.txt
├── frontend/                    # Aplikacja webowa Next.js
│   └── src/
├── desktop/                     # Aplikacja desktopowa (.exe)
│   ├── app.py                   # Interfejs graficzny (CustomTkinter)
│   ├── build_exe.bat            # Skrypt budujący .exe
│   └── requirements.txt
└── docs/
    └── Detekcja_Deepfake_dokumentacja.pdf
```

---

## 📋 Uruchomienie ze źródła (tryb deweloperski)

Wymagany Python 3.10/3.11/3.12 (64-bit).

```bash
# Klonowanie repozytorium
git clone <adres-repo>
cd DeepGuard/desktop

# Instalacja zależności
pip install -r requirements.txt

# Uruchomienie aplikacji desktopowej
python app.py
```

Budowanie pliku `.exe` (Windows):

```bash
# W folderze desktop/
python -m PyInstaller app.py --name "AI-Video-Detector" --onedir --noconsole \
  --add-data "best_model.pth;." \
  --collect-all customtkinter \
  --collect-all tkinterdnd2 \
  --collect-all torch \
  --collect-all torchvision \
  --noconfirm
```

Gotowa aplikacja pojawi się w folderze `dist/AI-Video-Detector/`.

---

## 📄 Dokumentacja

Pełna dokumentacja techniczna projektu (63 strony) dostępna jest do pobrania pod linkiem Google Drive zamieszczonym na początku tego pliku. Obejmuje:

- analizę problemu deepfake i przegląd literatury naukowej,
- opis architektury hybrydowego modelu EfficientNet-B0 z gałęzią FFT,
- specyfikację potoku przetwarzania danych,
- opis procesu trenowania i strategii optymalizacji,
- wyniki ewaluacji z krzywą ROC i metrykami F1,
- dokumentację backendu, frontendu i infrastruktury sieciowej,
- analizę ograniczeń i kierunki dalszego rozwoju.

---

## ⚖️ Zastrzeżenia i ograniczenia

> ⚠️ System nie jest narzędziem forensic-grade i nie może być stosowany jako jedyne źródło dowodowe. Model osiąga AUC = 0.87, co oznacza, że w 13% przypadków losowych porównań może podjąć błędną decyzję. Wyniki dla materiałów z nowych generatorów nieobecnych w zbiorze treningowym mogą być znacznie mniej trafne.

> ⚠️ Projekt ma charakter akademicki i badawczy. Nie jest przeznaczony do użytku w środowiskach wymagających certyfikowanej dokładności (np. postępowania prawne, weryfikacja dziennikarska o krytycznym znaczeniu).

> ⚠️ Żaden automatyczny system detekcji deepfake nie zastąpi eksperckiej analizy ludzkiej. Wynik działania modelu należy zawsze traktować jako wskazówkę do dalszej weryfikacji, nie jako ostateczny werdykt.

---

<div align="center">

Projekt zrealizowany przez studentów **Politechniki Wrocławskiej**
Wydział Informatyki i Telekomunikacji · Kierunek Telekomunikacja · 2025/2026

</div>
