The push notifications are separated for Windows Phone 8 and Windows Phone 8.1, as there are huge differences about the way they implement the push notifications.

##Windows Phone 8

The first thing to do is to **add the push notifications permission to your app**. In order to do this, you must edit the *WMAppManifest.xml* file under the Properties section, marking the ID_CAP_PUSH_NOTIFICATION capability.

![](https://www.filepicker.io/api/file/SzeWpK75ReCPupvYvVuh)

Another important thing to do is to **add the SQLitePCL nuget to your project**. If you don't do this, the SDK won't be able to save the push contents to the SQLite database. In order to do this, right click your project and then select "Manage NuGet packages..." and search for "SQLitePCL" and install the "Portable Class Library for SQLite" nuget. You should also have the SQLite for Windows Phone intalled, here are the links for version 3.8.5:

http://sqlite.org/2014/sqlite-wp80-winrt-3080500.vsix
http://sqlite.org/2014/sqlite-wp81-winrt-3080500.vsix
http://sqlite.org/2014/sqlite-winrt-3080500.vsix
http://sqlite.org/2014/sqlite-winrt81-3080500.vsix

Close Visual Studio and execute them. Then you will be able to right click your project and go to Add > Reference > Windows Phone SDK and check SQLite for Windows Phone.

After doing this initial setup, your project is ready to use the SDK in order to receive push notifications and the differences between the two operating systems come to the surface.

The next step is to call the RegisterForRemoteNotifications method to register your phone, making it able to receive push notifications via Shingle. You should do this **after starting the SDK,** as it uses the server URL and the app key (you can do this in the code behind of your main page, for example).

```csharp
TQ.SharedInstance().RegisterForRemoteNotifications("SampleChannel");
```

The "SampleChannel" string represents the name of the push notification channel that is registered on the phone and is kept open by the operating system. If you want to learn more about the channel, check [this page](http://msdn.microsoft.com/en-us/library/windows/apps/ff402558.aspx) for 8.0 and [this page](http://msdn.microsoft.com/en-us/library/windows/apps/hh913756.aspx) for 8.1.



Now that your app is ready to receive the push notifications, you should process the information you receive. As the only information that comes with a toast push notification is the Uri that you want to navigate to, with it's parameters, we have to do some workaround in order to receive the Push Id and download the contents from the server. The string you will receive will be like:

```
/NotificationReceived.xaml?pid=5555example5555;text=example text
```

Receiving the text like this has some consequences. The first one is that you will only have access to it inside the navigation events, the other one is that you **must** create a page named "NotificationReceived" on the project root folder, this will always be the path you receive. In this page you can pass the URI to the SDK and do some other processing. I suggest that you use the await method in order to give the SDK the needed time to download the push contents and show some kind of loading to the user, like in the following example:

```csharp
public partial class NotificationReceived : PhoneApplicationPage
{
  public NotificationReceived()
  {
    InitializeComponent();
  }

  protected override async void OnNavigatedTo(NavigationEventArgs e)
  {
    bool success = await TQ.SharedInstance().NotificationReceived(e.Uri.ToString());
    List<TQInboxMessage> list = TQInbox.SharedInstance().GetInboxMessages(TQInboxMessageStatus.TQInboxMessageStatusAll);
    List<string> strings = new List<string>();
    foreach (TQInboxMessage m in list)
    {
      strings.Add(m.Content);
    }
    lls.ItemsSource = strings;
    base.OnNavigatedTo(e);
  }
}
```

In this example, we pass the Uri to the SDK, then after the download finishes and the data is in our database, we put the contents inside a list of TQInboxMessage's and show it. The method used to get the messages is:

```csharp
TQInbox.SharedInstance().GetInboxMessages(TQInboxMessageStatus status);
```

This way you can get the messages and do what must be done.

##Windows Phone 8.1

Like in Windows Phone 8, you must also enable the notifications, but you do this in the Package.appxmanifest file, enabling toast notifications under the Application tab:

![](https://www.filepicker.io/api/file/l9NqMQrKS7uTCCRu3wOF)

Like on WP 8.0 project, you must also **install SQLitePCL** nuget, otherwise your project won't be able to save the messages to the database. If you don't do this, the SDK won't be able to save the push contents to the SQLite database. In order to do this, right click your project and then select "Manage NuGet packages..." and search for "SQLitePCL" and install the "Portable Class Library for SQLite" nuget. You should also have the SQLite for Windows Phone intalled, here are the links for version 3.8.5:

http://sqlite.org/2014/sqlite-wp80-winrt-3080500.vsix
http://sqlite.org/2014/sqlite-wp81-winrt-3080500.vsix
http://sqlite.org/2014/sqlite-winrt-3080500.vsix
http://sqlite.org/2014/sqlite-winrt81-3080500.vsix

Close Visual Studio and execute them. Reopen Visual Studio and then you will be able to right click your project and go to Add > Reference > Windows Phone SDK and check SQLite for Windows Phone.

Windows Phone 8.1 presented us some improvements on toast notifications, so now we are able to receive a JSON instead a URI like string and we don't necessarily need a NotificationReceived.xaml page. Now, in order to receive this JSON, you must override the OnLaunched event that is present on App.xaml code behind and pass the pis, text and cid (if it exists) to the NotificationReceived event, like in the following example:

```csharp
protected override async void OnLaunched(LaunchActivatedEventArgs e)
{
  //Getting arguments from toast notification
  string launchstring = e.Arguments;
  //Parsing the arguments
  Dictionary<string, string> values = null;
  if (!String.IsNullOrEmpty(launchstring))
  {
    values = JsonConvert.DeserializeObject<Dictionary<string, string>>(launchstring);
    //Get the custom content
    bool success =  await TQ.SharedInstance().NotificationReceived(values["pid"], values["text"], values["cid"]);
    if (success)
    {
      IList<TQInboxMessage> l = TQInbox.SharedInstance().GetInboxMessages(TQInboxMessageStatus.TQInboxMessageStatusAll);
    }
  }
}
```

Then, you can handle navigation, or if you want, you can also add the NotificationReceived page to the project, just like the Windows Phone 8 project.

##Getting custom content from the push notification

When you receive a push notification, you must call  `TQ.SharedInstance().NotificationReceived` method, passing the push notification param as an argument. It will request the custom content to the API and save it so you can get it. After doing this, you will be able to get the messages, with all contents inside, like in the following example.

```csharp
bool success = await TQ.SharedInstance().NotificationReceived(e.Uri.ToString());
List<TQInboxMessage> list = TQInbox.SharedInstance().GetInboxMessages(TQInboxMessageStatus.TQInboxMessageStatusAll);
```

The push custom contents can be obtained by getting the Content attribute from the TQInboxMessage object, like the following example:

```csharp
string s = message.Content;
```

If the content is a Tag, you will be able to get a dictionary from the message, using the following method:

```csharp
Dictionary<string, object> dict = message.GetTags();
```

If the content type is not tag, it will return an empty dictionary.
