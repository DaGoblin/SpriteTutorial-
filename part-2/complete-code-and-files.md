# Complete Code & Files

This brings us to the end of this tutorial thank you for taking the time to work along with me. Hopefully, you learnt some cool new features and way's to use Splashkit.&#x20;

Below is our complete and final code also please take a look at the next page for acknowledgments and sources for assets used in this game if you are interested in generating your own characters.&#x20;

## Assets and Code

{% file src="../.gitbook/assets/SpriteDemo_Stage2.zip" %}

## Full game Code

```csharp
using System;
using SplashKitSDK;
using System.Collections.Generic;

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
    private Hero _Hero;
    public List<Sprite> _sPlatforms = new List<Sprite>();
    private List<Adversary> _Adversary = new List<Adversary>();
    public Game(Window gameWindow)
    {
        _gameWindow = gameWindow;
        _Hero = new Hero(_gameWindow,this);

        Bitmap bGround = new Bitmap("TheGround", "Ground.png");
        Bitmap bPlatform = new Bitmap("Platform","Platform.png");

        _sPlatforms.Add(SplashKit.CreateSprite("TheGround", bGround));
        _sPlatforms[0].MoveTo(0,560);

        _sPlatforms.Add(SplashKit.CreateSprite("Platform1", bPlatform));
        _sPlatforms[1].MoveTo(200,400);

        _sPlatforms.Add(SplashKit.CreateSprite("Platform2", bPlatform));
        _sPlatforms[2].MoveTo(500,200);

        _sPlatforms.Add(SplashKit.CreateSprite("Platform3", bPlatform));
        _sPlatforms[3].MoveTo(150,100);

        NewAdversary(205,320);
        NewAdversary(505,120);
        NewAdversary(155,20);
        
    }
    
    public void Update()
    {
        SplashKit.ProcessEvents();
        _gameWindow.Clear(Color.LightSkyBlue);
        SplashKit.DrawAllSprites();
        SplashKit.UpdateAllSprites();
        _Hero.Update();
        foreach(Adversary a in _Adversary)
        {
            a.update();
        }
        KillAdversaries();
        _gameWindow.Refresh(60);
    }

    public void NewAdversary(float X, float Y)
    {
        Adversary newAdversary = new Adversary(_gameWindow,this,X,Y);
        _Adversary.Add(newAdversary);
    }

    public void KillAdversaries()
    {
        List<Adversary> KillList = new List<Adversary>();
        foreach(Adversary a in _Adversary)
        {
            if(a.isDead) KillList.Add(a);
        }

        foreach(Adversary a in KillList)
        {
            _Adversary.Remove(a);
        }

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
    private float _Bottom
    {
        get { return _sHero.Y + _sHero.Height; }
    }
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
        CollisionCheck();
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

    public void CollisionCheck()
    {
        foreach (Sprite p in _TheGame._sPlatforms)
        {
            Rectangle rect = p.CollisionRectangle;
            if (SplashKit.SpriteCollision(_sHero, p))
            {   
                //Bottom of Platform check
                if (SplashKit.RectangleBottom(rect) >= _sHero.Y-5 && SplashKit.RectangleTop(rect)  <=_sHero.Y)
                {
                    _isFalling = true;
                    _isJumping = true;
                    _sHero.Y = SplashKit.RectangleBottom(rect);
                } 

                //Top of Platform check          
                if (SplashKit.RectangleTop(rect) <=_Bottom &&  SplashKit.RectangleBottom(rect) >= _Bottom)
                {
                    _isFalling = false;
                    _isJumping = false;
                    _sHero.Y = Convert.ToSingle(rect.Y - _sHero.Height + 1);
                }
            }
        }
    }      
}

public class Adversary
{
    private Sprite _sAdversary;
    public bool isFalling { get; set; }
    private Window _gameWindow;
    private Game _TheGame;
    public bool isDead{get; private set;}

    private float _Bottom
    {
        get { return _sAdversary.Y + _sAdversary.Height; }
    }

    public Adversary(Window gameWindow, Game Game, float X, float Y)
    {
        _gameWindow = gameWindow;
        _TheGame = Game;
        Bitmap _PlayerAll = new Bitmap("Adversary", "Adversary.png");
        _PlayerAll.SetCellDetails(64, 64, 13, 21, 273);
        AnimationScript PlayerAnim = SplashKit.LoadAnimationScript("WalkingScript", "Adversary.txt");
        _sAdversary = SplashKit.CreateSprite(_PlayerAll, PlayerAnim);
        _sAdversary.MoveTo(X, Y);
        isFalling = true;
        _sAdversary.StartAnimation("WalkRight");
    }

    public void update()
    {
        CollisionCheck();
        Gravity();
    }

    private void Gravity()
    {
        const float G = 5; //Gravity
        const float TV = 10;//terminal Velocity
        if (isFalling)
        {
            if (_sAdversary.Dy < TV) _sAdversary.Dy += G;
        } 
    }

    public void CollisionCheck()
    {
        foreach (Sprite p in _TheGame._sPlatforms)
        {
            Rectangle rect = p.CollisionRectangle;
            if (SplashKit.SpriteCollision(_sAdversary, p))
            {
                if (SplashKit.RectangleBottom(rect) >= _sAdversary.Y-5 && SplashKit.RectangleTop(rect)  <=_sAdversary.Y)
                {
                    isFalling = true;
                    _sAdversary.Y = SplashKit.RectangleBottom(rect);
                }
                
                if (SplashKit.RectangleTop(rect) <=_Bottom &&  SplashKit.RectangleBottom(rect) >= _Bottom)
                {
                    isFalling = false;
                    _sAdversary.Y = Convert.ToSingle(rect.Y - _sAdversary.Height + 1);
                }

                if(_sAdversary.CenterPoint.X >= SplashKit.RectangleRight(rect))
                {
                    _sAdversary.StartAnimation("WalkLeft");
                }

                if(_sAdversary.CenterPoint.X <= SplashKit.RectangleLeft(rect))
                {
                    _sAdversary.StartAnimation("WalkRight");
                }

            }
        }
    }
}
```
