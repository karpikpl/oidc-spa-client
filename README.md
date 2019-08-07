# Building OIDC SPA Client
start by reading:
https://identityserver4.readthedocs.io/en/latest/

They also have few videos:
https://identityserver4.readthedocs.io/en/latest/misc/videos.html

Try going through quick starts and playing with the code.

Once you have good understanding of OIDC (Open ID Connect) try your knowledge on one of the playgrounds (grown-up kind)
https://www.oauth.com/playground/
(0Auth and Octa have pretty good docs on the subject as well)

# Workshop

## Build the API
```bash
mkdir api && cd api
dotnet new webapi
dotnet new sln -n api
dotnet sln add api.csproj
```
*Optional steps*
```
curl https://raw.githubusercontent.com/github/gitignore/master/VisualStudio.gitignore -o .gitignore
git init
git add .
git commit –am “init commit”
```

## Change the port
In *launchSettings.json*:
```json
"api": {
  "commandName": "Project",
  "launchBrowser": true,
  "launchUrl": "api/values",
  "applicationUrl": "https://localhost:6001;http://localhost:6000",
```

## Secure the API
In *Startup.cs*: 
```csharp
services
  .AddMvcCore()
  .AddJsonFormatters()
  .AddAuthorization()
  .SetCompatibilityVersion(CompatibilityVersion.Version_2_2);

 services.AddAuthentication("Bearer")
  .AddJwtBearer("Bearer", options =>
  {
      options.Authority = "https://demo.identityserver.io";
      options.Audience = "api";
  });
  ```
In *Configure(...)*:
```csharp
app.UseAuthentication();
app.UseMvc();
```

## Create controler to secure
```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace api.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class SecureController : Controller
    {
        [HttpGet]
        public IActionResult Info()
        {
            return new JsonResult(
                from c in User?.Claims
                select new { c.Type, c.Value });
        }
    }
}
```

## Test 1 - run and call the API
```bash
dotnet watch run
curl https://localhost:6001/api/secure -k -v
```

## Secure the API
```csharp
    [Authorize]
    [HttpGet]
    public IActionResult Info()
```

## Try again calling the API
```bash
curl https://localhost:6001/api/secure -k -v
```

## Trying to get in - get the token
Read https://identityserver4.readthedocs.io/en/latest/endpoints/token.html ​
get the token from https://demo.identityserver.io

## Get the token
```bash
curl -v -X POST https://demo.identityserver.io/connect/token -d "client_id=client&client_secret=secret&grant_type=client_credentials&scope=api" | json_pp
```
## Call the API
```bash
curl https://localhost:6001/api/secure -v -k --header "Authorization: Bearer xxx" | json_pp
```
where *xxx* is the **access_token**

## Inspect the token
http://jwt.io

## Create the SPA
Using samples from this repo

## Run the SPA
```bash
npx http-server
```

## Fix CORS in the API
In *ConfigureServices(..)*:

```csharp
 services.AddCors(c => c.AddDefaultPolicy(builder =>
    {
        builder
            .AllowAnyOrigin()
            .AllowAnyHeader()
            .AllowAnyMethod();
    }));
```
In *Configure(..)*:
```csharp
app.UseCors();
app.UseMvc();
```
