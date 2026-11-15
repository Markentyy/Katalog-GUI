# Каталог Персонажів

![build](https://github.com/Markentyy/CliInterface-ProjectOOP/actions/workflows/build.yml/badge.svg)
![dotnet](https://img.shields.io/badge/.NET-8.0-512BD4?logo=dotnet&logoColor=white)
![csharp](https://img.shields.io/badge/C%23-12-239120?logo=csharp&logoColor=white)
![license](https://img.shields.io/badge/license-MIT-green)

## Зміст

- [Можливості](#можливості)
- [Технології](#технології)
- [Структура](#структура)
- [Графічний інтерфейс](#графічний-інтерфейс)
- [Швидкий старт](#швидкий-старт)
- [Команди](#команди)
- [Сумісність з версіями .NET](#сумісність-з-версіями-net)
- [Дизайн-документ](#дизайн-документ)
- [Ролі класів](#ролі-класів)
- [Персонажі з АПІ](#персонажі-з-апі)
- [Автор](#автор)
- [Ліцензія](#ліцензія)

## Можливості

* інтерпретатор з рядком `>`, лапки групують слова з пробілами,
  `help` показує команди, згруповані за стратегіями;
* каталог персонажів: створення діалогом, спорядження, здібності,
  посилання на графіку у кожного персонажа;
* автоматичне збереження після кожної успішної мутації та завантаження
  при старті (`--data <шлях>`, файл за замовчуванням `data/world.json`);
* команди зібрані у три стратегії: інформаційні, трансформативні
  (автозбереження), інфраструктурні (дані і довідка);
* короткий вигляд - вирівняна таблиця з HP-баром, детальний - картка
  персонажа з посиланням на графіку.

## Технології

| Технологія | Версія / примітка |
|---|---|
| C# | 12 |
| .NET (цільова платформа) | 8.0 (`net8.0`, `RollForward LatestMajor`) |
| Збірка і тести | xUnit, `TreatWarningsAsErrors` + XML-документація |
| Дані | `System.Text.Json` |

## Графічний інтерфейс

Проєкт включає Avalonia UI (`src/Gui/`) - кроссплатформенний графічний інтерфейс на C# для перегляду каталогу персонажів. Розміщений як десктопний додаток ( однаково працює на Linux і Windows). Використовує існуючі шари без змін у них:

* `CharacterCatalogPresenter` - завантаження даних і форматування (вигляд-незалежний підання);
* `GenshinCharacterCatalogService` + `GraphicsService` - завантаження персонажів з демо-АПІ та кешування графіки.

### Запуск

1. Запустіть демо-АПІ (у окремому вікні):
   ```bash
   dotnet run --project src/Api/Api.csproj --urls "http://0.0.0.0:5000"
   ```
2. Запустіть GUI:
   ```bash
   dotnet run --project src/Gui/Gui.csproj
   ```

Графік персонажів кешується локально (SHA-256 хеш URL), тому повторні відкриття вікна не роблять нових HTTP-запитів.

## Структура

```
src/
  Core/         домен: CharacterStats, DamageCalculator, ICharacterCatalogService
  Data/         збереження: IWorldStore + JSON-сховище (world.json)
  Cli/          фасад, REPL, парсер, рендери, стратегії і команди
  App/          точка входу (Program.cs)
  Infrastructure/  GenshinCharacterCatalogService (API + локальний fallback)
  Presentation/   CharacterCatalogPresenter + CharacterCatalogViewModel (GUI-agnostic)
  Api/          локальна демо-API над data/characters.json (http://localhost:5000)
  Gui/          Avalonia UI графічний інтерфейс (каталог персонaжів)
  Tests.*       юніт-тести по шарах
```

Залежності: `App -> {Cli -> {Core, Data}, Data -> Core}`.
Шар `Core` не знає ні про консоль, ні про збереження, ні про мережу.
Шар `Infrastructure` залежить від `Core`.
Шар `Presentation` залежить від `Core` + `Infrastructure`.
`Api` залежить тільки від `Core`.
`Gui` залежить від `Core` + `Infrastructure` + `Presentation`.

## Швидкий старт

Потрібен SDK 8.0 (або новіший - з увімкненим RollForward).

```bash
dotnet build src/App/App.csproj

# інтерактивний запуск (дані у data/world.json)
dotnet run --project src/App/App.csproj --no-build

# свій файл даних
dotnet run --project src/App/App.csproj --no-build -- --data /tmp/demo.json

# прогін демо-сценарію
dotnet run --project src/App/App.csproj --no-build -- --data /tmp/demo.json < demo/demo.txt

# тести
dotnet test
```

## Команди

| Команда | Приклад | Стратегія | Опис |
|---|---|---|---|
| `ls` | `ls` | Info | таблиця всіх персонажів (HP-бар, ATK/ARM, кількість предметів і здібностей) |
| `info <id\|name>` | `info Mage` | Info | картка: здоров'я, бонуси, посилання на графіку, інвентар, книга здібностей |
| `create <char\|item\|ability>` | `create char` | Transform | діалог створення з усіма полями; порожній id - авто `char-N` |
| `add --char_id <id\|name> --id <id\|name>` | `add --char_id Mage --id Sword` | Transform | одягнути предмет або вивчити здібність; після успіху - збереження |
| `save` | `save` | Infra | зберегти каталог у файл даних |
| `load` | `load` | Infra | перечитати файл даних (скидає поточний стан) |
| `help` | `help` | Infra | список команд, згрупованих за стратегіями |
| `exit` | `exit` | - | вийти з інтерпретатора |


## Сумісність з версіями .NET

Цільова платформа проєкту - `net8.0` з `<RollForward>LatestMajor</RollForward>`,
тому збирається і під SDK 9/10. Proєкт `Api` має цільову платформу `net10.0`.

## Локальна демо-API

Проєкт `src/Api/` - це .NET Minimal API, що подає 5 резервних персонажів з
`data/characters.json`. Потрібен, оскільки усі публічні Genshin API
(`api.genshin.dev`, `genshin.jmp.blue` тощо) не відповідають (502 / 521
Cloudflare - початкові сервери офлайн). Локальна API - надійне джерело даних.

```bash
dotnet run --project src/Api/Api.csproj --urls "http://0.0.0.0:5000"
```

Ендпоінти:

* `GET /characters` - усі персонажі
* `GET /characters/{name}` - один персонаж за іменем (регістронезалежно), 404 якщо не знайдено

## Персонажі з АПІ

При старті CLI автоматично намагається завантажити персонажі з локальної демо-АПІ
(`http://localhost:5000/characters`) через `GenshinCharacterCatalogService` і
додає їх до каталогу одночасно з локально завантаженими з `data/world.json`.
Якщо АПІ недоступна, використовується локальний fallback `data/characters.json` -
програма не падає, виводиться попередження.

Отримати персонажів з АПІ в запущеній сесії:

```bash
# 1. Запустити демо-АПІ на порту 5000
dotnet run --project src/Api/Api.csproj --urls "http://0.0.0.0:5000"

# 2. Запустити CLI (у іншому вікні терміналу)
dotnet run --project src/App/App.csproj --no-build

# 3. У CLI переглянути персонажів:
> ls
> info Diluc
```

Ендпоінти демо-АПІ:

* `GET /characters` - усі персонажі
* `GET /characters/{name}` - один персонаж за іменем (регістронезалежно), 404 якщо не знайдено

## Дизайн-документ

Детальний опис інтерфейсу і діаграми: [docs/design.md](docs/design.md).

## Ролі класів

| Шар | Клас | Роль |
|---|---|---|
| Core | `Character`, `Equipment`, `Ability` | доменні об'єкти каталогу |
| Core | `CombatResolver` | чиста математика шкоди і лікування |
| Core | `Registry<T>` | іменоване сховище з авто-ідентифікаторами |
| Core | `CharacterWorld` | агрегат: реєстри + книга здібностей |
| Data | `IWorldStore` | порт збереження |
| Data | `JsonWorldStore` | атомарний JSON-файл (запис через `.tmp`) |
| Cli | `Cli` | фасад: `useStrategy`, `execCommand`, `display` |
| Cli | `LineParser`, `ParsedCommand` | розбір рядка: слова в лапках, `--опції` |
| Cli | `ICommandStrategy` + 3 реалізації | категорії команд за ТЗ |
| Cli | `IRenderer<T>`, `IRenderable` | розділення виводу від логіки |
| Cli | `CharacterTableRenderer`, `CharacterCardRenderer` | два вигляди каталогу |
| Cli | `Repl` | цикл «рядок за рядком» поверх фасада |

## Автор

Марк Воскобойников, група КН-31 -
[github.com/Markentyy](https://github.com/Markentyy).

## Ліцензія

MIT - див. [LICENSE](LICENSE).
