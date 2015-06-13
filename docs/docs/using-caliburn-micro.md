##Setup

If you are using Caliburn Micro, the right place to call the start method is inside the Bootstrapper:

```csharp
protected override void OnActivate(object sender, Microsoft.Phone.Shell.ActivatedEventArgs e)
{
  TQ.SharedInstance().Start("http://server.url.com", "appKey");
  base.OnActivate(sender, e);
}

protected override void OnStartup(object sender, System.Windows.StartupEventArgs e)
{
  TQ.SharedInstance().Start("http://server.url.com", "appKey");
  base.OnStartup(sender, e);
}
```

OnActivate will be called when the app is resumed and OnStartup will be called when the app is started.

When using Caliburn Micro, you have the option to call the OnStop method inside the Bootstrapper, using the OnDeactivate method:

```csharp
protected override void OnDeactivate(object sender, Microsoft.Phone.Shell.DeactivatedEventArgs e)
{
  if (e.Uri.ToString().Equals("app://external/"))
  TQ.SharedInstance().OnStop();
  base.OnDeactivate(sender, e);
}
```

##Analytics

With Shingle started you can add events, like in the following example:

```csharp
protected override void OnActivate()
{
  base.OnActivate();
  Dictionary<String, String> seg = new Dictionary<String, String>();
  seg.Add("Page name", "HomePage");
  TQAnalytics.SharedInstance().RecordEvent("Event name", seg, 1);
}
```

I highly recommend adding these methods inside OnActivate as this method is called every time the view appears (as the name says, every time it is activated).
