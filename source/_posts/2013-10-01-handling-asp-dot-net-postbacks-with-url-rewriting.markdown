---
layout: post
title: "Handling ASP.NET PostBacks with URL Rewriting"
date: 2013-10-01 22:18
comments: true
categories: [ASP.NET, C#, Web development]
keywords: "asp.net, C#, web, url rewriting"
---
Recently a collegue develop crossed with this strange error **_Validation of viewstate MAC failed..._** when posting a contacts form. Strangely the error only happened when using the re-written url off the contacts page (`contacts/`), calling `contacts.aspx` directly worked flawlessly. This is a C# port based on ScottGu's [solution](http://weblogs.asp.net/scottgu/archive/2007/02/26/tip-trick-url-rewriting-with-asp-net.aspx).<!-- more -->

A few minutes afeter I’ve remembered that I had to strugle with this issue when implementing a website using url-rewriting with IIRF. The problem is that when using url-rewriting the `<form>` control does not render the proper url but the re-written one. The solution is to write a form control adapter and I found, at the time I had the problem, the right solution from ScottGu's. Since my language of choice is C# I’m going to share the solution right here:

## Step 1
**Create a FormRewriterControlAdapter class**

```  
using System;
using System.Data;
using System.Configuration;
using System.Linq;
using System.Web;
using System.Web.Security;
using System.Web.UI;
using System.Web.UI.HtmlControls;
using System.Web.UI.WebControls;
using System.Web.UI.WebControls.WebParts;
using System.Xml.Linq;

public class FormRewriterControlAdapter : System.Web.UI.Adapters.ControlAdapter
{
    protected override void Render(HtmlTextWriter writer)
    {
        base.Render(new RewriteFormHtmlTextWriter(writer));
    }
}

public class RewriteFormHtmlTextWriter : HtmlTextWriter
{
    public RewriteFormHtmlTextWriter(HtmlTextWriter writer) : base(writer)
    {
        this.InnerWriter = writer.InnerWriter;
    }

    public RewriteFormHtmlTextWriter(System.IO.TextWriter writer) : base (writer)
    {
        base.InnerWriter = writer;
    }

    public override void WriteAttribute(string name, string value, bool fEncode)
    {
        // If the attribute we are writing is the "action" attribute, and we are not on a sub-control,
        // then replace the value to write with the raw URL of the request - which ensures that we'll
        // preserve the PathInfo value on postback scenarios

        if (name == "action")
        {
            HttpContext context = HttpContext.Current;

            if (context.Items["ActionAlreadyWritten"] == null)
            {
                // Because we are using an url reweriting HttpModule, we will use the
                // Request.RawUrl property within ASP.NET to retrieve the origional URL
                // before it was re-written.  You'll want to change the line of code below
                // if you use a different URL rewriting implementation.

                // value = context.Request.RawUrl;
                value = context.Request.ServerVariables["HTTP_X_REWRITE_URL"];

                // Indicate that we've already rewritten the <form>'s action attribute to prevent
                // us from rewriting a sub-control under the <form> control

                context.Items["ActionAlreadyWritten"] = true;
            }
        }

        base.WriteAttribute(name, value, fEncode);
    }
}
```
## Step 2
**Add adapter to App_Browsers**

1. Add `ASP.NET` folder if it does not exists;
2. Create `Form.browser`:

{% codeblock %}
<browsers>
    <browser refID="Default">
        <controlAdapters>
            <adapter controlType="System.Web.UI.HtmlControls.HtmlForm"
                adapterType="FormRewriterControlAdapter" />
        </controlAdapters>
    </browser>
</browsers>
{% endcodeblock %}

I hope you’ll find this information useful.
