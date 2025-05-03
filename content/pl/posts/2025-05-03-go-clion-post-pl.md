---
title: "Testy CLion z Nucleo G0"
date: 2025-05-03
draft: false
---

## Test CLion w Embedded

### Aktualne doświadczenie z różnymi środowiskami i edytorami

Ostatnio miałem okazję pracować w środowisku CLion, dlatego postanowiłem przetestować go w kontekście Embedded. Korzystałem z CLion przy mini-projekcie na studia w C++ i pisało się świetnie, więc stwierdziłem, że warto spróbować również w kontekście projektu Embedded. Moje doświadczenia z różnymi IDE do Embedded, takimi jak CubeIDE, CrossStudio, czy VSC z rozszerzeniami, nie były do tej pory zbyt imponujące (były okej, ale zawsze czegoś brakowało). Zwykle korzystam z procesorów STM32. Celem było sprawdzenie, czy CLion ułatwia debugowanie i pisanie kodu (1 IDE bez dodatkowych edytorów). Chciałem też uniknąć korzystania z OpenOCD (nie zawsze wydaję mi się, że jest to stabilne rozwiązanie - mam złe doświadczenia przy debugowaniu Zephyra na Windows).

W CubeIDE irytowała mnie prędkość działania, ale za to przy generowaniu projektu bezpośrednio z CubeIDE (korzystając z kreatorów tego środowiska) projekt działał od razu, tzn. po jego utworzeniu i kliknięciu 'Build', a potem 'Run'/'Debug' sesja debugowania lub wgrywanie firmware po prostu następowało bez jakiegoś grzebania w ustawieniach. Nie zmienia to faktu, że ciężko mi się pisze w tym CubeIDE (jestem przyzwyczajony do Visual Studio Code) i przez to pisałem kod w VSC właśnie, a debugowałem go w CubeIDE. Niedawno w CubeMX została dodana możliwość generowania projektów w typie CMake, co przydało mi się przy robieniu testów CLion.
 
Korzystając z Visual Studio Code było odwrotnie, tzn. z samym pisaniem i czytaniem kodu byłem zaznajomiony w tym edytorze, ale za to ustawienie debugera wymaga kilku kroków, które sprowadzają się do edycji kilku plików .json.
 
CrossStudio jest dla mnie podobne do Stm32CubeIDE, przy czym samo generowanie projektu uniemożliwia wykorzystanie bibliotek producenta bezpośrednio w generatorze projektu. Co może być pewnym problem, jeżeli (tak jak ja) zazwyczaj potrzebuję tych bibliotek bo chcę coś przetestować, a niekoniecznie pisać całej rozbudowanej aplikacji od zera. CrossStudio działa szybciej niż CubeIDE i ma również wygodną konfigurację z debugerem STM.

### Mini-test

