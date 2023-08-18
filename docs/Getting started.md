# Creating LSEdit extensions

There are two main ways of creating LSEdit extensions:
- Using the LSEdit **SDK**
- Using **DAP** (Direct Access Protocol)

## LSEdit SDK
The easiest way of creating LSEdit extensions is using the official **SDK**. To create an extension in Visual Studio, follow these steps:
1. Create a **WPF library** project
2. Add a reference to *Losch.LSEdit.Extensions.dll*. It can be downloaded [here](http://example.com).
3. Create one or more classes that implement the **IPackage** interface, which is the main interface for all LSEdit extensions.

Here is an example for a basic extension that adds an entry to the main menu:
````csharp
using Losch.LSEdit.Extensions;
using System.Collections.Generic;
using System.Windows;
using System.Windows.Controls;

public class MyExtension : IPackage, IMenu
{
    public PackageMetadata Metadata => new();

    public string ComponentName => "MyExtension";
    public string ComponentDescription => "My description";

    public List<MenuItem> TopLevelMenuItems => GetMenu();

    private List<MenuItem> GetMenu()
    {
        List<MenuItem> menuItems = new();

        MenuItem menu = new() { Header = "SDK" };

        MenuItem submenu = new() { Header = "Submenu" };
        submenu.Click += (_, _) =>
        {
            MessageBox.Show("Hello World!");
        };
        
        menu.Items.Add(submenu);

        menuItems.Add(menu);

        return menuItems;
    }
}
````
As you can see, creating extensions using the SDK is very easy and requires little boilerplate code. The problem is that, as of now, the SDK is very limited in features. This is where the **Direct Access Protocol** comes in.

## DAP Extensions
The **Direct Access Protocol** is a purely reflection-based way of writing LSEdit extensions. It is not limited by the SDK featureset and thus allows creating almost all types of extensions imaginable.

To create an extension using this method, simply create a WPF library project. This is the code for the same extension as above:
````csharp
using System.Windows;
using System.Windows.Controls;
using System.Linq;

namespace DapExtension;

public class Extension
{
    public static void ExtMain(Application app)
    {
        // Get the children of the LSEdit main grid
        UIElementCollection _mgChildren = (app.MainWindow.Content as Grid).Children;

        // Convert to an array
        object[] mainGridChildren = new object[_mgChildren.Count];
        _mgChildren.CopyTo(mainGridChildren, 0);

        // Get the title bar grid...
        Grid grid = (Grid)mainGridChildren.Where(o => Grid.GetRow((UIElement)o) == 0 && Grid.GetColumn((UIElement)o) == 0 && Grid.GetColumnSpan((UIElement)o) == 7).First();

        // ...and its children
        UIElementCollection _cgChildren = grid.Children;

        // Convert those to an array as well
        object[] childGridChildren = new object[_cgChildren.Count];
        _cgChildren.CopyTo(childGridChildren, 0);

        // Get the main menu
        Menu menu = (Menu)childGridChildren.Where(o => Grid.GetColumn((UIElement)o) == 2).First();

        MenuItem topLevelMenuItem = new()
        {
            Header = "DAP"
        };

        MenuItem subItem = new()
        {
            Header = "Submenu"
        };

        subItem.Click += (_, _) =>
        {
            MessageBox.Show("Hello World!");
        };

        topLevelMenuItem.Items.Add(subItem);

        menu.Items.Add(topLevelMenuItem);
    }
}
````
The entry point of a DAP extension is always called **ExtMain(Application)**. This method will be called by LSEdit on startup with a reference to the current LSEdit instance.

As you can see, this method of creating extensions is a lot more powerful, but at the same time it is more complicated to develop and could stop working when a change to the LSEdit UI is made.
