In most of the cases, a single Unicode character represents a single **grapheme**. For instance **U+0066 LATIN SMALL LETTER F** is an English writing f.

There are cases when a grapheme contains a sequence of characters.
**å** is an atomic grapheme in the Danish writing system. 

* It is displayed using **U+0061 LATIN SMALL LETTER A** (rendered as a) 
* Combined with a special character **U+030A COMBINING RING ABOVE** (rendered as ◌̊).

U+030A modifies the precedent character and is named **combining mark**:


```js
> '\u0061'
'a'
> '\u0061\u030A'
'å'
```

See the article [Combining marks](https://dmitripavlutin.com/what-every-javascript-developer-should-know-about-unicode/#25-combining-marks)