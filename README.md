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

#### Тематическое моделирование LDA <a name="041"></a>



#### Тематическое моделирование NMF <a name="042"></a>

#### Анализ с BERTopic <a name="043"></a>


### Ключевые выявленные тренды <a name="05"></a>
