# Logical Multilayered Architecture with Unity
Developing a solution using a [layered architecture](https://en.wikipedia.org/wiki/Multitier_architecture#Layers) approach was not a concept that I was familiar with a few years ago. However it has been one of the few concepts that makes me completely rethink how I approach a new development project. For those that are not familiar with this approach I will quickly explain what it aims to do. Depending on your solution you would typically have three to four layers. 
* Presentation: the user interface or view layer that the end-user will interact with.
* Application | Platform: This is a service layer that is typically self-contained and contains references to platform specific black-boxes. Are there specific assemblies you need to reference for your platform? All of that related code should sit on this layer.
* Business | Domain: This layer is dedicated to code that can be moved between platforms with ease. It contains logic and terminology specific to the industry | topic that it is referencing.
* Data Access: Supports the business layer by controlling access to the persistent data (file systems, databases, etc.)

## Future proof cross-platform development is the new hotness
Developing your solution so that all of your eggs are in the hands of a third party vendor | platform is an `old and busted` practice. I want to present a real-world use case for you to consider; while this article will be focusing mainly on the benefits of this architecture with game development, this use case will come from my experience in developing an Enterprise Resource Planning (ERP) solution. With my current employer we work on a very large platform, Microsoft Dynamics 365, and we maintain our solution using a layered approach to ease our maintenance and protect ourselves for the future.

Developing on Dynamics involves extending some of the existing software and offering new functionality at the same time. In order to do this some of our code needs to know about the platform, this is our combined `Presentation and Application` layer. New forms and event handlers live in this area of the code. Whenever Microsoft requires us to upgrade in the future, this is the code at risk of being rewritten or possibly abandoned altogether. This is exactly the scenario that my employer found itself in when Microsoft offered a new product, D365, to be the new hotness when compared to its AX 2012 offering. The code was not immediately compatible and required hiring my team to port over the previous solution. What a large expense, especially because there was no separation of layers at this time.

We began to introduce a business layer that used common terms and common logic amongst our processes. For example if you have a concept of an object, lets say a recipe to make food, what is the correct number of classes | data models to represent this concept? Like the Highlander, I would argue that there can only be one and everyone should know who that is. If other modules need to reference a recipe they should use a common assembly for that purpose. If you write unit tests against this recipe consider how bug proof and stable that code would be. An entire organization leveraging a common business model would enable software written by disconnected teams to easily integrate into a singular solution.

Now when Microsoft offers future updates, or perhaps if we want to offer our solution on a competing platform, none of the business logic would need to be changed. You just need to update the application layer to please the platform gods. Consider that previous thought for game development, what if the majority of your code could be ported over to a new game engine with no refactoring required?

## The grass is generally greener on the other side
Developing in Dynamics and Unity I have found many similarities. One surprising commonality is the lack of NuGet support for their platforms. I have to imagine that adding this support is not a trivial task, otherwise I would presume that they would offer this fantastic feature. If you develop your entire solution inside their specific platforms you would have to give up access to the package manager and its robust community. Do you recall the dark times before NuGet? Where you had to manage all of the assemblies yourself? And all of the assemblies that your referenced assemblies referenced? And recursively until you found an individual turtle at the bottom?

Within my business layer I make sure that I use NuGet to bring in anything that I need. So far I have just referenced [Stateless](https://www.nuget.org/packages/Stateless/) or my own [NuGet packages](https://www.nuget.org/packages?q=kabatra). Then using `dotnet publish` I can make sure that every assembly that I need is automatically copied into my Unity reference folder on each successful build. This makes development a breeze especially if I need to update a NuGet package that I am dependent upon, without this system I would have to manually update the reference within Unity for each modified assembly and then update my source. Instead all I do is update my business logic and add any fancy unit test that I want, start a build, and immediately begin editing my application layer.

Speaking of unit tests. Have you ever attempted to create unit tests within Unity? This is not a trivial task until you fully understand what Unity needs you to do. However introducing unit testing in a .NET standard library takes a couple of moments, and you can probably have running unit tests within 15 minutes from starting the task. I still leverage unit tests within Unity, but the coverage in my business layer was a breeze to implement. One quirk specific to Unity is that certain code only runs when a new frame is being processed. How long is each frame? Well that depends on how fast your machine is, and how much code you are making Unity process. Also you cannot control the order of execution between the various objects that are waiting for this update. This loose coupling makes testing an objects initialization rather cumbersome, want to test similar logic in a business layer? Consider the following example:

```csharp
    [Fact]
    public void CanMakeFancyObject()
    {
        var myFancyObject = new FancyObject();
        Assert.NotNull(myFancyObject);
    }
```

Granted the above is just a simple example, but it took me less than a minute to write all of it out. That includes renaming the Fancy Object because I originally wanted to call it a pseudo-object. Less than a minute and I had an internal argument about what to name something? To be fair, I think I should give you a similarly simple example for Unity.

```csharp
    private GameObject fancyObjectInstace;

    [SetUp]
    public void Setup()
    {
        var fancyObjectPrefab = Resources.Load<GameObject>("FancyOjectRepo/Prefabs/FancyObject");
        var fancyObjectInstance = MonoBehaviour.Instantiate(fancyObjectPrefab);
    }

    [TearDown]
    public void Teardown()
    {
        Object.Destroy(fancyObjectInstance);
    }

    [UnityTest]
    public void CanMakeFancyObject()
    {
        yield return new WaitForSeconds(0.1f);

        var fancyObject = GameObject.FindObjectOfType<FancyObject>();
        UnityEngine.Assertions.Assert.IsNotNull(fancyObject);
    }
```

Wow. The first example was a clean and concise seven line poem, while a similar example for Unity took twenty-four lines (with spaces). In addition, you will have to make sure that you have a `Prefab` stored and ready for the FancyObject. You also need to make sure that any changes to the Prefab have been applied or you will not be able to access them within the test. Then you need to make sure that you account for cleaning up your `GameObject`, lest another test could stumble upon it. Finally, the worst bit is waiting for the `Start` and possibly the `Update` methods to process. So far I have found a lot of success with `WaitForSeconds` but that is a dangerous game.

## Look on my works
I think that I have concluded the sales portion of this article, so now I want to show this in practice. Warning... lots of code to follow. Lets consider a `Player`, in this example we are making a player for a single player game. We only want there to be one player, and we want to track their health and score using this object. Such an implementation could look like this:

#### Business | Domain Layer

```csharp
namespace BlockBreaker.Logic.Player.DataModel
{
    using BlockBreaker.Logic.Player.PlayerHealth;
    using BlockBreaker.Logic.Player.PlayerScore;
    using Kabatra.Common.Singleton;

    /// <summary>
    ///     The player.
    /// </summary>
    public class Player : SingletonBase<Player>
    {
        public PlayerHealth Health;
        public PlayerScore Score;

        /// <summary>
        ///     Gets or creates an instance of the Player.
        /// </summary>
        /// <returns></returns>
        public static new Player GetOrCreateInstance()
        {
            var player = SingletonBase<Player>.GetOrCreateInstance();
            player.Health = PlayerHealth.GetOrCreateInstance();
            player.Score = PlayerScore.GetOrCreateInstance();

            return player;
        }
    }
}
```

Best to not worry about the details of the Singleton base class, but it is [open sourced](https://github.com/kevinkabatra/Kabatra.Common.Singleton) if you are interested. From here we would implement the health and score classes, which are obviously separate classes from the player to prevent it from becoming a monolithic deity. The health class is a straight-forward implementation of an interface with controls for modifying and getting a player's health.

```csharp
namespace BlockBreaker.Logic.Player.PlayerHealth
{
    using Kabatra.Common.Singleton;

    public class PlayerHealth : SingletonBase<PlayerHealth>, IPlayerHealth
    {
        private const int maxPlayerHealth = 15;
        private int defaultPlayerHealth = 3;
        private int playerHealth;

        /// <summary>
        ///     Constructor.
        /// </summary>
        public PlayerHealth()
        {
            playerHealth =  defaultPlayerHealth;
        }

        /// <inheritdoc/>
        public void AddDamage(int damage = 1)
        {
            playerHealth -= damage;
        }

        /// <inheritdoc/>
        public int GetPlayerHealth()
        {
            return playerHealth;
        }

        /// <inheritdoc/>
        public void IncreasePlayerHealth()
        {
            if((defaultPlayerHealth + 1) > maxPlayerHealth)
            {
                return;
            }

            defaultPlayerHealth++;
            playerHealth = defaultPlayerHealth;
        }

        /// <inheritdoc/>
        public void ResetPlayerHealth()
        {
            playerHealth = defaultPlayerHealth;
        }
    }
}
```

The implementation of player score is very similar in concept to player health. With refactoring it could be possible to combine these objects into a singular class with a generic defining what we are actually tracking. With that said, I think this implementation will be very easy to maintain in the future, or add additional features into.

```csharp
namespace BlockBreaker.Logic.Player.PlayerScore
{
    using Kabatra.Common.Singleton;

    /// <inheritdoc/>
    public class PlayerScore : SingletonBase<PlayerScore>, IPlayerScore
    {
        private int score;

        /// <inheritdoc/>
        public virtual void AddToScore(int points = 1)
        {
            score += points;
        }

        /// <inheritdoc/>
        public int GetScore()
        {
            return score;
        }

        /// <inheritdoc/>
        public void ResetScore()
        {
            score = 0;
        }
    }
}

```

#### Unity Layer
In Unity I chose to implement everything into a singular class and really it was just because it was convenient as I added the interfaces. I also like the way that the code reads when I type `player.AddScore` when compared to `player.Score.AddScore`. I am aware though that I have created a god-class here. It would be very straightforward to separate these into three distinct classes, especially because we have already done so within our business layer. 

You will notice, if you actually read this class, that most of the implementations first call their business layer counterpart and then do some Unity specific logic. Adding damage to a player for example will cause the player's health to decrease on the screen. There is no reason for the business layer to have an understanding on how this information is presented, just what the information is.

```csharp
using BlockBreaker.Logic.Player.PlayerHealth;
using BlockBreaker.Logic.Player.PlayerScore;
using Logic = BlockBreaker.Logic.Player.DataModel.Player;
using System.Collections.Generic;
using System.Linq;
using UnityEngine;
using UnityEngine.UI;

/// <summary>
///     The Player.
/// </summary>
public class Player : Singleton<Player>, IPlayerHealth, IPlayerScore
{
    public Heart heartOne;
    public Heart heartTwo;
    public Heart heartThree;

    public Heart continueHeartFour;
    public Heart continueHeartFive;
    public Heart continueHeartSix;
    public Heart continueHeartSeven;
    public Heart continueHeartEight;
    public Heart continueHeartNine;
    public Heart continueHeartTen;
    public Heart continueHeartEleven;
    public Heart continueHeartTwelve;
    public Heart continueHeartThirteen;
    public Heart continueHeartFourteen;
    public Heart continueHeartFifteen;

    [SerializeField] private Text playerScoreDisplay;
    
    private Logic playerLogic;
    private List<Heart> hearts;

    /// <summary>
    ///     Damages the player's health.
    /// </summary>
    /// <param name="damage"></param>
    public void AddDamage(int damage = 1)
    {
        playerLogic.Health.AddDamage(damage);
        HandleDamage();
    }

    /// <summary>
    ///     Adds to the player's score.
    /// </summary>
    /// <param name="points"></param>
    public void AddToScore(int points = 1)
    {
        playerLogic.Score.AddToScore(points);
        UpdateScoreDisplayToShowCurrentScore();
    }

    /// <summary>
    ///     Returns the health.
    /// </summary>
    /// <returns></returns>
    public int GetPlayerHealth()
    {
        return playerLogic.Health.GetPlayerHealth();
    }

    /// <summary>
    ///     Returns the score.
    /// </summary>
    /// <returns></returns>
    public int GetScore()
    {
        return playerLogic.Score.GetScore();
    }

    /// <summary>
    ///     Increases the player's health up to the hard coded max.
    /// </summary>
    public void IncreasePlayerHealth()
    {
        playerLogic.Health.IncreasePlayerHealth();

        var nextHeartToEnable = hearts.Where(heart => heart.isEnabled == false).First();
        nextHeartToEnable.isEnabled = true;

        SetEnabledHeartsToActive();
    }

    /// <summary>
    ///     Resets the player's health.
    /// </summary>
    public void ResetPlayerHealth()
    {
        playerLogic.Health.ResetPlayerHealth();
        SetEnabledHeartsToActive();
    }

    /// <summary>
    ///     Resets the score back to zero.
    /// </summary>
    public void ResetScore()
    {
        playerLogic.Score.ResetScore();
        UpdateScoreDisplayToShowCurrentScore();

    }
    /// <summary>
    ///     Start is called before the first frame update.
    /// </summary>
    private void Start()
    {
        playerLogic = Logic.GetOrCreateInstance();
        UpdateScoreDisplayToShowCurrentScore();

        hearts = new List<Heart>
        {
            heartOne,
            heartTwo,
            heartThree,
            continueHeartFour,
            continueHeartFive,
            continueHeartSix,
            continueHeartSeven,
            continueHeartEight,
            continueHeartNine,
            continueHeartTen,
            continueHeartEleven,
            continueHeartTwelve,
            continueHeartThirteen,
            continueHeartFourteen,
            continueHeartFifteen
        };

        EnableHeartsForDefaultHealth();
        SetEnabledHeartsToActive();
    }

    /// <summary>
    ///     Enables the hearts to match the Player's health.
    /// </summary>
    private void EnableHeartsForDefaultHealth()
    {
        var health = GetPlayerHealth();
        for (int iterator = 0; iterator < health; iterator++)
        {
            hearts[iterator].isEnabled = true;
        }
    }

    /// <summary>
    ///     Player has perished.
    /// </summary>
    private void GameOver()
    {
        SceneLoader.LoadGameOverScene();
    }

    /// <summary>
    ///     Executes logic based on amount of damage to the player.
    /// </summary>
    /// <remarks>
    ///     Need to update the hearts multiple times due to AddDamage
    ///     supporting adding multiple points of damage at one time.
    /// </remarks>
    private void HandleDamage()
    {
        SetAllHeartsToNotActive();
        var health = GetPlayerHealth();
        
        for (int iterator = 0; iterator < health; iterator++)
        {
            hearts[iterator].gameObject.SetActive(true);    
        }

        if(health == 0)
        {
            GameOver();
        }
        else if(health < 0)
        {
            // This only occurs during unit testing, where extra balls hit the collider before they are destroyed.
            playerLogic.Health.ResetPlayerHealth();
            HandleDamage();
        }

        // Find the ball each time we need to use the position handler. The original ball will be destroyed when the level that
        // created it is destroyed, and if this logic is in the Start method the position handler will be null for the next level.
        var ball = Ball.Get();
        var ballPosition = ball.GetComponent<BallPositionHandler>();
        ballPosition.ResetBall();
    }

    /// <summary>
    ///     Disable all hearts temporarily, this will allow simpler code to be used to enable required hearts
    /// </summary>
    private void SetAllHeartsToNotActive()
    {
        hearts.ForEach(heart => heart.gameObject.SetActive(false));
    }
    /// <summary>
    ///     Set any heart that has been enabled for the player to be active.
    /// </summary>
    private void SetEnabledHeartsToActive()
    {
        SetAllHeartsToNotActive();

        foreach (var heart in hearts.Where(heart => heart.isEnabled))
        {
            heart.gameObject.SetActive(true);
        }
    }

    /// <summary>
    ///     Updates the heads up display to show the current score.
    /// </summary>
    private void UpdateScoreDisplayToShowCurrentScore()
    {
        playerScoreDisplay.text = GetScore().ToString();
    }
}

```

It is also important to point out that in Unity all Game Objects must inherit from `MonoBehavior` and due to no support for multi-inheritance I must leverage interfaces to tell Unity what needs to be implemented for the business layer. Additionally if you are interested in how I implemented a Singleton within my Unity Layer you should check out the source in the [repository](https://github.com/kevinkabatra/UnityTutorialBlockBreaker/blob/main/Assets/UnityLayer/PlaySpace/Common/Singleton.cs).

## So long and thanks for all the fish
In conclusion I think that it is very easy to introduce this concept into a new or fairly new project in order to future proof your solution. You can introduce this concept into a legacy system, but you will need to be more measured in your approach to ensure stability. I think that the practice of using a multilayered architecture can apply to other forms of development rather than just Unity as I have offered an example for its usage in an internal ERP. 

I hope that you have enjoyed this article along with the random bits of humor sprinkled throughout. Have you tried to implement a project following this approach before? If so please let me know how that went and if you ran into any struggles. 

## Further reading
If you found this article interesting perhaps you would like to read how I:
* [supported localization within Unity using C# Resources](https://kevinkabatra.wordpress.com/2019/11/10/supporting-localization-in-unity-using-c-resources/).
* [used NuGet with Unity](https://kevinkabatra.wordpress.com/2020/11/14/using-nuget-with-unity/).
* Examples of multilayered architecture in Unity:
    * [Labyrinth](https://github.com/kevinkabatra/UnityTutorialLabyrinth). This was my first implementation following this pattern.
        * Additionally it contains examples of: localization, and NuGet
    * [Block Breaker](https://github.com/kevinkabatra/UnityTutorialBlockBreaker). This was my second attempt at this, when comparing to the previous implementation I think you will notice that it is much cleaner.

Also you can check out my blog [Computer Science for Humans](https://kevinkabatra.wordpress.com/), where I write articles on anything that interests me.

## Consider sponsoring
If you found this helpful please consider sponsoring additional content. You can sign up to be a sponsor through [GitHub Sponsors](https://github.com/sponsors/kevinkabatra) and there are a few tiers of support that you can choose from. Any contributions would be greatly appreciated.