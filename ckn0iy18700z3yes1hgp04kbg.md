---
title: "NDepend: Graph"
datePublished: Mon Jun 18 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0iy18700z3yes1hgp04kbg
slug: ndepend-graph
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617813024949/d7sFXOaop.png
tags: dotnet, dotnetcore

---


After [viewing the dashboard](http://kenbonny.net/2018/06/10/ndepend-introduction/), the next thing I took a look at is the Graph. This report gives an overview how the code is linked together. Either by projects or namespaces. In a smaller project, I immediately spotted something off. Let's delve a little deeper.

Just a small disclaimer if you didn't read the previous article: NDepend is sponsoring my blog with a one year subscription. My views are still my own, if I didn't like their product, I wouldn't be so positive about it.

Smaller projects are more easily analysed in this report. So I opened a smaller, non-trivial project to check out what I could learn. The first thing I noticed was that this screen can get pretty busy, pretty fast. Then again, displaying the relationships between projects is no easy task, [NDepend](https://www.ndepend.com/) displays this information as succinctly as possible. It's also quite easy to filter out a lot of classes to investigate a certain area of the codebase.

After opening the project and just looking at the graph, the first thing that was immediately clear is that there are way too many arrows pointing to the _DataAccess_ project. This indicates tight coupling and a lot of direct references. Upon further inspection, I discovered that the repository interfaces are located inside the _DataAccess_ project. This means that there is no inversion of control.

![dependency-graph-simple.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381089557/ijv5dAjia.png)

Let's imagine the perfect situation. There is a _Business_ project that needs data, so it creates a repository interface that asks for certain data. The _DataAccess_ project then implements that interface, fetches the data from SQL Server, Oracle database, Azure, CSV file, basically wherever it wants. When the _Business_ needs different data or needs to pass different parameters, it can change the interface. So if the interface changes, the _DataAccess_ would need to make the same changes. In this scenario, the Business project is in control of the repository interface.

Unfortunately, the graph indicates that this is not the ideal situation. The repository interface is located inside the _DataAccess_ project. So every change the _Business_ needs, it has to nicely ask the _DataAccess_ to implement. Here, the _DataAccess_ is in control. This also means that the _Business_ project has to have a dependency on the _DataAccess_ project for the interface, but not the implementation. Which will be passed to it via injection.

The whole point of injection is that the _Business_ project knows nothing of the _DataAccess_ project. Because there is a dependency, the _Business_ project indirectly takes a dependency on the underlying assemblies used by the _DataAccess_ project. So it knows which data access solution is chosen (SQL, Azure, files). This is the scenario inversion of control is trying to solve.

If the perfect situation was achieved, only then the startup project would know everything. Say there is a web project that will run the whole application, then that will know to load the Business project and to pass the correct repository implementation to it's classes. That implementation can come from the _DataAccess_ project, but it could also come from an _Experimental_ project if I want to try out a new data source.

Now that's harder because the _Experimental_ project would need to take a dependency on the _DataAccess_ project for the interface and thus take dependencies on _DataAccess_ data providers. Later, the _StartUp_ project would pass the _Experimental_ implementation to the _Business_ project. This means that the _Business_ project knows about the _DataAccess_ project for no reason, besides an interface. This makes maintenance and changing the projects over time more difficult.

NDepend graphs made this very easy to spot in smaller projects. A lot of arrows point to the _DataAccess_ class. Which means a lot of projects need to know about the data access. This automatically triggered warnings in my head that something was wrong.

Another nice overview the graph provides is namespace dependency overview. I opened GRAPH > View Application Assemblies Only and GRAPH > View Assembly Namespaces Only. This gives me a nice overview of all the namespaces in the project and this allowed me to spot the interface in the _DataAccess._

![namespace-graph-simple.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381091540/UGATRBilx.png)

In a larger project, such as the one that was featured heavily in the previous post about the dashboard, I found the View Assemblies and View Namespaces in the Graph menu a little too much information in graph form. It fills the screen with innumerable balloons. There is very little overview. This is convenient when I want to investigate a specific part of an application or when I already know where to start looking.

![dependency graph complex](https://kenbonnyblog.files.wordpress.com/2018/06/dependency-graph-complex.png)

![namespace graph complex](https://kenbonnyblog.files.wordpress.com/2018/06/namespace-graph-complex.png)

There is a nice feature to make this more easily consumable. When I right click on a node, I can choose to filter out all non used nodes. This makes the overview a lot more concise, but it sacrifices the overview of all items. Then again, in larger solutions, this makes the graph much more easy to navigate because it removes unnecessary clutter. It does require you to already know where to start looking. In the overview it was easy to spot that _DataAccess_ was used incorrectly, now I'd have to start investigating the central assemblies myself.

![namespace-graph-complex-zoom.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381093578/2NemBvGHs.png)

Bonus points for NDepend, in the informational box that pops up, it indicates that large solutions are better investigated via the Dependency Matrix overview. I'll take a look at that in the next blog post.
