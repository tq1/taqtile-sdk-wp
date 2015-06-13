##Recording events

Once TQ1 is started, you can add events using the *OnNavigatedTo* event called inside the view code-behind:

```csharp
protected override void OnNavigatedTo(NavigationEventArgs e)
{
  base.OnNavigatedTo(e);
  Dictionary<String, String> seg = new Dictionary<String, String>();
  seg.Add("Page name", "HomePage");
  TQAnalytics.SharedInstance().RecordEvent("Event name", seg, 1);
}
```

  - We suggest you using the strings that will be on  "Page name" and "Event name" places (these are only examples) as static resources, as you will probably use them on many pages, if you have to change them, you will suffer a lot.
