---
title: "How working software is made"
publishDate: "5 March 2024"
description: "How working software is made."
tags: ["agile"]
---
I often find myself working with non-technical executives who become very frustrated with their company's product. This always leads to a breakdown of trust with the Engineering team. Almost always, the problem is a fundamentally misunderstanding of how working software is made. This post is written to help the frustrated, non-technical executive.

## Why should you care
If you work at a company that sells a technical product to a customer it is important to realize that nearly *all* customer value that is delivered will be derived from the Engineering team. You simply cannot ship customer value without the Engineering team. Interestingly, the opposite is _not_ true! The Engineering team can ship customer value without a single executive, Product Manager, etc. Your business is squarely dependent on a high functioning Engineering team.

## What is working software
Before we get into how working software is made, we need to define working software. Each company needs to create their own definition of working software, but for this blog post I'll define it as software that:

1. Solves a customer problem;
2. Does not suffer from regressions;
3. Has little to no bugs;
4. Has acceptable uptime and performance to the customer.

If our software checks all these boxes, we presume it is software that customers want to buy. Our definition of working software always needs to be rooted in what our customers want to buy.

## How non-working software is made
To better understand how working software is made, let's walk through a fictitious example of how non-working software is made. At our fictitious company, an Executive is in charge of the decision making. The Executive decides the 4 customer problems the company will solve this year. Then the Executive delivers the problem set to the team.

Just like most other executives, our Executive craves certainty. Strangely enough, our Executive knows deep down that certainty is impossible, but still, it makes him sleep better at night the more we approach our work as if certainty can be achieved. To make things feel certain, the Executive requires the team to produce a detailed roadmap of when these problems will be solved.

Now it's the Product Manager's turn to step up to the plate. These customer problems need solutions so the Product Manage (PM) has a think on it. In a short enough time the PM determines the solutions to the customer problems. We're halfway there to achieving certainty!

Now comes the time to talk to the "coders" A.K.A the Engineers. To reduce "risk," the PM delivers each solution in neatly packaged "acceptance criteria." At this point, all the problem solving is figured out for the Engineers. The Engineers simply need to create a time estimate for the acceptance criteria.

Up to this point everything has gone great! But now tension starts occuring within the Engineering team. Deep down they do not trust their own estimates. They know correctly that it's essentially impossible to estimate these acceptance criteria in a codebase of hundreds of thousands of lines that no one person, or group of people, can keep in their head at one time. They know that no matter how much time they spend estimating, they still can't produce a good estimate.

But the signals the Engineers receive is that certainty is most important so they spend lots of time creating bogus estimates. They fly these estimates up the chain. This bogus estimate data is neatly stitched into our beautiful yearly roadmap. Our Executive absolutely loves the certainty it depcits. The team rejoices. We have solved the hard problems. Now we just need to write and ship the code!

The entire organization is now incorrectly lead to believe that the only measure of success is adherence to the almighty roadmap. Just hit the roadmap and success will follow.

Let's examine why this organization is on an excellent path to deliver non-working software. I'm going to work from the bottom up.

### Estimates are bad data
Our first earth shattering revelation, which, if you're anything like me, may take you many years to accept, is that estimates are complete garbage data. They are harming your ability to ship customer value in more ways than you realize.

First things first, the concept of an estimate is fundamentally flawed when applied to the type of work Engineers should be doing. Engineers should not be taking "tasks" or "acceptance criteria" and implementing them. That's a process of translating acceptance criteria into code. It's a "code monkey" task. It is something that ChatGPT can do relatively well. Instead, Engineers should be problem solving. They should be looking at the customer problem, and using the tools available to them, to try and solve it.

Why should Engineers be problem solving and not translating acceptance criteria to code? Because no other person within your entire organization can find a better technical solution to a customer problem that the Engineers. The Product Manager's solution is inferior because it does not come from a deep understanding of the technical options available to solve the problem. Product Manager should not be solving customer problems, but rather, helping the Engineers identify the highest priority customer problem to solve, and actively problem solving with the Engineers.

With this framing, I hope it is clear that putting an estimate on solving a problem is rather silly. If not, let's suppose somebody back in the day asked Albert Einstein for an estimate to solve problems and inconsistencies in the existing theories of physics. Most problems worth solving are impossible to estimate.






