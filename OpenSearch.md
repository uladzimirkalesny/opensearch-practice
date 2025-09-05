_Роль анализаторов в OpenSearch
=================================
Компоненты анализаторов

- Нормализатор
- Токенизатор
- Фильтр лексем
- Фильтры символов

-----------------------
Стандартный анализатор в OpenSearch состоит из трех основных компонентов:

1. **Фильтры символов**: Преобразуют текст до его токенизации (например, удаление HTML-тегов (`html_strip`), замена
   определенных символов).
2. **Токенизатор**: Разбивает текст на отдельные токены на основе определенных правил (пробельные символы, знаки
   препинания и т.д.).
3. **Фильтры токенов**: Преобразуют токены (например, приведение к нижнему регистру (`lowercase`), удаление стоп-слов,
   стемминг).

**Стемминг** - это процесс приведения слова к его корню (например, "_running_" становится "_run_").

Анализаторы - это специальные алгоритмы, которые преобразуют значения строкового поля в термины и созраняют в виде
инвертированного индекса.

Анализаторы используются как при индексировании, так и при поиске для обеспечения совместимости в ходе поиска.

-----------------------

Пример кастомного анализатора (`PUT /index_name`):

```json
{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "stop"
          ]
        }
      }
    }
  }
}
```

В этом примере создается кастомный анализатор `custom_analyzer`, который использует стандартный токенизатор и два
фильтра: `lowercase` (приведение к нижнему регистру) и `stop` (удаление стоп-слов).

- Type `custom` - указывает, что это кастомный анализатор
- Tokenizer `standard`  - разбивает текст на токены по пробелам и знакам препинания
- Фильтр `lowercase` - приводит все токены к нижнему регистру
- Фильтр `stop` - удаляет распространенные слова (например, "a", "the", "and"), которые не несут значимой информации для
  поиска

после чего можно использовать этот анализатор в маппинге поля (`PUT /index_name/_mapping`):

```json
{
  "properties": {
    "content": {
      "type": "text",
      "analyzer": "custom_analyzer"
    }
  }
}
```

или при поиске (`GET /index_name/_search`):

```json 
{
  "query": {
    "match": {
      "content": {
        "query": "The quick brown fox",
        "analyzer": "custom_analyzer"
      }
    }
  }
}
```

-----------------------

Чтобы изменить кастомный анализатор, нужно:

- вначале закрыть индекс(`POST /index_name/_close`),
- затем обновить настройки индекса с новым определением анализатора (`PUT /index_name/_settings`),

```json
{
  "index": {
    "analysis": {
      "analyzer": {
        "customHTMLSnowball": {
          "type": "custom",
          "char_filter": [
            "html_strip"
          ],
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "stop",
            "snowball"
          ]
        }
      }
    }
  }
}
```

- после этого снова открыть индекс (`POST /index_name/_open`).


- Фильтр символов `html_strip` - удаляет HTML-теги из текста.
- Токенизатор `standard` - разбивает текст на токены по пробелам и знакам препинания.
- Фильтр токенов `lowercase` - приводит все токены к нижнему регистру.
- Фильтр токенов `stop` - удаляет распространенные слова (например, "a", "the", "and"), которые не несут значимой
  информации для поиска.
