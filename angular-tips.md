在ng build --prod时候遇到了问题。
 
```
return binding.open(pathModule._makeLong(path), stringToFlags(flags), mode);
                 ^

Error: ENOENT: no such file or directory, open '/path/node_modules/_@angular_core@4.3.4@@angular/package.json'
    at Object.fs.openSync (fs.js:583:18)
    at Object.fs.readFileSync (fs.js:490:33)
```
遇到这个问题普遍的解决方法是不用cnpm,而用npm重新装包
 
此外有个快捷的解决方法: `--no-extract-license`
