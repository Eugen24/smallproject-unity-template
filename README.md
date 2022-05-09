# Template for creating SmallProject Games Easy

- Download Package
Get latest release here: https://github.com/Eugen24/smallproject-unity-template/releases

## General Conventions
#### Project Conventions:
- Every project should be in Assets/_Project
- In Build settings first scene should be FirstScene.unity
- Every scene should have System.prefab > Note! Only if you are developing Non Multiplayer Game
- All scriptable objects should be in Assets/_Project/Scripts/Configs
- Git ignore: https://github.com/Eugen24/highproject-template/blob/main/.gitignore

## Library includes:
- Scene progression 
- Pool
- Default SDKs for publishing (Facebook, GA, etc...)
- Other small tools for math and animations
- Easy Save - The Complete Save Data & Serialization Tool > Guides To Getting started: https://docs.moodkie.com/product/easy-save-3/es3-guides/
- Effects

## Pool
Pool is a concept for optimisation of object-reusability. The idea is to store GameObjects in memory instead of destroying them.
The sample of Template implementation is in Template/Samples/Pool

How it works, you need to create a prefab with some scripts (out case PoolObjectMono). PoolObjectMono should implement IPoolObject.

Here is example:
```c#
using Template.Scripts.Utils;
using UnityEngine;

namespace Template.Samples.Pool
{
    public class PoolObjectMono : MonoBehaviour, IPoolObject
    {
        public void OnEnterPool()
        {
            //fake destroy
            gameObject.SetActive(false);
        }

        public void OnExitPool()
        {
            //fake init
            gameObject.SetActive(true);
        }
    }
}
```

And there is a controller that handles memory management:

```c#
using System;
using System.Collections;
using Template.Scripts.Utils;
using UnityEngine;

namespace Template.Samples.Pool
{
    public class PoolController : MonoBehaviour
    {
        public PoolObjectMono Prefab;

        void Update()
        {
            if (Input.GetKeyUp(KeyCode.Space))
            {
                var obj = Pool<PoolObjectMono>.GetFromPool(() => Instantiate(Prefab));
                StartCoroutine(DestroyObjectAfterTime(obj));
            }
        }

        private IEnumerator DestroyObjectAfterTime(PoolObjectMono obj)
        {
            yield return new WaitForSeconds(2f);
            Pool<PoolObjectMono>.SetInPool(obj);
        }

        private void OnDestroy()
        {
            Pool<PoolObjectMono>.ForceClearAll();
        }
    }
}

```

Note! Pool-Clearing is handled also by controller. Auto-cleaning on change scene probably will be in future releases.

## Other Utils >

#### InputNormalized
InputNormalized.GetMousePos() - gets normalize Input relative for Input that is independent from resolution

#### TimeServiceMono > Local Utilization Only!
Custom Update for optimisation, please you use it only if there is a need in optimisation. 
Any exception during IUpdate can break other scripts, please don't change deltaTime!!!

```c#
public class ExampleScript : InjectedMono, IUpdater
{
    [In] protected TimeServiceMono _time;

    public void Start()
    {
        _time.Register(this);
    }
    public void OnUpdate(ref float deltaTime)
    {
        //Here should be update code
    }
}
```
#### Effects
Effects are available in Template/Effects. List of effects:
- Outline
- Skybox (Vadim request). Check Template/Samples/GradientSkybox

#### MathUtils
Some basic math that is not available in Unity:
- Rotation Wrap (0-360)

#### BasicCameraController
Default camera controller, add it when you need some camera that can be controlled by touch.

#### Haptic
Need to call HapticHandler.SmallHaptic();

There is a default UI that handles Disable/Enable haptic, 
please check it prefab 'Canvas-Settings' (also present in scene Level1.unity)


#### UiHelper

UiHelper.IsOnUi() can be called to check if current mouse position is touching UI (with rayCastTarget enable).
Ui Raycast also can be called/using from script.

#### StringUtils

Some string utils to work with float and int (in future support for long).
Please check the script is self-explaining. 

#### NaughtyAttributes
This library includes NaughtyAttributes, please check there doc: 
https://github.com/dbrizov/NaughtyAttributes

#### Default Images

In Template/Sprites you can find default logo and some other useful sprites

#### Restart scene in editor
You can restart scene in editor by using CTRL+R

#### Clear Player Pref in editor
You can clear PlayerPrefs(SaveSystem) in editor by using CTRL+DELETE > Note! Delete only you saved data in PlayerPrefs!

#### Camera bounds (area that camera see, ony orthographic support)
Camera.OrthographicBounds()

#### Wait System
You can inject WaitSystem and call WaitRoutine(float seconds, System.Action callBack)
