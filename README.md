### oh

* a stack based language using postfix notation

```oh
1 2 +
```

It's a javascript file that you can download from this repository

https://raw.githubusercontent.com/vms14/oh/refs/heads/main/oh.js

## Usage:

* In node it launches a read eval print loop that prints the contents of the stack

```bash
node oh.js
1 2 +
[ 3 ]
```

* In the browser it loads an executes `<oh></oh>` tags after the document loads

```html
<!DOCTYPE html>
<html>
<head>
 <script src="oh.js"></script>
</head>
<body>
 <oh> (p "oh...") dom to-body </oh>
</body>
</html>
```
* If the tags have a src attribute it will fetch the file and evaluate it

```html
<oh src="oh.oh"></oh>
```

* If a tag has both a src attribute and code inside it, the src file will get fetched and evaluated first, after that then the code inside the tag will be evaluated

```html
<oh src="oh.oh"> 1 2 + </oh>
```

## AI generated documentation:

* [Word list](https://github.com/vms14/oh/blob/main/words.md)
* [Overview](https://github.com/vms14/oh/blob/main/overview.md)
* [Internals](https://raw.githubusercontent.com/vms14/oh/refs/heads/main/internals.md)

### Video:
 [![Video oh](https://img.youtube.com/vi/ErPK1BeA27c/0.jpg)](https://youtu.be/ErPK1BeA27c)

### Audio: