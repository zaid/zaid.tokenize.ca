---
layout: post
title: Prevent destruction of objects with associations using CanCan
---

Sometimes you want to prevent the destruction of objects with
associations for certain user roles and this is easy to do if you are
using [CanCan](https://github.com/imathis/octopress).

Let us assume that we have two user roles, regular users and
administrators. Regular users cannot delete objects with associations
but administrators can.

Our ability class will look similar to this:

{% highlight ruby lineos %}
class Ability
  include CanCan::Ability

  def initialize(user)
    user ||= User.new

    if user.is_admin?
      can :manage, :all
    else
      can [:create, :read, :update], [User, Post, Comment]
      can :destroy, [User, Post, Comment] do |obj|
        has_no_associations?(obj)
      end
    end
  end

  def has_no_associations?(obj)
    obj.class.reflect_on_all_associations.each do |association|

      # We're only interested in :has_many and :has_and_belongs_to_many associations
      next unless [:has_many, :has_and_belongs_to_many].include?(association.macro)
      return false unless obj.send(association.name).empty?
    end
    return true
  end

end
{% endhighlight %}

With the above, you can check if a user can destroy an object by doing
the following in a view template:

{% highlight ruby lineos %}
  <% if can?(:destroy, @post) %>
    <%= link_to 'Delete', post_path(@post), :method => :destroy %>
  <% end %>
{% endhighlight %}

And last, make sure that you have
authorize_resource/load_and_authorize_resource in your controllers (or
manually call authorize!(:destroy, @some_obj) before calling destroy on
the object.
