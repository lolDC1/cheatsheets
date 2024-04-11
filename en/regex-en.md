### Links
- **[README](../README.md)**
- **Ansible** ([English](ansible-en.md)/[Russian](../ru/ansible-ru.md)): Automation of configuration and system management.
- **Elasticsearch** ([English](elastic-en.md)/[Russian](../ru/elastic-ru.md)): Search and data analysis.
- **Kubernetes** ([English](kube-en.md)/[Russian](../ru/kube-ru.md)): Container orchestration and cluster management.
- **Regular Expressions** ([**English**](regex-en.md)/[Russian](../ru/regex-ru.md)): A powerful tool for working with text.
- **Ubuntu** ([English](ubuntu-en.md)/[Russian](../ru/ubuntu-ru.md)): Basic settings and recommendations for using Ubuntu.
- **HashiCorp Vault** ([English](vault-en.md)/[Russian](../ru/vault-ru.md)): Management of secrets and access.

# Types of regular expressions

Implementations of regular expressions in different environments, for example, in programming languages like Java, Perl and Python, and in Linux tools like sed, awk and grep, have certain features. These features depend on so-called regular expression engines, which interpret patterns.

## Linux has two regular expression engines:

- An engine that supports the POSIX Basic Regular Expression (BRE) standard.
    
- An engine that supports the POSIX Extended Regular Expression (ERE) standard.

Most Linux utilities conform to at least the POSIX BRE standard, but some utilities (including sed) understand only a subset of the BRE standard. One of the reasons for this limitation is the desire to make such utilities as fast as possible in text processing.

The POSIX ERE standard is often implemented in programming languages. It allows you to use a large number of tools when developing regular expressions. For example, these could be special sequences of characters for frequently used patterns, such as searching for individual words or sets of numbers in text. Awk supports the ERE standard.

There are many ways to develop regular expressions, depending both on the opinion of the programmer and on the features of the engine for which they are created. It's not easy to write universal regular expressions that any engine can understand. Therefore, we will focus on the most commonly used regular expressions and look at the features of their implementation for sed and awk.

### POSIX Basic Regular Expression (BRE):
The simplest template, supported everywhere.

- *\\* - shielding symbol
    ```bash
    echo "3 / 2" | awk '/\//{print $0}'
    ```
- **\!** - logical NOT
    ```bash
    printf "welcome to likegeeks\nwebsite" | awk '!/geeks$/{print $0}'
    ```
- **\.** - character sequence
    ```bash
    awk '/.st/{print $0}'
    ```
- **\[]** - character class (list)
    ```bash
    printf "welcome to likegeeks\nwebsite" | awk '/.t[oe]/{print $0}'
    ```
    ```bash
    printf "Welcome to likegeeks\nwebsite" | awk '/^[Ww]/{print $0}'
    ```
- **\*** - the appearance of a character in a line any number of times, even if it is not there, is used when working with words in which a typo is often made
    ```bash
    echo "I like green color" | awk '/colou*r/{print $0}'
    ```
    ```bash
    echo "I like green colour" | awk '/colou*r/{print $0}'
    ```

#### Character ranges (0123456789 | abcdefghijklmnopqrstuvwxyz):

- **[a-z]** - any symbol in range of a-z
    ```bash
    printf "Welcome to likegeeks\n1s't website\n!" | awk '/^[a-zA-Z]/{print $0}'
    ```
- **[0-9]**
    ```bash
    printf "Welcome to likegeeks\n1s't website\n!" | awk '/^[1-9]/{print $0}'
    ```
#### Anchor symbols:
- **\^** - beggining of the line
    ```bash
    printf "welcome to likegeeks\nwebsite" | awk '/^web/{print $0}'
    ```
- **\$** - end on the line 
    ```bash
    printf "welcome to likegeeks\nwebsite" | awk '/geeks$/{print $0}'
    ```
#### Спецсимволы: 

- **[[:alpha:]]** — any alphabetical symbol [a-zA-Z]
- **[[:alnum:]]** — any alphabetical-numerical symbol [0-9a-zA-Z]
- **[[:blank:]]** — space and Tab symbols
- **[[:digit:]]** — any numerical symbol [0-9]
- **[[:upper:]]** — any alphabetical uppercase symbol [A-Z]
- **[[:lower:]]** — any alphabetical lowercase symbol [a-z]
- **[[:print:]]** — any printed symbol
- **[[:punct:]]** — punctuation marks
- **[[:space:]]** — space symbols: space, tab, symbols NL, FF, VT, CR

#### Other mentions:
- Erase spaces at the beggining of the line:
    ```bash
    sed 's/^ *//'
    ```
- Print all NOT blank lines:
    ```bash
    awk '!/^$/{print $0}'
    ```
- Combination of **\*** with **\.**:
    - Print everything between **I** and **colour**
        ```bash
        echo "I like green colour" | awk '/I.*colour/{print $0}'
        ```

### Regular expressions POSIX Extended Regular Expression (ERE)
The standard is more complex and is not used everywhere

- **\?** - repeating a character 0 or 1 time
    ```bash
    echo "tet" | awk '/tes?t/{print $0}'
    ```
    ```bash
    echo "test" | awk '/tes?t/{print $0}'
    ```
    ```bash
    echo "tesst" | awk '/tes?t/{print $0}'
    ```
- **\+** - repeating a character 1 and more times
    ```bash
    echo "test" | awk '/te+st/{print $0}'
    ```
    ```bash
    echo "teest" | awk '/te+st/{print $0}'
    ```
    ```bash
    echo "tst" | awk '/te+st/{print $0}'
    ```
- **{n}** - repeating a character exact times
    ```bash
    echo "tst" | awk '/te{1}st/{print $0}'
    ```
    ```bash
    echo "test" | awk '/te{1}st/{print $0}'
    ```
- **{n,m}** - repeating a character minimum n times and maximum m times
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
- **|** - logical OR
    ```bash
    echo "This is a test" | awk '/test|exam/{print $0}'
    ```
    ```bash
    echo "This is an exam" | awk '/test|exam/{print $0}'
    ```
    ```bash
    echo "This is something else" | awk '/test|exam/{print $0}'
    ```
- **()** - grouping fragments of different regular expressions
    ```bash
    echo "Like" | awk '/Like(Geeks)?/{print $0}'
    ```
    ```bash
    echo "LikeGeeks" | awk '/Like(Geeks)?/{print $0}'
    ```
## Examples:

- Validation of an email adress:
    - username@hostname.com
    ```bash
    ^([a-zA-Z0-9_\-\.\+]+)@([a-zA-Z0-9_\-\.]+)\.([a-zA-Z]{2,5})$
    ```
- Inserting at the beggining of line: 
    - Ctrl+H
    - Find: `(^)`
    - Replase: `string $1`
- Inserting at the end of line: 
    - Ctrl+H
    - Find: `($)` const const 
    - Replase: `$1 string`