- Фильтр токенов `snowball` - применяет алгоритм стемминга Snowball, который сокращает слова до их корней (например, "
  running" становится "run").

----------------------

Применение анализатора к любому полю в mapping (`PUT /index_name`):

```json
{
  "mappings": {
    "properties": {
      "text": {
        "type": "text",
        "fields": {
          "english": {
            "type": "text",
            "analyzer": "english"
          }
        }
      }
    }
  }
}
```

В этом примере поле `text` имеет подполе `english`, которое использует встроенный анализатор `english`.\
Этот анализатор включает в себя токенизацию, приведение к нижнему регистру, удаление стоп-слов и стемминг, что делает
его подходящим для обработки английского текста.\
Для поля `text` будет использоваться стандартный анализатор по умолчанию.

Используя API `_analyze`, можно протестировать, как анализатор обрабатывает текст (`GET /index_name/_analyze`):

```json
{
  "field": "text",
  "text": "Opensearch is an awesome search engine"
}
```

Response проверки анализатора для поля `text`:

```json
{
  "tokens": [
    {
      "token": "opensearch",
      "start_offset": 0,
      "end_offset": 10,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "is",
      "start_offset": 11,
      "end_offset": 13,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "an",
      "start_offset": 14,
      "end_offset": 16,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "awesome",
      "start_offset": 17,
      "end_offset": 24,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "search",
      "start_offset": 25,
      "end_offset": 31,
      "type": "<ALPHANUM>",
      "position": 4
    },
    {
      "token": "engine",
      "start_offset": 32,
      "end_offset": 38,
      "type": "<ALPHANUM>",
      "position": 5
    }
  ]
}
```

или

```json
{
  "field": "text.english",
  "text": "Opensearch is an awesome search engine"
}
```

Response проверки анализатора для поля `text.english`:

```json
{
  "tokens": [
    {
      "token": "opensearch",
      "start_offset": 0,
      "end_offset": 10,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "awesom",
      "start_offset": 17,
      "end_offset": 24,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "search",
      "start_offset": 25,
      "end_offset": 31,
      "type": "<ALPHANUM>",
      "position": 4
    },
    {
      "token": "engin",
      "start_offset": 32,
      "end_offset": 38,
      "type": "<ALPHANUM>",
      "position": 5
    }
  ]
}
```

В этом примере видно, что анализатор `english` удалил стоп-слова "_is_" и "_an_", а также применил стемминг к словам
"_awesome_" и "_engine_", сократив их до "_awesom_" и "_engin_".

-----------------------

Default анализаторы в OpenSearch:

- `standard` - стандартный анализатор, который разбивает текст на токены по пробелам и знакам препинания, а также
  приводит их к нижнему регистру и удаляет распространенные стоп-слова.
    - `POST _analyze`
      ```json
      {
        "analyzer": "standard",
        "text": "Opensearch is an awesome search engine"
      }
      ```
    - Response:
    ```json
        {
          "tokens": [
            {
              "token": "opensearch",
              "start_offset": 0,
              "end_offset": 10,
              "type": "<ALPHANUM>",
              "position": 0
            },
            {
              "token": "is",
              "start_offset": 11,
              "end_offset": 13,
              "type": "<ALPHANUM>",
              "position": 1
            },
            {
              "token": "an",
              "start_offset": 14,
              "end_offset": 16,
              "type": "<ALPHANUM>",
              "position": 2
            },
            {
              "token": "awesome",
              "start_offset": 17,
              "end_offset": 24,
              "type": "<ALPHANUM>",
              "position": 3
            },
            {
              "token": "search",
              "start_offset": 25,
              "end_offset": 31,
              "type": "<ALPHANUM>",
              "position": 4
            },
            {
              "token": "engine",
              "start_offset": 32,
              "end_offset": 38,
              "type": "<ALPHANUM>",
              "position": 5
            }
        ]
      }
    ```
- `simple` - простой анализатор, который разбивает текст на токены по небуквенно-цифровым символам и приводит их к
  нижнему регистру.
    - `POST _analyze`
      ```json
      {
        "analyzer": "simple",
        "text": "Opensearch, is an awesome search engine!"
      }
      ```
    - Response:
    ```json
        {
          "tokens": [
            {
              "token": "opensearch",
              "start_offset": 0,
              "end_offset": 10,
              "type": "word",
              "position": 0
            },
            {
              "token": "is",
              "start_offset": 12,
              "end_offset": 14,
              "type": "word",
              "position": 1
            },
            {
              "token": "an",
              "start_offset": 15,
              "end_offset": 17,
              "type": "word",
              "position": 2
            },
            {
              "token": "awesome",
              "start_offset": 18,
              "end_offset": 25,
              "type": "word",
              "position": 3
            },
            {
              "token": "search",
              "start_offset": 26,
              "end_offset": 32,
              "type": "word",
              "position": 4
            },
            {
              "token": "engine",
              "start_offset": 33,
              "end_offset": 39,
              "type": "word",
              "position": 5
            }
        ]
      }
    ```
- `whitespace` - анализатор, который разбивает текст на токены только по пробелам, сохраняя регистр символов.
    - `POST _analyze`
      ```json
      {
        "analyzer": "whitespace",
        "text": "Opensearch, is an awesome search engine!"
      }
      ```
    - Response:
    ```json
        {
          "tokens": [
            {
              "token": "Opensearch,",
              "start_offset": 0,
              "end_offset": 11,
              "type": "word",
              "position": 0
            },
            {
              "token": "is",
              "start_offset": 12,
              "end_offset": 14,
              "type": "word",
              "position": 1
            },
            {
              "token": "an",
              "start_offset": 15,
              "end_offset": 17,
              "type": "word",
              "position": 2
            },
            {
              "token": "awesome",
              "start_offset": 18,
              "end_offset": 25,
              "type": "word",
              "position": 3
            },
            {
              "token": "search",
              "start_offset": 26,
              "end_offset": 32,
              "type": "word",
              "position": 4
            },
            {
              "token": "engine!",
              "start_offset": 33,
              "end_offset": 40,
              "type": "word",
              "position": 5
            }
         ]
      }
    ```
    - `stop` - анализатор, который удаляет распространенные стоп-слова из текста (например, "a", "the", "and").
        - `POST _analyze`
          ```json
          {
            "analyzer": "stop",
            "text": "Opensearch, is an awesome search engine!"
          }
          ```
        - Response:
        ```json
        {
          "tokens": [
            {
              "token": "opensearch",
              "start_offset": 0,
              "end_offset": 10,
              "type": "word",
              "position": 0
            },
            {
              "token": "awesome",
              "start_offset": 18,
              "end_offset": 25,
              "type": "word",
              "position": 3
            },
            {
              "token": "search",
              "start_offset": 26,
              "end_offset": 32,
              "type": "word",
              "position": 4
            },
            {
              "token": "engine",
              "start_offset": 33,
              "end_offset": 39,
              "type": "word",
              "position": 5
            }
          ]
      }
      ```
- `pattern` - анализатор, который разбивает текст на токены на основе заданного регулярного выражения.
  Полезен для специфичных форматов текста. Он также поддерживает стоп-слова и преобразует токены к строчному виду.
    - `PUT my_index2`
      ```json
      {
        "settings": {
          "analysis": {
            "analyzer": {
              "my_email_analyzer": {
                "type": "pattern",
                "pattern": "\\W|_",
                "lowercase": true
              }
            }
          }
        }
      }
      ```
    - `POST my_index2/_analyze`
        ```json
        {
          "analyzer": "my_email_analyzer",
          "text": "sophia.julian@yopmail.com"
        }
      ```
    - Response:
      ```json
        {
          "tokens": [
            {
              "token": "sophia",
              "start_offset": 0,
              "end_offset": 6,
              "type": "word",
              "position": 0
            },
            {
              "token": "julian",
              "start_offset": 7,
              "end_offset": 13,
              "type": "word",
              "position": 1
            },
            {
              "token": "yopmail",
              "start_offset": 14,
              "end_offset": 21,
              "type": "word",
              "position": 2
            },
            {
              "token": "com",
              "start_offset": 22,
              "end_offset": 25,
              "type": "word",
              "position": 3
            }
         ]
      }
      ```
- `keyword` - анализатор, который не разбивает текст на токены, а сохраняет его как есть. Полезен для точного поиска.
    - POST _analyze
       ```json
         {
           "analyzer": "keyword",
           "text": "Opensearch, is an awesome search engine!"
         }
       ```
        - Response:
       ```json
         {
           "tokens": [
             {
               "token": "Opensearch, is an awesome search engine!",
               "start_offset": 0,
               "end_offset": 40,
               "type": "word",
               "position": 0
             }
           ]
         }
       ```
- `language` - набор анализаторов для различных языков (например, `english`, `french`, `german`), которые включают в
  себя токенизацию, приведение к нижнему регистру, удаление стоп-слов и стемминг, адаптированные для конкретного языка.
    - PUT /english_index
      ```json
      {
        "settings": {
          "analysis": {
            "filter": {
              "english_stop": {
                "type": "stop",
                "stopwords": "_english_"
              },
              "english_keywords": {
                "type": "keyword_marker",
                "keywords": ["example"]
              },
              "english_stemmer": {
                "type": "stemmer",
                "language": "english"
              },
              "english_possessive_stemmer": {
                "type": "stemmer",
                "language": "possessive_english"
              }
            },
            "analyzer": {
              "rebuilt_english": {
                "tokenizer": "standard",
                "filter": [
                 "english_possessive_stemmer",
                 "lowercase",
                 "english_stop",
                 "english_keywords",
                 "english_stemmer"
               ]
              }  
            }
          }
        }
      }
      ```
- `fingerprint` - преобразует текст к строчному виду, удаляет символы расширенных наборов, дедуплицирует и объединяет
  слова в один токен. При составлении запроса с помощью анализатора отпечатков стоп-слова удаляются. `Дедупликация` -
  это процесс удаления повторяющихся слов.
    - `POST _analyze`
      ```json
      {
        "analyzer": "fingerprint",
        "text": "Opensearch, is an awesome search engine! awesome"
      }
      ```
    - Response:
      ```json
      {
       "tokens": [
         {
          "token": "an awesome engine is opensearch search",
          "start_offset": 0,
          "end_offset": 48,
          "type": "fingerprint",
          "position": 0
         }
        ]
       }
       ```

Роль токенезаторов в OpenSearch
=================================

- **Токенизаторы** получают поток символов из строки и преобразуют его в отдельные слова, называемые токенами.
- **Токенизаторы** также отслеживают порядок токенов, учитывая смещение начального и конечного символов.

- `Standard` - использует алгоритм Unicode Text Segmentation для генерации токенов на основе грамматики.
    - Поддеживает различные языки и символы.
    - `POST _analyze`
      ```json
      {
        "tokenizer": "standard",
        "text": "OpenSearch, is an awesome search engine!"
      }
      ```
    - Response:
      ```json
      {
        "tokens": [
          {
            "token": "OpenSearch",
            "start_offset": 0,
            "end_offset": 10,
            "type": "<ALPHANUM>",
            "position": 0
          },
          {
            "token": "is",
            "start_offset": 12,
            "end_offset": 14,
            "type": "<ALPHANUM>",
            "position": 1
          },
          {
            "token": "an",
            "start_offset": 15,
            "end_offset": 17,
            "type": "<ALPHANUM>",
            "position": 2
          },
          {
            "token": "awesome",
            "start_offset": 18,
            "end_offset": 25,
            "type": "<ALPHANUM>",
            "position": 3
          },
          {
            "token": "search",
            "start_offset": 26,
            "end_offset": 32,
            "type": "<ALPHANUM>",
            "position": 4
          },
          {
            "token": "engine",
            "start_offset": 33,
            "end_offset": 39,
            "type": "<ALPHANUM>",
            "position": 5
          }
        ]
      }
      ```
- `custom`
    - `PUT my_index3`
      ```json
        {
          "settings": {
            "analysis": {
              "analyzer": {
                "my_analyzer": {
                  "tokenizer": "my_tokenizer"
                }
              },
              "tokenizer": {
                "my_tokenizer": {
                  "type": "standard",
                  "max_token_length": 6
                }
              }
           }
          }
        }
      ```
    - `POST my_index3/_analyze`
        ```json
        {
            "analyzer": "my_analyzer",
            "text": "Opensearch, is an awesome search engine!"
        }
        ```
    - Response:
      ```json
      {
        "tokens": [
          {
            "token": "Opense",
            "start_offset": 0,
            "end_offset": 6,
            "type": "<ALPHANUM>",
            "position": 0
          },
          {
            "token": "arch",
            "start_offset": 6,
            "end_offset": 10,
            "type": "<ALPHANUM>",
            "position": 1
          },
          {
            "token": "is",
            "start_offset": 12,
            "end_offset": 14,
            "type": "<ALPHANUM>",
            "position": 2
          },
          {
            "token": "an",
            "start_offset": 15,
            "end_offset": 17,
            "type": "<ALPHANUM>",
            "position": 3
          },
          {
            "token": "awesom",
            "start_offset": 18,
            "end_offset": 24,
            "type": "<ALPHANUM>",
            "position": 4
          },
          {
            "token": "e",
            "start_offset": 24,
            "end_offset": 25,
            "type": "<ALPHANUM>",
            "position": 5
          },
          {
            "token": "search",
            "start_offset": 26,
            "end_offset": 32,
            "type": "<ALPHANUM>",
            "position": 6
          },
          {
            "token": "engine",
            "start_offset": 33,
            "end_offset": 39,
            "type": "<ALPHANUM>",
            "position": 7
          }
        ]
      }
      ```
- `letter` - преобразует текст в токен, когда встречает небуквенный символ
    - `POST _analyze`
      ```json
      {
        "tokenizer": "letter",
        "text": "OpenSearch, is an awesome search-engine!"
      }
      ```
    - Response: ["OpenSearch", "is", "awesome", "search", "engine"]
- `lowercase` - преобразует все символы в нижний регистр и разбивает текст на токены, когда встречает любой небуквенный
  символ.
    - `POST _analyze`
      ```json
      {
        "tokenizer": "lowercase",
        "text": "OpenSearch, is an awesome search-engine!"
      }
      ```
    - Response: ["opensearch,", "is", "an", "awesome", "search", "engine"]
- `whitespace` - разбивает текст на токены только по пробелам, сохраняя регистр символов.
- `uax_url_email` - специализированный токенизатор, который разбивает текст на токены, учитывая URL-адреса и адреса
  электронной
  почты как отдельные токены.
    - `POST _analyze`
      ```json
      {
        "tokenizer": "uax_url_email",
        "text": "email me at sophia.julian@yopmail.com"
      }
      ```
    - Response:
      ```json
      {
        "tokens": [
          {
            "token": "email",
            "start_offset": 0,
            "end_offset": 5,
            "type": "<ALPHANUM>",
            "position": 0
          },
          {
            "token": "me",
            "start_offset": 6,
            "end_offset": 8,
            "type": "<ALPHANUM>",
            "position": 1
          },
          {
            "token": "at",
            "start_offset": 9,
            "end_offset": 11,
            "type": "<ALPHANUM>",
            "position": 2
          },
          {
            "token": "sophia.julian@yopmail.com",
            "start_offset": 12,
            "end_offset": 36,
            "type": "<EMAIL>",
            "position": 3
          }
        ]
      }
      ```
- `classic` - токенитор для английского языка, основанный на грамматике. Он распознает адреса электронной почты, имена
  сетевых хостов и др. и сохраняет их в виде одного токена. Он выделяет слова на основе знаков препинания, удаляя их.
    - `POST my_index3/_analyze`
      ```json
      {
        "tokenizer": "classic",
        "text": "Opensearch, is an awesome search-engine!"
      }
      ```
    - Response: ["Opensearch", "is", "an", "awesome", "search", "engine"]
- `partial word` - используются, если необходимо выполнить частичное сопоставление слов. Для этого слово разбивается на
  небольшие фрагменты.
- `ngram` - преобразует текст в непрерывную последовательность символов заданной длины. Эту последовательность можно
  использовать для частичного сопоставления слов, когда трудно подобрать точное совпадение. По умолчанию токенизатор
  ngram создает N-граммы, минимальная длина которых `1`, максимальная — `2`.
    - `POST _analyze`
    - ```json
      {
        "tokenizer": {
          "type": "ngram",
          "min_gram": 1,
          "max_gram": 2
        },
        "text": "OpenSearch"
      }
      ```
    - Response: ["O", "Op", "p", "pe", "e", "en", "n", "nS", "S", "Se", "e", "er", "r", "rc", "c", "ch"]
    - `min_gram` — по умолчанию минимальная длина символа равна 1 и изменяется с помощью этого параметра;
    - `max_gram` — по умолчанию максимальная длина символов равна 2 и изменяется с помощью этого параметра;
    - `token_chars` — в этом параметре можно указать классы символов, которые нужно включить в токен;
    - `custom_token_chars` — пользовательские символы, которые нужно включить в токен.
- `edge_ngram` - похож на ngram, но создает токены только из начала слова. Полезен для автозаполнения и поиска по
  префиксу. По умолчанию токенизатор edge_ngram создает N-граммы, минимальная длина которых `1`, максимальная — `2`.
    - `POST _analyze`
      ```json
      {
        "tokenizer": {
          "type": "edge_ngram",
          "min_gram": 1,
          "max_gram": 2
        },
        "text": "OpenSearch"
      }
      ```
    - Response: ["O", "Op"]
    - `min_gram` — по умолчанию минимальная длина символа равна 1 и изменяется с помощью этого параметра;
    - `max_gram` — по умолчанию максимальная длина символов равна 2 и изменяется с помощью этого параметра;
    - `token_chars` — в этом параметре можно указать классы символов, которые нужно включить в токен;
    - `custom_token_chars` — пользовательские символы, которые нужно включить в токен.
- `keyword` - не разбивает текст на токены, а сохраняет его как есть. Полезен для точного поиска.
    - `POST _analyze`
    - ```json
      {
        "tokenizer": "keyword",
        "text": "OpenSearch"
      }
      ```
    - Response: ["OpenSearch"]
- `pattern` - разбивает текст на токены с помощью регулярного выражения. Он работает так же, как и анализатор шаблонов.
  По умолчанию используется шаблон `\W+`, который разбивает текст на токены, когда видит символы, не являющиеся словами.

Фильтры токенов
=================================
Фильтры токенов принимают потоки токенов от токенизатора и изменяют их. Эти модификации используются для преобразования
текста в нижний регистр, удаления и добавления токенов и др.

- Фильтр `lowercase` преобразует полученный от токенизатора текст в нижний регистр.
- Фильтр `uppercase` преобразует полученный от токенизатора текст в верхний регистр.
- Фильтр `stop` удаляет стоп-слова из потоков токенов.
- Фильтр `reverse` меняет токены на обратные.
- Фильтр `elision` служит для удаления элизий. Элизии - это сокращенные формы слов, часто используемые в разговорной
  речи (например, "l'amour" становится "amour").
- Фильтр `truncate` сокращает токен до определенной длины. Длина по умолчанию устанавливается равной 10.
- Фильтр `unique` служит для индексирования уникальных токенов.
- Фильтр `duplicate` удаляет дубликаты токенов.

Фильтры символов
=================================
Фильтры символов используются до передачи потока символов токенизатору. Они обрабатывают поток символов, удаляя,
добавляя или изменяя их.

- Фильтр символов `html_strip` удаляет HTML-элементы из текста и заменить HTML-сущности, используя их декодированное
  значение.
    - `POST _analyze`
    - ```json
      {
        "tokenizer": "keyword",
        "char_filter": ["html_strip" ],
        "text": "<p>I&apos;m so <b>happy</b>!</p>"
      }
      ```
    - Response: [ \nI'm so happy!\n ]
    - `escaped_tags` — по умолчанию пустой массив. В этом параметре можно указать теги, которые не нужно удалять.
- Фильтр сопоставления `mapping char` использует ассоциативный массив с ключами и их значениями. В случае совпадения
  текста с ключом фильтр заменяет ключ его значением. Используя этот фильтр символов, можно преобразовать язык в любой
  другой язык. Например, с его помощью легко преобразовать индийские символы в арабские и арабские в латинские.
- Фильтр `pattern_replace` использует регулярные выражения для поиска и замены текста в потоке символов. В нем можно
  указать строку для замены. Фильтр символов `pattern_replace` поддерживает следующие параметры:
    - `pattern` — регулярное выражение в Java
    - `replacement` — строка замены, используемая для замены существующей строки при совпадении с регулярным выражением;
    - `flags`- флаги, разделенные пайпом, для регулярного выражения Java.

Нормализаторы
================================
Нормализаторы очень похожи на анализаторы, но генерируют только один токен вместо нескольких. Нормализаторы не содержат
токенизаторов и принимают некоторые фильтры символов и фильтры токенов.

Доступные для использования фильтры: `asciifolding`, `cjk_width`, `decimal_digit`, `elision`, `lowercase` и `uppercase`.

### Searching Query

Index: products

```json
{
  "mappings": {
    "properties": {
      "created": {
        "type": "date",
        "format": "yyyy/MM/dd HH:mm:ss||yyyy/MM/dd||epoch_millis",
        "print_format": "yyyy/MM/dd HH:mm:ss"
      },
      "description": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "in_stock": {
        "type": "long"
      },
      "is_active": {
        "type": "boolean"
      },
      "name": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "price": {
        "type": "long"
      },
      "sold": {
        "type": "long"
      },
      "tags": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      }
    }
  }
}
```

Поддерживается 2 типа синтаксиса запросов:

- Apache Lucene syntax:
    - `GET /products/_search?q=name:sauvingon AND tags:wine` - поиск документов, где поле `name` содержит `sauvingon` и
      поле `tags` содержит `wine`
    - `GET /products/_search?q=*:*` - возвращает все документы
- Query DSL syntax:

```json
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "sauvingon"
          }
        },
        {
          "match": {
            "tags": "wine"
          }
        }
      ]
    }
  }
}
```

Рассмотрим пример простого запроса, который возвращает все документы из индекса `products`:

```text
GET /products/_search
{
  "query": {
    "match_all": {}
  }
}
```

Компоненты запроса: `GET /products/_search`

- `GET` - HTTP метод
- `/products` - имя индекса (коллекции документов)
- `/_search` - endpoint для выполнения поиска
- `"query": { "match_all": {} }`
- `"match_all": {}` - специальный тип запроса, который соответствует всем документам в индексе
- Не имеет параметров и фильтров
- По умолчанию возвращает первые 10 документов (можно изменить параметром `size`)

```text
GET /products/_search
{
  "query": {
    "match_all": {}
  },
  "size": 20
}
```

- Сортировку по умолчанию (обычно по релевантности, но для match_all все имеют одинаковую релевантность)
    - Сортировка по релевантности - это порядок, при котором документы, наиболее соответствующие поисковому запросу,
      показываются первыми.
    - OpenSearch использует алгоритм `TF-IDF` (в более новых версиях - `BM25`) для расчета релевантности:
        - `TF (Term Frequency)` - Частота термина
            - Чем чаще поисковый термин встречается в документе, тем выше релевантность.
            - Пример: Поиск `"кошка"`
                - `Документ А`: "Кошка сидит на окне" → высокая релевантность
                - `Документ Б`: "Собака бежит" → низкая релевантность
        - `IDF (Inverse Document Frequency)` - Обратная частота документа
            - Чем реже термин встречается во всех документах индекса, тем он "ценнее".
            - Пример: Поиск `"гепард"`
                - `"Гепард"` встречается редко → высокая значимость
                - `"Животное"` встречается часто → низкая значимость
            - IDF (Inverse Document Frequency)
                - Чем чаще термин встречается во всех документах → тем он менее значим
                - Чем реже термин → тем он ценнее для поиска
                - Предположим статистику по вашему индексу:
                    - Термин: pasta - Количество документов 150 документов - Значимость (IDF) Высокая
                    - Термин: chicken - Количество документов 800 документов - Значимость (IDF) Низкая
        - `BM25` - это усовершенствованный алгоритм, который учитывает длину документа и нормализует частоту термина.
            - Можно использовать для получения всех документов в индексе, например, для анализа данных или отладки.

- Добавление сортировки по полю `in_stock` в порядке возрастания `(asc)`:

```text
GET /products/_search
{
  "query": {
    "match_all": {}
  },
  "size": 20,
  "sort": [
    {"in_stock": {"order": "asc"}} 
  ]
}
```

- `GET /products/_count` - это самый быстрый и эффективный способ, специально предназначенный для подсчета документов
  без загрузки самих документов.

```text
GET /products/_count
{
  "query": {
    "match_all": {}
  }
}
```

- Response:

```json
{
  "count": 1000,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  }
}
```

Для фильтрованного подсчета: Пример, который подсчитывает только те документы, где in_stock больше 20, но меньше 25

```text
GET /products/_count
{
  "query": {
    "range": {
      "price": {
        "gte": 20,
        "lte": 25
      }
    }
  }
}
```

Для пагинации результатов поиска используются параметры `from` и `size`.

- `from` - указывает, сколько документов пропустить (начальная точка).
    - `from` = (page - 1) * size, где `page` - номер страницы, а `size` - количество документов на странице.
- `size` - указывает, сколько документов вернуть (размер страницы).

```text
GET /products/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 10,
  "sort": [
    {"created": {"order": "desc"}}
  ]
}
```

- В этом примере:
    - `from: 0` - начать с первого документа (без пропуска).
    - `size: 10` - вернуть первые 10 документов.
    - `sort` - сортировка по полю `created` в порядке убывания (самые новые документы первыми).
- Для получения следующей страницы результатов, увеличьте `from` на значение `size`. Например, для второй страницы:

```text
GET /products/_search
{
  "query": {
    "match_all": {}
  },
  "from": 10,
  "size": 10,
  "sort": [
    {"created": {"order": "desc"}}
  ]
}
```

- Здесь `from: 10` пропускает первые 10 документов и возвращает следующие 10.
- from + size не может превышать 10,000 по умолчанию

Пагинация с фильтрацией

```text
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {"term": {"category": "electronics"}},
        {"range": {"price": {"gte": 100, "lte": 1000}}}
      ]
    }
  },
  "from": 20,
  "size": 15,
  "sort": [
    {"rating": {"order": "desc"}},
    {"_score": {"order": "desc"}}
  ]
}
```

- В этом примере:
    - `GET /products/_search`
        - `GET` - HTTP метод
        - `/products` - работаем с индексом "products"
        - `/_search` - выполняем поисковый запрос
    - Тип запроса (`query`) - `bool`
        - `bool` - позволяет комбинировать несколько условий
        - `must` - все условия ДОЛЖНЫ выполняться (логическое И - `AND`)
    - Условие #1 - категория
        - `term` - точное совпадение (без анализатора)
        - `{"term": {"category": "electronics"}}` - ищем документы, где поле `category` равно "electronics"
    - Условие #2 - диапазон цены
        - `range` - поиск по диапазону
        - `gte` - greater than or equal (≥ 100)
        - `lte` - less than or equal (≤ 1000)
        - Ищем товары от 100 до 1000 единиц
    - Пагинация
        - `from: 20` - пропускаем первые 20 результатов (начинаем с 21-го)
        - `size: 15` - возвращаем следующие 15 результатов
        - Номер страницы = (from / size) + 1 = (20 / 15) + 1 = 2.33 → третья страница
    - Сортировка - по рейтингу и релевантности]()
        - Первичная сортировка по `rating` в порядке убывания (самые высокие рейтинги первыми)
        - Вторичная сортировка по `_score` (оценка релевантности от OpenSearch) в порядке убывания
- Аналогичный SQL запрос:

```sql
SELECT *
FROM products
WHERE category = 'electronics'
  AND price BETWEEN 100 AND 1000
ORDER BY rating DESC, relevance DESC LIMIT 15
OFFSET 20;
```

Группы запросов в OpenSearch
=================================
OpenSearch поддерживает несколько типов запросов, которые можно использовать для различных целей поиска и фильтрации
данных. Вот основные группы запросов:

1. **Full-Text Queries (Запросы по полному тексту)**:
    - Используются для поиска текстовых данных с учетом анализа и релевантности.
    - Поиск в неструктурированных текстовых полях.
    - Применяются для текстовых полей (`text`), которые анализируются.
        - Примеры: контент статей, описания продуктов, отзывы пользователей.
    - Примеры:
        - `match` - ищет документы, содержащие указанный текст.
            - Если передаются несколько слов, OpenSearch анализирует их и ищет документы, содержащие все или некоторые
              из этих слов.
            - Можно указать оператор `operator`:
                - `or` (по умолчанию) - ищет документы, содержащие любое из слов.
                - `and` - ищет документы, содержащие все слова.
            - Можно использовать для дат, чисел и булевых полей, но обычно применяется для текстовых полей.
        - `multi_match` - ищет текст в нескольких полях.
        - `match_phrase` - ищет **точную** фразу (слова в указанном порядке). Полезен для поиска цитат или имен.
            - Inverted Index хранит позиции слов, что позволяет эффективно искать фразы.
        - `query_string` - позволяет использовать сложный синтаксис запросов.
        - `simple_query_string` - упрощенная версия `query_string`.

```text
GET /products/_search
{
  "query": {
    "match": {
      "description": "aliquam"
    }
  }
}
```

```text
GET /products/_search
{
  "query": {
    "match": {
      "name": "pasta CHICKEN"
    }
  }
}

После анализации текста "pasta CHICKEN" с помощью анализатора по умолчанию (обычно стандартного анализатора),
OpenSearch ищет документы, содержащие слова "pasta" и/или "chicken"


Если указать оператор "operator": "and", то OpenSearch ищет документы, содержащие оба слова "pasta" и "chicken".

GET /products/_search
{
  "query": {
    "match": {
      "name": {
        "query": "pasta CHICKEN",
        "operator": "AND"
      }
    }
  }
}
```

- Следующий пример ищет документы, где поле `description` содержит слово "aliquam".
- Перед выполнением запроса OpenSearch анализирует текст "aliquam" с помощью того же анализатора, который был
  использован при индексации поля `description`.
- Relevant score (оценка релевантности):
    - OpenSearch присваивает каждому документу оценку релевантности (score), которая указывает, насколько хорошо
      документ соответствует поисковому запросу.
    - Чем выше оценка, тем более релевантен документ.
    - Оценка релевантности рассчитывается на основе различных факторов, включая частоту термина (TF), обратную частоту
      документа (IDF) и длину документа.
    - В запросах full-text, таких как `match`, OpenSearch использует алгоритм BM25 для расчета оценки релевантности.

`multi_match` пример:

```text
Следующий пример ищет документы, где поле name ИЛИ поле tags содержит слово "vegetable".
GET /products/_search
{
  "query": {
    "multi_match": {
      "query": "vegetable",
      "fields": ["name", "tags"]
    }
  }
}

name^2 - повышает вес поля name в два раза по сравнению с полем tags.
GET /products/_search
{
  "query": {
    "multi_match": {
      "query": "vegetable",
      "fields": ["name^2", "tags"]
    }
  }
}

По умолчанию лучшая релевантность для определенного поля используется для всего оценки документа.
tie_breaker - используется для смешанных совпадений (documents matching both fields).
Если документ соответствует обоим полям, tie_breaker помогает определить, насколько сильно это совпадение
влияет на общую оценку релевантности документа.
Например, если документ соответствует полю name и полю tags, tie_breaker позволяет учесть оба совпадения,
но с определенным весом для второго поля.
GET /products/_search
{
  "query": {
    "multi_match": {
      "query": "vegetable",
      "fields": ["name", "tags"],
      "tie_breaker": 0.3
    }
  }
}
```

Пример `match_phrase`:

```text
-- Juice - Mango
GET /products/_search
{
  "query": {
    "match_phrase": {
      "name": "juice mango"
    }
  }
}


-- Call people. Or text them. Or **browse the Internet**. Or do other things.
GET /products/_search
{
  "query": {
    "match_phrase": {
      "description": "browse the internet"
    }
  }
}
```

2. **Term-Level Queries (Запросы на уровне терминов)**:
    - Используются для точного совпадения значений (`exact values` = `filtering`) в полях.
    - `Term-level` запросы не анализируют текст, а ищут точные совпадения. Это значит, что значение поиска используется
      в том же виде, как и для поиска по инвертированному индексу.
    - Case-sensitive (чувствительны к регистру)
    - Применяются для числовых, датированных, булевых полей и ключевых слов (`keyword`).
    - Не применяются для текстовых полей (`text`), так как они анализируются.
    - Примеры:

- `term` - ищет точное совпадение одного термина.

```text
GET /products/_search
{
  "query": {
    "term": {
      "tags.keyword": {
        "value": "Vegetable"
      }
    }
  }
}
```

```text
GET /products/_search
{
  "query": {
    "term": {
      "tags.keyword": {
        "value": "vegetable",
        "case_insensitive": true
      }
    }
  }
}
```

- `terms` - ищет совпадение любого из нескольких терминов.
    - tags.keyword contains "Soup" AND/OR "Meat"

```text
GET /products/_search
{
  "query": {
    "terms": {
      "tags.keyword": [
        { "value": "Soup" },
        { "value": "Meat" }
      ]
    }
  }
}

или 

GET /products/_search
{
  "query": {
    "terms": {
      "tags.keyword": ["Soup", "Meat"]
    }
  }
}

```

- `ids` - ищет документы по их уникальным идентификаторам.
    - В этом примере ищутся документы с ID 100, 200 и 300.
    - Если указанный ID не существует, он просто игнорируется.

```text
GET /products/_search
{
  "query": {
    "ids": {
      "values": ["100", "200", "300"]
    }
  }
}
```

- `range` - ищет значения в заданном диапазоне.
    - Примеры операторов:
        - `gt` - больше чем (>)
        - `gte` - больше или равно (≥)
        - `lt` - меньше чем (<)
        - `lte` - меньше или равно (≤)
        - `format` - формат даты (если поле датированное)
    - Пример ниже ищет документы, где значение поля `in_stock` больше 20 и меньше 30.
    - SQL эквивалент: `SELECT * FROM products WHERE in_stock > 20 AND in_stock < 30;`
    - Даты в OpenSearch по умолчанию в формате ISO 8601 (например, `2023-10-05T14:48:00Z`), другими словами в формате
      `yyyy-MM-dd'T'HH:mm:ssZ`, что эквивалентно UTC времени.
    - `time_zone` - позволяет указать временную зону для корректного сравнения дат. (UTC offset)

```text
GET /products/_search
{
  "query": {
    "range": {
      "in_stock": {
        "gt": 20,
        "lt": 30,
      }
    }
  }
}

или пример с датами:

GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2000/11/17 00:00:00",
        "lte": "2002/04/26 23:59:59"
      }
    }
  }
}

GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gt": "2000/11/17 00:00:00",
        "lt": "2002/04/26 23:59:59",
        "format": "yyyy/MM/dd HH:mm:ss"
      }
    }
  }
}

GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gt": "2000/11/17 00:00:00",
        "lt": "2002/04/26 23:59:59",
        "time_zone": "+01:00"
      }
    }
  }
}

```

- `exists` - проверяет наличие значения в поле.
    - SQL эквивалент: `SELECT * FROM products WHERE tags IS NOT NULL;`
    - Пример ниже ищет документы, где поле `tags.keyword` содержит значение (не пустое и не null).
    - Следует запомнить как сохраняются в индексе пустые поля:
        - Пустая строка `""` - сохраняется как есть
        - `null` - не сохраняется в индексе
        - Отсутствующее поле - не сохраняется в индексе
        - Пустой массив `[]` - не сохраняется в индексе
        - `null_value` - сохраняется в индексе как заданное значение

```text
-- SELECT * FROM products WHERE tags IS NOT NULL
GET /products/_search
{
  "query": {
    "exists": {
      "field": "tags.keyword"
    }
  }
}

-- SELECT * FROM products WHERE tags IS NULL;
GET /products/_search
{
  "query": {
    "bool": {
      "must_not": [
        {
          "exists": {
            "field": "tags.keyword"
          }
        }
      ]
    }
  }
}
```

- `prefix` - ищет документы, где поле начинается с указанного префикcа.
    - Следующий пример ищет документы, где поле `name.keyword` начинается с префикса "Past".
    - Если найдет term, начинающиеся с  "Pastry" или "Pastel", то вернет эти документы, так как они начинаются с "Past".

```text
GET /products/_search
{
  "query": {
    "prefix": {
      "name.keyword": {
        "value": "Past"
      }
    }
  }
}
```

- `wildcard` - ищет документы, соответствующие шаблону с использованием подстановочных знаков.
    - `*` - соответствует нулю или более символам (необходимо избегать использования в начале шаблона, так как это
      приводит к медленному поиску)
    - `?` - соответствует одному символу

```text
GET /products/_search
{
  "query": {
    "wildcard": {
      "tags.keyword": {
        "value": "Past?"
      }
    }
  }
}

GET /products/_search
{
  "query": {
    "wildcard": {
      "name.keyword": {
        "value": "Bee*"
      }
    }
  }
}

```

- `regexp` - ищет документы, соответствующие регулярному выражению.
    - Case-sensitive (чувствительны к регистру)
    - Можно отключить чувствительность к регистру с помощью параметра `case_insensitive: true`, но это может
      привести к медленному поиску.
    - Следующий пример ищет документы, где поле `tags.keyword` соответствует регулярному выражению `Bee(f|r)+`.
    - Нужно использовать Apache Lucene регулярные выражения, которые немного отличаются от стандартных регулярных
      выражений:
        - . - соответствует любому символу
        - ? - соответствует нулю или одному вхождению предыдущего символа
        -
            *
                - соответствует нулю или более вхождениям предыдущего символа
        -
            +
                - соответствует одному или более вхождениям предыдущего символа
        - {n} - соответствует ровно n вхождениям предыдущего символа
        - {n,} - соответствует n или более вхождениям предыдущего символа
        - {n,m} - соответствует от n до m вхождениям предыдущего символа
        - [] - соответствует любому символу внутри скобок
    - Регулярное выражение `Bee(f|r)+` соответствует строкам, которые начинаются с "Bee", за которым следует один или
      более символов 'f' или 'r'. Например, оно соответствует "Beef", "Beer".

```text
GET /products/_search
{
  "query": {
    "regexp": {
      "tags.keyword": {
        "value": "Bee(f|r)+"
      }
    }
  }
}

GET /products/_search
{
  "query": {
    "regexp": {
      "tags.keyword": {
        "value": "Bee[a-z]+"
      }
    }
  }
}
```

- `fuzzy` - ищет документы с учетом опечаток и похожих слов.
- `type` - ищет документы определенного типа (устаревший, используется в старых версиях).

3. **Compound Queries (Составные запросы)**:
    - Позволяют комбинировать несколько запросов с помощью логических операторов.
    - Примеры:
        - `bool` - комбинирует несколько запросов с использованием (для логических операций):
            - `must` - для И (AND)
            - `should` - для ИЛИ (OR)
            - `must_not` - для НЕ (NOT)
            - `filter` - для фильтрации
        - `constant_score` - оборачивает запрос и присваивает всем результатам одинаковый балл.
        - `dis_max` - выбирает наилучший результат из нескольких запросов.
        - `function_score` - позволяет изменять баллы релевантности с помощью функций.
4. **Joining Queries (Запросы соединения)**:
    - Используются для поиска связанных документов.
    - Примеры:
        - `nested` - ищет документы внутри вложенных объектов.
        - `has_child` - ищет документы, у которых есть дочерние документы, соответствующие запросу.
        - `has_parent` - ищет документы, у которых есть родительские документы, соответствующие запросу.
5. **Geo Queries (Геозапросы)**:
    - Используются для поиска документов на основе географических данных.
    - Примеры:
        - `geo_distance` - ищет документы в пределах заданного радиуса от точки.
        - `geo_bounding_box` - ищет документы внутри заданного прямоугольника.
        - `geo_polygon` - ищет документы внутри заданного многоугольника.
6. **Specialized Queries (Специализированные запросы)**:
    - Запросы для специфичных случаев использования.
    - Примеры:
        - `more_like_this` - ищет документы, похожие на указанный текст или документ.
        - `percolate` - позволяет сохранять запросы и проверять новые документы на соответствие этим запросам.
        - `script` - позволяет использовать скрипты для создания пользовательских запросов.
        - `span` - позволяет создавать сложные текстовые запросы с использованием позиций токенов.
        - `template` - позволяет использовать шаблоны для создания запросов.
        - `wrapper` - позволяет обернуть запрос в другой формат (например, JSON).
7. **Match All Query (Запрос на совпадение всех документов)**:
    - `match_all` - возвращает все документы в индексе без фильтрации.
8. **More Like This Query (Запрос "Похожее на это")**:
    - `more_like_this` - находит документы, похожие на указанный текст или документ.
9. **Percolate Query (Запрос перколяции)**:
    - `percolate` - позволяет сохранять запросы и проверять новые документы на соответствие этим запросам.
10. **Script Query (Скриптовый запрос)**:
    - `script` - позволяет использовать скрипты для создания пользовательских запросов._