# Test Driven Development (TDD)
Choosing a development process is very similar to choosing a religion, once you belong to a particular camp you tend to stop considering the other alternatives as realistic possibilities. Sometimes this is due to cognitive biases (I am guilty of this), historical decisions, or the feeling that you should not devote more time to consider another process when one is already working for you. I think this approach is rather short sighted especially in an era where your source could contain a small microservice or be a cog in a massive social media network. Long gone are the days where we would focus on a single discipline or language, we are now in an era where proficient generalism is greater than the mastery of a singular topic. So as perpetual students of our ever-changing Computer Science it is imperative that we consider alternatives to our approaches as they arrive. 

In this article I aim to introduce, or possibly re-introduce, you to the Test Driven Development ([TDD](https://en.wikipedia.org/wiki/Test-driven_development)) process. Kent Beck [reintroduced](https://www.quora.com/Why-does-Kent-Beck-refer-to-the-rediscovery-of-test-driven-development) the software developing masses to the forgotten concept of `writing units tests prior to writing behavior` in 2002 with his book [Test Driven Development: By Example](https://www.amazon.com/Test-Driven-Development-Kent-Beck/dp/0321146530). I imagine at this time developers were huddling together forming miniature cults to collectively worship around the altar of Beck. Prior to releasing his book, Kent was fine tuning this process as part of [Extreme Programming](https://en.wikipedia.org/wiki/Extreme_programming) in the late nineties. [He](https://en.wikipedia.org/wiki/Kent_Beck) has helped usher the introduction of software design patterns, xUnit, agile software development, and more. With such an impressive resume I feel like we need to consider his teachings and see what they can offer us.

Before I continue, there are of course many other processes and perhaps I will author future content covering them. If you are interested in that please be sure to leave a comment with what processes you would like to see covered.

## The development cycle
In the TDD gospel there are six steps within the development cycle. I wanted to say commandments, but I restrained myself.

1. Add a test to validate a behavior that does not exist yet. While this concept might be strange when you first encounter it, think of this step as drafting a contract for a vendor to build you a new home. Prior to the home existing the vendor and yourself came to an understanding as to what type of home would be built.
2. Prove the new test fails by running all of your tests. Everything should pass except for this new test. If this new tests passes then your test does not function properly and you must rewrite the test to fail, again think about your unbuilt home. In this example your test can be a simple `DoesHomeExist()`. Also if any other test fails, you must stop your new development as you have discovered a regression. This regression should be addressed prior to adding any new behavior.
3. Add the missing behavior. Time for the vendor to build the home. Importantly the vendor will only build your home to the specifications of the contract (the test) and will not deviate. After all we have a contract! Following this discipline requires you to write only as much is required to pass the unit test, no more and no less. This in theory should result in cleaner concise code that behaves exactly as prescribed.
4. Run all of your tests again, everything should pass. If the test introduced in step 2 still fails, well you need to go back and revisit step 3. If any of the other existing tests fail, then you caused a regression and you should resolve that immediately. Beck recommends reverting your changes in step 3 and starting over, but I think that is a heavy handed approach. If you are a competent developer you will instinctively hook up your debugger and isolate the cause of the regression. Once you understand what took place you can make a decision as to how to approach it.
5. Once all tests are passing, also known as "in the green", you can begin to start cleaning up your new code. The vendor would not leave you with a messy new home, so you should not leave your code in a state that will be hard to support and maintain in the future. With each change you must repeat step 4 to ensure that your code base is stable.
6. Repeat. For each new feature return to step 1 and run through the cycle again.

Automated tests are a fantastic way to prevent regressions and to help define what your code base should actually be doing. Again with TDD that definition is created prior to the actual behavior existing, therefore you will be able to test every piece of code as it is written. With each change to the system you should be able to validate every aspect with a click of a button and some patience as your tests run. Automated unit testing is amazing when it is done right. With quickly executing unit tests you will have very little friction standing between you and setting up the tests to run after each build. Whereas time consuming unit tests invokes a mental image of a popular comic strip that shows developers sword fighting while their code is compiling. This time consumption will quickly become overwhelming friction that causes the developers to run their tests infrequently, only to discover regressions and other failures far too late in the process. Therefore, quickly executing reliable unit tests are generally the foundation of any testing strategy.

## Example
I learn best through doing and watching an example rather than just reading about theories, so lets take some time to create a simple project following the Test Driven Development process. In this example we will create a simple calculator that is capable of addition and subtraction. Using Visual Studio we will create the `Calculator` project which of course at the same time creates a solution of the same name. This project will target .NET Standard 2.0, so that it could be used as a library for another application in the future. Once the project has been created delete any default classes that are created, in this example my Integrated Development Environment (IDE) created an empty class called `Class1`.

Now it is time to create our testing project, as we need to have unit tests prior to coding any behavior. This project will be called `Calculator.UnitTests`, I find it helpful to have such clear naming for projects and objects, it makes navigating and maintaining a solution so much easier in the long-term. This unit test project will be based on xUnit, but you can insert your preferred unit testing framework instead. Once that is created please delete any default classes again, in this case we have `UnitTest1`.

At this point your solution should look like this:

![Solution](.\TestDrivenDevelopmentAssets\SolutionWithNewProjects.png)

#### 1. Test: Can create new Calculator
Now it is time to create a class in the UnitTests project called `CalculatorTests`. In a larger solution you could separate the addition and subtraction tests into separate classes, but for this example that would be complete overkill. The first thing that I want to test is that I can in fact create a new calculator, without that there is nothing that I can add or subtract. Consider the following code:

```csharp
namespace Calculator.UnitTests
{
    using Xunit;

    public class CalculatorTests
    {
        [Fact]
        public void CanCreateNewCalculator()
        {
            var calculator = new Calculator();
            Assert.NotNull(calculator);
        }
    }
}
```

Now this code will not compile at this time. Why? Because `Calculator` does not exist yet. So lets create a class for that.

```csharp
namespace Calculator
{
    public class Calculator
    {
    }
}
```

Remember, without a unit test we cannot actually add any behavior to this object. Build just the Calculator project and then update the UnitTests project to reference the Calculator project. Once this is done you should be able to build the UnitTests project, before doing this though open up Test Explorer and configure our unit tests to run after each build. With that set, build your entire solution and watch as Test Explorer runs and passes our test.

![Test Explorer Results](.\TestDrivenDevelopmentAssets\CanCreateNewCalculatorTestPasses.png)

#### 2. Test: Can Add two numbers
Time for our first test that will actually do something. Sure initializing that Calculator was riveting but that was just the appetizer. Now we could just add the new test as:

```csharp
        [Fact]
        public void CanAddTwoNumbers()
        {
            const int one = 1;
            const int two = 2;
            const int expectedResult = 3;

            var calculator = new Calculator();
            var actualResult = calculator.Add(one, two);

            Assert.Equal(expectedResult, actualResult);
        }

```

But this method involves me duplicating the initialization code again and I do not want to have to do that for every test that we add. So instead I will first add a field to the class to store a Calculator. At the same time I will update the constructor to initialize this Calculator. xUnit will call the constructor with each test that executes.

```csharp
        private Calculator calculator;

        public CalculatorTests()
        {
            calculator = new Calculator();
        }
```

I will then update the class to inherit from `IDisposable`. Using the `Dispose` method xUnit will clean up the Calculator so that I can actually make a new Calculator for each test. In this example it is honestly not too important to have a brand new Calculator but it certainly is required when testing an object that has a state.

```csharp
        public void Dispose()
        {
            calculator = null;
        }
```

Now we will update the addition test to look like this:

```csharp
        [Fact]
        public void CanAddTwoNumbers()
        {
            const int one = 1;
            const int two = 2;
            const int expectedResult = 3;

            var actualResult = calculator.Add(one, two);

            Assert.Equal(expectedResult, actualResult);
        }
```

So our entire testing class should resemble the following code.

```csharp
namespace Calculator.UnitTests
{
    using System;
    using Xunit;

    public class CalculatorTests : IDisposable
    {
        private Calculator calculator;

        public CalculatorTests()
        {
            calculator = new Calculator();
        }

        public void Dispose()
        {
            calculator = null;
        }

        [Fact]
        public void CanCreateNewCalculator()
        {
            var calculator = new Calculator();
            Assert.NotNull(calculator);
        }

        [Fact]
        public void CanAddTwoNumbers()
        {
            const int one = 1;
            const int two = 2;
            const int expectedResult = 3;

            var actualResult = calculator.Add(one, two);

            Assert.Equal(expectedResult, actualResult);
        }
    }
}
```

Time to introduce the functionality. Lets introduce an `Add` method that takes two integer based parameters, and obviously based on the test it needs to add the two numbers together.

```csharp
        public int Add(int addendOne, int addendTwo)
        {
            return addendOne + addendTwo;
        }
```

Which makes our entire Calculator class contain the following code.

```csharp
namespace Calculator
{
    public class Calculator
    {
        public int Add(int addendOne, int addendTwo)
        {
            return addendOne + addendTwo;
        }
    }
}
```

With all of this in place you can now build the solution, which will automatically kick off our unit tests. Once the build completes successfully you will see that both of our tests are passing. If you do not get the same results compare your code to mine.

#### 3. Test: Can Subtract two numbers
As part of a challenge to yourself you should pause reading this article and attempt to implement a two number subtraction behavior following TDD. I find that I learn best when I have the ability to demonstrate my skills through a concrete example.

**Stop here if you want a challenge, then resume once you are finished.**

Well I hope that you took on the challenge. If you did I would like to know how that went, please leave a comment to let me know if perhaps my instructions were spot on or if I should have hammered something a little further. Introducing subtraction is very similar to our addition code so we should have no trouble adding this behavior. First the unit test to ensure our behavior is as expected:

```csharp
        [Fact]
        public void CanSubtractTwoNumbers()
        {
            const int three = 3;
            const int two = 2;
            const int expectedResult = 1;

            var actualResult = calculator.Subtract(three, two);

            Assert.Equal(expectedResult, actualResult);
        }
```

And then we introduce the functionality.

```csharp
        public int Subtract(int minuend, int subtrahend)
        {
            return minuend - subtrahend;
        }
```

Build your solution and all three of our tests should pass. Congratulations you have mastered the concept of TDD and are making good strides on implementing a Calculator. Here is the final source code for both files so that you can compare to mine.


```csharp
namespace Calculator.UnitTests
{
    using System;
    using Xunit;

    public class CalculatorTests : IDisposable
    {
        private Calculator calculator;

        public CalculatorTests()
        {
            calculator = new Calculator();
        }

        public void Dispose()
        {
            calculator = null;
        }

        [Fact]
        public void CanCreateNewCalculator()
        {
            var calculator = new Calculator();
            Assert.NotNull(calculator);
        }

        [Fact]
        public void CanAddTwoNumbers()
        {
            const int one = 1;
            const int two = 2;
            const int expectedResult = 3;

            var actualResult = calculator.Add(one, two);

            Assert.Equal(expectedResult, actualResult);
        }

        [Fact]
        public void CanSubtractTwoNumbers()
        {
            const int three = 3;
            const int two = 2;
            const int expectedResult = 1;

            var actualResult = calculator.Subtract(three, two);

            Assert.Equal(expectedResult, actualResult);
        }
    }
}
```

```csharp
namespace Calculator
{
    public class Calculator
    {
        public int Add(int addendOne, int addendTwo)
        {
            return addendOne + addendTwo;
        }

        public int Subtract(int minuend, int subtrahend)
        {
            return minuend - subtrahend;
        }
    }
}
```

## You can continue this forever
At this point our solution is very stable. Following the TDD approach it would be very reasonable to introduce additional features. I can think of adding support for multiplication, division, and of course all of the previous operations for N-number of numbers. I will stop the example here as I think that I have fully demonstrated how to develop a project using TDD. If you want further challenges for yourself, I do encourage you to implement some of the features that I called out above. 

## Further reading
If you found this article interesting perhaps you would like to check out my blog [Computer Science for Humans](https://kevinkabatra.wordpress.com/), where I write articles on anything that interests me. Is there a Computer Science topic that you would like explained or researched? If so let me know in the comments.

## Consider sponsoring
If you found this helpful please consider sponsoring additional content. You can sign up to be a sponsor through [GitHub Sponsors](https://github.com/sponsors/kevinkabatra) and there are a few tiers of support that you can choose from. Any contributions would be greatly appreciated.