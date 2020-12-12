---
title: custom Rake tasks in Rails
redirect_from:
  - /posts/custom-rake-tasks-in-rails/
---

Jason Seifer has a pithy, informative rake tutorial here, but one thing that tripped me up when I was building custom rake tasks recently is that you need to explicitly load the Rails environment into your task; simply sticking a task in a .rake file in your Rails app tree is not sufficient.


For example, here is a simplified version of a rake task I wrote for Tweetful:

	task :load do
	  tweets = Twitter.usertimeline("happythenewsad", :count => 199) tweets.each do |status|
		Post.create! do |post|
		  post.creator = status.user.screenname
		  post.content = status.text
		end
	  end
	end

This code will produce the following error when executed:

	rake aborted! uninitialized constant Object::Post

Experienced Rails users will quickly infer that post.rb is probably not in the load path, which is the case. Here is one quick edit that causes the above code to execute in the Rails environment, which will probably default to "development":

	task :load => :environment do 