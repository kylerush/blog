---
title: "An open minded approach to interviewing candidates"
publishDate: "20 February 2024"
description: "To get the best results when interviewing candidates, adop an open minded approach and avoid test-based interview questions."
coverImage:
  src: "./collaboration.webp"
  alt: "Software Engineer collaborative interview"
tags: ["interviews", "hiring", "management"]
---
In my experience, most companies design their talent acquisition and assessment process like this:

> We need more Software Engineers because of x. We want them to turn Jira tickets into code. To make sure they are "good" engineers, we'll assess them using coding tests. If they can pass the test, then we will hire them.

The reason (*x*) for needing more Software Engineers is usually something along the lines of:

- "We need to write code faster"
- "We need to launch a new product and need a new team to do it"
- "We have too much work for the current team"

But in any case, the team will get to work creating a candidate assessment process that is mainly designed to tell the interviewers if the candidate knows the same things that the interviewers know. They will design interview questions like:

- What is the difference between `unknown` and `any` in TypeScript;
- What is `as const` and why would you use it in TypeScript;
- Solve this coding challenge that I designed.

We'll call this the "test-based interview process." Relying solely on this approach can create some serious long term pitfalls.

## Why the "test-based" approach produces inferior results

![Software Engineer Test](./hiring-test.webp)

In the above example, the team will decide they need more Software Engineers that can write JavaScript, because their current application is built with JavaScript. Then they'll decide that they will assess the candidates on tasks that are similar to the work they are currently doing. I build React components based on comps from our designer. Or, I build new features/endpoints into our GraphQL API based on specs from our Product Manager. This interview process will tell them if candidates can do this kind of work. Great, right? No.

The first problem with this approach is it assumes that the solution to all your company's problems is the solution to the company's current problem. Keep building React components. Keep building new API endpoints. In other words, a "hammer" works very well for us right now, so let's get more hammers.

But what happens when your company's problem starts looking more like a screw than a nail? You'll continue using the hammer to apply the screw and you'll fail. Uh-oh. This problem is nearly inevitable. The challenges a company faces will evolve over time. It might be due to growth from the company's success or due to macro-environment changes. One way or another it will happen. In other words, the only constant is change.

The second problem is this approach can kneecap the knowledge and skill development of the team. How can hammers learn to solve problems other than nails if they only have exposure to hammers? A team full of hammers dramatically reduces your ability to think creatively about problem solving. Perhaps a saw is the better tool for the problem. The hammers on the team know that a saw exists, but it seems dangerous and they don't have experience with it. Best to stick with the hammer.

With the test-based approach everything will be fantastic at first. Another React engineer to take on more tickets! The CEO is super happy! But over the course of a year or two—which, trust me, is not a long time—the problems above will snowball. The hammers will keep contorting the new challenges to look like nails. They'll create some contorted React solution to solve a database problem or vice-versa. Bugs and performance problems will run rampant. The CEO will start to lose confidence in the team and believe there is a talent problem. Everything was going so well and now it's awful. Where did we go wrong? The hammers will have no idea because this problem is not a nail. In addition to the CEO, the hammers will lose confidence. Now, all that's left is a team that doesn't trust each other and can't solve the company's problems. We won't go into what happens next.

![Frustrated CEO](./frustrated-ceo.webp)

## Open-minded hiring

In your test-based hiring process you will inevitably come across valuable talent that can't pass the test. For example, a candidate that is a good problem solver and collaborator has experience with your typical MVC based PHP web application. They will certainly fail the TypeScript/React code test. In fact, this candidate may never even apply because your job description lays bare your test-based interview process: it requires experience with React and TypeScript. So now you've widdled out a good problem solver and collaborator from your interview process, or discouraged them from applying at all. Now all you have in your interview pipeline are hammers. No saws.

But wait a second, this is crazy! Why would we hire a PHP web developer for a job to write React code? The answer is because you want effective problem solvers and collaborators on your team. This is the most important thing for long term success. You want to hire problem solvers that think and work creatively and effectively. Problem solving is not limited to React and TypeScript. Even more, problem solvers that are exposed to a variety of tools are more effective problem solvers. Having the knowledge of how the MVP PHP community solves web application problems *helps* your React team solve problems better. Even more, if a PHP developer applies to your React engineer job, it signals a candidate that is open-minded and wants to try different solutions to solve problems. This is a *good* thing.

