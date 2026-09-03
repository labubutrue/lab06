# Лабораторная работа № 6

Данная лабораторная работа посвящена изучению средств пакетирования на примере CPack.

Репозиторий: https://github.com/labubutrue/lab06

Финальный релиз: https://github.com/labubutrue/lab06/releases/tag/v1.0.1

---

## Homework

После настройки непрерывной интеграции необходимо организовать автоматическую сборку пакетов для изменений, помеченных тегами.

Пакет должен содержать приложение `solver` из предыдущего задания.

Каждый новый релиз должен состоять из следующих компонентов:

- архивы с файлами исходного кода: `.tar.gz`, `.zip`;
- пакеты с бинарным файлом `solver`: `.deb`, `.rpm`, `.msi`, `.dmg`.

Если commit помечен тегом, CI должен собрать соответствующие пакеты и разместить их в GitHub Release.

---

# 1. Подготовка приложения `solver` к пакетированию

Для лабораторной работы использован проект с приложением `solver`.

В `solver_application/CMakeLists.txt` добавлена установка исполняемого файла:

```cmake
cmake_minimum_required(VERSION 3.10)

project(solver)

add_executable(solver equation.cpp)

target_link_libraries(solver PRIVATE
    formatter_ex
    solver_lib
)

install(TARGETS solver
    RUNTIME DESTINATION bin
)
```

Корневой `CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.10)

project(lab06 VERSION 1.0.0)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(formatter_lib)
add_subdirectory(formatter_ex_lib)
add_subdirectory(solver_lib)
add_subdirectory(hello_world_application)
add_subdirectory(solver_application)

include(CPackConfig.cmake)
```

Для проверки проект был сконфигурирован и собран локально.

### Команда

```bash
cmake -S . -B build
```

### Вывод

```text
-- The C compiler identification is AppleClang 17.0.0.17000013
-- The CXX compiler identification is AppleClang 17.0.0.17000013
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done (1.3s)
-- Generating done (1.1s)
-- Build files have been written to: /Users/mac/tp-labs/lab06/build
```

### Команда

```bash
cmake --build build
```

### Вывод

```text
[ 10%] Building CXX object formatter_lib/CMakeFiles/formatter.dir/formatter.cpp.o
[ 20%] Linking CXX static library libformatter.a
[ 20%] Built target formatter
[ 30%] Building CXX object formatter_ex_lib/CMakeFiles/formatter_ex.dir/formatter_ex.cpp.o
[ 40%] Linking CXX static library libformatter_ex.a
[ 40%] Built target formatter_ex
[ 50%] Building CXX object solver_lib/CMakeFiles/solver_lib.dir/solver.cpp.o
[ 60%] Linking CXX static library libsolver_lib.a
[ 60%] Built target solver_lib
[ 70%] Building CXX object hello_world_application/CMakeFiles/hello_world.dir/hello_world.cpp.o
[ 80%] Linking CXX executable hello_world
[ 80%] Built target hello_world
[ 90%] Building CXX object solver_application/CMakeFiles/solver.dir/equation.cpp.o
[100%] Linking CXX executable solver
[100%] Built target solver
```

Проверим установку именно приложения `solver`.

### Команда

```bash
cmake --install build --prefix build/install
```

### Вывод

```text
-- Install configuration: ""
-- Installing: /Users/mac/tp-labs/lab06/build/install/bin/solver
```

### Команда

```bash
find build/install -maxdepth 3 -type f -print
```

### Вывод

```text
build/install/bin/solver
```

Таким образом, бинарные пакеты содержат исполняемый файл `solver`.

---

# 2. Настройка CPack

Созданы файлы `DESCRIPTION`, `ChangeLog.md` и `CPackConfig.cmake`.

Для поддержки WiX используется текстовая копия файла лицензии:

```bash
cp LICENSE LICENSE.txt
```

Финальный `CPackConfig.cmake`:

```cmake
include(InstallRequiredSystemLibraries)

set(CPACK_PACKAGE_NAME "solver")
set(CPACK_PACKAGE_VENDOR "labubutrue")
set(CPACK_PACKAGE_CONTACT "labubutrue")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Quadratic equation solver")
set(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_CURRENT_SOURCE_DIR}/DESCRIPTION")

set(CPACK_PACKAGE_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PROJECT_VERSION_PATCH})
set(CPACK_PACKAGE_VERSION ${PROJECT_VERSION})

set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE.txt")
set(CPACK_RESOURCE_FILE_README "${CMAKE_CURRENT_SOURCE_DIR}/README.md")

set(CPACK_PACKAGE_FILE_NAME
    "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}-${CMAKE_SYSTEM_NAME}-${CMAKE_SYSTEM_PROCESSOR}"
)

set(CPACK_DEBIAN_PACKAGE_MAINTAINER "labubutrue")
set(CPACK_DEBIAN_PACKAGE_SECTION "utils")

set(CPACK_RPM_PACKAGE_LICENSE "MIT")
set(CPACK_RPM_PACKAGE_GROUP "Applications/Engineering")
set(CPACK_RPM_CHANGELOG_FILE "${CMAKE_CURRENT_SOURCE_DIR}/ChangeLog.md")

set(CPACK_WIX_UPGRADE_GUID "91E9DC44-70F8-4F38-A281-4E0F3CD9373E")

set(CPACK_SOURCE_GENERATOR "TGZ;ZIP")
set(CPACK_SOURCE_PACKAGE_FILE_NAME
    "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}-source"
)

set(CPACK_SOURCE_IGNORE_FILES
    "/build/"
    "/_build/"
    "/_CPack_Packages/"
    "/artifacts/"
    "/\\.git/"
    "\\.DS_Store"
)

include(CPack)
```

