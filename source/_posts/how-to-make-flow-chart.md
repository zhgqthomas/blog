---
title: 业务流程图制作三部曲
date: 2017-01-12 13:56:36
categories:
  	- 项目管理
tags:
	- 产品需求
---
本文整理自[此处](http://www.woshipm.com/pd/3795.html)，如有侵权，请告知。

制作业务流程图之前，请回答清楚以下几个问题，否则不要开始绘制流程图:
- 整个流程的起始点是什么？整个流程的终结点是什么？
- 在整个流程中，涉及到的角色都是谁？
- 在整个流程中，都需要做什么事情？（可以是一个会议，可以是一个任务）
- 这些会议和任务是可选还是必选的？
- 分别产出什么文档？

业务流程图的梳理，有两种:
- 一种是基于现实发生的业务流程如实反映。这显然不是你一个团队能够 YY 的结果。更需要走到现实环境中，去调研，去梳理，去确认。
- 另一种是基于流程优化的方案，当你已经掌握了目前的流程现实如何运作时，基于分析，讨论，能够判断出流程中不合理的地方，给出一个更完善或者有更效率、成本更低的新的流程出来——或许你要求增加一个部门，或者你需要删减一个环节，或者中间的若干步使用新开发的系统去取代。

大多数时候，你要想做第二种流程图，必然要先将第一种给梳理出来。所以，第一种如实反映的流程图是躲不过的。既然如此，基于 YY 或者头脑风暴是不现实的。我们需要走到前线去，掌握现实中业务是如何运作的。而且很多时候，越细节越好。
那怎么做呢？基于有限的知识与经验，我可以给如下建议：
1. 调研
2. 梳理呈现
3. 评审确认

三部曲，如图所示

![](http://upload-images.jianshu.io/upload_images/854027-f09deb728376f932.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<!-- more -->

### 调研：问正确的问题，多问问题，多问几个人
除了在本部分开始的那几个问题要顾及到，其实调研过程解决的仍然是who，what，why，how，以及 where 的问题：谁，在什么情况下，做了什么事情，这个事情需要什么前置条件，又输出了什么，这个事情在哪里完成的？搞明白这几个问题，我们的调研就可以圆满完成了。

流程图的表现，要回答这几个问题:
- Who——谁？部门，角色，岗位
- What——什么事情？
- Where——在哪里做的？在我梳理的业务流程图上，where 更多表示是文档还是各种系统，用来表示信息化的程度。比如当我们梳理中发现，有一项登记，是用 excel 而不是业务系统来进行的，那么在这里的 where 就可以表示为：excel文档。
- Document——那产生的这份文档叫什么名字？也写出来，代表有文件的传递，而以后要进行信息化的话，此份人肉文档也是需要被消除而被系统取代的。（相反，如果这项工作是在某个系统里操作的，where 就可以写成“人事系统”，文档可以继续存在，即该系统中的表单名称：“员工登记表单”）
- Condition——条件。在这种条件下，下一个活动还能够继续，即用逻辑链接线的方式来表示一项活动的输入和输出，指向某个活动的箭头就表示此活动的前置输入条件。
- Dicision——决策。有些活动会产生一个条件判断，根据不同的判断结果从而走不同的分支流程。比如输入员工信息的时候，可以根据员工之前是否就职过，选择不同的流程，对于已经就职过的，选用之前的工号而不用生成新的工号。

![](http://upload-images.jianshu.io/upload_images/854027-ece707cc2ccca6ff.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 梳理并呈现
你的调研和观察是你拥有了“烹饪”所需的原材料
- 角色：部门、岗位或人
- 活动：做了什么事情
- 次序：做这些事情的次序如何
- 规则：什么情况下到什么事情

![](http://upload-images.jianshu.io/upload_images/854027-7a01d587d3480904.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来的任务是不是很简单，对，就像填空题一样简单。将活动/事件按照一定的规则填到由部门和时间两条维度决定的框框里。

### 复杂流程的分解
不可能将所有的活动都放到一张图里呈现。

> 业务流程是有层次性的，这种层次体现在由上至下、由整体到部分、由宏观到微观、由抽象到具体的逻辑关系。这样一个层次关系符合人们的思维习惯，有利于企业业务模型的建立  企业部门之间的层次关系表。一般来说，我们可以先建立主要业务流程的总体运行过程（其中包括了整个企业的大的战略），然后对其中的每项活动进行细化，落实到各个部门的业务过程，建立相对独立的子业务流程以及为其服务的辅助业务流程。”  ——引自《百度百科》 业务流程词条

对于很多新人来讲，业务最难的在于划分业务流程图的层次上。

**首先，明确你要梳理的业务流程的范围**——用大的粗略的关键节点，讲清楚这个业务流程范围中的故事，就是顶层业务流程图。你的顶层业务流程图是业务全局故事的简单表达，但是请注意这里的业务全局不见得是公司整体的业务全局，而是你界定好的业务范围。

比如，下图是餐厅的日常运作流程图，若你界定的业务范围是面向顾客的点餐和结帐流程，那么这就是顶层业务流程图。但是若你界定的是整个餐厅的运作业务流程，那这显然还是一个子集——并没有包含餐厅的采购、供应商管理、一级库存管理等工作。

![](http://upload-images.jianshu.io/upload_images/854027-92cccf7b942e7fe0.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其次，先从顶层的业务流程分解开始，由粗至细。顶层业务流程图的梳理原则：
1. 界定范围内的业务全局故事。
2. 包含该范围内的关键节点。并且，当被质疑说某某环节怎么不存在时，自己要清楚它在下一层分解中应该被包含在那个关键节点中。比如，赠送 10  周年优惠券，应该会在结帐节点分解中出现。而打印分单，会在点菜节点中分解。而准备儿童座椅应该是接待入座环节。
3. 顶层流程图分解出来的关键节点未必都会细化分解下去，生成二级以及三级的流程图。这要看该节点涉及到的“活动”以及“角色”是否复杂。

### 流程图的常用图示

![](http://upload-images.jianshu.io/upload_images/854027-789e8117059f7f96.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

常用的就是前两行的“活动”，“判断”，“逻辑关系线”，“起始与终止”，以及第二行的“子流程”，和“文件/表单”。如果你不是符号控，我建议这几个就足够了。

其中，“子流程”此图示就是可以帮助你将流程分解得到的子流程能够串联起来，比如，当在”A流程”中涉及到进一步需要分解的”A1.1流程”时，就可以在”A流程”中用子流程符号代表“A1.1”。然后你的读者就会明白要想进一步了解”A1.1″应该参考另外一个流程图。

流程图的常用结构：

![](http://upload-images.jianshu.io/upload_images/854027-c202668594f67e74.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

案例，基本上包含大多数图示的流程图：

![](http://upload-images.jianshu.io/upload_images/854027-dc001446920bc98a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 泳道图精要

![](http://upload-images.jianshu.io/upload_images/854027-fb0e767a6f7ab351.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**2大维度：一般泳道图的横向会作为部门或岗位维，当然也有例外，如上述案例中就是横的泳道。而纵向则做为阶段维——时间是从上到下发展的。如果复杂的泳道图，在任务分解上可以在阶段维里做一些划分，比如“采购”，“生产”，“销售”，”配送”等。**

**活动流转：活动就像一个游泳员一样，游到不同的泳道中去执行任务。**

### 业务流程图的注意事项

Do:
1. **让涉众参与，不要闭门造车**：业务流程图包含了你图上的各个参与角色代表，与他们适时确认事情的原本流程，禁止自己 YY。
2. **恰当的层次分解，不要将所有都铺到一张图上**：如上所示。
3. **逐渐深入，先抓枝干**：切忌胡子眉毛一把抓。
4. **流程一定有开始和结束**：切忌交付出来的流程图，让读者还来问你：流程的开始点是什么？用清晰的代表开始和结束的符号来完成第一步和最后一步。
5. **编号，编号，编号**：这是让沟通效率更高的优化措施。当你有了编号系统，相当于对你的流程图都赋予了唯一识别身份证号。这比中文名称更有效。比如当我们完成了业务流程图后，负责业务流程规则审核和优化的部门能够清楚在邮件里传达：H5.1 流程优化，大家就更明确指的是什么。

Do Not:
1. 自己 YY 应用的环节而不是现实中的环节
2. 所有的环节都试图放到一张图上
3. 一开始就陷入细节，胡子眉毛一起抓
4. 流程很难让人分清楚从哪里开始，到哪里结束

## 评审及后续行动
验证你是否做到了以上的 DO，以及规避了 Do not 的做法是什么？

很好办，及时与各位进行评审。将各个涉众都叫到一起，给他们看你梳理出来的成果。

这会发现一些有意思的事情，除了评审你的流程图是否符合现实外，也会评审目前的业务流程是否符合理想。不同的部门和岗位的代表会在这个评审中，确认当前，也会相互提出意见，甚至吵起来，这不失于做流程优化的一个很好的契机。暂且不表了。