---
title: "String interpolation speed"
datePublished: Mon Aug 08 2016 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0j0ts800zyyes122tnhlb6
slug: string-interpolation-speed
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1618422320812/mF-VdEtUE.png
tags: csharp, performance, string, dotnet, dotnetcore

---


A year, maybe two ago, I saw a talk of a [colleague](https://wesleycabus.be/) about performance. He compared `await/async` with more traditional threading options, demonstrated how slow reflection is and compared the three string concatenation options: sum notation, `string.Format()` and `StringBuilder`. Now that `string` interpolation is here, I was curious how fast is it compared to the other methods.

This is not a definitive test environment because I executed these tests only on my local machine. It's an i5 that's 5 years old with 8GB DDR3 RAM. So it's not a very fast machine, but I can't complain. If you want to run these tests yourself you can get all the code from [GitHub](https://github.com/KenBonny/CorePerformanceTests). If you have tips to improve the precision, they are always welcome through [mail](mailto:bonny.ken@gmail.com) or [twitter](https://twitter.com/bonny_ken/).

For each test (sum, format, builder, interpolation), I wrote 4 statistics expressed in ticks to measure the speed: average, median, mode/modal and total. After those tests, I ran a simple for loop and appended a number to a string using the different string options. It looked like this:

![execution-10000](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381220455/g6_aI13GA.jpeg)

Most values from my complex tests lie very close together. Which means that on average, the concatenation of 2 strings is overall very fast. First I was looking at the milliseconds that were elapsed, but that showed 0 ms each time. I decided to display the ticks that had passed instead. The interpolation is the slowest on average when concatenating 2 strings. This is including instantiating a new `StringBuilder` on each pass through my complex test.

After I built those complex tests, I remembered that a simple test sometimes gives the best overview. I added string concatenation that adds numbers to a string using the various methods. The iterations in these simple tests are the same as in the complex tests. The results are at the bottom '_Simple string concatenation tests:_'.

The `StringBuilder` is the fastest way if I would want to concatenate a lot of strings together. The total run time is less than a millisecond. Second place is for the sum notation.

The last place is, in my opinion, a shared place. The interpolation and format are so close together that it doesn't make a lot of difference. I ran the test a couple more times and the numbers are almost always less than 5 milliseconds apart. I was not surprised as these options both use more complex logic to concatenate strings.

When I run the test with only 100 iterations I see that the averages and modes stay the same, but the simple tests decrease immensely.

![execution-100](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381222132/qzOhX3gKU.jpeg)

The larger the number of iterations, the more the simple tests split up into two sides: the faster `StringBuilder` and the summation method against the much slower string interpolation and format.

![execution-100000](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381223888/nZqQmvrIn.jpeg)

I can conclude that for a few concatenations, there is almost no difference between all the options. It's only with large iterations (10.000 and above) that `StringBuilder` and, to lesser extent, the summation have a big impact on performance.
