# decision-audit

Скилл для агентских сред, который меняет то, как агент принимает решения. Агент копает, пока факты не докажут лучший вариант, а не останавливается на первом правдоподобном ответе. Затем он оценивает кандидатов по двум шкалам (рейтинг и глубина доказательств, 0–10), отдаёт решение независимому ревьюеру и сам решает только то, где ошибка дёшева. Близкие оценки (разница меньше 2 баллов) и необратимые действия уходят пользователю. Решения логируются в проекте.

Изначально сделан для DeepSeek Harness (DSH): там модель несколько раз подряд переделывала собственные ошибки вместо того, чтобы сначала проверить причину.

## Состав

```
decision-audit/
  SKILL.md    сам скилл
LICENSE       MIT
```

## Установка в DSH

Положи папку `decision-audit/` (копией или ссылкой) в любой корень скиллов:

- `<project>/.dsh/skills/` — только для этого проекта (корень проекта — ближайшая папка с `.git`);
- `~/.dsh/skills/` — для всех проектов;
- свою папку, подключённую через `customSkillDirs`, — для всех проектов (см. ниже).

### Своя папка через `customSkillDirs`

В профилях, где подключён бандл `dsh-web-app` (в стандартной сборке это `web` и `desktop`), агенты не используют хостовую строку `skill-filesystem`. Бандл её отключает, а каждый пресет агента монтирует собственную копию, которая видит только корень проекта и пользовательские корни. Поэтому патч, который задаёт одни `customSkillDirs`, ничего не делает: строка остаётся `disabled: true`.

Строку нужно включить явно. Тогда её скиллы попадают в глобальный слой каталога, который видят агенты всех пресетов; при совпадении имён побеждает версия из пресета. `includeDefaultRoots: false` ограничивает строку этой папкой, чтобы корни проекта и пользователя не сканировались второй раз. Добавь в `~/.dsh/profiles/<profile>/cordis.patch.yml`:

```yaml
- id: skill-filesystem
  name: '@deepseek-ai/dsh-skill-filesystem'
  disabled: false
  config:
    includeDefaultRoots: false
    customSkillDirs:
      - C:/path/to/skills   # папка, внутри которой лежит decision-audit/
```

Конфиг хоста читается только при старте, поэтому после правки перезапусти DSH. Проверить, что строка включилась, можно командой `dsh --profile web --dump-config`. Профиль `desktop` так не собрать: им управляет приложение DeepSeek Harness. Проверь каталог в новой сессии в проекте без этой папки — `decision-audit` должен в нём быть.

Описание короче 500 символов, поэтому оно не обрезается даже при лимите каталога по умолчанию (`catalogDescriptionMaxLength: 500`).

Раньше скилл назывался `decide-under-uncertainty`. При обновлении удали старую папку из корня скиллов и замени имя в `AGENTS.md`, иначе в каталоге окажутся две версии.

## Что скилл ожидает от среды

- Инструмент делегирования `subagent`, который запускает дочернего агента с чистым контекстом. В DSH он работает в фоне: сразу возвращает id, а результат приходит сообщением ребёнка или уведомлением о завершении. Поэтому скилл не действует, пока вердикт не пришёл, и не ждёт его в цикле, а заканчивает ход. В другой среде замени `subagent` в шаге 4 на инструмент с чистым контекстом.
- `subagent_fork` для ревью не используется: форк наследует историю и модель автора и не видит текущий ход.
- Глубина делегирования не меньше 1. Внутри сабагента ревьюера запустить нельзя; как с этим работать, описано в разделе «Delegation».
- Если рядом установлен `diagnosing-bugs`, он ведёт цикл диагностики до подтверждённой причины, а этот скилл подключается до того, как фикс применён, чтобы его выбрать.

## Блок для AGENTS.md

Скилл срабатывает надёжнее, если его вызов закреплён в глобальных инструкциях. Блок, согласованный с текущей версией скилла:

```markdown
## Decisions

- **Run the `decision-audit` skill whenever a choice is not already settled by a fact, and
  before every delete, move, process kill, write-API call or shared-surface change, even when the
  user ordered it or it looks settled.** The skill's step 1 decides whether an action is settled and
  what a settled action still requires (target check, log line). It also covers choosing between
  approaches, libraries, configs or fixes, and editing a file you have not read. Diagnosis itself
  runs in `diagnosing-bugs` when it is available; this skill takes over before a fix is applied.
- **Two 0–10 scores, kept apart.** Option rating says how well a candidate answers the question;
  evidence depth says how proven it is. Keep researching the top-rated candidate until its depth
  reaches 9; if the skill's research exit applies first, hand the decision to the user. An
  independent reviewer sets its own rating and depth and discards candidates at depth 8 or below,
  but the user may still choose a discarded one.
- **A margin under 2 points goes to the user**, even for routine decisions that would otherwise be
  handled alone.
- **Log every decision the skill scored or handed over to `<project>/.dsh/decisions/`**, rejected
  candidates included. When the user is choosing, they see the discarded candidates and the full
  reasoning too.
- **When the user answers a decision the skill handed over, load the skill again before acting on
  the answer.**
```

## Логи

Решения пишутся в `<project>/.dsh/decisions/YYYY-MM-DD-<slug>.md`. Короткие записи о необратимых, аварийных и неудачных «решённых» действиях дописываются в `settled.md` в той же папке. Скилл добавляет `.dsh/decisions/` в локальный exclude-файл git (путь даёт `git rev-parse --git-path info/exclude`, это работает и в worktree): логи остаются локальными и не попадают в коммиты. Секреты в логах заменяются на `<REDACTED>`.

## Лицензия

MIT, см. [LICENSE](LICENSE).
