# decision-audit

Скилл для агентских сред, который меняет то, как агент принимает решения. Агент копает, пока факты не докажут лучший вариант, а не останавливается на первом правдоподобном ответе. Затем он оценивает кандидатов по двум шкалам (рейтинг и глубина доказательств, 0–10) и проверяет лидера до того, как что-то менять. Обратимую локальную правку файлов вне общих поверхностей, если их прежнее состояние сохранено в коммите или копии, откат занимает одну команду, а результат агент видит сам, он проверяет сам: заранее пишет команду отката, отвечает с фактами на три вопроса ревьюера, применяет правку, смотрит результат и называет второго кандидата. Всё остальное идёт к независимому ревьюеру. Близкие оценки после ревью (разница меньше 2 баллов) и необратимые действия уходят пользователю. Решения логируются в проекте.

Изначально сделан для DeepSeek Harness (DSH): там модель несколько раз подряд переделывала собственные ошибки вместо того, чтобы сначала проверить причину.

## Состав

```
decision-audit/
  SKILL.md                  ядро: все правила, которые нельзя потерять
  references/review.md      лёгкая проверка, бриф рецензента и разбор его ответа (шаг 4)
  references/log.md         правила и шаблон лога решений (шаг 7)
  references/delegation.md  делегирование и работа сабагентом
LICENSE                     MIT
```

### Почему ядро короткое

DSH отдаёт загруженный скилл агенту как результат инструмента. Результаты длиннее 8192 символов он позже обрезает до первых 4096 и последних 1024 символов (плагин `tool-result-pruner`, настройки `thresholdChars`, `headChars`, `tailChars`). Раньше скилл занимал около 24 тыс. символов и терял середину через несколько минут после загрузки, вместе с правилами лестницы, передачи решения и лога. Порог считается в кодовых точках Unicode, а не в байтах. Обёртка DSH вокруг тела скилла (теги `<skill_content>`, `<skill_instructions>` и строки про базовую папку) занимает 303 символа плюс путь к папке скилла: 336 для `C:\DSH\.dsh\skills\decision-audit`. Сейчас ядро с обёрткой занимает 7 867 символов при таком пути и 7 875 при `C:\Users\contr\.dsh\skills\decision-audit` (тело без frontmatter — 7 531) и не обрезается. Подробности лежат в `references/`: каждый файл короче порога даже в том виде, в каком его возвращает инструмент чтения DSH (с тегами и номерами строк; самый длинный, `review.md`, — около 7,9 тыс.), а читает его агент на том шаге, где он нужен. Если ядро будет расти, держи его вместе с обёрткой меньше примерно 7 900 символов: путь к скиллу бывает длиннее, и нужен запас до 8192.

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

Описание занимает 487 символов, поэтому оно не обрезается даже при лимите каталога по умолчанию (`catalogDescriptionMaxLength: 500`). Кроме случаев, когда скилл нужен, оно называет и те, когда он не нужен: вопросы только на чтение, исследование без выбора и правки внутри подхода, который одобрил пользователь или выбрала проверка. При правках оставляй запас до 500.

Раньше скилл назывался `decide-under-uncertainty`. При обновлении удали старую папку из корня скиллов и замени имя в `AGENTS.md`, иначе в каталоге окажутся две версии.

## Что скилл ожидает от среды

- Инструмент делегирования `subagent`, который запускает дочернего агента с чистым контекстом. В DSH он работает в фоне: сразу возвращает id, а результат приходит сообщением ребёнка или уведомлением о завершении. Поэтому скилл готовит бриф и черновик лога до вызова, после вызова пишет одну строку без инструментов и заканчивает ход. Вердикт начинает следующий ход; опрашивать `list_agents` и `job_output` незачем. В другой среде замени `subagent` в шаге 4 на инструмент с чистым контекстом.
- `subagent_fork` для ревью не используется: форк наследует историю и модель автора и не видит текущий ход.
- Глубина делегирования не меньше 1. Внутри сабагента ревьюера запустить нельзя; как с этим работать, описано в разделе «Delegation».
- Если рядом установлен `diagnosing-bugs`, он ведёт цикл диагностики до подтверждённой причины, а этот скилл подключается до того, как фикс применён, чтобы его выбрать.

## Блок для AGENTS.md

Скилл срабатывает надёжнее, если его вызов закреплён в глобальных инструкциях. Блок, согласованный с текущей версией скилла:

```markdown
## Decisions

- **Run the `decision-audit` skill whenever a choice is not already settled by a fact, and
  before every delete, move, process kill, write-API call or shared-surface change (another person's
  data, a corporate system, a running service, or the harness itself: its config, installed plugins
  and UI), even when the user ordered it or it looks settled.** The skill's step 1 decides whether an action is settled and
  what a settled action still requires (target check, log line). It also covers choosing between
  approaches, libraries, configs or fixes, and editing a file you have not read. It is not needed
  for read-only questions, research with no choice, or edits inside an approach the user approved
  or the skill's check chose.
  Diagnosis itself runs in `diagnosing-bugs` when it is available; this skill takes over before a
  fix is applied.
- **Two 0–10 scores, kept apart.** Option rating says how well a candidate answers the question;
  evidence depth says how proven it is. Keep researching the top-rated candidate until its depth
  reaches 9; if the skill's research exit applies first, hand the decision to the user.
- **One check before acting on a choice the skill scored.** A reversible, local file edit outside
  any shared surface, with the prior state committed or copied aside, a one-command undo stated in
  advance and a result you can observe, gets the skill's light self-check. Anything else goes to an independent reviewer, which sets its own rating and depth
  and discards candidates at depth 8 or below, but the user may still choose a discarded one.
- **After a review, a margin under 2 points goes to the user**, even for routine decisions that
  would otherwise be handled alone. A light-checked change reports its runner-up and its undo.
- **Ask before every key press, click or reload in the user's live app**, naming them; never type
  into its chats or forms.
- **Log every decision the skill scored or handed over to `<project>/.dsh/decisions/`**, rejected
  candidates included; a light-checked change gets one line in `settled.md`. When the user is
  choosing, they see the discarded candidates and the full reasoning too.
- **When the user corrects, rejects or asks to rework a result, or answers a decision the skill
  handed over, load the skill again before changing anything.** A rework is a new decision: an
  earlier approval covers only what it named.
```

## Логи

Решения пишутся в `<project>/.dsh/decisions/YYYY-MM-DD-<slug>.md`. Короткие записи о правках с лёгкой проверкой (со вторым кандидатом и командой отката) и о необратимых, аварийных и неудачных «решённых» действиях дописываются в `settled.md` в той же папке. Скилл добавляет `.dsh/decisions/` в локальный exclude-файл git (путь даёт `git rev-parse --git-path info/exclude`, это работает и в worktree): логи остаются локальными и не попадают в коммиты. Секреты в логах заменяются на `<REDACTED>`.

## Лицензия

MIT, см. [LICENSE](LICENSE).
