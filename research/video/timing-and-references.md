# Тайминги и референсы в видеомоделях - что общее, что своё

Дата: 10 ноября 2026. Цель: общая часть универсального скилла промптов для видео. Сравнение поперёк моделей. Полные описания моделей - в отдельных отчётах (Omni: `../google-omni-1.2.md`; Seedance, Kling, Veo - готовят другие агенты).

Модели: Gemini Omni 1.2 Flash, Seedance 2.5 (и 2.0 для сравнения), Kling 3.0 / 3.0 Omni (4.0 упоминается вскользь), Veo 3.1, Sora 2, Wan 3.0 / 2.7 / 2.6, Runway Gen-4.5, Luma Ray3, MiniMax Hailuo.

Пометки надёжности:

- **[док]** - официальная документация или блог разработчика модели.
- **[док-посредник]** - гайд площадки, которая перепродаёт модель (Runway про Seedance, fal, Luma). Не разработчик, но часто с доступом к закрытым гайдам.
- **[тест]** - практик показал прогоны. В скобках - размер выборки, если он известен.
- **[мнение]** - утверждение без показанных тестов.
- **(по выдаче)** - страницу не открывал, цитата из пересказа поисковика. Надёжность ниже.

---

## Короткий итог

Общее для всех:

1. Порядок событий модели держат хорошо, точное время - плохо. Таймкод - это темп, а не метка кадра.
2. На короткий ролик - мало событий. Переполненный отрезок даёт пропуски, лишние склейки или дёрганое движение. Недогруженный - ожидание, повторы, замедление.
3. Речь: безопасно около 2 слов в секунду. На 4 словах в секунду речь обрезается.
4. Одно движение камеры на план или отрезок.
5. Опора задаёт внешность лучше текста. Опору надо назвать и дать ей одну узкую роль. Без роли с опоры тянется всё подряд, включая фон.
6. Первый кадр в тексте не пересказывать: писать, что меняется.
7. Постоянство между роликами: те же файлы опор, те же слова описания, последний кадр как начало следующего.

Где модели расходятся:

- **Синтаксис времени.** Seedance 2.5, Wan, Veo, Omni понимают таймкоды. Seedance 2.0 - только «Shot N». Kling - отдельные поля планов с длительностью.
- **Таймкоды и склейки.** У Seedance и Wan отрезки с таймкодами часто становятся планами. У Veo в одном тесте таймкоды дали один непрерывный план без склеек.
- **Ссылки на опоры.** Seedance - `@Image 1`. Wan 3.0 - `Image 1`. Kling - `@ИмяЭлемента`. Omni - только словами, теги сняты. Veo - словами, «using the provided images».
- **Пересказывать ли героя с опоры.** Противоречие не решено (раздел 2.4).
- **Лимиты опор** отличаются в 10 раз: от 2-3 (Sora, Veo, Runway) до 30-50 (Seedance 2.5).

---

## 1. Тайминги

### 1.1. Способы задать порядок и время

| Способ | Кто поддерживает | Надёжность |
|---|---|---|
| Слова порядка («сначала», «потом», «через две секунды») | Все. Wan прямо велит связки, иначе реплики «may merge» | [док] Wan; [тест] aireiter: все 6 моделей держали порядок |
| Таймкоды `[0-3s]` | Omni («no precise syntax needed»), Veo (`[00:00-00:02]` в гайде Google), Seedance 2.5 (целые секунды), Wan 3.0/2.7/2.6 | [док] |
| Таймкоды не работают | Seedance 2.0: «does not respond to timestamps and only responds to shot numbers», «support for precise timing… is unstable» | [док] BytePlus |
| «Shot 1 / Shot 2» | Seedance 2.0 и 2.5, Wan («Shot 1 [0–3 s]»), Kling, Sora | [док] |
| Отдельные поля планов с длительностью | Kling 3.0 Custom Multi-Shot: до 6 планов, длина каждого задаётся | [док] Kling |
| Привязка к событию в кадре («когда чашка падает, камера…») | Советы fal и Kapwing для Seedance; Omni - официальные демо | [мнение], [док-посредник] |
| Ключевые кадры | Первый и последний кадр - почти все. Kling 4.0 - до 10, Luma Ray 3.2 - до 64 с номером кадра | [док] Luma; Kling 4.0 (по выдаче) |
| Раскадровка-картинка | Omni (сказать порядок чтения клеток), Seedance (сетка) | [док] DeepMind; [тест] Starrd |

