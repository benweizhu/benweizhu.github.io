---
layout: post
title: "关于运行Active Record数据迁移"
date: 2015-09-10 08:51:45 +0800
comments: true
categories: 
---
Active Record数据库迁移是 Active Record提供的一个功能，按照时间顺序管理数据库模式。使用迁移，无需编写 SQL，使用简单的Ruby DSL就能修改数据表，对数据库的操作和所用的数据库种类无关。

你可以把每个迁移看做数据库的一个修订版本。数据库中一开始什么也没有，各个迁移会添加或删除数据表、字段或记录。Active Record知道如何按照时间线更新数据库，不管数据库现在的模式如何，都能更新到最新结构。同时，Active Record还会更新db/schema.rb文件，匹配最新的数据库结构。

##db:migrate

Rails提供了很多Rake任务，用来执行指定的迁移。

其中最常使用的是rake db:migrate，执行还没执行的迁移中的change或up方法。如果没有未运行的迁移，直接退出。rake db:migrate按照迁移文件名中时间戳顺序执行迁移。

注意，执行db:migrate时还会执行db:schema:dump，更新db/schema.rb文件，匹配数据库的结构。

##db:migrate VERSION

如果指定了版本，Active Record会运行该版本之前的所有迁移。版本就是迁移文件名前的数字部分。例如，要运行 20080906120000 这个迁移，可以执行下面的命令：

{% codeblock lang:ruby %}
rake db:migrate VERSION=20080906120000
{% endcodeblock %}

如果20080906120000比当前的版本高，上面的命令就会执行所有20080906120000之前（包括 20080906120000）的迁移中的change或up方法，但不会运行20080906120000之后的迁移。如果回滚迁移，则会执行 20080906120000之前（不包括20080906120000）的迁移中的down方法。

##db:rollback

还有一个常用的操作时回滚到之前的迁移。例如，迁移代码写错了，想纠正。我们无须查找迁移的版本号，直接执行下面的命令即可：

{% codeblock lang:ruby %}
rake db:rollback
{% endcodeblock %}

这个命令会回滚上一次迁移，撤销 change 方法中的操作，或者执行 down 方法。如果想撤销多个迁移，可以使用 STEP 参数：

{% codeblock lang:ruby %}
rake db:rollback STEP=3
{% endcodeblock %}

这个命令会撤销前三次迁移。

##db:redo

db:migrate:redo 命令可以回滚上一次迁移，然后再次执行迁移。和 db:rollback 一样，如果想重做多次迁移，可以使用 STEP 参数。例如：
{% codeblock lang:ruby %}
rake db:migrate:redo STEP=3
{% endcodeblock %}

这些 Rake 任务的作用和 db:migrate 一样，只是用起来更方便，因为无需查找特定的迁移版本号。

##db:migrate:up和db:migrate:down

如果想执行指定迁移，或者撤销指定迁移，可以使用db:migrate:up和db:migrate:down任务，指定相应的版本号，就会根据需求调用change、up或down方法。例如：

{% codeblock lang:ruby %}
rake db:migrate:up VERSION=20080906120000
{% endcodeblock %}

这个命令会执行20080906120000迁移中的change方法或up方法。db:migrate:up 首先会检测指定的迁移是否已经运行，如果Active Record任务已经执行，就不会做任何操作。

##修改现有的迁移

有时编写的迁移中可能有错误，如果已经运行了迁移，不能直接编辑迁移文件再运行迁移。Rails 认为这个迁移已经运行，所以执行 rake db:migrate 任务时什么也不会做。这种情况必须先回滚迁移（例如，执行 rake db:rollback 任务），编辑迁移文件后再执行 rake db:migrate 任务执行改正后的版本。

一般来说，直接修改现有的迁移不是个好主意。这么做会为你以及你的同事带来额外的工作量，如果这个迁移已经在生产服务器上运行过，还可能带来不必要的麻烦。你应该编写一个新的迁移，做所需的改动。编辑新生成还未纳入版本控制的迁移（或者更宽泛地说，还没有出现在开发设备之外），相对来说是安全的。

##在不同的环境中运行迁移

默认情况下，rake db:migrate 任务在 development 环境中执行。要在其他环境中运行迁移，执行命令时可以使用环境变量 RAILS_ENV 指定环境。例如，要在 test 环境中运行迁移，可以执行下面的命令：

$ rake db:migrate RAILS_ENV=test


参考资料：     
1.http://guides.ruby-china.org/active_record_migrations.html