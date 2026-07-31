# AGENTS.md

## Проект

- Это форк расширения VS Code/Cursor [Favorites Panel](https://github.com/sabitovvt/vscode-favorites-panel) (пакет `favorites-panel-fork`, publisher `hy6`).
- Панель быстрого доступа к командам, файлам, папкам, URL, программам и сниппетам.
- Два места показа: отдельный Activity Bar (`favoritesPanel`) или Explorer (`favoritesPanelExplorer`) — переключается `favoritesPanel.explorerView`.
- Основная логика: `src/extension.ts` (загрузка конфига, активация), `src/commands.ts` (исполнение), `src/FavoritesPanelProvider.ts` (TreeView).

## Архитектура

| Файл | Роль |
| --- | --- |
| `src/extension.ts` | `activate`, сбор команд в дерево, регистрация TreeDataProvider и команд |
| `src/commands.ts` | `openFile`, `run`, `runCommand`, `insertNewCode`, `runSequence`, deprecated `openUrl` |
| `src/FavoritesPanelProvider.ts` | `TreeDataProvider`, `refresh()` перечитывает конфиг |
| `src/TreeItem.ts` | узлы дерева (группы через `commands[]`) |
| `src/types.ts` | `ICommand`, `ICommandWithSequence`, `TCommand` |
| `src/consts.ts` | `PLUGIN_NAME = 'favoritesPanel'` |
| `resources/demosettings.json` | демо-конфиг, если пользовательских команд нет |

- Команды расширения регистрируются как `favoritesPanel.<command>` (`openFile`, `run`, `runCommand`, `insertNewCode`, `runSequence`, …).
- Элемент дерева с `sequence` вызывает `favoritesPanel.runSequence`; с вложенным `commands` — группа (collapsible), без собственного action.
- Пути к файлам: workspace-relative через `path.join(workspaceFolders[0], …)`; абсолютные — второй аргумент `"external"` (как у `openFile`, так и у `revealInExplorer` внутри `runCommand`).

## Загрузка настроек

Порядок склейки (все источники concat в один список):

1. `favoritesPanel.commands` (User/Workspace settings)
2. `favoritesPanel.commandsForWorkspace` (только Workspace settings)
3. файл из `favoritesPanel.configPath`
4. файл из `favoritesPanel.configPathForWorkspace`
5. `<workspace>/.vscode/favoritesPanel.json`
6. `<workspace>/.favoritesPanel.json`
7. `<workspace>/favoritesPanel.json`

Если после склейки список пуст — показывается демо из `resources/demosettings.json`, и при активации открывается этот файл.

Формат внешнего JSON-файла:

- до 1.3.0: объект `{ "favoritesPanel.commands": [ ... ] }`
- с 1.4.0: также плоский массив `[ ... ]` (`getCommandsFromFile` принимает оба)

## Встроенные command-типы элемента

| `command` | Поведение |
| --- | --- |
| `openFile` | Открыть файл; args: `[path]` или `[path, "external"]` |
| `run` | `child_process.exec` shell-команды; args: `[program]` |
| `runCommand` | `vscode.commands.executeCommand`; спец-кейсы ниже |
| `insertNewCode` | Поиск regexp в файле и insert/replace; см. ниже |
| `runSequence` | Не пишется в JSON напрямую — задаётся полем `sequence: ICommand[]` |
| `openUrl` | **DEPRECATED** — показывать info и делегировать в `vscode.open` |

### `runCommand` — особые случаи

- `vscode.open` — аргумент как `Uri.parse`
- `vscode.openFolder` — аргумент как `Uri.file`
- `revealInExplorer` — строковый путь резолвится как у `openFile` (workspace-relative / `"external"`), затем `Uri.file`; без пути — голый `revealInExplorer`
- остальное — `executeCommand(command, ...rest)` as-is (объекты args для findInFiles, insertSnippet и т.п.)

### `insertNewCode`

Args: `[file, searchPattern, newCode, action?]`.

- `action`: `before` (default) | `replace` | `replaceAll` | `after` (не реализован — ошибка)
- `replaceAll` в коде — регистр важен (`replaceAll`); в старых README встречалось `replaceALL` — это неверный вариант
- поиск — `RegExp`; `before`/`replace` — первое совпадение; файл всегда относительно корня workspace

### `sequence`

Поддерживаются внутри sequence только: `openFile`, `run`, `runCommand`, `insertNewCode`. Остальное — information message.

## UI / настройки панели

- Toolbar: `refreshPanel`, `openFavoritesPanelSettings` (User `settings.json` → `favoritesPanel.commands`), `openWorkspaceJsonSettings` (workspace file → `favoritesPanel.commandsForWorkspace`).
- Не открывать Settings UI search для этих кнопок — иначе фриз редактора (фиксы 1.4.3–1.4.5).
- Иконки: codicon id (`item.icon`) + `ThemeColor(item.iconColor)`; дефолтные иконки по типу command в `getIcon`.
- Кастомные цвета: `workbench.colorCustomizations` с ключами вроде `favoritesPanel.myColorGreen`, затем `iconColor: "favoritesPanel.myColorGreen"`.

## Важный контекст (регрессии)

- `revealInExplorer` через `runCommand` ожидает `Uri`, не строку — резолв путей обязателен.
- `openFavoritesPanelSettings` / `openWorkspaceJsonSettings` должны использовать `revealSetting` с `edit: false`, иначе в JSON появляются пустые строки.
- Не возвращать deprecated `rootPath`; только `workspaceFolders`.
- Кроссплатформенные пути — через `path.join`, не конкатенация строк.
- При пустом конфиге не считать это ошибкой: демо + открытие `demosettings.json`.

## Изменения в коде

- Сохраняй разделение: конфиг/активация в `extension.ts`, исполнение в `commands.ts`, дерево в `FavoritesPanelProvider`.
- Новые типы действий панели — через регистрацию `favoritesPanel.*` и обработчик в `commands.ts`; для sequence добавляй case в `runSequence`.
- Не раздувай README пользовательскими API-деталями, которые нужны только агентам — держи их здесь.
- Тесты: `src/test/` (mocha + vscode-test). Не добавляй heavy integration/e2e без явной просьбы.

## Проверки

- `npm run compile` — TypeScript → `out/`
- `npm run lint` — eslint по `src`
- `npm test` — pretest (compile+lint) + `out/test/runTest.js`
- После правок логики команд/путей — как минимум `npm run compile`
- Если меняется упаковка или версия — собери `.vsix` и проверь установку

## Версия и упаковка

- Если пользователь пишет `подними и упакуй` / `bump` без версии — `+0.0.1`, затем `.vsix`.
- Явно указанная версия важнее дефолта.
- Версия: `npm version <version> --no-git-tag-version`
- Упаковка: `npx @vscode/vsce package` (артефакт `favorites-panel-fork-<version>.vsix`)