Общее правило из всех гайдов: таймкоды нужны, когда в одном плане несколько событий и важен порядок. Простое действие - без таймкодов. Так пишет fal: «I do not put timestamps in every prompt», таймкоды «give Seedance fewer chances to skip the middle or rush to the ending» [док-посредник, 10 роликов без чисел успеха].

Правила записи таймкодов, общие для гайдов Seedance 2.5 и Wan:

- Покрыть всю длину без дыр. Runway: «A gap between two timestamped blocks doesn't get filled in; it's left ungoverned» [док-посредник].
- Целые секунды: «1-second intervals as the basic unit» [док ByteDance, по Aliteq]. fal при этом пишет дробные (0–3.2 с) - проверки нет.
- Перед отрезками - одна фраза-сводка всей сцены. Так велят Wan («Overall description»), ByteDance для 2.5 и Runway: без неё «the shots have no anchor» [док].

**Сравнительных тестов «таймкоды против слов порядка» на одних и тех же промптах не найдено.** Есть только косвенное:

- aireiter (июль 2026, 17 роликов, по 1-3 прогона): «первая утка падает, примерно через две секунды - вторая». Слова, не таймкоды. Порядок верный у 5 из 6 моделей (Hailuo 02 пропустил событие). Интервал 2 с не попал ни у кого: Kling 2,9 с, Seedance 2.0 Fast 1,5 с, Wan 2.7 3,0 с, Veo 3.1 3,2 с [тест]. Veo и Kling вдобавок «поставили» упавших уток обратно.
- Wireflow (октябрь 2026, Veo 3.1 Fast, 1 прогон): три отрезка с таймкодами - порядок и время примерно соблюдены, но «scene detection found zero cuts». Таймкоды не дали склеек [тест].
- Для Omni см. отчёт по Omni: invideo - таймкоды соблюдает чётко (старые версии); DataCamp - звон опоздал на 0,15 с [тест].

### 1.2. Насколько точно соблюдается время

Одинаково у всех: **время - ориентир, не метка кадра.**

- Runway про Seedance 2.5: «treats these timestamps as pacing guidance rather than frame-exact cut points» [док-посредник].
- ByteDance про 2.0: точное время «unstable», жёсткие длительности отрезков «may lead to abnormal generation results», совет «Do not impose strict limits on the duration of each segment» [док].
- ByteDance про 2.5: переполненный отрезок «may contain excessive cuts or omit parts of the plot» [док, по Aliteq].
- Starrd: раскадровка «retains a degree of autonomy» [док ByteDance, по Aliteq].
- Тест aireiter выше: промахи 0,5-1,2 с при заказанных 2 с [тест].

Что помогает (без сравнительных тестов):

- Меньше событий на отрезок (все гайды).
- Действие, видимое в кадре, как точка привязки камеры: fal - «tie each camera move to an event the model can see» [мнение посредника].
- Цельное действие с началом, развитием и концом вместо цепочки микрособытий. Kapwing: иначе модель «has to guess the in-between motion», выходит «jittery or fragmented» [мнение].
- Готовый последний кадр - для точной конечной точки (Omni: Segmind; Kling: Futurepedia) [тест, мнение].
- Kling: отдельные поля планов с длительностью - самый жёсткий способ из найденных. Насколько точно соблюдается длина плана, замеров нет.

Где модели расходятся:

- **Таймкод = склейка?** Seedance 2.5 и Wan по своим гайдам режут на планы по отрезкам («Hard cut transition» в примере Wan). Veo в тесте Wireflow - один непрерывный план. Google в своём гайде называет таймкоды «multi-shot sequence», но тест этого не подтвердил. Omni по умолчанию сама режет на несколько планов, один план надо просить.
- **Veo и таймкоды вообще.** Гайд Google (блог Cloud, октябрь 2025) показывает `[00:00-00:02]`. ScreenWeaver (октябрь 2026) пишет, что документация API Veo таймкодов не описывает, и пишет прозой [мнение]. Противоречие: блог есть, справка молчит.

### 1.3. Сколько событий, планов и реплик на 5, 10, 15 секунд

Норм от разработчиков почти нет. Что есть:

