---
layout: post
title: Testing an observer with DelayedJob and RSpec
---

On one of the projects that I am working on, there was an after_create
controller filter which I wanted to refactor into an observer. The
actual refactoring was pretty painless but testing it proved to be
more work than I thought.

The first thing that I noticed was that observer callbacks were not actually
called when running my tests, which is good since it means that my
regular tests weren't affected.

The observer is for a comment model which basically notifies the owner
of the commentable object that a new comment was posted.

To add support for creator/updater fields, I am using the
[Userstamp](https://github.com/delynn/userstamp) gem.

For the purpose of explaining how the test goes, lets assume that the
observer looks like the following:

{% highlight ruby lineos %}
class CommentObserver < ActiveRecord::Observer
  def after_create(comment)
    recipient = comment.commentable.creator
    UserMailer.delay.send_comment_notification_email(recipient.email,
comment.id) unless recipient == comment.creator
  end
end
{% endhighlight %}

I created a mailer_macros.rb file and included it in spec_herlp.rb. The file contains the following:
{% highlight ruby lineos %}
module MailerMacros
  def last_email
    ActionMailer::Base.deliveries.last
  end

  def reset_emails
    ActionMailer::Base.deliveries = []
  end
end
{% endhighlight %}

The first test that we'd like to do is to test that no emails will get
sent if the commentable creator is the same user as the comment
creator.
The second test will be to use different users to create the
comment and commentable objects and checking to see if the commentable creator will be the
recipient of the notification email.

{% highlight ruby lineos %}
require 'spec_helper'

describe CommentObserver do
  before(:each) do
    @user = FactoryGirl.create(:user)
    User.stamper = @user

    reset_emails
  end

  describe "comments on posts" do

    context "when the same user creates the post and comment" do
      it "should not email anyone" do
        post = FactoryGirl.create(:post, :title => 'Hello world post!',
:user => @user)

        comment = post.comments.create!(:content => 'First comment!')

        observer = CommentObserver.instance
        observer.after_create(comment)

        # Execute queued job
        Delayed::Worker.new.work_off

        last_email.should be_nil
      end
    end

    context "when different users create the post and comment" do
      it "should email the post creator" do
        post = FactoryGirl.create(:post, :title => 'Another hello world
post!', :user => @user)

        User.stamper = FactoryGirl.create(:user)

        comment = post.comments.create!(:content => 'Another comment!')

        observer = CommentObserver.instance
        observer.after_create(comment)

        # Execute queued job
        Delayed::Worker.new.work_off

        last_email.to.should == [post.creator.email]
      end
    end
  end
end
{% endhighlight %}

That's pretty much the basics to start testing observers and delayed_job
queued items.
