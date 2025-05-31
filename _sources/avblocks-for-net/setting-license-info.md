---
title: Setting License Info
html_meta:
    description: This topic describes how to use an AVBlocks license file.
taxonomy:
    category: docs
---

# Setting License Info

This topic describes how to use an AVBlocks license file.

## AVBlocks License File

The AVBlocks license file is sent to you via email when you purchase a license. The license file is an XML document that looks similar to this one: 

``` xml
<!-- 
This file contains PrimoSoftware license(s). 
Line breaks and indentation between elements can be edited, 
but any other reformatting may invalidate the license(s). 
-->
<primoSoftware>
  <license version='1.0'>
    <token>abcdefghijklmnlo</token>
    <revision>1</revision>
    <issueDate>2013-05-29</issueDate>
    <expireDate>2015-05-30</expireDate>
    <updateTime>2014-11-02 22:59:08</updateTime>
    <item id='avb-win'>
      <product>avb</product>
    </item>
    <item id='avb-mac'>
      <product>avb</product>
    </item>
    <item id='avb-linux'>
      <product>avb</product>
    </item>
    <item id='avb-linux-debian'>
      <product>avb</product>
    </item>
    <signature>very_very_long_signature_string_here</signature>
  </license>
</primoSoftware>
```

You have to pass the XML as a string to Library.SetLicense. 

## Example

`````{tab-set}

````{tab-item} C&#35;
In C#, it is possible to use a string literal, e.g. `@"xml goes here"`:    

```{code-block} csharp
using System;
using System.Diagnostics;

using PrimoSoftware.AVBlocks;

namespace SetLicense
{
    class Program
    {
        // Primo Software License in XML format. 
        // You will have this after obtaining a commercial license 
        private const string licenseXml = 
            @"
            <!-- 
            This file contains PrimoSoftware license(s). 
            Line breaks and indentation between elements can be edited, 
            but any other reformatting may invalidate the license(s). 
            -->
            <primoSoftware>
              <license version='1.0'>
                <token>abcdefghijklmnlo</token>
                <revision>1</revision>
                <issueDate>2013-05-29</issueDate>
                <expireDate>2015-05-30</expireDate>
                <updateTime>2014-11-02 22:59:08</updateTime>
                <item id='avb-win'>
                  <product>avb</product>
                </item>
                <item id='avb-mac'>
                  <product>avb</product>
                </item>
                <item id='avb-linux'>
                  <product>avb</product>
                </item>
                <item id='avb-linux-debian'>
                  <product>avb</product>
                </item>
                <signature>very_very_long_signature_string_here</signature>
              </license>
            </primoSoftware>
            ";

        static void Main(string[] args)
        {
            Library.Initialize();

            // Optionally enable TLS for license verification. 
            // This must be done before calling Library.SetLicense
            Library.SetLicenseTls(true);

            // Pass the license XML as string to Library.SetLicense
            // NOTE: If the license expireDate is in the past, 
            // this will trigger a license verification call 
            // to Primo Software over the Internet. 
            Library.SetLicense(licenseXml);

            Debug.Assert(LicenseStatusFlags.Ready == Library.LicenseStatus);

            Library.Shutdown();
        }
    }
}
```
````

````{tab-item} Visual Basic

In VB, you can use XElement and [XML literals](https://msdn.microsoft.com/en-us/library/bb384629.aspx). Note that, in order for this to work, you have to remove the comment from the top of the XML text:

```{code-block} vbnet
Imports PrimoSoftware.AVBlocks

Class Program

    ' Primo Software License in XML format. 
    ' You will have this after obtaining a commercial license 
    Private Shared licenseXml As XElement =
            <primoSoftware>
                <license version='1.0'>
                    <token>abcdefghijklmnlo</token>
                    <revision>1</revision>
                    <issueDate>2013-05-29</issueDate>
                    <expireDate>2015-05-30</expireDate>
                    <updateTime>2014-11-02 22:59:08</updateTime>
                    <item id='avb-win'>
                        <product>avb</product>
                    </item>
                    <item id='avb-mac'>
                        <product>avb</product>
                    </item>
                    <item id='avb-linux'>
                        <product>avb</product>
                    </item>
                    <item id='avb-linux-debian'>
                        <product>avb</product>
                    </item>
                    <signature>very_very_long_signature_string_here</signature>
                </license>
            </primoSoftware>

    Public Shared Sub Main()
        Library.Initialize()

        ' Optionally enable TLS for license verification. 
        ' This must be done before calling Library.SetLicense
        Library.SetLicenseTls(True)
        
        ' Pass the license XML as string to Library.SetLicense
        ' NOTE: If the license expireDate is in the past, 
        ' this will trigger a license verification call 
        ' to Primo Software over the Internet. 
        Library.SetLicense(licenseXml.ToString())
        
        Debug.Assert(LicenseStatusFlags.Ready = Library.LicenseStatus)

        Library.Shutdown()
    End Sub

End Class
```
````

`````