| Источник | Что сказано | Надёжность |
|---|---|---|
| Google, Veo 3.1 | Пример: 4 плана по 2 с на 8 с | [док] |
| Google, Cloud (Omni и Veo) | «One focused moment» на короткий ролик; цепочка событий «often leads to muddled or incomplete videos» | [док] |
| ScreenWeaver про Veo | «Eight seconds is roughly one beat of a scene» | [мнение] |
| OpenAI, Sora 2 | Пример: 2 плана на 4 с. Короткие ролики слушаются лучше: два по 4 с лучше одного на 8 с | [док] |
| Kling 3.0 | До 6 планов на 3-15 с. «Each shot should add a distinct visual or narrative beat» | [док] |
| Wan | Пример: 3 плана на ~10 с | [док] |
| fal, Seedance 2.5 | 4 отрезка на 15 с, 6 на 30 с. Мало движения на долгий отрезок → «waiting, repetition, or slow motion» | [док-посредник] |
| Runway, Seedance 2.5 | 6-8 с на отрезок; финальный 3-4 с. Больше одного движения камеры - только если отрезок длиннее ~4 с | [док-посредник] |
| Kapwing, Seedance 2.5 | Отрезок короче 8-10 с «doesn't have time to land» (сами при этом пишут по 5-7 с) | [мнение] |
| Пересказ нескольких гайдов Seedance 2.5 | 3-4 смысловых отрезка на 15 с | [мнение] (по выдаче) |
| ByteDance, Seedance 2.0 | Одно движение камеры на план | [док] |
| Сторонний гайд Sora 2 | 2-4 плана в одном промпте | [мнение] (по выдаче) |

Сводка, которую можно брать за ориентир (вывод из таблицы, не норма):

- **5 с** - одно цельное действие или 1-2 плана.
- **8-10 с** - 2-4 плана, если монтаж с короткими планами (Veo, Sora); 1-2 смысловых отрезка, если план непрерывный (Seedance).
- **15 с** - 3-4 отрезка (Seedance, fal), до 6 планов (потолок Kling).

Расхождение по сути: Google и OpenAI показывают планы по 2 с, посредники Seedance советуют отрезки по 4-8 с. Похоже, это разница между «монтажной нарезкой» и «одним непрерывным планом с событиями», а не между моделями. Тестов, что Seedance плохо держит план 2 с, не найдено.

### 1.4. Речь: сколько слов в секунду

Разработчики норм в словах не дают. OpenAI: 4 с - «one or two short exchanges», 8 с - «a few more»; «Long, complex speeches are unlikely to sync well» [док]. Wan: «Lip sync to exact words does not work reliably» [док, страница Alibaba].

Замеры и советы:

| Модель | Данные | Надёжность |
|---|---|---|
| Veo 3.1 Fast | 8 слов за 4,3 с (1,9 сл/с) - удачно. 39 слов: ~4 сл/с, обрыв после 31-го слова, вдобавок не тот пол голоса. Совет: до ~15 слов на 8 с | [тест] Wireflow, 1 прогон |
| Veo 3 | 2-3 сл/с, 16-22 слова на 8 с | [мнение] veo3ai (по выдаче) |
| Veo 3 | Синхрон: ~80% с одним говорящим, <40% с несколькими, ~30% на китайском. Один говорящий на 8 с, реплика до 5 с | [тест] Eastondev (по выдаче, выборка неизвестна) |
| Kling 3.0 | 8-12 слов на 5 с; 4-6 слов на 3 с | [мнение] VideoAI.me (по выдаче) |
| Kling 3.0 Pro | Если ролик начинается с действия, на речь остаётся ~2,5 с из 5 - около 8 слов. Один говорящий: слова успел, но скомкано | [тест] Wireflow (по выдаче) |
| Kling 3.0 | Английская реплика 8-12 слов, одна фраза на ход, до 2 людей в кадре | [тест] Atlas Cloud (по выдаче) |
| Seedance 2.5 | В примере fal 14 слов на 0-3 с (~4,7 сл/с). Итог ролика не оценён | [док-посредник], расчёт мой |
| Omni (старые версии) | Синхрон одного говорящего держится 6-7 с | [тест] invideo |

Общий вывод: **около 2 слов в секунду - безопасно, 4 - уже обрыв или скороговорка.** Время на действие до реплики вычитать из бюджета речи.

