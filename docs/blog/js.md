## 前言

在网上看到一个有趣的测试，[访问地址](https://jsisweird.com/)。里面包含了 25 道选择题，每个都是一个简单的表达式，然后让你选择，都是一些 JavaScript 怪异行为的体现，最后网站生成答案和解析，帮助你更好的理解 JavaScript 怪异的行为。

这里我分享 10 道我认为有趣的，希望对大家有所帮助

## 测试

1. `[,,,].length`

   1. 0
   2. 3
   3. 4
   4. SyntaxError

   

2. 10,2

   1. 10.2
   2. 10
   3. 2
   4. 20

   

3. true == "true"

   1. true
   2. false

   

4. 010 - 03

   1. 7
   2. 5
   3. 3
   4. NaN

   

5. **1/0 > Math.pow(10, 1000)**

   1. true
   2. false
   3. NaN
   4. SyntaxError

   

6. **0/0**

   1. 0
   2. Infinity
   3. NaN
   4. SyntaxError

   

7. **true++**

   1. 2
   2. undefined
   3. NaN
   4. SyntaxError

   

8. true + ("true" - 0)

   1. 1
   2. 2
   3. NaN
   4. SyntaxError

   

9. **undefined + false**

   1. "undefinedfalse"
   2. 0
   3. NaN
   4. SyntaxError

   

10. **NaN++**

    1. NaN
    2. Undefined
    3. TypeError
    4. SyntaxError

## 答案

1. `[,,,].length`

   1. 0
   2. 3
   3. 4
   4. SyntaxError

   解析：b。稀疏数组，`[,,]` 中间的元素为 `empty` ，这种我们就称为稀疏数组，我们也可以通过类似 new Array(2) 的方式创建稀疏数组。那为什么不是 4 而是 3 呢？这个跟 JavaScript 的[尾后逗号](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Trailing_commas) (Trailing commas) 有关。MDN 中的解析如下：

   > **尾后逗号** （有时叫做 “终止逗号”）在向 JavaScript 代码添加元素、参数、属性时十分有用。如果你想要添加新的属性，并且上一行已经使用了尾后逗号，你可以仅仅添加新的一行，而不需要修改上一行。这使得版本控制的代码比较（diff）更加清晰，代码编辑过程中遇到的麻烦更少。

   其实这个在我们平时写对象的时候用得比较多（假如 ESlint 允许的话）。

   ```js
   var object = {
     foo: "bar",
     baz: "qwerty",
     age: 42, // 注意
   };
   ```

   其实数组中也一样，这样就不难理解上面的输出了。

   ```js
   var arr = [
     1,
     2,
     3,
   ];
   
   arr; // [1, 2, 3]
   arr.length; // 3
   ```

2. 10,2

   1. 10.2
   2. 10
   3. 2
   4. 20

   解析：c。逗号操作符只返回最后一个操作符的值。这允许你创建一个复合表达式，在其中计算多个表达式，复合表达式为最后一个表达式的值。在 for  循环中可能会用到。

   

   ```js
   for (var i = 0, j = 9; i <= 9; i++, j--)
     console.log('a[' + i + '][' + j + '] = ' + a[i][j]);
   ```

   ```js
   function myFunc() {
     var x = 0;
   
     return (x += 1, x); // the same as return ++x;
   }
   ```

3. true == "true"

   1. true
   2. false

   解析：b。根据隐式类型转换的规则。

   ```js
   Number(true); // -> 1
   Number("true"); // -> NaN
   1 == NaN; // -> false
   ```

4. 010 - 03

   1. 7
   2. 5
   3. 3
   4. NaN

   解析：b。010 被 JavaScript 视为八进制数。因此，它的值是以8为基数的。010 被解析成 8，减 3 得 5。

5. **1/0 > Math.pow(10, 1000)**

   1. true
   2. false
   3. NaN
   4. SyntaxError

   解析：b。两个表达式都是 Infinity。Infinity 两者相等。

   ```js
   1/0; // -> Infinity
   Math.pow(10, 1000); // -> Infinity
   Infinity > Infinity; // -> false
   ```

6. **0/0**

   1. 0
   2. Infinity
   3. NaN
   4. SyntaxError

   解析：由于等式 `0/0` 是没有有意义的数值答案，输出简单地是 `NaN`。

7. **true++**

   1. 2
   2. undefined
   3. NaN
   4. SyntaxError

   解析：d。会存在以下的怪异行为，undefined 不会报错。【这里我也找不到合适的理由去解释】。

   ```js
   1++; // -> SyntaxError
   "x"++; // -> SyntaxError
   undefined++; // -> NaN
   ```

   假如要使它不报错的话，就得：

   ```js
   let x = true;
   x++;
   x; // -> 2
   ```

8. true + ("true" - 0)

   1. 1
   2. 2
   3. NaN
   4. SyntaxError

   解析：同 3。

   ```js
   Number("true"); // -> NaN
   ```

9. **undefined + false**

   1. "undefinedfalse"
   2. 0
   3. NaN
   4. SyntaxError

   解析：`false` 可以转换为数字，`undefined` 不能。

   ```js
   Number(false); // -> 0
   Number(undefined); // -> NaN
   NaN + 0; // -> NaN
   ```

   但是 undefined 又可以转换成 false。

   ```js
   !!undefined === false; // -> true
   ```

   所以要以上为 0。应该：

   ```js
   !!undefined + false; // -> 0
   ```

10. **NaN++**

    1. NaN
    2. Undefined
    3. TypeError
    4. SyntaxError

    答案：a。`NaN` 不是一个数字，所以它不能递增。这也意味着 `NaN` 和 `NaN++` 表示相同的值。

## 结语

Javascript 之所以有以上怪异表现，主要是初期设计过于匆忙，1995 年仅用用了 10 天来完成的。可能上面的行为我们用得不多，但了解它们对于我们更加深入了解 JavaScript 也是有所帮助。所以强烈建议大家去做一些这 [25 道题目](https://jsisweird.com/)，感受一下。