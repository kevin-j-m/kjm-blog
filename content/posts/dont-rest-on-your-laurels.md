+++
title = "Don't REST on your Laurels"
date = 2025-11-15T10:23:22-04:00
tags = ["ruby", "rails", "software-design"]
summary = "Cultivate a growth mindset without growing the size of your classes"
description = "On the benefits of using the default RESTful actions to prevent controllers from becoming too big and confusing."
+++

## RESTful Routes

You can find many resources [trumpeting the benefits](https://www.rubyevents.org/talks/in-relentless-pursuit-of-rest) of following RESTful routes in the context of a [Rails application](https://www.thegnar.com/blog/always-use-restful-routes). I want to focus on my personal favorite benefit:

Adherence to the default RESTful actions creates a constraint, and is a noticeable heuristic, that aids in limiting the surface area of classes.

## Planting an example

Let's say we have an application that tracks people's accomplishments. We call each instance of an accomplishment a laurel wreath. We love ourselves a metaphor.

```ruby
class LaurelWreathsController < ApplicationController
  def show
    @laurel_wreath = LaurelWreath.find(params[:id])
  end
end
```

This is one of the [mappings](https://guides.rubyonrails.org/routing.html#crud-verbs-and-actions) of HTTP verb and URL to controller action that Rails expects and provides out of the box.

## Growing branches

We have new functionality we want to support. Users can acknowledge that other people assisted in achieving their accomplishment. This publicly demonstrates the impact others had in a person's growth in the best possible way: by impersonally posting it on a website.

We already have a controller about these accomplishments. Right now it has the ability to display one. We can [build on that success](https://www.phrases.org.uk/meanings/rest-on-his-laurels.html) by using that controller to wire up this action. It *is* acting on the laurel wreath resource after all. It's a natural fit for it.

```ruby
class LaurelWreathsController < ApplicationController
  def branch_owner
    @laurel_wreath = LaurelWreath.find(params[:id])
    @laurel_wreath.branches << User.find(params[:branch_user_id])

    redirect_to laurel_wreath_path(@laurel_wreath)
  end
end
```

We're again continuing the metaphor. These people are branches on the tree that make up the wreath.

> The reason for that association takes us into the myth of Apollo’s love for the nymph Daphne, who turned into a Bay tree just as Apollo approached her (anything could happen if you were a Greek god). Undeterred, Apollo embraced the tree, cut off a branch to wear as a wreath and declared the plant sacred.

On the routing side, we add [another route](https://guides.rubyonrails.org/routing.html#adding-more-restful-routes):

```ruby
resources :laurel_wreaths, only: [:show] do
  member do
    put "branch_owner"
  end
end
```

## Admiring our success

This works! It's relatively contained. It feels logically located. It's *only* one additional method. What's the big deal?

I'll grant you that you could argue whether this is RESTful. But what we can hopefully agree on is that this is expanding on the default routes. Rails supports it. So we're good.

Until we aren't. To illustrate what I mean, let's take a little diversion.

## A big trunk

You may commonly hear that code constructs that are "too big" are troublesome. Big classes. Big methods. Big numbers of arguments. Big numbers of methods that an object can respond to. The trouble could be for many reasons.

1. Big things are hard to keep in your head.
1. Big things make it more difficult to find details, which may obscure bugs.
1. Big things may rob us of a more declarative, or better understood, abstraction.

There are probably more. I wouldn't want to make a list that's too big though.

Even if we do agree that this is undesirable, we have a *big* problem. Each one of us has a different definition of what "too big" is. One person's long method is another person's explanatory implementation.

How many lines of code is too many in a class? When is a PR too big? When should we decompose something? These are __difficult__ questions. They have a lot of nuance. They involve a lot of personal biases. It's frustrating to achieve consensus here.

As a rubyist, you may be familiar with Sandi Metz's "[rules](https://thoughtbot.com/blog/sandi-metz-rules-for-developers)". I think you could make an argument that all four of them deal with concerns about making things too big. Two of the four provide direct heuristics we can apply to determining when something is "too big":

> 1. Classes can be no longer than one hundred lines of code.
> 2. Methods can be no longer than five lines of code.

If you agree with those is an entirely different conversation. But there is *something* we could use to ground a discussion on when something is "too big".

## Trimming the tree

Rails' default routing provides another useful grounding. And that is to stick with the standard seven routes in a controller. More than that and you risk something becoming "too big".

That doesn't mean that seven is the correct number of things before it's too big, except that [maybe it is](https://en.wikipedia.org/wiki/The_Magical_Number_Seven,_Plus_or_Minus_Two). We might not even *get* to seven, like in our example. We only have TWO methods in that controller. By all of Sandi's rules, we didn't introduce something that was too big. Yet I'd still propose reducing it to one by introducing a new controller.

```ruby
class LaurelWreathBranchesController < ApplicationController
  def create
    @laurel_wreath = LaurelWreath.find(params[:id])
    @laurel_wreath.branches << User.find(params[:branch_user_id])

    redirect_to laurel_wreath_path(@laurel_wreath)
  end
end
```

None of the implementation changed here. We don't get a performance benefit out of doing this. What we get is a way to order the [chaos of life](https://youtu.be/sicUhD1n-es?si=FLKjEc28Yh7fmaJm). We have a mechanism to prevent any one controller from becoming "too big". No matter what your definition may be.

## Thoughts to leaf you with

Why break this out into a separate controller? You may gain some benefits in clarity around naming. You can clump related functionality together. You're not tied to controllers named after the ActiveRecord resource they're related to. It's harder to do, because naming things is hard. But by doing the hard work, we spend the time to consider how to best express an idea. We're not limiting ourselves to how to do some functionality and stopping there.

The impact of code written for a feature doesn't end when it's shipped. We need to maintain and understand that code for the remainder of its useful life. Knowing when something has too many responsibilities? Knowing when it's too big? Knowing when it has too much gravitational pull? That's all hard. By leaning on the conventions that Rails provides, we have a tidy rule of thumb to use. In this case, it helps in reducing the complexity of our controllers. Rails provides the ability to extend it. It's not "wrong" to add another method. But just because we can, [should we]({{< ref "railsconf-2019" >}})?

The next time you find yourself adding "just one more custom action" to a Rails controller, consider not resting on your laurels. You already came up with a way to make it work. Awesome! Can you use HTTP verbs to your advantage to demonstrate intent and meaning? Ask yourself if you can prevent that controller from growing in size and responsibility.

__Then__ declare victory.
