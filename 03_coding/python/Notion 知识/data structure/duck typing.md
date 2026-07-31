---
title: "duck typing"
publish: false
tags: ["Python"]
---
# duck typing

鸭子模型是Python中的一种编程哲学，也被称为“[鸭子类型](https://www.zhihu.com/search?q=%E9%B8%AD%E5%AD%90%E7%B1%BB%E5%9E%8B&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A%223408399909%22%7D)”。它来源于一句话：“如果它走起路来像鸭子，叫起来也像鸭子，那么它就是鸭子。”这个哲学思想强调对象的行为比其具体类型更重要。与C++、Java等[编译型语言](https://www.zhihu.com/search?q=%E7%BC%96%E8%AF%91%E5%9E%8B%E8%AF%AD%E8%A8%80&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A%223408399909%22%7D)不一样的是，Python作为解释器语言，其语言层面的设计理念有独特之处，鸭子模型便是其中之一。在面向对象的世界中，编译型语言判断一个对象是否隶属于某个类，依靠的是类的继承机制，换句话说，即使一个对象实现了某个类的所有方法也不行；而在Python中，只要实例对象实现了某个类的所有必要的方法，即使不存在继承关系，也可以看作是这个类。

![](https://pica.zhimg.com/80/v2-e232adcdd299a969fb72b8246db7b3ac_720w.webp)

鸭子模型的简单实例

运行一下程序，结果如下：

![](https://picx.zhimg.com/80/v2-35c69ddefaccafcdf5916bc340ec3329_720w.webp)

鸭子模型的运行结果

我们来分析下这个简单示例，首先定义了[Duck类](https://www.zhihu.com/search?q=Duck%E7%B1%BB&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A%223408399909%22%7D)和Bird类，分别具有walk方法和[swim方法](https://www.zhihu.com/search?q=swim%E6%96%B9%E6%B3%95&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A%223408399909%22%7D)，两者并不具备任何继承关系；然后定义一个[duck_action函数](https://www.zhihu.com/search?q=duck_action%E5%87%BD%E6%95%B0&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A%223408399909%22%7D)，该函数接收一个对象，在函数内调用该对象的walk方法和swim方法；最后实例化一个Duck和Bird对象，先后传递给duck_action函数来执行，并都可以执行成功。在这个例子里，我们可以引入一个"协议"的概念，"协议"代表一系列特征，譬如鸭子协议是walk和swim，任何实现了鸭子协议的对象都可以当作鸭子，这个就是鸭子模型和协议。

---

再举个Python中更加常见的案例："with"，上下文管理器。众所周知，在任何编程语言里面，文件处理都是基本的IO操作，都包含open和close两个操作，因为这两个操作是[操作系统](https://www.zhihu.com/search?q=%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A%223408399909%22%7D)要求的。在操作系统里，每当打开一个文件，获取一个[文件句柄](https://www.zhihu.com/search?q=%E6%96%87%E4%BB%B6%E5%8F%A5%E6%9F%84&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A%223408399909%22%7D)后就会占用操作系统的资源，所以操作系统要求文件处理完后需要应用去close，释放掉文件句柄资源。然而现实情况是，开发者往往忘记close，最终导致资源泄漏问题频发。为了解决这个问题，Python使用with来进行上下文管理，在with的语句块结束后自动close，不再需要手动操作了，效率蹭蹭地上去。上下文管理器示例如下：

![](https://picx.zhimg.com/80/v2-692bf3f3096886929c76ba24871fd5ad_720w.webp)

在上面这个例子中，在with语句中进行open操作，然后调用read方法，最后并没有显示调用close，但不存在资源释放问题，因此with结束后会自动调用了close。那么with的底层原理是什么？类比鸭子模型介绍中关于协议的概念：

> 协议（Protocol）则是一种约定或契约，描述了对象应该具有的方法和属性。在Python中，协议是一种非正式的接口定义方式，它没有严格的语法要求，只需确保对象实现了协议中定义的方法和属性即可。协议允许我们根据对象的行为来定义接口，而不依赖于具体的类或类型。使用协议，我们可以通过定义一个适当的接口来描述对象的行为，而不仅仅依赖于继承关系。这样，不同的对象可以来自不同的类，但只要它们实现了相同的协议，我们就可以在代码中使用它们。
> 
> 
> [可迭代协议](https://www.zhihu.com/search?q=%E5%8F%AF%E8%BF%AD%E4%BB%A3%E5%8D%8F%E8%AE%AE&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A%223408399909%22%7D)
> 
> [可调用协议](https://www.zhihu.com/search?q=%E5%8F%AF%E8%B0%83%E7%94%A8%E5%8D%8F%E8%AE%AE&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A%223408399909%22%7D)
> 
> [上下文管理器协议](https://www.zhihu.com/search?q=%E4%B8%8A%E4%B8%8B%E6%96%87%E7%AE%A1%E7%90%86%E5%99%A8%E5%8D%8F%E8%AE%AE&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A%223408399909%22%7D)
> 

我们回到刚才的示例中，在with中对应的是实现上下文管理器协议__enter__方法和__exit__方法，换句话说，只要一个类具备__enter__方法和__exit__方法，就可以使用with管理，with开始时调用__enter__方法，结束时自动调用__exit__方法，示例代码如下：

![](https://pic4.zhimg.com/80/v2-5b4afbfff9f75af2ddf32d77153ff86d_720w.webp)

自定义具备__enter__方法和__exit__方法的Person类

运行下，查看结果：

![](https://pic1.zhimg.com/80/v2-ba92d5b94cc030731efd7c12b52d3d4c_720w.webp)

运行结果

在这个例子中，我们可以看到没有显示调用Person的__enter__方法和__exit__方法，由with自动调用了，因此对于资源释放类，资源释放的操作可以放到__exit__方法中，这样配合with语句使用会方便很多，也降低出错的概率。

总结一下，在Python中，鸭子模型指的是我们关注对象的行为（方法和属性）而不是其具体的类型。如果一个对象具有我们所期望的行为，我们就可以将其视为满足我们的需求，而无需关注其实际的类型。这种灵活性使得在Python中编写可重用和灵活的代码变得更加容易。例如，如果我们编写了一个需要迭代对象的函数，我们只关心对象是否具有`__iter__()`方法，而不关心它是否是一个具体的列表、[元组](https://www.zhihu.com/search?q=%E5%85%83%E7%BB%84&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A%223408399909%22%7D)或集合。

总之，鸭子模型让我们专注于对象的行为而不是其具体类型，在**编写灵活、可重用**的代码时非常有用。
