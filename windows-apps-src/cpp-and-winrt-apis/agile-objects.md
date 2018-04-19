---
author: stevewhims
description: An agile object is one that can be accessed from any thread. Your C++/WinRT types are agile by default, but you can opt out.
title: Agile objects with C++/WinRT
ms.author: stwhi
ms.date: 04/19/2018
ms.topic: article
ms.prod: windows
ms.technology: uwp
keywords: windows 10, uwp, standard, c++, cpp, winrt, projection, agile, object, agility, IAgileObject
ms.localizationpriority: medium
---

# Agile objects with [C++/WinRT](intro-to-using-cpp-with-winrt.md)
In the vast majority of cases, an instance of a Windows Runtime classes&mdash;like a standard C++ object&mdash;can be accessed from any thread. Such a class is *agile*. Only a small number of Windows Runtime classes that ship with Windows are non-agile, but when you consume them you need to take in to consideration their threading model and marshaling behavior (marshaling is passing data across a thread or process boundary). It's a good default for every Windows Runtime object to be agile, so your own C++/WinRT types are agile by default.

But you can opt out. You might have a compelling reason to require an object of your type to reside, for example, in a given single-threaded apartment. This typically has to do with reentrancy requirements. But increasingly, even user interface (UI) APIs offer agile objects. In general, agility is the simplest and most performant option. Also, when you implement an activation factory, it must be agile even if your corresponding runtime class isn't.

> [!NOTE]> The Windows Runtime is based on COM. In COM terms an agile class is registered with `ThreadingModel` == *Both*. For more info about COM threading models, see [Understanding and Using COM Threading Models](https://msdn.microsoft.com/library/ms809971).

Let's use an example implementation to illustrate how C++/WinRT supports agility.

```cppwinrt
#include "winrt/Windows.Foundation.h"

using namespace winrt;
using namespace Windows::Foundation;

struct MyType : implements<MyType, IStringable>
{
	winrt::hstring ToString(){ ... }
};
```

Because we haven't opted out, this implementation is agile. The [**winrt::implements**](/uwp/cpp-ref-for-winrt/implements) base struct implements [**IAgileObject**](https://msdn.microsoft.com/library/windows/desktop/hh802476) and [**IMarshal**](https://docs.microsoft.com/previous-versions/windows/embedded/ms887993). The **IMarshal** implementation uses **CoCreateFreeThreadedMarshaler** to do the right thing for legacy code that doesn't know about **IAgileObject**.

This code checks an object for agility. The call to [**IUnknown::as**](/uwp/cpp-ref-for-winrt/windows-foundation-iunknown#iunknownas-function) throws an exception if `myimpl` is not agile.

```cppwinrt
winrt::com_ptr<MyType> myimpl = winrt::make_self<MyType>();
winrt::com_ptr<IAgileObject> iagileobject = myimpl.as<IAgileObject>();
```

Rather than handle an exception, you can call [**IUnknown::try_as**](/uwp/cpp-ref-for-winrt/windows-foundation-iunknown#iunknowntryas-function) instead.

```cppwinrt
winrt::com_ptr<IAgileObject> iagileobject = myimpl.try_as<IAgileObject>();
if (iagileobject) { /* myimpl is agile. */ }
```

**IAgileObject** has no methods of its own, so you can't do much with it. This next variant, then, is more typical.

```cppwinrt
if (myimpl.try_as<IAgileObject>()) { /* myimpl is agile. */ }
```

**IAgileObject** is a *marker interface*. The mere success or failure of querying for **IAgileObject** is the extent of the information and utility you get from it.

## Opting out of agile object support
You can choose explicitly to opt out of agile object support by passing the [**winrt::non_agile**](/uwp/cpp-ref-for-winrt/non_agile) marker struct as a template argument to your base class.

If you derive directly from **winrt::implements**.

```cppwinrt
struct MyImplementation: implements<MyImplementation, IStringable, non_agile>
{
	...
}
```

If you're authoring a runtime class.

```cppwinrt
struct MyRuntimeClass: MyRuntimeClassT<MyRuntimeClass, non_agile>
{
	...
}
```

It doesn't matter where in the variadic parameter pack the marker struct appears.

## Important APIs
* [IAgileObject interface](https://msdn.microsoft.com/library/windows/desktop/hh802476)
* [IMarshal interface](https://docs.microsoft.com/previous-versions/windows/embedded/ms887993)
* [winrt::implements struct template](/uwp/cpp-ref-for-winrt/implements)
* [winrt::non_agile marker struct](/uwp/cpp-ref-for-winrt/non_agile)
* [winrt::Windows::Foundation::IUnknown::as function](/uwp/cpp-ref-for-winrt/windows-foundation-iunknown#iunknownas-function)
* [winrt::Windows::Foundation::IUnknown::try_as function](/uwp/cpp-ref-for-winrt/windows-foundation-iunknown#iunknowntryas-function)

## Related topics
* [Understanding and Using COM Threading Models](https://msdn.microsoft.com/library/ms809971)