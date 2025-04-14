---
title: "NDepend: Matrix"
datePublished: Mon Jun 25 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iyfon0104zfs108042vfm
slug: ndepend-matrix
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617812931130/V-x1YeP14.png
tags: dotnet, dotnetcore

---


The graph is a good way to understand the structure of smaller projects, but gets messy in larger projects. That's why I compare the matrix of a big project against the matrix of the smaller project.

> Just a small disclaimer: NDepend is sponsoring my blog with a one year subscription. My views are still my own, if I didn't like their product, I wouldn't be so positive about it.

The overview a matrix provides for bigger projects is immediately obvious, although it will still take me time to sift through all the data. Unfortunately because big projects are big, this will never be an easy task. NDepend does provide the data in an easily consumable format. Left are all the projects in the solution followed by all the assemblies that those projects use. On the top are all the projects in the solution. The matrix displays how many dependencies there are between two projects. A green box indicates that the left assembly uses the top assembly, a blue box indicates that the left assembly is using the top assembly. There is also a black colour, not present in this solution, that indicates that the projects are mutually dependant.

![matrix-big-project.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381108831/jd_CbHlSC.png)

In this case, the numbers represent _direct_ types used. This means that actual classes need to be used to get a count. The small project's domain has a dependency on the data access, which I talked about last week. However, there is a count of 0 in this box. This is because the domain only uses an interface from the data access and no actual classes. This is good to know, because it means that if I can move all interfaces out of the data access project, I could get rid of this dependency.

Let's take a closer look at the big project. In the left corner of the matrix, there are two dependencies that have 202 and 301 direct references. The 301 references are from a test project to the _Win_ project which is the presentation layer, the front end or at least a part of it. This explains why there are so many usages between the two projects as the tests instantiate a lot of different classes in the _Win_ project to be able to test them. It also illustrates that these two projects are very tightly coupled and if I would like to separate them, I'd have a lot of work ahead of me.

There is another useful view in the matrix and that is the direct and indirect dependencies. The view doesn't change dramatically, but the meaning does. I'm going to focus on the big project here for a moment because that's the one that has the best and easiest to understand examples.

![matrix-big-project-dependencies.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381110579/2i0qbcDoL.png)

The graph now displays how many projects it takes to connect these two assemblies. This lets me know that the _ReturnWin_ (graphical user interface) needs to go through _Master.Win_ project and another business layer to get to the Pool.Business layer. Personally, I'd like to see this change, because this indicates that there are hidden dependencies. Hidden dependencies always make the job of maintaining software harder. If I change something in what seems to be a completely unrelated project, it could have consequences in other projects that are not immediately clear.

![dependency-graph-win-business.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381112369/H-FL1ofLB.png)

What I would like to see is that the multiple _Win_ projects merge into one project that takes care of all the graphical user interface interactions. The different business and database access projects can then be referenced by the single _Win_ project. This would not only simplify the architecture, it would also bring to light a lot of dependencies that should not be made.

I do understand that quick solutions sometimes are needed to solve current problems. Unfortunately that means that in the long run, many more problems will arise from these short term solutions. Maintaining this big project is not fun and is slow going. On top of that, each change is a veiled threat that something else, somewhere unrelated will break. Don't make these decisions lightly.

After spending some time in the matrix report, it's very clear that there is a lot more information hidden here. Proper analysis takes time and in my opinion, NDepend can help speed this up and help shine a light into a complex code base. There are probably a dozen features I know nothing about that can make this even easier. Learning to use the tools at hand to their full extent is also a valuable as they can save me loads of time.

NDepend isn't just great for architectural purposes, it also offers a wide range of insights into how clean the code that was written is. It has numerous code inspections available to me. I'll delve into those next.
