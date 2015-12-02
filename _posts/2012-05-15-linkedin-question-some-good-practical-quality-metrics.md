---
layout: post
---
Naveen Maram [asked on LinkedIn](http://lol.cat/2G) "What are some good, practical quality metrics for your IT development team that worked in your setting and you have personal experience with?". 

As a [software architect](http://yet.another.linux-nerd.com/what-i-do), I've been using metrics to guide me for my code reviews, here is my two cents on the subject.

The main objective of metrics is to help the team keep the project under control, but be careful: avoid vanity metrics (e.g having a high code coverage does not mean you code is well/thoroughly tested), only pick actionable ones. If your continuous integration system shows green ([or blue](http://jenkins-ci.org/node/377)) lights, it does not imply that the project is perfect : code review is still mandatory, all metrics should be considered in order to avoid a specific pitfall : badly architectured project (making the code non agile and not adapted to change), code redondancy, [big ball of mud syndrome](http://en.wikipedia.org/wiki/Big_ball_of_mud), developper take over among others. I like the following ones - which I usually look per developer in order to check a specific element of quality. 

- Code duplication 
- Cyclomatic complexity 
- Adherence to code formatting standards 
- Ratio of successful/failed build 
- Change Risk Analysis and Prediction (CRAP) index 
- Performance on preproduction servers which is often done too late in projects, and should be automatic as the unit tests are. 
- Number of $something (with $something could be variables, interfaces, children, overridden methods) and see if one of your team member is not creating too much or too little $something. 
- Unit test coverage per directory/package/class ** if well done => be sure to look at the tests themselves ** 

Sometimes, developers and architects set goals (e.g. enhance code formatting) and always adjust them on every sprints because your expectation do change during the project : for instance, you may not need great performance at the beginning of the project, or you may want more stable release after a few sprints. You can also look for new metrics during project if a specific problem arise : it should be flexible.

Those metrics are only a way for your architect or team leader to find quickly code that could potentially be bad or a team member that is not engaged enough, but are in no way a substitute for human work/review. 
