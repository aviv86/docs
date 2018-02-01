# Troubleshooting : Sending support ticket

When sending a support ticket, it is good to include as much information about the issue as you can. The following article describes what can be sent along with the issue description to let us better understand your problem.

1. Server and client version
2. Web Traffic logs. We recommend using a [FiddlerCap](http://www.telerik.com/fiddler/fiddlercap) recorder which will create a SAZ file that we can analyze further:
    - [Download](http://www.telerik.com/fiddler/fiddlercap) FiddlerCap
    - Close all instances of Internet Explorer (or other browsers).
    - Run the FiddlerCapSetup.exe file.
    - FiddlerCap will start automatically when the installer completes.
    - Inside FiddlerCap press Clear Cookies button and then the Clear Cache button.
    - Inside FiddlerCap, click the **1. Start Capture** button.
    - Record web traffic using your application.
    - To add a screenshot to your capture, press the +Screenshot button inside FiddlerCap.
    - Inside FiddlerCap, click the **2. Stop Capture** button.
    - Click the **3. Save Capture** button. Save the .SAZ file to your desktop.
3. Debug Info Package. RavenDB supports creating info package (replication, performance, i/o, storage, memory, etc.) that can be created for the entire cluster or for current server only. Go to `Studio -> Manage Server -> Gather Debug Info`.
4. [Server logs](../../server/troubleshooting/enabling-logging)
5. [Statistics](../../server/administration/statistics)
6. Unit test

{INFO: Monitoring local traffic }
The easiest way to monitor traffic sent to `http://localhost` or `http://127.0.0.1` is to provide a machine name instead of **localhost** or **127.0.0.1**.
For example if your RavenDB server address is `http://localhost:8080` then in an application config file change it to  `http://[machine-name]:8080`. You can also use one of these two aliases instead:  `http://ipv4.fiddler`, `http://localhost.fiddler`.

For an ASP.NET application you can also configure proxy server as follow:

{CODE-START:xml /}
<system.net>
  <defaultProxy>
    <proxy bypassonlocal="False" usesystemdefault="True" proxyaddress="http://127.0.0.1:[port number]" />
  </defaultProxy>
</system.net>
{CODE-END /}

For an ASP.NET CORE application, instruction on how to configure proxy server can be found in this [article](http://josephwoodward.co.uk/2016/07/proxying-http-requests-asp-net-core-using-kestrel)  

By default Fiddler listens on port `8888` and FiddlerCap on `8889`.
{INFO/}

## Writing unit tests

The NuGet package has been created for easier RavenDB test creation and can be downloaded [here](https://www.nuget.org/packages/RavenDB.TestDriver/).

The package contains base class [RavenTestDriver](../../start/test-driver) with various methods useful for test creation:

- `GetDocumentStore`
- `WaitForIndexing`
- `WaitForUserToContinueTheTest`
- ...and more

{CODE support_1@Server\Troubleshooting\Support.cs /}