Konfiguracja testowa była następująca:
- **Nucleo G071RB** ([link do płytki](https://www.st.com/en/evaluation-tools/nucleo-g071rb.html)),
- **CLion 2025.1** ([link do CLion](https://www.jetbrains.com/clion/)),
- **STM32CubeCLT** ([link do STM32CubeCLT](https://www.st.com/en/development-tools/stm32cubeclt.html)),
- **STM32CubeMX 6.14.1** ([link do STM32CubeMX](https://www.st.com/en/development-tools/stm32cubemx.html)),
- **arm-none-eabi-gcc 10.3** ([link do gcc](https://developer.arm.com/Tools%20and%20Software/GNU%20Toolchain)),
- Mój stary, niedokończony (😅) projekt zegara do gry w piłkę [[repozytorium](https://github.com/dunajski/football_timer)],
- **Windows 11**.

CubeMX oraz CLion miałem już zainstalowane. CLion sugerował, abym zainstalował CubeCLT, aby poprawnie wygenerować projekt.

> W chwili pisania tego tekstu JetBrains wprowadza nowe funkcjonalności dla Embedded Development, szczególnie dla STM32. W przyszłości warto będzie je przetestować, by sprawiedliwie porównać możliwości.

Zacząłem od przeczytania dokumentacji na temat uruchamiania projektów z pliku `.ioc`([link do rozdziału](https://www.jetbrains.com/help/clion/2025.1/embedded-development.html?Embedded_Development&keymap=VSCode#open-project)). Wydaje mi się jednak, że ta metoda niedługo stanie się przestarzała, biorąc pod uwagę, że CubeMX teraz umożliwia generowanie projektów z toolchainem CMake.![cube_mx_wybor_toolchaina](/images/cube_mx_cmake.png)

Metoda z dokumentacji JetBrains nie działała poprawnie, więc postanowiłem spróbować tej dostępnej bezpośrednio w CLion. Po wybraniu opcji **File -> New -> Project -> STM32CubeMX** pojawiło się okno, które pozwoliło mi na załadowanie projektu. Działało to bez problemu, ale najlepiej sprawdza się w przypadku nowych projektów wygenerowanych przez CubeMX, w których od razu wybrano toolchain CMake.

#### Czysty projekt, a stary projekt
Jeżeli masz możliwość, spróbuj testować CLion na świeżych projektach z wybranym toolchainem CMake. W moim przypadku musiałem wykonać kilka kroków w CubeMX: zapisałem mój stary projekt w nowym miejscu, zmieniłem toolchain na CMake i wygenerowałem projekt na nowo. Dodałem również ręcznie pliki do CMakeLists.txt.

```c
# Add sources to executable
target_sources(${CMAKE_PROJECT_NAME} PRIVATE
    # Add user sources here
        ${CMAKE_SOURCE_DIR}/Core/Src/buttons.c
        ${CMAKE_SOURCE_DIR}/Core/Src/menus.c
        ${CMAKE_SOURCE_DIR}/Core/Src/game.c
)
```

### Debugowanie

Po kilku drobnych problemach z CLion, CMake i CubeMX udało mi się w końcu zbudować projekt (domyślnie `ctrl + shift + B` w CLion). Następnie skonfigurowałem sesję debugową za pomocą GDB servera ST-Link, co okazało się niezwykle pomocne dzięki [dokumentacji JetBrains](https://www.jetbrains.com/help/clion/2025.1/embedded-gdb-server.html?Embedded_Development&keymap=VSCode&utm_source=product&utm_medium=link&utm_campaign=CL&utm_content=2025.1). 

Za pomocą okna **New Embedded GDB Server Run Configuration** stworzyłem konfigurację, która uruchamiała sesję debugową po naciśnięciu F5.![new_emb_gdb_server](/images/new_emb_gdb_server.png)

Moja konfiguracja miała nazwę `stm-gdb` i teraz po naciśnięciu F5 rozpoczyna się sesja debugowa. ![debug_session](/images/debug_session.png)

Musiałem jeszcze dołączyć plik .svd od producenta żeby móc mieć podgląd na wartości rejestrów MCU w zakładce **Peripherals** .

### Ocena

Testy CLion w Embedded z użyciem Nucleo G071RB, STM32CubeMX wstępnie oceniam na **4/5**. Po kilku początkowych problemach z konfiguracją, CLion udostępnia wygodne narzędzia do pisania i debugowania kodu. Podobne środowiska (takie jak CubeIDE) oferują łatwiejszą konfigurację debuggera, ale CLion sprawdza się dobrze, szczególnie z projektami opartymi na CMake. Nie mogę dać wyższej oceny ze względu na to, że kilkukrotnie miałem crash CLiona przy zmianie projektów oraz za to, że dla projektu wcześniej utworzonego (a nie nowego) nie poszło to tak gładko.

Nie zmienia to faktu, że testy były dość powierzchowne więc żeby mieć taką ugruntowaną opinię musiałbym więcej spędzić czasu przy używaniu CLiona. Co nie zmienia faktu, że zamierzam, go teraz używać do przyszłych projektów i mnie nie zniechęciło.
