oK Manual
=========

oK aims to be an implementation of [k5](http://kparc.com), the still-evolving bleeding edge version of the K programming language. This manual will serve as a short but more comprehensive reference than the official [k5 reference card](http://kparc.com/k.txt). Since both oK and k5 are unfinished, this document will attempt to describe oK's *current behavior* and will be updated as semantics are brought in line with k5.

K may look daunting at first, but the syntax of the language is very simple and regular. Programs consist of a series of expressions which are made up of *nouns*, *verbs* and *adverbs*. Nouns can be simple *atomic* types like *numbers*, *characters* or *symbols* or they can be the compound datatypes *lists*, *dictionaries* or *functions*. In general, `()` are for grouping subexpressions or forming lists, `[]` are used for creating dictionaries, indexing into a list or applying arguments to a function, `{}` are used for delimiting functions and newlines always behave identically to semicolons (`;`). The colon (`:`) has several possible meanings in different contexts, but most frequently behaves as an assignment operator binding the result of a right-hand expression to a name:
	
	quux: 23+9

Nouns
-----

- Numbers are composed of digits and can optionally have a leading negative sign or decimal part.

- Characters are enclosed in double quotes (") and can make use of the escape sequences `\n`, `\t`, `\"` or `\\` to produce a newline, tab, double quote or backslash character, respectively. If more than one unescaped character is enclosed in quotes, the noun is a list of characters (see below), also known as a *string*.

- Symbols are prefixed with a backtick (\`) and can consist of zero or more alphabetic characters. Symbols are mainly useful as handles to variable names or keys in dictionaries.

- Lists are a sequence of nouns. If several numbers have spaces separating them, they form a list:

		2 78 3

	Symbols work the same way, but the spaces are optional:

		`bread `apples `soap
		`bread`apples`soap

	Boolean lists are suffixed with `b` and require no spaces between elements:

		  10010b
		1 0 0 1 0

	Mixed-type lists are enclosed in `()` and each element should be separated by a semicolon:

		(1 2 3;"food";`b)

- Dictionaries are enclosed in `[]` and consist of key-value pairs separated by semicolons (;) in which keys are backtick-less symbols and values are any expression:

		[a:34;b:"feed me"]

- Functions are enclosed in `{}` and consist of one or more expressions separated by semicolons (;). When evaluated, functions execute these expressions left to right and return the value of their final expression. The special names `x`, `y` and `z` are implicit arguments to a function. If all three are present, the function takes 3 arguments. If only `x` and `y` are present, the function takes 2 arguments. If only `x` is present, the function takes 1 argument. Functions are applied to arguments with `[]` or simply by juxtaposing the function and the argument in the case of single-argument functions:

		mean:{(+/x)%#x}
		mean[4 7 18]
		{x*x} 5

Verbs
-----

K has 19 primitive verbs which are each represented as a single character: `+-*%!&|<>=~,^#_$?@.` Each has different behavior when used as a *monad* (with one argument) or as a *dyad* (with two arguments). Verbs behave as dyads if there is a noun immediately to their left, and otherwise they behave as a monad. Sometimes it is necessary to disambiguate, so using a colon (`:`) as a suffix will force a verb to behave as a monad. Verbs have uniform right-to-left evaluation unless explicitly grouped with `()`. If applied to nouns without a verb argument, the adverbs `/`, `\` and `'` can behave as verbs.

Some verbs are *atomic*. A *fully atomic* verb penetrates to the atoms of arbitrary nested lists as arguments:

	  1 3 5+(3; 1 2; 5)
	(4
	 4 5
	 10)

A *right atomic* verb only penetrates to the atoms of the right argument, leaving the left argument intact. Similarly, a *left atomic* verb only penetrates to the atoms of the left argument.

See below for a complete reference to verb behaviors.

Adverbs
-------

K has 6 primitive adverbs, some of which consist of two characters: `'`, `/`,`\`, `/:`, `\:` and `':`. Adverbs take a verb as a left argument and apply it to a right or right and left noun argument in some special way. Some adverbs have several different behaviors based on the types of their arguments.

For the purposes of the adverb and verb references, the following conventions will be used:

- `a`: any atom
- `n`: any number
- `l`: any list
- `m`: monadic verb or function
- `d`: dyadic verb or function

For a dyadic verb, the names `x` and `y` refer to the left and right arguments, respectively. For a monadic verb, the name `x` refers to the right argument.

Verb Reference
--------------
As a general note, verbs which operate on numbers will coerce characters to their ascii values when applied.

[+](#flip) [-](#negate) [*](#first) [%](#sqrt) [!](#int) [&](#where) [|](#reverse) [<](#asc) [>](#desc) [=](#group) [~](#not) [,](#enlist) [&#94;](#null) [\#](#count) [_](#floor) [$](#string) [?](#distinct) [@](#type) [.](#val) ['](#bin) [/](#join) [\ ](#split)

<table border=1>
<tr>
	<th>Monadic</th>
	<th>Dyadic</th>
</tr>
<tr>
	<td>
		<a name="flip"/>
		<tt>+l</tt> is <b>flip</b>. Takes the transpose of matrices.<br>
		Atoms are spread to match the dimension of other columns.
<pre><code>  +(1 2;3 4)
(1 3
 2 4)
  +(5;3 4)
(5 3
 5 4)</code></pre>
	</td>
	<td>
		<tt>n+n</tt> is <b>plus</b>. Fully atomic.
<pre><code>  3+4 5 6
7 8 9
  "ab"+5
102 103</pre></code>
	</td>
</tr>
<tr>
	<td>
		<a name="negate"/>
		<tt>-n</tt> is <b>negate</b>. Flips the sign of numbers. Right atomic.
<pre><code>  -(4 -10 8)
-4 10 -8
  -"b"
-98</code></pre>
	</td>
	<td>
		<tt>n-n</tt> is <b>minus</b>. Fully atomic.
<pre><code>  23 9-5 10
18 -1</code></pre>
	</td>
</tr>
<tr>
	<td>
		<a name="first"/>
		<tt>*l</tt> is <b>first</b>. Extracts the first element of a list.
<pre><code>  *5 19 8
5</code></pre>
		<tt>*d</tt> extracts the first <i>value</i> from a dictionary.
<pre><code>  *[a:23;b:34]
23</code></pre>
	</td>
	<td>
		<tt>n*n</tt> is <b>times</b>. Fully atomic.
<pre><code>  5 0 -3*2 2 1
10 0 -3</code></pre>
	</td>
</tr>
<tr>
	<td>
		<a name="sqrt"/>
		<tt>%n</tt> is <b>square root</b>. Right atomic.
<pre><code>  %25 7 100
5 2.6458 10</code></pre>
	</td>
	<td>
		<tt>n%n</tt> is <b>divide</b>. Fully atomic.
<pre><code>  2 20 9%.5 10 2
4 2 4.5</code></pre>
	</td>
</tr>
<tr>
	<td>
		<a name="int"/>
		<tt>!n</tt> is <b>int</b>. Generate a range from 0 up to but excluding N.
<pre><code>  !5
0 1 2 3 4</code></pre>
		<tt>!l</tt> is <b>odometer</b>. Generate ranged permutations.
<pre><code>  !2 3
(0 0 0 1 1 1
 0 1 2 0 1 2)</code></pre>
		<tt>!d</tt> is <b>keys</b>. List of keys from dictionary.
<pre><code>  ![a:3;b:4]
`a`b</code></pre>
	</td>
	<td>
		<tt>n!n</tt> is <b>mod</b>. Take y modulo x. Right atomic.
<pre><code>  3!34 2 8
1 2 2</code></pre>
		<tt>l!l</tt> is <b>map</b>. Make dictionary from x keys and y value(s). Right atomic.
<pre><code>  `a`b!3 4
[a:3;b:4]
  4 5!6 7
4 5!6 7</code></pre>
	</td>
</tr>
<tr>
	<td>
		<a name="where"/>
		<tt>&l</tt> is <b>where</b>. Make N copies of sequential indices of elements.<br>
		For a boolean list this gathers indices of nonzero elements.
<pre><code>  &2 3 1
0 0 1 1 1 2
  &1 0 0 1 0 1
0 3 5</code></pre>
	</td>
	<td>
		<tt>n&n</tt> is <b>min</b>. Fully atomic. For booleans, effectively logical <b>and</b>.
<pre><code>  3 5 7&0 6 9
0 5 7</code></pre>
	</td>
</tr>
<tr>
	<td>
		<a name="reverse"/>
		<tt>|l</tt> is <b>reverse</b>.
<pre><code>  |"ABDEF"
"FEDBA"</code></pre>
		<tt>|d</tt> reverses both keys and values in a dictionary.
<pre><code>  |[a:3;b:4]
[b:4;a:3]</code></pre>
	</td>
	<td>
		<tt>n|n</tt> is <b>max</b>. Fully atomic. For booleans, effectively logical <b>or</b>.
<pre><code>  3 5 7|0 6 9
3 6 9</code></pre>
	</td>
</tr>
<tr>
	<td>
		<a name="asc"/>
		<tt>&lt;l</tt> is <b>asc</b>, also known as "grade up".<br>
		Generate a permutation vector which would sort argument into ascending order.
<pre><code>  <5 8 2 7
2 0 3 1</code></pre>
	</td>
	<td>
		<tt>n&lt;n</tt> is <b>less</b>. Fully atomic.
<pre><code>  2 3 5<1 0 6
0 0 1</code></pre>
	</td>
</tr>
<tr>
	<td>
		<a name="desc"/>
		<tt>&gt;l</tt> is <b>desc</b>, also known as "grade down".<br>
		Generate a permutation vector which would sort argument into descending order.
<pre><code>  <5 8 2 7
1 3 0 2</code></pre>
	</td>
	<td>
		<tt>n&gt;n</tt> is <b>more</b>. Fully atomic.
<pre><code>  2 3 5>1 0 6
1 1 0</code></pre>
	</td>
</tr>
<tr>
	<td>
		<a name="group"/>
		<tt>=l</tt> is <b>group</b>. Generate a dictionary from items to the indices where they were found.
<pre><code>  =`c`a`b`b`a`c`a
[c:0 5;a:1 4 6;b:2 3]</code></pre>
	</td>
	<td>
		<tt>a=a</tt> is <b>equal</b>. Fully atomic.
<pre><code>  3 4 6=6
0 0 1</code></pre>
	</td>
</tr>
<tr>
	<td>
		<a name="not"/>
		<tt>~n</tt> is <b>not</b>. Nonzero numbers become 0 and 0 becomes 1. Right atomic.
<pre><code>  ~(0 1;3 7 -1)
(1 0
 0 0 0)</code></pre>
		<tt>~d</tt> applies to the values of dictionaries.
<pre><code>   ~[a:2 0;b:1]
[a:0 1;b:0]</code></pre>
	</td>
	<td>
		<tt>a~a</tt> is <b>match</b>. Returns 1 if x and y are recursively identical.
<pre><code>  (`a;2 3 4)~(`a;2 3)
0
  (`a;2 3 4)~(`a;2 3 4)
1</code></pre>
	</td>
</tr>
<tr>
	<td>
		<a name="enlist"/>
		<tt>,a</tt> is <b>enlist</b>. Place item in a 1-length list.
<pre><code>  ,1 2 3
,1 2 3</code></pre>
	</td>
	<td>
		<tt>a,a</tt> is <b>concat</b>. Join together lists or atoms to produce a list.
<pre><code>  1,2 3
1 2 3
  2,3
2 3
  1,()
,1</code></pre>
		Concatenating dictionaries favors values of y:
<pre><code>  [a:1;b:3],[b:5;c:7]
[a:1;b:5;c:7]</code></pre>
	</td>
</tr>

<tr>
	<td>
		<a name="null"/>
		<tt>^a</tt> is <b>null</b>. Is this item a null? Right atomic.
<pre><code>  ^(5;`;0N)
0 1 1</code></pre>
	</td>
	<td>
		<tt>l^a</tt> or <tt>l^l</tt> is <b>except</b>. Remove the first instance of each of y from x.
<pre><code>  1 3 2 5 1 2 3^1 3 5
2 1 2 3</code></pre>
		
		<tt>n^a</tt> or <tt>n^l</tt> is equivalent to <tt>(!n)^l</tt>:
<pre><code> 10^1 3 7
0 2 4 5 6 8 9</code></pre>
	</td>
</tr>

<tr>
	<td>
		<a name="count"/>
		<tt>#l</tt> is <b>count</b>. Atoms have count 1.
<pre><code>  #4 7 10
3
  #[a:3;b:17]
17 17 17
  #"c"
1</code></pre>
	</td>
	<td>
		<tt>n#l</tt> or <tt>n#a</tt> is <b>take</b>. Truncate or repeat y to produce a list of length x.<br>
		A negative x takes from the end of y.
<pre><code>  2#"ABC"
"AB"
  6#"ABC"
"ABCABC"
  -2#"ABC"
"BC"</code></pre>

		<tt>l#l</tt> or <tt>l#a</tt> is <b>reshape</b>.<br>
		Works like take, but creates an arbitrary dimensioned result based on x.<br>
<pre><code>  3 2#1 2 3
(1 2
 3 1
 2 3)
  3 2 2#1 2 3
((1 2
  3 1)
 (2 3
  1 2)
 (3 1
  2 3))</code></pre>

		If the leading or trailing element of the rank vector is <tt>0N</tt>, reshape treats that dimension as maximal:<br>
<pre><code>  0N 3#!6
(0 1 2
 3 4 5)
  2 0N#!8
(0 1 2 3
 4 5 6 7)</code></pre>
	</td>
</tr>
<tr>
	<td>
		<a name="floor"/>
		<tt>_n</tt> is <b>floor</b>. Right atomic.
<pre><code>  _2.3 7.6 9 -2.3
2 7 9 -3</code></pre>
	</td>
	<td>
		<tt>n_l</tt> is <b>drop</b>. Remove x elements from the start of y.<br>
		A negative x drops from the end of y.
<pre><code>  3_"ABCDE"
"DE"
  -3_"ABCDE"
"AB"</code></pre>
		<tt>l_l</tt> is <b>cut</b>. Splits y at the indices given in x.<br>
		The indices must be ascending.
<pre><code>  0 4_"feedface"
("feed";"face")
  1 2 4_"feedface"
(,"e";"ed";"face")</code></pre>
	</td>
</tr>
<tr>
	<td>
		<a name="string"/>
		<tt>$a</tt> is <b>string</b>. Convert atoms into strings. Right atomic.
<pre><code>  $120 4
("120";,"4")
  $`beef
"beef"</code></pre>
	</td>
	<td>
		<tt>n$a</tt> is <b>pad</b>. Add spaces to strings to make them x characters long.<br>
		A negative value pads from the left. Mostly fully atomic, strings are treated specially.
<pre><code>  5$"beef"
"beef "
  -7$"beef"
"   beef"
2 3 4$("a";"b";"c")
("a "
 "b  "
 "c   ")</code></pre>

		<tt>a$a</tt> is <b>cast</b>. Convert values to a different type based on a symbol.<br>
		Fully atomic.
<pre><code>  `i$"Hello."
72 101 108 108 111 46
  `c$72 101 108 108 111 46
"Hello."
  `f`i`b$31
31 31 1</code></pre>
		A short list of conversion symbols:
		<ul>
			<li><tt>`c</tt>: convert to character</li>
			<li><tt>`i</tt>: convert to integer</li>
			<li><tt>`b</tt>: convert to boolean (bitwise AND with 1)</li>
		</ul>
	</td>
</tr>
<tr>
	<td>
		<a name="distinct"/>
		<tt>?l</tt> is <b>distinct</b>. Produce a list of the unique elements in a list.
<pre><code>  ?(`a`b;3;`a`b;7;3)
(`a`b
 3
 7)</code></pre>

		<tt>?n</tt> produces a list of x random floats from 0 up to but excluding 1.
<pre><code>  ?6
0.197 0.8382 0.1811 0.9084 0.6113 0.1958</code></pre>
	</td>
	<td>
		<tt>l?a</tt> is <b>find</b>. Determine the index of y in x. Returns 0N if not found.</br>
		Right atomic.
<pre><code>  "XYZ"?"XYXZB"
0 1 0 2 0N</code></pre>

		<tt>n?n</tt> is <b>random</b>. Produce x random integers from 0 up to but excluding y.
<pre><code>  5?10
0 3 3 7 7
  5?10
3 5 2 7 9</code></pre>

		<tt>n?l</tt> picks random elements from y.
<pre><code>  8?"ABC"
"ACBBCBCB"</code></pre>
	</td>
</tr>
<tr>
	<td>
		<a name="type"/>
		<tt>@a</tt> or <tt>@l</tt> is <b>type</b>. Returns a magic number indicating the type of the noun.<br>
		General lists are 0, listy things are positive and non-listy things are negative.
<pre><code>  @`a
-11
  @"d"
-10
  @()
0</code></pre>
	</td>
	<td>
		<tt>x@y</tt> is <b>at</b>. Index a list or dictionary x, apply a single argument to a function x.
<pre><code>  3 7 8@0 1 1
3 7 7
  [a:4;b:7]@`a
4
  {x*x}@5
25</code></pre>

		Invalid indices will produce 0N:
<pre><code>  [a:4;b:7]@`q
0N
  3 7 8@4
0N</code></pre>
	</td>
</tr>
<tr>
	<td>
		<a name="val"/>
		<tt>.l</tt> is <b>value</b>. Evaluate K expressions from strings.
<pre><code>  ."1+2"
3</code></pre>
	</td>
	<td>
		<tt>l.a</tt> or <tt>l.l</tt> is <b>dot-apply</b>. Index at depth or apply a list of arguments to a function.
<pre><code>  (2 3;4 5).(1 0)
4
  {x,2*y}.(3 5)
3 10</code></pre>
	</td>
</tr>
<tr>
	<td>
		<a name="bin"/>
		n/a
	</td>
	<td>
		<tt>l'a</tt> is <b>bin</b>. Perform a binary search for y in x. Right atomic.<br>
		x must already be sorted.
<pre><code>  0 2 4 6 8 10'5
2
  0 2 4 6 8 10'-10 0 4 5 6 20
-1 0 2 2 3 5</code></pre>
	</td>
</tr>
<tr>
	<td>
		<a name="join"/>
		n/a
	</td>
	<td>
		<tt>a/l</tt> is <b>join</b>. Place the character x between strings y.
<pre><code>  "|"/("a";"bc";"def")
"a|bc|def"</code></pre>
	<tt>l/l</tt> is <b>encode</b>. Combine digits y in a base x into a single value.
<pre><code>  2 2 2/1 0 1
5
  10 10 10/3 4 5
345</code></pre>
	</td>
</tr>
<tr>
	<td>
		<a name="split"/>
		n/a
	</td>
	<td>
		<tt>a\l</tt> is <b>split</b>. Break the string y apart at instances of character x.
<pre><code>  ","\"cat,dog,banana"
("cat"
 "dog"
 "banana")</code></pre>
		<tt>l\n</tt> is <b>decode</b>. Split a number y into a given base x.
<pre><code>  2 2 2\5
1 0 1
  10 10 10\345
3 4 5</code></pre>
	</td>
</tr>
</table>

Adverb Reference
----------------

['](#each) [':](#eachprior) [/:](#eachright) [\:](#eachleft) [/](#over) [\ ](#scan)

<a name="each"/>
`m'l` is <b>each</b>. Apply the monad to each x, producing a new list. If x is an atom, this is equivalent to `m@a`.

	  {2*x}'5 7 2
	10 14 4

`x d'y` is <b>each dyad</b>. Pair up values from x and y and apply them to the dyad, producing a new list. If x or y is an atom, spread it to the elements of the other sequence.

	  2 7 9{x,2*y}'1 3 4
	(2 2
	 7 6
	 9 8)
	  (5),'1 3 4
	(5 1
	 5 3
	 5 4)

<a name="eachprior"/>
`d':l` is <b>eachprior</b>. Apply the dyad to each element of the list (left argument) and the element preceding that element in the list (right argument), producing a new list. Consistent with list indexing, the first element of the list will thus be paired up with 0N. Some primitive verbs result in a different special-cased initial value: `+`, `*`, `-` and `&` are provided with 0, 1, 0 or the first element of the sequence, respectively.

	  =':3 3 4 4 5
	0 1 0 1 0
	  -':1 3 5 2 9
	1 2 2 -3 7
	  ,':2 3 4
	(2 0N
	 3 2
	 4 3)

`a d':l` lets you specify an explicit initial value for eachprior:

	  99,':2 3 4
	(2 99
	 3 2
	 4 3)

<a name="eachright"/>
`x d/:l` is <b>eachright</b>. Apply the dyad to the entire left argument and each right argument, producing a new list.

	  2 3,/:4 5 6
	(2 3 4
	 2 3 5
	 2 3 6)

<a name="eachleft"/>
`l d\:x` is <b>eachleft</b>. Apply the dyad to each left argument and the entire right argument, producing a new list.

	  2 3,\:4 5 6
	(2 4 5 6
	 3 4 5 6)

<a name="over"/>
`d/l` is <b>over</b>, also known as *foldl*. Apply the dyad to pairs of values in x from left to right and carrying each result forward, reducing the list to a single value. Some primitive verbs result in a different special-cased value when applied to an empty list: `+`, `*`, `|` and `&` result in 0, 1, negative infinity or positive infinity, respectively.

	  +/4 5 2
	11
	  |/()
	-0w

`x d/l` lets you specify an initial value for over:

	  4+/5 2
	11

`m/x` is <b>fixedpoint</b>. Repeatedly apply the monad to x until it stops changing or repeats its initial value. The scan form of this adverb is often useful for getting a better understanding of what is happening.

	  {_x%2}/32
	0

`n m/y` is <b>for</b>. Apply the monad to y, x times.

	  5{,x}/7
	,,,,,7

`m m/x` is <b>while</b>. Repeatedly apply the monad to x as long as the left monad applied to the result is true.

	  {x<100}{x*2}/1
	128

<a name="scan"/>
`d\l` is <b>scan</b>. Scan and its variants all behave identically to over, except they accumulate a list of intermediate results rather than just returning the final result. Apply the dyad to pairs of values in x from left to right and carrying each result forward.

	  +\4 5 6
	4 9 15

`x d\l` lets you specify an initial value for scan. Note that this value is not included in the results:

	  2*\3 2 9
	6 12 108

`m\x` is <b>scan-fixedpoint</b>. Repeatedly apply the monad to x until it stops changing or repeats its initial value and accumulate a list of intermediate results.

	  {_x%2}\32
	32 16 8 4 2 1 0
	  {3!1+x}\2
	2 0 1

`n m/y` is <b>scan-for</b>. Apply the monad to y, x times and accumulate a list of intermediate results.

	  5{,x}\7
	(7
	 ,7
	 ,,7
	 ,,,7
	 ,,,,7
	 ,,,,,7)

`m m\x` is <b>scan-while</b>. Repeatedly apply the monad to x as long as the left monad applied to the result is true and accumulate a list of intermediate results.

	  {x<100}{x*2}\1
	1 2 4 8 16 32 64 128

Special Forms
-------------
The verbs `?`, `@` and `.` have special triadic and tetradic overloads in k5 to perform miscellaneous functions. This section will try to explain known overloads on a case-by-case basis.

`?[l;x;v]` is <b>splice</b>. Replace the elements of `l` in the interval given by `x` with `v`. `x` must be a length-2 list.

	  ?[1 2 3;1 1;4]
	1 4 2 3
	  ?["test";1 3;"u"]
	"tut"
	  ?["hello world";0 5;"goodbye"]
	"goodbye world"

If `v` is a monadic verb train or function, it is applied to the elements specified by the interval rather than simply replacing them:

	  ?[2 7 9;1 2;2*]
	2 14 9
	  ?["a look back";2 6;|:]
	"a kool back"
