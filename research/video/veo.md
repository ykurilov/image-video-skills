# Google Veo - заметки исследования

Дата: 10 ноября 2026. Цель: основа для файла Veo в универсальном скилле видеопромптов (как писать промпт, а не как звать API).

Пометки надёжности:

- **[док]** - документация или блог Google (Gemini API, Google Cloud, DeepMind).
- **[док пос]** - документация или страница посредника (Higgsfield и др.).
- **[CLI]** - живая схема `higgsfield model get <модель>`, снята 10 ноября 2026. Самое точное про то, что доступно пользователю через CLI.
- **[тест]** - практик показал свои прогоны. Число прогонов указано, если известно.
- **[мнение]** - утверждение без показанных тестов.

## 1. Какие версии актуальны

- **Новейшая Veo - 3.1.** Veo 4 официально не выходила: на сентябрь 2026 нет ни id модели, ни карточки, ни цен (zebracat, pixverse, evolink); новых объявлений в октябре-ноябре не найдено. [док пос; не найдено]
- **Семья 3.1:** `veo-3.1-generate-001` и `veo-3.1-fast-generate-001` - общий доступ (GA) на Google Cloud с 17 ноября 2025, снятие «May 4, 2027 or later»; `veo-3.1-lite-generate-001` - предварительная версия с 2 апреля 2026. [док Cloud]
- **В Gemini API предварительные Veo 3.1 закрыты 22 октября 2026.** Google предлагает переход на Gemini Omni 1.2 Flash или на GA-модели Google Cloud. Veo 3 и Veo 3 Fast помечены устаревшими. [док Gemini API]
- **Преемник по факту - Gemini Omni** (май 2026, 1.2 - ноябрь 2026). Google не называет её Veo 4. [док; док пос]
- Обновления 3.1 в 2026: 13 января - улучшение «Ingredients to Video» (по опорным картинкам) и вертикальный формат в YouTube Shorts и Gemini; 2 апреля - Lite. [док пос, пересказ поиска]
- **У пользователя через Higgsfield:** `veo3_1` (Google Veo 3.1) и `veo3_1_lite`. Совпадает с актуальной версией. [CLI] Changelog Higgsfield: 4K («Ultra HD») для Veo 3.1 с января 2026, Lite - с апреля. [док пос]

## 2. Как модель читает промпт

### Официальное

- **Формула Google для 3.1:** кинематография (план, движение камеры) + субъект + действие + контекст (место) + стиль и атмосфера. «A structured prompt yields consistent, high-quality results.» Камера - «the most powerful tool for conveying tone and emotion». [док, блог Cloud]
- Обязательные части по Gemini API: субъект, действие, стиль; необязательные: камера, композиция, фокус и объектив, атмосфера. Слово «portrait» усиливает детали лица. [док]
- **Veo нужны точные указания.** Руководство DeepMind по Omni прямо противопоставляет: Veo - точные инструкции, Omni - меньше предписаний. [док DeepMind]
- **Один момент на короткий ролик.** «For short videos, dedicate each prompt to a single, focused moment»; цепочка «A then B then C» «often leads to muddled or incomplete videos». [док Cloud, общая страница Veo и Omni]
- **Картинка→видео:** писать только движение, картинку не пересказывать («Redundant prompts confuse the model»); героя звать обобщённо («the woman», «he»); три вида движения: камера (самое надёжное), герой, окружение. [док Cloud]
- **Отрицания - описанием желаемого,** а не «no X»: «пустынный пейзаж без зданий и дорог» вместо «без построек». [док, блог Cloud] Vertex: избегать «no» и «don't», описывать, что не нужно видеть, коротким перечнем в поле отрицаний. [док пос screenweaver, пересказ Vertex]
- **Предел текста - 1024 токена.** [док Gemini API]
- **Язык промпта:** «English (EN) is fully supported», другие языки «have not been evaluated». На Cloud у всех трёх моделей язык промпта - английский. [док]
- Google советует просить Gemini обогатить простой промпт кинематографическим языком. [док]

### Опыты

