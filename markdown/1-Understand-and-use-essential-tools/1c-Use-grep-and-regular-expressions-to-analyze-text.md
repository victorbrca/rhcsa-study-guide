1.c Use grep and regular expressions to analyze text
===

|  Operator 	 |  Description                                                              |
| --------------- | --------------------------------------------------------------------------- |
| `^`             | Match expr at the start of line                                             |
| `$`             | Match expr at the end of line                                               |
| `\`             | Turn off special meaning                                                    |
| `[ ]`           | Match any of the enclosed chars                                             |
| `[^ ]`          | Match any char except the enclosed                                          |
| `.`             | Match a single char                                                         |
| `?`             | Match zero or one chars                                                     |
| `+`             | Match one or more chars                                                     |
| `*`             | Match zero or more chars                                                    |
| `\{x,y\}`       | Match x to y occurrences of preceding expr                                  |
| `\{x\}`         | Match x occurrences of preceding expr                                       |
| `\{x,\}`        | Match x or more occurrences of preceding expr                               |
| `[:class:]`     | Matches all chars in class (alnum, alpha, digit, space, upper, lower, etc.) |

#### Character Classes

`[:alnum:], [:alpha:], [:cntrl:], [:digit:], [:graph:], [:lower:], [:print:], [:punct:], [:space:], [:upper:], and [:xdigit:]`

**📌 EXAM TIP**
+ Use `man 7 regex` to get information on regex
+ You can find the character classes in grep's man page
+ Almost all character classes are defined with a 5 character word

---
[⬅️ Back](1-Understand-and-use-essential-tools.md)
