"Jscex" is short for "JavaScript Computation EXpressions". It provide a monadic extensions for JavaScript language and would significantly improve your programming life in certain scenarios.

The core feature of Jscex is a compiler which turns a normal JavaScript function into a monadic one:

    var monadicFunction = eval(builderName, function (arg0, arg1, ...) {
        ... 
    });

As the most common usage, we use the $async builder for asynchronous programming (this sample can be found in samples/async/move.html):

    var moveAsync = eval(Jscex.compile("$async", function (e, startPos, endPos, duration) {
        for (var t = 0; t < duration; t += 50) {
            e.style.left = startPos.x + (endPos.x - startPos.x) * t / duration;
            e.style.top = startPos.y + (endPos.y - startPos.y) * t / duration;
            $await(Jscex.Async.sleep(50));
        }

        e.style.left = endPos.x;
        e.style.top = endPos.y;
    }));
    
    var moveSquareAsync = eval(Jscex.compile("$async", function (e) {
        $await(moveAsync(e, {x:100, y:100}, {x:400, y:100}, 1000));
        $await(moveAsync(e, {x:400, y:100}, {x:400, y:400}, 1000));
        $await(moveAsync(e, {x:400, y:400}, {x:100, y:400}, 1000));
        $await(moveAsync(e, {x:100, y:400}, {x:100, y:100}, 1000));
    }));

We can express our algorithms linearly, the compiler would generate the code as below:

    var moveAsync = function (e, startPos, endPos, duration) {
        return $async.Delay(function () {
            return $async.Combine(
                $async.Delay(function () {
                    var t = 0;
                    return $async.For(
                        function () { return t < duration; }, 
                        function () { t += 50; },
                        $async.Delay(function () {
                            e.style.left = startPos.x + (endPos.x - startPos.x) * t / duration;
                            e.style.top = startPos.y + (endPos.y - startPos.y) * t / duration;
                            return $async.Bind(Jscex.Async.sleep(50), function () {
                            return $async.Normal();
                        });
                    })
                );
            }),
            $async.Delay(function () {
                e.style.left = endPos.x;
                e.style.top = endPos.y;
                return $async.Normal();
            }));
        });
    };

    var moveSquareAsync = function (e) {
        return $async.Delay(function() {
            return $async.Bind(moveAsync(e, {x: 100, y: 100}, {x: 400, y: 100}, 1000), function() {
                return $async.Bind(moveAsync(e, {x: 400, y: 100}, {x: 400, y: 400}, 1000), function() {
                    return $async.Bind(moveAsync(e, {x: 400, y: 400}, {x: 100, y: 400}, 1000), function() {
                        return $async.Bind(moveAsync(e, {x: 100, y: 400}, {x: 100, y: 100}, 1000), function() {
                            return $async.Normal();
                        });
                    });
                });
            });
        });
    };
    
Currently Jscex and Jscex.Async are just prototypes for simplifing asynchronous programming. In theory, we can do more in Jscex with different types of builders.

For more information, please search for "F#", "Computation Expressions" and "Asynchronous Workflow" in your favourite search engines.

\- jeffz (jeffz[at]live[dot]com)
