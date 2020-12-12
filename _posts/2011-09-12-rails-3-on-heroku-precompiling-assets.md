---
title: Rails 3.1 on Heroku - precompiling assets
redirect_from:
  - /posts/rails-31-on-heroku-precompiling-assets/
---

I pushed a Rails 3.1 app to Heroku for the first time today (plugsandwifi.com). It ran smoothly in development mode, but I got a nasty 500 error in production:

	Completed 500 Internal Server Error in 75ms
	2011-09-11T19:34:20+00:00 app[web.1]:
	2011-09-11T19:34:20+00:00 app[web.1]: ActionView::Template::Error (logo.png isn't precompiled):

Logo.png is a simple image. Heroku attempts to precompile your assets (this is now a necessary step for whizzy things like sass and coffee script) if you haven't already as part of their one-step deploy process: http://devcenter.heroku.com/articles/rails31_heroku_cedar.


For some reason, the precompile process was failing for me, although Heroku's log should have acknowledged that issue when I deployed. To fix the problem I simply ran this command: ```RAILS_ENV=production bundle exec rake assets:precompile```. I then ```git add```'d the generated directory and deployed to Heroku. Note: For those following along with Heroku's instructions, Heroku did NOT acknowledge my pre-compiled assets in its printout.