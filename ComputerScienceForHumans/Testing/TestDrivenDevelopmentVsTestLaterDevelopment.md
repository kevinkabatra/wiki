# Test Driven Development (TDD) vs Test Later Development (TLD)
Choosing a development process is very similar to choosing a religion, once you belong to a particular camp you tend to stop considering the other alternatives as realistic possibilities. 


ToDo: reconsider the following line, as I tend to lean towards TLD: I think this approach is rather short sighted especially in an era where your source could contain a small microservice or be a cog in a massive social media network. 

ToDo: add more filler

## ToDo: add subtitle
TDD is a fantastic buzz word, or words I guess, to place on a resume or drop in an interview. "In my latest project we successfully introduced TDD and managed to reduce world hunger at the same time". There is nothing wrong with using TDD, I am personally just averse to anything that becomes so popular that you can create a celebrity for it. Enter Kent Beck. I first read [Test Driven by Development](https://www.amazon.com/Test-Driven-Development-Kent-Beck/dp/0321146530), Kent Beck's instruction manual for TDD, in 2019 and I thought that this approach would solve all of the problems that I had run into during development. 

By this point in my career I had worked on multiple Enterprise Resource Planning (ERP) platforms and I constantly saw how complex it was to make any additions to an existing system. Touching one part of the system would have a strong chance to result in a negative effect elsewhere. Commonly these issues are referred to as regressions, and are best prevented via automated testing. Honestly since you are reading this article, I would hope that you already had that knowledge. In TDD, Kent championed writing small unit tests against an expected behavior, this code would fail as the behavior had not yet been written, but then it would enable a developer to write just enough code to satisfy the unit test's requirements. In theory this results in cleaner concise code that behaves exactly as prescribed.

Had the initial ERP projects leveraged an approach such as this novel concept I think that regressions would be completely avoidable. With TDD, no behavior can exist prior to the automated testing that will exercise it properly, therefore with each change to the system you should be able to validate every aspect with a click of a button. And some patience as your tests run. Automated unit testing is amazing when it is done right. With quickly executing unit tests you will have very little friction standing between you and setting up the tests to run after each build.

## 
Messiahs are just people who happen to be selling a message. 

I watched an interview where Kent Beck explained how Facebook was far too massive for him and his processes. He went on to explain that it was difficult to work on a system so large where he could not contain abstractions of all its pieces within his mind.

ToDo: explain how his admission was a crushing blow to my belief in his system. Leaders of the cult should not tell their audience that the Kool-aid tastes funny.


ToDo: TLD is great for a developer who wants to procrastinate. I think that this method works fine as long as your create the unit tests for the behavior either in the same commit or the very next one. This way the accepted behavior becomes written in virtual stone and can be trusted as you continue to modify the system.