Не найдено: замеров для русской речи. Русские слова длиннее английских, поэтому норма «2 слова в секунду» для русского, скорее всего, завышена. Это мой вывод, не тест. Из опыта пользователя с Omni: на несложном русском тексте губы попадают.

Общие правила записи реплик:

- Говорящего называть и держать реплику при нём (Kling, Wan, Sora, Seedance) [док].
- Несколько говорящих - по очереди, метки уникальные и одни и те же, без местоимений (Wan, Sora) [док].
- Кто молчит, тому - «держит рот закрытым» (fal) [мнение посредника].
- Одна реплика на один отрезок действия (Kapwing) [мнение].
- Один язык в реплике, не смешивать (ByteDance) [док].

Расхождение - формат реплики. Veo (блог Google): в кавычках. Omni (док Cloud): двоеточие без кавычек, чтобы не появились субтитры. Seedance 2.0: в фигурных скобках `{}`. Runway про Seedance 2.5: «Dialogue (имя): '…'». Sora: отдельным блоком под описанием сцены. Сравнительных тестов форматов нет ни по одной модели.

### 1.5. Темп действия, камера, монтаж во времени

Общее:

- Одно движение камеры на план или отрезок: ByteDance «only 1 type of camera movement in a single shot», OpenAI «one clear camera move and one clear subject action», Kapwing «one camera move per beat» [док].
- Движение камеры и движение героя спорят. Медиум-практик: когда камера игнорировалась, помогло убрать одно из двух («if the subject rotated and the camera pushed in, I picked one») [тест, 1 человек].
- Темп действия задаётся счётом шагов и паузами. OpenAI: «Actions work best when described in beats or counts», пример - четыре шага, пауза, штора «in the final second» [док].
- Финал ролика - слабое место. Omni теряет последнее движение камеры (Segmind), действие кончается рано (DataCamp). Seedance «rush to the ending» (fal). Veo вернул уток в исходное положение к концу (aireiter) [тест].
- Монтаж: Kling - поле Multi-Shot (выключен = один план). Wan 2.7 - фраза «Generate single shot.», без неё может резать. Omni - режет по умолчанию. Veo - по тесту не режет даже по таймкодам. Sora - несколько планов в промпте «also valid» [док; Veo - тест].

---

## 2. Референсы

### 2.1. Виды входов

| Вход | Где есть | Заметки |
|---|---|---|
| Первый кадр | Все | Sora: картинка того же размера, что ролик [док]. Kling: соотношение сторон первого кадра задаёт ролик [мнение] |
| Последний кадр | Omni, Veo, Kling, Seedance, Luma, Wan | Почти везде только вместе с первым. Veo: пара кадров - только 8 с; с опорами не совмещается [форум, не док] |
| Опорные картинки героя или товара | Все | Лимиты ниже |
| Опора стиля | Omni (словами), Seedance, Runway, Veo - блог про «style», но в API только тип `asset` | Расхождение: у Veo 3.1 отдельной опоры стиля в API не найдено (по выдаче) |
| Опора места | Seedance («scene image»), Wan, Runway, Omni (Flow), Veo («office setting» в примере Google) | |
| Опорное видео: движение, камера, ритм | Seedance (до 10 у 2.5), Kling (1 ролик до 15 с у 3.0), Wan (видео героя), Omni (до 3 по 3 с) | Omni: звук из видео игнорируется [док] |
| Видео героя | Sora («characters», 2-4 с), Kling (Element из видео), Wan («Video 1» - персонаж) | |
| Звук и голос | Seedance (`@Audio N`, тембр), Kling (голос к Element), Wan (тембр с опоры). Omni и Veo - нет | |
| Сохранённый персонаж | Kling Elements, Sora characters (до 2), Omni во Flow (`@имя`) | |

Лимиты числа опор:

- Seedance 2.5: до 30 картинок, 10 видео, 10 звуков, всего до 50 [док-посредник Runway, fal]. Seedance 2.0: 9 картинок, 3 видео, 3 звука (по выдаче).
- Kling 3.0: до 7 без видео, до 4 с видео (по выдаче); 4.0 - до 15 (по выдаче).
- Omni 1.2: 6 картинок (по 3 на 2 героя) в Gemini API, 10 на Cloud [док].
- Veo 3.1: до 3 картинок одного героя или товара, ролик только 8 с [док].
- Sora 2: до 2 персонажей, «no more than 2 characters per generation» [док].
- Runway Gen-4: до 3 [мнение, по выдаче].
- Wan 2.6: до 3 героев [док].

