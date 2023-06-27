---
title:          "Fixing Unity's Shortcomings Using a Little Bit of Dark Magic"
description:    "X questionable strategies reestablish your dominance over Unity" 
logo:           "{{ site.baseurl }}/img/logo.svg"
author:         johannesvollmer
published: 	    false
---


<!-- 

title:
- Taming Unity: How to stay on top
- How to ride on top Unity's Flood of Runtime Errors
- Surfing Unity: Staying on top of its design flaws
- How to survive surfing the Unity 3D waves
- How to not burnout using Unity
- Using code to fight Unity pitfalls
- Coding your way out of Unity's Error Hell
- Fixing Unity's Shortcomings Using a Little Bit of Dark Magic
- X questionable Strategies regain your Dominance over Unity
- X Tools I wish I had created right at the start
- Your Normal Map is not marked as Normal Map!

audience: 
    unity junior dev

the one thing i want to say:
    "don't hesitate to use black magic if it fixes a real problem"
    or maybe rather 

topic (pick narrow and deep, not wide and shallow..!?):
    X strategies to tame Unity for production
    X Strategies regain your Dominance over Unity
    X Tools I wish I had created right at the start

core reasoning:
- why: unity annoying bugs at runtime
- what: make unity more reliable
- how: checks and tests

introduction:
- working with unity for over a decade, full time for about two years
- want to share some of my strategies 
- biggest pain point: unreliability, fear of releasing broken stuff, but also compiling to connected device for minutes, then instantly falling due to minor oversight
- disclaimer: working with AR & Cloud Anchors often prevents the use of Unity Remote / Unity ARFoundation Remote 3rd party plugin. some of the pain points might be specific to this use case

metaphors:
unity development is like surfing, without tying the surfboard to yourself
feels amazing to hit the spot, but in reality you will be falling of a lot more than you are riding on top

asset strategies:
- use code instead of assets when possible 
- validate assets using scripts A LOT
- automatically run tests (& checks)
- automating workflows (build & bundles)

code discipline strategies:
- a lot of runtime assertions
- better types and exceptions
- raising level of abstraction (+ logging)
- fix bugs in unity (meaning workarounds)
- fix shortcomings of C# (using reflection)

strategies dump:
- run tests before building
- run tests in ci
- extending methods to throw exceptions
- nullable extensions 
- Logging utilities (expeessions, to string)
- Path type
- Units types
- UniRX from 6y ago better than async
- raising level of abstraction with extensions
- reflective equals & to string
- tests that analyse scenes and prefabs
- ExpectAssignedInEditor & ...
- testing for existence of script in scene
- assign asset bundles by folder strictly
- generating assets
- check texture setting based on name
- set project settings by code (dbg/release)


examples for missing exceptions in unity:
- when trying to open a scene from a copied asset during batch asset editing, it gives you an empty valid scene instead of invalid or correctly filled scene
- when scene loading fails, gives you a scene object that has the invalid flag set
- component not found on object
it's a design basic rule that the user should not be forced to remember doing a thing, "if only we had a computer to do such tedium for me" (https://eev.ee/blog/2016/12/01/lets-stop-copying-c/)
 -->

# Content

<!-- #1 focus lead (probably narrative, anectode) 3-4§ -->


# "Your Normal Map is not marked as Normal Map!"
Working with Unity can be ... frustrating at times. 

