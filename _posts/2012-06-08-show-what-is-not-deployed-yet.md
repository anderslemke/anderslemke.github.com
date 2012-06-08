---
layout: post
tags : [heroku, git, github, bash, scripts]
---
{% include JB/setup %}

I continously find myself in a situation where I want to know if a specific feature, that I know is committed in master, is deployed to Heroku.

Basically, just open a compare view on Github that shows the difference between what is currently running on Heroku, and the current master.

To do this quickly from a shell (Mac OS), I do this

    cd [the directory that holds your project];
    set `git ls-remote heroku-production`;
    open "https://github.com/[your_github_user]/[the_github_project]/compare/$1...master";

(Remember to change the stuff in the brackets.)

You then get i nice view that shows exactly what you need:

![Compare view on Github](/assets/images/compare_view_on_github.png)

Awesome, right?