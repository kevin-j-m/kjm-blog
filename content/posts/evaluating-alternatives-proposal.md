+++
title = "I know I can, but should I? Evaluating Alternatives - The Proposal"
date = 2022-08-03T21:23:22-04:00
tags = ["conference-proposal", "conference", "railsconf"]
summary = "Presented at RailsConf 2019"
description = "The proposal for a talk I gave at RailsConf 2019."
+++

## Abstract

You __can__ use a hammer to drive a screw into wood, but I’d recommend a screwdriver. Why? And when is a hammer the better option? This talk will propose a framework to use when comparing alternative technical choices. I won’t decide for you, but will leave you with a structure to apply in your decision-making process.

The ruby toolbox is vast. While Rails provides a default experience, it leaves plenty of room for alternatives. In learning how to do something, you may uncover different ways to accomplish the same goal. Determine which tool fits best in your situation with these lessons.

## Details

The Rails Doctrine frequently speaks to the breadth of options available at your disposal within the framework. There are many [sharp knives](https://rubyonrails.org/doctrine/#provide-sharp-knives) in ruby that, when used judiciously, can facilitate a great result for your product, your team, and your code. However, knowing in which situations to reach for one tool, technology, or architecture when the framework supports [many paradigms](https://rubyonrails.org/doctrine/#no-one-paradigm) is difficult. Having the option to [make substitutions](https://rubyonrails.org/doctrine/#omakase) is liberating, but imposing.

Some groups or situations may demand rigorous up-front [evaluations](https://en.wikipedia.org/wiki/Analysis_of_Alternatives) prior to determining a solution. While those exercises can inform our choices, a lighter-weight approach is more appropriate in most decisions we make on a daily basis as developers. When presented with multiple options that will solve a problem, the following criteria are helpful to consider:

* What impact does this have on the code, the product, and the team?
* Has the team encountered this before? What was the result?
* What cost and risk will be incurred by introducing this solution?
* How will this be maintained over time?

Not all of these questions may be applicable in all situations, and the relative weight of one over the others will also vary, depending on the context of the team or the problem. These questions will be exercised in this talk to reinforce their value and applicability in making practical decisions about daily programming choices such as:

* Should I use an ActiveRecord callback, or wrap this logic in another method?
* How, if at all, should this change be tested?
* Can I write this myself, or use an external library?
* Do I expose default timestamp values to users or add a new attribute?

### Outline

1. Introduction
2. “Traditional” Project Management Approach
  * Methodology
    - Present a detailed, analytical approach before work is done.
    - Identify all criteria and factors applicable to the project (cost, effectiveness, risk, etc.), with relative weights of importance.
    - Score all identified alternatives on criteria, multiply by weight, compute final score.
    - Implement highest-scoring option.
  * Limitations
    - Significant upfront investment in time.
    - Assumes perfect knowledge prior to execution.
    - Inflexible to new information or constraints as the project evolves.
  * Merits
    - Requires thoughtful and thorough consideration of all options.
    - Provides cover and justification for decisions made.
    - Rallies all stakeholders around a consensus determination.
3. Evolving Traditional Approach for Daily/Common Decisions
  * What impact will this have on the project?
    - Research, prototype, or spike out how well this resolves the issue at hand.
    - Identify how your supporting team may react to the change. Consider their past experience, strongly-heldbeliefs, and ability to build on or maintain this.
    - Contemplate what, if any, standard this sets for future similar problems. Communicate, if necessary, not only for the current team but future contributors, the lens through which this should be evaluated for upcoming features.
    - *Example*: Should I use an ActiveRecord callback, or wrap this logic in another method?
  * Has the team encountered this before?
    - Determine what institutional knowledge can be brought to bear to short-circuit the decision-making process.
    - Evaluate the relevance and applicability of any past decisions or solutions to similar problems.
    - Identify what, if any, conventions or standards should be adhered to within this codebase or organization.
    - *Example*: How, if at all, should this change be tested?
  * What cost and risk will be incurred by introducing this solution?
    - Recognize that “correctness” is not the only consideration in delivering features.
    - Take into account contextual pressures. Keep in mind what amount of time or money is available to address this problem.
    - Expand the evaluation of cost and risk beyond the initial implementation and through the entire lifecycle of the product.
    - *Example*: Can I write this myself, or use an external library?
  * How will this be maintained over time?
    - Speculate on how extensible or malleable this implementation is to future requirements or features.
    - Identify who will need to be involved in the long-term viability of this solution and solicit their input.
    - Determine your own appetite to continue the use of this work as a proxy, if nothing else is available.
    - *Example*: Do I expose default timestamp values to users or add a new attribute?
4. Conclusion

## Pitch

For many, the initial excitement of learning to program is in creating something. Having a computer perform a task can feel like magic. That rush can get you far, resulting in successful projects, companies, and careers.

At some point, understanding different ways to solve the same problem becomes compelling, either due to intellectual curiosity, organizational need, or other circumstances. However, ingesting all of the alternatives that exist can be exhausting, leading to analysis paralysis. Having a structure in place to help implement these decisions is crucial with the current number of options available to us as developers.

As a consultant, I’m exposed to a number of organizations looking to solve difficult problems. Each of them has different constraints that influence one choice over another. Much of my job involves understanding the pressures facing the team and proposing solutions that will not only solve the problem, but do so in a way that best meets the needs and addresses the goals of the group.

## Bio

Kevin lives near Boston, where he is a Software Developer at The Gnar Company. As a consultant, evaluating technical alternatives and proposing solutions is an important part of his job.
