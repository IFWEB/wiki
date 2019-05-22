## ptional install error
### Package require os(darwin) not compatible with your platform(win32)
虽然提示不适合Windows，但一般是某个包出了问题。根据错误提示判断是哪个包出了问题。解决方案一般是重新rebuild一下。  
比如后面描述是node-sass出了问题，解决方案  
**方案一：**  
```
cnpm rebuild node-sass
#不放心可以重新安装下
cnpm install
```
**方案二：**  
```
npm update
npm install
nodejs node_modules/node-sass/scripts/install.js
npm rebuild node-sass
```



