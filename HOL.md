# Part 1: Create the Main Solution
- From Visual Studio 2017 create new solution `ASP.NET Web Application .NET Framework`.
- Choose **Web API** from the template box.
- Change the Authetication to **Individual User Accounts**
- Click OK

# Part 2: Force HTTPs
- Right + click on the project and create new folder **Filters**
- Add new class `RequireHttpsAttribute.cs` inside the folder **Filters**
- Add the following directive:
```
using System.Net;
using System.Net.Http;
using System.Web.Http.Controllers;
using System.Web.Http.Filters;
```
- Add the following code snippet inside the class:
```
        public int Port { get; set; }

        public RequireHttpsAttribute()
        {
            Port = 443;
        }

        public override void OnAuthorization(HttpActionContext actionContext)
        {
            var request = actionContext.Request;

            if (request.RequestUri.Scheme != Uri.UriSchemeHttps)
            {
                var response = new HttpResponseMessage();

                if (request.Method == HttpMethod.Get || request.Method == HttpMethod.Head)
                {
                    var uri = new UriBuilder(request.RequestUri);
                    uri.Scheme = Uri.UriSchemeHttps;
                    uri.Port = this.Port;

                    response.StatusCode = HttpStatusCode.Found;
                    response.Headers.Location = uri.Uri;
                }
                else
                {
                    response.StatusCode = HttpStatusCode.Forbidden;
                }

                actionContext.Response = response;
            }
            else
            {
                base.OnAuthorization(actionContext);
            }
        }
```
- In the **App_Start** folder:
  - Register the *RequireHTTPs* by appending `filters.Add(new RequireHttpsAttribute());` to `RegisterGlobalFilters` method inside the **FilterConfig.cs** class.
  - Insert the following configuration `config.Filters.Add(new Filters.RequireHttpsAttribute());` at the begining of `Register` method inside **WebApiConfig** class.
  The method at the begining will looks like:
  ```
        public static void Register(HttpConfiguration config)
        {
            // Web API configuration and services
            config.Filters.Add(new Filters.RequireHttpsAttribute());
  ```
- Select the project and press F4 in order to display the property window.
- Set **SSL Enabled** attribute value to **True**.
- Copy the **SSL URL** attribute value that begins with *https*.
- Select the project and press Shift+F4 and the Propoerty pages will appear and select the *Web* tab.
- Replace the *Project URL* with the *SSL URL* that copied from the propoerty window, the press Ctrl+S to save.
- Press F5 and you will notice the local IIS runs the project URL with HTTPs.

*Be noted that some browsers might not verify the SSL running by local ISS and it will ask from user to accept the risk. This can be configured by creating self-signed certificate, or will run proporly on production level if your domain supports SSL from a trusted authority*

# Part 3: Authorization
We will make the authorization the Values API which is default controller added automatically when project created.
- Go to ValuesController and add `[Authorize]` attribute at the top of the class:
```
[Authorize]
public class ValuesController : ApiController
{
        ...
```
- Press F5 to run the application
- From REST client tool:
  - Register a user first
  ```
        POST /api/account/register HTTP/1.1
        Host: (PROJECT URL)
        Content-Type: application/json
        
        Request Body:
        {
                "email" : "user1@example.com",
                "password" : "P@ssw0rd",
                "confirmpassword" : "P@ssw0rd"
        }
  ```
  - Generate Token:
  ```
        POST /token HTTP/1.1
        Host: (PROJECT URL)
        Content-Type: application/x-www-form-urlencoded
        Request Body: username=user1@0example.com&password=P@0ssw0rd&grant_type=password
  ```
  - Call Values API:
  ```
        GET /api/values HTTP/1.1
        Host: (PROJECT URL)
        Authorization: Bearer TOKEN_HERE
  ```
