# Test Driven Development (TDD)
Choosing a development process is very similar to choosing a religion, once you belong to a particular camp you tend to stop considering the other alternatives as realistic possibilities. I think this approach is rather short sighted especially in an era where your source could contain a small microservice or be a cog in a massive social media network. As perpetual students of our ever-changing Computer Science it is imperative that we consider alternatives to our approaches as they arrive. In this article I aim to compare two development processes that in my opinion are very similar but are still distinct enough to warrant a comparison. There are of course many other processes and perhaps I will author future comparisons. If you are interested in that content please be sure to leave a comment with what processes you would like to see covered.

## Test Drive Development (TDD)
TDD is a fantastic buzz word, or words I guess, to place on a resume or drop in an interview. "In my latest project we successfully introduced TDD and managed to reduce world hunger at the same time". There is nothing wrong with using TDD, I am personally just averse to anything that becomes so popular that you can create a celebrity for it. Enter Kent Beck. I first read [Test Driven by Development](https://www.amazon.com/Test-Driven-Development-Kent-Beck/dp/0321146530), Kent Beck's instruction manual for TDD, in 2019 and I thought that this approach would solve all of the problems that I had run into during development. 

By this point in my career I had worked on multiple Enterprise Resource Planning (ERP) platforms and I constantly saw how complex it was to make any additions to an existing system. Touching one part of the system would have a strong chance to result in a negative effect elsewhere. Commonly these issues are referred to as regressions, and are best prevented via automated testing. Honestly since you are reading this article, I would hope that you already had that knowledge. In TDD, Kent championed writing small unit tests against an expected behavior, this code would fail as the behavior had not yet been written, but then it would enable a developer to write just enough code to satisfy the unit test's requirements. In theory this results in cleaner concise code that behaves exactly as prescribed.

Had the initial ERP projects leveraged an approach such as this novel concept I think that regressions would be completely avoidable. With TDD, no behavior can exist prior to the automated testing that will exercise it properly, therefore with each change to the system you should be able to validate every aspect with a click of a button and some patience as your tests run. Automated unit testing is amazing when it is done right. With quickly executing unit tests you will have very little friction standing between you and setting up the tests to run after each build. Whereas time consuming unit tests invokes a mental image of a popular comic strip that shows developers sword fighting while their code is compiling. This time consumption will quickly become overwhelming friction that causes the developers to run their tests infrequently, only to discover regressions and other failures far too late in the process. Therefore, quickly executing reliable unit tests are generally the foundation of any testing strategy.

### 1. Example
I learn best through doing and watching an example rather than just reading about theories, so lets take some time to create a simple project following the Test Driven Development process. In this example we will create a simple calculator that is capable of addition and subtraction. Using Visual Studio we will create the `Calculator` project which of course at the same time creates a solution of the same name. This project will target .NET Standard 2.0, so that it could be used as a library for another application in the future. Once the project has been created delete any default classes that are created, in this example my Integrated Development Environment (IDE) created an empty class called `Class1`.

Now it is time to create our testing project, as we need to have unit tests prior to coding any behavior. This project will be called `Calculator.UnitTests`, I find it helpful to have such clear naming for projects and objects, it makes navigating and maintaining a solution so much easier in the long-term. This unit test project will be based on xUnit, but you can insert your preferred unit testing framework instead. Once that is created please delete any default classes again, in this case we have `UnitTest1`.

At this point your solution should look like this:

![Solution](.\TestDrivenDevelopmentAssets\SolutionWithNewProjects.png)

#### 1a. Can create new Calculator
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

Remember, without a unit test we cannot actually add any behavior to this object. Build just the Calculator project and then update the UnitTests project to reference the Calculator project. Once this is done you should be able to build the UnitTests project now, before doing this lets open up Test Explorer and configure our unit tests to run after each build. With that set build your entire solution and watch as Test Explorer runs and passes our test.

![Test Explorer Results](.\TestDrivenDevelopmentAssets\CanCreateNewCalculatorTestPasses.png)

#### 1b. Can Add two numbers
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

#### 1c. Can Subtract two numbers

#### You can continue this forever
At this point our solution is very stable. Following the TDD approach it would be very reasonable to add additional features. I can think of adding support for multiplication, division, and of course all of the previous operations for N-number of numbers. I will stop the example here as I think that I have fully demonstrated how to develop a project using TDD.