# Crash course in Nifki

Nifki takes the “own little world” model to an extreme. You cannot use it to write anything except little games that run in a single window using a sprite-based graphics library. The upside is that the Nifki language can be extremely simple and well suited to its intended use. For example, all I/O is done using special language keywords, and there is no need for standard libraries.

Hello world is written as follows:

```
DUMP "Hello, world!\A/"
```

(The `DUMP` command is really intended for debugging. It’s not very powerful.)

Nifki encourages plagiarism, which some people call “code re-use”. To this end, source code on one page can refer to subroutines, resource files and other global variables belonging to other pages, simply by prefixing their variable names with the page name and an underscore. For example, any page can use `Rocks_wallPNG` to refer to the `wallPNG` image of the `Rocks` page. When the program is compiled, the compiler works out what it has to include.

Now that the `Rocks` page exists, the minimal maze game is written as follows:


```
Rocks_playUntilWin(["####", "#A+#", "####"])
```

In addition to global variables there are also local variables, which are private to subroutine, let alone a page. Loop variables and subroutine arguments are always local variables. You are discouraged from using a global and a local with the same name on the same page. With the exception of subroutine names and picture names, which are always global variables, variables are local by default. To make a variable global, every assignment to it must either use a page prefix (e.g. `Rocks_`) or the keyword `GLOBAL`. When reading from a global variable, the page prefix can be omitted if the variable belongs to your own page.

Here’s a simple program that uses local variables `pos` and `char`:

```
FOR pos=char IN "Hello, world!" {
  DUMP "Character " DUMP pos DUMP " is " DUMP char DUMP "\A/"
}
```

As you see, there is no statement separator in Nifki. The syntax is designed not to need one. In theory. ☺

Subroutines are defined using the keyword `DEF`. Subroutine definitions can appear anywhere in the code but cannot be nested. All subroutine definitions on all pages are processed before execution starts, so it doesn’t matter where the definitions are.

Here’s a program that tabulates the factorials of the numbers from 0 to 9:

```
FOR i= IN 10 { DUMP factorial(i) }

DEF factorial(i) {
  IF (i==0) { RETURN 1 }
  RETURN i * factorial(i-1)
}
```

Almost everything is immutable in Nifki. This is another way of saying that almost everything is passed by value. For example, the following code prints `6` whereas the corresponding code in Python or Lua would print `7`:

```
a = [6]
b = a
b[0] = 7
DUMP a[0]
```

The only exceptions are sprites and the `WINDOW` object, which represents the screen. These are passed by reference. For example, the following snippets of code *both* turn the background red:

```
SET WINDOW.R = 1
WAIT

a = WINDOW
SET a.R = 1
WAIT
```

The `SET` keyword reminds you that you are modifying an object in-place. You must use it when modifying attributes of sprites or the window, and you must not use it for ordinary assignments to variables. The two cases are compiled completely differently, and if you use the wrong syntax you will get an error at run-time.

The `WAIT` command in the above examples is necessary for all graphical output. It tells the virtual machine to draw everything and then pause until it is time for the next animation frame.

The following code fades the screen slowly from black to red:

```
FOR i= IN 1000 {
  SET WINDOW.R = i/1000
  WAIT
}
```

To put a picture of a man on the screen, do this:

```
MOVE SPRITE(Rocks_manPNG) TO (112, 112)
WAIT
```

The coordinates `(112, 112)` will put the man sprite, which is 32x32 pixels, in the centre of the screen, if the screen is 256x256 pixels. They identify the top-left corner of the sprite. Coordinates are measured right and down.

To put a row of men on the screen, do this:

```
FOR x= IN 225 {
  MOVE SPRITE(Rocks_manPNG) TO (x, 112)
  WAIT
}
```

To move a man across the screen, do this instead:

```
man = SPRITE(Rocks_manPNG)
FOR x= IN 225 {
  MOVE man TO (x, 112)
  WAIT
}
```

The difference is that the first version uses the `SPRITE` command every time round the loop, where as the second version uses it once before the loop starts. This illustrates that sprites are passed by reference, unlike most other values in Nifki.

To make the man fill the screen, do this:

```
man = SPRITE(Rocks_manPNG)
MOVE man TO (WINDOW.X, WINDOW.Y)
RESIZE man TO (WINDOW.W, WINDOW.H)
WAIT
```

