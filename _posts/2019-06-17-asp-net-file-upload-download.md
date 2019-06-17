---
layout: post
title: "Uploading and downloading files with ASP.NET WEB API"
category: homework
date: 2019-06-17
---

Code from the book *ASP.NET Web API 2 Recipes*, Section 4-10. Serve Binary Data from ASP.NET Web API

## Serving Binary Data as *StreamContent*

```csharp
public HttpResponseMessage Get(string filename) {
    var path = Path.Combine(ROOT, filename);
    // ROOT would be in our case equal to HttpContext.Current.Server.MapPath("~/App_Data");
    if (!File.Exists(path))
    throw new HttpResponseException(Request.CreateErrorResponse(HttpStatusCode.NotFound));
    // The above line will not work with my setup
    // When I change it to 
    // throw new HttpResponseException(Request.CreateErrorResponse(HttpStatusCode.NotFound, "Not found"));
    // It will compile but the Exception handling will not be elegant

    var result = new HttpResponseMessage(HttpStatusCode.OK);
    var stream = new FileStream(path, FileMode.Open);
    result.Content = new StreamContent(stream);
    result.Content.Headers.ContentType = new MediaTypeHeaderValue("application/octet-stream");
    // There are different options and different outcomes based on how you set up the ContentType
    // and ContentDisposition headers
    // One example:
    // result.Content.Headers.ContentDisposition = new ContentDispositionHeaderValue("attachment") { FileName = name };
    return result;
}
```

## Serving Binary Data as *ByteArrayContent*

```csharp
public HttpResponseMessage Get(string filename) {
    var data = GetFromSomewhere(filename);
    if (data = null)
    throw new HttpResponseException(Request.CreateErrorResponse(HttpStatusCode.NotFound));

    var result = new HttpResponseMessage(HttpStatusCode.OK);
    result.Content = new ByteArrayContent(data);
    result.Content.Headers.ContentType = new MediaTypeHeaderValue("application/octet-stream");
    return result;
}
```

## Serving Binary Data as *MultipartContent*

```csharp
public HttpResponseMessage Get(string file1, string file2) {
    var fileA = new StreamContent(new FileStream(Path.Combine(ROOT, file1), FileMode.Open));
    filaA.Headers.ContentType = new MediaTypeHeaderValue("image/jpeg");

    var fileB = new StreamContent(new FileStream(Path.Combine(ROOT, file2), FileMode.Open));
    filaB.Headers.ContentType = new MediaTypeHeaderValue("image/jpeg");

    var result = new HttpResponseMessage(HttpStatusCode.OK);
    result.Content = new MultipartContent();
    result.Content.Add(fileA);
    result.Content.Add(fileB);

    return result;
}
```

*TO-DO: Add the upload options*

Relevant sections from the MSDN documentation: [Sending HTML Form Data](https://docs.microsoft.com/en-us/aspnet/web-api/overview/advanced/sending-html-form-data-part-2), [Action Results in Web API 2](https://docs.microsoft.com/en-us/aspnet/web-api/overview/getting-started-with-aspnet-web-api/action-results).

Classes used: [StreamContent](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.streamcontent), [HttpResponseMessage](https://docs.microsoft.com/en-us/dotnet/api/system.net.http.httpresponsemessage).

Also relevant: [Why should I use IHttpActionResult instead of HttpResponseMessage](https://stackoverflow.com/questions/21758615/why-should-i-use-ihttpactionresult-instead-of-httpresponsemessage).