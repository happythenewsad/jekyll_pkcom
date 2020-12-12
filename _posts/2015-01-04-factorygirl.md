---
title: Using FactoryGirl factories in Rails development fixtures
redirect_from:
  - /posts/using-factorygirl-factories-in-rails-development-fixtures/
---

Development fixtures (what guides.rubyonrails.org refers to as ‘seed data’ ) tend to balloon and unDRY fairly quickly in a large application. Using FactoryGirl factories for fixtures can eliminate the clutter because you only need to specify model attributes when they need to diverge from your factory template defaults.
I decided to try this in a brand new Rails 4.1.5 app this week. After defining my ‘Post’ factory in spec/factories/post.rb, I encountered an error while adding

    FactoryGirl.create(:post)

to seed.rb:

```ruby
NoMethodError: undefined method `delete' for nil:NilClass
    from /Users/dev/.rbenv/versions/2.1.1/lib/ruby/gems/2.1.0/gems/activerecord-4.1.5/lib/active_record/attribute_methods/write.rb:81:in `write_attribute_with_type_cast'
```

Turns out I had overriden Post#initialize without calling ‘super’, so my Post instance never inherited a 'delete' method. FactoryGirl assumes the existence of a 'delete' method on ActiveRecord instances it tries to create. Hope this will save someone a few minutes of time, as the error message doesn’t obviously point to this issue.

Environment:

rails (4.1.5)
factory_girl_rails (4.5.0)