---

# 3. Архивы исходного кода `.tar.gz` и `.zip`

Создадим архив исходного кода в формате `.tar.gz`.

### Команда

```bash
cpack --config CPackSourceConfig.cmake -G TGZ
```

### Вывод

```text
CPack: Create package using TGZ
CPack: Install projects
CPack: - Install directory: /Users/mac/tp-labs/lab06
CPack: Create package
CPack: - package: /Users/mac/tp-labs/lab06/build/solver-1.0.0-source.tar.gz generated.
```

Создадим архив исходного кода в формате `.zip`.

### Команда

```bash
cpack --config CPackSourceConfig.cmake -G ZIP
```

### Вывод

```text
CPack: Create package using ZIP
CPack: Install projects
CPack: - Install directory: /Users/mac/tp-labs/lab06
CPack: Create package
CPack: - package: /Users/mac/tp-labs/lab06/build/solver-1.0.0-source.zip generated.
```

---

# 4. Пакет `.dmg`

На macOS для создания `.dmg` используется генератор `DragNDrop`.

### Команда

```bash
cpack -G DragNDrop
```

### Вывод

```text
CPack: Create package using DragNDrop
CPack: Install projects
CPack: - Run preinstall target for: lab06
CPack: - Install project: lab06 []
CPack: Create package
CPack: - package: /Users/mac/tp-labs/lab06/build/solver-1.0.0-Darwin-arm64.dmg generated.
```

Проверка созданных локально пакетов:

### Команда

```bash
ls -lh *.tar.gz *.zip *.dmg
```

### Вывод

```text
-rw-r--r--@ 1 mac  staff    38K  3 сент. 05:25 solver-1.0.0-Darwin-arm64.dmg
-rw-r--r--  1 mac  staff   1,0M  3 сент. 05:24 solver-1.0.0-source.tar.gz
-rw-r--r--  1 mac  staff   1,0M  3 сент. 05:24 solver-1.0.0-source.zip
```

---

# 5. Автоматическая сборка `.deb`, `.rpm`, `.msi`, `.dmg`, `.tar.gz`, `.zip`

По аналогии с учебным материалом и примером отчёта CI настроен через GitHub Actions.

Workflow запускается только при отправке тегов `v*`.

Файл `.github/workflows/release.yml`:

<details>
<summary>Показать workflow</summary>

```yaml
name: Build packages and release

on:
  push:
    tags:
      - 'v*'

permissions:
  contents: write

jobs:
  linux:
    name: Linux packages
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Install dependencies
        run: |
          sudo apt-get update
          sudo apt-get install -y cmake g++ rpm

      - name: Configure
        run: cmake -S . -B build -DCMAKE_BUILD_TYPE=Release

      - name: Build
        run: cmake --build build

      - name: Build DEB
        working-directory: build
        run: cpack -G DEB

      - name: Build RPM
        working-directory: build
        run: cpack -G RPM

      - name: Build source TGZ
        working-directory: build
        run: cpack --config CPackSourceConfig.cmake -G TGZ

      - name: Build source ZIP
        working-directory: build
        run: cpack --config CPackSourceConfig.cmake -G ZIP

      - name: Show packages
        run: find build -maxdepth 1 -type f \( -name "*.deb" -o -name "*.rpm" -o -name "*.tar.gz" -o -name "*.zip" \) -print

      - name: Upload Linux artifacts
        uses: actions/upload-artifact@v4
        with:
          name: linux-packages
          path: |
            build/*.deb
            build/*.rpm
            build/*.tar.gz
            build/*.zip
          if-no-files-found: error

  macos:
    name: macOS DMG
    runs-on: macos-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Configure
        run: cmake -S . -B build -DCMAKE_BUILD_TYPE=Release

      - name: Build
        run: cmake --build build

      - name: Build DMG
        working-directory: build
        run: cpack -G DragNDrop

      - name: Show package
        run: find build -maxdepth 1 -type f -name "*.dmg" -print

      - name: Upload DMG
        uses: actions/upload-artifact@v4
        with:
          name: macos-package
          path: build/*.dmg
          if-no-files-found: error

  windows:
    name: Windows MSI
    runs-on: windows-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Install WiX
        shell: powershell
        run: choco install wixtoolset --no-progress -y

      - name: Configure
        run: cmake -S . -B build

      - name: Build
        run: cmake --build build --config Release

      - name: Build MSI
        working-directory: build
        run: cpack -C Release -G WIX

      - name: Show package
        shell: powershell
        run: Get-ChildItem build -Filter *.msi

      - name: Upload MSI
        uses: actions/upload-artifact@v4
        with:
          name: windows-package
          path: build/*.msi
          if-no-files-found: error

  release:
    name: Create GitHub Release
    needs:
      - linux
      - macos
      - windows
    runs-on: ubuntu-latest

    steps:
      - name: Download all artifacts
        uses: actions/download-artifact@v4
        with:
          path: artifacts
          merge-multiple: true

      - name: Show release files
        run: find artifacts -maxdepth 1 -type f -print

      - name: Create release
        uses: softprops/action-gh-release@v2
        with:
          files: |
            artifacts/*.deb
            artifacts/*.rpm
            artifacts/*.msi
            artifacts/*.dmg
            artifacts/*.tar.gz
            artifacts/*.zip
```

