---
layout: post
title: "Notes on Finalization and Disposition in .NET"
date: 2015-03-12 22:04:15 +0100
comments: true
categories: [.net, microsoft, disposition]
footer: true
sharing: true
---

Have you ever thought why disposition pattern is designed the way it is? I hadn't. Sometimes we learn things along the way and never question its basics or roots again. We do it that way because we're told so or because it just worked so far and we never looked back.

Nicole Calinoiu's talk from NDC Oslo taught me many things one of which is how IDisposable pattern is designed and became the way it is. It's an awesome talk, one that I can strongly recommend. I've embedded the video at the bottom of this page, make sure to check it out.

I saw the talk twice and wanted to take notes for myself. Notes that I can revisit when the details get blurry in my head six months or a year down the path. I've also double checked or dived into some other details through MSDN and included those in the notes as well.

## Finalization

* Never ever set any object to NULL to "help" Garbage Collector do his job. You're not helping him and even making things worse.

* Although .NET Runtime knows about cleaning up memory really well, it can't possibly know the shared resources (file system, network connections, registery etc...) that your objects are using. Finalization is the object's opportunity to cleanup those resources when it's about to be removed from the memory.

* If a .NET object overrides the method Object.Finalize(), the Garbage Collector will call the Finalize method before cleaning up the object from memory.

* If the object does not override the method Finalize, Garbage Collector doesn't finalize the object (obviously). Here is what <a href="https://msdn.microsoft.com/en-us/library/system.object.finalize%28v=vs.110%29.aspx" target="_blank">MSDN</a> says:

>The garbage collector does not mark types derived from Object for finalization unless they override the Finalize method.

* Now comes the part that's obscure in the MSDN documentation. <a href="https://msdn.microsoft.com/en-us/library/system.object.finalize%28v=vs.110%29.aspx" target="_blank">MSDN</a> says (emphasis mine):

>You should override Finalize for a class that uses __unmanaged resources__ such as file handles or database connections that must be released when the managed object that uses them is discarded during garbage collection.

* __Unmanaged resources__ mentioned in the documentation are __NOT__ the managed wrappers. If you have code that directly uses IntPtr and DllImport then you're using unmanaged resources and you need to be thinking about Finalization. 

* Managed wrappers have their own Finalizers (hopefully) so it's not your responsibility to supply another Finalizer on top of that. But if you have any code in a class that looks like the one below, then you need a Finalizer for that specific class.

``` csharp
[DllImport("advapi32.dll", CharSet= CharSet.Auto, SetLastError=true)]
private static extern int RegOpenKeyEx(IntPtr hKey, 
						               String lpSubKey, 
						               int ulOptions, 
						               int samDesired,
						               out IntPtr phkResult);
```                  

* Writing an __unnecessary__ Finalizer comes with a cost: delaying the cleanup process of your objects. Finalization brings extra steps to the cleaning process (adding those objects to Finalization queue and running their Finalization logic in a separate thread) which delay freeing up memory for those objects. So beware.

* Finalizers should not access any managed resources. Any managed resource at that point could already be harvested by garbage collector. So basically none of the managed objects can be trusted to exist. Finalizers should only cleanup the unmanaged resources and return.

## Disposition

* Finalization happens when the Garbage Collector "feels like it". In general you don't have (and shouldn't take) any control on when the GC is running or not. But wouldn't it be nice to cleanup and let go of those shared resources when they're not needed anymore? Hell yeah. It would be terrible to keep locking a file and wait until GC comes to clean it up. This is exactly why Disposition exists.

* When a class implements IDisposable it is basically declaring that it needs to be cleaned up. Whenever you're done with an IDisposable class and you know you're the __only__ one using it or the __owner__ of its lifecycle, you need to call its Dispose method to clean it up.

* While Disposing a class and releasing its shared resources, we also want to call Finalize if our class is Finalizable. If you try to do that though, compiler gives an error message: "Destructors and Finalize can not be called directly". That's why there is a pattern called IDisposable pattern.

* <a href="https://msdn.microsoft.com/en-us/library/system.idisposable(v=vs.110).aspx" target="_blank">MSDN documentation on IDisposable interface</a> includes a C# code sample showing how the pattern should be implemented. Notice that because Finalize can not be called directly, another Dispose method is introduced that takes a boolean argument which determines whether the class is being disposed or finalized.

``` csharp

using System;

class BaseClass : IDisposable
{
   // Flag: Has Dispose already been called? 
   bool disposed = false;

   // Public implementation of Dispose pattern callable by consumers. 
   public void Dispose()
   { 
      Dispose(true);
      GC.SuppressFinalize(this);           
   }

   // Protected implementation of Dispose pattern. 
   protected virtual void Dispose(bool disposing)
   {
      if (disposed)
         return; 

      if (disposing) {
         // Free any other managed objects here. 
         //
      }

      // Free any unmanaged objects here. 
      //
      disposed = true;
   }

   ~BaseClass()
   {
      Dispose(false);
   }
}

```

* Let's look at a few important aspects of this implementation:
  * Class keeps track of whether it has been disposed or not and prevents being disposed more than once.
  * When a class is being Disposed, the Finalization logic also runs. So it tells the Garbage Collector not to bother Finalizing this object anymore (performance gain):

``` csharp
GC.SuppressFinalize(this);
```

* These days writing a Finalizer is fairly uncommon. Most .NET applications doesn't require a usage of direct pointer to an unmanaged resource. On top of that, now there is a class called <a href="https://msdn.microsoft.com/en-us/library/system.runtime.interopservices.safehandle%28v=vs.100%29.aspx" target="_blank">SafeHandle</a> which automatically wraps a pointer to an unmanaged resource. It also handles finalization for you. So this means writing a Finalizer now is very very uncommon. Check the article from Joe Duffy all the way back from 2005: <a href="http://joeduffyblog.com/2005/12/27/never-write-a-finalizer-again-well-almost-never/" target="_blank">Never write a finalizer again (well, almost never)</a>

* The whole purpose of IDisposable pattern is to accomodate Finalization and Disposition under a single solution. But because you may never ever write a finalizer again, the whole IDisposable pattern ends up being designed to cover a very very corner case. And it's unfortunate that nothing has changed since 2005...

## Nicole Calinoiu's talk from NDC Oslo 2014
<br/>
<iframe src="//player.vimeo.com/video/97519508" width="100%" height="381" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

