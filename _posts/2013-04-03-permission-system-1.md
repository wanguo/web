---
layout: post
title: "权限系统设计与实现1-基本理解"
date: 2013-04-03 15:31
category: "Tech"
tags: [权限]
---

本文简单介绍权限系统设计中涉及到的基本概念：Subject、Operation和Resource，以及实现一个通用权限系统的基本思路。

在系统中发生的事情，可以总结为某个主体（Subject）在某个资源（Resource）下做了某个操作（Operation），如下图所示：
![permisstion](/assets/images/p.png)

所谓的权限系统，就是在这个过程中加入一些限制条件，只有满足了相应的条件才能进行下一步操作。 根据限制条件的粒度不同，我们可以把权限判断分为三层：认证（Authentication）、授权（Authoration）和详细控制（DetailControl）。 

1,Authentication:最粗粒度的控制，不管要进行何种操作，首先必须知道当前操作的Subject是谁。反应到系统中就是登陆的过程，只有登陆的用户才能使用某些功能。判断函数这样就可以了：boolean isAllowed(subject); 

2,Authoration:稍微复杂的控制，Subject只能进行被允许的Operation，需要了解Operation的细节，只要被允许的Operation，就不管是针对何种Resource，都可以进行操作。判断函数：boolean isAllowed(subject,operation); 

3,DetailControl:较复杂的控制，Subject在被允许的Operation基础上，只能在特定的Resource上进行操作，需要了解Resource的细节。判断函数为：boolean isAllowed(subject,operation,resource);

最简单的情况，权限数据是一张二维矩阵。

OperationA:

 	ResourceA	ResourceB
	SubjectA	1	0
	SubjectB	0	1 

当有多个Operation时，权限数据也可以相应叠加。 

OperationA,OperationB:

 	ResourceA	ResourceB
	SubjectA	10	01
	SubjectB	01	10 

如果Subject和Resource特别多，权限矩阵数据就会急剧膨胀，数据量是M*N。

因此我们引入角色Role的概念，将矩阵分解：

OperationA,OperationB:

 	ResourceA	ResourceB
	RoleA	10	01
	RoleB	01	10

其中RoleA和RoleB分别包括了多个Subject。数据量变为了M*R+R*N。如果R的数量远小于Subject和Resource，则实现简化。这就是经典的RBAC：基于角色的访问控制（Role based access control）。这样做还有一个额外的好处就是在还没有任何用户的时候，系统仍然可以表达部分权限信息。可以说角色在权限系统中是Subject的代理。 

权限控制可以看作一个filter模式的应用, 这也符合AOP思想的应用条件。理论上我们只需要在所有需要做权限控制的地方加上isAllowed(subject,operation,resource);函数判断就可以了，不过具体实现的时候仍然会遇到一些细节上的问题，关键在于很难在细粒度上决定权限控制的规则，比如是否需要考虑权限规则的扩展性，Subject是否设置类型、等级区分（比如你希望VIP用户可以直接拥有低级权限），Operation之间是相互独立（独立的话就会带来一个量的问题，上面的矩阵就会等比例增长）还是继承关系（组织成一个树形结构），Resource是否需要增加过滤、分级或者其他附属属性。而在和程序的结合的时候，AOP虽然使得我们可以扩展任何函数，但这种扩展依赖于Cutpoint所能取得的信息，因此会非常依赖函数的良好设计，比如所有函数都传入Operation和Resource所需要的信息。
 
很多时候，将Subject、Operation、Resource分解为非常复杂的结构效果未必理想，很多时候只是个管理模式的问题，应该尽量通过重新设计权限空间（既进行局部化设计，在一系列权限验证后进入权限空间，在其中进行操作就不需要任何验证了）的结构来加以规避。不过在一些非常复杂的权限控制环境下，也许简单的描述信息确实很难有效的表达权限策略，此时尝试一下规则引擎可能比在权限系统中强行塞入越来越多的约束要好的多。

