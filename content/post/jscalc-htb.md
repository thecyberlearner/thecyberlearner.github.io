+++
title = 'jscalc HackTheBox'
date = 2024-08-22T04:29:09-04:00
summary = 'HTB web challenge jscalc walkthrough'
+++

# jscalc htb

*This is one of the web challenges in hackthebox. It is a calculator web application, that, uses eval for calculations. Who does that anyway?🤮*

Booting up the server, we find the calculator. Do a simple math and it performs the arithmetic

![](/images/jscalc/jscalc.png)

We are provided with the source code files. There are docker files(not important for us in this challenge), then the challenge files. The first file I checked was the index.js file to see what the application is doing after you enter some arithmetic. Most important part:

![](/images/jscalc/jscalcmainjs.png)

As we can see, it stringifies our input into a json with the name formula, then send it via POST to an api, /api/calculate.

Checking in routes how the api handles the data:

![](/images/jscalc/jscalcindexjs.png)

The result if okay is sent to the function Calculator.calculate(), which as we can see in the declaration is in /helpers/calculatorHelper.js. Visiting it, this is how it calculates:

```
module.exports = {
    calculate(formula) {
        try {
            return eval(`(function() { return ${ formula } ;}())`);

        } catch (e) {
            if (e instanceof SyntaxError) {
                return 'Something went wrong!';
            }
        }
    }
}
```

It passes the input passed via formula without any sanitization, an uses eval to calculate. This means we can inject commands. **Here you can craft commands to read files, access global variables, etc**. I crafted a js command to read the contents of a file synchronously, which I presumed should be flag.txt:
```
require('fs').readFileSync('/flag.txt', 'utf8')
```

![](/images/jscalc/jscalcCommandInjection.png)

We get the flag!