When I started working full time with Unity, before releasing, I manually went through the Textures and checked their Settings. "Are all the normal maps set to the texture type Normal Map?" Usually, Unity warns you, when you incorrectly use a normal map slot. But it does so only for the Standard Shaders, not in your custom Shadergraph Shaders, which we have a lot of. (Some random feature missing, like the last puzzle piece missing in an otherwise complete picure, is a common theme in Unity. (Actually, the vast majority of this article solves problems you would expect Unity to have solved already. But we'll get to that.))

Of course, I knew from the start that this manual process was absolutely horrible. So one day, I quickly whipped up a script that checks the texture settings. First it only checked the texture setting based on the file name, on button click. But over time, I improved it, and now it also checks whether all textures used in a "normal" slot in any shadergraph material are actually normal maps. The script runs when any texture assets are changed, and also before building the app. So you don't even notice its existence, apart from all textures always being correct.

To this day, my 3D Artist coworker has praised this script multiple times. It prevented errors and made his life easier. This is the ultimate compliment. That's why we're programmers, to automate tedious things! Excellent!


<!-- #2 nut graph 2-3§ -->


# What's this article about?

When googling for 'how to improve unity c#' or similar topics, you might notice that most of the articles are about improving code performance. Of course, we don't want to be wrong a lightning speeds when we could be right at acceptable speed. But even advanced developers make mistakes, especially when working with an API as ... _peculiar_ as in Unity. So here are some advanced techniques the beginner does not have to worry about, but are essential for correctness in advanced projects.

I've been working --with-- against Unity full-time for a few years now, and I've always wanted to share this experience with all of you. So, after reflecting on my code for some time now, I condensed X strategies that helped me make the code more reliable. Ye be warned, some of these might require the use of forbidden magic, as you will see. So, here are those strategies on how to improve your unity code, most important one first, in my experience.

### Assets: Exploit scripting the Editor
Adding basic functionality it should have shipped with from the start.

1. Use code instead of assets
2. Validate assets with code
3. Automate workflows. With code.

Starting to see a pattern? ;D

### Code: Replace the Unity C# API with your own
Adding basic functionality it should have shipped with from the start.
By extensive use of extension methods.

1. Add a few missing types that every programming language should have
2. Use Exceptions, without exception
3. Raise the level of abstraction


<!-- #3 body of story -->


# Scripting the Editor

## Use Code Instead of Assets
"But code has to recompile, while I could simply edit assets with the Editor", you might think. And that's exactly the point.

Why? Programming is a craft hat has a unique super power: it can verify itself. First of all, the compiler finds simple mistakes. Furthermore, you can write code that tells you whether your code is correct, automated tests! That's just a blessing! Do you realize how lucky we are, as an industry?

But there are event more advantages to generating Assets instead of editing them by hand. You can add comments that explain why something must be in a certain way. You can factor out common patterns, reducing everything to its very essence, and thereby raising the level of abstraction. This also means looping, instead of us silly squishy human duplicating objects by hand.

Here are some of my practices to generate Assets:
- For simple Objects, create them at Runtime using `new Object(name)` and `.AddComponent<AudioSource>()`

    Make sure to raise that level of abstraction again and again. For example `myObject.NewChild().WithAudioSource(my_audio_clip);`. It's definitely worth it! Try extension methods on the existing Unity classes!
- Create a `ScriptableObject` that has Settings and then generates Assets into its folder
- Use Custom Importers and Asset Post Processors to augment or modify imported data automatically instead of manually

These techniques are useful across all aspects in Unity. Use them whenever you see fit.

This article has been pretty harsh on Unity so far, and will be going further. So let's take a moment to be thankful for those extensive scripting possibilities. Thanks!



## Validate Assets with Code
Earlier, I said that we can write code that verifies other code. Perhaps you thought, can't we verify Assets too, using code? Yes! Not only can we do it, we absolutely __should do it__.

### Texture Settings
For example, I wrote a script that checks for suspicious texture settings. It checks whether texture that look like normal maps actually have the normal map setting, but also makes sure that non-color textures use linear color space.

### Assign Assets to Asset Bundles Automatically
In addition to generating assets, you can edit assets using code. For example, I wrote a script that automatically keeps all assets in one folder assigned to an Asset Bundle, and deassigns all Assets outside that folder.

### Check for Missing Prefabs, Missing Scripts and Unassigned Script Variables
It was a common error in the project that some variable was exposed in the editor, but I forgot to assign it before launching the app to my mobile device. Furthermore, missing assignments and missing scripts can occur regularly when working with version control in Unity. What script in your scene could go missing without breaking your app? None in my project. However, if something goes wrong, Unity will not complain. There is no way to make Unity complain, even for release builds. What?! 

I don't know about you, but I certainly don't want a merge conflict to break the release build, silently! So I had to do it myself. 


### How to do it
Often, those scripts will want to go through all assets one by one, but not all assets, because you probably don't want to include assets from third party packages.
To solve this, I used a custom ScriptableObject named `CheckAssetsInFolder`. Then, whenever some script wants to crawl all assets, they look for those ScriptableObjects, and use their folders as root folders. Or, when any asset changed, filter the list of changed assets and only process the assets if they are in any of the marked folders.

Those scripts typically run before the build. They also typically include a unit test, so they are also part of the test suite.

Furthermore, it is not as simple as opening a scene. As we have multiple Unity projects but want to reuse our assets, most of them come from a custom package. For some curious reason, Unity can't open a scene file from a package, because those files are read only. Why does opening a scene automatically and always need write access to the file? I don't know. Anyways, the workaround is to temporarily copy that scene into the Assets folder. Great! I factored that out into a function.

Here are some key sections from the scripts:

```cs
[CreateAssetMenu(fileName = "Test Assets in Folder", menuName = "ScriptableObjects/Test Assets in Folder", order = 1)]
public class TestAssetsInFolder : ScriptableObject {
    public static IEnumerable<Path> AllTestedFolders() =>
        EditorPaths.FindAnyAssetsOfType<TestAssetsInFolder>()
            .Select(asset => asset.Parent());
}
```

After loading the scene in the editor, we have access to regular `GameObject`s, just like when running the game.

```cs
[Test]
public void TestAllSceneAndPrefabFiles() {
    var tmpPath = EditorPaths.Assets.Then("tmp-copy-of-readonly-scene-3451812739487389.unity");
    
    try {
        EditorSceneManager.SaveCurrentModifiedScenesIfUserWantsTo(); // in case this test is executed in isolation
        InspectAssets.ForAllObjectsInScenesAndPrefabs(
            TestAssetsInFolder.AllTestedFolders().ToArray(),
            tmpPath, SceneTests.CheckGameObject
        );
    }
    finally {
        tmpPath.TryDeleteAsset();
        EditorUtility.UnloadUnusedAssetsImmediate(true);
    }
}

private static void CheckGameObject(GameObject gameObject) {
    SceneTests.CheckGameObjectForMissingPrefabs(gameObject);
    SceneTests.CheckGameObjectMembers(gameObject);
}
```

The above code goes through all the game objects in all the assets to find suspicious data.

```cs
private static void CheckGameObjectForMissingPrefabs(GameObject gameObject) {
    // lul https://forum.unity.com/threads/detecting-missing-nested-prefab.697562/
    if (gameObject.name.ExactlyEndsWith("(Missing Prefab)")) {
        throw new Exception(
            $"Missing Prefab: \"{gameObject.name}\"\n"
            + $"in \"{gameObject.scene.path}\": "
            + $"{gameObject.LocationInHierarchy()}"
        );
    }

    gameObject.GetDirectChildren().ForEach(SceneTests.CheckGameObjectForMissingPrefabs);
}

private static void CheckGameObjectMembers(GameObject gameObject) {
    var components = gameObject.GetComponents<MonoBehaviour>();
    
    foreach (var component in components) {
        if (component == null) throw new UnassignedScriptException(gameObject);
        if (component is ExpectValidInEditor validate) validate.ExpectValidInEditor();
        SceneTests.CheckComponentMembers(component);
    }
    
    gameObject.GetDirectChildren().ForEach(SceneTests.CheckGameObjectMembers);
}
```

The code above finds missing Prefabs and missing scripts.

We don't check all the members though, only those marked with a custom attribute. This is what a typical script in our project looks like:
```cs
public class SetMaterialFloat: MonoBehaviour {
        
    [SerializeField]
    [ExpectNotEmptyInEditor]
    private string uniformName;

    [SerializeField]
    [ExpectAssignedInEditor]
    private Material material;

    [SerializeField]
    [ExpectAssignedInEditor]
    private new Renderer renderer;

    // ...
}
```
Those fields will be checked using reflection. This is surprisingly fast, perhaps  because it's cached after the first component of any given type is encountered.

This is literally the code in the project. As you can see, this code uses a lot of abstraction on top of the cumbersome Unity API. For example, the path class can be seen in action here. The `Action<Scene> check` argument will be called for all scenes in the project.


## Custom Types

A disproportionate amount of the bugs that appeared while developing those scripts were due to some malformed path string. I really didn't want to do it for a long time, but in the end I gave in, and created an appropriate `Path` type. Since then, I have no regrets, and zero bugs due to path concatenation slashes not matching up.

I could not find pretty standard path functionality in neither Unity nor C#, such as testing whether some path is a child of another path. Can you believe it? Unity uses strings all over the place. Get your stringly typed API out of my way!


# Performance
This doesn't mean the suggested code is slow. It means that I most of the code was not used in performance-critical sections. When you notice a performance problem, you should profile and optimize that particular code.

<!-- #4 kicker -->