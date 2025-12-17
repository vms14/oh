### oh

oh is a stack based language using postfix notation implemented in javascript that runs in the browser and in node.

```oh
1 2 +
```

* It has a fake compilation procedure that turns everything into a function. Some words are immediate and execute at compile time.

* It tries to rely as much as possible on the host language (javascript) and to provide seamless interoperation with it.

* The data types are the data types of javascript. The stack is a javascript array and words can push or pop any number and type of elements from it.

* It is a tiny core that gets extended through the definition of words in the compile time environments.


It's a javascript file that you can download from this repository

https://raw.githubusercontent.com/vms14/oh/refs/heads/main/oh.js

## Usage:

* In node it launches a read eval print loop that prints the contents of the stack

```bash
node oh.js
1 2 +
[ 3 ]
```

* In the browser it loads an executes <oh> tags after the document loads

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

## Documentation:

[Tutorial](https://github.com/vms14/oh/blob/main/TUTORIAL.md)
[Internals](https://github.com/vms14/oh/blob/main/INTERNALS.md)