- **Wireflow** (20 октября 2026, Veo 3.1 Fast, 8 промптов по одному прогону, 8 с, 720p, 26-76 слов): движение камеры выполнено наполовину (вперёд и вниз, без подъёма); модель добавила непрошеный дым из трубы, облака и мыс; действие с предметом изменено (поставила вне кадра вместо полки). [тест]
- **DreamPixelForge** (Veo 3.1 и Fast, по одному прогону): кадрирование «на вытянутой руке» дало более тесный план без руки; «общий план» вышел средним; натриевый янтарный свет - смешанная палитра. Стиль пластилина, названный первым словом, удался полностью. [тест]
- **Хабр studyai** (по 10-15 генераций на сценарий, ссылки на ролики в двух тестах не вставлены): модель на русском «понимает сюжет, но термины лучше давать по-английски»; синтаксис в стиле Midjourney «читает как мусор»; фоновых персонажей надо описывать, иначе все уставятся на главного героя; безобидное «shoot» может включить фильтр. [тест, частично без доказательств]
- **wan27.org:** 15-30 слов для полной модели, 10-15 для Lite. [мнение]

## 3. Тайминги, планы, порядок событий

- **Длительность - только 4, 6 или 8 с.** 8 с обязательны для 1080p, 4K, опорных картинок и продления. [док Gemini API] Через Higgsfield: `duration` 4 / 6 / 8, по умолчанию 8. У Lite с первым и последним кадром - только 8. [CLI]
- **Таймкоды официально есть** («timestamp prompting»): формат `[00:00-00:02] план + субъект + действие`, в образце Google - четыре куска по 2 с на 8 с, в каждом свой план и метки `SFX:` и `Emotion:`. Герой описан полностью в первом куске, дальше - коротким именем. Предела числа кусков Google не даёт. [док, блог Cloud]
- **Таймкоды дают порядок, но не время и не склейки:**
  - Wireflow (Fast, 1 прогон): три куска с разными планами - порядок и примерное время соблюдены, но «as one continuous move», склеек ноль. [тест]
  - DreamPixelForge (Fast, 1 прогон): два куска по 4 с - ровно одна склейка, два плана, порядок верный, но склейка на 2,83 с вместо 4,00. «buys you ordering, not timing». [тест]
  - InVideo (30+ роликов): попадание в таймкоды у Omni заметно точнее, чем у Veo 3.1. [тест]
  - Тип склейки модель выбирает сама (жёсткая или растворение). [мнение, DEV]
  - Заявление о точности «0,5 с» - из демо Google, независимо не подтверждено. [мнение]
- **Привязка звука к событию держится:** лайм ударил о воду на ~3,6 с, пик звука там же. [тест Wireflow]
- **Длительность против таймкодов.** Длина задаётся полем; Google о конфликте с таймкодами в тексте не пишет. vc.ru: при заказе 5 с модель выдала 8 - потому что 5 не поддерживается. [док; тест]
- **Сколько событий выдерживает ролик:** официально - один момент. Таймкоды Google - до 4 кусков по 2 с. Независимых замеров предела нет. Тестеры советуют ролик на план и монтаж. [док; мнение]

## 4. Референсы

- **Первый кадр** (`image`): «Think of your source image as the first frame of your film»; резкая, хорошо скомпонованная картинка. [док Cloud]
- **Первый и последний кадр** (`image` + `lastFrame`): последний только вместе с первым; плавный переход со звуком; в промпте описать переход и звук. [док] Форум разработчиков: пара кадров работает только при 8 с. [тест-отчёт форума]
- **Опорные картинки** («Ingredients to Video», `referenceImages`): до 3 картинок типа `asset` - один человек, персонаж или товар, модель «preserves the subject's appearance». В Vertex есть ещё тип `style` - не больше одной картинки (цвет, свет, фактура). Только 8 с. Lite опорных картинок не принимает. [док Gemini API, Vertex]
- **Синтаксиса ссылок нет.** Картинки называют словами: в образце Google - «Using the provided images for the detective, the woman, and the office setting…». [док] «has no token syntax». [док пос DreamPixelForge]
- **Запрещённые сочетания:** опорные картинки только без первого и последнего кадра (форум, aifreeapi). [тест-отчёт форума; мнение] Опорные картинки - только 8 с. [док]
- **Порядок важен:** главное (обычно персонаж) - первой картинкой. [мнение veo3ai]
- **Число опорных картинок - расхождение:** 3 (Google), 4 (getimg). У Fast опор нет (getimg), есть 1-3 (veo3flow). [док против док пос]
- **Что берётся:** внешность человека или товара - да; «близко, но не идентично» - типовая жалоба (media.io). Независимых замеров согласованности не найдено. [мнение]
- **Видео и звук на вход** для новой генерации не принимаются: «Audio input: Not supported». Видео - только для продления своего ролика Veo. [док Cloud]
- **Реальные люди:** `personGeneration` - для картинки→видео, кадров и опор только `allow_adult`; в ЕС, Великобритании, Швейцарии и MENA - только `allow_adult`. [док] DreamPixelForge: картинка→видео с фотореальным человеком «runs, bills nothing, and returns an empty result». [тест, 20 июля 2026]
- **У пользователя через Higgsfield CLI** (`veo3_1`): **только первый кадр** (`start_image`). Последнего кадра и опорных картинок в схеме нет. У `veo3_1_lite` - первый и последний кадр (`start_image`, `end_image`). [CLI] Скилл Higgsfield: Veo 3.1 «Max 1 reference», лишние картинки CLI отклоняет. [док скилла] Веб-сайт Higgsfield заявляет «Start & End Frame Mode» и «Multi-Image Reference Mode» (Standard - опоры 1-3, Fast - кадры). [док пос] Значит, в вебе возможностей больше, чем в CLI.

