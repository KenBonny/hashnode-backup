---
title: "Comparing a PDF to a golden master"
datePublished: Mon Nov 13 2017 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0ir5y300x7yes147dnggo3
slug: comparing-a-pdf-to-a-golden-master
tags: csharp, unit-testing, testing, dotnet, dotnetcore

---


To create a report, I had to combine the contents of several PDFs into one. Thanks to [iTextSharp](https://www.nuget.org/packages/iTextSharp/), it's really easy. Then I had the problem, how do I test this?

When I started this feature, I got an example how the PDF should look like. I knew how to get to the information to recreate the example PDF. So I had a golden master to test against and I could recreate it. Now I had to be sure that the test compared the generated PDF correctly to the original.

The first and most easy solution was to load both files with `System.IO.File.ReadAllBytes` and check if the `byte` arrays are the same. They never were. On several test runs, each time a different byte caused the comparison to fail.

After a bit of searching (and no small amount of swearing), I discovered that when you load a PDF with `ReadAllBytes`, you also load the metadata. Since two files can never be created at the same time, this option was no good.

After a bit of thinking (and a few more swears), I remembered that I can load a page via iTextSharp. So I wrote a bit of code that loads all pages from a PDF into a `byte` array and I compared those arrays. Lo and behold, the test passed.

Here is the code to load the content of a PDF:

```
private byte[] GetPdfContent(string filename)
{
  if (!File.Exists(filename)) return new byte[0];
  var content = new List<byte>();
  using (var reader = new PdfReader(filename))
  {
    for (var page = 1; page <= reader.NumberOfPages; page++)
    {
      var pageContent = reader.GetPageContent(page);
      content.AddRange(pageContent);
    }
  }
  return content.ToArray();
}
```

> Small note, the pages start from position 1, not 0.

The test itself then becomes very easy:

```
public void VerifyGeneratedPdf()
{
  var goldenMasterContent = GetPdfContent("goldenMasterLoction");
  var generatedPdfContent = GetPdfContent("generatedPdfLocation");
  Assert.AreEqual(goldenMasterContent, generatedPdfContent);
}
```

It would also be possible to get the `byte` array from the code that generates the PDF. In this case, I want to store the PDF for later use, so I load the stored PDF.

So if you want to check if a PDF matches another PDF, just load every page one by one and compare those `byte` arrays.

**EDIT**:

Unfortunately, this broke after I changed some code to make the generation of the PDF more efficient. The `goldenMasterContent` contains approximately 4000 bytes and after optimizing the PDF generation, the `generatedPdfContent` contains only about 300 bytes. So there is a huge mismatch, yet the two PDFs contain the same data and look the same. Unfortunately they are now very different and I was back to square one.

To solve this little conundrum, I decided to do a less accurate check and load the text from the PDFs and compare that. The new `GetPdfContent` method now looks like this:

```
private static string GetPdfContent(string filename)
{
  if (!File.Exists(filename)) return string.Empty;
  var pdf = new StringBuilder();
  using (var reader = new PdfReader(filename))
  {
    ITextExtractionStrategy strategy = new SimpleTextExtractionStrategy();
    for (var page = 1; page <= reader.NumberOfPages; page++)
    {
      var currentText = PdfTextExtractor.GetTextFromPage(reader, page, strategy);
      currentText = Encoding.UTF8.GetString(Encoding.Convert(Encoding.Default,
                             Encoding.UTF8,
                             Encoding.Default.GetBytes(currentText)));
      pdf.Append(currentText);
    }
  }
  return pdf.ToString();
}
```

Now it works more consistently, but the check is less accurate. I'm not happy with it because it doesn't check layout, but this is what I'll use until I find a better solution.
