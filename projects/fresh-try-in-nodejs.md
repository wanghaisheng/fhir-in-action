By Rob Mulders – During the May 2015 FHIR connectathon in Paris, somebody asked me how much time it would take him to make his first FHIR client application. It inspired me to write a sort of Hello World program and see how much time it would take me. You have to know that I haven’t been programming for many years. The last time was about 12 years ago in Delphi. And before that I used to code in FoxPro 2.6 for DOS.

In this blog I would like to share with you the steps needed to create a very simple FHIR client. First, the client connects to a FHIR server on the internet. After that, the client fetches all patients from the server having “Rob” in the name, to show them in a form.

Step 1 – Create a project

This step assumes you have Visual Studio ready on your machine. I used Visual Studio 2012 Professional. Fire up Visual Studio without any old project open. Choose File – New – Project from the menu. In the dialog box, choose Visual C# – Windows – Windows Form Application. In the Name field, enter the name you want to have for your first program. Then click OK. Visual Studio creates a solution with a form called Form1. Press F5 or click on Start to run and test your program. If everything is ok, you will see the following screen:

MyFirstFhirProject-Img1

Step 2 – Get the .NET API for connecting to FHIR servers

Since your program wants to connect to a FHIR server without doing complex stuff yourself, you will need the .Net API that is provided by my colleagues at Furore. On the menu, choose Project – Manage NuGet Packages. A dialog box will appear. In the upper right corner, type FHIR in the search box. A list of about six APIs will appear. Make sure you choose HL7 FHIR Core support API for DSTU2. Then click on Install. If everything is ok, a green marker will appear. If not, I don’t know what to do :-). The NuGet manager will automatically include the JSON.net package for you, since the FHIR API is dependent on that one. Close the dialog box and run your program again to verify it still works.

Step 3 – Searching patients on a FHIR server

I chose to use a button to start the connection to a FHIR-server. On the menu click View – Toolbox and search for button. Click on Button and drag a button on your form. Widen the button a bit to be able to display some text in it. Then double click on your own button. You will come to a piece of code called button1_Click. In this function, copy a few lines of code from the box below into your click function. You should have your function as follows:

private void button1_Click(object sender, EventArgs e)
{
   var endpoint = new Uri("http://spark-dstu2.furore.com/fhir"));
   var client = new FhirClient(endpoint);

   var query = new string[] { "name = Rob" };
   var bundle = client.Search("Patient", query);

   button1.Text = "Got " + bundle.Entry.Count() + " records!";
}
Now run your program. You will get an error while compiling! Visual Studio says “The type or namespace FhirClient could not be found”. What we have to do is include the FHIR API classes in our code module, by typing three lines of code in the top part. In your source code, scroll up and add three using statements below the others:

using Hl7.Fhir;
using Hl7.Fhir.Rest;
using Hl7.Fhir.Model;
Run your program again. This time, it rolls without compile errors. Click on your own button. If everything goes right, you should see the form like this:

MyFirstFhirProject-Img2

Step 4 – Displaying patient information from a FHIR bundle

In your code, you have created a variable called bundle. A bundle is the FHIR way to have a collection of data. Actually, the bundle not only knows how much records it got from the FHIR server, it also contains the data itself. So, to display the patients, we need to display the data from the bundle onto the form.

First, add a label to your form to display information in. On the menu click View – Toolbox and search for label. Click on Label and drag a label on your form. Click on your form to activate it. Put the label a bit to the left under your button to make space for showing a list.

Now, go to your code and add some lines below your last line of code. In total, the function should look like this:

private void button1_Click(object sender, EventArgs e)
{
   var endpoint = new Uri("http://spark-dstu2.furore.com/fhir"));
   var client = new FhirClient(endpoint);

   var query = new string[] { "name = Rob" };
   var bundle = client.Search("Patient", query);

   button1.Text = "Got " + bundle.Entry.Count() + " records!";

   label1.Text = "";
   foreach (var entry in bundle.Entry)
   {
      Patient p = (Patient)entry.Resource;
      label1.Text = label1.Text + p.Id + " " + p.Name.First().Text + "\r\n";
   }
}
If you run your program, you will get output like this:

MyFirstFhirProject-Img3

Congratulations! You just made your first FHIR client showing patients from a FHIR server.

Finally

I will end with some words about the FHIR resource Patient though. You can see there is no “Rob” in any of the names in my output form. But didn’t we search on that? Yes, we did.

You have to know a Patient in FHIR has a complex structure for names. Name consists of an array of HumanNames. The function Name.First() in the code is not about someone’s first name, something I thought at first. It returns the first HumanName that meets a condition. But since I didn’t specify a condition to First(), we simply get the first HumanName of the list. HumanName is a combination of different fields: family, given, prefix and suffix. Once you have a HumanName, you can use the .Text property to make a decent name from the four fields involved. So, even though there is no “Rob” in the visible list in the form, there apparently are multiple HumanNames attached to the eight patients found. For each patient there is at least one HumanName with “Rob” in it. And that could also be the family name “Roberts”.

Please feel free to contact me at r.mulders@furore.com with any questions.

Have fun with FHIR!
