# Moving with Animations

## Using Animations

Now we have the animation sprite drawing on the window; we need to modify the HandleInput method to use the animations. An animation can be called by using sprites .`StartAnimation()` function. Update the HandleInput method so it has the following.

Notice we also have a check to see if the current animation has finished before playing the next one. Without that holding, the key down would constantly restart the animation.&#x20;

```csharp
if (SplashKit.KeyDown(KeyCode.LeftKey)) if (_sHero.AnimationHasEnded) _sHero.StartAnimation("WalkLeft");
if (SplashKit.KeyDown(KeyCode.RightKey)) if (_sHero.AnimationHasEnded) _sHero.StartAnimation("WalkRight");
if (SplashKit.KeyReleased(KeyCode.LeftKey)) _sHero.StartAnimation("FaceForward");
if (SplashKit.KeyReleased(KeyCode.RightKey)) _sHero.StartAnimation("FaceForward");
if(SplashKit.KeyDown(KeyCode.SpaceKey))
{
    const float Jump = -8;
    _sHero.Dy = Jump;
}
```

Now our hero should be able to walk around.

## Updating Jumping

Next, we are going to improve our jumping and add some functions that will be used with platforms. First, add a new method to the Hero class called Jump(). Also as some private variables and change the spacebar key down check to call Jump.

<pre class="language-csharp"><code class="lang-csharp">// Private variables
    private bool _isJumping;
    private bool _isFalling;
    private float _JumpLimit;
<strong>
</strong><strong>// Jump Method
</strong>private void Jump()
{
    const float JumpSpeed = -8;
    if (!_isJumping)
    {
        _isFalling = false;
        _isJumping = true;
        _JumpLimit = (_sHero.Y - _sHero.Height * 3);
    }

    if (_isJumping &#x26;&#x26; !_isFalling)
    {
        if (_sHero.Y > _JumpLimit)
        {
            _sHero.Dy = JumpSpeed;
        }
        else
        {
            _isFalling = true;
        }
    } 
}  

//Handle Input Update
if (SplashKit.KeyDown(KeyCode.SpaceKey)) Jump();
if (SplashKit.KeyReleased(KeyCode.SpaceKey))
 {
    isFalling = true;
    _Splayer.Dy = 0;
 }

</code></pre>

Our Jump method check that we are not already jumping and sets isJumping to True, isFalling to false and sets a jump limit three times the height of our hero. If we are already jumping we don't want to be able to start a second jump although you could modify this for a second jump or create the effect used in the flappy bird style games.&#x20;

If you have run your code you will have noticed you only got one jump in then it doesn't work anymore that's because we need something to reset our jump once we are on the ground. Since we haven't added ground yet we will modify the StayOnWindow method Y check to reset our jump.

```csharp
if (_sHero.Y > _gameWindow.Height - _sHero.Height - margin)
{
    _sHero.Y = _gameWindow.Height - _sHero.Height - margin;
    _isFalling = false;
    _isJumping = false;
}
```

We also want to modify the gravity so it becomes conditional on and only active when we are falling. Add a is falling check to the Gravity method.&#x20;

**We also want to add a `_isFalling = true;` to the Hero constructor.**

```csharp
private void Gravity()
{
    const float G = 5; //Gravity
    const float TV = 10;//Terminal Velocity
    if (_isFalling)
    {
        if (_sHero.Dy < TV) _sHero.Dy += G;
    } 
}
```

#### Full code for reference

```
using System;
using SplashKitSDK;

namespace SpriteDemo_Stage2
{
    public class Program
    {
        public static void Main()
        {
            Window gameWindow = new Window("PlatformDemo",800,600);
            Game MyGame = new Game(gameWindow);

            while ( ! gameWindow.CloseRequested )
            {
                MyGame.Update();
            }
            gameWindow.Close();
        }
    }
}

public class Game
{
    private Window _gameWindow;
    private Sprite _sHero;
    private Hero _Hero;
    public Game(Window gameWindow)
    {
        _gameWindow = gameWindow;
        _Hero = new Hero(_gameWindow,this);
    }
    
    public void Update()
    {
        SplashKit.ProcessEvents();
        _gameWindow.Clear(Color.LightSkyBlue);
        SplashKit.DrawAllSprites();
        SplashKit.UpdateAllSprites();
        _Hero.Update();
        _gameWindow.Refresh(60);  
    }
}

public class Hero
{
    private Window _gameWindow;
    private Game _TheGame;
    private Sprite _sHero;
    private bool _isJumping;
    private bool _isFalling;
    private float _JumpLimit;
    public Hero(Window gameWindow, Game Game)
    {
        _gameWindow = gameWindow;
        _TheGame = Game;

        Bitmap _PlayerAll = new Bitmap("PlayerBMP","Character.png");
        _PlayerAll.SetCellDetails(64,64,13,21,273);
        AnimationScript PlayerAnim = SplashKit.LoadAnimationScript("WalkingScript", "Hero.txt");
        _sHero = SplashKit.CreateSprite("Player1", _PlayerAll, PlayerAnim);
        _sHero.StartAnimation("FaceForward");
        _sHero.MoveTo(100, 150);

        _isFalling = true;
    }

    public void Update()
    {
        HandleInput();
        Gravity();
        StayOnWindow();
    }
    public void HandleInput()
    {
        if (SplashKit.KeyDown(KeyCode.LeftKey)) if (_sHero.AnimationHasEnded) _sHero.StartAnimation("WalkLeft");
        if (SplashKit.KeyDown(KeyCode.RightKey)) if (_sHero.AnimationHasEnded) _sHero.StartAnimation("WalkRight");
        if (SplashKit.KeyReleased(KeyCode.LeftKey)) _sHero.StartAnimation("FaceForward");
        if (SplashKit.KeyReleased(KeyCode.RightKey)) _sHero.StartAnimation("FaceForward");
        if (SplashKit.KeyDown(KeyCode.SpaceKey)) Jump();
        if (SplashKit.KeyReleased(KeyCode.SpaceKey))
         {
            _isFalling = true;
            _sHero.Dy = 0;
         }

    }

    private void Gravity()
    {
        const float G = 5; //Gravity
        const float TV = 10;//Terminal Velocity
        if (_isFalling)
        {
            if (_sHero.Dy < TV) _sHero.Dy += G;
        } 
    }

    void StayOnWindow()
    {
        int margin = 1;
        if (_sHero.X < 0 + margin) _sHero.X = margin;
        if (_sHero.X > _gameWindow.Width - _sHero.Width - margin) _sHero.X = _gameWindow.Width - _sHero.Width - margin;
        if (_sHero.Y > _gameWindow.Height - _sHero.Height - margin)
        {
            _sHero.Y = _gameWindow.Height - _sHero.Height - margin;
            _isFalling = false;
            _isJumping = false;
        }
    }
    private void Jump()
    {
        const float JumpSpeed = -8;
        if (!_isJumping)
        {
            _isFalling = false;
            _isJumping = true;
            _JumpLimit = (_sHero.Y - _sHero.Height * 3);
        }

        if (_isJumping && !_isFalling)
        {
            if (_sHero.Y > _JumpLimit)
            {
                _sHero.Dy = JumpSpeed;
            }
            else
            {
                _isFalling = true;
            }
        } 
    }      
}
```
