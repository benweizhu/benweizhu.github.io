---
layout: post
title: "当whenever遇到capistrano和rbenv - Linux下的cron job"
date: 2015-09-27 11:01:41 +0800
comments: true
categories: 
---
当你遇到这样一个需求：用户订阅了很多信息，服务器需要每周，或者每月给用户发送邮件，告知用户订阅信息的更新内容。你需要怎么做？

在RoR的环境下，发送邮件使用Rails Action Mailer。参考资料：http://guides.ruby-china.org/action_mailer_basics.html 。

那么如何定时呢？答案是Cron Job。

如果大家谷歌rails cron job，就可以得到答案，一个是关于Cron Job的答案， http://www.gotealeaf.com/blog/cron-jobs-and-rails ，一个是whener gem， https://github.com/javan/whenever 。


Cron是*nux系统中一个基于时间的任务安排软件。通过crontab 命令，我们可以在固定的间隔时间执行指定的系统指令或 shell script脚本。时间间隔的单位可以是分钟、小时、日、月、周及以上的任意组合。

更多关于crontab定时任务，可以查看： http://linuxtools-rst.readthedocs.org/zh_CN/latest/tool/crontab.html

关于时间的定义：

{% codeblock lang:ruby %}
# +---------------- minute (0 - 59)
# |  +------------- hour (0 - 23)
# |  |  +---------- day of month (1 - 31)
# |  |  |  +------- month (1 - 12)
# |  |  |  |  +---- day of week (0 - 6) (Sunday=0)
# |  |  |  |  |
  *  *  *  *  *  command to be executed
{% endcodeblock %}

这里，我就不细谈crontab定时任务，大家可以去看上面那个链接。

whenever是一个Ruby的gem，它可以提供一个清晰的语法来编写定时任务。

如果你刚才有停下来看crontab的内容，你会发现crontab任务的语法非常的复杂。

whenever的目的就是为了让你以ruby的语法来编写要执行的任务，再由它来生成真正的crontab任务。


如下：

{% codeblock lang:ruby %}
set :environment, "staging"
set :output, {:error => "log/cron_error_log.log", :standard => "log/cron_log.log"}

weekly_of_day = :sunday # every sunday
weekly_of_time = {:at => '0:00 am'} # every sunday 0:00am

monthly = '0 0 1 * *' # monthly 0 0 1 * * (every first day of month at 0:00am)

quarterly = '0 0 1 1,4,7,10 *' # quarterly 0 0 1 1,4,7,10 * (every 3 months at 0:00am start from the first day in january)

job_type :rbenv_rake, %Q{export PATH=~/.rbenv/shims:~/.rbenv/bin:/usr/bin:$PATH; eval "$(rbenv init -)"; cd :path && RAILS_ENV=staging bundle exec rake :task --silent >> log/cron_log.log 2>> log/cron_error_log.log}

every weekly_of_day, weekly_of_time do
  rbenv_rake "email_notification:send_email_notification_weekly"
end

every monthly do
  rbenv_rake "email_notification:send_email_notification_monthly"
end
{% endcodeblock %}

更多关于如何使用whenever的内容，请查看： https://github.com/javan/whenever

whenever提供提供了四种job类型，如下：

{% codeblock lang:ruby %}
job_type :command, ":task :output"
job_type :rake,    "cd :path && :environment_variable=:environment bundle exec rake :task --silent :output"
job_type :runner,  "cd :path && bin/rails runner -e :environment ':task' :output"
job_type :script,  "cd :path && :environment_variable=:environment bundle exec script/:task :output"
{% endcodeblock %}

但是，你在我给的例子中可以看到，我并没有使用其中的任何一个，而是自定义了一个rbenv_rake，这便是接下来的重点。

如果你产品环境的Ruby环境是通过rbenv配置，那么在你使用crontab任务的可能会遇到这个问题。

{% codeblock lang:ruby %}
../spec_set.rb:92:in `block in materialize': Could not find rake-10.4.2 in any of the sources (Bundler::GemNotFound)
{% endcodeblock %}

关于原因，可以查看这个： http://benscheirman.com/2013/12/using-rbenv-in-cron-jobs/

Crontab运行在一个受限的环境下，所以.bash_profile的配置方式，并不起作用。需要在每次运行该任务前，重新初始化一次rbenv。

最后一点：

关于whenever和capistrano的集成，其实官方网站上也给出了，但是测试过不起作用，至少我这没起作用，所以我在部署的ruby文件中添加了一个task。

{% codeblock lang:ruby %}
namespace :deploy do

  desc "Update crontab with whenever"
  task :update_cron do
    on roles(:app) do
      within current_path do
        execute :bundle, :exec, "whenever --update-crontab #{fetch(:application)}"
      end
    end
  end

  after :finishing, 'deploy:update_cron'
end
{% endcodeblock %}