**Общее правило из разных гайдов: не забивать лимит.** ByteDance: 4-5 опор (1-2 лица, 1 место, 1 видео камеры, 1 звук), «not recommended to use the full asset limit», больше 4 людей - «stability… decreases» [док]. Omni (jxp, старая версия): новая опора посреди цепочки правок дестабилизирует сцену [тест].

### 2.2. Как назначать роль в тексте

Одинаково у всех: **каждой опоре одна узкая роль словами.** Сама загрузка роль не задаёт.

- ByteDance 2.5: «Clearly identify each image, video, or audio asset by its upload order and intended purpose» [док].
- fal: «Uploading several files does not tell the model how to divide responsibility between them»; «@Image1 controls only [...]» [док-посредник].
- Kapwing: «reference count and reference control are not the same thing»; «state what it should _not_ influence» [мнение].
- Runway про Seedance: «A loosely bound reference… tends to pull in qualities you didn't intend» [док-посредник].
- Veo (Google): «Using the provided images for the detective, the woman, and the office setting…» [док].
- Omni: «Describe how each image or video should be used in natural language» [док].
- Wan: «Image 1 in Image 2» или «the cat in Image 1 plays in the room in Image 2» [док].

Расходится синтаксис и счёт:

- Seedance: `@Image 1`, номер по порядку загрузки. Картинки, видео, звуки считаются отдельно [док].
- Wan 3.0/2.7: `Image 1`, `Video 1` с большой буквы и пробелом; одну опору можно без номера [док]. Wan 2.6: `character1`.
- Kling: `@ИмяЭлемента` [мнение, по выдаче].
- Omni: теги `<IMAGE_REF_0>` сняты, только словами [док].
- Luma Ray3 Modify: `image1` / `cref` [док, по выдаче].

Мелкие общие правила (по одному-двум источникам):

- Важная опора раньше в тексте: «The more an asset requires precise reference, the earlier it should be placed in the prompt» [док ByteDance].
- Путаница героев - грузить опоры в порядке первого появления героев [док ByteDance, по Aliteq].
- Подпись роли только внутри картинки (надпись на листе) не работает: «can easily cause character confusion or duplication» [док ByteDance, по Aliteq].
- Метка героя одна и та же на всём промпте, каждое упоминание - явно [док ByteDance, Wan].
- Правка или продление: ссылаться на видео прямо, не «reference Video 1», иначе модель примет за задачу по опоре [док ByteDance; Runway].

### 2.3. Что берётся надёжно, что плохо, что просачивается

Надёжно (по всем моделям):

- Лицо, одежда, свет, место, если опора чистая. Starrd: модели «hold wardrobe, lighting and scene very well» [тест, Seedance 2.0].
- Товар целиком, включая надписи на нём: Kling - худи с текстом «nailed» (Chase Jarvis, ранний тест); Seedance 2.5 - выдуманный ботинок совпал (RefVid) [тест, по 1 случаю].
- С видео - движение камеры, ритм, хореография. fal: «A video reference can carry camera movement, performance, or timing. An image is usually better for the look» [док-посредник]. Медиум: с опорным видео походка стабильна, без него - разная каждый раз [тест].

Плохо:

- Мелкие детали: татуировки, украшения дрейфуют даже с привязкой (Kling) [мнение практика].
- Много героев: у Kling лица плывут при росте числа героев, 2-3 держит хорошо [тест, 2 недели одного автора]; Seedance - больше 4 людей нестабильно [док]; Sora - не больше 2 [док]; Omni - 1-2 объекта надёжно, 4 - нет [тест, старая версия].
- Цвет одежды: Omni 1.2 - серое пальто стало почти чёрным (ZOOOP) [тест].
- Сгенерированный лист персонажа теряет сходство с живым лицом: «structural rather than fixable by prompting» (Starrd). Лечится вторым входом - исходным фото [тест].
- Опоры одного ракурса: первые два Element у Kling-практика вышли нестабильными из-за похожих углов [тест].
- Мелкий текст на опоре: «tends to survive into the video as garbled lettering» (Starrd) [тест].

