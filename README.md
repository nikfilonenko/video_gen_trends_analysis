# About the project ***`video_gen_trends_analysis`***

## 📊 Ноутбук с выполненным заданием: [Здесь](src/notebooks/arXiv_video_generation_trends.ipynb)

`src/notebooks/arXiv_video_generation_trends.ipynb`

<br>

## 📃 Ноутбук на google colab: [Здесь](https://colab.research.google.com/drive/1CgY7crZzAQyeoNTpKLkrM6AvO8LLqc-C?usp=sharing)

`google colab`

### Содержание

1. [Описание проекта](#01)
2. [Сбор данных через arXiv API](#02)
3. [EDA + предобработка текста](#03)
4. [Анализ основных тем/трендов](#04)
   - [Тематическое моделирование LDA](#041)
   - [Тематическое моделирование NMF](#042)
   - [Анализ с BERTopic](#043)
5. [Ключевые выявленные тренды](#05)
6. [Вывод](#06)

### Описание проекта <a name="01"></a>

**Цель проекта** - выявить ключевые тренды в области генерации видео в 2024 году на основе анализа научных статей из arXiv.

**Реализовано:**

- Сбор 2100 статей за 2024 год через arXiv API
    
- Комплексная предобработка текстовых данных
    
- Применение трех методов тематического моделирования:
    
    - LDA (Latent Dirichlet Allocation)
        
    - NMF (Non-Negative Matrix Factorization)
        
    - BERTopic (на основе эмбеддингов)
        
- Сравнительный анализ результатов
    
- Визуализация и интерпретация трендов


### Сбор данных через arXiv API <a name="02"></a>

Для сбора данных использовался официальный API arXiv, который предоставляет удобный доступ к метаданным научных статей

Также была использована библиотека arXiv, которая предоставляет удобную высокоуровневую абстракцию для работы с API arXiv

- структура выполненного запроса:

```python
query = (
    'all:video generation OR all:video synthesis OR '
    'all:diffusion video OR all:GAN video OR '
    'all:text-to-video OR all:image-to-video OR '
    'all:scene generation '
    'AND submittedDate:[20240101 TO 20241231]'
)
```

В дополнение к этому:

- была выполнена ленивая загрузка (использование генератора для экономии памяти)
- была выполнена дополнительная фильтрация по году публикации

#### Результаты сбора:

- Всего собрано статей: **2100**
    
- Период: только **2024 год**

    
Структура данных:
    
 - Заголовок (`title`)
     
 - Аннотация (`abstract`)
     
 - Дата публикации (`published`)
     
 - Список авторов (`authors`)
     
 - Ссылка на статью (`url`)


### EDA + предобработка текста <a name="03"></a>

В этой часте задания я провел EDA и подготовку текста для дальнейшего тематического моделирования 

#### Получение общего представления о данных

Базовый EDA показал, что в датасете нет пропущенных значений, а все столбцы представлены в формате `object` (строка). Это логично, учитывая текстовую природу данных. Однако столбец `published` необходимо привести к типу `datetime`, чтобы выделить дополнительные признаки — год и месяц публикации. Это поможет глубже исследовать временные тенденции.

Ключевые наблюдения:

- Столбцы `title`, `abstract` и `url` содержат полностью уникальные значения (2100 из 2100).
    
- В `published` 2098 уникальных значений, что указывает на публикации с одинаковой временной меткой.
    
- `authors` содержит 2091 уникальное значение — это означает, что некоторые авторские коллективы публиковали несколько статей в течение года.
    

В целом, структура данных предсказуемая, но требует нормализации и очистки. Встречаются нежелательные символы (например, `\n`), которые нужно удалить. Приведение `published` к `datetime` позволило дополнительно выделить `year` и `month`. Анализ распределения публикаций показал, что активность авторов была высокой в конце 2024 года, а также в марте.

---

#### Подготовка данных для анализа тем и трендов в видео генерации

Для дальнейшего тематического анализа я сформировал единый текстовый столбец, объединяющий `title` и `abstract`, так как именно он содержит основную информацию о содержании статей.

Далее выполнил предобработку текста:

- Привел текст к нижнему регистру;
    
- Удалил специальные символы и лишние пробелы с помощью регулярных выражений;
    
- Исключил стоп-слова для повышения качества анализа;
    
- Выполнил токенизацию;
    
- Применил лемматизацию (только для count-based методов, таких как LDA и NMF, чтобы уменьшить размерность признакового пространства, но не для продвинутых, например BERTopic, чтобы не терять ценной информации в сформированных эмбеддингах).
    

Лемматизацию **не применял** для методов, основанных на эмбеддингах (например, BERTopic), так как она могла бы исказить смысл научных терминов и снизить качество тематического моделирования.

- Пример очищенного текста после лемматизации:

```
video video generative adversarial network fewshot learning based policy gradient development sophisticated model videotovideo synthesis facilitated recent advance deep reinforcement learning generative adversarial network gans paper propose new deep neural network approach based reinforcement learning unsupervised conditional videotovideo synthesis preserving unique style source video domain approach aim learn mapping source video domain target video domain train model using policy gradient employ convlstm layer capture spatial temporal information designing finegrained gan architecture incorporating spatiotemporal adversarial goal adversarial loss aid content translation preserving style unlike traditional videotovideo synthesis method requiring paired input proposed approach general require paired input thus dealing limited video target domain ie fewshot learning particularly effective experiment show produce temporally coherent video result result highlight potential approach advance videotovideo synthesis
```


- Пример очищенного текста БЕЗ лемматизации:

```
video video generative adversarial network fewshot learning based policy gradient development sophisticated models videotovideo synthesis facilitated recent advances deep reinforcement learning generative adversarial networks gans paper propose new deep neural network approach based reinforcement learning unsupervised conditional videotovideo synthesis preserving unique style source video domain approach aims learn mapping source video domain target video domain train model using policy gradient employ convlstm layers capture spatial temporal information designing finegrained gan architecture incorporating spatiotemporal adversarial goals adversarial losses aid content translation preserving style unlike traditional videotovideo synthesis methods requiring paired inputs proposed approach general require paired inputs thus dealing limited videos target domain ie fewshot learning particularly effective experiments show produce temporally coherent video results results highlight potential approach advances videotovideo synthesis
```

Для предобработки текста я  также написал кастомный препроцессор `CustomTextPreprocessor`, который выполняет очистку, токенизацию, удаление стоп-слов и, при необходимости, лемматизацию.

```python
import nltk

nltk.download('punkt')
nltk.download('stopwords')
nltk.download('wordnet')

from nltk.corpus import stopwords
from nltk.tokenize import word_tokenize
from nltk.stem import WordNetLemmatizer
import re

class CustomTextPreprocessor:
    def __init__(self, lemmatization=False):
        self.lemmatization = lemmatization
        self.stop_words = set(stopwords.words('english'))
        self.lemmatizer = WordNetLemmatizer()
        self.regex_pattern = r'[^\w\s]'

    def preprocess(self, text: pd.Series) -> pd.DataFrame:
        cleaned_txt = text.apply(lambda x: self._clean_text(x))

        if self.lemmatization:
            cleaned_txt = cleaned_txt.apply(lambda x: self._lemmatize_txt(x))

        result_df = pd.DataFrame({
            'text': cleaned_txt
        })
        return result_df

    def _clean_text(self, text: str) -> str:
        if not isinstance(text, str):
            return ""

        text = re.sub(self.regex_pattern, '', text)

        tokens = word_tokenize(
            text.lower()
        )

        tokens = [word for word in tokens if word not in self.stop_words and word.isalpha()]
        return " ".join(tokens)

    def _lemmatize_txt(self, text: str) -> str:
        if not isinstance(text, str) or not text:
            return ""

        tokens = word_tokenize(text)
        lemmas = [self.lemmatizer.lemmatize(token) for token in tokens]
        return " ".join(lemmas)
```


Таким образом, после этапа EDA данные были очищены, нормализованы и готовы к дальнейшему анализу.





### Анализ основных тем/трендов <a name="04"></a>

#### Тематическое моделирование LDA <a name="041"></a>

В ходе исследования я провел серию экспериментов с LDA (Latent Dirichlet Allocation), чтобы выявить скрытые тематические структуры в научных публикациях о генерации видео. Вот ключевые моменты этого анализа:

**Выбор параметров модели:**

- Использовал Bag-of-Words (BoW) представление текста, так как LDA работает с частотными характеристиками слов
    
- Тестировал количество тем в диапазоне от 3 до 55 с шагом 2
    
- Для оценки качества использовал метрику Coherence Score
    

**Результаты экспериментов:**

- При малом количестве тем (3-10) получались слишком общие, плохо интерпретируемые темы с низким Coherence Score (~0.2)
    
- Максимальный Coherence Score (0.366) достигнут при 43 темах
    
- Оптимальный баланс между качеством и интерпретируемостью найден при 29 темах (Coherence Score 0.364)
    

**Проблемы и решения:**

- Обнаружил, что многие темы содержат малосодержательные слова ("model", "method", "using")
    
- Добавил эти термины в кастомные стоп-слова, что значительно улучшило качество тем
    
- При 43 темах наблюдалась избыточная фрагментация - близкие темы делились на несколько мелких
    

**Интерпретация результатов:**  
После обучения модели с 29 темами провел два типа анализа:

1. Ручная интерпретация ключевых слов - выделил 7 основных трендов
    
2. Автоматизированная кластеризация тем с помощью BERT+UMAP+HDBSCAN - получил 5 семантических кластеров
    

**Выявленные тренды:**

1. Доминирование diffusion-моделей в генерации видео
    
2. Применение в медицине (анализ МРТ/КТ)
    
3. Мультимодальные системы (видео+текст+аудио)
    
4. Генерация движений и анимации
    
5. Повышение качества видео (HDR, optical flow)
    

#### Тематическое моделирование NMF <a name="042"></a>

Для сравнения с LDA применил метод NMF (Non-Negative Matrix Factorization) с TF-IDF представлением текста:

**Особенности подхода:**

- TF-IDF лучше подходит для NMF, чем BoW
    
- Тестировал модели с 10 и 19 темами
    
- При 10 темах получались слишком общие категории
    
- 19 тем дали хороший баланс между детализацией и интерпретируемостью
    

**Уникальные находки NMF:**

1. Направление нейросетевого сжатия видео (тема 16):
    
    - Ключевые слова: compression, coding, codecs
        
    - Применение: потоковое видео, уменьшение задержек
        
2. Детекция deepfake (тема 12):
    
    - Ключевые слова: detection, anomaly, forgery
        
    - Актуально для борьбы с фейками
        
3. Audio-to-Video преобразование (тема 13):
    
    - Ключевые слова: audio, music, alignment
        
    - Перспективное направление для медиаиндустрии
        

**Преимущества NMF:**

- Лучше выделяет технические аспекты
    
- Дает более четкие границы между темами
    
- Хорошо подходит для анализа нишевых направлений
    

#### Анализ с BERTopic <a name="043"></a>


### Ключевые выявленные тренды <a name="05"></a>

Суммаризируя вместе результаты всех трех методов, можно выделить 13 устойчивых направлений:


Сводная таблица 13 ключевых трендов:

| №   | Тренд                                                       | Метод    | Характеристики                                                                                                                                                                                                                                    |
| --- | ----------------------------------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Генерация видео на основе diffusion-моделей                 | LDA      | Diffusion-модели стали стандартом для создания реалистичного видео                                                                                                                                                                                |
| 2   | Повышение качества видео: HDR и динамические сцены          | LDA      | HDR (расширенный динамический диапазон) стал стандартом; Генеративные модели помогают ретушировать и апскейлить видео                                                                                                                             |
| 3   | Распознавание + генерация движений                          | LDA      | Анимация персонажей (pose, action) выходит на новый уровень; Распознавание действий в видео и их дорисовка становится всё точнее                                                                                                                  |
| 4   | Мультимодальность: видео + текст + аудио                    | LDA      | Всё больше технологий для обработки длинных видео (long-context)                                                                                                                                                                                  |
| 5   | Применение видеогенерации в робототехнике и медицине        | LDA      | Роботы учатся видеть и понимать окружающий мир через видео (obstacle avoidance, event cameras);<br>    <br>В медицине видеогенерация помогает анализировать MRI, CT и ultrasound                                                                  |
| 6   | Оптимизация качества и разрешения видео через AI-компрессию | NMF      | AI-методы компрессии позволяют сжимать видео без потери качества;<br>    <br>Улучшенные кодеки (neural video coding, video compression) снижают нагрузку на сеть;<br>    <br>AI-декодирование сокращает задержки в потоковом видео;               |
| 7   | AI-редактирование видео                                     | NMF      | Генеративное редактирование видео: объекты и сцены можно менять автоматически;<br>    <br>AI-видеомонтаж и интеллектуальные фильтры делают обработку проще;<br>    <br>Контекстное редактирование изображений в видео — тренд для кино и рекламы; |
| 8   | Детекция deepfake и выявление аномалий                      | NMF      | Постоянно появляются новые AI-методы обнаружения deepfake;<br>    <br>Видеофальсификации сложно выявить, поэтому растёт спрос на автоматизированные системы;<br>    <br>Компании ищут алгоритмы поиска подозрительных изменений в видео;          |
| 9   | Audio to Video                                              | NMF      | Синхронизация аудиофайлов с видео открывает новые возможности для контента;<br>    <br>Автоматическая генерация видеоряда под звук (музыка, речь, эффекты);                                                                                       |
| 10  | Talking head: интерактивные аватары                         | BERTopic | Это уже не просто генерация лиц, а создание реалистичных интерактивных аватаров;<br>    <br>Используется для виртуальных помощников, customer service и анимации;                                                                                 |
| 11  | Policy learning для роботов через видео                     | BERTopic | Роботы учатся не просто распознавать объекты, но и действовать на основе видео;<br>    <br>Видео становится основным источником обучения для policy learning;                                                                                     |
| 12  | SORA как бизнес-тренд                                       | BERTopic | Это не просто diffusion, а новый виток развития AI-видео;<br>    <br>Активное применение в рекламе, кино и digital production;                                                                                                                    |
| 13  | Quality Assessment для соцсетей                             | BERTopic | Растёт запрос на автоматизацию модерации видео;<br>    <br>AI-фильтрация контента помогает соблюдать платформенные стандарты;                                                                                                                     |



### Вывод <a name="06"></a>

Проведенное исследование позволило систематизировать и проанализировать ключевые направления в области генерации видео по данным научных публикаций 2024 года. 

Вот основные итоги:

- Комбинация трех подходов (LDA, NMF, BERTopic) доказала свою эффективность, позволив получить согласованные результаты
    
- Гибридная методика (LDA + кластеризация эмбеддингов) показала особую ценность для выявления скрытых взаимосвязей
    
- Разработанный кастомный пайплайн предобработки текста успешно адаптирован под специфику научных публикаций

- Diffusion-модели подтвердили статус основного инструмента генерации видео
    
- Выявлен рост интереса к прикладным аспектам: от медицинской визуализации до коммерческих решений (SORA)
    
- Обнаружены перспективные нишевые направления (Audio-to-Video, интерактивные аватары)

