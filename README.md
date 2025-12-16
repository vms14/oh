# oh

```oh
1 2 +
```

* A stack rpn language

It's a javascript file that you can download and run in browser or in node:

https://raw.githubusercontent.com/vms14/oh/refs/heads/main/oh.js
or
https://vms14.github.io/oh.js

## Usage:

```bash
node oh.js
1 2 +
[ 3 ]
```
* In node it launches a read eval print loop that prints the contents of the stack

```html
<!DOCTYPE html>
<html>
<head>
 <script src="https://vms14.github.io/oh.js"></script>
</head>
<body>
 <oh> (p "oh...") dom to-body </oh>
</body>
</html>
```

* In the browser it loads an executes <oh> tags after the document loads

```html
<oh src="oh.oh"></oh>
```

* If the tags have a src attribute it will fetch then and evaluate it

```html
<oh src="oh.oh"> 1 2 + </oh>
```

* If a tag has both a src attribute and code inside it, the src file will get fetched and evaluated first, after that then the code inside the tag will evaluate