Просачивается без спроса (общее явление, точных замеров нет ни у кого):

- **Фон опоры.** RefVid: «Anything in the photo can leak into the clip» [мнение автора-продавца]. Kapwing: «If an uploaded character reference also changes your background, define the reference's role more narrowly» [мнение]. fal пишет запреты заранее: «Do not copy @Image1's studio background» [док-посредник].
- **Облик с опорного видео.** Гайды Seedance велят писать «Do not transfer the room, clothing, face, camera shake… from @Video 1» и советуют силуэт или зелёный фон; Clipdance переводит видео в карту глубины, чтобы не перенести лицо и одежду. Это косвенное доказательство утечки, прямого теста нет [мнение].
- **Камера с опорного видео**, когда просили только движение героя [мнение, bestseedanceprompts].
- **Поза и выражение листа как подсказка движения.** Starrd: разная поза рук и улыбка в одном виде «read as a motion cue»; разный свет в клетках - «different scenes» [тест].
- **Сетка-раскадровка как монтаж.** Starrd: сетка 6 клеток одного момента дала «six cuts» вместо облёта; лист из 4 видов плюс «interpolate rather than cut» - исправило [тест, Seedance 2.0].
- **Субтитры и «близнецы»** (Seedance): полностью не убираются; в горизонтальном кадре субтитров меньше [док ByteDance].

### 2.4. Какие опоры годятся

Общее:

- Высокое разрешение, ровный свет, лицо видно, без фильтров (Runway, Luma, RefVid, ByteDance, Omni) [док, мнение].
- Чистый нейтральный фон для героя и товара (Omni во Flow, RefVid, Starrd) [док, мнение].
- Без лишнего текста на опоре: ByteDance велит убирать; Starrd - текст выйдет кашей [док, тест].
- Одна опора - один герой: ByteDance против «близнецов» - «single-person reference images»; Wan 2.6 - одно видео, один персонаж [док].
- Опоры места и стиля без лишних предметов (Omni, Flow) [док].
- Несколько ракурсов лучше одного: RefVid «Three to five photos beat one», профиль нужен, иначе нос и ухо модель угадывает; Runway - фас, профиль, три четверти, для роста - и лицо, и тело [мнение, док].
- Kling: Element из 2-4 картинок или видео; видео героя 3-8 с, один герой, лицо видно [док Kling, мнение].

**Главное противоречие - листы персонажа.**

- ByteDance (Seedance 2.0): «Using multi-view character images is not recommended»; лучше отдельная картинка - только голова крупно [док].
- Starrd (Seedance): лист из ~6 видов - «sweet spot», меньше 3 - модель додумывает, больше 8 - клетки мелкие [тест на своём конвейере].
- PromptFrenzy: «Seedance reads sheets natively» [мнение].
- Omni: до 3 картинок на героя отдельными файлами [док].

Как совместить (вывод, не тест): у Seedance отдельные файлы ракурсов надёжнее листа, лист - запасной ход. У Kling ракурсы идут в Element отдельными картинками. У Omni и Veo - отдельные картинки, лимит 3.

### 2.5. Текст и опора: что пересказывать, что нет

Одинаково у всех: **первый кадр не пересказывать, писать движение и перемены.**

- Omni и Veo (Cloud): «Redundant prompts confuse the model» [док].
- Wan: «The image defines the entity, scene, and style. Your prompt should focus on motion and camera movements» [док].
- Sora: «the image as an anchor for the first frame, while your text prompt defines what happens next» [док].
- Kling: картинка - первый кадр, текст описывает, что меняется [мнение, по выдаче].
- Продолжение ролика: не пересказывать прошлый клип. Kapwing: иначе модель повторит уже сделанное действие [мнение]; fal: «I do not retell the first clip» [док-посредник].

С первым и последним кадром: описывать путь между кадрами, который реально приводит к последнему. Текст, который спорит с кадрами, даёт дрейф (Kling, по выдаче); Google описывает переход и звук [док].

**Противоречие - пересказывать ли внешность героя с опорной картинки.**

