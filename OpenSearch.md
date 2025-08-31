Роль анализаторов в OpenSearch
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