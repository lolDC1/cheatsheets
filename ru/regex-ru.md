### Table of Contents
- **[README](../README.md)**
- **Ansible** (English/[Russian](ansible-ru.md)): Automation of configuration and system management.
- **Elasticsearch** (English/[Russian](elastic-ru.md)): Search and data analysis.
- **Kubernetes** (English/[Russian](kubectl-ru.md)): Container orchestration and cluster management.
- **Regular Expressions** (English/[Russian](regex-ru.md)): A powerful tool for working with text.
- **Ubuntu** (English/[Russian](ubuntu-ru.md)): Basic settings and recommendations for using Ubuntu.
- **HashiCorp Vault** ([English](../en/vault-en.md)/Russian): Management of secrets and access.

# Типы регулярных выражений

Реализации регулярных выражений в различных средах, например, в языках программирования вроде Java, Perl и Python, в инструментах Linux вроде sed, awk и grep, имеют определённые особенности. Эти особенности зависят от так называемых движков обработки регулярных выражений, которые занимаются интерпретацией шаблонов.

## В Linux имеется два движка регулярных выражений:

- Движок, поддерживающий стандарт POSIX Basic Regular Expression (BRE).
    
- Движок, поддерживающий стандарт POSIX Extended Regular Expression (ERE).

Большинство утилит Linux соответствуют, как минимум, стандарту POSIX BRE, но некоторые утилиты (в их числе — sed) понимают лишь некое подмножество стандарта BRE. Одна из причин такого ограничения — стремление сделать такие утилиты как можно более быстрыми в деле обработки текстов.

Стандарт POSIX ERE часто реализуют в языках программирования. Он позволяет пользоваться большим количеством средств при разработке регулярных выражений. Например, это могут быть специальные последовательности символов для часто используемых шаблонов, вроде поиска в тексте отдельных слов или наборов цифр. Awk поддерживает стандарт ERE.

Существует много способов разработки регулярных выражений, зависящих и от мнения программиста, и от особенностей движка, под который их создают. Непросто писать универсальные регулярные выражения, которые сможет понять любой движок. Поэтому мы сосредоточимся на наиболее часто используемых регулярных выражениях и рассмотрим особенности их реализации для sed и awk.

### Регулярные выражения POSIX Basic Regular Expression (BRE):
Самый простой шаблон, поддерживается везде.

- *\\* - экранирующий символ
    ```bash
    echo "3 / 2" | awk '/\//{print $0}'
    ```
- **\!** - логическое НЕ
    ```bash
    printf "welcome to likegeeks\nwebsite" | awk '!/geeks$/{print $0}'
    ```
- **\.** - поиск строк по последовательности символов
    ```bash
    awk '/.st/{print $0}'
    ```
- **\[]** - класс символов (список)
    ```bash
    printf "welcome to likegeeks\nwebsite" | awk '/.t[oe]/{print $0}'
    ```
    ```bash
    printf "Welcome to likegeeks\nwebsite" | awk '/^[Ww]/{print $0}'
    ```
- **\*** - появление символа в строке любое колличество раз, даже если его там нет, исплользуют в работе со словами в которых часто делают опечатку
    ```bash
    echo "I like green color" | awk '/colou*r/{print $0}'
    ```
    ```bash
    echo "I like green colour" | awk '/colou*r/{print $0}'
    ```

#### Диапазоны символов (0123456789 | abcdefghijklmnopqrstuvwxyz):

- **[a-z]** - любой символ в диапазоне a-z
    ```bash
    printf "Welcome to likegeeks\n1s't website\n!" | awk '/^[a-zA-Z]/{print $0}'
    ```
- **[0-9]**
    ```bash
    printf "Welcome to likegeeks\n1s't website\n!" | awk '/^[1-9]/{print $0}'
    ```
#### Якорные символы:
- **\^** - привязывает к началу строки
    ```bash
    printf "welcome to likegeeks\nwebsite" | awk '/^web/{print $0}'
    ```
