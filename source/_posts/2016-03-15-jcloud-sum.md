title: 某超长文档的探索之路
subtitle: 加量不加价~ 比字数谁都赢不了我!
date: 2016-03-15 16:18:00
cover: //img.aotu.io/Littly/2016-03-15/title.png
categories: 项目总结
tags: 
 - jcloud
 - sum
author:
  nick: Littly
  github_name: Littly
---

所谓的文档, 大概就是指那种洋洋洒洒写了不下几万字, 最后没有多少人会看完的东西. 对, 比技术不一定, 但比字数谁都赢不了我! 

<!-- more -->

这次京东云改版的项目, 除了需要搭建一个放置京东云文档的平台, 还包括了文档录入的工作. 之前使用[**Hexo**](http://hexo.io)做过[**HaloJS**](http://aotu.io/halojs/)的文档平台, 在文档录入的时候并没有碰到太多问题. 本以为搭建文档页面也就仅此而已, 直到碰到了长达104页的doc文档...


### 搭建框架

不同于其他零散的页面, 文档页面是一整个系列的, 具有相同的页面结构. 页面头部, 底部, 导航等组件. 如果像平时制作静态页面那样每个页面都从一个空白的模板html文件开始下手, 在开发量达到一定程度的时候, 页面整体的维护就会非常困难. 首先是无法很痛快地进行某些公共模块的修改: 就算只是在底部加多一个链接, 或者在导航中删除一个条目, 我们都需要对每个页面都进行手动修改. 另外, 页面的资源管理会越来越难. 每个页面都需要引用整体页面的样式文件与脚本文件, 还都有页面私有的样式文件, 脚本文件, 还有图片文件等. 如果这些资源共用一个文件夹, 有时候就会面临文件命名的难题. 

如上面说, 文档页面可谓是重复工作最严重的一种页面. 好在搭建框架的小伙伴非常给力, 使用了同样很给力的开源的前端工程化开发工具[Athena](https://github.com/o2team/athena),自带支持代码热更新的预览服务, 自带压缩代码, 编译sass/less的工具, 自带上传sftp的工具等等. 而对于这个需求来说, 最好用的莫过于ejs模板语法的支持了. 有了模板, 公共的组件在不同页面都可以随意引用, 工作量减少了非常多. 

每个页面都由组件组成, 平时写页面的时候只需要点名引用需要的组件. 搭建完成后, 使用Athena工具的build命令, 就可以将页面的静态文件输出. 对于这批文档的页面, 每个页面会被拆分为头部组件, 底部组件, 导航组件. 如果开发过程中某个组件产生变化, 只需要直接修改组件, 再重新运行build命令, 就可以使每个引用了这个组件的页面都一起变过来, 可谓十分方便.

### 录入文档
录入文档可能是这一整个需求最痛苦的工作了. 一百多页A4纸大小的DOC文档, 还需要依据设计稿重新设置样式, 是十分机械化的工作. 在这里, 我的小伙伴将文档中的不同样式建立为不同的组件, 比如一级标题组件, 或是二级标题组件等等. 这些组件使用起来就像下面这样:

```html
<%= widget.load('doc_mod_list3', {
  title1: '操作指南',
  item1: ['挂载云硬盘','绑定公网IP','常见问题','主机数量','监控报警','删除云主机','绑定监控报警','重置密码','制作镜像'],
  title2: '常见问题',
  item2: ['主机数量','监控报警','删除云主机'],
  item3show: false
}) %>
```

组件本身是一个ejs模板:

```html
<ul class="doc_mod_list3">
  <li class="item1">
    <div class="doc_mod_list3_title"><%= title1 %></div>
    <span class="doc_mod_list3_link">
      <% for (var i=0;i<item1.length;i++) { %>
        <a href=""><%= item1[i] %></a>
      <% } %>
      <a href="" class="doc_mod_list3_lmore">更多&gt;&gt;</a>
    </span>
  </li>
  <li class="item2">
    <div class="doc_mod_list3_title"><%= title2 %></div>
    <span class="doc_mod_list3_link">
      <% for (var i=0;i<item2.length;i++) { %>
        <a href=""><%= item2[i] %></a>
      <% } %>
      <a href="" class="doc_mod_list3_lmore">更多&gt;&gt;</a>
    </span>
  </li>
  <% if(item3show == true) { %>
    <li class="item3">
      <div class="doc_mod_list3_title"><%= title3 %></div>
      <span class="doc_mod_list3_link">
        <% for (var i=0;i<item3.length;i++) { %>
          <a href=""><%= item3[i] %></a>
        <% } %>
        <a href="" class="doc_mod_list3_lmore">更多&gt;&gt;</a>
      </span>
    </li>
  <% } %>
</ul>
```

这样一来, 我们写页面的工作, 变成了写ejs组件, 再用widget.load函数加载自定义的组件, 用getCSS/getJS函数调用对应的css/js文件资源. 把页面中比较通用, 复杂的组件拆分出来, 有利于将页面模块化, 代码复用可以更爽快.

#### 思考
具体到这个项目, 文档录入是个非常痛苦的活. 按照目前的方案, 文档里面的一级标题, 二级标题, 正文, 图片等, 都是由组件构成, 像`<h3 class=“xxxx”>xxxx</h3>`这样的简单代码, 也被做成了组件. 个人认为这样的做法不太科学. 这种太过细小的组件反而使录入文档变成了十分痛苦的事情. 上面所说的情况, 一个标签可以解决的问题, 变成引用组件的工作后, 代码量并没有多少差别:

```html
<!-- 声明组件 -->
<h3 class="doc_mod_title3">
  <%= title %>
</h3>

<!-- 使用组件 -->
<%= widget.load('doc_mod_title3', {
  title: '京东云入门手册'
}) %>
```

左边是引用组件的写法, 右边是组件内部的代码. 可以看见, **过度模块化导致了细小组件的产生**. 这里应该有更加方便的做法. 

Python中有个第三方模块docx2html, 实现了将docx文件转化为html代码的功能, 并且转化完的代码十分纯净, 只有html结构与内容, 不带样式, 很适合在导出后再重置样式. 针对这个项目, 我们可以做个小改造, 每个页面依然由头部, 导航还有底部组成, 而文档内容则可以定义一个css类`doc_mod_content`, 表示其中是文档的内容, 而文档的各种样式, 则写在这个类的后代选择器上. 如果标签的种类无法满足区分特定节点的需要, 还可以通过编辑器的搜索替换功能去批量添加特定的类. 这样的话, 首先是少去了开发组件的工作, 文档的录入也会轻松很多. 遗憾的是, 由于时间紧迫, 这个需求并没有使用这种方式. 过后有机会可以尝试. 文档录入本来就该是个轻松的活. 

### 导航优化
对于普通的页面, 侧导航跟页面主体并不需要分开, 直接放置即可, 就像hexo的[文档页面](https://hexo.io/api/)一样. 但前面说过, 文档页是非常长的. 对于这种超长的页面, 使用上述的实现方式, 有可能会出现滚动到页面内容中间, 看不见导航的窘迫局面. 

![img](//img.aotu.io/Littly/2016-03-15/hexo.png)

看见这种情况, 产品的小伙伴就该找我们了. 重构哥哥, 我想让导航始终在视野里面. 你能实现不?
Of Course! 只需要把导航栏设置为position: fixed就ok了. 不过对于这种页面中同时还存在头部与底部的页面, 会比较麻烦. 跟交互的童鞋商量过后, 我们把方案确定了下来. 具体而言, 侧导航共有三种状态:

1. 导航条尚未滚动到页面顶部的状态, `position: static`
2. 导航条滚动到超出页面顶部的位置, `position: fixed`
3. 导航条底部与页面内容的底部对齐, `position: absolute; bottom: 0`,  或者使用JS控制导航条的底部对齐页面内容底部. 

再具体一点说, 这三种状态的触发条件分别是:
1. 页面滚动距离 < 导航距离顶部的距离
2. 页面滚动距离 > 导航距离顶部的距离 && 页面滚动距离 < 内容的高度 + 内容距离顶部的距离 - 导航条的高度 + 导航条的滚动距离
3. 剩下的情况. 

就算如此, 这样的实现还是会有一些问题存在, 比如在跳转到某个页面之后, 导航的高亮条目不一定会在视野中, 这会导致在不同页面间跳转的过程中, 导航条不固定, 找不到高亮条目的尴尬. 

![img](//img.aotu.io/Littly/2016-03-15/out_of_view.png)


在当前实现方式下, 并没有特别好的办法能解决这种情况. 不同页面的长度不是相同的, 导航的位置跟页面的位置关系处于上面三种状态中的哪一种也是难以预测的, 就更没办法将导航移入视野中了. 说到底, 这种情况都是由于页面和导航太长导致的. [腾讯云](http://www.qcloud.com/product/cvm.html)的文档页面也是这样处理的. 


#### 思考

那么, 在不缩短页面导航的情况下, 有没有办法做好这样的页面的效果呢? 答案是有的, 并且不止一种. 

首先, 有一种比较折中的方案(当然, 要先问问产品的小伙伴同不同意), 把导航中的高亮条目抽出来放到导航的第一位. 参考各种基于JSDoc3的文档页面. 下面是[PIXI](http://pixijs.github.io/docs/)的文档: 

![img](//img.aotu.io/Littly/2016-03-15/pixi.png)


放到项目中的话, 这种办法大有曲线救国的味道: 不惜修改几十个页面中的导航顺序来达成高亮条目的固定. 这种机械化的工作, 做起来已经很累人. 如果做完后产品的小伙伴对效果不满意…. 是的, 还得几十个页面一个一个的改回来. 当然, 我们也可以预先把老版本备份一下, 不满意了就直接覆盖文件完事. 这一切做起来是那么的麻烦. 但如果是搭配着[Athena](https://github.com/o2team/athena)工具使用, 就简单多了. 导航栏组件中用数组menu来储存导航栏条目信息, 只需要用forEach就可以输出所有的菜单项. 如果要将高亮条目与第一项对调, 只需要一句话就完事:

```js
  // var menu, curIdx;
  menu.unshift(menu.splice(curIdx/*高亮条目的索引*/, 1)[0]);
  // menu.forEach(function (val, idx) {...});
```

是不是有点爽?

上面这种方法, 说实话是有点不完美的. 用户当然不希望侧导航的位置一直不固定. 在这里, 我们可以使用页面局部刷新技术来规避这个问题. 如果能够在跳转到其他文档的时候不刷新页面, 而是在脚本中通过异步请求获取到新的文档的内容. 这个内容可以是json, 也可以是一个html文档, 甚至是一个html片段. 过后我们再将其用合适的姿势展示在右侧内容区. 结果上看, 页面仅仅是刷新了内容区, 导航位置不会改变, 比起之前的版本, 效率还会更高. 

### 感悟
这个项目做下来, 最后还是留下了一些遗憾的地方. 迫于时间, 感觉手头还有很多的技术没用上, 或者还有许多的方案没实现. 要做好一个项目, 还是需要从项目一开始就与产品, 交互, 视觉等业务上游团队保持良好的沟通, 对整个项目拥有相当的了解, 闭门造车不可取, **用户体验**才是王道. 