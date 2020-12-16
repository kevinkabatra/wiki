# Introducing an Expansion with Unity
Recently I completed another Unity development tutorial, announcement [here](https://kevinkabatra.wordpress.com/2020/11/30/unity-tutorial-block-breaker/), and I uploaded the game so that others could "look upon my works". Surprisingly it found a fervent following amongst a friend's household, I was told that "my dinner conversation is revolving around your game". He then reported that his children wanted to see improvements to the base game, as all gamers today are used to an ever-improving product after initial delivery via updates and downloadable content. As you will see by the included conversation below I was very conflicted about returning to this tutorial, in my mind it was just a class and that course had been completed. The next game course is styled as a vertically scrolling shooter (Space Defenders, Sky Force, 1942) and I am looking forward to building that. So any extra work on `Block Breaker` would only introduce a delay to my progress, hold me back if you will. 

![Conversation that sparked the expansion](.\IntroducingAnExpansionWithUnityAssets\ConversationThatSparkedTheExpansion.png)

But he is my friend and his kids seem to generally enjoy the initial product. I decided that I should look at this as gaining experience updating a released product and to see what that work flow looked like. Out of the initial expansion ideas that we spoke about, I thought that introducing explosions would be the most fun. This article will explain how I leveraged `clean coding` practices during my initial development and my expansion work to be able to introduce this new functionality to the public three days after we first discussed it. 

## Timeline
* Tuesday Dec 8 06:17 PM: The topic of expanding the game is brought up.
* Tuesday Dec 8 08:58 PM: Initial commit to the git repository. I have tracked down the assets from `Open Game Art` that I will use for this work. This commit is just updating the ReadMe to give proper credit to the artists. I am pretty sure that I also had dinner during this time, which impeded this progress.
* Tuesday Dec 8 10:02 PM: Introduced the explosion element, all of the artwork and sounds are working correctly. During this time I had to look up how to slice a sprite sheet in order to be used as an animation, thankfully this was actually pretty straightforward. Currently the explosion does not damage the blocks around it, and I am trying to think best how to introduce that. 
* Tuesday Dec 8 10:08 PM: I had an epiphany and realized that I could just attach a `Rigid Body` and `Collider` components to the explosion game object. Then I can let Unity figure out how to damage the blocks that the explosion touches. I wanted to go to sleep at this point, but I had to crank this out quickly lest I risk forgetting it by the time I awake.
* Wednesday Dec 9 07:41 AM: Added a testing level so that I could manipulate a scene without the risk of damaging my production level. Also introduced a block variant to be used for explosions.
* Wednesday Dec 9 08:17 PM: Fortunately I have a full time job and children of my own to bring to school. So after the previous update life prevented me from continuing on this endeavor. With this commit I had finished implementing the entire explosion feature.

All of my future commits on Wednesday and Thursday were dedicated to adding additional levels into the game and play testing to make sure that everything behaved as anticipated. I actually had completed development on Thursday and intentionally delayed the release until Friday, and that was mainly because that was the initial timeline that I had communicated publicly on LinkedIn.

## Development
So how did the development for the explosion feature get completed so fast, the answer to that can be stated in just two words: `clean code`. Code so clean that Uncle Bob would be proud. Code that was so thoroughly tested through automated tests that Kent Beck would approve.


#### Extensions through publicly available APIs
With this expansion we want a `Block` to explode once it is damaged to the point of destruction. But we only want some of the blocks to have this behavior, these exploding blocks will be represented as a distinct game object when compared to their non-volatile counterparts. In Unity this actually helps us choose which classes will be attributed to a particular game object, this will allow us to prevent the original block from worrying about spontaneous explosions. What an existence that would be.

First we will look at the parts of the `Block` class that could support us in this effort:

```csharp
public class Block : AudioPlayer<Block>
{
    public bool isDestroyed = false;

    private readonly UnityEvent blockDestroyedEvent = new UnityEvent();
    private int damageLevel = 0;

    /// <summary>
    ///     Handles the damage to the block.
    /// </summary>
    public void HandleDamage()
    {
        damageLevel++;

        switch (damageLevel)
        {
            case 1:
                spriteRenderer.sprite = blockWithOneDamage;
                break;
            case 2:
                spriteRenderer.sprite = blockWithTwoDamage;
                break;
            case 3:
                DestroyBlock();
                break;
            default:
                throw new System.NotImplementedException();
        }
    }

    /// <summary>
    ///     Handles collision.
    /// </summary>
    /// <param name="collision"></param>
    protected override void OnCollisionEnter2D(Collision2D collision)
    {
        base.OnCollisionEnter2D(collision);
        HandleDamage();
    }

    /// <summary>
    ///     Destroys the block.
    /// </summary>
    private void DestroyBlock()
    {
        AudioSource.PlayClipAtPoint(blockDestroyedAudioClip, transform.position, 2f);
        isDestroyed = true;
        Destroy(gameObject);
        blockDestroyedEvent.Invoke();
    }        
```

Above you will see that the block has a collider that will increase the damage to the block on each collision. This is helpful as we want the explosion to damage all neighboring blocks, so we will not need to introduce any code for that to work as expected. How much damage does it take to destroy the block? Well that does not matter for the explosion, in fact there is some code from that class that I omitted above simply because the explosion does not rely on it. Instead we will take a look at the `Unity Event` that is invoked during `DestroyBlock()`, `blockDestroyedEvent`.

This event has been declared as private so I think that we have some options to add the support that we are looking for. First we could introduce a publicly available static method on the `Block`, that would enable me to add a listener to the event. But there are lots of blocks so finding the particular instance would be impossible. Altering that approach we could add a publicly available instance method on the `Block`, then the explosion handler that we would add will know which block to listen to. Because of Unity magic. You could even make this approach feature agnostic by passing in the `Unity Action` to be triggered as the call back to the event. I have an example of this in the `Virtual Controller` class.

```csharp
    /// <summary>
    ///     Informs the event to trigger the callback when event is invoked.
    /// </summary>
    /// <param name="callback"></param>
    public void FireEventSubscribe(UnityAction callback)
    {
        fireEvent.AddListener(callback);
    }
```

I, however, did not want to have to add so much additional code to the `Block` class. Instead I made the event itself publicly available so that I could add a listener in an event handler class. 

```csharp
public class BlockExplosionEventHandler : MonoBehaviour
{
    Block thisBlock;

    // Start is called before the first frame update
    private void Start()
    {
        thisBlock = GetComponent<Block>();

        var explosionAction = new UnityAction(() => { DoExplosion(); });
        thisBlock.blockDestroyedEvent.AddListener(explosionAction);
    }
}
```

I am honestly not sure which approach I like more. The controller's implementation is clean as I will not have any duplicated code in the listening event handlers, but its data model has to know about adding and removing subscribers. In the case of the block, there is only one event to be called, but for the controller there could be several. Having more than one event to be called would really make managing the listeners more difficult if the data model did not handle its subscriptions. Alas I am torn, but I think I am leaning towards the controller's implementation.

I want to point out for more junior developers that this approach follows the `Open Closed Principle` where the original `Block` data model was extended rather than truly modified for this expansion. Sure we had to open up its private Unity Event, but I think that is mainly because I was not thinking about making an expansion during the initial development. I think in the future I will add more Unity Events and be sure to set them as public so that they could be expanded upon at a future date. This also follows `Single Responsibility Principle` as the `BlockExplosionEventHandler` only has one purpose and that is to cause an explosion.

#### Boom time
Now all we need to do is to implement a `DoExplosion` method to tell the game engine what should happen when this particular block explodes, if interested you can view that in full detail below.

```csharp
    private void DoExplosion()
    {
        var explosionPrefab = Resources.Load<GameObject>("Playspace/Explosion/Prefabs/BlockExplosion");
        var explosionInstance = Instantiate<GameObject>(explosionPrefab);
        explosionInstance.transform.position = gameObject.transform.position;
    }
```

The explosion was super simple to implement, especially when you realize that the code is very similar to how we are implementing our unit testing. But an issue did appear that was not originally anticipated. The `Ball` can collide with the explosion prior to the Game Object's destruction. This should have been obvious, as the explosion can collide with the neighboring blocks. It would have been rather simple to just remove that functionality and have the explosion be visual only, but I was quite attached to the idea that the explosions could affect the state of the game. Through some searching I found `Physics2D.IgnoreCollision` which allows you to tell two colliders to ignore one another. Following is my usage of this functionality:

```csharp
/// <summary>
///     The explosion when a block explodes.
/// </summary>
public class Explosion : MonoBehaviour
{
    public float delay = 0f;

    // Start is called before the first frame update
    private void Start()
    {
        PreventCollisionsWithBall();
        Destroy(gameObject, GetComponent<Animator>().GetCurrentAnimatorStateInfo(0).length + delay);
    }

    private void PreventCollisionsWithBall()
    {
        var ballCollider = Ball.Get().GetComponent<Collider2D>();
        var thisCollider = GetComponent<Collider2D>();

        Physics2D.IgnoreCollision(ballCollider, thisCollider);
    }
}
```

## Distribution of a new release
Distribution is very simple for this project. It begins by first building this project targeting the WebGL platform, once this build is complete I will create a zip-based archive containing all of the output from this build. This includes the `Build` and `TemplateData` folders as well as the `index.html` file. This artifact will be used later on to distribute the newest version of the game. I will now create a new release on the repository hosted [here](https://github.com/kevinkabatra/UnityTutorialBlockBreaker) on GitHub, I also upload the build artifact here for anyone who wants to download it directly. Currently this game is only hosted on [here](https://kevinkabatra.itch.io/unity-tutorial-block-breaker) on itchi.io, and releasing a new version is as simple as uploading the build artifact to the site and clicking the save button. You can remove the previously uploaded artifact if you want, but I tend to just set them to be hidden. This distribution model is the basis for Games as a Service, but obviously on a much smaller scale, as the users are only able to access the current version of the game.

## References:
1. Ignore Collision: https://docs.unity3d.com/ScriptReference/Physics2D.IgnoreCollision.html

## Further reading
If you found this article interesting perhaps you would like to read how I:
* [supported localization within Unity using C# Resources](https://kevinkabatra.wordpress.com/2019/11/10/supporting-localization-in-unity-using-c-resources/).
* [used NuGet with Unity](https://kevinkabatra.wordpress.com/2020/11/14/using-nuget-with-unity/).
* develop games in Unity using [logical multilayered code](https://wordpress.com/post/kevinkabatra.wordpress.com/101). Examples of this in practice:
    * [Labyrinth](https://github.com/kevinkabatra/UnityTutorialLabyrinth). This was my first implementation following this pattern.
        * Additionally it contains examples of: localization, and NuGet
    * [Block Breaker](https://github.com/kevinkabatra/UnityTutorialBlockBreaker). This was my second attempt at this, when comparing to the previous implementation I think you will notice that it is much cleaner.

Also you can check out my blog [Computer Science for Humans](https://kevinkabatra.wordpress.com/), where I write articles on anything that interests me.

## Consider sponsoring
If you found this helpful please consider sponsoring additional content. You can sign up to be a sponsor through [GitHub Sponsors](https://github.com/sponsors/kevinkabatra) and there are a few tiers of support that you can choose from. Any contributions would be greatly appreciated.