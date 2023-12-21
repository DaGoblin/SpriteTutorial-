---
description: Adding Animations to our program
---

# Adding Animations

Now that we have our animation configuration file ensure it is in the Resources\animations folder. We will clean up our code a little by creating a new class for our hero and moving a lot of our code over there. For simplicity in this demo, we will keep all classes in one file, but the best practice would be to create separate files for these classes.

Create our new Hero class passing in the window and game to the constructor.&#x20;

```csharp
public class Hero
{
    private Window _gameWindow;
    private Game _TheGame;
    private Sprite _sHero;
    public Hero(Window gameWindow, Game Game)
    {
        _gameWindow = gameWindow;C#
        _TheGame = Game;   
    }
}
```

Next, add our bitmap, sprite and animation script. You will notice we pass the animation script to the sprite CreateSprite function. We are also going to set the animation and starting location.&#x20;

```csharp
Bitmap _PlayerAll = new Bitmap("PlayerBMP","Character.png");
_PlayerAll.SetCellDetails(64,64,13,21,273);
AnimationScript PlayerAnim = SplashKit.LoadAnimationScript("WalkingScript", "Hero.txt");
_sHero = SplashKit.CreateSprite("Player1", _PlayerAll, PlayerAnim);
_sHero.StartAnimation("FaceForward");
_sHero.MoveTo(100, 150);
```

In our Game class, add a local private _Hero variable and create a new_ \_Hero in the Game constructor

```csharp
private Hero _Hero;
.......................................
_Hero = new Hero(_gameWindow,this);
```

Now, if we run our program, you will notice we get two heroes, but the new one doesn't do anything. To fix this, we will move the Handel Input, Gravity and StayOnWindow methods to our Hero class and add an Update method plus, we will remove the old hero code from the game constructor and adjust the game update method. In the end, the full game code should look like this:

```csharp
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
    }

    public void Update()
    {
        HandleInput();
        Gravity();
        StayOnWindow();
    }
    public void HandleInput()
    {
        int Speed = 4;
        if(SplashKit.KeyDown(KeyCode.LeftKey))
        {
            _sHero.Dx = -Speed;
            _sHero.HideLayer(0);
            _sHero.ShowLayer(1);
        }

        if(SplashKit.KeyDown(KeyCode.RightKey))
        {
            _sHero.Dx = Speed;
            _sHero.HideLayer(0);
            _sHero.ShowLayer(2);
        }

        if(SplashKit.KeyReleased(KeyCode.LeftKey))
        {
            _sHero.Dx = 0;
            _sHero.HideLayer(1);
            _sHero.ShowLayer(0);
        }

        if(SplashKit.KeyReleased(KeyCode.RightKey))
        {
            _sHero.Dx = 0;
            _sHero.HideLayer(2);
            _sHero.ShowLayer(0);
        }

        if(SplashKit.KeyDown(KeyCode.SpaceKey))
        {
            const float Jump = -8;
            _sHero.Dy = Jump;
        }

    }

    public void Gravity()
    {
        const float G = 1; //Gravity
        const float TV = 10;//terminal Velocity
        if(_sHero.Dy<TV) _sHero.Dy += G;
    }

    void StayOnWindow()
    {
        int margin = 1;
        if (_sHero.X < 0 + margin) _sHero.X = margin;
        if (_sHero.X > _gameWindow.Width - _sHero.Width - margin) _sHero.X = _gameWindow.Width - _sHero.Width - margin;
        if (_sHero.Y > _gameWindow.Height - _sHero.Height - margin) _sHero.Y = _gameWindow.Height - _sHero.Height - margin;
    }      
}
```

Running your code will result in some strange behaviour as we no longer have our layers, and we are yet to set up the animation call functions.&#x20;
