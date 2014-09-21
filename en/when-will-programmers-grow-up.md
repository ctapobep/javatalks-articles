Do you still think that high-quality programming skills, knowing SOLID principles, writing maintenable tests is enough for you to be
called a professional? Let's elaborate this topic.

When I was young and -stupid- passionate about high-quality programming, all my attention was drawn by maintenable, flexible
easily readable code. I was reading hell a lot of literature on this - both books and articles. Later on I started to teach others. 
I was conducting webinars & tech sessions (both at work and on the internet), writing articles and discussing these topics at 
[JTalks](http://jtalks.org). And I still continue on this, though way less actively.

But this passion was leaving me. Gradually, year by year. First of all because I had another passion now - Continuous Deliver, and
second - it started to seem that people who are well-educated in programming can harm projects dramatically. Here is the best example
in my carrier that illustrates what I'm talking about:

----
At some point of my life I decided to join a startup. It seemed to be a good option to me: they used modern technologies and frameworks and during the interview they asked me about best practices which was a promising sign. Moreover teleworking was really handy to me at that time. So here I am resigned from my regular job for the sake of intriguing startup. 
From the first days it started to look wrong: they didn't have daily meet-ups to be in synch with each other (there were weekly meetings which is pretty rare) and they had a dozen of repos in VCS - too many for the first months of the project! That seemed suspicious, and it didn't turn better.. 
It appeared that last couple of weeks were spent for designing domain model and database scheme (wat?). And they were not there yet! I started to be involved in all this designing and I found that every team member suggested their vision of the model and here comes the discussion.. That discussion was overwhelmed with buzz-words like "Best Practices", "OOD", "Performance" and in the end they chose the most complex scheme suggested.
I appealed to the customer who (surprise-surprise!) was an ex-developer with questions and suggestions on improving the current situation:
- Team communicated once in a week, for that project that meant they worked couple of days and then waited several days for the meeting to share their thoughts. That's extremely ineffective.
- They started with distributed storage, complex designing and performance in mind while there was nothing to optimize and refactor yet.
But the customer, even though he was irritated by the fact that there was nothing ready by that time, was too involved into technical details. He was possessed by "we should implement it in a right way!". To me that was too ineffective for such a small-budget project. So call me a rat, but I left that sinking boat after 2 weeks of struggling. I denied to take money from the customer, I knew he'd need them in the near future. Couple of years have passed, but on their website I still see 'Coming soon..'.
----
That situation was a good lesson to me - I started to realise how stubborn and selfish I was in the past. I recall working in a small company where I was asked to write a tiny project to mix some videos, list them and show them to the users who can vote. The project had a budget for 2 months and I could it make! But I was too blind because of good practices and maintainability. I introduced complex technologies, caching; of course I split the project into different layers and added a lot of extra interfaces for flexibility. Should I say I missed deadline? Surely I did. I was asked why it took me that long, but it was hard for managers to understand the importance of that complexity. Now it's hard even for myself! That project didn't require maintainability, extensibility, flexibility or any other "bility", it didn't! No changes were made to that project since then. In retrospective I understand that all that crap was needed for myself, I was selfish. I needed to ramp up my skills (which is of course important), but I did it at the expense of missed deadline. And now, when I hear developers critisizing managers, I'm careful in supporting the conversation. Because who knows, who knows..

I don't want to say that well-written code is bad, I don't! And neither I defend bad-smelling code. But both extremes are equally bad. Both situation lead to money lost. On one hand it's possible to end up with a mess that's impossible to maintain and which leads to lots of nasty bugs. On the other hand the project can die bofore it's even born! 

But you may still say: "Hey, I always write high quality code that doesn't smell and I always spend weeks on careful designing, but my customer doesn't go broke! He still is happy with the results!". Well, that might be true. But first of all, customer doesn't necessarily live on the edge, it's pretty common for the customer to have big funds and even if your project doesn't bring any value, he still might be happy, he just doesn't notice ineffectiveness! And second - in the majority of cases the costs spent on the project development are much smaller than overall spendings on the business. Even if customer heavily relies on the software you work on, there is still plenty of stuff to care about: marketing, 3d party contracts, client-facing offices and personnel, taxes, transport, et cetera, et cetera. So in the end ineffective development might not be *that* important for the business.

Then who cares? If I work a large wealthy company, what's the big deal if I'm ineffective? Here is my view on this:
- 
