# Unit Testing with Unity
It is important to introduce a mechanism to test your code early and often. This will help reduce the friction from introducing new unit tests for each code change. I use the term friction here as "anything that increases the mental resistance to attempting | completing some task". The early introduction of a testing framework will help guide your development of your objects and logic. You will naturally ask yourself "how can I validate that object A does its thing?", you could of course manually test this functionality. You could then add this manually regression test to a larger body of manual tests that increase in scope and time to complete. This list is inherently flawed for two reasons:
* As the list grows the friction for completing the list with each code change will increase. I would venture to guess that it would not take long for the entire regression test suite to be thrown out of a moving car's window and you begin to test only the areas of the code that you think you changed. This is especially true if your efforts are commercial in nature, and then managers and executives will begin asking what you can do to reach increasingly tighter deadlines.
* This list is being validated by humans and even the rocket scientists amongst us make mistakes in validating their checklists.

With this in mind I think it is quite obvious that automated unit tests are more sustainable and accurate in the long term. In this article I will explain how I have introduced unit testing for my games based on the Unity game engine. I leverage not only a multilayered architecture, which I wrote a previous [article](https://kevinkabatra.wordpress.com/2020/12/02/logical-multilayered-architecture-with-unity/) on, but also the [Unity Testing Framework](https://docs.unity3d.com/Packages/com.unity.test-framework@1.1/manual/index.html).

## Test-Driven Development (TDD) vs Test-Later Development (TLD)


Business | domain logic layer
    * Player
        * Health
        * Score

Using multilayered architecture to have a separate business layer for code that does not depend on Unity assemblies. This allows normal unit tests to be written.
Using Unity Testing Framework, find link, to create Play Mode unit tests. Play Mode unit tests allow you to create an instance of your game objects and test their behaviors.

MonoBehavior.Instantiate requires a Prefab, and the Prefab must be placed under Assets/Resources folder in order to access it.
It is very important to separate the concept of the Prefab from the object that is in the actual scene. For example a few times I would make a change on the game objects within the Scene and forget to apply it back to the Prefab, then run the unit tests and they would fail. Possibly nine out of ten times a new test failure would be because of this.

Mention timing, especially the limitation on the Start method not being called prior to the unit test starting to run. And not having any proper control over the timing of the objects.

Mention how much Singletons suck for unit testing and why they should be avoided in Unity at all costs. The tests are not running in parallel but they are also not finished with cleaning themselves up prior to the next test starting. And due to where Destroy is in the execution list, bad things tend to happen with Singletons.