Of course these technology examples are contrived and not the main point. This has nothing to do with React and PHP. The main point is that a team is simply more effective at problem solving if it has seen several different ways to solve a problem.

![Teaming](./teaming.webp)

## Why you should consider the open minded approach

By adopting an open-minded approach you can have a higher chance of success. The open-minded approach is exactly what is sounds like. The reason it produces better results is because it doesn't assume that the interviewers are omnipotent. It actually presumes the opposite. It presumes that there isn't one correct way to solve a problem. It doesn't assume that tomorrow's challenges will look like today's. It doesn't assume that the best hire is pre-determined.

What exactly is an open minded interview process? It's a process that is designed to let the interviewer show you who they are. This is in contrast to the test-based approach which tells you if a candidate is like you. Here's a simple way to think about it. When you're interviewing a candidate, the main goal is not to see if they know what you know. The main goal is to see what you don't know. I mean this in a board sense. It's not necessarily about nuances of a programming language, although it can be. But it's also about how someone thinks and how they work.

## Some ideas for open minded interviewing

An open minded approach is not a specific set of interview questions, tasks, or steps. Instead, it is a mindset. You must think if your interview design allows you to know who a candidate is, or, if it tells if you if the candidate is like you. This will look different for every company and team.

That said, here are some suggestions.

### Code challenges

An open minded approach is not against coding challenges in interviews. I would argue coding challenges in interviews are a great open minded approach. But how you design the code challenge makes all the difference. In a test-based process, an interviewer picks the code challenge (or creates on themself), writes the answer to it, and probably creates a rubric to judge the candidates work. Sometimes the coding challenge will happen live while the interviewer watches the candidate code. Or sometimes it's a take home challenge.

How might it look with an open minded approach? A good idea is to design a code challenge that you will work together with the interviewer on. You might do [pair programming](https://softwareengineering.stackexchange.com/questions/444098/whats-the-correct-way-to-do-pair-programming), or even better, [mob programming](https://www.agilealliance.org/resources/experience-reports/mob-programming-agile2014/#:~:text=Mob%20programming%20is%20a%20software,code%20at%20the%20same%20time.). You have a driver (the one who writes code) and a navigator. The interviewer and candidate switch between these two roles every five minutes during the coding challenge. This allows you to assess the candidate holistically, not just on how they singularly write code. This should give you good signal on many things like how they collaborate, how they problem solve, and yes, their knowledge of programming language nuance.

Another modification to the coding challenge is to choose the coding exercise together with the candidate. Pick from a list of coding challenges that neither of you have done before. This might help the interviewer get their brain out of "do they know the difference between `any` and `unknown`" mode. Instead of the interviewer's brain narrowly focused on a checklist, they can actually assess how the candidate solves problems. As an interviewer, you should get out of the mindset that you are there to assess the candidate and into the mindset that you are there to sole a coding challenge with the candidate.

### Interview questions

With the test-based approach, you'll often see questions like "Tell me about a time when you..." The problem of course is that these questions probe for a narrow, specific thing that the interviewer wants to know. Instead, you want questions that will let the candidate tell you who they are. Open ended questions will achieve this goal. An open ended question has no assumed endpoint. You have no idea where it will go. Consider the question "What would you say is your biggest success in your current role?" We have no idea where that one will go. And there's an entire work of open-ended follows ups:

- "How do you know that it was a success?"
- "What was the secret to the success?"
- "What impact did your success have?"

You can also flip that around and make the question about their biggest failure.

## In closing

As I mentioned above, an open minded process is a mindset. Ask yourself if your process is designed to tell you specific, narrow, pass/fail type questions. Or is your process designed to allow you to spot collaborative, creative problem solvers. A team full of the latter will have a much higher chance of success.

Hiring can be very challenging and there's no one-size fits all approach. This is why I frame open minded hiring as a mindset and not specific steps. You still need to do a good job assessing the candidate. I also don't mean to suggest that someone's experience with certain technologies is irrelevant. I am suggesting that their experience with certain technologies is one part of the overall benefit the candidate provides. If you looked at the big picture with an open mind and still determined that you just need another React engineer, that's totally fine too. But keep in mind the problems that snowball over time with test-based hiring and course correct accordingly.