| Не пересказывать | Пересказывать |
|---|---|
| Luma: «You don't have to re-describe the character's physical appearance» [док] | Runway про Seedance 2.5: описание словами вместо ссылки «risks the model reading that description as a new element», но 2-3 главные черты повторять «at each major beat» [док-посредник] |
| RefVid: «Do not repeat her hair colour or her jacket», при расхождении слов и фото «the model blends them» [мнение продавца] | fal: повторяет черты товара при опоре (кобальтовый корпус, медная кнопка) [док-посредник] |
| Danex, Selfielab про Runway: лишнее описание мешает [мнение] | Kling-практики, Wan-гайды: одинаковое описание в каждом плане [мнение] |
| Google про Veo: опора задаёт облик, текст - взаимодействие (пример не описывает внешность) [док по смыслу] | Omni (Cloud): описание внешности и голоса слово в слово в каждом промпте - для постоянства [док] |

Как это сходится (вывод, не тест): лицо не пересказывать. Пересказывать коротко 2-3 приметы, которые легко проверить и которые модель теряет (цвет одежды, логотип, предмет в руках), и то, чего на опоре нет. Слова не должны спорить с картинкой. Флоу (Omni): текст «should complement, not contradict» [док].

Стиль: ByteDance - либо назвать стиль словами, либо перевести опору в нужный стиль заранее [док]. Пространство: «express spatial relationships through reference images first, reducing complex text descriptions» [док ByteDance]. Голос с опоры - описать ещё и словами [док ByteDance].

### 2.6. Постоянство героя и товара между роликами

Общее (у всех моделей одно и то же):

1. **Тот же набор файлов опор.** RefVid: «Never change the set between clips»; новая стрижка или одежда для модели - новый герой [мнение]. Kapwing: «use the same character references and continuity language» [мнение].
2. **Те же слова описания** в каждом промпте. OpenAI: «reuse phrasing for continuity, and avoid mixing traits that may compete» [док]. Omni (Cloud): слово в слово [док]. Kling-практик: даже мелкая правка закреплённого блока заставляет модель перечитать лицо [мнение].
3. **Последний кадр как первый кадр следующего** (Kapwing, Runway, fal) [мнение, док-посредник]. У Seedance продление по стыку даёт скачок - ByteDance советует срезать 6 кадров с конца первого клипа и 1 с начала второго [док].
4. **Свет и палитра одинаковые.** OpenAI: «keeping lighting logic consistent is what makes the edit seamless», назвать 3-5 цветов [док]. Runway про Seedance: строка о постоянстве в конце промпта [док-посредник].
5. **Сохранённый персонаж**, где есть: Kling Elements, Sora characters, Flow `@имя`. Kling: слова сами по себе героя не держат - нужен Element [мнение, по выдаче]. RefVid: без фото - «Close, but not Mia» [тест].
6. **Seed** - где есть (Omni на Cloud, Veo) [док].
7. **Предмет, ушедший из кадра, назвать снова** при продлении (Omni) [док].

Где расходятся:

- Продление: Sora берёт весь исходник как контекст [док]; Omni - последние 10 с [док]; Seedance - качество падает с каждым продлением, «will compound» [док]; Veo - голос, которого нет в последней секунде, «doesn't extend well» [по ScreenWeaver].
- Реальные лица: Seedance 2.5 через BytePlus отказывает на фото реальных людей [док, по выдаче]; Omni блокирует «certain recognizable people» [док]. Для товара и нарисованных героев Seedance держит детали хорошо (RefVid) [тест].

---

## 3. Чего не нашёл

- Сравнительного теста «таймкоды против слов порядка» на одинаковых промптах.
- Замеров точности длины плана в Kling Custom Multi-Shot.
- Норм скорости речи от разработчиков. Любых замеров для русской речи.
- Сравнения форматов реплик (кавычки, двоеточие, скобки) ни по одной модели.
- Прямого замера утечки фона с опорной картинки и облика с опорного видео.
- Официального гайда ByteDance по Seedance 2.5 (лежит в Lark за входом; цитаты - через Aliteq и Vogoo).
- Официального гайда Runway Gen-4.5 (Runway Academy; справка Runway вернула 403).
- Официального гайда Kling по первому и последнему кадру.
- Минимальной длины плана, ниже которой модель его пропускает.
- Тестов листа персонажа против отдельных ракурсов на Kling, Veo, Omni.

## 4. Противоречия списком

