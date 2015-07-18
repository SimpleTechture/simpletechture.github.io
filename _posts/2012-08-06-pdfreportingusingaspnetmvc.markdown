---
layout: post
title: PDF reporting using ASP.NET MVC
tags:
- ASP.NET MVC
- Reporting
---

[Source on GitHub]()

![Report Preview](../../../img/ASP_MVC3_PDF_Reporting_Preview.jpg)

#Introduction
Almost every web application (that I have built) needs some kind of reporting. Many times the client wants to generate a PDF report that corresponds with the web page he or she is viewing. Using [iTextSharp](http://sourceforge.net/projects/itextsharp/), a free C# PDF library, this is possible. This article describes and includes a solution for creating reports using [iTextSharp](http://sourceforge.net/projects/itextsharp/) and ASP.NET MVC3.

#Background
[iTextSharp](http://sourceforge.net/projects/itextsharp/) is a free C# PDF library that is ported from the Java-PDF library [iText](http://itextpdf.com/).[iText](http://itextpdf.com/) was launched in 2000 and is a popular Open Source Java library for programmatic creation and manipulation of PDF. As with many successful Open-Source Java libraries, it was ported to C# under the name iTextSharp. iTextSharp has been under development since 2008 and is distributed under the [GNU Affero General Public License version 3](http://www.gnu.org/licenses/agpl.html).

At its core, [iTextSharp](http://sourceforge.net/projects/itextsharp/) is an API to manipulate PDF creation. For a recent project, I was looking for a solution that would enable me to create PDF documents from existing HTML documents or Views used in an ASP.NET MVC3 project. The optimal solution for me would be to throw an existing ASP.NET MVC3 view at it and let it generate a PDF that is exactly the same as the HTML rendered by a browser.

The solution described here uses the <code>HTMLWorker</code> class from [iTextSharp](http://sourceforge.net/projects/itextsharp/) to generate a PDF from an HTML view. HTMLWorker is an [iTextSharp](http://sourceforge.net/projects/itextsharp/) class that is able to parse an HTML document and generate a PDF using what is called the <code>SimpleParser</code> of [iTextSharp](http://sourceforge.net/projects/itextsharp/). Notice the name <code>SimpleParser</code> which indicates that not all HTML elements and CSS styles are supported. However, I was able to get good results using this solution.

To add reporting to your ASP.NET MVC3 project, first add the PDFReportGenerator project to your solution and add the <em>iTextSharp.dll</em> binary to your solution. PDFReportGenerator needs a reference to the binary. By adding the binary to the solution, you are able to build the project directly from source control. Check the demo source for an example.

#Creating a report
Follow these steps to generate an actual report from your web application:

- Create a controller that derives from <code>PdfViewController</code>
- Create a view that generates the HTML which should be translated to a PDF repor
- Create an action on a controller which calls the <code>ViewPDF</code> method on the <code>PdfViewController</code>
- Create a link to trigger the action on the controller

##Create a controller that derives from PdfViewController
Create a new or reuse an existing controller and let it derive from <code>PdfViewController</code> from the PdfReportGenerator project. This enables your controller to call 
the <code>ViewPDF</code> method of the <code>PDFViewController</code> which generates the actual PDF. In the demo project, this is the <code>HomeController</code>.

##Create a view that generates the HTML
Create a view that should be translated to a report. This could be an existing view or a new view specially for reporting. I usually create a new view as it lets me control 
the HTML markup for the report. As stated earlier, the report generator does not support all the HTML markup. In the demo project, this is the <code>PrintDemo</code> view.

Below, the PrintDemo view from the demo project is shown. As can be seen, this is just a simple ASP.NET Razor view with a table and some rows. 
It uses a strongly typed model but that is not necessary. A <strong>tip</strong> when trying to design your report is to add borders to your <code>table</code> or <code>div</code>. 
Using these borders, when looking at the generated PDF, you can clearly see the start and end of the areas of your report.

{% highlight xml %}
@model CustomerList
<br />
<table cellpadding="3" cellspacing="3">
    <tr border="1" bgcolor="#777777" color="#ffffff">
        <td>Name</td>
        <td>Address</td>
        <td>Place</td>
    </tr>
    @foreach (Customer customer in Model)
    {
        <tr border="1">
            <td>@customer.Name</td>
            <td>@customer.Address</td>
            <td>@customer.Place</td>
        </tr>
    }
</table>
{% endhighlight %}

##Create an action which calls the ViewPDF method
The <code>PdfViewController</code> class from which your controller derives contains a <code>ViewPDF</code> method. This method has the following signature:

{% highlight csharp %}
protected ActionResult ViewPdf(string pageTitle, string viewName, object model)
{% endhighlight %}

**Parameters**
| pageTitle | The title of the report which appears on the header of the page
| viewName| The name of the view which should be converted to a report
| model| The model that is rendered by the view.

This methods generates the HTML view and converts it into a PDF report and sends this PDF as a binary stream back to the client. This means that when the client has a PDF plug-in installed, the PDF is shown inside the browser.

From an action inside your controller, this method should be called to generate the report and send it to the client. The following action from the demo application generates the PDF. "Customer report" is the title of the report, "PrintDemo is the name of the view, and the model is returned by the <code>CreateCustomerList()</code> which as its name implies generates a dummy list with customers.

{% highlight csharp %}
public ActionResult PrintCustomers()
{
   return this.ViewPdf("Customer report", "PrintDemo", CreateCustomerList());
}
{% endhighlight %}

The last step is to create a link on a page that calls this action to actually print the report.

##Trigger the action on the controller
A simple method to create a link to trigger the action on the controller is by using an <code>ActionLink</code>. This link calls the action that we defined on the controller.

{% highlight csharp %}
@Html.ActionLink("Print customers", "PrintCustomers", null,  new { target = "_blank" })
{% endhighlight %}

These are the steps you need to be able to create PDF reports from your ASP.NET MVC3 projects. Read the next part of the article if you are interested in the details of how the PdfReportGenerator actually converts the ASP.NET MVC3 view into a report.

#Detailed Reporting Project overview
The PdfReportGenerator project consist of six classes which can be seen in the image below. The PdfReportGenerator assembly works by rendering the ASP.NET MVC3 view into a string and converting this string with HTML using the iTextSharp into a PDF report.

![Classes](../../../img/ASP_MVC3_PDF_Reporting_2.png)

#Rendering an ASP.NET MVC3 view into a string
The class <code>HtmlViewRenderer</code> is responsible for rendering the ASP.NET view into a string. The method <code>RenderViewToString</code> has the following signature.

{% highlight csharp %}
public string RenderViewToString(Controller controller, string viewName, object viewData)
{% endhighlight %}

The first argument is <code>viewName</code> which is the name of the view that should get rendered to a <code>string</code> including the model <code>viewData</code> that is needed by the view. The controller is necessary to be able to use to render the view using the view engine of ASP.NET MVC.

##Convert the HTML string into a PDF byte array
Once we have the HTML in a string, <code>StandardPdfRenderer</code> converts the HTML string into a PDF byte array. The class <code>StandardPdfRenderer</code> has a method <code>Render</code> with the following signature.

{% highlight csharp %}
public byte[] Render(string htmlText, string reportTitle)
{% endhighlight %}

<code>htmlText</code> is the rendered view as a string that was produced by the <code>HtmlViewRender</code>; <code>pageTitle</code> is, as the name suggests, the title of the report.

##Send the byte array back to the client as a stream
The last step is to convert the byte array into an instance of the <code>BinaryContentResult</code> class. The class <code>BinaryContentResult</code> derives from the ASP.NET MVC
<code>ActionResult</code>. It overrides <code>ExecuteResult</code> and returns the content as a binary stream.

The class <code>PdfViewController</code> is the class that combines these classes. The method <code>ViewPdf</code> uses all the three previously mentioned classes to generate the PDF as shown in the code below:

{% highlight csharp %}
protected ActionResult ViewPdf(string pageTitle, string viewName, object model)
{
    // Render the view html to a string.
    string htmlText = this.htmlViewRenderer.RenderViewToString(this, viewName, model);

    // Let the html be rendered into a PDF document through iTextSharp.
    byte[] buffer = standardPdfRenderer.Render(htmlText, pageTitle);

    // Return the PDF as a binary stream to the client.
    return new BinaryContentResult(buffer, "application/pdf");
}
{% endhighlight %}

This is in the controller.

#Points of interest
Some remarks on using iTextSharp

##Colors
iTextsharp supports colors out of the box; in the demo application, the background colors of the rows are alternated using different colors. These colors are visible in the report.

##New page support
One thing that I needed with my project that was not supported by the HTML conversion in iTextSharp was functionality to force a page break. Most of the time, with reporting, you need to be able to force a page break, for example, if you want a graph to start always on a new page. The way I solved this was to add support for it to iTextSharp. As it is Open-Source, you are able to add new features to it. I added support for a non-existing HTML tag called <code><np /></code>  which forces iTextSharp to create a new page. I created a patch so that it could be committed to the <a href="http://sourceforge.net/tracker/?func=detail&amp;aid=3396638&amp;group_id=72954&amp;atid=536238">iTextSharp project</a>.I do not know if it will be included in the main trunk of the project as it feels somewhat strange to invent new HTML tags just to support a page break. But if you need it, you can use the patch to compile iTextSharp with new page support.
    
##Images
It is possible to add images to the report by using an <code>&lt;img src="" /&gt;</code> tag. I had no success with dynamically generated images. So I generate the images that I needed before the action conversion process takes place. A single static image can be seen in the report.
    
##ASP.NET support
It should be possible to use the same kind of solution using older versions of ASP.NET MVC. However, I have not tried it.
