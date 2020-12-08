# Build Amazing Cloud Connected Apps with Xamarin, Azure, and App Center

Xamarin + Azure + App Center = One Amazing Cloud Connected App!

Each of these technologies by themselves are powerful, but when you combine them, you're able to build a cloud connected cross platform app that is able to harness the power of Azure in a matter of minutes. And it's even able to create user accounts, a very tricky feature that's necessary, but difficult to understand.

Now - I said you can create it in minutes ... that's not _exactly_ true. You can't create anything in minutes. It takes work. I won't kid you.

But what does take minutes is to get a fundamental understanding of how these three pieces of technology work on a fundamental level and interact. And then once you have that - you can be off and running to create apps of your own!

## Like Videos?

Do you want to watch a recording of what this README is all about? 

Who wouldn't?

It just so happens I did a live session for .NET Conf 2019 that demonstrates this app and gives a high level overview of all the concepts involved - in about 25 minutes!

So [check it out](https://channel9.msdn.com/Events/dotnetConf/NET-Conf-2019/Build-Amazing-Cloud-Connected-Apps-With-Xamarin-Azure-and-App-Center?WT.mc_id=mobile-0000-masoucou) - and be sure to come back here for a more in-depth discussion.

## What Are We Going to Build?

In the video above, throughout this README, and with all the code in this repository; we're going to build a weather application. (Based on the [Pretty Weather sample](https://github.com/jamesmontemagno/app-pretty-weather).

I know, I know - another weather app.

But this one goes further than most. It actually retrieves the weather via a custom API! It allows users to sign-in to an Active Directory B2C instance. It stores weather preferences into a Azure Cosmos DB instance!

It's a fully enabled cloud app. Showing off how you can integrate a ton of cloud services into your Xamarin application.

## How Did We Build It?

* An iOS and Android app - all built with Xamarin.Forms. Shared application logic. Shared user interface. Shared code all over the place!
* Azure Functions wraps up a 3rd party weather service into a custom API for the mobile app to call. 
* The mobile app uses the Mobile Backend as a Service (MBaaS) functionality of App Center to build out a Azure AD B2C instance so users can log in. Authentication and authorization is... _confusing at best_ and App Center's MBaaS makes it easy.
* The mobile app makes uses Azure Cosmos DB - so users can persist the weather - again with App Center's MBaaS making authentication/authorization to Cosmos seamless.

Xamarin! Azure Functions! App Center! Amazing!

Let's take a look at each of the components used to build this app in turn.

## Xamarin

Xamarin's claim to fame is that it allows you to build 100% completely native iOS and Android applications using C# and .NET. But it does more! You can also build macOS, UWP, tvOS, WatchOS, even Tizen applications using it too!

You can find out how Xamarin works - it's techniques for [code sharing here](https://docs.microsoft.com/xamarin/get-started/what-is-xamarin?WT.mc_id=mobile-0000-masoucou). And you can find out how Xamarin.Forms extends that [code sharing to the user interface layer here](https://docs.microsoft.com/xamarin/get-started/what-is-xamarin-forms?WT.mc_id=mobile-0000-masoucou).

In this application - we use Xamarin.Forms to build out the user interface and the application logic layer. In fact, the only code that runs on iOS or Android alone is the code which bootstraps Xamarin.Forms. It's boilerplate code added with the File->New process when creating a project. For our purposes - this application is 100% shared code!

The app uses the following features of Xamarin.Forms:

* [MVVM pattern](https://docs.microsoft.com/xamarin/xamarin-forms/enterprise-application-patterns/mvvm?WT.mc_id=mobile-0000-masoucou) - with [data binding](https://docs.microsoft.com/xamarin/xamarin-forms/xaml/xaml-basics/data-binding-basics?WT.mc_id=mobile-0000-masoucou)
* [Shell](https://docs.microsoft.com/xamarin/xamarin-forms/app-fundamentals/shell/introduction?WT.mc_id=mobile-0000-masoucou)
* [Messaging Center](https://docs.microsoft.com/xamarin/xamarin-forms/app-fundamentals/messaging-center?WT.mc_id=mobile-0000-masoucou)
* [Value Converters](https://docs.microsoft.com/xamarin/xamarin-forms/app-fundamentals/data-binding/converters?WT.mc_id=mobile-0000-masoucou)

The app is built of 4 distinct pages:

* `AppShell.xaml` -> the overall container that provides Shell functionality
* `MainPage.xaml` -> displays the current weather conditions for the selected city & allows the user to log in via App Center
* `SavedCitiesPage.xaml` -> displays the saved cities from App Center
* `AddCitySearchPage.xaml` -> allows the user to select a new city to display weather conditions from and saves it to App Center

### MVVM Pattern

MVVM stands for Model-View-ViewModel. The pattern is a lot like MVC (just more V's and more M's 🤣). Its purpose in life is to decouple any application logic from the application's user interface. You put all logic into the ViewModel. The ViewModel returns information - in a Model class - to be displayed in a View.

And you can use what's known as data binding to have the view updated automatically whenever the data it's bound to changes (and vice versa).

This app uses the MVVM pattern for every page. For the `MainPage.xaml` which shows the current weather conditions - it has an associated view model of `WeatherViewModel`. And that's associated in the `MainPage.xaml` with the following syntax:

```language-xaml
<ContentPage.BindingContext>
    <viewmodel:WeatherViewModel Temp="61" />
</ContentPage.BindingContext>
```

Then the binding to each user control is done with a syntax that looks like this:

```language-xaml
<Label Text="{Binding Temp}"/>
```

That tells Xamarin.Forms to look into the ViewModel for a property called `Temp`.

Then in order for the `Label`s `Text` property to change anytime the ViewModel's `Temp` property changes - the ViewModel must implement the `INotifyPropertyChanged` interface. And the `Temp` property's implementation will look like this:

```language-csharp
int temp;
public int Temp {
    get => temp;
    set 
    {
        temp = value;
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(Temp)));
    }
}
```

Learn more about data binding, MVVM, and building XAML-based Xamarin.Forms apps in this [101 video series](https://www.youtube.com/watch?v=JH8ekYJrFHs&list=PLdo4fOcmZ0oU10SXt2W58pu2L0v2dOW)!

### Shell

So let's talk about Xamarin.Forms Shell for a bit. Shell gives applications an opinionated way to do a user interface. Tabs. Flyout menus. [Navigation](https://docs.microsoft.com/xamarin/xamarin-forms/app-fundamentals/shell/navigation?WT.mc_id=mobile-0000-masoucou). This app only uses a subset of the [navigation functionality](https://docs.microsoft.com/xamarin/xamarin-forms/app-fundamentals/shell/navigation?WT.mc_id=mobile-0000-masoucou).

You can see the navigation happening in the `WeatherViewModel` class's `ExecuteSavedCitiesCommand`. 

```language-csharp
async Task ExecuteShowSavedCitiesCommand()
{
    try
    {
        // some App Center stuff we'll talk about later
         
        var citiesPage = new SavedCitiesPage();

        await Shell.Current.Navigation.PushModalAsync(citiesPage);
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine(ex);
    }
}
```

Shell is cool in that it allows the app to navigate to new pages from view models. First we new up a `Page`. Then navigate to it - asynchronously of course. `await Shell.Current.Navigation.PushModalAsync(citiesPage);`

Shell provides means to both push onto the navigation stack and also push onto a modal stack.

### Messaging Center

Messaging Center is a great way to send... well messages.. between any class in a Xamarin.Forms solution - even the Android or iOS projects!

In the weather app, I'm using Messaging Center to indicate when a city has been selected from the search list. This will allow the `SavedCitiesViewModel` to add the newly selected city to its display list and also save it to Cosmos.

One thing to note - when sending messages using Messaging Center - use custom classes as the origin of the message. _Not_ the view model or page which you sent them from. This makes the message more reusable. See [my article here](https://codemilltech.com/messing-with-xamarin-forms-messaging-center/) for more info on it.

And here's a quick code sample on how it's done.

```language-csharp
var message = new SearchCitySelectedMessage { SelectedCity = SelectedCity };

MessagingCenter.Send(message, SearchCitySelectedMessage.Message);
```

The `SearchCitySelectedMessage` is the custom class. And `MessagingCenter` is using it as the origin of the message.

### Value Converters

The final cool thing that's happening in the application is the use of value converters. You use these cool cats during data binding, but when you don't want the control to display the value of the property you're binding to. Rather you want the control to display a different value - but that value is based on the property the view model.

Take a peek at an example.

```language-xaml
<FontImageSource Glyph="{Binding Icon, Converter={converters:CurrentConditionsIconConverter}}"/>
```

Here the `Glyph` property of `FontImageSource` is bound to the view model's `Icon` property. But I really want something else to be displayed. So it's attached to a value converter that looks something like this.

```language-csharp
public class CurrentConditionsIconConverter : IMarkupExtension, IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
    {
        // these hex codes coorespond to font aweosome solid codes
        if (value is string condition)
        {
            if (condition == "clear-day")
                return ((char)0xf185).ToString();

            if (condition == "clear-night")
                return ((char)0xf186).ToString();

        // And there's more
        }
    }
}
```

That's all the cool Xamarin.Forms stuff that's going into the app - now let's take a peek at what's happening in Azure Functions!

## Azure Functions

[Azure Functions](https://docs.microsoft.com/azure/azure-functions/?WT.mc_id=mobile-0000-masoucou) is Microsoft's serverless offering. And you can think of serverless as the next evolution to the "as-a-Service". This is Functions-as-a-Service. And you only have to worry about writing code. No operating system. No hardware. You don't even have to think about scaling. You just have to think about what's core to your application.

And our application is using Functions as something of a backing Web API. Functions can do more - and you can read about [bindings and triggers here](https://docs.microsoft.com/azure/azure-functions/functions-triggers-bindings?WT.mc_id=mobile-0000-masoucou).

We're using HTTP triggers. Meaning anytime an HTTP request comes in - our function reacts to it. (And only while it's running do we get charged for it too!)

You can find the code for our Function in the `PrettyWeather.Functions` solution.

There's one Function involved and it calls out to the Dark Sky Weather API.

You may be wondering why don't we just put the call to Dark Sky right into the Xamarin app? There's a couple of reasons.

* Dark Sky relies on an API key. We don't want that API key on a user device. So it's better to have it stored in the cloud.
* Having the weather API in a Function allows us to shape the response to make it easier for our app to deal with. And it makes it easier for other apps we create (even web apps) to consume it.
* And having all the logic for calling the 3rd party weather service in a single place makes it easier to refactor should we need to do that.

So ... if you want to reproduce this sample as is - [sign up for a Dark Sky account](https://darksky.net/dev/register) - it's free. And then create a `local.settings.json` file that looks like the following:

```language-json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet",
    "DarkSkyKey": "**** YOUR KEY IS GOING TO GO HERE ****",
    "DarkSkyUrl": "https://api.darksky.net/forecast",
    "DarkSkyParams": "exclude=minutely,hourly,alerts,flags,daily"
  }
}
```

Then you'll be good to go.

### Invoking a Function From a Xamarin App

There is nothing special calling an Azure Function from a Xamarin application. It's the same as [calling any web api](https://docs.microsoft.com/xamarin/cross-platform/data-cloud/web-services/?WT.mc_id=mobile-0000-masoucou).

```language-csharp
public async Task<WeatherInfo> GetWeatherInfo(double latitude, double longitude)
{
    try
    {
        var infoJson = await httpClient.GetStringAsync($"{functionsUrl}?lat={latitude}&long={longitude}");

        var weatherInfo = JsonConvert.DeserializeObject<WeatherInfo>(infoJson);

        return weatherInfo;
    }
    catch (Exception ex)
    {
        System.Diagnostics.Debug.WriteLine(ex);
    }

    return null;
}
```

That `httpClient` variable is of the type `System.Net.HttpClient` - new'd up with the default constructor. That's it. Done and done.

## App Center

Now [App Center](https://docs.microsoft.com/appcenter/?WT.mc_id=mobile-0000-masoucou). App Center is known for its DevOps functionality. You can hook it up to a source control repository and have it build on every push. Have it distribute to testers or the store. Have it track crashes and analytics.

But recently App Center has been developing a Mobile Backend as a Service functionality that makes it [easier to interact with Azure AD B2C](https://docs.microsoft.com/appcenter/auth/?WT.mc_id=mobile-0000-masoucou) for letting users create accounts and sign-in to your apps. They have also been working with Azure Cosmos DB to make it [better to work with data](https://docs.microsoft.com/appcenter/data/?WT.mc_id=mobile-0000-masoucou) - both online and off - seamlessly. Including having it work with with Identity so you can store personal data without having to worry about handling the tokens from the AD B2C instance yourself!

The MBaaS portion of App Center does have a bit of ceremony to setup - so follow the [steps here for Identity](https://docs.microsoft.com/appcenter/sdk/auth/xamarin?WT.mc_id=mobile-0000-masoucou). And [here for Data](https://docs.microsoft.com/appcenter/sdk/data/xamarin?WT.mc_id=mobile-0000-masoucou). But once you're up and running interacting with them from a Xamarin app couldn't be easier. Seriously.

To sign somebody in:

```language-csharp
var userInfo = await Auth.SignInAsync();
```

Yeah, that's it.

Then to save some data to a user's private stash in Cosmos DB

```language-csharp
await Data.CreateAsync<CityInfo>($"{city.CityName.Replace(" ",string.Empty)}-{city.State}", city,
        DefaultPartitions.UserDocuments);
```

That first parameter specifies the ID to be used in Cosmos. The second one the object to store. Then the last one where you want to store it. If you use `DefaultPartitions.UserDocuments`, the App Center SDK will use the signed-in user from the Auth portion and do everything. You don't have to worry about handling tokens yourself.

Similarly to read documents for a particular user:

```language-csharp
public async Task<IEnumerable<CityInfo>> GetSavedCities()
{
    var allSavedCities = new List<CityInfo>();

    var savedCitiesFromCosmos = await Data.ListAsync<CityInfo>(DefaultPartitions.UserDocuments);

    allSavedCities.AddRange(savedCitiesFromCosmos.CurrentPage.Items.Select(ci => ci.DeserializedValue));

    while (savedCitiesFromCosmos.HasNextPage)
    {
        await savedCitiesFromCosmos.GetNextPageAsync();

        allSavedCities.AddRange(savedCitiesFromCosmos.CurrentPage.Items.Select(ci => ci.DeserializedValue));
    }

    return allSavedCities;
}
```

There's a bit more there - but that's because App Center returns data in pages - so it doesn't return everything all at once. But the key here is `Data.ListAsync<CityInfo>(DefaultPartitions.UserDocuments);` That's going to use the signed in user - and only return they're documents that match the `CityInfo` data type.

The App Center SDKs are awesome. Check them out.

## Summing It Up

So while you won't be able to build a cloud connected app in minutes - there are a ton of tools at your disposal to let you build one in a straight forward manner.

* Xamarin - for creating iOS and Android (and other) apps while sharing a ton of code across the platforms
* Azure Functions - for creating a Web API that lets you concentrate on your code and nothing else
* App Center - a new Mobile Backend as a Service that lets you connect to an AD B2C and Cosmos DB instance and handles all the hard work for you!

If you have any questions - reach out anytime at [@codemillmatt](https://twitter.com/codemillmatt) - I'd love to hear from you!