## 5. Речь, звук, надписи, русский язык

### Речь и звук

- Звук родной, синхронный: речь, эффекты, фон. «guided by the prompt». [док]
- **Формат реплик - Google противоречит сам себе:**
  - Gemini API и блог Cloud: «Use quotes for specific speech» - `A woman says, "We have to leave now."` [док]
  - Vertex «Best practices»: «use a colon (`:`) after the speaker's action … and avoid using quotation marks» - иначе слова рисуются в кадре; образец `A woman says: My name is Clara.` [док]
  - Replicate (Veo 3): двоеточие и «(no subtitles)». veo3ai: двоеточие - «the most consistent single fix reported by creators». HitPaw: ещё и без сокращений («I am» вместо «I'm»). GlobalGPT: кавычки и запрет субтитров в поле отрицаний. [мнение, сравнительных тестов на 3.1 нет]
  - Тесты речи в кавычках прошли без жалоб на субтитры: vc.ru (русская фраза дословно), DreamPixelForge (дословно), Wireflow (дословно). [тест] О субтитрах в этих тестах не сообщают.
- **Звук описывать отдельными предложениями** («use separate sentences»). Метки из блога Google: `SFX:` для эффектов, `Ambient noise:` для фона, подача голоса словами («says in a weary voice»). [док]
- Неописанный звук модель выбирает сама. [док пос, пересказ Google]
- **Темп речи:** около 1,9 слова в секунду (8 слов за 4,3 с). Длинная реплика на 39 слов обрезана на 31-м слове, и модель выбрала не того говорящего (молодую женщину). [тест Wireflow, Fast] Вывод: на 8 с - до ~15 слов.
- **Звук может провалиться:** шипение пропало в «цифровую тишину» на 2,6-3,5 с. [тест Wireflow] Генерацию может заблокировать проблема со звуком; такие не оплачиваются. [док]
- **Несколько говорящих:** реплики уходят не тому персонажу. Советы: один говорящий на ролик; метки «SPEAKER A:», «SPEAKER B:»; называть по виду и месту («женщина слева в красном»); «strict turn-taking, no overlap». [мнение, prompt-architects, veo3gen]
- **Голос между роликами не закрепить:** поля выбора голоса нет. Google советует одно и то же описание голоса слово в слово в каждом промпте и тот же seed. [док; мнение]

### Надписи в кадре

- Точный текст в кавычках выходит хорошо: меню, все слова и цены верны в каждом кадре. Вывески без заданного текста - выдуманные слова («CERALTER», «REDRF»). [тест Wireflow]
- **Кириллица в кадре плохая:** буквы «плывут», выходит «набор похожих букв». Совет: добавлять русский текст в монтаже или через готовый кадр и картинку→видео. [мнение/тест без цифр: Pikabu, Bratuha] Хабр: короткий тикер смешанным письмом остался узнаваемым. [тест без ролика]

### Русский язык

- Google русскую речь не оценивал (только английский). [док]
- **Тест vc.ru** (одна фраза с «щ», «ж», «ч», 2026): Veo 3.1 произнесла дословно, распознавание - русский с уверенностью 0,99. Губы не мерили. Лучший результат вместе с Omni. [тест]
- **Хабр studyai:** язык указывать явно («speaks in Russian»), иначе губы пойдут по-английски; реплики до 7 слов; при этом фраза из ~10 слов «движения рта попадают в слоги». [тест, частично без роликов; правило автора противоречит его же тесту]
- Pikabu: русская реплика внутри английского описания сцены звучит с правильным произношением. [мнение]
- Stiva.ai: модель может изменить слова или не договорить фразу - слушать каждую. [мнение]

## 6. Правка и продление

- **Правки готового ролика словами нет.** «Add/remove object» в Flow работает на модели Veo 2 и без звука. [док, блог Cloud] Через Higgsfield правки Veo нет. [CLI]
- **Продление** (Gemini API, когда работал): только для роликов Veo, +7 с за ход, до 20 ходов, вход до 141 с, итог до 148 с, только 720p, 16:9 или 9:16. Модель «finalizes the final second or 24 frames of your video and continues the action». Голос продлевается, только если звучит в последней секунде. Lite не продлевает. [док]
- **На Google Cloud цифры другие:** вход - ролик Veo длиной 1-30 с, +7 с; итог около 37 с. Artlist пишет 2-30 с. [док Vertex против док Gemini API; док пос]
- **Gemini API для Veo закрыт с 22 октября 2026** - схема 148 с там больше недоступна. [док]
- Через Higgsfield продления Veo нет. [CLI] Обходной путь - последний кадр как первый кадр следующего ролика. [мнение Atlas Cloud]

## 7. Поля запроса

### Higgsfield CLI (то, что у пользователя) [CLI]

| Модель | Поля и значения |
|---|---|
| `veo3_1` | `prompt` (обяз.), `aspect_ratio` 16:9 / 9:16 (16:9), `duration` 4 / 6 / 8 (8), `quality` basic / high / ultra (basic), `variant` veo-3-1-preview / veo-3-1-fast (fast), `start_image` |
| `veo3_1_lite` | `prompt` (обяз.), `aspect_ratio` 16:9 / 9:16 / auto, `duration` 4 / 6 / 8 (8; при двух кадрах только 8), `generate_audio` (false), `start_image`, `end_image` |

- **По умолчанию у `veo3_1` вариант Fast, а не полная модель.** [CLI]
- **Что значат basic / high / ultra, Higgsfield не пишет.** По changelog «Ultra HD» - это 4K. Связь high с 1080p - догадка. [док пос; не найдено]
- **У Lite звук по умолчанию выключен** (`generate_audio: false`). [CLI]
- Поля отрицаний, seed и числа роликов в схеме Higgsfield нет. [CLI]

### Google (Vertex) [док]

- `aspectRatio` 16:9 (по умолчанию) / 9:16; `resolution` 720p (по умолчанию) / 1080p / 4k (4K - только полная 3.1, только 8 с); `durationSeconds` 4 / 6 / 8; `generateAudio` (в Vertex есть, в Gemini API звук всегда включён); `negativePrompt`; `personGeneration`; `seed` («doesn't guarantee determinism, but slightly improves it»); `sampleCount` до 4 на Cloud (в Gemini API - 1); `image`, `lastFrame`, `referenceImages` (`referenceType` asset или style), `video` для продления.
- 24 кадра в секунду. Только регион `us-central1`. SynthID на всём. [док]
- Время генерации: от 11 с до 6 минут в часы пик. [док] На практике 85-271 с у Fast. [тест Wireflow]

## 8. Типичные провалы и что с ними делать

| Провал | Что делать | Лечится текстом? | Источник |
|---|---|---|---|
| Несколько событий слиплись или выпали | Один момент на ролик; история - несколько роликов | Да, разбивкой | [док] |
| Таймкоды не дали склеек или склейка не в ту секунду | Ролик на план и монтаж | Нет | [тест] |
| Камера выполнила движение наполовину | Одно простое движение | Частично | [тест Wireflow] |
| Непрошеные детали в кадре (дым, облака) | Описать нужную картину конкретно | Частично | [тест Wireflow] |
| Реплика проявилась субтитрами | Двоеточие вместо кавычек, «no subtitles» | Частично, без гарантий | [док Vertex; мнение] |
| Длинная реплика обрезана | До ~15 слов на 8 с | Да | [тест Wireflow] |
| Реплика не тому персонажу | Один говорящий на ролик; метки и внешность говорящего | Частично | [мнение] |
| Голос меняется между роликами | Одно описание голоса слово в слово, тот же seed (в Higgsfield seed нет); или голос отдельно | Слабо | [док; мнение] |
| Кириллица в кадре искажена | Надпись в монтаже или на первом кадре | Нет | [мнение] |
| Русская речь с английскими губами | Явно «speaks in Russian», короткая фраза, лицо в камеру | Частично | [мнение Хабр] |
| Выдуманные слова на вывесках | Точный текст в кавычках | Да | [тест Wireflow] |
| Картинка→видео с реальным человеком возвращает пусто | Другой кадр или модель | Нет | [тест DreamPixelForge] |
| Персонаж «похож, но не тот» | Опорные картинки (в CLI пользователя недоступны) | Нет | [мнение] |
| Сбой физики рук и предметов | Меньше рук, предмет закреплён, 3-4 попытки | Частично | [тест Хабр] |
| Нужна длина не 4/6/8 | Генерировать 8 и резать | Нет | [мнение] |

## 9. Противоречия и пробелы

1. **Реплики: кавычки или двоеточие.** Gemini API и блог Cloud - кавычки; Vertex «Best practices» - двоеточие без кавычек. Сравнительных тестов на 3.1 нет. Тесты с кавычками прошли без жалоб на субтитры.
2. **Число опорных картинок:** 3 (Google) против 4 (getimg); есть ли опоры у Fast - источники расходятся.
3. **Продление:** 148 с (Gemini API, закрыт) против ~37 с (Vertex) и вход 1-30 против 2-30 с (Artlist).
4. **Higgsfield: веб против CLI.** Сайт заявляет последний кадр и опорные картинки у Veo 3.1; в CLI у `veo3_1` только первый кадр.
5. **Значение quality basic / high / ultra** у Higgsfield не описано.
6. **Опорные картинки вместе с первым кадром** - запрет известен из форума и сторонних гидов, не из документации.
7. **Lite и картинки:** Cloud в одном месте пишет «Image Not supported», в другом - картинка→видео поддержана.
8. **Синхрон губ на русском** никто не мерил, только разборчивость.
9. **Veo 4** - не найдено ни одного официального подтверждения.
10. **Таймкоды на полной модели (не Fast)** - тестов не найдено; оба замера на Fast.

## 10. Источники

Официальные Google:

- https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/veo/3-1-generate
- https://aistudio.google.com/docs/veo
- https://aistudio.google.com/docs/video
- https://cloud.google.com/blog/products/ai-machine-learning/ultimate-prompting-guide-for-veo-3-1
- https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/video/best-practice
- https://docs.cloud.google.com/vertex-ai/generative-ai/docs/video/extend-a-veo-video
- https://docs.cloud.google.com/vertex-ai/generative-ai/docs/model-reference/veo-video-generation
- https://deepmind.google/models/veo/
- https://deepmind.google/models/gemini-omni/prompt-guide/
- https://discuss.ai.google.dev/t/how-to-correctly-structure-object-if-using-first-and-last-frames-and-reference-images-for-veo-3-1-endpoint/111495
- https://discuss.ai.google.dev/t/veo-3-1-reference-images-docs-say-available-api-says-not-supported/111853

Посредники:

- `higgsfield model get veo3_1`, `higgsfield model get veo3_1_lite` (10 ноября 2026)
- https://higgsfield.ai/veo3.1
- https://higgsfield.ai/blog/How-to-Use-Google-Veo-3.1-Complete-Guide-for-the-New-Model
- https://higgsfield.ai/creator-hub/changelog
- https://www.screenweaver.ai/blog/prompting-veo-guide
- https://replicate.com/blog/using-and-prompting-veo-3
- https://getimg.ai/blog/google-veo-3-1-review
- https://www.aifreeapi.com/en/posts/veo-3-1-modes-tutorial
- https://artlist.io/blog/new-veo-3-1-extend/
- https://www.atlascloud.ai/blog/tips/veo-3-longer-than-8-seconds
- https://www.zebracat.ai/features/veo-4-video-generator
- https://pixverse.ai/en/blog/veo-4-release

Практики:

- https://www.wireflow.ai/blog/veo-3-prompts
- https://www.dreampixelforge.com/blog/veo-3-prompts
- https://invideo.io/faq/how-well-does-google-gemini-omni-flash-follow-time/
- https://vc.ru/promtstudio/3046354-neyroseti-dlya-video-s-ozvuchkoy
- https://habr.com/ru/companies/study_ai/articles/1078096/
- https://pikabu.ru/story/google_veo_31__novaya_versiya_neyroseti_dlya_generatsii_video_13294072
- https://bratuha.ru/tools/veo-3-1
- https://stiva.ai/models/veo-3-1
- https://www.veo3ai.io/blog/veo-3-remove-subtitles-captions-fix-2026
- https://edimakor.hitpaw.com/subtitle-tips/veo3-subtitles.html
- https://www.glbgpt.com/hub/how-to-make-characters-speak-in-veo-3-1-the-ultimate-guide-to-dialogue-audio-lip-sync/
- https://prompt-architects.com/blog/101-veo-dialogue-prompts
- https://www.veo3gen.app/blog/veo-31-dialogue-sfx-prompts-that-actually-sync-a-creator-workflow-for-clean-mout
- https://dev.to/akaranjkar08/veo-31-developer-guide-timestamp-prompting-multi-shot-video-and-full-api-june-2026-3l5i
- https://wan27.org/blog/veo-3-1-prompt-guide
