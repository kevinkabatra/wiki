## Test Later Development (TLD), not Test Last Development
TDD is very simple to introduce to a greenfield (new) project. Generally for such projects there are no previously established processes that you are restricted by, and there is certainly no legacy source that you have to be the steward of. TDD is also easy enough to introduce for new work in an existing project, so long as you maintain the diligence of writing the tests prior to creating the behavior. The lack of this diligence is is a gap hopefully filled by TLD. 

As a professional developer you might be required to develop the behavior first in order to have various stakeholders approve of the design, once that is complete you could go back and build your unit tests to ensure that this behavior remains stable in the future. You could also be tied to a very tight deadline where you do not have the time to both write the unit tests and the behavior. This second point is rather complex as you are essentially robbing Peter to pay Paul. Sure in the short-term, if you have a stable project already, you can introduce new behaviors and do some manual regression testing before that important demonstration. But if you work in an environment where you are constantly under the same gun, are you actually going back to add in the tests that you skipped? If not then in the long-term you will have a greater chance of creating a very unstable project that takes longer to ship and has increased development costs.



ToDo: I am not currently happy with the organization of this article. At this point I am not quite sure what point I am making and I can see that come through the content that I have written so far. Each section should be further expanded upon to highlight their strengths and weaknesses. 







Messiahs are just people who happen to be selling a message. 

I watched an interview where Kent Beck explained how Facebook was far too massive for him and his processes. He went on to explain that it was difficult to work on a system so large where he could not contain abstractions of all its pieces within his mind.

ToDo: explain how his admission was a crushing blow to my belief in his system. Leaders of the cult should not tell their audience that the Kool-aid tastes funny.


ToDo: TLD is great for a developer who wants to procrastinate. I think that this method works fine as long as your create the unit tests for the behavior either in the same commit or the very next one. This way the accepted behavior becomes written in virtual stone and can be trusted as you continue to modify the system.