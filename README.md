# Unity Utilities
A collection of project-agnostic Unity utilities. Each folder comes with its own assembly definition
and documentation.

## Attributes 

### [ReadOnlyAttribute](Attributes/Runtime/ReadOnlyAttribute.cs)
Creates an immutable variable in the inspector.
```csharp
[ReadOnly] public int variable; 
```

### [RenameAttribute](Attributes/Runtime/RenameAttribute.cs)
Renames a variable in the inspector.
```csharp
[Rename("More Specific Name"), SerializeField] private int variable; 
```

## Data Structures

### [MinPriorityQueue](DataStructures/Runtime/MinPriorityQueue.cs)

This is a minimum priority queue. Unity's .NET version predates .NET 6, which introduced `PriorityQueue`.

```csharp
MinPriorityQueue<int> queue = new MinPriorityQueue(); 
queue.Enqueue([2, 1]); // adds 2 and 1 to the queue
int first = queue.Dequeue(); // returns 1
queue.Any(); // returns true
```

## [Internal Debug](InternalDebug/Runtime/InternalDebug.cs)
 
This is a static `InternalDebug` wrapper class for the `UnityEngine.Debug` class.
Any call from this class will only be present when the build is either a Development Build or build in
Development Mode.

```csharp
InternalDebug.Log("Hello World!"); // Will not run in production builds unlike Debug.Log
```

*Modified from: https://github.com/JoebRogers/UnityDebug/blob/master/src/InternalDebug_NoNamespace.cs*

## Pooling 

Pooling is a common pattern in video-games in order to increase performance when instantiating many objects.
This pooling library takes hints from Unity's first (and last) open project, of which you can read their wiki 
article on pooling [here](https://github.com/UnityTechnologies/open-project-1/wiki/Object-pooling). 

### [Factory](Pooling/Runtime/Factory.cs)
Creates a factory for a component called `MyComponent`
```csharp
public class MyComponentFactory : Factory<MyComponent>
{
    public override MyComponent CreateInstance() {
        return new MyComponent();
    }
}
```

### [Component Pool](Pooling/Runtime/ComponentPool.cs)
Create a pool for the `MyComponentFactory` above.
```csharp
[CreateAssetMenu(fileName = "MyPool", menuName = "Pool/MyPool")]
public class MyPool : ComponentPool<MyComponent>
{
	[field: SerializeField] private MyComponentFactory factory { get; set; }
	[field: SerializeField] private int initialPoolSize { get; set; }
}
```

Example usage during runtime
```csharp
public class RuntimeScript : MonoBehaviour 
{
    [SerializeField] private MyPool pool;
    
    private void Awake()
    {
        pool = ScriptableObject.CreateInstance<MyPool>(); 
        pool.Factory = ScriptableObject.CreateInstance<MyComponentFactory>(); 
    }
    
    private void Start() 
    {
        pool.Prewarm(10);
        MyComponent component = pool.Request(); 
        // do things with the component then
        pool.Return(component); 
    }
}
```

## [Singletons](Singletons/Runtime/Singleton.cs) 

⚠️ The usage of Singletons in Unity is highly discouraged. Use ScriptableObjects instead wherever possible. 
Be aware this will not prevent a non singleton constructor such as `T myT = new T();`
To prevent that, add `protected T () {}` to your singleton class.

Create a singleton. Add this script to a single object in the scene. 

```csharp
public class MySingleton : Singleton<MyComponent> 
{
    public int count = 0;
}
```
Access the singleton during runtime using `Singleton.Instance`

```csharp
public class RuntimeScript : MonoBehaviour 
{
    private void Start() 
    {
        MySingleton.Instance.count++;        
    }
}
``` 

## Splines 

### [SplineCombinator](Splines/Runtime/SplineCombinator.cs)

Can combine splines in the Editor and during runtime when attached to a GameObject. 
This utility uses the `UnityEngine.Splines` package. 


Runtime example: 

```csharp
public class RuntimeScript : MonoBehaviour 
{
    [SerializeField] private SplineCombinator combinator;
    [SerializeField] private SplineContainer container;
    
    private void Start()
    {
        combinator.sourceSplineContainer = container; 
        combinator.scale = new(1, 1, 1); 
        combinator.GenerateCombinedSpline(); // applies changes directly to the container
    }
}
```

Comes with two static functions, 
```csharp
IEnumerable<Vector3> SplineCombinator.GetPositions(ISpline spline, int subdivisions)
```
which gets positions along a spline, and 
```csharp 
Spline SplineCombinator.ReverseFlow(Spline spline)
```
which reverse the flow of a spline. 

## UI Toolkit 

Utility for the `UnityEngine.UIElements` package.

### [Text Element AutoFitter](UIToolkit/Runtime/TextElementAutoFitter.cs)
Allows Text Elements to be resized based on a screen percent, which is 
not a feature of Unity as of 2023. The components in this folder should 
be automatically recognized by UIToolkit. Resizable Label and Button components are also provided.

Create a new resizable text element (from [LabelAutoFit.cs](UIToolkit/Runtime/LabelAutoFit.cs)): 

```csharp
using UnityEngine.UIElements;

namespace Mushakushi.UIToolkit.Runtime
{
    /// <summary>
    /// Auto-sizing <see cref="Label"/>.
    /// </summary>
    public class LabelAutoFit : Label
    {
        [UnityEngine.Scripting.Preserve] public new class UxmlFactory : UxmlFactory<LabelAutoFit, UxmlTraits>
        {
            public override string uxmlNamespace => "Mushakushi";
        }

        public LabelAutoFit() => TextElementAutoFitter.RegisterAutoFitCallbacks(this);
    }
}
```

See original issue on the [Unity Forum](https://forum.unity.com/threads/button-text-size-in-doesnt-work.1037551/#post-8865495)
where I first proposed this solution.