I could have written `(0, 0)` for the position and `(256, 256)` for the size, but that would have assumed facts about the `WINDOW` object. The `MOVE` and `RESIZE` commands can be applied to the `WINDOW` just as to sprites, and the effect is scrolling and zooming. As above, the current position of the `WINDOW` or of a sprite can be read using the attributes `X`, `Y`, `W` and `H`. If you want, you can modify these attributes using the `SET` command instead of the `MOVE` and `RESIZE` commands.

There is another attribute `IsVisible` which controls whether a sprite appears at all. (The `WINDOW` has an `IsVisible` attribute but ignores it). It takes a boolean value: either `TRUE` or `FALSE`. You can change the `IsVisible` attribute to `FALSE` using the `HIDE` command. The `MOVE` command sets it to `TRUE`. Alternatively, you can just using plain old `SET`. You can also hide all sprites using `CLS`.

The following code flashes a man on and off (you’ll want to run it with a slow frame rate):

```
man = SPRITE(Rocks_manPNG)
WHILE (TRUE) {
  MOVE man TO (112, 112)
  WAIT
  HIDE man
  WAIT
}
```

Sprites also have an attribute called `Picture` which controls the picture used to display the sprite. The following code toggles a sprite between a rock and a diamond:

```
sp = SPRITE(Rocks_rockPNG)
MOVE sp TO (112, 112)
WHILE (TRUE) {
  SET sp.Picture = Rocks_rockPNG
  WAIT
  SET sp.Picture = Rocks_diamondPNG
  WAIT
}
```

Random numbers can be generated using the `RANDOM` keyword. The following code gradually fills the screen with rocks:

```
WHILE (TRUE) {
  MOVE SPRITE(Rocks_rockPNG) TO (224*RANDOM, 224*RANDOM)
  WAIT
}
```

To read the keyboard, use the `KEYS` command. The following code:

```
DUMP KEYS
```

prints the current state of the keyboard, which will look something like this:

```
["Alt"=FALSE, "BackQuote"=FALSE, "BackSlash"=FALSE, "BackSpace"=FALSE,
"Break"=FALSE, "CapsLock"=FALSE,    ... snip several lines ...    ,
"Shift"=FALSE, "Slash"=FALSE, "Space"=FALSE, "Tab"=FALSE, "UpArrow"=FALSE]
```

This is a table mapping strings (the names of the keys) to booleans (`TRUE` if the key is currently pressed, otherwise `FALSE`). The keys are listed in alphabetical order.

Usually you will only be interested in particular keys. You have a choice of two notations for extracting values from a table. For example, to read the state of the `UpArrow` key, you can write either of the following:

```
IF KEYS["UpArrow"] { DUMP "Yes\A/" } ELSE { DUMP "No\A/" }

IF KEYS.UpArrow { DUMP "Yes\A/" } ELSE { DUMP "No\A/" }
```

The former notation is the more general, because the latter notation can only be used when the key (here `UpArrow`) is a constant string that looks like a variable name.

The following code uses the latter notation to move a man around the screen:

```
man = SPRITE(Rocks_manPNG)
MOVE man TO (112, 112)
WHILE (TRUE) {
  IF KEYS.UpArrow { MOVE man BY (, -1) }
  IF KEYS.DownArrow { MOVE man BY (, 1) }
  IF KEYS.LeftArrow { MOVE man BY (-1, ) }
  IF KEYS.RightArrow { MOVE man BY (1, ) }
  WAIT
}
```

Almost anything can be used as a key in a table, including booleans, numbers, strings and pictures. You can also use whole tables as keys, provided they do not contain anything that can’t be used as a table key. The things which can’t be used as keys are subroutines, sprites and the window object.

The following code uses a table with numbers as keys. It implements the sieve of Eratosthenes to calculate all prime numbers less than 1000:

```
max = 1000
primes = []
FOR n= IN max-2 { primes[n+2] = TRUE }
FOR n= IN primes { IF primes[n] {
  DUMP n DUMP " "
  k = n*n
  WHILE k<max {
    primes[k] = FALSE
    k = k + n
  }
}}
DUMP "\A/"
```

Keys in a table are always sorted. The following program uses this fact to sort a list of strings:

```
toSort = ["frog", "goose", "sheep", "pig", "horse", "cow", "duck"]
table = []
FOR =animal IN toSort { table[animal] = [] }
FOR animal= IN table { DUMP animal DUMP "\A/" }
```

The first `FOR` loop is through the *values* in the list. The keys of the list are just 0, 1, 2, …, 6. The second `FOR` loop is through the *keys* of the table. The values are all the empty table `[]`.

Tables are very powerful. You can use them to build pretty much any data structure conveniently.
