The first step in order to use TQ1 in your Windows Phone app is to add the Nuget package to your project. You can do this by right clicking you project on Solution explorer and selecting "Manage NuGet Packages...".

In order to be able to see the TQ1 package on the manager, you have to add our package source to Visual Studio.  Once it's done, you'll be able to install TaqtileSDK package.

In order to start the TQ service, you have to call:

```csharp
TQ.SharedInstance().Start("http://server.url.com", "appKey");
```

And you have to do this in a place that is called when the app starts or it is resumed. You have access to these events inside App.xaml code-behind:

```csharp
protected override void OnActivated(IActivatedEventArgs args)
{
  TQ.SharedInstance().Start("http://server.url.com", "appKey");
  base.OnActivate(sender, e);
}

protected override void OnLaunched(LaunchActivatedEventArgs args)
{
  TQ.SharedInstance().Start("http://server.url.com", "appKey");
  base.OnStartup(sender, e);
}
```

OnActivated is used when the app is resumed and OnLaunched when the app is started.

Now, when you leave the app, even if you are terminating it or just leaving it running in background, you gave to stop TQ, making it send the end_session method. As Windows Phone won't let you execute anything after you leave the app, you have to do this right before leaving. You must create a base page, or put this method inside every code-behind:

```csharp
protected override void OnNavigatedFrom(NavigationEventArgs e)
{
  base.OnNavigatedFrom(e);
  if (e.Uri.ToString().Equals("app://external/"))
    TQ.SharedInstance().OnStop();
}
```
See that we are verifying if the user is navigating to outside the app, as OnNavigatedFrom is called every time you leave the page, even inside your app.
