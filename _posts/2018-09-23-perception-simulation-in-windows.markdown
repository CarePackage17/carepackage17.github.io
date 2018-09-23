---
layout: post
title:  "Perception Automation API in Windows Mixed Reality"
date:   2018-09-23 15:44:07 +0200
categories: windows mixed-reality uwp
---
There's some interesting functionality built into Windows as of version 14393: The [perception automation API](https://docs.microsoft.com/en-us/uwp/api/windows.perception.automation.core).

Unfortunately, there are no source code samples on how to use this API. Granted, it's kind of low-level and not really useful for the majority of developers, but interesting nonetheless.  

The docs I've linked to above show that there's only a single static method in the `Windows.Perception.Automation.Core` namespace: `SetActivationFactoryProvider(IGetActivationFactory)`. Here's what the method description says: "Overrides the Windows Perception API and provides a custom class provider to implement the API".

Now that's a mouthful. It doesn't really tell us which exact APIs are overridden, though.  
There are 4 namespaces in the UWP API that contain perception functionality (not counting the `Devices` and `Automation` ones):
- `Windows.Perception`
- `Windows.Perception.People`
- `Windows.Perception.Spatial`
- `Windows.Perception.Spatial.Surfaces`

Are all of their classes overridden, or just some of them? Are they just from these namespaces or others too? Well, nobody (outside of Microsoft) knows, so let's experiment a bit.

The perception APIs are used on HoloLens and Windows Mixed Reality headsets. Since the `Windows.Perception.Automation.Core` namespace is in a Desktop extension contract, we're limited to that anyway, which is fine.  
We're going to start from the [BasicHologram sample](https://github.com/Microsoft/Windows-universal-samples/tree/master/Samples/BasicHologram/cppwinrt). If you want to follow along, clone it from GitHub and open it in Visual Studio.

The API provider we are going to write needs to be a WinRT component, so let's add that to our solution first:  
![I'm using the default C++/WinRT component template from VS2017](/assets/sln-layout.PNG)

After we've created the component project we need a class to implement the `IGetActivationFactory` interface, as the `SetActivationFactoryProvider` function expects that as its argument.  
![Here's our Class.idl](/assets/class-idl.PNG)

![Class.h](/assets/class-h.PNG)

![and Class.cpp](/assets/class-cpp.PNG)

Now, let's instantiate our class implementing `IGetActivationFactory` and pass it to the automation API when the sample app executes `AppView::Initialize`:
![](/assets/appview-initialize.PNG)

What happens now? We get an access violation, when attempting to create a `HolographicSpace` which didn't happen before. Probably because our implementation is not returning anything useful when asked for it, but at least we do get asked for it, which means `HolographicSpace` is on the list of overridden APIs.

Anyway, as we can see in the call stack, we've successfully replaced the Windows-provided implementation with our own:
![](/assets/callstack.PNG)

It doesn't work, but still.

If you've made it to the end, congratulations! (I hope it wasn't too boring)  
If you know more about the topic and want to give me some hints, my e-mail address is at the bottom of the page.