参考资料：https://github.com/isaacs/node-glob

下文是关于 Gulp 的 `gulp.src(globs[, options])` 方法的第一个参数 `globs` 的学习。

需要注意的是：
当该参数 `globs` 为数组时，其包含的多个 glob 会**按顺序进行解析**，这意味着以下意图是可以实现的：

    // 排除所有以字母 b 开头的 js 文件，但不排除 bad.js
    gulp.src(['*.js', '!b*.js', 'bad.js'])

---

## Glob
使用 shell 里的 patterns 匹配文件，如 `*` 等。

## 用法
### 用 npm 安装 glob

    npm i glob
    
编写js代码：

    var glob = require("glob");
    
    // options 是可选参数
    glob("**/*.js", options, function (er, files) {
      // files 参数是一个文件名数组。
      // 若参数 options 的 `nonull` 属性为 true，则在匹配不到文件时， files 参数则为 ["**/*.js"]。
      //（ 若 `nonull` 为 false 时， files 为空数组）。
      // er 是一个 error 对象或 null。
    })
 
## Glob 初级
"Globs" 是你通过在命令行输入字符后完成某些操作时的 pattern。如 `ls *.js`，或将 `build/*` 放在 `.gitignore`。

在解析路径段的 patterns 前，braced sections 会展开为一个集合。braced sections 以 `{` 开头，`}` 为结尾，中间部分以英文逗号 `,` 分隔。braced sections 可以含有斜杠符号 `/`。因此，`a{/b/c,bcd}` 会展开为 `a/b/c` 和 `abcd`。

以下字符用在路径段时，会拥有特别的含义：

 - `*` ： 匹配0或多个字符。

        glob("js/*.js", function(err, files){
        	console.log(files);
        });
获取js目录下的所有js文件（不包括以`.`开头的文件，下文有方法解决：对 glob 方法的 options 参数的属性 `dot:true`）。


 - `?` ： 匹配一个字符（不能为空）。

        glob("js/a?.js", function(err, files){
            console.log(files);
        }
获取js目录下所有文件名长度为1字符的js文件。例如：能匹配 js/ab.js，不能匹配 js/a.js。

        
 - `[...]` ： 匹配该路径段中在指定范围内的一个字符。
注意：不能组合，只能匹配其中**一个**字符。另外，如果指定范围的首字符是 `!` 或 `^`，则匹配**不在指定范围内**的**一个**字符。
    
        glob("js/a[0-3].js", function(err, files){
            console.log(files);
        })
获取js目录下以`a`开头，第二个字符为0-3之间（包括0和3）的js文件。若改为 `["js/[^ab].js"]`，则匹配  js/c.js，不匹配 js/cd.js、js/ac.js。


 - `!(pattern|pattern|pattern)` ： 匹配（完全且精确地匹配，且不可组合）不符合任何模型之一的字符。注意 `|` 前后不能有空格，下同。
 
        glob("js/!(a|b).js", function(err, files){
        	console.log(files);
        });
匹配 js 目录下的 aa.js、ab.js、ba.js、c.js 不匹配 a.js、b.js。



 - `?(pattern|pattern|pattern)`：匹配多个 pattern 中 0 或 1 个（精确匹配，不可以组合）。
 
        glob("js/?(a|b).js", function(err, files){
        	console.log(files);
        });
匹配 js 目录下的 a.js、b.js，不匹配 ab.js


 - `+(pattern|pattern|pattern)` ： 至少匹配多个 pattern 中的一个。与`*(pattern|pattern|pattern)` 不用的是，它必须1个及以上，不能为空。

        glob("js/+(a|b)b.js", function(err, files){
         	console.log(files);
        });
匹配 js 目录下的 ab.js、bb.js、ababab.js，不能匹配 abcd.js（也就是说：只允许匹配出现在范围内的字符） ，也不能像 `js/*(a|b)b.js` 那样匹配 b.js。


 - `*(a|b|c)` ： 匹配括号中多个 pattern 中0或任意多个（pattern可相互组合）。 
 
        glob("js/*(a|b|c).js", function(err, files){
        	console.log(files);
        });
匹配 js 目录下的 a.js、ab.js、abc.js、ba.js，不匹配 abcd.js（也就是说：只允许匹配出现在范围内的字符）。


 - `@(pattern|pattern|pattern)` 匹配多个 pattern 中的任意一个（即不可以组合，且不能为空或大于1个）。与 `?(pattern|pattern|pattern)` 区别是不可为空。

        glob("js/@(a|b)b.js", function(err, files){
        	console.log(files);
        });
匹配 js 目录下的 ab.js、bb.js，不匹配 b.js、abb.js、abc.js。


 - `**` 与 `*` 类似，可以匹配任何内容（可匹配空），但 `**` 不仅能匹配路径中的特定一段，还能匹配后代所有目录（即多段路径段）。

        glob("js/**/*.js", function(err, files){
        	console.log(files);
        });
匹配 js 目录下所有js文件，如 js/a.js 或 js/a/b/c/d.js。
    
## Dots(即 `.`)
如果文件或目录的某路径段以 `.` 作为首字符，那么该路径段不会符合任何  glob pattern，除非该 pattern 的相应路径段同样以 `.` 作为首字符。

例如，pattern `a/.*/c` 会匹配文件 `a/.b/c`，而 pattern `a/*/c` 则不会匹配该文件，因为 `*` 不会匹配以 `.` 字符开头的文件。

可通过在 options 设置 `dot: true`，让 glob 将 `.` 视为普通字符。

## Basename 匹配
如果在 options 设置 `matchBase: true`，且 pattern 不含有 `/`，那么将会寻找任何匹配 basename 的文件，即在当前路径下的文件树进行搜索。例如，`*.js` 会匹配 `test/simple.basic.js`。

## 空集
如果不匹配任何文件，则会返回空数组。这点与 shell 不同，shell 会返回自身 pattern。

    $echo echo a*s*d*f
    a*s*d*f

若想得到 bash 那样的行为，可对 options 参数设置 `nonull:true`。

若发现文中有任何错误，或有任何好的建议，欢迎评论。

-------

GitHub：[关于 Glob (gulp) 的学习](https://github.com/JChehe/blog/blob/master/posts/%E5%85%B3%E4%BA%8E%20Glob%20%E7%9A%84%E5%AD%A6%E4%B9%A0.md)
