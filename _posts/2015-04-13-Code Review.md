Code Review
===================
A couple of months ago my team and me had barely tried [code reviews](http://en.wikipedia.org/wiki/Code_review). We had heard about it, but we really never got around to try it. Personally, I did not really know how to conduct one and if it really was something that would save us time.

The turnaround was when we took over an existing project that used [github](https://github.com) as source control. We found that when we started creating pull request and commenting on them in github. We decided to be very particular and thorough on coding standards and the way we wrote the code in the files that were changed. After doing this for a while, we saw that the code quality increased and we caught bugs that perhaps would not be found if we did not do the code reviews.

A curious side effect were that the team made it a sport and strived to have the perfect code review. We made an effort to not make syntax and logical errors, and the ones that did the code reivew tried to find errors. It has made an "I want to write better code" culture in the team. When we finish up the code we usually spend 5 extra minutes to perform the [boy scout rule](http://programmer.97things.oreilly.com/wiki/index.php/The_Boy_Scout_Rule), look for duplicate code, check code syntax, etc.

A different side effect was that the team members understood the system faster.In our case the new project we took over had a learning curve. We had to learn the architecture, the tools and domain. I believe that the discussions and reading the code from the code reviews let us learn and understand more efficiently. Moreover, we learnt more from each other as well. For example, different programming techniques and optimization tricks. To take a concrete example, I learnt from a code review today that catching an exception and throwing the same exception lead to a misleading stack trace. Instead just use 'throw' without the exception itself.

Even though the code reviews took time it would, all in all, take less time because of faster [feedback loops](http://www.infoq.com/news/2011/03/agile-feedback-loops) and less bugs. Our main feedback loop was from the QA's and the customer itself. When the developer transferred the code into production it might take days before a bug might have reported. With code reviews, it took hours or minutes instead of days. Since the silly bugs that could lead to an unstable program or perhaps spelling mistakes would be caught by the code review.

To sum it all up, here are the reasons to start performing code reviews:
- Less bugs
- Shorter feedback loops
- Higher quality code
- Team members get better understanding of the code and learn more from each other.

So how can your team start with code reviews? Your team can sit down and create some rules that you can follow for a code review. Moreover, I also recommend starting out beeing strict about them. Here are some rules that we try to follow and hopefully you will appreciate them aswell:
- Is the code following coding standard?
- Do you easily understand the code?
- Are tests written?
- Are there any spelling mistakes?
- Is the code optimized?
- Is the code [DRY](http://en.wikipedia.org/wiki/Don%27t_repeat_yourself)?
- Is there any dead code or unused imports or usings
- If there is a GUI
    - Does it look good?
    - Can you understand how to use the feature easily?

