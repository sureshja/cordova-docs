# Create a Cordova plugin for Windows and Windows Phone

A Cordova plugin is a cross-platform library that accesses native code and device capabilities through a JavaScript interface. When required, the plugin also updates the platform manifest to enable device capabilities. In this tutorial, you will create a Cordova plugin for Windows Phone, <span class="input">ToUpperPlugin</span>, add it to a Cordova app, and call the plugin method from the app. The plugin has one method, <span class="input">ToUpper</span>, that converts a string to uppercase. The plugin supports both Windows Phone 8 and Windows Phone (Universal). The Windows Phone 8 plugin uses C# and the Windows Phone (Universal) plugin uses JavaScript.

The tutorial assumes you have some familiarity with Cordova plugins. For an introduction to plugins, see <span>[Manage Plugins for Apps Built with Visual Studio Tools for Apache Cordova](https://msdn.microsoft.com/en-us/library/dn757051.aspx)</span> and [Apache Cordova Overview](http://cordova.apache.org/docs/en/4.0.0/guide_overview_index.md.html#Overview).

The early sections of the tutorial take you straight through the process with little detail. The later sections explain the code in more detail and will help you create, configure, debug, and deploy your own plugins.

The [Windows Phone Plugin Generator](http://download.microsoft.com/download/5/B/4/5B433693-63A4-4509-A6F5-17A892A7D59E/PluginSourceAndApp.zip) download includes the code for this tutorial and an app that generates a simple plugin.

*   <span>[Create the Cordova plugin](#CreateCordova)</span>

*   <span>[Add the plugin to the Cordova project](#AddPlugin)</span>

*   <span>[Call the plugin from your app](#CallPlugin)</span>

*   <span>[Run the app with your plugin](#RunTheApp)</span>

*   <span>[Creating a plugin method for Windows Phone 8](#CreatingMethod)</span>

*   <span>[Creating a plugin method for Windows Phone 8 (Universal)](#CreatingUniversal)</span>

*   <span>[Configuring the plugin.xml file](#ConfiguringXML)</span>

*   <span>[Creating the JavaScript plugin interface file](#CreatingInterface)</span>

*   <span>[Calling plugin methods from your app](#CallingPlugins)</span>

*   <span>[How to remove a local plugin from your app](#RemovePlugin)</span>

*   <span>[Developing and debugging the plugin code](#DebugPlugin)</span>

*   <span>[Troubleshooting](#Troubleshooting)</span>

*   <span>[Next steps](#NextSteps)</span>

##<a id="CreateCordova"></a>Create the Cordova plugin

The <span class="input">ToUpperPlugin</span> plugin needs four files.

*   A native C# file that contains the plugin methods for Windows Phone 8\.

*   A native JavaScript file that contains the plugin methods for Windows Phone (Universal).

*   A JavaScript file that calls <span class="code">cordova.exec</span>. This file is the bridge between the JavaScript code in your app and the native code in your plugin.

*   An XML file that describes your plugin, defines the supported platforms, and specifies the locations of the C# file and the JavaScript file.

<div>

### Create folders

You start by creating the following folder and file structure. While this is not the only folder structure you can use, it is the structure that many plugins follow. The folder structure looks like this:

![The file structure of a plugin.](https://i-msdn.sec.s-msft.com/dynimg/IC776843.png "The file structure of a plugin.")1.  Create a root folder for your plugin and name it <span class="input">ToUpperPlugin</span>.

1.  In the <span class="input">ToUpperPlugin</span> folder, create a folder named <span class="input">src</span>.

2.  In the <span class="input">src</span> folder, create a folder named <span class="input">wp</span>.

3.  In the <span class="input">src</span> folder, create a folder named <span class="input">windows</span>.

4.  In the <span class="input">ToUpperPlugin</span> folder, create a folder named <span class="input">www</span.

### Add files

Now you add the files.

1.  In the <span class="input">wp</span> folder, add an empty C# class file, <span class="input">ToUpperPlugin.cs</span>.

4.  In the <span class="input">windows</span> folder, add an empty JavaScript file, <span class="input">ToUpperPluginProxy.js</span>.

7.  In the <span class="input">www</span> folder, add an empty JavaScript file, <span class="input">ToUpperPlugin.js</span>.

10.  In the <span class="input">ToUpperPlugin</span> folder, add an empty xml file, <span class="input">plugin.xml</span>.

You now have the files you need for the plugin:

*   <span class="input">ToUpperPlugin\plugin.xml</span>

*   <span class="input">ToUpperPlugin\src\wp\ToUpperPlugin.cs</span>

*   <span class="input">ToUpperPlugin\src\windows\ToUpperPluginProxy.cs</span>

*   <span class="input">ToUpperPlugin\www\ToUpperPlugin.js</span>

### Add the plugin method for Windows Phone 8

The native code for the Windows 8 platform is written in C#. Add the following code to the <span class="input">ToUpperPlugin.cs</span> file:

```C#
// C#

using System;

namespace WPCordovaClassLib.Cordova.Commands
{
    public class ToUpperPlugin : BaseCommand
    {
        public void ToUpper(string options)
        {
            string upperCase = JSON.JsonHelper.Deserialize<string[]>(options)[0].ToUpper();
            PluginResult result;
            if (upperCase != "")
            {
                result = new PluginResult(PluginResult.Status.OK, upperCase);
            } else
            {
                result = new PluginResult(PluginResult.Status.ERROR, upperCase);
            }

            DispatchCommandResult(result);
        }
    }
}
```

The code in the <span class="input">ToUpperPlugin.cs</span> is discussed in more detail in the <span>[Creating a plugin method for Windows Phone 8](#CreatingMethod)</span> section later in this tutorial.

</div></div><div>

### Add the plugin method for Windows Phone (Universal)

The native code for Windows Phone (Universal) is written in JavaScript. Add the following code to the <span class="input">ToUpperPluginProxy.js</span> file:

```C#
//  C#

var cordova = require('cordova'),
    ToUpperPlugin= require('./ToUpperPlugin');

module.exports = {

    ToUpper: function (successCallback, errorCallback, strInput) {

        var upperCase = strInput[0].toUpperCase();
        if(upperCase != "") {
            successCallback(upperCase);
        }
        else {
            errorCallback(upperCase);
        }
    }
};
```

require("cordova/exec/proxy").add("ToUpperPlugin", module.exports);

The code in this JavaScript file is discussed in more detail in the <span>[Creating a plugin method for Windows Phone (Universal)](#CreatingMethod)</span> section later in this tutorial

### Add code to the plugin.xml file

Add the following code to the <span class="input">plugin.xml</span> file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<plugin xmlns="http://apache.org/cordova/ns/plugins/1.0"
      id="com.contoso.ToUpperPlugin"
      version="0.1.0">
    <name>ToUpperPlugin</name>
    <description>ToUpper plugin for Apache Cordova</description>
    <license>MIT</license>
    <keywords></keywords>
    <repo></repo>
    <issue></issue>

    <js-module src="www/ToUpperPlugin.js" name="ToUpperPlugin">
        <clobbers target="ToUpperPlugin" />
    </js-module>

    <!-- wp8 -->
    <platform name="wp8">
        <config-file target="config.xml" parent="/*">
            <feature name="ToUpperPlugin">
                <param name="wp-package" value="ToUpperPlugin"/>
            </feature>
        </config-file>

        <source-file src="src/wp/ToUpperPlugin.cs" />
    </platform>

    <!-- windows -->
    <platform name="windows">
        <js-module src="src/windows/ToUpperPluginProxy.js" name="ToUpperPluginProxy">
            <merges target="" />
        </js-module>
    </platform>

</plugin>
```

The code in the <span class="input">plugin.xml</span> file is discussed in more detail in the <span>[Configuring the plugin.xml file](#ConfiguringXML)</span> section later in this tutorial.

### Add code to the JavaScript plugin interface file

Add the following code to the <span class="input">ToUpperPlugin.js</span> file:

```javascript
var ToUpperPlugin = {
    ToUpper: function (successCallback, errorCallback, strInput) {
        cordova.exec(successCallback, errorCallback, "ToUpperPlugin", "ToUpper", [strInput]);
    }
}

module.exports = ToUpperPlugin;
```

The code in <span class="input">ToUpperPlugin.js</span> is discussed in more detail in the <span>[Creating the JavaScipt plugin interface file](#CreatingInterface)</span> section later in this tutorial.

Your plugin is complete and ready to add to a project.

## <a id="AddPlugin"></a>Add the plugin to the Cordova project

1.  Create a new Cordova project.

2.  Right-click <span class="input">config.xml</span> in <span class="label">Solution Explorer</span> and select <span class="label">View Designer</span>.

6.  Select the <span class="label">Plugins</span> tab.

8.  Select the <span class="label">Custom</span> heading.

10.  Select <span class="label">Local</span>.

12.  Browse to the top of the plugin folder structure.

13.  Click <span class="label">Add</span>.

    Visual Studio adds the plugin to your project and creates the folder structure shown in the following image:

    ![Project structure of a plugin in a Cordova app.](https://i-msdn.sec.s-msft.com/dynimg/IC776844.png "Project structure of a plugin in a Cordova app.")
15.  Save the project.

16.  To run the app on Windows 8, choose the <span class="label">Windows Phone 8</span> platform and the <span class="label">Emulator WVGA 512MB</span>, as shown in the following image.

    ![Selecting WP emulator from the toolbar.](https://i-msdn.sec.s-msft.com/dynimg/IC780863.png "Selecting WP emulator from the toolbar.")
19.  Press F5 to run your project and verify that it runs the Windows Phone emulator without error as shown in the following image.

    ![The app running in a WP emulator.](https://i-msdn.sec.s-msft.com/dynimg/IC775935.png "The app running in a WP emulator.")
20.  To run the app on Windows 8 (Universal), choose the <span class="label">Windows Phone (Universal)</span> platform and the <span class="label">Emulator 8.1 WVGA 4 inch 512MB</span>.

23.  Press F5 to run your project and verify that it runs on the Windows Phone emulator.

Now that you have added the plugin to the project, you are ready to call the <span class="code">ToUpper</span> method from your code.

<div class="alert"><table><tbody><tr><th align="left">![Note](https://i-msdn.sec.s-msft.com/areas/global/content/clear.gif "Note")**Note**</th></tr><tr><td>

You cannot debug the Windows Phone Cordova project. There are a couple of options for debugging the app. The first is to use the platform-specific project as described in the section <span>[Developing and debugging the plugin code](#DebugPlugin)</span> later in this tutorial. Another option is to use [Web Inspector Remote](http://msopentech.com/blog/2013/05/31/now-on-ie-and-firefox-debug-your-mobile-html5-page-remotely-with-weinre-web-inspector-remote/).


## <a id="CallPlugin"></a>Call the plugin from your app

Open <span class="input">index.html</span> and add this code before the closing tag <span class="code"></body></span>.

<div class="codeSnippetContainer" id="code-snippet-5" xmlns=""><div class="codeSnippetContainerTabs"><div class="codeSnippetContainerTabSingle" dir="ltr"><a>HTML</a></div></div><div class="codeSnippetContainerCodeContainer"><div class="codeSnippetToolBar"><div class="codeSnippetToolBarText">[Copy](javascript:if (window.epx.codeSnippet)window.epx.codeSnippet.copyCode('CodeSnippetContainerCode_8dc2f118-81c7-430e-b451-5c7e1c3fb452'); "Copy to clipboard.")</div></div><div class="codeSnippetContainerCode" id="CodeSnippetContainerCode_8dc2f118-81c7-430e-b451-5c7e1c3fb452" dir="ltr"><div style="color:Black;"></div></div></div></div>

The code you added to <span class="input">index.html</span> is discussed in more detail in the<span>[Calling Plugin Methods from Your App](#CallingPlugins)</span> section later in the tutorial

## <a id="RunTheApp"></a>Run the app with your plugin

1.  Press F5 to run your project, as shown in the following image:

    ![The app with input box and button.](https://i-msdn.sec.s-msft.com/dynimg/IC775936.png "The app with input box and button.")
2.  Verify that the app returns a successful result if you enter text in the input box and returns an error result if the input box is empty.

You now have a complete plugin and a project that uses the plugin. The following sections describe the files in more detail.

## <a id="CreatingMethod"></a>Creating a plugin method for Windows Phone 8

This section discusses the C# plugin method in more detail. Here’s the code you used for your plugin:

<div class="codeSnippetContainer" id="code-snippet-6" xmlns=""><div class="codeSnippetContainerTabs"><div class="codeSnippetContainerTabSingle" dir="ltr"><a>C#</a></div></div><div class="codeSnippetContainerCodeContainer"><div class="codeSnippetToolBar"><div class="codeSnippetToolBarText">[Copy](javascript:if (window.epx.codeSnippet)window.epx.codeSnippet.copyCode('CodeSnippetContainerCode_e82f8d3a-5b52-4853-ab26-6dcd143344ca'); "Copy to clipboard.")</div></div><div class="codeSnippetContainerCode" id="CodeSnippetContainerCode_e82f8d3a-5b52-4853-ab26-6dcd143344ca" dir="ltr"><div style="color:Black;"></div></div></div></div>

There are a few key elements that you have to get right so that you can call your plugin method and so that you can get results from your plugin method.

*   **Namespace.** Your plugin class must exist in the <span class="code">WPCordovaClassLib.Cordova.Commands</span> namespace. That namespace, which is added by Visual Studio to the Windows Phone version of the app, contains the <span class="code">BaseCommand</span> class that you must derive from.

*   **Base class.** You can name your plugin class whatever you like, but it must derive from <span class="code">BaseCommand</span>. The <span class="code">BaseCommand</span> class is created by Visual Studio and added to the Windows Phone project. You can find the class in the <span class="code">cordovalib</span> folder of the Windows Phone project. It’s in the <span class="code">WPCordovaClassLib.Cordova.Commands</span> namespace. The location is shown in the following image.

    ![The location of BaseCommand in the project.](https://i-msdn.sec.s-msft.com/dynimg/IC775937.png "The location of BaseCommand in the project.")
*   **Signature.** Your method must use this signature:

    <div class="codeSnippetContainer" id="code-snippet-7" xmlns=""><div class="codeSnippetContainerTabs"><div class="codeSnippetContainerTabSingle" dir="ltr"><a>C#</a></div></div><div class="codeSnippetContainerCodeContainer"><div class="codeSnippetToolBar"><div class="codeSnippetToolBarText">[Copy](javascript:if (window.epx.codeSnippet)window.epx.codeSnippet.copyCode('CodeSnippetContainerCode_aac4c6fb-02c7-4ad5-ab3d-8adafb0282f2'); "Copy to clipboard.")</div></div><div class="codeSnippetContainerCode" id="CodeSnippetContainerCode_aac4c6fb-02c7-4ad5-ab3d-8adafb0282f2" dir="ltr"><div style="color:Black;"></div></div></div></div>

    You can change <span class="code">MethodName</span> and <span class="code">options</span> to something more meaningful for your app, but the rest must stay the same. The arguments that are passed to the method are converted into a JSON object. You need to parse the JSON to retrieve the arguments. For example, in <span class="input">ToUpperPlugin</span>, the string is passed as the first item in an array that is converted to JSON. Visual Studio includes a JSON visualizer to view the contents of a JSON package. To display the visualizer, in debug mode, mouse over the variable containing JSON. From the context menu, select JSON visualizer, as shown in the following image. For information on how to debug the Windows Phone project, see the section <span>[Developing and debugging the C# plugin code](#DebugPlugin)</span> later in this tutorial.

    ![How to select the JSON visualizer.](https://i-msdn.sec.s-msft.com/dynimg/IC775938.png "How to select the JSON visualizer.")![The JSON visualizer.](https://i-msdn.sec.s-msft.com/dynimg/IC775939.png "The JSON visualizer.")
*   **PluginResult.** If you want to return something from your method to your app, or if you want to trigger either a success callback method or an error callback method in your app, you need to create and dispatch a <span class="code">PluginResult</span> object. In the constructor for <span class="code">PluginResult</span>, you indicate whether the method succeeded or not, and you include the value to be passed back to your app. The code follows this pattern:

    <div class="codeSnippetContainer" id="code-snippet-8" xmlns=""><div class="codeSnippetContainerTabs"><div class="codeSnippetContainerTabSingle" dir="ltr"><a>C#</a></div></div><div class="codeSnippetContainerCodeContainer"><div class="codeSnippetToolBar"><div class="codeSnippetToolBarText">[Copy](javascript:if (window.epx.codeSnippet)window.epx.codeSnippet.copyCode('CodeSnippetContainerCode_b980a570-4e3e-4f2f-9280-a463ad77744a'); "Copy to clipboard.")</div></div><div class="codeSnippetContainerCode" id="CodeSnippetContainerCode_b980a570-4e3e-4f2f-9280-a463ad77744a" dir="ltr"><div style="color:Black;"></div></div></div></div>


## <a id="CreatingUniversal"></a>Creating a plugin method for Windows Phone 8 (Universal)

This section discusses the JavaScript plugin method in more detail. Here’s the code you used for your plugin:

<div class="codeSnippetContainer" id="code-snippet-9" xmlns=""><div class="codeSnippetContainerTabs"><div class="codeSnippetContainerTabSingle" dir="ltr"><a>JavaScript</a></div></div><div class="codeSnippetContainerCodeContainer"><div class="codeSnippetToolBar"><div class="codeSnippetToolBarText">[Copy](javascript:if (window.epx.codeSnippet)window.epx.codeSnippet.copyCode('CodeSnippetContainerCode_3d6f83c8-44a2-4117-9f43-1982e23e9b00'); "Copy to clipboard.")</div></div><div class="codeSnippetContainerCode" id="CodeSnippetContainerCode_3d6f83c8-44a2-4117-9f43-1982e23e9b00" dir="ltr"><div style="color:Black;"></div></div></div></div>

There are a few key elements that you have to get right so that you can call your plugin method and so that you can get results from your plugin method.

*   **Exports**. The <span class="code">module.exports</span> and <span class="code">require</span> statements make the <span class="code">ToUpper</span> function available outside the source file as part of the plugin.

*   **Signature.** Your method must use this signature:

    <div class="codeSnippetContainer" id="code-snippet-10" xmlns=""><div class="codeSnippetContainerTabs"><div class="codeSnippetContainerTabSingle" dir="ltr"><a>JavaScript</a></div></div><div class="codeSnippetContainerCodeContainer"><div class="codeSnippetToolBar"><div class="codeSnippetToolBarText">[Copy](javascript:if (window.epx.codeSnippet)window.epx.codeSnippet.copyCode('CodeSnippetContainerCode_8f9aa6d2-accd-4055-bd95-53fa2e2f35fc'); "Copy to clipboard.")</div></div><div class="codeSnippetContainerCode" id="CodeSnippetContainerCode_8f9aa6d2-accd-4055-bd95-53fa2e2f35fc" dir="ltr"><div style="color:Black;"></div></div></div></div>

    You can change method name and parameter names to something more meaningful for your app. The <span class="code">args</span> parameter is an array.

*   **Callback functions**. If you want to return something from your method to your app, you can use the <span class="code">sucessCallback</span> and <span class="code">errorCallback</span> methods.


## <a id="ConfiguringXML"></a>Configuring the plugin.xml file

This section discusses the <span class="input">plugin.xml</span> file in more detail. Here’s the plugin.xml file that you created.

<div class="codeSnippetContainer" id="code-snippet-11" xmlns=""><div class="codeSnippetContainerTabs"><div class="codeSnippetContainerTabSingle" dir="ltr"><a>Xml</a></div></div><div class="codeSnippetContainerCodeContainer"><div class="codeSnippetToolBar"><div class="codeSnippetToolBarText">[Copy](javascript:if (window.epx.codeSnippet)window.epx.codeSnippet.copyCode('CodeSnippetContainerCode_7207435a-0f50-4b76-8e11-71cd1857bf0a'); "Copy to clipboard.")</div></div><div class="codeSnippetContainerCode" id="CodeSnippetContainerCode_7207435a-0f50-4b76-8e11-71cd1857bf0a" dir="ltr"><div style="color:Black;"></div></div></div></div>

The XML contains the minimum elements you need to get your plugin working. There are a few key elements that you have to get right so Visual Studio and Cordova can find your plugin files and call your plugin methods.

*   **<plugin>** The <span class="code">id</span> attribute that you add to the <span class="code"><plugin></span> element is used to by Visual Studio to create a project folder with the plugin files. It is the name of your plugin and it is carried over into the project’s <span class="code">config.xml</span> file. Here is how it appears in <span class="code">plugin.xml</span>, part of the plugin:

    <div class="codeSnippetContainer" id="code-snippet-12" xmlns=""><div class="codeSnippetContainerTabs"><div class="codeSnippetContainerTabSingle" dir="ltr"><a>Xml</a></div></div><div class="codeSnippetContainerCodeContainer"><div class="codeSnippetToolBar"><div class="codeSnippetToolBarText">[Copy](javascript:if (window.epx.codeSnippet)window.epx.codeSnippet.copyCode('CodeSnippetContainerCode_687486a1-195e-4297-8716-67a0658542bd'); "Copy to clipboard.")</div></div><div class="codeSnippetContainerCode" id="CodeSnippetContainerCode_687486a1-195e-4297-8716-67a0658542bd" dir="ltr"><div style="color:Black;"></div></div></div></div>

    This is how it appears in <span class="code">config.xml</span>, part of the Cordova project:

    <div class="codeSnippetContainer" id="code-snippet-13" xmlns=""><div class="codeSnippetContainerTabs"></div><div class="codeSnippetContainerCodeContainer"><div class="codeSnippetToolBar"><div class="codeSnippetToolBarText">[Copy](javascript:if (window.epx.codeSnippet)window.epx.codeSnippet.copyCode('CodeSnippetContainerCode_af5e710a-3550-427b-a9ca-a0b115a297f0'); "Copy to clipboard.")</div></div><div class="codeSnippetContainerCode" id="CodeSnippetContainerCode_af5e710a-3550-427b-a9ca-a0b115a297f0" dir="ltr"><div style="color:Black;"></div></div></div></div>
*   **<js-module>** The <span class="code"><js-module></span> element specifies the location and filename of your plugin interface file, in this case <span class="code">ToUpperPlugin.js</span>. The interface file specifies the JavaScript signatures of your plugin methods and makes the all-important call to <span class="code">cordova.exec</span>. The <span class="code"><clobbers></span> element specifies the plugin name that you will use in your code that calls the plugin. The <span class="code">src</span> attribute specifies the name of the file and its location relative to the <span class="code">plugin.xml</span> file.

    <div class="codeSnippetContainer" id="code-snippet-14" xmlns=""><div class="codeSnippetContainerTabs"><div class="codeSnippetContainerTabSingle" dir="ltr"><a>Xml</a></div></div><div class="codeSnippetContainerCodeContainer"><div class="codeSnippetToolBar"><div class="codeSnippetToolBarText">[Copy](javascript:if (window.epx.codeSnippet)window.epx.codeSnippet.copyCode('CodeSnippetContainerCode_edc413d9-d2c6-490d-ac5b-8319f0a059a1'); "Copy to clipboard.")</div></div><div class="codeSnippetContainerCode" id="CodeSnippetContainerCode_edc413d9-d2c6-490d-ac5b-8319f0a059a1" dir="ltr"><div style="color:Black;"></div></div></div></div>
*   **<platform name="wp8">** The <span class="code"><platform></span> element specifies the location and filename of the native code file, in this case <span class="code">ToUpperPlugin.cs</span>. The <span class="code">name</span> attribute is limited to a set of names that are supported by Cordova. In this case, <span class="code">"wp8"</span> is used for Windows Phone 8\. The <span class="code"><config-file></span> element contains plugin information that is added to the <span class="code">config.xml</span> file of the Cordova project. It makes the Windows Phone project aware of the plugin source. The <span class="code"><source-file></span> element specifies the location and filename of the code file.

    <div class="codeSnippetContainer" id="code-snippet-15" xmlns=""><div class="codeSnippetContainerTabs"><div class="codeSnippetContainerTabSingle" dir="ltr"><a>Xml</a></div></div><div class="codeSnippetContainerCodeContainer"><div class="codeSnippetToolBar"><div class="codeSnippetToolBarText">[Copy](javascript:if (window.epx.codeSnippet)window.epx.codeSnippet.copyCode('CodeSnippetContainerCode_0bc284db-7489-483b-b525-6e229e7f669c'); "Copy to clipboard.")</div></div><div class="codeSnippetContainerCode" id="CodeSnippetContainerCode_0bc284db-7489-483b-b525-6e229e7f669c" dir="ltr"><div style="color:Black;"></div></div></div></div>
*   **<platform name="windows">** This element specifies the location and filename of the native code file for the Windows Phone (Universal) platform. The <span class="code"><js-module></span> element specifies the location and filename of the code file.

    <div class="codeSnippetContainer" id="code-snippet-16" xmlns=""><div class="codeSnippetContainerTabs"><div class="codeSnippetContainerTabSingle" dir="ltr"><a>Xml</a></div></div><div class="codeSnippetContainerCodeContainer"><div class="codeSnippetToolBar"><div class="codeSnippetToolBarText">[Copy](javascript:if (window.epx.codeSnippet)window.epx.codeSnippet.copyCode('CodeSnippetContainerCode_fa5be9be-1604-4b3f-a17b-d976ba156bc6'); "Copy to clipboard.")</div></div><div class="codeSnippetContainerCode" id="CodeSnippetContainerCode_fa5be9be-1604-4b3f-a17b-d976ba156bc6" dir="ltr"><div style="color:Black;"></div></div></div></div>


## <a id="CreatingInterface"></a>Creating the JavaScript plugin interface file

This section describes the JavaScript interface file in more detail. Here’s the <span class="input">toupperplugin.js</span> file that you created.

<div class="codeSnippetContainer" id="code-snippet-17" xmlns=""><div class="codeSnippetContainerTabs"><div class="codeSnippetContainerTabSingle" dir="ltr"><a>JavaScript</a></div></div><div class="codeSnippetContainerCodeContainer"><div class="codeSnippetToolBar"><div class="codeSnippetToolBarText">[Copy](javascript:if (window.epx.codeSnippet)window.epx.codeSnippet.copyCode('CodeSnippetContainerCode_87f2b03d-f922-45ab-9fe5-30192b09e113'); "Copy to clipboard.")</div></div><div class="codeSnippetContainerCode" id="CodeSnippetContainerCode_87f2b03d-f922-45ab-9fe5-30192b09e113" dir="ltr"><div style="color:Black;"></div></div></div></div>

There are a few key elements that you have to get right so Visual Studio and Cordova can find your plugin files and call your plugin methods.

*   **ToUpper.** The function specifies how you call the function from your Cordova app. In this case you have three arguments, two callback functions and one string. The arguments passed to the native code vary by platform. For example, the call to the <span class="code">ToUpper</span> method in the <span class="input">ToUpperPluginProxy.js</span> file receives all three arguments while the call to <span class="code">ToUpperPlugin.cs</span> receives only one argument.

*   **cordova.exec.** The call to <span class="code">cordova.exec</span> has four required parameters. The first two are the success and error callback functions. The third is the name of the service (or plugin). The fourth is the name of the method. In Cordova, this is called the action. Any remaining arguments are passed as an array to the plugin method.

*   **module.exports.** The last line of code makes the <span class="code">ToUpper</span> function available to the rest of the project.

## <a id="CallingPlugins"></a>Calling plugin methods from your app

This section describes the code in <span class="input">index.html</span> in more detail. Here’s the code you added that calls the plugin method, <span class="code">ToUpper</span>.

<div class="codeSnippetContainer" id="code-snippet-18" xmlns=""><div class="codeSnippetContainerTabs"><div class="codeSnippetContainerTabSingle" dir="ltr"><a>HTML</a></div></div><div class="codeSnippetContainerCodeContainer"><div class="codeSnippetToolBar"><div class="codeSnippetToolBarText">[Copy](javascript:if (window.epx.codeSnippet)window.epx.codeSnippet.copyCode('CodeSnippetContainerCode_58d7839c-093c-4438-92d6-693967e85c41'); "Copy to clipboard.")</div></div><div class="codeSnippetContainerCode" id="CodeSnippetContainerCode_58d7839c-093c-4438-92d6-693967e85c41" dir="ltr"><div style="color:Black;"></div></div></div></div>

The call to <span class="code">ToUpper</span> method takes three parameters, a callback function for success, a callback function for error, and the string you want to convert to uppercase. The callback functions extract the uppercase string, contained in <span class="code">arguments</span>, and display them in the <span class="code">toUpperResult</span> paragraph element. The <span class="code">ToUpper</span> method is called from the <span class="code">ToUpperPlugin</span> module.

## <a id="RemovePlugin"></a>How to remove a local plugin from your app

The Visual Studio designer for config.xml that removes plugins does not remove local plugins. To delete your plugin:

1.  Delete the folder from the project.

2.  Delete the <span class="code">vs:plugin</span> element in the config.xml file.

## <a id="DebugPlugin"></a>Developing and debugging the plugin code

When you ran your Cordova project, Visual Studio created a Windows Phone projects in the <span class="input">platforms\wp8</span> and <span class="input">platforms\windows</span> folders of your project folder. The projects are recreated every time you build or clean your Cordova project.

You can copy the projects to a new location and use them to develop your plugin. When you have your plugin working, you need to copy the final code back to the plugin location you created at the beginning of the tutorial.

<div class="alert"><table><tbody><tr><th align="left">![Caution note](https://i-msdn.sec.s-msft.com/areas/global/content/clear.gif "Caution note")**Caution**</th></tr><tr><td>

Visual Studio will update or delete the projects every time you build your Cordova app. You need to copy the projects to a new location to prevent Visual Studio from overwriting them.

##<a="Troubleshooting"></a>Troubleshooting</span>](javascript:void(0) "Collapse")<div class="LW_CollapsibleArea_HrDiv">

These are some of the issues that might happen as you create and use your plugin.

### The plugin does not load into the Cordova project

There might be a problem with the names of the files and folders as compared to the files and folders listed in plugin.xml.

### The plugin does not load into the platform-specific project.

There might be problem in how you defined the <span class="code"><platform></span> element of <span class="input">plugin.xml</span>. There is a limited set of valid platform names. The plugin might load into the Cordova project, but at build time, the plugin files will not be added to the platform-specific projects.

### The call to the plugin fails silently.

The plugin name or the method name you used in the code does not match the plugin you created.

### It just doesn't work!

Check the following items:

*   Make sure the casing on every instance of the plugin name and the method name is consistent across all the files, including both the filenames and the text inside the files. While there is some leeway in casing as you cross boundaries within and between JavaScript and C#, be consistent and you won't have to troubleshoot casing issues.

*   Make sure the folders and files are named exactly as specified in the <span class="input">plugin.xml</span> file. A single typo can invalidate an entire plugin.

*   Carefully check any text substitutions for spelling errors you've made along the way.

*   Scale back to a minimal plugin and get that working. Then add more pieces.

*   Watch the Output window closely while the project is building. Some messages appear only briefly during the build process.


## <a id="NextSteps"></a>Next Steps

This tutorial created a very basic plugin. From here you could add any number of customizations.

*   You can share your plugin on the [Apache Cordova Plugins Registry](http://plugins.cordova.io/#/). The registry is also an excellent source of plugin samples.

*   You can add more methods to your plugin.

*   You can add more source files to your plugin. In this tutorial, you had one method in one source file. As your plugin becomes more complex, you can distribute code across source files as the architecture dictates.

*   You can support more or fewer platforms. To learn more about developing plugins for other platforms, see the [Plugin Development Guide](http://cordova.apache.org/docs/en/4.0.0/guide_hybrid_plugins_index.md.html#Plugin%20Development%20Guide).

![Download the tools](https://i-msdn.sec.s-msft.com/dynimg/IC795792.png "Download the tools") [Get the Visual Studio Tools for Apache Cordova](http://aka.ms/mchm38) or [learn more](https://www.visualstudio.com/cordova-vs.aspx

## See Also

[Windows Phone Plugin Generator](http://download.microsoft.com/download/5/B/4/5B433693-63A4-4509-A6F5-17A892A7D59E/PluginSourceAndApp.zip)
[Plugin Development Guide (Apache Cordova Documentation)](http://cordova.apache.org/docs/en/4.0.0/guide_hybrid_plugins_index.md.html#Plugin%20Development%20Guide)
  