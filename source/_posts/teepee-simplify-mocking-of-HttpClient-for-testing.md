---
title: "Introducing TeePee: Simplify mocking of HttpClients for testing"
date: 2023-08-10 08:24:18
tags:
  - Testing
  - Mocks
  - HTTP
---

### Introducing TeePee: A fluent API library for mocking HttpClient and HttpClientFactory for testing

**_A clean way to set up your HTTP mocks_**

[![TeePee Logo](https://raw.githubusercontent.com/oatsoda/TeePee/main/TeePee/teepee-icon.png)](https://github.com/oatsoda/TeePee)

I started the [TeePee](https://github.com/oatsoda/TeePee) (The name is based on the last two characters of HT*TP*) library a couple of years ago and it has since become battle tested and proven to work quite nicely.

It's quite common to need to mock your HTTP responses and confirm that you're sending the expected data to the correct place. This can be quite painful to set up, especially when using HttpClientFactory and the various ways you can inject HttpClients.

Here is the [Nuget Package](https://www.nuget.org/packages/TeePee/).

All of the various ways you can use it are documented in the [Readme](https://github.com/oatsoda/TeePee#readme), including both a) if you cover DI in behavioural tests or b) if you want to manually inject HttpClient/HttpClientFactory into your SUT in a unit test.

There's also [an extension Package for Refit](https://www.nuget.org/packages/TeePee.Refit/). Why wouldn't I just mock the refit interface I hear you say? Well you might decide that's enough for your situation, but it won't mean that all the setup code in your startup is covered by the tests - such as Base URI and any Http Handlers which often configure authentication for third party APIs.

Here's an example of mocking a GET request and using a simple _manual_ injection of a Named HttpClient:

```[csharp]
// Subject Under Test
public class ToTest
{
   public ToTest(HttpClientFactory httpClientFactory)
   {
      var httpClient = httpClientFactory.CreateClient("MyHttpClient");
      // ... etc.
   }
}

// Test Code
var builder = new TeePeeBuilder("MyHttpClient");
builder.ForRequest("https://some.api/path/resource", HttpMethod.Post)
       .ThatHasBody(new { Value = 12 })
       .ThatContainsQueryParam("filter", "those")
       .ThatContainsHeader("ApiKey", "123abc-xyz987")
       .Responds()
       .WithStatus(HttpStatusCode.Created)
       .WithBody(new { Id = Guid.NewGuid() });

var httpClientFactory = builder.Build()
                               .Manual("https://some.api")
                               .CreateHttpClientFactory();
/*
Sadly, if your production code DI registers the BaseURL then you have to duplicate that in the test, passing it to .Manual(); no coverage with manual injection like this.
*/

var sut = new ToTest(httpClientFactory);
// ... etc.
```

And here's an example of using Refit and also covering startup registrations in the test:

```[csharp]
/* --- Production Code --- */

public interface IApiService
{
    [Get("/users/{user}")]
    Task<User> GetUser(string user);
}

public static class GitHubApiStartupExtensions
{
   // Extension called from Startup class in production code
   public static void AddGitHubApi(this IServiceCollection services)
   {
      services.AddRefitClient<IApiService>()
              .ConfigureHttpClient(c => c.BaseAddress = new("https://api.github.com"));
   }
}

public class SomeLogic
{
   private IApiService _apiService;
   public SomeLogic(IApiService api) { _apiService = api }

   public async Task<string> Greeting(string userId)
   {
      var name = await _apiService.GetUser(userId);
      return $"Hello, {name}";
   }
}

/* --- Tests --- */

// In reality, this wouldn't be in the test, you would call production code to setup registrations
var services = new ServiceCollection();
services.AddGitHubApi();
services.AddTransient<SomeLogic>();

// Test setup
var builder = new TeePeeBuilder();
builder.ForRequest("https://api.github.com/users/abc-123", HttpMethod.Get)
       .Responds()
       .WithBody(new { Name = "User's Name" })
       .WithStatus(HttpStatusCode.OK);

services.AttachToRefitInterface<IApiService>(builder.Build());

// Simulate Production Code - you'd probably be testing some edge API not just a class like this
var greeting = await services.BuildServiceProvider().GetRequiredService<SomeLogic>().Greeting("abc-123");

Assert.Equal("Hello, User's Name", user.Name);
```

All the documentation is in the [Readme](https://github.com/oatsoda/TeePee#readme) so I would start there. There are also [Examples](https://github.com/oatsoda/TeePee/tree/main/Examples).

Any feedback or issues welcome, either via [GitHub](https://github.com/oatsoda/TeePee/issues) or [Twitter](https://twitter.com/oatsoda).
