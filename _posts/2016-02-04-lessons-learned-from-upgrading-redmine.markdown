---
title: Lessons Learned from Upgrading Redmine
date: 2016-02-04T01:02:02+00:00
categories: redmine upgrade
layout: post
---

Upgrading Redmine from `1.x` to `3.x` was a lot smoother than one would imagine.  Kudos to the Redmine team for making that possible.

However, there were some hiccups (mainly due to our own customizations) so I'll capture them here for posterity.

Unknown Admin
-------------
Our Redmine admin account's password has long since been lost, but luckily there is a way to reset that password:
[Redmine FAQ#Reset-password...](http://www.redmine.org/projects/redmine/wiki/FAQ#Reset-password-lost-without-admin-redmine-account-but-with-admin-redmine-database-account)

The reason this is necessary is because we migrated our CAS plugin from [Emergya's redmine_cas](https://github.com/Emergya/redmine_cas) to [nine.ch 's redmine_cas](https://github.com/ninech/redmine_cas).  This name conflict led to quite a bit of confusion.  I am still not 100% sure that the original plugin was [Emergya's](https://github.com/Emergya) version.

Cookie Overflow
---------------
Certain users experienced `500` errors because they were approved for too many applications.  This is because the plugin tries to shove all of the CAS attributes into the session, including every single CAS group the user belongs to.  By default, Redmine/Rails uses cookies for sessions.

To resolve the cookie overflow error, configure Redmine/Rails to use database sessions:
In `config/application.rb`, replace:
{% highlight ruby %}
config.session_store :cookie_store,
{% endhighlight %}
with
{% highlight ruby %}
config.session_store :active_record_store,
{% endhighlight %}

and then integrating the [Active Record Session](https://github.com/rails/activerecord-session_store) gem.

Disappearing Files
------------------
Redmine stores file attachments on the filesystem under the [`/{install directory}/files/` folder](https://github.com/redmine/redmine/tree/2a6637ed2ef05ad0770bb76094c67f230d2341f6/files) by [default](https://github.com/redmine/redmine/blob/2a6637ed2ef05ad0770bb76094c67f230d2341f6/config/configuration.yml.example#L69-L76).

So we have to ensure those files persist across deployments.

Mismatched JRuby
----------------
During the deployment, we encountered an issue where bundler could not find the gems that were in the deploy package.  The reason was because the application was using a different JRuby version than the operating system.

The operating system's JRuby was used to install the gems using this command:
{% highlight shell %}
jruby -S bundle install --deployment
{% endhighlight %}
The command left a `.bundle/config` file that was unintentionally picked up by the application.  

The OS JRuby installed the gems to it's location, `vendor/gems/jruby-x.x` while the application was looking for the gems in it's own location, `vendor/gems/jruby-y.y`.

The reason the application used a different version of JRuby, was because warbler packaged that version with the application.  We will need to wait for warbler to be upgraded before both the application and OS JRuby can match.

Database Mirage
---------------
In our staging environment, I was trying to check the database migrations with:
{% highlight shell %}
jruby -S bundle exec rake db:migrate:status
{% endhighlight %}

but was getting a misleading access denied error.

It turns out that the `rake` task was trying to hit the development database, I had to run this instead:
{% highlight shell %}
set RAILS_ENV=production
jruby -S bundle exec rake db:migrate:status
{% endhighlight %}

New Secrets
-----------
Just a note that Rails is now expecting all secret tokens to be located in `config/secrets.yml`.  There were no issues involving this during the upgrade.

No Rails Console
---------------
Just another note.  I still am unable to get `rails console` to work on both staging and production.

