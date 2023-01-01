# Demo Extension - Line Counter
This extension adds a menu entry that counts the lines of all opened documents.

__Helpers.cs:__
````csharp
using System.Windows.Controls;

namespace DapExtensionsDemo;

internal static class Helpers
{
    public static object[] ToArray(this UIElementCollection collection)
    {
        object[] array = new object[collection.Count];
        collection.CopyTo(array, 0);
        return array;
    }
}
````

__LineCountExtension.cs:__
````csharp
using System.Windows;
using System.Windows.Controls;
using System.Linq;
using System.Collections.ObjectModel;
using System;

namespace DapExtensionsDemo;

public class LineCountExtension
{
    private static Application app;

    public static void ExtMain(Application _app)
    {
        app = _app;

        InitializeMenu();
    }

    private static void InitializeMenu()
    {
        Grid grid = (Grid)(app.MainWindow.Content as Grid).Children.ToArray().Where(o => Grid.GetRow((UIElement)o) == 0 && Grid.GetColumn((UIElement)o) == 0 && Grid.GetColumnSpan((UIElement)o) == 7).First();

        Menu menu = (Menu)grid.Children.ToArray().Where(o => Grid.GetColumn((UIElement)o) == 2).First();

        MenuItem topLevelMenuItem = new()
        {
            Header = "Analysis"
        };

        MenuItem subItem = new()
        {
            Header = "Count lines"
        };

        subItem.Click += CountLines;

        topLevelMenuItem.Items.Add(subItem);

        menu.Items.Insert(6, topLevelMenuItem);
    }

    private static void CountLines(object sender, RoutedEventArgs e)
    {
        dynamic viewModel = app.MainWindow.DataContext;

        if (viewModel.Tabs.Count > 0)
        {
            int lineCount = 0;

            ObservableCollection<TabItem> tabs = viewModel.Tabs;

            foreach (TabItem tab in tabs)
            {
                try
                {
                    dynamic editor = (((tab.Content as UserControl).Content as Grid).Children.ToArray().Where(o => Grid.GetRow((UIElement)o) == 1).First() as Grid).Children[0];

                    lineCount += editor.LineCount;
                }
                catch (NullReferenceException)
                {
                    continue;
                }
            }

            MessageBox.Show($"Line count of opened files: {lineCount}");
        }
        else
        {
            MessageBox.Show("No files opened");
        }
    }
}
````