1. Seedance: отрезки 2 с (как у Veo и Sora) или 4-10 с (Runway, Kapwing). Тестов нет.
2. Veo: таймкоды - многоплановая сцена (блог Google) или один непрерывный план (тест Wireflow). И описаны ли таймкоды в справке API.
3. Kapwing: «отрезок короче 8-10 с не успевает» против своих же примеров по 5-7 с.
4. Лист персонажа: ByteDance против, Starrd за.
5. Пересказывать внешность героя при опоре: Luma и RefVid против, Runway, fal и Omni (Cloud) за.
6. Формат реплики: разный у каждой модели, тестов нет.
7. Wan: гайд Alibaba даёт формулу многих планов, и та же страница пишет «One clip = one continuous shot». Возможно, это разные разделы или старая версия.
8. Veo 3.1: опора стиля упомянута в блоге запуска, в API есть только тип `asset`.

## 5. Источники

Документация разработчиков:

- https://ai.byteplus.com/ark/region:ap-southeast-1/docs/seedance-2-0-prompt-guide
- https://cloud.google.com/blog/products/ai-machine-learning/ultimate-prompting-guide-for-veo-3-1
- https://ai.google.dev/gemini-api/docs/veo
- https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/video/generate-videos-from-references
- https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/video/best-practice
- https://developers.openai.com/cookbook/examples/sora/sora2_prompting_guide
- https://kling.ai/blog/kling-video-3-multi-shot-guide
- https://www.alibabacloud.com/help/en/model-studio/text-to-video-prompt
- https://lumalabs.ai/learning-hub/luma-video-models-guide-ray3.14-veo-sora-kling-compared
- https://lumalabs.ai/learning-hub/ray3-modify-user-guide
- https://runway.com/resources/create-consistent-ai-characters
- Omni - см. `../google-omni-1.2.md`

Гайды посредников:

- https://runway.com/resources/seedance-2-5-prompt-guide
- https://fal.ai/learn/devs/seedance-2-5-prompting-guide
- https://lumalabs.ai/learning-center/articles/seedance-2-5-complete-guide

Тесты практиков:

- https://aireiter.com/blog/ai-video-generator-prompt-adherence-tested
- https://www.wireflow.ai/blog/veo-3-prompts
- https://www.wireflow.ai/blog/kling-ai-prompts
- https://www.atlascloud.ai/blog/tips/kling-4-dialogue-lip-sync-test
- https://eastondev.com/blog/en/posts/ai/20251207-veo3-audio-generation-guide/
- https://www.getstarrd.app/blog/seedance-character-sheet-vs-storyboard-grid
- https://aliteq.com/seedance-prompts
- https://dev.to/ref_vid/how-to-keep-the-same-character-across-ai-video-clips-with-reference-images-kling-30-omni-minimax-187f
- https://medium.com/@info.booststash/i-spent-two-weeks-pushing-kling-3-0-to-its-limits-heres-everything-nobody-is-telling-you-411cc0d49f51
- https://chasejarvis.com/blog/kling-3-0-ai-video-is-here-my-100-honest-review/
- https://medium.com/@social_18794/how-to-use-reference-video-in-seedance-2-0-to-copy-motion-camera-moves-6ee78dd117e7
- https://clipdance.ai/templates/depth-dance

Мнения без тестов:

- https://www.kapwing.com/resources/how-to-prompt-seedance-2-5-a-guide-for-ai-video-creators/
- https://www.screenweaver.ai/blog/prompting-veo-guide
- https://morphic.com/resources/how-to/seedance-2-guide
- https://www.bestseedanceprompts.com/blog/seedance-2-5-r2v-reference-guide
- https://www.michydev.com/kling-3-0-character-consistency-subject-binding-workflow/
- https://videoai.me/blog/kling-3-0-prompt-guide
- https://www.veo3ai.io/blog/veo-3-text-to-speech-voiceover-guide-2026
- https://www.promptfrenzy.com/prompt-university/ai-character-sheet-generator
- https://www.cineprompt.pro/blog/runway-gen-4-5-prompting-guide-for-consistent-shots
- https://danex.ai/runway-gen-4-character-consistency
- https://magichour.ai/blog/kling-30-reference-guide
- https://github.com/aedev-tools/kling-3-prompting-skill/blob/main/skills/kling-3-prompting/SKILL.md