- **\$** - привязывает к концу строки 
    ```bash
    printf "welcome to likegeeks\nwebsite" | awk '/geeks$/{print $0}'
    ```
#### Спецсимволы: 

- **[[:alpha:]]** — любое алфавитное значение [a-zA-Z]
- **[[:alnum:]]** — любое алфавитное и цифровое значение [0-9a-zA-Z]
- **[[:blank:]]** — соответствует пробелу и знаку табуляции
- **[[:digit:]]** — любое цифровое значение [0-9]
- **[[:upper:]]** — любое алфавитное значение в верхнем регистре [A-Z]
- **[[:lower:]]** — любое алфавитное значение в нижнем регистре [a-z]
- **[[:print:]]** — любой печатаемый символ
- **[[:punct:]]** — знаки препинания
- **[[:space:]]** — пробельные символы: пробел, знак табуляции, символы NL, FF, VT, CR

#### Остальные примеры:
- Убрать ненужные пробелы в начале:
    ```bash
    sed 's/^ *//'
    ```
- Пример вывода не пустых строк:
    ```bash
    awk '!/^$/{print $0}'
    ```
- Комбинирование **\*** с **\.**:
    - Вывод любых строк с любыми словами между **I** и **colour**
        ```bash
        echo "I like green colour" | awk '/I.*colour/{print $0}'
        ```

### Регулярные выражения POSIX Extended Regular Expression (ERE)
Стандарт сложнее, используется не везде

- **\?** - провторение символа 0 либо 1 раз
    ```bash
    echo "tet" | awk '/tes?t/{print $0}'
    ```
    ```bash
    echo "test" | awk '/tes?t/{print $0}'
    ```
    ```bash
    echo "tesst" | awk '/tes?t/{print $0}'
    ```
- **\+** - провторение символа 1 и более раз
    ```bash
    echo "test" | awk '/te+st/{print $0}'
    ```
    ```bash
    echo "teest" | awk '/te+st/{print $0}'
    ```
    ```bash
    echo "tst" | awk '/te+st/{print $0}'
    ```
- **{n}** - повторение точное число раз
    ```bash
    echo "tst" | awk '/te{1}st/{print $0}'
    ```
    ```bash
    echo "test" | awk '/te{1}st/{print $0}'
    ```
- **{n,m}** - повторение минимум n раз, максимум m раз
    ```bash
    echo "tst" | awk '/te{1,2}st/{print $0}'
    ```
    ```bash
    echo "test" | awk '/te{1,2}st/{print $0}'
    ```
    ```bash
    echo "teest" | awk '/te{1,2}st/{print $0}'
    ```
    ```bash
    echo "teeest" | awk '/te{1,2}st/{print $0}'
    ```
- **|** - логическое ИЛИ 
    ```bash
    echo "This is a test" | awk '/test|exam/{print $0}'
    ```
    ```bash
    echo "This is an exam" | awk '/test|exam/{print $0}'
    ```
    ```bash
    echo "This is something else" | awk '/test|exam/{print $0}'
    ```
- **()** - групирование фрагментов регулярных выражений
    ```bash
        echo "Like" | awk '/Like(Geeks)?/{print $0}'
    ```
    ```bash
        echo "LikeGeeks" | awk '/Like(Geeks)?/{print $0}'
    ```
## Примеры:

- Проверка email адреса:
    - username@hostname.com
    ```bash
    ^([a-zA-Z0-9_\-\.\+]+)@([a-zA-Z0-9_\-\.]+)\.([a-zA-Z]{2,5})$
    ```
- Ввод значения в начало строки: 
    - Ctrl+H
    - Find: `(^)`
    - Replase: `string $1`
- Ввод значения в конец строки: 
    - Ctrl+H
    - Find: `($)` const const 
    - Replase: `$1 string`