</details>

Таким образом:

- Ubuntu создаёт `.deb`, `.rpm`, `.tar.gz`, `.zip`;
- macOS создаёт `.dmg`;
- Windows создаёт `.msi`;
- после успешного выполнения всех трёх jobs результаты объединяются и публикуются в GitHub Release.

---

# 6. Создание tagged release

После настройки пакетирования изменения были закоммичены.

### Команда

```bash
git add .gitignore CMakeLists.txt solver_application/CMakeLists.txt CPackConfig.cmake DESCRIPTION ChangeLog.md .github/workflows/release.yml
git commit -m "Add cross-platform packaging"
```

### Вывод

```text
[main 26a19d9] Add cross-platform packaging
 7 files changed, 215 insertions(+), 1 deletion(-)
 create mode 100644 .github/workflows/release.yml
 create mode 100644 CPackConfig.cmake
 create mode 100644 ChangeLog.md
 create mode 100644 DESCRIPTION
```

После исправления конфигурации WiX был создан финальный tagged release `v1.0.1`.

### Команда

```bash
git tag v1.0.1
```

### Команда

```bash
git push origin v1.0.1
```

### Вывод

```text
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
To https://github.com/labubutrue/lab06.git
 * [new tag]         v1.0.1 -> v1.0.1
```

После отправки тега GitHub Actions автоматически создал release:

https://github.com/labubutrue/lab06/releases/tag/v1.0.1

В release находятся все требуемые заданием пакеты:

```text
artifacts/solver-1.0.0-source.tar.gz
artifacts/solver-1.0.0-Windows-AMD64.msi
artifacts/solver-1.0.0-Linux-x86_64.deb
artifacts/solver-1.0.0-Linux-x86_64.rpm
artifacts/solver-1.0.0-Darwin-arm64.dmg
artifacts/solver-1.0.0-source.zip
```

Таким образом, присутствуют все шесть требуемых типов:

- `.tar.gz`;
- `.zip`;
- `.deb`;
- `.rpm`;
- `.msi`;
- `.dmg`.

---

# 7. Исключение сборочных артефактов

Сборочные каталоги и созданные пакеты добавлены в `.gitignore`:

```gitignore
build/
_build/
*build*/
*install*/
CMakeFiles/
CMakeCache.txt
cmake_install.cmake
Makefile
.DS_Store
.idea/
_CPack_Packages/
*.deb
*.rpm
*.msi
*.dmg
*.zip
*.tar.gz
artifacts/
```

Проверим, что сборочные артефакты и пакеты не отслеживаются Git.

### Команда

```bash
git ls-files | grep -E '(^|/)(_?build|install|_CPack_Packages|artifacts)(/|$)|CMakeCache\.txt$|cmake_install\.cmake$|Makefile$|\.(deb|rpm|msi|dmg|zip|tar\.gz)$'
```

Команда не вывела строк.

Проверим финальное состояние репозитория.

### Команда

```bash
git status
```

### Вывод

```text
On branch main

Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

Проверим последние коммиты.

### Команда

```bash
git log --oneline -6
```

### Вывод

```text
d002a88 (HEAD -> main, tag: v1.0.1, origin/main) Fix WiX license file
7297ac3 (tag: v1.0.0) Update README for lab06
26a19d9 Add cross-platform packaging
2c8d48f Add solver_lib and solver application
570408e Add formatter_ex library and hello_world application
fc8ee2b Add formatter library CMake configuration
```

Проверим теги.

### Команда

```bash
git tag --list
```

### Вывод

```text
v1.0.0
v1.0.1
```

---

# Итог

В ходе лабораторной работы:

- настроено пакетирование приложения `solver` с помощью CPack;
- созданы архивы исходного кода `.tar.gz` и `.zip`;
- настроена сборка Linux-пакетов `.deb` и `.rpm`;
- настроена сборка Windows-пакета `.msi` через WiX;
- настроена сборка macOS-пакета `.dmg` через DragNDrop;
- CI запускается при создании тега `v*`;
- после успешной сборки пакеты автоматически публикуются в GitHub Release;
- финальный релиз `v1.0.1` содержит все требуемые заданием форматы;
- сборочные каталоги и сгенерированные пакеты не отслеживаются Git.
