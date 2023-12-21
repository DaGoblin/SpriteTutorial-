# Adversary (bonus)

As a quick bonus, we are going to add some adversaries to our game but I will leave it up to you to implement your own collision check system to decide what happens if the hero comes into contact with an adversary.

## Create an Adversary

For our adversaries, we will make a new class called Adversary is will use a lot of the same conditions as the Hero class just a bit stripped back and the adversary will have its own animation file. We're taking the jumping configuration out but you might wish to keep it.

This is the full class note it is very close to the player class the notable differences are in the constructor we are taking in an XY position and starting the animation walking to the right also in the collision check you can see two new statements. This checks if the adversary reaches the end of a platform and turns them around. We have also added a `public bool isDead{get; private set;}` which can be used by the game to kill advesaries.

<pre class="language-csharp"><code class="lang-csharp">public class Adversary
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

<strong>    public Adversary(Window gameWindow, Game Game, float X, float Y)
</strong>    {
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
            if (_sAdversary.Dy &#x3C; TV) _sAdversary.Dy += G;
        } 
    }

    public void CollisionCheck()
    {
        foreach (Sprite p in _TheGame._sPlatforms)
        {
            Rectangle rect = p.CollisionRectangle;
            if (SplashKit.SpriteCollision(_sAdversary, p))
            {
                if (SplashKit.RectangleBottom(rect) >= _sAdversary.Y-5 &#x26;&#x26; SplashKit.RectangleTop(rect)  &#x3C;=_sAdversary.Y)
                {
                    isFalling = true;
                    _sAdversary.Y = SplashKit.RectangleBottom(rect);
                }
                
                if (SplashKit.RectangleTop(rect) &#x3C;=_Bottom &#x26;&#x26;  SplashKit.RectangleBottom(rect) >= _Bottom)
                {
                    isFalling = false;
                    _sAdversary.Y = Convert.ToSingle(rect.Y - _sAdversary.Height + 1);
                }

                if(_sAdversary.CenterPoint.X >= SplashKit.RectangleRight(rect))
                {
                    _sAdversary.StartAnimation("WalkLeft");
                }

                if(_sAdversary.CenterPoint.X &#x3C;= SplashKit.RectangleLeft(rect))
                {
                    _sAdversary.StartAnimation("WalkRight");
                }

            }
        }
    }
}
</code></pre>

### Animation Script

```
SplashKit Animation

//Frames are declared with an f: and contain
//the following comma separated values
//ID,CELL,DUR,NEXT
f:99,78,1,

//Multi-frame: ranges are in[]
//[a-b] = numbers from a to b inclusive
//[a,b,c] = explicit values
//[a-b,c] = combination
f:[0-8],[143-151],3,0
f:[9-17],[117-125],3,9

i:WalkRight,0
i:WalkLeft,9
i:FaceForward,99

v:[0-8],2,0
v:[9-17],-2,0
v:99,0,0
```

## Spawn Adversaries

Now to spawn our adversaries we will add a new list to the Game class a method to create Adversaries and two methods one to make them and one to kill them. Finally, we will update the constructor to add three adversaries that will land on the platforms. Also, make sure you add KillAdversaries and Adversary update to the game update method.&#x20;

```csharp
//New variable
 private List<Adversary> _Adversary = new List<Adversary>();
 
//add to Game constructor
NewAdversary(205,320);
NewAdversary(505,120);
NewAdversary(155,20);

//New Methods
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

//Add to Game Update method
KillAdversaries();
foreach(Adversary a in _Adversary)
{
    a.update();
}
```
