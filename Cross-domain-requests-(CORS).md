To allow a browser to upload files from a page on a different domain you will need to enable cross origin resource sharing (CORS).

*Note*: There is an open issue on creating a helper for getting exposed headers or the entire cors policy: https://github.com/smatsson/tusdotnet/issues/19

# ASP.NET 4.x

Install package Microsoft.Owin.Cors and modify your Startup class as below.

```
public void Configuration(IAppBuilder app)
{
    var corsPolicy = new System.Web.Cors.CorsPolicy
    {
        AllowAnyHeader = true,
        AllowAnyMethod = true,
        AllowAnyOrigin = true
    };

    // ExposedHeaders has a private setter for some reason so one must use reflection to set it.
    corsPolicy.GetType()
        .GetProperty(nameof(corsPolicy.ExposedHeaders))
        .SetValue(corsPolicy, tusdotnet.Helpers.CorsHelper.GetExposedHeaders());

    app.UseCors(new CorsOptions
    {
        PolicyProvider = new CorsPolicyProvider
        {
            PolicyResolver = context => Task.FromResult(corsPolicy)
        }
    });

    app.UseTus(...);
}
```

# ASP.NET Core

Install package Microsoft.AspNetCore.Cors and modify your Startup class as below.

```
public void ConfigureServices(IServiceCollection services)
{
	services.AddCors();
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
	app.UseCors(builder => builder
                .AllowAnyHeader()
                .AllowAnyMethod()
                .AllowAnyOrigin()
                .WithExposedHeaders(tusdotnet.Helpers.CorsHelper.GetExposedHeaders())
        );
	app.UseTus(...);
}

```